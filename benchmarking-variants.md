# Benchmarking Parquet Variants through Iceberg

## Key Questions
1. Are variants ready to use?
2. If not, what is needed?

## Answers

1. They are surprisingly slow, especially when shedded.
   While shredding offers significant compression benefits, there is currently a tangible
   performance penalty.

2. What is needed?
   * Predicate pushdown all the way from Iceberg to the parquet reader
   * The causes of the "unexpected outcomes" in the benchmarking experiments to be identified and addressed.
     This could include identifying flaws in the benchmarks: review of those PRs is needed to give convidence in their conclusions.

At the time of the writing of the initial document (07-04-2026) it is faster to perform filtering or projection on  variant data stored in Avro in Iceberg + Spark queries than it is on data stored in Parquet.

## Relevant Pull Requests

This is a list of PRs by myself and Qiegang Long which should improve query/read time.
Sorted numerically by project.

| Project | PR                                                       | Title                                                                                  | Author         |
|---------|----------------------------------------------------------|----------------------------------------------------------------------------------------|----------------|
| Iceberg | [14707](https://github.com/apache/iceberg/issues/14707)  | Vectorized read for variant                                                            | Qiegang Long   |
| Iceberg | [15510](https://github.com/apache/iceberg/issues/15510)  | Parquet Rowgroup skipping for variant predicate                                        | Qiegang Long   |
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants                                           | Steve Loughran |
| Spark   | [54598](https://github.com/apache/spark/pull/54598)      | Enable Parquet rowgroup skipping for variant filters to improve query-time performance | Qiegang Long   |
| Spark   | [54394](https://github.com/apache/spark/pull/54394)      | Support variant_get predicate for DSv2 filter pushdown                                 | Qiegang Long   |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants                                              | Steve Loughran |


## Benchmark Design and Test Setup

Two core benchmark suites were written for Parquet And Spark, to measure:
1. Time to construct variants through builders.
2. Time to read data from a file containing shredded and unshredded variants.

In both iceberg and spark, Variant Builder performance appears to be functional with `O(n)` scalability.
Deeply nested structures are bit less efficient because the Java `HashMap` instances constructed at each level preallocate space for 16 entries.
These results are not covered in this report.

Reading data from files, with a simple test structure, produced disappointing result.
Not only are variants slow to process in queries, shredded variants are even slower to process.

### Schema

The "record" is as follows.
```
id: long -> unique per row
category: int32  (0-19)
nested: variant
    .idstr: string -> id as string
    .varid: int64  -> id
    .varcategory: int32 -> category (0-19). 0-9 or 10-19 per file
    .col4: string -> 20 values from category
```

The ID is is a row counter. Category is calculated from the file number and ID, such that
all rows in a file will be in the category range 0-9 or 10-19.

Example: the iceberg row construction, which uses iceberg structures and types

```java

  private void writeOneFile(DataWriter<Record> writer, VariantMetadata metadata, int fileNum)
      throws IOException {
    try (writer) {
      GenericRecord record = GenericRecord.create(SCHEMA);
      int categoryBase = (fileNum % 2) * 10;
      for (int i = 0; i < NUM_ROWS_PER_FILE; i++) {
        long id = (long) fileNum * NUM_ROWS_PER_FILE + i;
        int category = (int) (id % 10) + categoryBase;
        Variant variant = buildVariant(metadata, id, category, repeatedStrings[category]);
        writer.write(
            record.copy(ImmutableMap.of(COL_ID, id, "category", category, "nested", variant)));
      }
    }
  }
  
private static Variant buildVariant(
    VariantMetadata metadata, long id, int category, String col4) {
  ShreddedObject obj = Variants.object(metadata);
  obj.put("idstr", Variants.of("item_" + id));
  obj.put("varid", Variants.of(id));
  obj.put("varcategory", Variants.of(category));
  obj.put("col4", Variants.of(col4));
  return Variant.of(metadata, obj);
}  
```

The iceberg schema is minimal, as none of the fields within the variant are defined.
```java
private static final Schema SCHEMA =
  new Schema(
      required(1, "id", Types.LongType.get()),
      required(2, "category", Types.IntegerType.get()),
      required(3, "nested", Types.VariantType.get()));
```

The Parquet schema declares that there is a group `nested` of containing two required binary fields, `metadata` and `value` 
```parquet
message vschema {
  required int64 id;
  required int32 category;
  required group nested (VARIANT(1)) {
    required binary metadata;
    required binary value;
  }
}
```
*Note*: it's not clear whether the variant group should be declared as optional or not.
The examples in [the parquet format specification](https://parquet.apache.org/docs/file-format/types/variantencoding/) use `optional`.
However, as these examples needed changes to actually work (GH-561: variant schema examples to use (VARIANT(1))), they can't be considered normative.

When writing a shredded parquet file in the parquet benchmarks, the schema was expanded to declare that there was an optional group `typed_value`, inside which each shredded element was declared with full type information.

```parquet
message vschema {
  required int64 id;
  required int32 category;
  required group nested (VARIANT(1)) {
    required binary metadata;
    optional binary value;
    optional group typed_value {
      required group idstr { optional binary value; optional binary typed_value (STRING); }
      required group varid { optional binary value; optional int64 typed_value; }
      required group varcategory { optional binary value; optional int32 typed_value; }
      required group col4 { optional binary value; optional binary typed_value (STRING); }
    }
  }
}
```

Note when working with schemas and their error messages it's good to check that the files generated through iceberg are consistent with this.
They are, as the parquet cli `schema` command shows

```
Properties:
  iceberg.schema: {"type":"struct","schema-id":0,"fields":[{"id":1,"name":"id","required":true,"type":"long"},{"id":2,"name":"category","required":true,"type":"int"},{"id":3,"name":"nested","required":true,"type":"variant"}]}
Schema:
message table {
  required int64 id = 1;
  required int32 category = 2;
  required group nested (VARIANT(1)) = 3 {
    required binary metadata;
    optional binary value;
    optional group typed_value {
      required group col4 {
        optional binary value;
        optional binary typed_value (STRING);
      }
      required group idstr {
        optional binary value;
        optional binary typed_value (STRING);
      }
      required group varcategory {
        optional binary value;
        optional int32 typed_value;
      }
      required group varid {
        optional binary value;
        optional int64 typed_value;
      }
    }
  }
}
```


# Test Hardware Setup

The tests were conducted on an M1 MacBook Pro with 32 MB RAM.
This doesn't resemble production systems, and comes with the following differences which may affect results

1. Not an x86 part: no `rdtscp` opcode for benchmarking to nanosecond accuracy. 
2. Different memory access architecture, with NVMe SSD as the "disk" layer of the hierarchy.
3. Not Linux: some operations may be faster or slower.
4. No hadoop native libs. This will hurt performance validating file block checksums and compression/decompression.

A key issue to be aware of is that without the `rdtscp` opcode, Java's `System.nanotime()` method is only accurate to the 50 MHz system clock.
All benchmarks which may complete within nanoseconds must be repeated thousands of times so that the durations can be measured effectively with such a low resolution clock.

To remove all checksum verification overhead when reading files, checksum verification has been disabled (`fs.file.checksum.verify=false`).
Compression was disabled where possible.

I did set up a test run with a i7 laptop running Ubuntu as the System Under Test; the benchmarks took longer and it was harder to iterate on benchmark execution or performance sampling.

Because of the hardware differences, absolute timings in the results should be considered irrelevant -what matters more
is the relative timings between different configurations. 

# Iceberg query benchmarks

The iceberg query benchmarks generated the a test dataset as: Parquet Unshredded, Parquet Shredded and Avro.
Variants are stored in the Avro files using Iceberg's variant ser/deser code: they are not saved as
simple columnar values.


## Table setup

Tables were generated within the local filesystem comprising of five files, each with 200,000 elements,
resulting into 1M records overall.
* Compression was disabled.
* Partitioning was not enabled.
* In the absence of of [#14707 Vectorized read for variant](https://github.com/apache/iceberg/issues/14707) vectorization was disabled.    


```java
  protected Table initTable() {
    HadoopTables tables = new HadoopTables(hadoopConf());
    Map<String, String> properties = Maps.newHashMap();
    properties.put(TableProperties.FORMAT_VERSION, "3");
    properties.put(TableProperties.SPLIT_OPEN_FILE_COST, Integer.toString(128 * 1024 * 1024));
    // turn off compression to remove it as a factor.
    properties.put(TableProperties.METADATA_COMPRESSION, "none");
    properties.put(TableProperties.PARQUET_COMPRESSION, "none");
    properties.put(TableProperties.AVRO_COMPRESSION, "none");
    // variant projection pushdown not supported with the vectorized reader.
    properties.put(TableProperties.PARQUET_VECTORIZATION_ENABLED, "false");

    return tables.create(SCHEMA, PartitionSpec.unpartitioned(), properties, newTableLocation());
  }

```

## Iceberg Tests



The simplest test is a count operation
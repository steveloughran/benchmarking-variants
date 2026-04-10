# Benchmarking Variants

This repo is to show results of benchmarking variants through iceberg+spark and in parquet itself.

This is being done through two PRs

| Project | PR                                                       | Title                                        |
|---------|----------------------------------------------------------|----------------------------------------------|
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants    |

Work in progress: [writeup](benchmarking-variants.md)

Results

| Benchmark                                                              | Results                             | Source                                                                                                                                                                                  |
|------------------------------------------------------------------------|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [IcebergSourceVariantIOBenchmark](results/iceberg/index.html)          | Spark SQL Queries on Iceberg tables | [source](https://github.com/steveloughran/iceberg/blob/pr/benchmark-variant/spark/v4.1/spark/src/jmh/java/org/apache/iceberg/spark/source/parquet/IcebergSourceVariantIOBenchmark.java) |
| [VariantSerializationBenchmark](results/iceberg-variant-serialization) | Variant Serialization               | [source](https://github.com/steveloughran/iceberg/blob/pr/benchmark-variant/core/src/jmh/java/org/apache/iceberg/variants/VariantSerializationBenchmark.java)                           | 


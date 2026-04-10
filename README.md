# Benchmarking Variants

This project shows the results of benchmarking variants through iceberg+spark and in parquet itself.

The benchmarks are implemented in two PRs

| Project | PR                                                       | Title                                        |
|---------|----------------------------------------------------------|----------------------------------------------|
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants    |

## Report (Work in progress)

[Benchmarking Parquet Variants through Iceberg](benchmarking-variants.md)

## Results

| Benchmark                                                              | Results                             | Source                                                                                                                                                                                  |
|------------------------------------------------------------------------|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [IcebergSourceVariantIOBenchmark](results/iceberg/index.html)          | Spark SQL Queries on Iceberg tables | [source](https://github.com/steveloughran/iceberg/blob/pr/benchmark-variant/spark/v4.1/spark/src/jmh/java/org/apache/iceberg/spark/source/parquet/IcebergSourceVariantIOBenchmark.java) |
| [VariantSerializationBenchmark](results/iceberg-variant-serialization) | Variant Serialization               | [source](https://github.com/steveloughran/iceberg/blob/pr/benchmark-variant/core/src/jmh/java/org/apache/iceberg/variants/VariantSerializationBenchmark.java)                           | 
| [Parquet](results/parquet)                                             | Parquet Variant Benchmarks          | [source](https://github.com/steveloughran/parquet-mr/tree/pr/benchmark-variant/parquet-benchmarks/src/main/java/org/apache/parquet/variant)                                             | 

## Site Repository

This site is automatically regenerated when data is published to its github repository 
[benchmarking-variants](https://github.com/steveloughran/benchmarking-variants)

## Published Results

The content is all published via GitHub Pages at [steveloughran.github.io/benchmarking-variants/](https://steveloughran.github.io/benchmarking-variants/)
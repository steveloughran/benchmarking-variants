# Benchmarking Variants

This repo is to show results of benchmarking variants through iceberg+spark and in parquet itself.

This is being done through two PRs

| Project | PR                                                       | Title                                        |
|---------|----------------------------------------------------------|----------------------------------------------|
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants    |

Work in progress: [writeup](benchmarking-variants.md)

Results of the [IcebergSourceVariantIOBenchmark](results/iceberg/index.html)
# Benchmarking Apache Parquet Variants through Apache Iceberg

#### Steve Loughran,
#### May 2026


This project shows the results of benchmarking Parquet's Variant type through Apache Iceberg and Spark,
and in the Parquet library alone.

The benchmarks are implemented in two PRs

| Project | PR                                                       | Title                                        | Status |
|---------|----------------------------------------------------------|----------------------------------------------|--------|
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants | Open   |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants    | Merged |

I currently (May 2026) have other variant-related PRs up, for parquet: hardening reading, and for iceberg improving those benchmark numbers.
The latest results _do show improvements_.

## Key Questions

1. Are variants ready to use through Spark + Iceberg?
2. If not, what is needed?

## Answers

### Are variants ready to use through Spark + Iceberg?
Yes, but
1. It is very slow when filtering on a shedded field.
2. Spark SQL queries do not show performance issues when projecting a field within a variant, shredded or not.
3. The parquet-java library has some odd behaviour related to the schema used when reading a file.
   Only affects performance, not correctness.
4. A bit more robustness reading files is needed across all the Java implementations. Ongoing.

### If not, what is needed?

1. Predicate pushdown through the stack.
2. The causes of the "unexpected outcomes" in the benchmarking experiments to be identified and addressed.
  This could include identifying flaws in the benchmarks: review of those PRs is needed to give convidence in their conclusions.
3. A bit more profiling of the Parquet benchmarking
4. An Iceberg benchmark run with all the pending PRs merged to see what difference that makes. 

At the time of the writing of the initial document (10-04-2026) the benchmark results imply that it is faster to perform filtering on variant data stored in Avro in Iceberg + Spark queries than it is on data stored in Parquet -and that shredded variants are the worst.

The May 21 2026 findings show that with rowgroup filtering and a dataset and test quereies tuned to exclude entire rowgroups, numbers are better.
There's still a lot to be done.

## Report

[Benchmarking Parquet Variants through Iceberg](./benchmarking-variants.md)

## Results

| Benchmark Set                                                        | Results                                   | Date       |
|----------------------------------------------------------------------|-------------------------------------------|------------|
| [Iceberg Variant Benchmarks](./results/iceberg/index.html)           | Spark SQL Queries on Iceberg tables       | 2026-04-09 |
| [Iceberg + Predicate Pushdown](./results/iceberg-predicate-pushdown) | Iceberg benchmark with predicate pushdown | 2026-05-21 |
| [Parquet](./results/parquet)                                         | Parquet Variant Benchmarks                | 2026-04-10 |
| [Parquet Performance Graphs](./results/parquet-performance)          | Parquet Performance Graphs                | 2026-04-23 |
| [Experiments](./experiments)                                         | Before/after comparison of changes        | 


## Site Repository

This site is automatically regenerated when data is published to its GitHub repository 
[benchmarking-variants](https://github.com/steveloughran/benchmarking-variants).

## Published Results

The content is all published via GitHub Pages at [steveloughran.github.io/benchmarking-variants/](https://steveloughran.github.io/benchmarking-variants/)

## Links

* [Hardened JMH tabulator](https://github.com/steveloughran/jmh-tabulate) Fork of [JMH Tabulate](https://github.com/JohnTortugo/jmh-tabulate)
  whose `hardened` branch fixes the chart.js branch cryptographically and adds a `run-secure.sh` shell script which runs the report generator
  in a macos sandbox with restricted file and network access.
  Use this to compare JMH result JSON files. The unforked version just pulls down the latest version of a charting JS file from NPM, which is something nobody should ever do.
# Benchmarking Parquet Variants through Iceberg

#### Steve Loughran,
#### April 2026


This project shows the results of benchmarking variants through Iceberg+Spark and in the Parquet library alone.

The benchmarks are implemented in two PRs

| Project | PR                                                       | Title                                        |
|---------|----------------------------------------------------------|----------------------------------------------|
| Iceberg | [15629](https://github.com/apache/iceberg/pull/15629)    | Core, Spark: Add JMH benchmarks for Variants |
| Parquet | [3452](https://github.com/apache/parquet-java/pull/3452) | GH-3451. Add a JMH benchmark for variants    |


## Key Questions
1. Are variants ready to use?
2. If not, what is needed?

## Answers

1. They can be slow when filtering on a shedded field.
2. Spark SQL queries do not show performance issues when projecting a field within a variant, shredded or not.
3. The parquet-java library has some odd behaviour related to the schema used when reading a file.
4. What is needed?
   * Predicate pushdown all the way from Iceberg to the Parquet reader
   * The causes of the "unexpected outcomes" in the benchmarking experiments to be identified and addressed.
     This could include identifying flaws in the benchmarks: review of those PRs is needed to give convidence in their conclusions.
   * A bit more profiling of the Parquet benchmarking
   * An Iceberg benchmark run with all the pending PRs merged to see what difference that makes. 

At the time of the writing of the initial document (10-04-2026) the benchmark results imply that it is faster to perform filtering on variant data stored in Avro in Iceberg + Spark queries than it is on data stored in Parquet -and that shredded variants are worse.
This should not be the case.

## Report

[Benchmarking Parquet Variants through Iceberg](./benchmarking-variants.md)

## Results

| Benchmark                                                  | Results                             |
|------------------------------------------------------------|-------------------------------------|
| [Iceberg Variant Benchmarks](./results/iceberg/index.html) | Spark SQL Queries on Iceberg tables |
| [Parquet](./results/parquet)                               | Parquet Variant Benchmarks          | 

## Site Repository

This site is automatically regenerated when data is published to its github repository 
[benchmarking-variants](https://github.com/steveloughran/benchmarking-variants)

## Published Results

The content is all published via GitHub Pages at [steveloughran.github.io/benchmarking-variants/](https://steveloughran.github.io/benchmarking-variants/)
# Parquet Performance: ClickHouse vs DuckDB

Parquet, a popular columnar storage format, has become a linchpin for data-driven organizations seeking efficient and scalable solutions. But when it comes to unleashing the true potential of Parquet data, two contenders stand out: ClickHouse and DuckDB.

In this blog post, we embark on a journey to explore and compare the performance of these two robust database systems when handling Parquet data. Rather than seeking a definitive winner, our aim is to highlight the unique strengths and use cases where each database shines. In the process you will also learn how to reproduce these performance results yourself using an [open source test program](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance) and see that neither solution is universally superior, but rather tailored for particular scenarios and preferences.

## How To Run the Program?
The program to compare the Parquet performance between ClickHouse and DuckDB was run on the Hetzner cloud machine CPX51. Considering you are trying to run the program under the same conditions on the clean machine you can follow the instructions below. 

In order to reproduce the performance results using your own machine you need to get the [tests program used for this article](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance) using git.

```bash
git clone https://github.com/Altinity/clickhouse-regression.git
```

To run the program you need to have Python (version: 3.8 or greater) and pip package-management system.

To install pip run the following commands:
  ```bash
  sudo apt update
  sudo apt install python3-pip
  ```

You also need to get all python dependencies from `requirements.txt` file located in the root directory:

```bash
pip install -r requirements.txt
```

And install `unzip` if you don't already have it installed (unzip is required to get the DuckDB binary)


```bash
sudo apt install unzip
```

You also need docker to run the test program, for ubuntu refer to [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

Now that all of the dependencies are installed and the prerequisites are satisfied we can navigate to the `/parquet/performance/` and run the program.
```bash
./performance.py --clickhouse-binary-path docker://clickhouse/clickhouse-server:23.8.2.7-alpine --clickhouse-version 23.8.2.7-alpine --duckdb-binary-path https://github.com/duckdb/duckdb/releases/download/v0.8.1/duckdb_cli-linux-amd64.zip 
```
> For additional details refer to the program's README file

## Performance Results
The way to compare the parquet performance between these two tools is pretty simple, we just need a large dataset stored inside the parquet file and a set of queries we are going to run on both ClickHouse and DuckDB. The [test program](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance) uses two datasets. One is an [ontime dataset containing airline flight data](https://clickhouse.com/docs/en/getting-started/example-datasets/ontime) and the other is [hits dataset](https://github.com/ClickHouse/ClickBench#data-loading) used by [ClickBench](https://github.com/ClickHouse/ClickBench).

Let's see how fast can these database systems read from the Parquet file. We are going to be comparing the ClickHouse 23.8.2.7 and DuckDB 0.8.1 versions.

### Ontime

Let's start with the ontime dataset. If you want to see more details about the queries that are used for comparison and the results along with the raw data collected from the run, you can visit the [results](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance/results/ontime/23.8.2.7) page on github.

![bar_chart](https://github.com/Selfeer/blogpost-test/assets/26748221/23ed607d-5088-4b16-92aa-92c89e68c029)

This bar chart shows the runtime difference between ClickHouse and DuckDB on 10 different queries that read data from the Parquet file.

On the chart, we can see that when it comes to ontime dataset with 200 million rows inside the Parquet file the ClickHouse runs slower on 9 queries out of 10. The ClickHouse seems to be around 8% faster thought on query number seven.

Here is query number seven, which provides the results faster on ClickHouse. It simply calculates the percentage of flights delayed for more than 10 minutes, by year.
```sql
 SELECT Year, avg(DepDelay>10)*100 FROM file('ontime_parquet_9f6790d3_4cc4_11ee_924e_01a4aa584ed2.parquet') GROUP BY Year ORDER BY Year;
```

And here is query number five in which DuckDB is faster. It returns the percentage of delays by carrier for the year 2007.

```sql
 SELECT IATA_CODE_Reporting_Airline AS Carrier, avg(DepDelay>10)*100 AS c3 FROM file('ontime_parquet_9f6790d3_4cc4_11ee_924e_01a4aa584ed2.parquet') WHERE Year=2007 GROUP BY Carrier ORDER BY c3 DESC
```


Given these numbers, one would deduce that DuckDB is always faster than ClickHouse right? Well, it is not as straightforward as it seems, don't forget we still have to look at the results with hits dataset that has a completely different set of queries.

### Hits

As mentioned above, we used the [hits dataset](https://github.com/ClickHouse/ClickBench#data-loading) from [ClickBench](https://github.com/ClickHouse/ClickBench). For the test, we took a portion of the queries that ClickBench uses and ran them on ClickHouse and DuckDB. Again, more details can be found inside the [results](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance/results/hits/23.8.2.7) page on github.

![bar_chart (1)](https://github.com/Selfeer/blogpost-test/assets/26748221/668cea45-bbd0-40d5-85c6-dc0d054bc8d1)

We see a completely different picture now, ClickHouse is still not on par with DuckDB yet but there are specific scenarios where it is faster. The most notable query here in regards to performance is this.

```sql
 SELECT COUNT(*) FROM file(hits_parquet_8d18d12d_4cc2_11ee_924e_01a4aa584ed2.parquet);
```
ClickHouse took only `0.076` seconds to run compared to DuckDB's `0.12` seconds.

It is worth noting though that ClickHouse is supposed to be much faster with queries like number nineteen given that ClickHouse introduced performance updates for 23.8.

```sql
 SELECT UserID FROM file(hits_parquet_8d18d12d_4cc2_11ee_924e_01a4aa584ed2.parquet) WHERE UserID = 435090932899640449;
```
After analyzing these results an [issue was raised on github](https://github.com/ClickHouse/ClickHouse/issues/54372) regarding the parquet fullscan reading speed by ClickHouse.

## Conclusion
Performance results were far from uniform. Depending on the dataset and the specific query, either ClickHouse or DuckDB emerged as the frontrunner, showcasing the nuanced nature of database performance. For the ontime dataset, ClickHouse outperformed DuckDB in one query but was slower in most others. In contrast, for the hits dataset, ClickHouse showed more instances of better performance.

We would also like to encourage you to check the different results between versions of [ClickHouse](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance/results).

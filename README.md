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

### Run The Program

Now that all of the dependencies are installed and the prerequisites are satisfied we can navigate to the `/parquet/performance/` and run the program.
```bash
./performance.py --clickhouse-binary-path docker://clickhouse/clickhouse-server:23.8.2.7-alpine --clickhouse-version 23.8.2.7-alpine --duckdb-binary-path https://github.com/duckdb/duckdb/releases/download/v0.8.1/duckdb_cli-linux-amd64.zip 
```
> For additional details refer to the program's README file

## Performance Results
The way to compare the parquet performance between these two tools is pretty simple, we just need a large dataset stored inside the parquet file and a set of queries we are going to run on both ClickHouse and DuckDB. The [test program](https://github.com/Altinity/clickhouse-regression/tree/main/parquet/performance) uses two datasets. One is an [ontime dataset containing airline flight data](https://clickhouse.com/docs/en/getting-started/example-datasets/ontime) and the other is [hits dataset] used by [clickbench](https://github.com/ClickHouse/ClickBench).



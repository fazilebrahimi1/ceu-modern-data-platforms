# Exam Questions

## CEU Data Engineering 2 - Modern Data Platforms Exam

The exam will consist of 15 questions. The questions will either be taken directly from the list below or be slightly different variations. All the information required to answer the exam questions can be found in the example answers provided here. You'll have 60 minutes to answer them.

# General Architecture

**What is the difference between OLTP and OLAP, and why can't a single database design serve both purposes well?**

*Example Answer:* OLTP (Online Transaction Processing) databases are optimized for running business operations -- lots of inserts, updates, and deletes on individual records. They use few indexes so writes are fast. OLAP (Online Analytical Processing) databases are optimized for analytics -- complex SELECT queries with JOINs and aggregations. They use many indexes and columnar storage so reads are fast. These are conflicting designs: more indexes mean faster reads but slower writes, so a single design cannot optimally serve both.

*References:*
- Course slides, pages 10-15

**What is a traditional Data Warehouse, and what are its main advantages and limitations?**

*Example Answer:* A traditional Data Warehouse is a database optimized for analytical queries (OLAP). It stores well-structured data using columnar storage, many indexes, and follows a "write once, read many times" philosophy -- making it ideal for dashboarding. Its main limitations are: it can only store structured data (all data must be cleaned and structured before loading), and storage and compute are typically coupled on the same machines, so you pay for whichever is larger.

*References:*
- Course slides, pages 16-17

**What is a Data Lake, and what problems can arise if it is not managed properly?**

*Example Answer:* A Data Lake is a storage system (typically backed by HDFS or S3) that can hold data in any format -- structured files like CSV and Parquet, as well as unstructured data like images, videos, and log files. If not kept documented and organized, a Data Lake can become a "Data Swamp" where nobody knows what data exists or where it is. Data Lakes are also not ideal for real-time dashboarding because file-based storage cannot be optimized for query performance as well as a Data Warehouse.

*References:*
- Course slides, pages 23-24

**What is the difference between the ETL and ELT patterns, and why has ELT become more common in modern data platforms?**

*Example Answer:* In ETL (Extract, Transform, Load), data is extracted from sources, transformed with scripts (cleansing, joining, aggregating), and then only the clean, structured result is loaded into the Data Warehouse. In ELT (Extract, Load, Transform), raw data is loaded directly into the Data Warehouse/Lake first, then transformed in place using SQL. ELT became more common because cloud storage became cheap and scalable, and SQL-based technologies, such as dbt can do the transformation steps right in a Modern Data Warehouse (like Snowflake/Databricks) so it is no longer necessary to clean data before loading.

*References:*
- Course slides, pages 18, 38

**What is the Medallion Architecture and what are its three layers?**

*Example Answer:* The Medallion Architecture organizes data in a data platform into three layers: Bronze (raw ingestion tables containing data as-is from the source), Silver (refined/cleansed tables where data has been cleaned and standardized, _but still record-level data, not aggregated_), and Gold (aggregated, business-ready tables optimized for analytics and dashboarding). Each layer progressively improves data quality and usability.

*References:*
- Course slides, page 41
- [Delta Lake Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)

**What is a Lakehouse, and how does it relate to Data Warehouses and Data Lakes?**

*Example Answer:* A Lakehouse is a modern data platform architecture that combines the strengths of Data Warehouses and Data Lakes into a single platform. It stores all types of data (structured, semi-structured, and unstructured) like a Data Lake, but provides the data management features and query performance of a Data Warehouse (through technologies like Delta Lake that add ACID transactions and indexing to file-based storage). Databricks and Snowflake are the two main Lakehouse platforms. A downside of a Lakehouse is that it's slower than traditional Data Warehouse solutions.

*References:*
- Course slides, pages 33-36
- [Databricks Lakehouse Architecture](https://docs.databricks.com/en/lakehouse/index.html)

**What are the main components of "The Modern Data Stack"?**

*Example Answer:* The Modern Data Stack typically consists of: (1) Extract/Load tools like Airbyte or Fivetran that pull data from sources into the warehouse, (2) a cloud Data Warehouse such as Snowflake or Databricks for storage and compute, (3) a transformation tool like dbt for SQL-based transformations inside the warehouse, (4) BI tools like Tableau or Power BI for dashboards and reporting.

*References:*
- Course slides, page 42

**What is the Parquet file format, and why is it preferred over CSV for analytical workloads?**

*Example Answer:* Parquet is a binary, column-based file format (unlike CSV which is text-based and row-based). It is preferred because: (1) it has a built-in schema, so no schema inference is needed; (2) being columnar, queries that only need certain columns only read those columns; (3) it stores block-level statistics (min/max per column block) enabling predicate pushdown -- skipping entire blocks of data that don't match filter conditions; (4) being binary, it is much more compact and faster to read/write than text-based CSV.

*References:*
- Course slides, page 30
- [Apache Parquet Documentation](https://parquet.apache.org/docs/)

**What is the difference between row-based and column-based storage, and which is better suited for OLTP vs. OLAP?**

*Example Answer:* Row-based storage stores data record by record (each row together on disk), making it easy and fast to insert, update, or delete individual records -- ideal for OLTP workloads. Column-based storage stores data column by column, so analytical queries that only need specific columns can read just those columns without scanning entire records -- ideal for OLAP workloads. Data warehouses and file formats like Parquet use columnar storage for this reason.

*References:*
- Course slides, page 30
- [Apache Parquet Documentation](https://parquet.apache.org/docs/)

# dbt

**What is dbt, and what role does it play in the ELT pattern?**

*Example Answer:* dbt (data build tool) is the "T in ELT" -- it handles the Transform step by turning raw data into analysis-ready tables using SQL. It does not extract or load data; instead it sends SQL to the data warehouse (e.g., Snowflake) which performs the actual computation. dbt brings software engineering practices to analytics: version control (git), testing, documentation, and dependency management -- all using just SQL, making data pipelines accessible to data analysts.

*References:*
- [dbt Introduction](https://docs.getdbt.com/docs/introduction)
- Course slides, pages 50-54

**What are the four materialization types in dbt, and when would you use each one?**

*Example Answer:* (1) **View**: Creates a database view; use when the transformation is lightweight and data is not reused often. (2) **Table**: Creates a physical table; use when the model is queried frequently or involves expensive operations like joins. (3) **Incremental**: Appends only new records instead of rebuilding the whole table; use for large fact tables with append-only data (e.g., reviews, events). (4) **Ephemeral**: Not materialized at all -- the SQL is injected as a CTE wherever referenced; use for intermediate staging models that analysts don't need to query directly.

*References:*
- [dbt Materializations](https://docs.getdbt.com/docs/build/materializations)
- Course slides, pages 64-67

**What is the difference between the `ref()` and `source()` functions in dbt, and why are they important?**

*Example Answer:* `source()` is used to reference raw input tables defined in a `sources.yml` file (e.g., `{{ source('airbnb', 'listings') }}`). `ref()` is used to reference other dbt models and seeds (e.g., `{{ ref('dim_listings_cleansed') }}`). The main difference is that sources point to data that we don't have control over (i.e. another team manages it) while other models reference data we have full control over (as dbt materializes them).

Both are important because dbt uses them to automatically build the dependency graph (DAG) -- it knows which models depend on which, and runs them in the correct order. Without these functions, dbt cannot track lineage or determine execution order.

*References:*
- [dbt ref function](https://docs.getdbt.com/reference/dbt-jinja-functions/ref)
- [dbt source function](https://docs.getdbt.com/reference/dbt-jinja-functions/source)
- See the dbt project: `airbnb/models/src/src_listings.sql` and `airbnb/models/dim/dim_listings_cleansed.sql`

**How does an incremental model work in dbt? What happens on the first run vs. subsequent runs?**

*Example Answer:* On the first run, an incremental model behaves like a table materialization -- it creates the table and loads all data. On subsequent runs, it only processes new records (e.g., where `review_date > max(review_date)` already in the table) and appends them. This is controlled by the `is_incremental()` Jinja function, which returns false on the first run and true on subsequent runs. The `{{ this }}` keyword references the already-materialized table to compare against. You can force a full rebuild with `dbt run --full-refresh`.

*References:*
- [dbt Incremental Models](https://docs.getdbt.com/docs/build/incremental-models)
- See the dbt project: `airbnb/models/fct/fct_reviews.sql`

**What are the three types of tests in dbt, and how do they differ?**

*Example Answer:* (1) **Generic tests** are built-in, reusable tests configured in YAML schema files. The four built-in generic tests are: `unique`, `not_null`, `accepted_values`, and `relationships` (referential integrity). (2) **Singular tests** are custom SQL queries stored in the `tests/` folder -- the test passes if the query returns zero rows. (3) **Unit tests** (introduced in dbt v1.8) test model logic with mocked input data and expected output data, without needing real database data.

*References:*
- [dbt Data Tests](https://docs.getdbt.com/docs/build/data-tests)
- [dbt Unit Tests](https://docs.getdbt.com/docs/build/unit-tests)
- See the dbt project: `airbnb/models/schema.yml`, `airbnb/tests/dim_listings_minimum_nights.sql`, `airbnb/models/mart/unit_tests.yml`

**What is source freshness in dbt and why is it useful?**

*Example Answer:* Source freshness lets you check whether your raw source data is up to date before running the pipeline. In the `sources.yml` file, you configure a `loaded_at_field` (a date/timestamp column) and freshness thresholds (`warn_after` and/or `error_after`). Running `dbt source freshness` checks if the most recent record falls within the threshold. A warning continues the pipeline (exit code 0), while an error halts it (exit code 1), signaling the orchestrator to stop.

*References:*
- [dbt Source Freshness](https://docs.getdbt.com/docs/build/sources#source-data-freshness)
- See the dbt project: `airbnb/models/sources.yml`

**What are seeds in dbt and how do they differ from sources?**

*Example Answer:* Seeds are small CSV files stored in the dbt project's `seeds/` folder that get loaded into the data warehouse as tables when you run `dbt seed`. They are version-controlled alongside your project and are useful for small, static reference data (e.g., a list of full moon dates or country codes). Sources, on the other hand, are an abstraction layer over existing raw tables already in the data warehouse -- they are not loaded by dbt but defined in a `sources.yml` file for documentation, freshness checks, and dependency tracking.

*References:*
- [dbt Seeds](https://docs.getdbt.com/docs/build/seeds)
- [dbt Sources](https://docs.getdbt.com/docs/build/sources)
- See the dbt project: `airbnb/seeds/seed_full_moon_dates.csv`


**What is a Slowly Changing Dimension Type 2 (SCD2), and how does it track changes over time?**

*Example Answer:* SCD2 is a technique for preserving the full history of changes in a dimension table. Instead of overwriting a record when it changes, SCD2 keeps the old version and inserts a new one. Each version has a `valid_from` and `valid_to` timestamp: when a change is detected, the old record's `valid_to` is set to the current timestamp (closing it), and a new record is inserted with `valid_from` set to now and `valid_to` set to null (indicating it is the current version). This way, you can always query what the data looked like at any point in time.

*References:*
- [dbt Snapshots](https://docs.getdbt.com/docs/build/snapshots)
- See the dbt project: `airbnb/snapshots/snapshots.yml`

**What are dbt snapshots and what problem do they solve?**

*Example Answer:* dbt snapshots implement Slowly Changing Dimension Type 2 (SCD2) -- they track how source data changes over time. When a record's `updated_at` timestamp changes, the snapshot closes the old version (sets `dbt_valid_to`) and inserts a new version, preserving the full history. This solves the problem of source tables only containing current data with no record of past states. Without snapshots, you would lose the ability to know what data looked like at a previous point in time.

*References:*
- [dbt Snapshots](https://docs.getdbt.com/docs/build/snapshots)
- See the dbt project: `airbnb/snapshots/snapshots.yml`

**How does dbt decide the materialization of a model when it is configured at multiple levels?**

*Example Answer:* dbt uses a specificity hierarchy: model-level configuration (using `{{ config(materialized='table') }}` inside the SQL file) overrides folder-level configuration (set in `dbt_project.yml`), which overrides the global default (view). dbt always takes the most specific directive. For example, if the `dim/` folder is set to `view` in `dbt_project.yml`, but a specific model has `{{ config(materialized='table') }}`, that model will be materialized as a table.

*References:*
- [dbt Model Configuration](https://docs.getdbt.com/reference/model-configs)
- See the dbt project: `airbnb/dbt_project.yml` and `airbnb/models/dim/dim_listings_w_hosts.sql`

# Databricks and Apache Spark

**What is Apache Spark's architecture, and what are the roles of the Driver and Executors?**

*Example Answer:* Spark uses a distributed architecture with a single Driver node and one or more Executor nodes. The Driver orchestrates computation -- it hosts the user's program (Python shell, notebook), plans query execution, and distributes work. The Executors are worker nodes that perform the actual data processing: reading data from storage, computing transformations, and writing results. Data is stored externally (on S3, HDFS, etc.), not on Spark itself -- Spark is purely a compute engine.

*References:*
- [Spark Cluster Overview](https://spark.apache.org/docs/latest/cluster-overview.html)
- Course slides, pages 28-29, 72-73
- Databricks notebook: `00 - Databricks Introduction/ASP 1.1 - Databricks Platform.py`

**What is the difference between transformations and actions in Spark, and what is lazy evaluation?**

*Example Answer:* Transformations are operations that define a new DataFrame from an existing one (e.g., `select`, `filter`, `groupBy`) but do not trigger any computation -- they are lazily evaluated, meaning Spark just builds up a logical plan. Actions are operations that trigger actual computation and produce results (e.g., `show`, `count`, `collect`, `write`). Lazy evaluation allows Spark to optimize the entire chain of transformations before executing anything.

*References:*
- Databricks notebook: `01 - Spark Core/ASP 2.1 - Spark SQL.py`

**What does it mean that Spark DataFrames are lazy and immutable, and why are these properties important for optimization?**

*Example Answer:* Laziness means that transformations on a DataFrame (e.g., `select`, `filter`, `groupBy`) do not execute immediately -- they only build up a logical plan of what to do. Computation only happens when an action (e.g., `show`, `count`, `write`) is triggered. Immutability means that every transformation creates a new DataFrame; the original is never modified. These two properties are critical for optimization because they allow the Optimizer to see the entire chain of transformations before any execution begins, and rearrange, combine, or eliminate steps to produce the most efficient physical execution plan.

*References:*
- [Spark Programming Guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations)
- Databricks notebook: `01 - Spark Core/ASP 2.1 - Spark SQL.py`


**What is Delta Lake, and what advantages does it provide over plain Parquet files?**

*Example Answer:* Delta Lake is an open-source storage layer that adds reliability features on top of Parquet files. It provides: ACID transactions (atomic, consistent reads and writes), time travel (querying previous versions of data), and an audit history of all changes. It achieves this through a transaction log (`_delta_log` directory) that records every change as an ordered, atomic commit in JSON files.

*References:*
- [Delta Lake Documentation](https://docs.delta.io/latest/index.html)
- Databricks notebook: `Reference 2 - Delta Lake/ASP 6.1 - Delta Lake.py`

**What is time travel in Delta Lake, and what operation can make it unavailable?**

*Example Answer:* Time travel allows you to query previous versions of a Delta table, either by version number (`SELECT * FROM table VERSION AS OF 0`) or by timestamp. Delta Lake retains a 30-day version history by default. The VACUUM operation cleans up old data files that are no longer referenced by the current version. After vacuuming, time travel to versions older than the retention period is no longer possible because the underlying data files have been deleted.

*References:*
- [Delta Lake Time Travel](https://docs.delta.io/latest/delta-batch.html#query-an-older-snapshot-of-a-table-time-travel)
- Databricks notebook: `Reference 2 - Delta Lake/ASP 6.1 - Delta Lake.py`

**What are Jobs, Stages, and Tasks in Spark's execution model, and how do they relate to each other?**

*Example Answer:* A Job is triggered whenever an action (e.g., `count()`, `show()`) is called. Each Job is divided into Stages, which are separated by shuffle boundaries (operations that require data exchange between executors, like `groupBy`). Each Stage consists of Tasks -- one task per data partition. Tasks are the smallest units of work and run in parallel on executor cores. The number of tasks equals the number of partitions in that stage.

*References:*
- Databricks notebook: `03 - Tasks, Jobs and Stages.py`
- [Spark Job Scheduling](https://spark.apache.org/docs/latest/job-scheduling.html)

**Why is providing an explicit schema preferred over using `inferSchema` when reading CSV files in Spark?**

*Example Answer:* When using `inferSchema`, Spark must read the entire CSV file to determine column data types -- this is expensive because it requires executors to scan all data and cannot be done lazily. When you provide an explicit schema (via a DDL string or StructType), reading becomes a lazy, driver-only operation -- no data is scanned until an action is triggered. For production pipelines, explicit schemas are recommended for performance and reliability, since `inferSchema` may also guess types incorrectly.

*References:*
- Databricks notebook: `01 - Spark Core/ASP 2.2 - Reader & Writer.py`
- [Spark DataFrameReader](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrameReader.html)

**What is DBFS (Databricks File System) and what purpose does it serve?**

*Example Answer:* DBFS is a thin abstraction layer in Databricks that provides a unified, cloud-agnostic interface to the underlying object storage (S3, Azure Data Lake, GCS). It is not a separate storage system -- it simply lets you access cloud storage as if it were a local file system using a consistent API, regardless of which cloud provider is used underneath. This makes it easier to write portable code and simplifies file operations in notebooks.

*References:*
- [Databricks DBFS](https://docs.databricks.com/en/dbfs/index.html)
- Databricks notebook: `00 - Databricks Introduction/ASP 1.1 - Databricks Platform.py`

**What is Unity Catalog in Databricks and what does it manage?**

*Example Answer:* Unity Catalog is Databricks' centralized governance layer for managing data assets. It provides a hierarchical namespace: Metastore > Catalogs > Schemas > (Tables, Views, Volumes). Beyond traditional tables and views, it also manages Volumes (pointers to S3 locations for unstructured files), AI models, and other assets. Unity Catalog handles access control, enabling a single point of governance across all data in a Databricks workspace.

*References:*
- [Databricks Unity Catalog](https://docs.databricks.com/en/data-governance/unity-catalog/index.html)
- Course slides, page 74

**Why is Snappy compression preferred over ZIP/GZIP in distributed systems like Spark?**

*Example Answer:* In distributed systems, large files are split across multiple executor nodes for parallel processing. ZIP/GZIP files have a single decompression header at the beginning, so only the executor that receives the start of the file can decompress -- the others cannot process their portions independently. Snappy compression duplicates the decompression information throughout the file, allowing each executor to independently decompress its portion. This is why Snappy is the standard compression format for Parquet files in Spark.

*References:*
- [Snappy Compression](https://google.github.io/snappy/)
- [Spark Configuration - Compression Codec](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

**What happens when Spark writes a DataFrame to storage? Why does it produce a directory with multiple files instead of a single file?**

*Example Answer:* Spark writes in a distributed manner -- multiple executors write in parallel, each producing its own output file. The result is a directory containing multiple data files (one per partition/executor thread) plus a `_SUCCESS` marker file. The `_SUCCESS` file (zero-length) is written only after all executors complete, confirming the write is complete and the dataset is consistent. Without it, you cannot be certain the write finished successfully (e.g., the cluster could have crashed mid-write).

*References:*
- Databricks notebook: `01 - Spark Core/ASP 2.2 - Reader & Writer.py`

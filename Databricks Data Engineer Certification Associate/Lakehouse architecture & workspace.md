# **Databricks Lakehouse Architecture & Workspace**


## **1. Introduction to Lakehouse Architecture**

**Lakehouse = Data Lake + Data Warehouse**

* **Data Lake** stores data in its original format (raw, unstructured, semi-structured, or structured). It's optimized for **big data** and uses **ELT** (Extract, Load, Transform).
* **Data Warehouse** stores **structured data** in a **schema-on-write** fashion, typically optimized for fast querying, reporting, and analytics.

**Lakehouse Benefits:**

* Unified platform: avoids data silos between lake and warehouse.
* Simplified architecture: supports both BI and ML workloads.
* Scalable, low-cost storage + high-quality, ACID-compliant transactions.


## **2. Data Lake vs Data Warehouse vs Lakehouse**

| Feature      | Data Lake                   | Data Warehouse     | Lakehouse              |
| ------------ | --------------------------- | ------------------ | ---------------------- |
| Format       | Open(non-proprietory), raw  | Closed(proprietory), structured | Open      |
| Data types   | All (raw, semi, structured) | Structured only    | All                    |
| Schema       | Schema-on-read              | Schema-on-write    | Both                   |
| Cost         | Low                         | High               | Medium                 |
| Transactions | No ACID                     | ACID               |  Yes (via Delta Lake) |
| Use cases    | ML, data science            | BI, reporting      | Both                   |



## **3. Delta Lake – Core of the Lakehouse**

**Delta Lake** is an open-source **storage framework layer** that brings **ACID transactions**, **schema enforcement**, and **time travel** to data lakes. This is the technology that enables building a lakehouse. 

#### Key Concepts:

* Delta lake is a component that is deployed in the cluster as part of the **databricks runtime**. 
* Built on top of **Apache Parquet**.
* Stores data files + a **transaction log (delta log)**.
* Guarantees **read/write consistency** and avoids **dirty reads**.
* Supports **schema evolution**, **MERGE**, **UPSERT**, and **DELETE** operations.

####  Delta Lake Technical Details:

* When a **Delta Table** is written to, a `.parquet` file is created and the metadata is logged in a **delta log (JSON)**. This order ensures that incomplete or failed write files are never read, since the reader can’t find them in the delta log.
* Spark queries check the delta log to find **active versions** of data, then checks the active parquet file for the data.
* Supports **time travel** (query previous versions) using `VERSION AS OF` or `TIMESTAMP AS OF`.
* Since Parquet files are **immutable**, any time you make an update a new copy of the file containing the updated record is created. The old copy is then marked as inactive in the delta log.
* This log captures metadata information about the changes made to the Delta table. This includes the **operation type**, the name of the newly created data files, the **transaction timestamp**, and any other relevant information.
* When a reader queries the table, it doesn’t scan all the Parquet files directly. Instead, it looks inside the _delta_log/ folder, finds the latest numbered JSON file (or a checkpoint), and uses it to determine the current, valid set of data files. In this way, the transaction log acts as the single source of truth for the table’s state. A reader can load the checkpoint at version 10, then just apply 11 and 12 to reconstruct the current snapshot.


```pgsql
000010.checkpoint.parquet
000011.json
000012.json
```

####  Delta Lake Advantages: 

* **Open-source**: No vendor lock-in.
* **Supports ACID transactions** (atomic, consistent, isolated, durable).
* **Multiple query engines**: SQL, Spark, Python, etc.
* **Versioning**: Each change to data is logged with snapshots.
* **Error resilience**: Handles partial writes, retries, and corrupted file detection.
*  **Scalable metadata handling**:It also includes table statistics to accelerate operations.
*  **Full audit logging**: The transaction log serves as a comprehensive audit trail that captures every change occurring on the table. 

#### Delta Table Operations:

* `MERGE INTO`: Upsert pattern.
* `OPTIMIZE`: Compact small files.
* `VACUUM`: Clean old files.
* `DESCRIBE HISTORY`: View version history.

#### **Delta Table Hands-On**:

1. **Creating a Delta Table**: Creating Delta Lake tables closely resembles the conventional method of creating tables in standard SQL. It’s worth mentioning that explicitly specifying USING DELTA identifies Delta Lake as the storage layer for the table, but this clause is optional. Even in its absence, the table will still be recognized as a Delta Lake table since DELTA is the default table format in Databricks.
>```pyspark
>CREATE TABLE product_info (
> product_id INT,
> product_name STRING,
> category STRING,
> price DOUBLE,
> quantity INT
>)
>USING DELTA;
>```

2. **Creating a Standard Catalog**: Similar to creating a database in SQL Server, in Databricks you can create a standard catalog to serve that organizational purpose. Within a catalog, you can create databases, which are more like schemas in SQL Server.Types in Databricks: Default Catalog--Usually hive_metastore or main. & Unity Catalog--Introduced for fine-grained governance; supports multiple catalogs per workspace.

>```pyspark
>CREATE CATALOG my_catalog;
>```

>| Feature  | Database                  | Standard Catalog                               |
>| -------- | ------------------------- | ---------------------------------------------- |
>| Level    | Mid-level container       | Top-level container                            |
>| Contains | Tables, views, functions  | Databases                                      |
>| Scope    | Within a catalog          | Across workspace                               |
>| Purpose  | Organize tables logically | Organize multiple databases and control access |

The Standard Catalog is different from the Unity Catalog

>| Feature          | Standard Catalog | Unity Catalog                     |
>| ---------------- | ---------------- | --------------------------------- |
>| Governance       | Basic            | Fine-grained (table/column level) |
>| Scope            | Single workspace | Multi-workspace / centralized     |
>| Metadata storage | Hive metastore   | Unity Catalog metastore           |
>| Access control   | Workspace-level  | Centralized, auditable            |

3. **Output console**: In a notebook cell, if you run multiple operations, only the result of the last operation is displayed in the output console. To see the results of earlier operations, you must explicitly print them.

4. **Table Directory**: To view metadata for a Delta table, use the DESCRIBE DETAIL command. It returns key information such as numFiles, which shows the number of data files in the current table version. For external tables, it also provides the storage location. However, for tables managed by Unity Catalog, the location is not shown, since Unity Catalog deliberately hides physical storage details and treats tables as logical objects.
You can view table metadata with:

>```pyspark
>DESCRIBE DETAIL table_name;
>```

>To inspect the underlying files that make up the table, list the contents of its storage location:

>```pyspark
>%fs ls 'dbfs:/.....table path...'
>```

5. **Table History**: The transaction log maintains the history of changes made to the tables. To access the history of a table, you can use the DESCRIBE HISTORY command:

>```pyspark
>DESCRIBE HISTORY product_info
>```

>To view the files tht make up the transaction log by using the `%fs ls`:

>```pyspark
>%fs ls 'dbfs:/.....table path.../_delta_log'
>```

6. **Time Travel**: Time travel is a feature in Delta Lake that allows you to retrieve previous versions of data in Delta Lake tables. This versioning provides an audit trail of all the changes that have happened on the table.
> *Querying Older Versions by timestamp*:
> ```pyspark
> SELECT * FROM <table_name> TIMESTAMP AS OF <timestamp>
> ```
> *Querying Older Versions by version number*:

> ```pyspark
> SELECT * FROM <table_name> VERSION AS OF <version>
> SELECT * FROM product_info@v2 --where 2 is the version number
> ```

> *Rolling Back to Previous Versions*:

> ```pyspark
> RESTORE TABLE <table_name> TO TIMESTAMP AS OF <timestamp>
> RESTORE TABLE <table_name> TO VERSION AS OF <version>
> ```

7. **Optimizing Delta Lake Tables**: Say you have a table that has accumulated many small files due to frequent write operations. By running the OPTIMIZE command, these small files can be compacted into one or more larger files. 

>```pyspark
>OPTIMIZE <table_name>
>```
>A notable extension of the OPTIMIZE command is the ability to leverage Z-Order indexing. Z-Order indexing involves the reorganization and co-location of column information within the same set of files. To perform Z-Order indexing, you simply add the ZORDER BY keyword to the OPTIMIZE command.
>```pyspark
>OPTIMIZE <table_name>
>ZORDER BY <column_names>
>```


## **4. Medallion Architecture in Lakehouse**

####  Bronze Layer:
* Raw, ingested data.
* Typically unfiltered or semi-structured.
* Stored as-is for audit/replay purposes.

####  Silver Layer:
* Cleaned and conformed data.
* Joins, filters, schema enforcement applied.
* Used for analytics across departments.

####  Gold Layer:
* Business-level aggregates and metrics.
* Used for reporting, BI tools, dashboards.

>  Use **Delta Live Tables** to manage transformation pipelines across Bronze → Silver → Gold automatically.


##  **5. Databricks Architecture (2024)**

* **Control Plane (Managed by Databricks)**:

  * Web UI, job orchestration, Unity Catalog, notebook interface.
  * Hosted by Databricks in its own account.
  * Manages workspace logic, APIs, cluster control.

* **Compute Plane (Classic or Serverless)**:

  * Where jobs execute and data is stored.
  * **Classic Compute Plane**: Uses your cloud resources (Azure/AWS/GCP).
  * **Serverless Compute Plane** (2024): Hosted by Databricks, charges to Databricks account.

    * Serverless = zero infrastructure management.
    * Pay-per-use model.

####  Charging:

* Anything on the **Control Plane** or **Serverless Compute** is billed to your **Databricks account**.
* Classic compute plane jobs are billed to **your cloud account**.


##  **6. Databricks Unity Catalog**

* Unified governance layer for **data access control**, **audit**, and **discovery**.
* Manages metadata across **workspaces and clouds**.
* Tracks lineage of data, supports RBAC at **table**, **column**, and **view** levels.
* **Replaces Hive Metastore** in newer deployments.


##  **7. Databricks Workspace Essentials**

A Databricks workspace includes:

| Component         | Description                                    |
| ----------------- | ---------------------------------------------- |
| **Clusters**      | Scalable compute resources for processing jobs |
| **Jobs**          | Automation of notebooks/scripts                |
| **Notebooks**     | Interactive development (SQL, Python, Scala)   |
| **Repos**         | Git-integrated code versioning                 |
| **Dashboards**    | Visual analytics                               |
| **Unity Catalog** | Centralized metadata and security management   |






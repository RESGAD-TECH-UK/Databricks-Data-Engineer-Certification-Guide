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

* When a **Delta Table** is written to, a `.parquet` file is created and the metadata is logged in a **delta log (JSON)**.
* Spark queries check the delta log to find **active versions** of data, then checks the active parquet file for the data.
* Supports **time travel** (query previous versions) using `VERSION AS OF` or `TIMESTAMP AS OF`.


##  **4. Delta Lake Advantages**

* **Open-source**: No vendor lock-in.
* **Supports ACID transactions** (atomic, consistent, isolated, durable).
* **Multiple query engines**: SQL, Spark, Python, etc.
* **Versioning**: Each change to data is logged with snapshots.
* **Error resilience**: Handles partial writes, retries, and corrupted file detection.


## **5. Medallion Architecture in Lakehouse**

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


##  **6. Databricks Architecture (2024)**

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


##  **7. Databricks Unity Catalog**

* Unified governance layer for **data access control**, **audit**, and **discovery**.
* Manages metadata across **workspaces and clouds**.
* Tracks lineage of data, supports RBAC at **table**, **column**, and **view** levels.
* **Replaces Hive Metastore** in newer deployments.


## **8. Delta Tables in Practice**

* Tables are defined with a **metastore catalog** (`catalog.schema.table`).
* Databricks supports:
  * **Managed tables**: Data stored inside the metastore.
  * **External tables**: Data resides in external storage (e.g., ADLS).

#### Delta Table Operations:

* `MERGE INTO`: Upsert pattern.
* `OPTIMIZE`: Compact small files.
* `VACUUM`: Clean old files.
* `DESCRIBE HISTORY`: View version history.



##  **9. Databricks Workspace Essentials**

A Databricks workspace includes:

| Component         | Description                                    |
| ----------------- | ---------------------------------------------- |
| **Clusters**      | Scalable compute resources for processing jobs |
| **Jobs**          | Automation of notebooks/scripts                |
| **Notebooks**     | Interactive development (SQL, Python, Scala)   |
| **Repos**         | Git-integrated code versioning                 |
| **Dashboards**    | Visual analytics                               |
| **Unity Catalog** | Centralized metadata and security management   |






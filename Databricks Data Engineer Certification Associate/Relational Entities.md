# Relational Entities in Databricks

### Databases in Databricks
In Databricks, a database essentially corresponds to a schema in a data catalog. This means that when you create a database, you’re essentially defining a logical structure where tables, views, and functions can be organized. This collection of database objects is called a schema. You have the flexibility to create a database using either the 'CREATE DATABASE' or 'CREATE SCHEMA' syntax, as they are functionally equivalent.

#### Creating a Database / Schema:
With **Unity Catalog**, you can create **schemas** in any supported storage location by specifying a custom path using the **LOCATION** keyword in the `CREATE SCHEMA` statement. In this case, the **schema definition** is stored in **Unity Catalog**, while the underlying data files are placed in the designated custom location. Any **tables** created within these schemas will have their data stored in the corresponding folder under the specified path.

#### Working in a Custom-Location Schema (Unity Catalog)
In Unity Catalog, you can create a **schema** in a custom storage location outside of the default path.  
Use the `CREATE SCHEMA` statement with the `LOCATION` keyword, pointing to the desired storage path (for example, `dbfs:/Shared/schemas`):  
```sql
CREATE SCHEMA my_catalog.custom_schema
LOCATION 'dbfs:/Shared/schemas/custom_schema.db';
```
You can verify the schema creation in the Catalog Explorer, where it appears under the specified catalog. To confirm the storage path, use:
```sql
DESCRIBE SCHEMA EXTENDED my_catalog.custom_schema;
```
### Hive_metastore
Before now, every **Databricks workspace** included a local data catalog called **hive_metastore**, which all clusters could access to persist **object metadata**. The **Hive metastore** served as a repository for essential metadata, storing information about **databases**, **tables**, and **partitions**, including details like **table definitions**, **data formats**, and **storage locations**. However, the Hive metastore is now considered a **legacy feature**, and Databricks strongly encourages migrating to **Unity Catalog**, which offers richer capabilities such as **centralized governance**, **auditing**, **lineage tracking**, and **fine-grained access control**. Even after enabling Unity Catalog, the **hive_metastore** remains visible as a **top-level catalog**, allowing access to older tables or metadata that have not yet been migrated. Migrations are supported through tools like **Federation** (to reference Hive Metastore tables) and **Upgrade Wizards**, **SYNC**, and **CLONE** commands (to move or replicate tables into Unity Catalog). Ultimately, once migration is complete, access to the legacy Hive metastore can be **disabled** via **workspace settings** or **Spark configurations**.

### Tables in Databricks
In Databricks, there are two types of tables: managed tables and external tables. Understanding the distinction between them is essential for effectively managing your data.
> **1. Managed Table**
> - Created within its own database directory
> - Metastore (Hive/Unity Catalog) owns both metadata and data.
> - Data stored in a location controlled by the metastore.
> - Dropping the table deletes both metadata and underlying data files. Simplifies lifecycle management but requires caution since data is permanently removed.
  ```sql
  CREATE TABLE table_name
  -- Since we’re not specifying the LOCATION keyword, this table is considered managed
  ```
  >
  >**2. External Table**
> - Created outside the database directory with a specified path
> - Metastore manages only metadata; data files remain in the external path (e.g., S3, Azure Storage).
> - Dropping the table removes only metadata, leaving underlying data intact.
> - Useful for external storage scenarios or when separating data file management from metadata.
> - Can be created in default, custom, or user-specified database locations using USE DATABASE / USE SCHEMA.
```sql
CREATE TABLE table_name
LOCATION <path>
```

You can use the `DESCRIBE EXTENDED` command is used to show detailed metadata about catalog objects. E.g., Views, Columns within a table, Tables (managed or external)
```sql
DESCRIBE EXTENDED my_table;
-- Shows schema (columns, data types): Table properties, Storage information (format, location, provider)
```
>NOTES: `DESCRIBE EXTENDED` does not work on databases/schemas directly — for that you’d use:
>```sql
>DESCRIBE DATABASE EXTENDED my_schema;
>-- or
>DESCRIBE SCHEMA EXTENDED my_schema;
>```
Remember, you can manually remove the table directory and its content by running the `dbutils.fs.rm` function in Python. E.g.,
```pyspark
dbutils.fs.rm("/databricks-datasets/")
```

### Setting Up Delta Tables
One of the key features of Delta Lake tables is their flexibility in creation. While traditional methods like the regular `CREATE TABLE` statements are available, Databricks also supports CTAS, or `CREATE TABLE AS SELECT`, statements. CTAS statements allow the creation and population of tables at the same time based on the results of a SELECT query. This means that with CTAS statements, you can create a new table from existing data sources:
```sql
CREATE TABLE table_2
AS SELECT * FROM table_1
```
Here are the different options of delta table definitions: [Table Creation syntax Options](https://github.com/Stephen-Data-Engineer-Public/Databricks-Data-Engineer-Certification-Guide/blob/main/Databricks%20Data%20Engineer%20Certification%20Associate/Code%20Snippet/Create%20Table%20Syntax.md)

**Comparison of CREATE TABLE and CTAS**

| Feature              | CREATE TABLE                                                                 | CTAS (CREATE TABLE AS SELECT)                                      |
|----------------------|-------------------------------------------------------------------------------|--------------------------------------------------------------------|
| **Syntax Example**   | `CREATE TABLE table_2 (col1 INT, col2 STRING, col3 DOUBLE)`                  | `CREATE TABLE table_2 AS SELECT col1, col2, col3 FROM table_1`     |
| **Schema Declaration** | Requires **manual schema declaration** (explicitly define columns & types). | **Automatically infers schema** from the `SELECT` query.           |
| **Populating Data**  | Creates an **empty table**; must load data separately (e.g., `INSERT INTO`). | Creates the table **with data populated** from the `SELECT` query. |
| **Table Constraints** | Can add constraints after creation using `ALTER TABLE`. Supported: **NOT NULL**, **CHECK**. | Same as CREATE TABLE; constraints can be added post-creation.      |

**Example constraint usage**:  
```sql
ALTER TABLE my_table
ADD CONSTRAINT valid_date CHECK (date >= '2024-01-01' AND date <= '2024-12-31');
```

### Cloning Delta Lake Tables
In Databricks, you can efficiently duplicate or back up your **Delta Lake tables** using **deep clone** or **shallow clone** operations.

**1. Deep Cloning**

Deep cloning copies **both data and metadata** from a source table to a target table. Example:
```sql
CREATE TABLE my_catalog.table_clone
DEEP CLONE my_catalog.source_table;
```
- Deep cloning ensures a full independent copy, but may take longer for large tables because all data is physically copied.
- Supports incremental synchronization, allowing updates from source to target if needed.

**2. Shallow Cloning**

Shallow cloning copies only the Delta transaction logs, not the underlying data:
```sql
CREATE TABLE my_catalog.table_clone
SHALLOW CLONE my_catalog.source_table;
```
- This is faster and ideal for development or testing environments, where you want to experiment without duplicating large datasets.
- Changes to the cloned table are tracked separately, preserving the integrity of the original table.

>NOTE: **Data Integrity**
>- Both cloning methods ensure that any modifications in the cloned table do not affect the original source table.
>- Cloning works seamlessly with Unity Catalog schemas and supports all Delta Lake table types, including managed and external tables.

### Exploring Views in Databricks

**Definition:**  
- A **view** is a **virtual table**; it stores a **SQL query** without storing physical data.  
- Querying a view executes its underlying query against the source table(s) each time.

#### View Types

| View Type               | Syntax                     | Accessibility             | Lifetime                                   | Notes |
|-------------------------|---------------------------|--------------------------|-------------------------------------------|-------|
| **Stored View**          | `CREATE VIEW view_name AS <query>` | Across sessions & clusters | Dropped only via `DROP VIEW`             | Metadata stored in metastore; data queried from source table. |
| **Temporary View**       | `CREATE TEMP VIEW view_name AS <query>` | Session-scoped           | Dropped automatically when session ends  | Useful for temporary analyses; not persisted in any database. |
| **Global Temporary View**| `CREATE GLOBAL TEMP VIEW view_name AS <query>` | Cluster-scoped           | Dropped when cluster restarts/terminates | Accessible across notebooks on same cluster; stored in `global_temp` database. |

#### Key Concepts

- **Stored views** are persisted in the **metastore** (Hive or Unity Catalog) and accessible across sessions and clusters.  
- **Temporary views** exist **only for the current session**; useful for ad-hoc computations.  
- **Global temporary views** exist for the **lifetime of the cluster** and are accessible across multiple notebooks.  
- **Query execution:** Views execute the underlying SQL query on source tables each time they are queried.  
- **Dropping views:**
```sql
  DROP VIEW view_name;
  DROP VIEW temp_view_name;
  DROP VIEW global_temp.global_temp_view_name;
```
>NOTE: Databricks doesn't support materialized views, but you can achieve similar functionality using Delta tables, caching, or Delta Live Tables.
>```sql
>CACHE TABLE view_name;
>```

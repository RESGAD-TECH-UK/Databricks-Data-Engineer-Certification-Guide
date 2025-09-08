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
Here are the different options of delta table definitions: [Table Creation syntax Options](Databricks Data Engineer Certification Associate/Code Snippet/Create Table Syntax.md)

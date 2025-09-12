# Transforming Data Using Spark

## Querying Data Files
To initiate a file query, we use the `SELECT * FROM` syntax, followed by the file format and the path to the file.  It’s important to note that the filepath is specified between backticks `` (`</path/>`) ``, and not single quotes `('</path/>')`. A filepath in this context can refer to a single file, or it can incorporate a wildcard character to simultaneously read multiple files. Alternatively, the path can point to an entire directory, assuming that all files within that directory adhere to the same format and schema.

```sql
SELECT * FROM format.`path/file.format`
```

### Querying JSON Format
To read a single JSON file, the SELECT statement is used with the syntax `SELECT * FROM json`.
```sql
SELECT * FROM json.`${dataset.mybigdata}/mydata-json/export_001.json`
```
To query multiple files simultaneously, you can use the wildcard character `(*)` in the path. For instance, you can easily query all JSON files starting with the name `export_`:
```sql
SELECT * FROM json.`${dataset.mybigdata}/mydata-json/export_*.json`
```

### Querying text Format
When dealing with a variety of text-based files, including formats such as JSON, CSV, TSV, and TXT, Databricks provides the flexibility to handle them using the text format:
```sql
SELECT * FROM text.`path/file.txt`
-- Or
SELECT * FROM text.`path/file.csv...` --It can read JSON, CSV, or TSV
```
This format allows you to extract the data as raw strings, which provide significant advantages, especially in scenarios where input data might be corrupted or contain anomalies. By extracting data as raw strings, you can leverage custom parsing logic to navigate and extract relevant values from the text-based files. E.g., 
>Suppose you have a “students” CSV file like this:
```pgsql
id,name,age
1,John,15
2,Mary,16
3,"Tom, Jr.",17   <-- contains a comma inside quotes
4,Alice,N/A       <-- missing value / bad type
5,Bob             <-- missing column
```
If you try to load this directly as `csv.``…``` it may throw errors or silently drop rows. Instead, you can first read it as raw text (like your example), then parse manually.
SQL-based parsing
```sql
-- Read as raw text
SELECT value
FROM text.`${dataset.school}/students-csv`
```
```bash
Sample output:
1,John,15
2,Mary,16
3,"Tom, Jr.",17
4,Alice,N/A
5,Bob
```
Now, you can parse with `split()` and handle anomalies:
```sql
SELECT
  split(value, ',')[0] AS id,
  split(value, ',')[1] AS name,
  TRY_CAST(split(value, ',')[2] AS INT) AS age
FROM text.`${dataset.school}/students-csv`
WHERE value NOT LIKE 'id,%'  -- skip header
```
`split(value, ',')` breaks the row into columns.
`TRY_CAST(... AS INT)` avoids errors when age is not numeric (N/A becomes NULL).
You can filter, clean, or reformat further as needed.

### Querying Using binaryFile Format
There are scenarios where the binary representation of file content is essential, such as when working with images or unstructured data. In such cases, the binaryFile format is suited for this task:
```sql
SELECT * FROM binaryFile.`path/sample_image.png`
```
We can use the binaryFile format to extract the raw bytes and some metadata information of the student files:
```sql
SELECT * FROM binaryFile.`${dataset.school}/students-csv`
```
by using the binaryFile format, you can access both the content and metadata of files, offering a detailed view of your dataset.

### Querying Non-Self-Describing Formats
When dealing with non-self-describing formats like comma-separated-value (CSV), the SELECT statement may not be as informative. Unlike JSON and Parquet, CSV files lack a predefined schema, making the format less suitable for direct querying. In such cases, additional steps, such as defining a schema, may be necessary for effective data extraction and analysis. E.g., let say you are querying a csv that isn't delimited by a comma instead it was a semicolon. Using the statement below will return a single column and the header row will be extracted as a table row. 
```sql
SELECT * FROM csv.`${dataset.school}/courses-csv`
```
<img width="400" height="193" alt="image" src="https://github.com/user-attachments/assets/d9cf5d99-4a17-44ca-b2fe-64ae3b3651db" />

This issue highlights a challenge with querying files without a well-defined schema, particularly in formats like CSV.

### Registering Tables from Files with CTAS
Using CTAS (CREATE TABLE AS SELECT) statements allows you to register tables from files, particularly when dealing with well-defined schema sources like Parquet files. This process is crucial for loading data into a lakehouse, allowing you to take full advantage of the Databricks platform’s capabilities:
```sql
CREATE TABLE table_name
AS SELECT * FROM <file_format>.`/path/to/file`
```
>**NOTE:** CTAS statements automatically infer schema information from the query results, making them a suitable choice for external data ingestion from sources with well-defined schemas, such as Parquet files.

### Registering Tables on Foreign Data Sources
In scenarios where additional file options are necessary like choosing delimiter type, etc., an alternative solution is to use the regular `CREATE TABLE` statement, but with the USING keyword. Unlike **CTAS** statements, this approach is particularly useful when dealing with formats that need specific configurations. The **USING** keyword provides increased flexibility by allowing you to specify the type of foreign data source, such as CSV format, as well as any additional files options, such as delimiter and header presence:
```sql
CREATE TABLE table_name
   (col_name1 col_type1, ...)
USING data_source/Format
OPTIONS (key1 = val1, key2 = val2, ...)
LOCATION path
```
>However, it’s crucial to note that this method creates an external table, serving as a reference to the files without physically moving the data during table creation to Delta Lake. Unlike CTAS statements, which automatically infer schema information, creating a table via the USING keyword requires you to provide the schema explicitly. 

Another scenario where the `CREATE TABLE` statement with the USING keyword proves useful is when creating a table using a JDBC connection, which allows referencing data in an external SQL database. This approach enables you to establish a connection to an external database by defining necessary options such as the connection string, username, password, and specific database table containing the data.
Here is an example of creating an external table using a JDBC connection:
```sql
CREATE TABLE jdbc_external_table
USING JDBC
OPTIONS (
 url = 'jdbc:mysql://your_database_server:port',
 dbtable = 'your_database.table_name',
 user = 'your_username',
 password = 'your_password'
);
```

#### Impact of not having a Delta table
The absence of a Delta table introduces certain limitations and impacts. Unlike Delta Lake tables, which guarantee querying the most recent version of source data, tables registered against other data sources, like CSV, may represent outdated cached data. Spark automatically caches the underlying data in local storage for better performance in subsequent queries. However, the external sources does not natively signal Spark to refresh this cached data. Consequently, the new data remains invisible until the cache is manually refreshed using the REFRESH TABLE command:
```sql
REFRESH TABLE courses_csv
```
#### Hybrid approach
To address this limitation and leverage the advantages of Delta Lake, a workaround involves creating a temporary view that refers to the foreign data source. Then, you can execute a CTAS statement on this temporary view to extract the data from the external source and load it into a Delta table. 
```sql
CREATE TEMP VIEW courses_tmp_vw
  (course_id STRING, title STRING, instructor STRING, category STRING, 
  price DOUBLE)
USING CSV
OPTIONS (
 path = "${dataset.school}/courses-csv/export_*.csv",
 header = "true",
 delimiter = ";"
);

CREATE TABLE courses AS
SELECT * FROM courses_tmp_vw;
```
### Replacing Data
You can completely replace the content of a Delta Lake table either by overwriting the existing table or by other traditional methods like dropping and re-creating it. However, overwriting Delta tables offers several advantages over the approach of merely dropping and re-creating tables.
| Aspect                             | Drop and Recreate Table                                                                 | Overwrite Table                                                                 |
| ---------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Processing time**                | Time-consuming as it involves recursively listing directories and deleting large files. | Fast process since the updated data is just a new table version.                |
| **Leveraging Delta’s time travel** | Deletes the old versions of the table, making its historical data unavailable.          | Preserves the old table versions, allowing easy retrieval of historical data.   |
| **Concurrency**                    | Concurrent queries are unable to access the table while the operation is ongoing.       | Concurrent queries can continue reading the table seamlessly during the update. |
| **ACID guarantees**                | If the operation fails, the table cannot be reverted to its original state.             | If the operation fails, the table reverts to its previous state.                |

In summary, the process of overwriting tables provides efficiency, reliability, and seamless integration with Delta’s features such as time travel and ACID transactions. In Databricks, there are two methods to completely replace the content of Delta Lake tables: `CREATE OR REPLACE TABLE` statements & `INSERT OVERWRITE` statements. 

**1. CREATE OR REPLACE TABLE statement**

The first method to achieve a complete table overwrite in Delta Lake is by using the CREATE OR REPLACE TABLE statement, also known as the `CRAS (CREATE OR REPLACE AS SELECT)` statement. This statement fully replaces the content of a table each time it executes:
```sql
CREATE OR REPLACE TABLE enrollments AS
SELECT * FROM parquet.`${dataset.school}/enrollments`
```

**2. INSERT OVERWRITE**
The second method for overwriting data in Delta Tables involves using the `INSERT OVERWRITE` statement:
```sql
INSERT OVERWRITE enrollments
SELECT * FROM parquet.`${dataset.school}/enrollments`
```
### Appending Data
One of the simplest methods to append records to Delta Lake tables is through the use of the INSERT INTO statement. This statement allows you to easily add new data to existing tables from the result of a SQL query:
```sql
INSERT INTO enrollments
SELECT * FROM parquet.`${dataset.school}/enrollments-new`
```
### Merging Data
The MERGE INTO statement enables you to perform upsert operations—meaning you can insert new data and update existing records—and even delete records, all within a single statement. we aim to update student data with modified email addresses and add new students into the table. To accomplish this, we first create a temporary view containing the updated student data. This view will serve as the source from which we’ll merge changes into our students table:
```sql
CREATE OR REPLACE TEMP VIEW students_updates AS
SELECT * FROM json.`${dataset.school}/students-json-new`;
```
The following merge operation is executed to merge the changes from the student_updates temporary view into the target students table, using the student ID as the key for matching records:
```sql
MERGE INTO students c
USING students_updates u
ON c. student_id = u. student_id
WHEN MATCHED AND c.email IS NULL AND u.email IS NOT NULL THEN
 UPDATE SET email = u.email, updated = u.updated
WHEN NOT MATCHED THEN INSERT *
```
>**NOTE:** You can use the insert-only merge to prevent duplicate entries. E.g,
>```sql
>MERGE INTO courses c
>USING courses_updates u
>ON c.course_id = u.course_id AND c.title = u.title
>WHEN NOT MATCHED THEN
> INSERT *
>```

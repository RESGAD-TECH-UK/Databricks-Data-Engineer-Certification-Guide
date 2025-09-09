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

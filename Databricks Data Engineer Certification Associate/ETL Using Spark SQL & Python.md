# Transforming Data Using Spark

## Querying Data Files
To initiate a file query, we use the `SELECT * FROM` syntax, followed by the file format and the path to the file.  It’s important to note that the filepath is specified between backticks `` (`</path/>`) ``, and not single quotes `('</path/>')`. A filepath in this context can refer to a single file, or it can incorporate a wildcard character to simultaneously read multiple files. Alternatively, the path can point to an entire directory, assuming that all files within that directory adhere to the same format and schema.

```sql
SELECT * FROM json.`path/file.json`
```

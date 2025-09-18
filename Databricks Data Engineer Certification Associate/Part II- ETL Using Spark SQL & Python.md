# Performing Advanced Data Transformations
## Dealing with Nested Json Data

Try running this: 
```python
import json

# Nested JSON data (same structure, but we'll save line-delimited)
records = [
    {
        "id": 1,
        "name": "Alice",
        "contact": {
            "email": "alice@example.com",
            "phone": "123-456-7890"
        },
        "orders": [
            {"order_id": "A100", "amount": 250.75, "items": {"Laptop":"Mouse"}},
            {"order_id": "A101", "amount": 99.99, "items": {"Keyboard":"buttons"}
        ]
    },
    {
        "id": 2,
        "name": "Bob",
        "contact": {
            "email": "bob@example.com",
            "phone": "555-987-6543"
        },
        "orders": [
            {"order_id": "B200", "amount": 49.50, "items": ["Book", "Pen"]}
        ]
    }
]

# Save in NDJSON format (one JSON object per line)
output_path = "/Volume_path/nested_example.json"

with open(output_path, "w") as f:
    for rec in records:
        f.write(json.dumps(rec) + "\n")

```
JSON is a self-describing format, we can go ahead to use CTAS to create a delta table for our nested_example json file.

```sql
CREATE OR REPLACE TABLE nested_example AS
SELECT * FROM json.`/Volume_path/nested_example.json`;
SELECT * FROM nested_example
```
<img width="1177" height="161" alt="image" src="https://github.com/user-attachments/assets/0cada9b6-f48f-4f55-860a-dcbd399a516e" />

Now run:
```sql
DESCRIBE nested_example;
```
You should get the result:
<img width="845" height="222" alt="image" src="https://github.com/user-attachments/assets/28ffa56a-e7ae-4270-acb0-83a80c0aeff1" />

You can see that `contact` is a struct, which is a spark data type with nested attributes. 

### Parsing JSON into Struct Type
**Why would I want to parse JSON into a struct type?**

- JSON is just text (strings, numbers, booleans, arrays, objects).
- A struct type in a language like Spark, Go, Rust, C#, or Swift has explicit fields with types.
- Parsing JSON into a struct forces the data to match your expected schema. Example: if your struct expects int Age, but JSON has "Age": "abc", parsing fails immediately instead of letting bad data creep in.
- With raw JSON, you’d constantly write string lookups like json["user"]["name"].
- With a struct, you can just use user.Name, which is simpler and less error-prone.
- Struct fields often get code completion in your IDE, making development faster.

Spark SQL goes further by providing functionality to parse JSON objects into struct types—a native Spark type with nested attributes. The from_json function is employed for this task, but it requires knowledge of the schema of the JSON object in advance:
```sql
SELECT from_json(nested_column, <schema>) FROM Table_name;
```
E.g., let's say our contact column was a nested string json data type & we needed to parse it into a struct type,
```sql
CREATE OR REPLACE TEMP VIEW parsed_table_name AS
 SELECT id, from_json(contact, schema_of_json('{"first_name":"Sarah",
 "last_name":"Lundi", "address":{"street":"8 Greenbank Road",
 "city":"Ottawa", "country":"Canada"}}')) AS profile_struct
 FROM Table_name;

SELECT * FROM parsed_table_name
SELECT contact.address.country FROM parsed_table_name -- This will return the country column for the table.
```

#### Flattening Struct Types
Once a JSON string is converted to a struct type, Spark SQL introduces a powerful feature—the ability to use the star (*) operation to flatten fields and create separate columns:
```sql
CREATE OR REPLACE TEMP VIEW nested_final AS
 SELECT id, name, contact.*
 FROM parsed_table_name;

SELECT * FROM nested_final;
```

### Leveraging the explode Function
Spark SQL provides dedicated functions for efficiently handling arrays, like the explode function. This function allows us to transform an array into individual rows, each representing an element from the array:
```sql
SELECT id, name, explode(orders) AS course
FROM nested_example
```

### Aggregating Unique Values
The `collect_set` function, this function is an aggregation function that returns an array of unique values for a given field. It can even deal with fields within arrays. In this example, the orders column is formed as an array of arrays:

```sql
SELECT id, name
 collect_set(orders.items) AS item_set
FROM nested_example
GROUP BY id, name
```

### Join Operations in Spark SQL
The syntax used for joining data in Spark SQL follows the conventions of standard SQL. We specify the type of join we want (inner, outer, left, right, etc.), the tables we are joining, and the conditions for the join.
```sql
CREATE OR REPLACE VIEW dataset_enriched AS
SELECT *
FROM (
  SELECT *, explode(column_1) AS column_name
  FROM dataset_table ) e
INNER JOIN dataset2_table c
ON e.column_1.id = c.id;

SELECT * FROM dataset_enriched
```
Other operations are; `UNION ALL, INTERSECT, and MINUS`, which is like SQL's EXCEPT operation.

### Pivoting 
Spark SQL supports creating pivot tables for transforming data perspectives using the PIVOT clause. This provides a means to generate aggregated values based on specific column values. This transformation results in a pivot table, wherein the aggregated values become multiple columns. 
```sql
SELECT * FROM (
SELECT id, column_1.id AS course_id, column_1.subtotal AS subtotal
  FROM dataset_enriched
 )
 PIVOT (
 sum(subtotal) FOR id IN (
   'C01', 'C02', 'C03', 'C04', 'C05', 'C06',
   'C07', 'C08', 'C09', 'C10', 'C11', 'C12')
)
```
>How PIVOT works in Spark SQL
>- PIVOT takes one measure column (here: subtotal) and aggregates it (here: SUM).
>- It takes one pivot column (here: id) and spreads its values across new columns.
>- Everything else that’s selected but not aggregated or pivoted becomes the row identifier(s).

| id | C01   | C02   | C03  | ... | C12  |
| ----------- | ----- | ----- | ---- | --- | ---- |
| 101         | 250.0 | 100.0 | NULL | ... | 75.0 |
| 102         | NULL  | 200.0 | 50.0 | ... | NULL |

### Higher-Order Functions
Higher-order functions in Databricks provide a powerful toolset for working with complex data types, such as arrays. The two essential functions are: `FILTER and TRANSFORM`.
**FILTER**

The `FILTER` function is a fundamental higher-order function that enables the extraction of specific elements from an array based on a given lambda function.
```SQL
SELECT
id,
column_1,
FILTER (column_1,
       namexxx -> namexxx.discount_percent >= 60) AS highly_discounted_column_1
FROM dataset_table
```
However, we will get result with several empty arrays where the discounted percentage is not upto 60. we could use the `WHERE` clause to filter them out.
```sql
SELECT id, highly_discounted_column_1
FROM (
 SELECT
id,
column_1,
FILTER (column_1,
       namexxx -> namexxx.discount_percent >= 60) AS highly_discounted_column_1   
FROM dataset_table
WHERE size(highly_discounted_column_1) > 0;
```
> NOTE: namexxx is not an actual column, just like in python for i in range(10), i = namexxx in this context.

**TRANSFORM**

The `TRANSFORM` function is another essential higher-order function that facilitates the application of a transformation to each item in an array, extracting the transformed values.
```SQL
SELECT
id,
column_1,
TRANSFORM (
  column_1,
  namexxx -> ROUND(namexxx.subtotal * 1.2, 2) ) AS column_1_after_tax
FROM dataset_table;
```
### Developing SQL UDFs
SQL user-defined functions (UDFs) are a powerful way to encapsulate custom logic with a SQL-like syntax, making it reusable across different SQL queries. Unlike external UDFs written in Scala, Java, Python, or R, which appear as black boxes to the Spark Optimizer, SQL UDFs leverage Spark SQL directly. This typically provides better performance when applying custom logic to large datasets.
**Creating UDFs**

To create a SQL UDF, you need to specify a function name, optional parameters, the return type, and the custom logic.
```sql
CREATE OR REPLACE FUNCTION get_letter_grade(column_name DOUBLE)
RETURNS STRING
RETURN CASE
        WHEN column_name >= 3.5 THEN "A"
        WHEN column_name >= 2.75 AND gpa < 3.5 THEN "B"
        WHEN column_name >= 2 AND gpa < 2.75 THEN "C"
        ELSE "F"
     END
```
**Applying UDFs**

Once the UDF is created, you can use it in any SQL query like a native function.
```sql
SELECT id, column_1, get_letter_grade(column_1) AS grade
FROM Table_name
```
**Understanding UDFs**

SQL UDFs are permanent objects stored in the database, allowing them to be used across different Spark sessions and notebooks.
```sql
DESCRIBE FUNCTION get_letter_grade
DESCRIBE FUNCTION EXTENDED get_letter_grade -- to view more details about the function.
```
**Dropping UDFs**

Finally, you can remove UDFs when they are no longer needed by using the DROP FUNCTION command:
```sql
DROP FUNCTION get_letter_grade;
```

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

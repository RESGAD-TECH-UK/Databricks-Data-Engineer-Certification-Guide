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
            {"order_id": "A100", "amount": 250.75, "items": ["Laptop", "Mouse"]},
            {"order_id": "A101", "amount": 99.99, "items": ["Keyboard"]}
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
You will get the result:
<img width="1177" height="288" alt="image" src="https://github.com/user-attachments/assets/e5cad800-324d-4703-915a-893c5fe3b092" />

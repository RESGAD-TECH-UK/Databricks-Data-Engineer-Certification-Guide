```sql
{ { [CREATE OR] REPLACE TABLE | CREATE [EXTERNAL] TABLE [ IF NOT EXISTS ] }
  table_name
  [ table_specification ]
  [ USING data_source ]
  [ table_clauses ]
  [ AS query ] }

table_specification
  ( { column_identifier column_type [ column_properties ] } [, ...]
    [ , table_constraint ] [...] )

column_properties
  { NOT NULL |
    COLLATE collation_name |
    GENERATED ALWAYS AS ( expr ) |
    GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [ ( [ START WITH start | INCREMENT BY step ] [ ...] ) ] |
    DEFAULT default_expression |
    COMMENT column_comment |
    column_constraint |
    MASK clause } [ ... ]

table_clauses
  { OPTIONS clause |
    PARTITIONED BY clause |
    CLUSTER BY clause |
    clustered_by_clause |
    LOCATION path [ WITH ( CREDENTIAL credential_name ) ] |
    COMMENT table_comment |
    TBLPROPERTIES clause |
    DEFAULT COLLATION default_collation_name |
    WITH { ROW FILTER clause } } [...]

clustered_by_clause
  { CLUSTERED BY ( cluster_column [, ...] )
    [ SORTED BY ( { sort_column [ ASC | DESC ] } [, ...] ) ]
    INTO num_buckets BUCKETS }
```

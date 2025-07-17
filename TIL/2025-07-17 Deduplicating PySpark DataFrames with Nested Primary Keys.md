
### Introduction

Imagine that you have some semi-structured nested data, which includes stuct ids
```python
    {"id": {"a": 1, "b": "X"}, "value": 100, "update_ts": "2024-01-01"},
    {"id": {"a": 1, "b": "X"}, "value": 200, "update_ts": "2024-01-02"},
    {"id": {"a": 2, "b": "Y"}, "value": 300, "update_ts": "2024-01-01"}
```

And you want to deduplicate on both `id.a` and `id.b` 

### The issue
```python
df.dropDuplicates(["id.a", "id.b"])
```
Won't work and will raise an error : `Cannot resolve column name "id.a" among (id, update_ts, value).`

### How to fix it

```python
from pyspark.sql import DataFrame
from pyspark.sql.functions import col, struct, max

def clean_dataframe(df: DataFrame, primary_keys: list[str], update_col: str | None, ordering_fields: list[str]) -> DataFrame:
    if not primary_keys:
        return df

    if update_col:
        return deduplicate_with_sorting(df, primary_keys, update_col)
    elif ordering_fields:
        return deduplicate_with_sorting(df, primary_keys, ordering_fields[0])
    else:
        return df.dropDuplicates(primary_keys)

def deduplicate_with_sorting(df: DataFrame, key_fields: list[str], sorting_field: str) -> DataFrame:
    cols = df.columns  # Save initial column order
    other_fields = [c for c in cols if c != sorting_field and c not in key_fields]

    # Unnest nested PKs
    unnested_key_fields = [k.replace('.', '_') for k in key_fields]
    df_unnested = df
    for orig, new in zip(key_fields, unnested_key_fields):
        df_unnested = df_unnested.withColumn(new, col(orig))

    # Prevent PK/sortfield duplicates
    other_col_names = [sorting_field] + [f for f in other_fields if f not in unnested_key_fields]

    # Build struct of all other fields
    df_struct = df_unnested.select(
        *[col(k) for k in unnested_key_fields],
        struct(*[col(f) for f in other_col_names]).alias("otherCols")
    )

    # Group and get max
    res1 = df_struct.groupBy(*[col(k) for k in unnested_key_fields]) \
                    .agg(max("otherCols").alias("latest"))
    res2 = res1.select(*[col(k) for k in unnested_key_fields], col("latest.*"))

    # Restore original columns
    return res2.select(*[col(c) for c in cols])
```

This snippet works by:

1. **"Unnesting"** the nested primary keys into temporary, flat columns.
2. **Bundling** all other relevant columns (including the sorting column like `update_ts`) into a single **struct**.
3. **Grouping** by the temporary, flat primary key columns and using `max()` on the struct. Because the sorting field is the first element in the struct, `max()` correctly identifies the record with the latest value for that field among duplicates.
4. **Expanding** the resulting struct back into individual columns and restoring the original DataFrame schema.

_NB : This also works on standard primary keys at the root_

_NB2 : This won't work on array of struct, since this would break the "1 primary key <=> 1 row" rule (Atomicity 1NF)_


##### Example of usage

```python

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, struct, max

# Exemple de DataFrame avec des clés imbriquées
data = [
    {"id": {"a": 1, "b": "X"}, "value": 100, "update_ts": "2024-01-01"},
    {"id": {"a": 1, "b": "X"}, "value": 200, "update_ts": "2024-01-02"},
    {"id": {"a": 2, "b": "Y"}, "value": 300, "update_ts": "2024-01-01"}
]

df = spark.read.json(spark.sparkContext.parallelize(data))

# Appelle à la fonction avec clé imbriquée "id.a" et "id.b"
dedup_df = clean_dataframe(
    df,
    primary_keys=["id.a", "id.b"],
    update_col="update_ts",
    ordering_fields=[]
)

dedup_df.show(truncate=False)
```

This will correctly shows 
```
+------+----------+-----+ 
|id |update_ts |value| 
+------+----------+-----+ 
|{1, X}|2024-01-02|200 | 
|{2, Y}|2024-01-01|300 | 
+------+----------+-----+
```


### Use Cases / When to Apply This (Business Context)

This pattern is especially useful in data engineering workflows:

- **CDC (Change Data Capture) Processing:** Efficiently selecting the **latest record state** from incremental updates.
    
- **Semi-structured Data Ingestion:** Deduplicating logs or events where IDs are nested, ensuring the most recent action is kept.
    
- **Data Lake Deduplication:** Maintaining **data quality** by removing duplicates, particularly with complex or evolving schemas.
    
- **Master Data Management (MDM):** Identifying the **"golden record"** for entities (like customers or products) based on recency.
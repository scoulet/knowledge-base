
### Introduction

Imagine that you have some nested data, which includes stuct ids
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

This snippet does X Y Z 

_NB : This won't work on array of struct, since this would break the "1 primary key <=> 1 row" rule_


##### Example of usage

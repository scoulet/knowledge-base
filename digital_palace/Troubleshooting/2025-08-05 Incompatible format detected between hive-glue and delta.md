

---

### **The Problem**

I encountered the error `Incompatible format detected. A transaction log for Delta was found... but you are trying to read from ... using format('hive')`. The issue occurred when a Glue table, created to point to a Delta Lake table, was being queried by tools that assume a Hive format. This often happens on platforms like Databricks or other Spark environments when the metadata in Glue isn't correctly configured for Delta Lake. The "Sample Data" tab on my data catalog interface failed with this error because the underlying read operation was using the wrong format.

---

### **Solution A - Modify the `TableInput` (Boto3)**

The fix is to explicitly define the table's format by setting `'spark.sql.sources.provider': 'delta'` in the `Parameters` of the `TableInput` object. This tells AWS Glue to treat the table as a Delta table.

Python

```
import boto3

# ... (other code)

table_input = {
    'Name': table_name,
    'TableType': 'EXTERNAL_TABLE',
    'Parameters': {
        'EXTERNAL': 'TRUE',
        'spark.sql.sources.provider': 'delta'  # The key to the solution
    },
    'StorageDescriptor': {
        # ... (storage descriptor details)
    }
}

glue_client.create_table(
    DatabaseName=database_name,
    TableInput=table_input
)
```

---

### **Solution B - Use PySpark SQL (Databricks/Spark)**

A simpler and more common approach on platforms like Databricks is to use a `CREATE TABLE` SQL statement with the `USING DELTA` syntax. This automatically configures the table's metadata in the Glue Catalog correctly, making it readable by all Delta-compatible tools.

Python

```
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("GlueDeltaTableFix").getOrCreate()
delta_table_path = "s3://..."
database_name = "hive_metastore"
table_name = "mwaa_resource_usage"

# The `USING DELTA` syntax handles the configuration automatically
spark.sql(f"""
  CREATE TABLE IF NOT EXISTS {database_name}.{table_name}
  USING DELTA
  LOCATION '{delta_table_path}'
""")
```
# TIL – Writing to Unity Catalog from EMR (not just Databricks!)

## 🎯 Problem / Context  
We built a custom Spark-based ELT pipeline running on **AWS EMR**, which initially relied on **AWS Glue** for metadata cataloguing.  
Now, we want to migrate to **Databricks Unity Catalog** to benefit from its advanced governance features — but **without migrating the ELT itself** from EMR to Databricks.

**Why Unity over Glue?**  
- Centralized and consistent access control (RBAC)  
- Table-level, column-level, and even row-level security  
- Built-in data lineage and discoverability  
- Unified across multiple workspaces and environments  

Unity is deeply integrated with Databricks by default, which makes external usage (e.g., from EMR or even Docker) more complex — but not impossible with the right configuration.

## 🐛 Common Pitfall  
- Assuming Unity just replaces Glue without extra setup  
- Forgetting required Spark config: Unity won’t work without proper `.uri`, `.token`, `.auth.type`  
- Defaulting back to Hive metastore silently  
- Table not created in Unity, but in EMR-local storage  
- `saveAsTable` fails silently or creates unmanaged table outside Unity  

## 💡 Solution / Snippet  

### 1. Set up your SparkSession with Unity Catalog options

```python
from pyspark.sql import SparkSession
import os

# These could be fetched from AWS Secrets Manager, Vault, or env vars
unity_catalog = os.environ["UNITY_CATALOG"]
unity_schema = os.environ["UNITY_SCHEMA"]
unity_token = os.environ["UNITY_TOKEN"]
unity_host = os.environ["UNITY_HOST"]  # ex: "https://<workspace>.cloud.databricks.com"

spark = SparkSession.builder \
    .appName("UnityFromEMR") \
    .config("spark.sql.defaultCatalog", unity_catalog) \
    .config(f"spark.sql.catalog.{unity_catalog}", "io.unitycatalog.spark.UCSingleCatalog") \
    .config(f"spark.sql.catalog.{unity_catalog}.uri", f"{unity_host}/api/2.1/unity-catalog") \
    .config(f"spark.sql.catalog.{unity_catalog}.token", unity_token) \
    .config(f"spark.sql.catalog.{unity_catalog}.auth.type", "token") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .config("spark.hadoop.fs.s3.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem") \
    .getOrCreate()
```

### 2. Clean overrides to disable Hive/Glue if needed

EMR tends to default to Hive or Glue — override this **only if you're writing to Unity**:

```python
spark.conf.set("spark.hadoop.hive.metastore.uris", "")
spark.conf.set("spark.sql.catalogImplementation", "in-memory")
spark.conf.set("hive.metastore.client.factory.class", "org.apache.hadoop.hive.metastore.HiveMetaStoreClientFactory")
```

✅ **Best practice**: wrap these overrides in a condition like `if use_unity_catalog:` to avoid messing up non-Unity pipelines.

---

### 3. Save your DataFrame to Unity Catalog table

```python
df.write \
    .format("delta") \
    .option("optimize", "true") \
    .option("maxFileSize", "128MB") \
    .mode("overwrite") \
    .partitionBy(*partition_list) \
    .option("path", path_delta) \
    .saveAsTable(f"{unity_catalog}.{unity_schema}.{table_name}")
```

⚠️ This will create an **external table** — ensure the corresponding **external location** is configured and associated with your Unity metastore, with correct S3 paths and permissions.

## 🔍 Why It Works  
Unity Catalog requires explicit declaration of:
- the catalog name (`spark.sql.defaultCatalog`)  
- its URI (`.uri`)  
- authentication (`.token` and `.auth.type`)  

Once this is properly injected into the SparkSession, **Unity becomes usable from any Spark engine** — including EMR or Docker.

It enables:
- Consistent governance  
- Interoperability with other Databricks-managed resources  
- Centralized security without forcing compute to run on Databricks  

## 🛠️ When to Use It  
- Migrating metadata from Glue to Unity  
- Running Spark workloads on EMR while leveraging Unity governance  
- Decoupling compute from catalog and security  
- Testing Unity jobs locally or inside Docker containers  
- Using Unity across multiple workspaces and environments  
- Avoiding Hive/Glue limitations (like eventual consistency, lack of column ACLs)  

## 🧠 Key Ideas to Remember  
- Unity requires explicit config: `.uri`, `.token`, `.auth.type`  
- External Spark jobs (EMR, Docker, local) can use Unity — if properly configured  
- You’re no longer locked to the Databricks runtime for catalog and access control  
- Unity Catalog is compute-agnostic: works with EMR, Docker, or even local Spark  

## 📝 Sources (optional)  
- [Unity Catalog Docs](https://docs.databricks.com/data-governance/unity-catalog/index.html)  
- [External Locations Setup](https://docs.databricks.com/data-governance/unity-catalog/manage-external-locations.html)  
- [Delta Lake Table Docs](https://docs.delta.io/latest/delta-batch.html)  

## 📝 What to add to make this an article  
- Visual diagram: EMR + S3 + Unity as external metastore  
- Error-handling examples (`Access Denied`, `table not found`, etc.)  
- Token management patterns (Vault, Secrets Manager)  
- Side-by-side cost or governance comparison: Glue vs Unity  
- How to integrate Unity-aware Spark jobs into Airflow pipelines  

---

**Tags**:  
#spark #unitycatalog #emr #delta #databricks #governance #migration #pyspark #aws #TIL #glue #elt

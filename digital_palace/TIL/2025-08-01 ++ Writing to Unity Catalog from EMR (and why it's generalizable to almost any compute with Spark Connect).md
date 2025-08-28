
## 🎯 Problem / Context  
We built a custom Spark-based ELT pipeline running on **AWS EMR**, which initially relied on **AWS Glue** as the metadata catalog.  
We now want to migrate to **Databricks Unity Catalog** for centralized governance, lineage, and fine-grained access control — **without moving our compute away from EMR**.

But here’s the twist: although we illustrate the setup using EMR, **this pattern is generalizable to almost any environment — even outside Spark clusters — using Spark Connect**.

Unity is typically tied to Databricks runtimes, but if you configure Spark properly (or delegate execution to a Unity-aware server), it becomes fully decoupled from the underlying compute engine.

## 🐛 Common Pitfall  
- Believing Unity only works inside Databricks  
- Using Spark outside Databricks without proper `.uri`, `.token`, `.auth.type` setup  
- Falling back silently to Hive or Glue if Unity is not declared  
- Forgetting that writing to Unity is a server-side operation — so it depends on *where Spark runs*, not *where the code is written*

## 💡 Solution / Snippet  

### 1. SparkSession with Unity Catalog options (e.g. on EMR)

```python
from pyspark.sql import SparkSession
import os

unity_catalog = os.getenv("UNITY_CATALOG")
unity_schema  = os.getenv("UNITY_SCHEMA")
unity_token   = os.getenv("UNITY_TOKEN")
unity_host    = os.getenv("UNITY_HOST")

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

### 2. Disable default metastore if needed

```python
spark.conf.set("spark.hadoop.hive.metastore.uris", "")
spark.conf.set("spark.sql.catalogImplementation", "in-memory")
spark.conf.set("hive.metastore.client.factory.class", "org.apache.hadoop.hive.metastore.HiveMetaStoreClientFactory")
```

---

### 3. Save to Unity table

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

⚠️ This will create an **external table** — ensure the external location is configured in Unity and mapped to your S3 path.

---

## 🔁 Generalizing with Spark Connect

While the example uses EMR directly, **this setup can be generalized to any environment that:**

1. **Runs Spark** (or connects to a Spark server)
2. **Has access to Unity Catalog API** via `.uri`, `.token`, and `.auth.type`
3. **Uses Spark 3.4+ and Delta 3.2+** (to support Unity-compatible Spark Connect)
4. **Delegates execution to a Unity-enabled Spark backend**

Thanks to **Spark Connect**, you can now:
- Write to Unity from **a FastAPI app**, **a CLI script**, or **a CI/CD runner**
- Run the logic on **remote Spark clusters** (e.g. Databricks, K8s, or EMR)
- Separate business logic from execution environment completely

## 🔍 Why It Works  
Unity doesn’t require Databricks compute — just a properly configured Spark session.  
If you send your job to a Spark engine that knows how to speak Unity (via the right configs), you're good to go — whether from EMR, Docker, or even your local laptop via spark-connect ([[2025-07-26 Spark 4.0 - spark-connect - Remote spark without local installation]])

This decoupling:
- Makes Unity available to non-Databricks users  
- Unlocks governance from within any enterprise Spark job  
- Enables hybrid architectures (Databricks + EMR + local jobs)

## 🛠️ When to Use It  
- Migrating metadata from Glue to Unity while keeping EMR compute  
- Developing on local Spark or in Docker, then deploying to Unity-aware Spark server  
- Integrating Spark in microservices (e.g. FastAPI) via Spark Connect  
- Decoupling governance from infrastructure for better portability  

## 🧠 Key Ideas to Remember  
- Unity is a server-side metastore — your SparkSession must be configured, not necessarily your code  
- Spark Connect enables lightweight clients to interact with Unity-aware servers  
- This approach works across EMR, Kubernetes, Docker, and even local Spark setups  

## 📝 Sources (optional)  
- [Unity Catalog Docs](https://docs.databricks.com/data-governance/unity-catalog/index.html)  
- [Use Unity Catalog from EMR (AWS blog)](https://aws.amazon.com/blogs/big-data/use-databricks-unity-catalog-open-apis-for-spark-workloads-on-amazon-emr/)  
- [Spark Connect Overview](https://spark.apache.org/docs/latest/spark-connect.html)

## 📝 What to add to make this an article  
- Diagram: app → Spark Connect → Unity  
- Show example with FastAPI + Spark Connect client  
- Compare latency and security between Glue and Unity  
- Tips for token/secret management in CI/CD  
- Integration into data platform architecture (Airflow, Docker, etc.)

---

**Tags**:  
#spark #unitycatalog #emr #delta #databricks #sparkconnect #governance #pyspark #aws #TIL #metadata #elt #catalog #portable-data

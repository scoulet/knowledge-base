
# TIL – How Databricks’ Spark Differs from Open-Source Spark

## 🎯 Problem / Context  
Apache Spark was originally created (UC Berkeley, 2009) to overcome the inefficiencies of Hadoop MapReduce for iterative, large-scale data processing. While powerful, open-source Spark wasn’t designed as a high-performance, native query engine — a requirement for Databricks’ **Lakehouse** vision, which merges the scalability of data lakes with the management capabilities of data warehouses.  

To compete with solutions like Snowflake, BigQuery, and Redshift, Databricks had to **dramatically improve Spark’s query performance** without breaking compatibility for existing users.

## 🐛 Common Pitfall  
Open-source Spark’s JVM-based engine and row-oriented in-memory representation create performance bottlenecks for:
- Highly variable, messy, or wide datasets typical in Lakehouse workloads.
- Large heap memory (>64 GB) due to garbage collection limits.
- Columnar file formats (e.g., Parquet) that require costly row-pivoting.
- Need for low-level optimizations (SIMD, memory control) not easily achievable in the JVM.

## 💡 Solution / Snippet  

Databricks built:
1. **Databricks Runtime (DBR)** – A fork of Spark with enhanced reliability & performance.
2. **Photon Engine** – A **native C++ vectorized execution engine** integrated into DBR as a new set of physical operators.

Key technical shifts:
- **From JVM to C++** → full control over memory, SIMD, and CPU-level optimizations.
- **Vectorized execution** → batch-based processing, adaptable at runtime to data properties.
- **Columnar in-memory format** → avoids column-to-row conversion, aligns with Parquet/ORC.
- **Operator fallback** → unsupported queries revert seamlessly to SparkSQL.

"""
# Example: switching to Databricks runtime
spark = SparkSession.builder \
    .appName("LakehouseApp") \
    .getOrCreate()

# Queries automatically leverage Photon where possible
df = spark.read.format("delta").load("/mnt/datalake/sales")
df.groupBy("region").sum("revenue").show()
"""

## 🔍 Why It Works  
- **Columnar + vectorized execution** maximizes CPU cache locality and SIMD parallelism.
- **Native code** removes JVM overhead and GC pauses.
- **Runtime adaptivity** lets Photon choose specialized code paths for different data characteristics.
- Maintains **full Spark API compatibility**, enabling transparent performance gains.

## 🛠️ When to Use It  
- High-volume, mixed-format Lakehouse workloads (structured + semi-structured data).
- Workloads with many small files or wide schemas.
- Query-heavy pipelines where latency is critical.
- Migration to Databricks with minimal code changes.

### ✅ Before  
- JVM limits on heap size and GC overhead.  
- Row-oriented memory causing columnar file inefficiency.  
- Limited low-level performance tuning.  

### ✅ With This Solution  
- Native execution with SIMD & controlled memory usage.  
- Columnar pipeline from disk to memory to output.  
- Transparent fallback to JVM Spark for unsupported features.  

## 🧠 Key Ideas to Remember  
- Databricks Spark ≠ Open-Source Spark — the former embeds Photon and DBR for OLAP-level performance.  
- Vectorized columnar execution is critical for modern analytical workloads.  
- Native C++ offers deterministic, explainable performance gains over JVM.  

## 📝 Sources  
- https://blog.dataengineerthings.org/how-is-databricks-spark-different-from-open-source-spark-c8017ce01256
- _Databricks,_ [_Photon: A Fast Query Engine for Lakehouse Systems_](https://people.eecs.berkeley.edu/~matei/papers/2022/sigmod_photon.pdf) _(2022)._
- _Michael Armbrust, Reynold S. Xin, Cheng Lian, Yin Huai, Davies Liu, Joseph K. Bradley, Xiangrui Meng, Tomer Kaftan, Michael J. Franklin, Ali Ghodsi, Matei Zaharia_ [_Spark SQL: Relational Data Processing in Spark_](https://people.csail.mit.edu/matei/papers/2015/sigmod_spark_sql.pdf) _(2015)_
- Liz Elfman, [A brief history of Databricks](https://www.bigeye.com/blog/a-brief-history-of-databricks) (2023)
- Databricks, *Photon: A Fast Query Engine for Lakehouse Systems* (2022)  
- Michael Armbrust et al., *Spark SQL: Relational Data Processing in Spark* (2015)  

## 📝 What to add to make this an article  
- Benchmarks comparing Apache Spark vs Databricks Spark (with Photon) on real workloads.  
- Deep dive into vectorization internals and SIMD optimizations.  
- Practical migration steps from open-source Spark to Databricks Spark.  
- Cost/performance trade-off analysis versus competitors (Snowflake, BigQuery).  

---

**Tags**: #spark #databricks #photon #bigdata #lakehouse #performance #vectorization #cpp #columnar

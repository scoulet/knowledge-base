> Stream Rows to Avro, Convert to Parquet Later

## 🎯 Problem / Context  
Dumping very large relational tables into a data lake via Spark often fails or drags forever if you do a naïve:

```
df = spark.read.format("jdbc").options(...).load()
df.write.format("parquet").save("s3://bucket/out/")
```

Why? Columnar writes (Parquet) tend to buffer sizeable batches, JDBC reads can overload the source with too many concurrent fetches, and you end up with unstable jobs, snapshot inconsistency, or “mystery” memory spikes.  
I ran into this when exporting multi-billion-row tables from a legacy database: the job would start, run for hours, then crash because Spark was buffering too much before writing.

## 🐛 Common Pitfall  
- Treating JDBC like a file source (one giant pull with default settings).  
- Writing directly to Parquet on the landing zone.  
- Forgetting snapshot consistency on `[from, to)` windows (rows change mid-dump).  
- Zero control on throughput: the DB gets hammered by too many connections or oversized fetches.  

## 💡 Solution / Snippet  
**Land first in Avro (row-based) with controlled chunking, then convert to Parquet in a separate job.**

### Example with Spark
```python
# JDBC → Avro (landing)
(
  spark.read.format("jdbc")
    .option("url", "jdbc:postgresql://host:5432/db")
    .option("dbtable", "public.big_table")
    .option("user", "user")
    .option("password", "pw")
    # Control parallelism & chunking
    .option("partitionColumn", "id")
    .option("lowerBound", 0)
    .option("upperBound", 1_000_000_000)
    .option("numPartitions", 8)      # modest to protect the DB
    .option("fetchsize", 10_000)     # smooth network/memory usage
    .load()
    .write.mode("overwrite")
    .format("avro")
    .save("s3://my-bucket/bronze/big_table/")
)

# Later: Avro (bronze) → Parquet (silver)
df = spark.read.format("avro").load("s3://my-bucket/bronze/big_table/")
(
  df.write.mode("overwrite")
    .partitionBy("updated_at")
    .format("parquet")
    .save("s3://my-bucket/silver/big_table/")
)
```

### Example outside Spark (streaming ResultSet → Avro → S3)  
Use a server-side cursor to iterate row by row, rotate files every N rows, write Avro incrementally, and upload via multipart to S3.  
This avoids buffering huge datasets in memory and protects the database from overload.

## 🔍 Why It Works  
- **Avro (row-based)** writes incrementally: more resilient for huge dumps than columnar Parquet.  
- **Chunking** (N rows per file + controlled fetch size + modest numPartitions) = balanced throughput for both DB and compute.  
- **Staged processing** (Avro landing → Parquet later) decouples failures and makes retries simpler.  
- **Snapshot consistency** (transaction isolation or “as-of timestamp”) ensures coherent data in the `[from, to)` window.  

## 🛠️ When to Use It  
- Full exports or backfills from Oracle/PostgreSQL/SQL Server.  
- Sensitive databases where concurrency must be limited.  
- Bronze-first pipelines where **reliability > speed** on landing.  

### ✅ Before  
- Single-thread JDBC reads or overly aggressive partitioning hammering the DB.  
- Direct Parquet writes buffering too much in memory.  
- Dumps inconsistent because rows changed mid-extraction.  

### ✅ With This Solution  
- Stable landing in Avro, rotating files every N rows with sync markers.  
- Controlled throughput (fetch size, partitions) and consistent snapshots.  
- Reliable Parquet conversion later with proper partitioning and compaction.  

## 🧠 Key Ideas to Remember  
- **Format matters**: Avro for write-heavy landing, Parquet for read-heavy analytics.  
- **Control the firehose**: adjust `numPartitions`, `fetchsize`, and consider sharding by ID/time.  
- **Use logical types**: Avro `decimal` for numeric precision, `timestamp-millis` and `date` for temporal values.  
- **Think in stages**: Bronze (raw, reliable) → Silver (optimized).  
- **Measure everything**: row counters, nulls, type conversions, throughput, latency, lineage events.  

## 📝 Sources (optional)  
- [Breaking up large JDBC writes with Spark (StackOverflow)](https://stackoverflow.com/questions/79201441/breaking-up-a-large-jdbc-write-with-spark)  
- [Throttling Spark JDBC writes (Oudeis blog)](https://www.oudeis.co/blog/2020/spark-jdbc-throttling-writes/)  
- [Parquet vs Avro: when to use which (Airbyte)](https://airbyte.com/data-engineering-resources/parquet-vs-avro)  

## 📝 What to add to make this an article
- Benchmarks: JDBC→Avro vs JDBC→Parquet (memory, stability, runtime).  
- Spark UI screenshots showing effect of `numPartitions` and `fetchsize`.  
- Schema typing examples for Avro logical types (`decimal`, `timestamp-millis`).  
- A production checklist: multipart uploads, retries/backoff, snapshot isolation, monitoring with lineage events.  

---

**Tags**: #spark #jdbc #bronze #avro #parquet #etl #performance #dataengineering

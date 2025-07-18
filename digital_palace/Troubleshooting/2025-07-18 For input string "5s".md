



---
### Bug Resolved: `java.lang.NumberFormatException: For input string: "5s"` with S3A/Spark configurations

---

### The Problem

While working with PySpark (or Scala/Java Spark) and interacting with S3a, I encountered a `java.lang.NumberFormatException: For input string: "5s"` during DataFrame operations (e.g., writing to S3). This error was perplexing as the input string "5s" seems related to a time duration, but Spark/Java's `NumberFormatException` typically expects a pure numeric string.

The root cause was that several `fs.s3a.connection` and `fs.s3a.threads` configurations, when set to their **default string values** (like "5s" for "5 seconds", "24h" for "24 hours", "1d" for "1 day"), were being parsed as numbers internally, leading to this format exception. [cite_start]Spark expects these specific configurations to be provided in **milliseconds** (numeric value), not human-readable string durations. 



### Solution B - 

[cite_start]The fix involved explicitly setting these S3a-related configurations in **milliseconds**.  [cite_start]By overriding their default string values with numeric millisecond values, the `NumberFormatException` was resolved. 

```python
# Assuming 'spark' is your SparkSession and 'conf' is spark.sparkContext._jconf
# Or you can set these directly when building SparkSession:
# SparkSession.builder.config("spark.hadoop.fs.s3a.connection.establish.timeout", "5000")

conf.set("fs.s3a.connection.establish.timeout", "5000")    # 5 seconds
conf.set("fs.s3a.connection.timeout", "200000")          # 200 seconds
conf.set("fs.s3a.threads.keepalivetime", "60000")        # 60 seconds
conf.set("fs.s3a.multipart.purge.age", "86400000")       # 24 hours (24*60*60*1000 ms)



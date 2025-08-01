
## 🎯 Problem / Context  

When copying many files between two S3 buckets (e.g., during data migration, replication, or staging workflows), using standard tools like `s3-dist-cp` can be slow or overly rigid. In a Spark job, we needed more granular control, better logging, and flexible parallelization for a selective list of files.  
The goal was to optimize throughput and take advantage of AWS S3’s multipart capabilities — without launching a full EMR cluster just for `s3-dist-cp`.

## 🐛 Common Pitfall  

- **s3-dist-cp** requires an EMR cluster and copies entire directories/prefixes.
- Lacks control over concurrency granularity.
- Can be overkill for selective or small-to-medium batch transfers.
- Doesn't expose low-level retries, logging, or file-specific error handling.

## 💡 Solution / Snippet  

In **Scala**, the solution uses:
- `TransferManager` from AWS SDK
- A manually tuned `ForkJoinPool`
- Parallel iteration over a list of files

Here is a **Python version** using `boto3` and `concurrent.futures.ThreadPoolExecutor`:

```
# Install the required package
pip install boto3
```

```python
import boto3
import concurrent.futures

s3 = boto3.client('s3')
s3_resource = boto3.resource('s3')

def copy_file(source_bucket, source_key, dest_bucket, dest_key):
    copy_source = {'Bucket': source_bucket, 'Key': source_key}
    print(f"Copying: s3://{source_bucket}/{source_key} → s3://{dest_bucket}/{dest_key}")
    s3_resource.Object(dest_bucket, dest_key).copy(copy_source)

def parallel_copy(source_bucket, dest_bucket, source_prefix, dest_prefix, files, max_workers=60):
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [
            executor.submit(
                copy_file,
                source_bucket,
                f"{source_prefix}/{file}",
                dest_bucket,
                f"{dest_prefix}/{file}"
            )
            for file in files
        ]
        concurrent.futures.wait(futures)
```

## 🔍 Why It Works  
> What is the underlying principle or architecture?

- **Thread-based parallelism** allows high I/O throughput on S3 operations.
- **TransferManager** (in Scala/Java) and **boto3** (in Python) both support multipart transfers under the hood.
- Fine control of concurrency via `ForkJoinPool` or `ThreadPoolExecutor` enables scaling based on expected S3 limits and available CPU.

## 🛠️ When to Use It  
- You need to copy a selective list of files (not full prefixes).
- You're operating in a non-EMR context (e.g., Spark job on Kubernetes or local runner).
- You want programmatic control over retries, logging, and error handling.
- You want to avoid the overhead of launching EMR or using Hadoop tooling.

## 🐳 What About Docker / Compose?

### ✅ Before  
- Using EMR/s3-dist-cp requires infrastructure setup.
- Copying files in sequence can be painfully slow.
- Hard to integrate into lightweight, container-based jobs.

### ✅ With This Solution  
- Embeds easily in Python or Scala jobs.
- Can run inside any container with AWS credentials.
- Zero external dependencies aside from the SDK.

## 🧠 Key Ideas to Remember  
- S3 is naturally highly concurrent — parallelism is key for performance.  
- You don’t need Hadoop to copy data efficiently on S3.  
- Fine-grained parallel copy gives flexibility **and** speed.

## 📝 Sources (optional)  
- [AWS boto3 documentation – S3 copy](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.copy)
- [TransferManager Java Docs](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/transfer/TransferManager.html)

## 📝 What to add to make this an article

- 📊 **Benchmark and Metrics Are a Must**: Since this is a performance-focused topic, you need hard numbers. Compare your custom solution with `s3-dist-cp` on a concrete dataset (e.g., 1,000 files totaling 100 GB). Show metrics like total transfer time, cost (EMR cluster vs standalone compute), and memory footprint.
- 📈 **Visualization Ideas**: Include a plot showing throughput (MB/s) on the Y-axis versus number of workers on the X-axis. Overlay performance of your custom copy function versus `s3-dist-cp` to illustrate scaling efficiency.
- ❓ **When Is `s3-dist-cp` Still Relevant?**: Discuss the situations where `s3-dist-cp` remains a solid choice — for example, large-scale full-prefix transfers within EMR-based workflows. Does it still have a place in your day-to-day work?
- 🧪 **From PoC to Production**: Right now, the code feels PoC-oriented and test-friendly. Have you used this approach in real production workloads? If so, how did you handle:
  - **Exponential backoff** and retries?
  - **Partial failures** and error handling?
  - **Monitoring and alerting** (e.g., CloudWatch, custom logs)?

---

**Tags**:  
#aws #s3 #boto3 #parallelism #spark #dataengineering #til #scala2python #migration #tools
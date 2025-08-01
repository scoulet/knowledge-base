# TIL – How to List S3 Files Between Two Dates

## 🎯 Problem / Context  
Standard S3 file listing with `boto3` (or AWS CLI) does **not support filtering by last modified date**.  
In ingestion workflows, you often need to detect **only new or updated files** since the last successful run.

However, calling `list_objects_v2` gives you *everything* under a prefix — you have to filter manually in your code.  
This becomes essential in large buckets to avoid unnecessary data processing and cost.

## 🐛 Common Pitfall  
- Assuming AWS lets you list files "between two timestamps" — it doesn’t.  
- Using `aws s3 sync` or `s3-dist-cp` expecting smart filtering — it won’t happen.  
- Ignoring timestamps and reprocessing entire prefixes at each run.

## 💡 Solution / Snippet  
A clean, Pythonic utility using `boto3` that **paginates S3 listings** and filters based on `LastModified` using a **generator pattern**.

```
# Install boto3 if needed
pip install boto3
```

```python
import boto3
from datetime import datetime
from typing import Optional, Iterable


def list_s3_files_between_dates(
    bucket: str,
    prefix: str,
    start_date: Optional[datetime] = None,
    end_date: Optional[datetime] = None,
    max_files: Optional[int] = None,
    verbose: bool = False
) -> Iterable[str]:
    """
    List S3 object keys in a bucket/prefix filtered by last modified date.

    Yields:
        str: Matching S3 object keys, streamed one by one.
    """
    s3 = boto3.client("s3")
    paginator = s3.get_paginator("list_objects_v2")

    count = 0
    for page in paginator.paginate(Bucket=bucket, Prefix=prefix):
        for obj in page.get("Contents", []):
            key = obj["Key"]
            modified = obj["LastModified"]

            if start_date and modified < start_date:
                continue
            if end_date and modified > end_date:
                continue

            if verbose:
                print(f"[MATCH] {key} (LastModified: {modified.isoformat()})")

            yield key

            count += 1
            if max_files and count >= max_files:
                return
```

## 🔍 Why It Works  
- Combines **manual filtering** with **AWS-native pagination** to remain scalable.  
- `yield` makes this a **generator**, so only one file is in memory at a time.  
- You can run `for f in files: process(f)` safely — even with millions of objects.

## 🧯 Why This Isn't a Job for `s3-dist-cp` or `aws s3 sync`  
You might think you could do the same with AWS tools — but no:

- `aws s3 sync` compares file content or size — not modification date.  
- `s3-dist-cp` (on EMR) doesn’t support `--startDate` or `--endDate`.  
- `S3 Select` only queries inside files, not their metadata.  
- No native S3 mechanism lets you copy files *between two timestamps*.  
👉 You're forced to **list + filter + act manually** — this snippet fills that gap.

## 🛠️ When to Use It  
- Delta ingestion workflows in Data Lakes  
- Detecting recent file drops for alerts or processing  
- Backfilling data over specific time windows  
- Efficient cleanup or audit routines

## 🧠 Key Ideas to Remember  
- S3 APIs don’t support time-based filtering — it’s up to you.  
- Use `yield` to scale gracefully over huge file sets.  
- Combine this with checkpointing logic to make ingestion idempotent.

## 📝 Sources
- [`boto3` S3 paginator docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Paginator.ListObjectsV2)  

## 📝 What to add to make this an article
- Benchmarks with and without this filtering on large buckets  
- Storing checkpoints in DynamoDB or a local state file  
- Building a minimal delta-ingestion framework around this pattern  
- Comparing this approach with EventBridge / notification-based ingestion

---

**Tags**: #aws #s3 #boto3 #python #dataengineering #deltaingestion #cloud #til


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
A simple Python utility using `boto3` that paginates S3 listings and filters based on `LastModified`.

```
# Install boto3 if needed
pip install boto3
```

```python
import boto3
from datetime import datetime
from typing import Optional, List

def list_files_between_dates(
    bucket: str,
    prefix: str,
    start_date: Optional[datetime] = None,
    end_date: Optional[datetime] = None,
    max_files: Optional[int] = None
) -> List[str]:
    s3 = boto3.client("s3")
    paginator = s3.get_paginator("list_objects_v2")

    matched_files = []
    for page in paginator.paginate(Bucket=bucket, Prefix=prefix):
        for obj in page.get("Contents", []):
            last_modified = obj["LastModified"]

            if start_date and last_modified < start_date:
                continue
            if end_date and last_modified > end_date:
                continue

            matched_files.append(obj["Key"])
            if max_files and len(matched_files) >= max_files:
                return matched_files

    return matched_files
```

## 🔍 Why It Works  
- Combines **manual filtering** with **AWS-native pagination** to remain scalable.  
- Applies filters **as early as possible**, instead of post-processing huge lists.  
- Enables workflows like: "process only files modified since last run".

## 🧯 Why This Isn't a Job for `s3-dist-cp` or `aws s3 sync`  
You might think you could do the same with AWS tools like `sync` or `s3-dist-cp` — **but no**:

- `aws s3 sync` only compares file content or size — **not modification dates**.  
- `s3-dist-cp` (on EMR) doesn’t support `--startDate` or `--endDate` parameters.  
- `S3 Select` doesn’t work here either: it queries *inside files*, not metadata.  
- No native S3 mechanism lets you copy files *between two timestamps*.  
👉 You're forced to **list + filter + copy manually** — this snippet fills that gap.

## 🛠️ When to Use It  
- Delta ingestion workflows in Data Lakes.  
- Detecting recent file drops for processing or alerts.  
- Backfilling data for a specific time window.  
- Periodic cleanups or audits over time-filtered sets.

## 🧠 Key Ideas to Remember  
- S3 does **not** support time-based filtering in its API — filtering is your job.  
- Use `LastModified` and pagination together for scalable file selection.  
- This pattern unlocks **idempotent ingestion logic** in pipeline systems.

## 📝 Sources (optional)  
- [`boto3` S3 paginator docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Paginator.ListObjectsV2)  
- Scala original from `datafactory`'s S3ClientUtil

## 📝 What to add to make this an article
- Benchmarks with vs without filtering on large buckets  
- Tips for storing checkpoints (DynamoDB, S3 tags, etc.)  
- Combine with file copy (`copy_object`) to build a real delta sync tool  
- Explore edge cases: deleted objects, versioning, eventual consistency

---

**Tags**: #aws #s3 #boto3 #python #dataengineering #deltaingestion #cloud #til

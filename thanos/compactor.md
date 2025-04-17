# Thanos Compactor 

## Explanation 

### 💡 What does the **Thanos Compactor** do?

The **Compactor** is a component of Thanos that:

1. **Compacts smaller blocks into larger ones**  
   - Prometheus writes data in 2-hour blocks.
   - The sidecar uploads these raw blocks to object storage.
   - Over time, this results in lots of small blocks.
   - The Compactor merges them (e.g., 2h blocks → 10h → 48h → 2-week blocks), improving query performance.

2. **Downsamples old data** (optional)  
   - Creates lower-resolution versions (e.g., 5m, 1h step sizes) to speed up long-range queries.
   - Useful for dashboards or queries covering weeks/months of data.

3. **Applies retention policies**  
   - Deletes old blocks that exceed the configured retention period.

---

### So when do you need the Compactor?

You need the **Compactor** if:
- You're using **object storage** (S3, GCS, etc.) for long-term data
- You want efficient storage and faster queries
- You need **downsampling** or **retention control**

### On what does then compactor act 

  * The compactor act directly on the data uploaded to s3-storage

### How many times do I need to install compactor ? 

  * I will only need to install it once in the cluster (and i needs read/write access to the s3 data) 




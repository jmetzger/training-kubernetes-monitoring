# Thanos 

## What is Thanos ?

**Thanos** is an open-source **highly available, long-term storage solution for Prometheus**. It extends Prometheus by adding **global querying, deduplication, downsampling, and retention capabilities** across multiple Prometheus instances.

## Why use Thanos (or: What are the problems with Prometheus)

### 🔍 In simple terms:
Prometheus is great for monitoring, but it has **limitations**:
- **No built-in high availability** (HA)
- **No long-term storage** (TSDB is local and limited)
- **Difficult to query across multiple Prometheus servers**

**Thanos solves all that** by sitting on top of Prometheus.

## What are the key components of Thanos ?

### 💡 Key Components of Thanos:

1. **Sidecar** – Sits next to each Prometheus, uploads data to object storage (e.g. S3, GCS, MinIO).
2. **Store Gateway** – Reads historical data from the object storage.
3. **Query** – Global query engine that federates multiple Prometheus instances.
4. **Compactor** – Compacts and down-samples metrics data to reduce storage usage.
5. **Ruler** – Allows global alerting & recording rules.
6. **Receiver (optional)** – Receives remote writes directly, useful in cloud-native setups.

## Benefits 

### 📦 What You Get with Thanos:

| Feature               | Benefit                                  |
|----------------------|------------------------------------------|
| Global Query View    | One place to query multiple Prometheus   |
| HA via Sidecars      | Prometheus replicas + deduplication      |
| Object Storage       | S3/GCS/MinIO for infinite retention      |
| Downsampling         | Better performance for old data          |
| Alerting Rules       | Centralized alerting with Thanos Ruler   |

## Architecture 

![image](https://github.com/user-attachments/assets/90610bd3-c63b-4e62-b03e-41e0eaf4a371)

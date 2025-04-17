# How does Grafana connect to Thanos ? 

## Route 

![Grafana to Thanos Routing](/images/grafana-thanos-route.png)


## Explanation 

### 📊 Grafana connects to **Thanos Query**

---
### 🔍 **Thanos Query** is the main entry point for querying Thanos

- It speaks the **Prometheus HTTP API** (`/api/v1/query`, `/api/v1/query_range`, etc.), so Grafana can treat it like any Prometheus data source.
- It **aggregates data** from:
  - Live Prometheus instances (via Sidecars)
  - Historical data (via Store Gateways)
  - Downsampled data
  - Optional: Thanos Ruler, Thanos Receive

---

### ✅ In Grafana, you do this:

1. **Add a new data source**
2. Choose **Prometheus** (yes — even though it’s Thanos)
3. Set the **URL to your Thanos Query endpoint** (e.g., `http://thanos-query.monitoring.svc.cluster.local:9090`)
4. Optionally enable things like:
   - **Custom HTTP headers** if needed
   - **Query timeout** tuning for large ranges

---

### 🚀 Pro tips:

- **Deduplication** is handled by Thanos Query, so if you have HA pairs of Prometheus, Thanos Query hides duplicates.
- You can expose **multiple Thanos Query components** for HA and let Grafana load balance across them (using a proxy or round-robin).
- If you use **downsampling**, Thanos Query will automatically pick the best resolution for large-range queries.

---

Let me know if you want a quick Grafana+Thanos example dashboard or tips on setting up alerts across clusters!

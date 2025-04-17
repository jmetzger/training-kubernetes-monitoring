# Prometheus (Full) with Sidecar Thanos and Grafana

Great! Here's a basic **Kubernetes manifest example** and also a **Helm-based setup** outline for deploying **Thanos components** alongside **Prometheus**, including long-term storage with **MinIO (S3-compatible)**. This setup is **lab-ready**, simple, and suitable for small-scale clusters or training.

---

## 🧩 Thanos Setup with Helm (Prometheus + Thanos + MinIO)

### Option A: 🧱 **Using Helm**

#### 1. Add Repos
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

#### 2. Deploy MinIO (S3-compatible storage)
```bash
helm install minio bitnami/minio \
  --set accessKey.password=minioadmin \
  --set secretKey.password=minioadmin \
  --set defaultBuckets=thanos \
  --namespace monitoring --create-namespace
```

---

#### 3. Deploy Prometheus with Thanos Sidecar

Create `values-prometheus.yaml`:
```yaml
extraContainers:
  - name: thanos-sidecar
    image: quay.io/thanos/thanos:v0.34.0
    args:
      - sidecar
      - --tsdb.path=/data
      - --prometheus.url=http://localhost:9090
      - --objstore.config-file=/etc/thanos/objstore.yaml
    volumeMounts:
      - name: prometheus-data
        mountPath: /data
      - name: thanos-objstore
        mountPath: /etc/thanos

extraVolumes:
  - name: thanos-objstore
    configMap:
      name: thanos-objstore

service:
  enabled: true
  type: ClusterIP
```

Create a ConfigMap for `objstore.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: thanos-objstore
  namespace: monitoring
data:
  objstore.yaml: |
    type: s3
    config:
      bucket: "thanos"
      endpoint: "minio.monitoring.svc.cluster.local:9000"
      access_key: "minioadmin"
      secret_key: "minioadmin"
      insecure: true
```

Install Prometheus:
```bash
helm install prometheus prometheus-community/prometheus \
  -f values-prometheus.yaml \
  --namespace monitoring
```

---


### 🔁 **Prometheus Responsibilities**
- **Scrapes metrics** from your applications and Kubernetes targets.
- Stores those metrics locally (TSDB).
- **Has all the `scrape_configs`** (e.g., pods, services, kubelets, etc.).
- Runs as usual with no changes to scraping behavior.
- Now runs **with a Thanos Sidecar** container.

---

### 🔌 **Thanos Sidecar Responsibilities**
- Reads data from the local TSDB (no scraping itself).
- Uploads blocks to object storage (e.g., MinIO, S3).
- Exposes a gRPC endpoint to Thanos Query or Store Gateway.
- Acts as a **bridge** between Prometheus and Thanos.

---

### 🧠 Where Are Scrape Configs Defined?
- They are defined in:
  - `prometheus.yaml` if you're managing raw config.
  - Or via `values.yaml` in Helm under `serverFiles.prometheus.yml.scrape_configs`.

Example in Helm:
```yaml
serverFiles:
  prometheus.yml:
    scrape_configs:
      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
          - role: node
        relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
```


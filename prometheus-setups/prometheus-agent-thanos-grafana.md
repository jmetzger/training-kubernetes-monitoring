# Stack with prometheus-agent && Centralized Monitoring Cluster 

## Part 1: The agent side (on each cluster) 

Awesome — here’s a **complete working setup** for deploying **Prometheus in `agent` mode with `remote_write`** using **Helm** in Kubernetes. We'll use the **`kube-prometheus-stack`** Helm chart, and assume you're pushing data to a **Thanos Receive** endpoint.

---

## 🧩 Goal
- Deploy Prometheus in **Agent mode**
- Use **`remote_write`** to send metrics to **Thanos Receive**
- Retain **no local TSDB**
- Scrape Kubernetes nodes (or other jobs)
- Run everything in a namespace like `monitoring`

---

## 🚀 Step-by-step Helm Setup

### 1. 📦 Add Helm Repo and Create Namespace
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace monitoring
```

---

### 2. 📝 Create `values-agent.yaml`

```yaml
# values-agent.yaml

prometheus:
  prometheusSpec:
    enableRemoteWriteReceiver: false
    enableAdminAPI: false
    externalLabels:
      cluster: my-cluster

    # 💡 Enable Agent mode
    extraArgs:
      enable-feature: agent

    # ⛔ No TSDB storage needed
    retention: 0h
    storageSpec: {}

    # 📤 Remote write to Thanos Receive
    remoteWrite:
      - url: http://thanos-receive.monitoring.svc:19291/api/v1/receive
        write_relabel_configs:
          - source_labels: [__name__]
            regex: ".*"
            action: keep

    # ✅ Add your scrape jobs here
    additionalScrapeConfigs:
      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
          - role: node
        relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
          - source_labels: [__address__]
            target_label: __address__
            regex: (.*):10250
            replacement: ${1}:9100
            action: replace

# Optional: disable default recording/alerting rules
kube-prometheus-stack:
  defaultRules:
    create: false # default rules can be set to true, but agent does not use them 

# Optional: disable other components like alertmanager, grafana, etc.
alertmanager:
  enabled: false

grafana:
  enabled: false
```

> You can also use `configMap` for `additionalScrapeConfigs` if you prefer that method.

---

### 3. 🛠 Install the chart with custom values

```bash
helm install prom-agent prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values-agent.yaml
```

This will deploy:
- Prometheus running in **agent mode**
- Scraping Kubernetes nodes
- Streaming data to `remote_write` (e.g., Thanos Receive)

---


## Part 2: The monitoring cluster side (once) 

Perfect — here’s a complete Helm-based setup to deploy **Thanos components** in Kubernetes:

- ✅ **Thanos Receive** — accepts remote_write from Prometheus agents  
- ✅ **Thanos Store Gateway** — reads from object storage (e.g. S3 or MinIO)  
- ✅ **Thanos Query (Querier)** — unified query layer  
- ✅ **Grafana** — dashboards that query Thanos Query  

We’ll also deploy **MinIO** as an S3-compatible store.

---

## 📦 Folder Structure (you'll get this as a zipped Helm lab)
```
thanos-lab/
├── values/
│   ├── minio.yaml
│   ├── thanos-receive.yaml
│   ├── thanos-store.yaml
│   ├── thanos-query.yaml
│   ├── grafana.yaml
├── install.sh   # helper script to install everything
```

---

## ✅ 1. MinIO (Object Storage)

### `values/minio.yaml`
```yaml
auth:
  rootUser: minioadmin
  rootPassword: minioadmin

defaultBuckets: thanos

resources:
  requests:
    memory: 256Mi
    cpu: 100m
```

### Install:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install minio bitnami/minio -n monitoring --create-namespace -f values/minio.yaml
```

---

## ✅ 2. Thanos Receive

### `values/thanos-receive.yaml`
```yaml
objstoreConfig:
  type: s3
  config:
    bucket: "thanos"
    endpoint: "minio.monitoring.svc.cluster.local:9000"
    access_key: "minioadmin"
    secret_key: "minioadmin"
    insecure: true

receive:
  enabled: true
  tsdbRetention: 24h
  replicaLabel: replica
  hashPartitioning:
    replicationFactor: 1
    enableSharding: false

service:
  type: ClusterIP
```

### Install:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install thanos-receive bitnami/thanos -n monitoring -f values/thanos-receive.yaml
```

---

## ✅ 3. Thanos Store Gateway

### `values/thanos-store.yaml`
```yaml
objstoreConfig:
  type: s3
  config:
    bucket: "thanos"
    endpoint: "minio.monitoring.svc.cluster.local:9000"
    access_key: "minioadmin"
    secret_key: "minioadmin"
    insecure: true

storegateway:
  enabled: true

service:
  type: ClusterIP
```

### Install:
```bash
helm install thanos-store bitnami/thanos -n monitoring -f values/thanos-store.yaml
```

---

## ✅ 4. Thanos Query

### `values/thanos-query.yaml`
```yaml
query:
  enabled: true
  replicaLabel: replica

  stores:
    - thanos-receive.monitoring.svc.cluster.local:10901
    - thanos-store.monitoring.svc.cluster.local:10901

service:
  type: ClusterIP
```

### Install:
```bash
helm install thanos-query bitnami/thanos -n monitoring -f values/thanos-query.yaml
```

---

## ✅ 5. Grafana (with Thanos Query as datasource)

### `values/grafana.yaml`
```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
      - name: Thanos
        type: prometheus
        access: proxy
        url: http://thanos-query.monitoring.svc.cluster.local:9090
        isDefault: true
        jsonData:
          timeInterval: "15s"
adminPassword: "admin"
```

### Install:
```bash
helm install grafana bitnami/grafana -n monitoring -f values/grafana.yaml
```

---

## 🚀 One-liner install script (optional)

You can create a small script `install.sh`:

```bash
#!/bin/bash
NAMESPACE="monitoring"
helm install minio bitnami/minio -n $NAMESPACE --create-namespace -f values/minio.yaml
helm install thanos-receive bitnami/thanos -n $NAMESPACE -f values/thanos-receive.yaml
helm install thanos-store bitnami/thanos -n $NAMESPACE -f values/thanos-store.yaml
helm install thanos-query bitnami/thanos -n $NAMESPACE -f values/thanos-query.yaml
helm install grafana bitnami/grafana -n $NAMESPACE -f values/grafana.yaml
```

---

Would you like me to package this into a downloadable ZIP for you?

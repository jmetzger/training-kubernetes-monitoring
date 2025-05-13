# Installation von Loki (Achtung: nur Single Instance)  

## Voraussetzung: 

  * Prometheus / Grafana Monitoring - Stack läuft bereits im namespace "monitoring"
  * Prometheus ist als release "prometheus" mit helm installiert 

## 🥇 Schritt 1: Projektordner anlegen

```bash
cd ~
mkdir loki-single
cd loki-single
```

Damit wird dein Projekt im Home-Verzeichnis (`~/loki-single`) angelegt.

---

## 🥈 Schritt 2: `values.yaml` erstellen

```
nano values.yaml
```

```
loki:
  commonConfig:
    replication_factor: 1
  schemaConfig:
    configs:
      - from: "2024-04-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
  pattern_ingester:
      enabled: true
  limits_config:
    allow_structured_metadata: true
    volume_enabled: true
  ruler:
    enable_api: true

minio:
  enabled: true
      
deploymentMode: SingleBinary

singleBinary:
  replicas: 1

# Zero out replica counts of other deployment modes
backend:
  replicas: 0
read:
  replicas: 0
write:
  replicas: 0

ingester:
  replicas: 0
querier:
  replicas: 0
queryFrontend:
  replicas: 0
queryScheduler:
  replicas: 0
distributor:
  replicas: 0
compactor:
  replicas: 0
indexGateway:
  replicas: 0
bloomCompactor:
  replicas: 0
bloomGateway:
  replicas: 0

```

## 🥉 Schritt 3: Installieren 

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install loki grafana/loki-stack \
  --namespace loki --create-namespace \
  -f values.yaml
```

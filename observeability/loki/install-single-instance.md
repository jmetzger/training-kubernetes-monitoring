# Installation von Loki (Achtung: nur Single Instance)  

## Voraussetzung: 

  * Prometheus / Grafana Monitoring - Stack läuft bereits im namespace "monitoring"
  * Prometheus ist als release "prometheus" mit helm installiert 

## Schritt 0: csi ausrollen (nfs-server muss eingerichtet sein) 

```
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version v4.11.0
```

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 10.135.0.34
  share: /var/nfs
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

```
kubectl apply -f .
```

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

helm upgrade --install loki grafana/loki \
  --namespace loki --create-namespace --version 6.29.0 \
  -f values.yaml
```

## Ref:

  * https://grafana.com/docs/loki/latest/setup/install/helm/install-monolithic/

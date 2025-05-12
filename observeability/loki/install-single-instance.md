# Installation von Loki (Achtung: nur Single Instance)  

## 🥇 Schritt 1: Projektordner anlegen

```bash
cd ~
mkdir loki-single
cd loki-single
```

Damit wird dein Projekt im Home-Verzeichnis (`~/loki-single`) angelegt.

---

## 🥈 Schritt 2: `values.yaml` erstellen

```bash
cat <<EOF > values.yaml
loki:
  enabled: true

promtail:
  enabled: true
  config:
    clients:
      - url: http://loki:3100/loki/api/v1/push

grafana:
  enabled: false

serviceAccounts:
  loki:
    create: true

serviceMonitor:
  enabled: true

rbac:
  create: true
EOF
```

---

## 🥉 Schritt 3: `install.sh` Skript erstellen

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install loki grafana/loki-stack \
  --namespace loki --create-namespace \
  -f values.yaml
```

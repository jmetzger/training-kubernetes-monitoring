# Exercise: Nginx mit ServiceMonitor anlegen 

## Voraussetzung:

  * kube-prometheus-stack muss installiert sein -> [Kube-Prometheus-Stack installieren](prometheus-grafana/prometheus-grafana/install-with-helm-letsencrypt-basic-auth.md)


## 🔧 Vorbereitung: Verzeichnisstruktur anlegen

```bash
cd ~
mkdir -p manifests
cd manifests
mkdir svcm-nginx
cd svcm-nginx
```

> 🔎 Alle YAML-Dateien werden in diesem Verzeichnis erstellt und mit `kubectl apply -f .` angewendet.

---

## 1. Namespace

```bash
nano 01-namespace.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web-demo
```

```bash
kubectl apply -f .
```

---

## 2. ConfigMap: Stub Status aktivieren

```bash
nano 02-nginx-stubstatus-configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-stubstatus
  namespace: web-demo
data:
  default.conf: |
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }

      location /stub_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
      }
    }
```

```bash
kubectl apply -f .
```

---

## 3. Deployment mit Sidecar (Exporter)

```bash
nano 03-nginx-deployment-metrics.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: web-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      - name: exporter
        image: nginx/nginx-prometheus-exporter:latest
        args:
        - "-nginx.scrape-uri=http://localhost:80/stub_status"
        ports:
        - containerPort: 9113
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-stubstatus
```

```bash
kubectl apply -f .
```

---

## 4. Service mit zusätzlichem Metrics-Port

```bash
nano 04-nginx-service-with-metrics.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: web-demo
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: metrics
    port: 9113
    targetPort: 9113
```

```bash
kubectl apply -f .
```

## 5. Ingress (Optional)

```bash
nano 06-nginx-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: web-demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app.tln1.do.t3isp.de
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

```bash
kubectl apply -f .
```


## 6. ServiceMonitor

```bash
nano 05-nginx-servicemonitor.yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx
  namespace: web-demo
  labels:
    release: prometheus  # muss zu Helm-Werten passen!
spec:
  selector:
    matchLabels:
      app: nginx
  namespaceSelector:
    matchNames:
    - web-demo
  endpoints:
  - port: metrics
    path: /metrics
    interval: 15s
```

```bash
kubectl apply -f .
```

```
# Welches Label prometheus hat, könnt ihr prüfen
kubectl -n monitoring get pods -l release=prometheus
```

```
# Ist der ServiceMonitor konfiguriert ?
kubectl -n web-demo get servicemonitor nginx
kubectl -n web-demo get smon nginx
kubectl -n web-demo describe smon nginx 
```

## 7. Targets finden (in web gui) 

```
# im Browser öffnen und nach web-demn suchen 
https://prometheus.<du>.do.t3isp.de/targets
# oder über tunnel
```

### 8. mit promql abfragen

```
1. Zunächst finden wir heraus, welche labels diese pods haben (siehe Punkt 7)
das sieht nach job="nginx" aus

Jeder ServiceMonitor (z.B. unser, der nginx heisst), wird beim Scrapen als job="<serviceMonitorName>"
automatisch von Kubernetes abgefragt.
```

```
d.h. wir können Fragen

# (gilt dann für alle pods) 
up
# gilt für alle pods in einen für bestimmten job  
up {job="nginx"}
# gilt für alle pods in einem bestimmten namespace
up {namespace="web-demo"}
# and combining all endpoints with job=nginx in the namespace web-demo
up {job="nginx",namespace="web-demo"}
```

```
# Regular Expressions also work:
up{namespace="web-demo", pod=~"nginx-.*"}
```

```
#Pratical - on: https://prometheus1.tln1.do.t3isp.de/query
+ enter:
up {"job=nginx"} + Press "Execute"
```

![image](https://github.com/user-attachments/assets/a946076e-9a62-4dd3-a468-ed6653524616)

```
You can not also click on Graph
```

## 9. In Grafana ein Dashboard erstellen 

### Step 1: New Dashboard 

```
Oben rechts auf neues Dashboard erstellen klicken
-->
```
![image](https://github.com/user-attachments/assets/150b70df-9ef3-4014-b5a4-69ae7f62af06)

### Step 2: Add Visualization 

![image](https://github.com/user-attachments/assets/b91e2df1-b11a-4d19-8b06-99ef1de66b08)

### Step 3: Datasource -> Prometheus (Default) auswählen / Visualisation + Query definieren

  * DataSource Prometheus is bereits vorkonfiguriert

![image](https://github.com/user-attachments/assets/c1c2af0b-dfda-4e31-9470-5864dd630cbc)

  * Choose Visualisation ( Stat, Gauge, or Bar Gauge )
  * Set the query: up | job | nginx
  * run query

![image](https://github.com/user-attachments/assets/c70bc905-7196-4b73-a625-cbd9132530a3)





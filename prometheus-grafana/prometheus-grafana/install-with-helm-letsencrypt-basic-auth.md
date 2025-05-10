# Prometheus with Grafana, letsencrypt and basic auth for prometheus (Install with helm)

  * using the kube-prometheus-stack (recommended !: includes important metrics)

## Attention: Upgrades and uninstall can be a bit tricky 

  * CRD's need to deleted manually after uninstall
  * Before Upgrade update the CRD's
  * https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/UPGRADE.md

## What do we want to do ? 

  * We want to protect prometheus with basic-auth
  * We want to protect alertmanager with basic-auth 
  * We want to use letsencrypt 

## Prerequisites 

```
# 1. With have setup ingress-controller Service type:LoadBalancer -> external
# 2. We have a subdomain 
# 3. Already done for you 
sudo apt install apache2-utils
```

## Step 1: Create our project - folder (just to be organized) 

```
cd
mkdir -p manifests 
cd manifests 
mkdir -p monitoring 
cd monitoring 
```

## Step 2: Create basic-auth  

```
kubectl create ns monitoring 
htpasswd -c auth admin  # Enter your desired password
kubectl create secret generic prometheus-basic-auth --from-file=auth -n monitoring
```

## Step 3: Install cert-manager  

```
helm repo add jetstack https://charts.jetstack.io
```

```
nano cert-manager-values.yml 
```

```
crds:
  enabled: true
```

```
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace -f cert-manager-values.yml 
```

## Step 4: Create ClusterIssuer 

```
nano clusterissuer.yaml
```

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
```

```
kubectl apply -f clusterissuer.yaml 
```

## Step 5: Prepare Monitoring Stack (values - file) 

```
nano monitoring-values.yaml
```

```
grafana:
  fullnameOverride: grafana
  enabled: true
  adminUser: admin
  adminPassword: "yourStrongPassword"
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      cert-manager.io/cluster-issuer: letsencrypt-prod
    hosts:
      - grafana.<du>.do.t3isp.de
    path: /
    pathType: Prefix
    tls:
      - hosts:
          - grafana.<du>.do.t3isp.de
        secretName: grafana-tls

prometheus:
  fullnameOverride: prometheus 
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/auth-type: basic
      nginx.ingress.kubernetes.io/auth-secret: prometheus-basic-auth
      nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
      cert-manager.io/cluster-issuer: letsencrypt-prod
    hosts:
      - prometheus.<du>.do.t3isp.de
    paths:
      - /
    pathType: Prefix
    tls:
      - hosts:
          - prometheus.<du>.do.t3isp.de
        secretName: prometheus-tls

# Optional: Persist data
prometheusOperator:
  admissionWebhooks:
    enabled: true

alertmanager:
  fullnameOverride: alertmanager
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/auth-type: basic
      nginx.ingress.kubernetes.io/auth-secret: prometheus-basic-auth
      nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
      cert-manager.io/cluster-issuer: letsencrypt-prod
    hosts:
      - alertmanager.<du>.do.t3isp.de
    paths:
      - /
    pathType: Prefix
    tls:
      - hosts:
          - alertmanager.<du>.t3isp.de
        secretName: alertmanager-tls


kube-state-metrics:
  fullnameOverride: kube-state-metrics

prometheus-node-exporter:
  fullnameOverride: node-exporter
```

## Step 6: Install with helm 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f monitoring-values.yaml --namespace monitoring --create-namespace --version 72.3.0
```

## Step 7: Connect to prometheus from the outside world 

```
https://prometheus.<du>.t3isp.de
```

## Step 8: Connect to the grafana from the outside world 

```
https://grafana.<du>.t3isp.de
```

## References:

  * https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md
  * https://artifacthub.io/packages/helm/prometheus-community/prometheus

  

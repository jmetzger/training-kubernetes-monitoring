# Datasource von loki automatisch provisionieren 

  * Damit wir loki auslesen können, müssen wir eine neue datasource einrichten 

## Hintergründe 

  * grafana sollte eine sidecar haben (2. Container)
  * Dieser liest bestimmte ConfigMaps (für datasources und dasboards, aber keine Alterts) automatisch ein

## Variante 1: BESTE ! über configmap 

```
# man siehe diese sidecar container - mit
# Interessanterweise funktioniert das auch ohne vollständigen Namen des Pods ?
# kubectl -n monitoring describe pods grafana 
nano 02-configmap-datasource.yaml 

```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-datasource
  namespace: monitoring
  labels:
    grafana_datasource: "1"
data:
  loki-datasource.yaml: |
    apiVersion: 1
    datasources:
      - name: Loki
        type: loki
        access: proxy
        url: http://loki.loki.svc.cluster.local:3100
        isDefault: false
        jsonData:
          maxLines: 1000
```

```
kubectl apply -f .
```

## Variante 2: Nur im Notfall 

# Datasource ueber helm anpassen (kube-prometheus-stack)

  * Damit wir loki auslesen können, müssen wir eine neue datasource einrichten

## Values.yaml entsprechend ergänzen 

```
# Achtung, alle anderen Sachen müssen da auch noch mit rein
# s. helm -n monitoring get values prometheus  
grafana:
  enabled: true
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Loki
          type: loki
          access: proxy
          url: http://loki.loki.svc.cluster.local:3100
          isDefault: false
          editable: true
```

```
# Version anpassen
# herausfinden mit helm list -A 
helm upgrade --install --namespace=monitoring --create-namespace --version=x.x.x.x -f values.yaml
```

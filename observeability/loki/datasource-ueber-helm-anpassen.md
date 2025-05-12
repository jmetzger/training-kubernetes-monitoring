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

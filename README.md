# Kubernetes Monitoring 

## Agenda 

  1. Vorbereitung
     * [Self-Service Cluster ausrollen](/monitoring/training-stack/install.md)

  1. Kubernetes Monitoring Grundlagen
     * [Abgrenzung zu Observeability](/monitoring/grundlagen/monitoring-vs-observeability.md)
  
  1. Kubernetes Monitoring (Single Cluster / Instance of Prometheus) 
     * [Prometheus Monitoring Server (Overview)](prometheus/overview.md)
     * [Prometheus / Grafana Stack installieren (advanced)](prometheus-grafana/prometheus-grafana/install-with-helm-letsencrypt-basic-auth.md)
     * [Prometheus / blackbox exporter](prometheus-grafana/z_blackbox-exporter.md)
    
  1. Prometheus Praxis
     * [Nginx mit ServiceMonitor und export konfigurieren (sidecar)](monitoring/praxis/03-nginx-servicemonitor.md)

  1. Grafana - Alterting and Notifications
     * [Grafana neuen alert anlegen](/monitoring/alerts/neuen-alert-in-grafana-anlegen.md)
     * [Grafana Notifications/Contact points](/monitoring/alerts/notification-in-grafana.md)

  1. Kubernetes Multi-Cluster (Types of setups including disadvantags/advantages)
     * [Recommended: Variant 1: prometheus agent + thanos/grafana stack](prometheus-setups/prometheus-agent-thanos-grafana.md)
     * [Variant 2: Full prometheus in each cluster with thanos sidecar](prometheus-setups/prometheus-full-sidecar-thanos-grafana.md)

  1. Grafana Loki  
     * [Installation von Grafana Loki - Singe Instance - für Testing](observeability/loki/install-single-instance.md)
     * [Datasource in Grafana bereitstellen per helm](observeability/loki/install-single-instance.md)
     * [Wo finde ich Loki in Grafana ?](observeability/loki/where-to-find-loki-in-grafana.md)

  ## Backlog / Sammlung 

  1. Prometheus
     * [Prometheus-Metriktypen (engl. metric types)](prometheus/metrics/overview.md)

  1. Kubernetes Multi-Cluster (using Thanos) 
     * [Prerequisites: What is Thanos](thanos/what-is-thanos.md)
     * [Components](thanos/components.md)
     * [Thanos Compactor](thanos/compactor.md)

  1. Kubernetes Multi-Cluster (using Cortex - multi-tenant tsdb's) 

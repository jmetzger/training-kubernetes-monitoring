# Prometheus bitte kein Agent verwenden, immer den vollständigen Prometheus-Server 

## Warum ? 

 * Coole Objekte wie PodMonitor, ServiceMonitor, PrometheusRules funktionieren
 * Das ist schlecht und macht Dein unnötig schwer.
 * Dann musst du nämlich die alten ScrapeConfigs verwenden (IHHHHH !) 

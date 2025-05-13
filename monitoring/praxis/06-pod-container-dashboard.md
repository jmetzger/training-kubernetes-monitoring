# Übung Grafana Dashboard

## **Grafana GUI-Anleitung: Pod & Container Dashboard**

### **1. Dashboard erstellen**

1. In Grafana links auf **“Dashboards” > “New” > “New Dashboard”** klicken.
2. Auf **“Add a new panel”** klicken.

---

### **2. Panel 1: Nicht-laufende Pods pro Namespace**

* **Titel:** „Fehlgeschlagene Pods je Namespace“
* **Abfrage (PromQL):**

  ```promql
  count by (namespace) (kube_pod_status_phase{phase=~"Pending|Failed|Unknown"})
  ```
* **Panel-Typ:** **Bar chart**
* **X-Achse:** `namespace`
* **Y-Achse:** Anzahl Pods
* Speichern mit **“Apply”**

---

### **3. Panel 2: Top 5 Container mit höchsten Restarts**

* Neues Panel > Titel: „Top 5 Container Restarts“
* **Abfrage:**

  ```promql
  topk(5, sum by (pod, container) (kube_pod_container_status_restarts_total))
  ```
* **Panel-Typ:** **Table**
* Optional: Transformationen aktivieren für bessere Darstellung
* Apply

---

### **4. Panel 3: CPU-Nutzung pro Container**

* Neues Panel > Titel: „CPU pro Container (millicores)“
* **Abfrage:**

  ```promql
  sum by (namespace, pod, container) (
    rate(container_cpu_usage_seconds_total{container!=""}[5m])
  ) * 1000
  ```
* Panel-Typ: **Time series**
* Legende anzeigen: `{{namespace}} / {{pod}} / {{container}}`
* Apply

---

### **5. Panel 4: Memory-Nutzung pro Container**

* Titel: „Speicher pro Container (MiB)“
* **Abfrage:**

  ```promql
  sum by (namespace, pod, container) (
    container_memory_usage_bytes{container!=""}
  ) / (1024 * 1024)
  ```
* Panel-Typ: **Time series**
* Legende anzeigen: `{{namespace}} / {{pod}} / {{container}}`
* Einheit: **MiB**
* Apply

---

### **6. Panel 5: Container Ready Status als Gauge**

* Titel: „Container bereit (Ready)“
* **Abfrage:**

  ```promql
  max by (pod, container) (kube_pod_container_status_ready)
  ```
* **Panel-Typ:** **Gauge**
* **Thresholds:**

  * Min: `0`, Max: `1`
  * Thresholds definieren: `0 = rot`, `1 = grün`

    * `0.5` -> rot
    * `0.99` -> gelb
    * `1` -> grün
* Einheit: none
* Apply

---

### **7. Optional: Variables hinzufügen (Namespace, Pod)**

* Dashboard bearbeiten (Zahnrad oben rechts) > **Variables** > „New“:

  * **Name:** `namespace`
  * **Type:** Query
  * **Datasource:** Prometheus
  * **Query:** `label_values(kube_pod_info, namespace)`
* Genauso für `pod` mit Query: `label_values(kube_pod_info{namespace="$namespace"}, pod)`

---

### **8. Save & teilen**

* Dashboard speichern (Diskette-Symbol oben)
* Optional: In Ordner verschieben oder als Start-Dashboard setzen

---

**Möchtest du eines der Panels genauer als Screenshot oder mit JSON-Code sehen?** Ich kann dir auch ein Demo-Dashboard JSON liefern.

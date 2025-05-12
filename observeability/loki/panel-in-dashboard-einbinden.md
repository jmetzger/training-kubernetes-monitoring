# Panel im Dashboard (Loki)

## 📊 1. **Dashboards → Logs Panels einbinden**

In jedem **Dashboard-Panel** kannst du Loki nutzen:

1. Neues Panel erstellen
2. Als **Data Source: Loki** auswählen
3. Unter **Query** LogQL-Abfrage eingeben (z. B. `rate({job="nginx"} |= "5xx" [5m])`)
4. Panel als **Logs** oder **Time series** anzeigen lassen

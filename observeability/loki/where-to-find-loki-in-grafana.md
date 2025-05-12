# Wo finde ich Loki in Grafana  
---

## 🔎 1. **Explore → Logs (Ad-hoc-Logsuche)**

### Schritte:

1. Links im Menü auf **"Explore"** klicken 🔍
2. Oben links im Dropdown die **Loki-Data Source** auswählen (z. B. `Loki`)
3. Du kannst jetzt:

   * Nach **Labels** filtern (`{job="my-app"}`)
   * Per LogQL Abfragen wie `|= "error"` verwenden
   * Live-Logs anzeigen (unten rechts: **Live** aktivieren)


# Ein Alert fuer ein Event aus Loki definieren 

## 🚨 1. **Alerting → Alerts mit LogQL (wenn unterstützt)**

Wenn du **Grafana Unified Alerting** nutzt (empfohlen), kannst du auf Loki-Log-Abfragen triggern:

1. Menü: **Alerting → Alert Rules**
2. Regel erstellen → **Loki als Data Source** wählen
3. Abfrage definieren, z. B.:

   ```
   rate({app="my-app"} |= "error" [5m]) > 0.1
   ```
4. Benachrichtigungen konfigurieren

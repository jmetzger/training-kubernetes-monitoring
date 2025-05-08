# Prometheus-Metriktypen (metric types) 

## Welche gibt es ? 

  * Counter
  * Gauge
  * Histogram
  * Summary

In der Prometheus-Dokumentation werden sie auch explizit als metric types bezeichnet. Wenn du also über „eine Gauge“ sprichst, meinst du korrekt:
„Eine Metrik vom Typ Gauge“

## 📊 1. **Counter**

Ein Counter ist ein **ständig wachsender Wert**. Er beginnt bei 0 und kann **nur steigen** (außer bei einem Reset, z. B. Pod-Restart).

**Beispiel:**

* `http_requests_total` – zählt, wie viele HTTP-Anfragen es gab
* `errors_total` – zählt Fehlermeldungen

> ➕ **Nur hochzählend!** Für „Rate“-Abfragen ideal.

🔍 **Typischer PromQL-Ausdruck:**

```promql
rate(http_requests_total[5m])
```

Zeigt, wie viele Anfragen pro Sekunde in den letzten 5 Minuten kamen.

---

## 📈 2. **Gauge**

Ein Gauge ist ein **aktueller Messwert**, der **steigen und sinken** kann – wie ein Thermometer.

**Beispiel:**

* `memory_usage_bytes` – aktueller Speicherverbrauch
* `cpu_temperature` – aktuelle CPU-Temperatur

> 📉 Ideal für Zustände wie Auslastung, offene Verbindungen etc.

🔍 **PromQL:**

```promql
memory_usage_bytes
```

zeigt den letzten bekannten Wert.

---

## ⏱️ 3. **Histogram**

Ein Histogram misst **Verteilungen von Werten**, z. B. Antwortzeiten. Es zählt, **wie viele Ereignisse in bestimmte Wertebereiche ("Buckets")** fallen.

**Beispiel:**

* `http_request_duration_seconds_bucket`

Diese Metrik ist gekoppelt mit:

* `_count` (Gesamtanzahl)
* `_sum` (Summe aller Werte)

> 📊 Sehr nützlich für Latenzen und Antwortzeitverteilungen.

🔍 **PromQL-Beispiel (90. Perzentil über 5 Minuten):**

```promql
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[5m]))
```

---

## 🔣 4. **Summary** (ähnlich wie Histogram, aber clientseitig berechnet)

Ein Summary enthält direkt **Perzentile**, allerdings:

* weniger aggregierbar über mehrere Instanzen
* erzeugt mehr Metriken
* eher selten verwendet in modernen Setups

**Beispiel:**

* `http_request_duration_seconds{quantile="0.9"}`

> ⚠️ Für verteilte Systeme nicht gut skalierbar → lieber Histogram verwenden!

---

## 🧠 **Zusammenfassung als Tabelle:**

| Typ       | Eigenschaften            | Beispiel                       | Ideal für...                  |
| --------- | ------------------------ | ------------------------------ | ----------------------------- |
| Counter   | nur steigend             | `http_requests_total`          | Events, Fehler, Anfragen      |
| Gauge     | auf- und absteigend      | `memory_usage_bytes`           | Zustände, Nutzung             |
| Histogram | Buckets + Summe/Count    | `*_bucket`, `*_sum`, `*_count` | Latenzverteilung, SLA-Analyse |
| Summary   | Clientseitige Perzentile | `*_quantile`                   | Einfache Latenzmetriken       |

---

Wenn du magst, kann ich dir ein kleines Demo-Projekt mit allen Typen in einer Beispiel-Go-App zeigen. Möchtest du das?

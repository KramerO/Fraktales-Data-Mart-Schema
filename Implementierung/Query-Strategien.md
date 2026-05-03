# Query-Strategien

#fdm #implementierung #queries

Die [[Operatoren]] des FDM lassen sich physisch auf drei Weisen umsetzen. Die Wahl hängt von Abfragefrequenz, Datenvolumen und Latenzanforderungen ab.

## On-the-fly

Dynamische Joins zur Laufzeit. Entspricht der direkten Ausführung des [[Operatoren#Fraktaler Join|fraktalen Joins]].

**Vorteile:**
- Maximale Flexibilität
- Kein zusätzlicher Speicherbedarf
- Immer aktuell

**Nachteile:**
- Hohe Latenz bei tiefer Rekursion
- Ressourcenintensiv bei großen Datenmengen

**Geeignet für:** Ad-hoc-Analysen, explorative Queries, niedrige Datenvolumina.

## Materialisierung

Vorab berechnete Aggregate als persistente Tabellen.

```sql
CREATE TABLE agg_customer_sales_behavior AS
SELECT
    s.customer_id,
    c.region,
    SUM(s.revenue) AS total_revenue,
    SUM(b.visits) AS total_visits
FROM fact_sales s
JOIN dim_customer c ON s.customer_id = c.customer_id
LEFT JOIN fact_customer_behavior b ON c.customer_id = b.customer_id
GROUP BY s.customer_id, c.region;
```

**Vorteile:**
- Schnelle Abfragen
- Entlastet die Quell-Tabellen

**Nachteile:**
- Zusätzlicher Speicher und Wartungsaufwand
- Risiko veralteter Daten ohne Refresh-Strategie

**Geeignet für:** Dashboards, regelmäßige Reports, hohe Datenvolumina.

## Hybrid

Die pragmatische Empfehlung: **häufige Pfade materialisieren, seltene dynamisch berechnen.**

### Entscheidungsmatrix

| Kriterium | → Materialisieren | → Hybrid (Einzelfallentscheidung) | → Dynamisch |
|-----------|------------------|----------------------------------|-------------|
| Abfragefrequenz | hoch (>10/Tag) | mittel (1–10/Tag) | niedrig (<1/Tag) |
| Datenvolumen | >1M Zeilen im Join | 100K–1M Zeilen | <100K Zeilen |
| Latenzanforderung | <1s | 1–10s akzeptabel | tolerant |
| Aktualisierungsfrequenz | täglich/stündlich | stündlich | Echtzeit nötig |

## Weiterführend

- [[Performance-Optimierung]] — technische Optimierungen
- [[dbt-Modellierung]] — Materialisierungsstrategien in dbt
- [[Operatoren]] — theoretische Grundlage

# Performance-Optimierung

#fdm #implementierung #performance

Rekursive Joins über mehrere Sternschema-Ebenen sind inhärent teurer als flache Abfragen. Diese Optimierungen reduzieren die Kosten.

## Denormalisierte Views

Für BI-Tools (vgl. [[BI-Kompatibilität]]) werden flache Views generiert, die den [[Operatoren#Fraktaler Join|fraktalen Join]] vorwegnehmen:

```sql
CREATE VIEW mart_sales_flat AS
SELECT
    s.sale_id,
    s.revenue,
    s.quantity,
    s.sale_date,
    c.region,
    c.segment,
    b.visits,
    b.conversions,
    b.channel
FROM fact_sales s
JOIN dim_customer c ON s.customer_id = c.customer_id
LEFT JOIN fact_customer_behavior b ON c.customer_id = b.customer_id;
```

## Partitionierung

Zeitbasierte Partitionierung auf Faktentabellen:

```sql
-- BigQuery-Syntax
CREATE TABLE fact_sales
PARTITION BY sale_date
AS SELECT ...;
```

**Regel:** Partitionierung nach der primären Zeitdimension jedes Facts.

## Clustering

Clustering nach Business Keys, die häufig in Joins verwendet werden:

```sql
-- BigQuery-Syntax
CREATE TABLE fact_sales
PARTITION BY sale_date
CLUSTER BY customer_id, product_id
AS SELECT ...;
```

## Aggregation Tables

Für häufig abgefragte Pfade (vgl. [[Query-Strategien#Hybrid|Hybrid-Strategie]]):

| Aggregation Table | Quelle | Granularität |
|-------------------|--------|-------------|
| `agg_daily_sales_by_region` | `fact_sales ⋈ dim_customer` | Tag × Region |
| `agg_monthly_behavior` | `fact_customer_behavior` | Monat × Kunde |

## Empfohlene Reihenfolge

1. **Partitionierung** — immer, kostenlos in modernen DWHs
2. **Clustering** — bei bekannten Join-Patterns
3. **Views** — für BI-Kompatibilität
4. **Aggregation Tables** — nur bei nachgewiesenem Performance-Bedarf

## Weiterführend

- [[Query-Strategien]] — wann materialisieren
- [[BI-Kompatibilität]] — Views für BI-Tools
- [[dbt-Modellierung]] — Materialisierungseinstellungen

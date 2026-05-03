# Physisches Design

#fdm #implementierung #sql

Die physische Abbildung des FDM auf relationale Datenbanken folgt einem einfachen Prinzip: **Jede Rekursionsebene wird als eigenständiges Sternschema implementiert.**

## Prinzip

```
Ebene 0:  fact_sales ─── dim_customer, dim_product, dim_date
Ebene 1:  fact_customer_behavior ─── dim_date  (channel als Degenerate Dimension)
Ebene 1:  fact_product_returns ─── dim_reason, dim_date
```

Die Verbindung zwischen Ebenen erfolgt über den **Business Key** der verknüpfenden Dimension.

## Schema-Beispiel

```sql
CREATE TABLE fact_sales (
    sale_id      BIGINT PRIMARY KEY,
    customer_id  BIGINT NOT NULL,
    product_id   BIGINT NOT NULL,
    revenue      DECIMAL(18,2),
    quantity     INT,
    sale_date    DATE
);

CREATE TABLE dim_customer (
    customer_id  BIGINT PRIMARY KEY,
    region       TEXT,
    segment      TEXT
);

CREATE TABLE fact_customer_behavior (
    behavior_id  BIGINT PRIMARY KEY,
    customer_id  BIGINT NOT NULL,
    visits       INT,
    conversions  INT,
    channel      TEXT,
    event_date   DATE
);
```

## Beispiel-Join

Der [[Operatoren#Fraktaler Join|fraktale Join]] wird physisch als mehrstufiger SQL-Join abgebildet:

```sql
SELECT
    s.revenue,
    c.region,
    b.visits
FROM fact_sales s
JOIN dim_customer c ON s.customer_id = c.customer_id
LEFT JOIN fact_customer_behavior b ON c.customer_id = b.customer_id;
```

`LEFT JOIN` ist hier bewusst gewählt: nicht jeder Kunde muss Verhaltensdaten haben. Die Wahl zwischen `INNER` und `LEFT JOIN` hängt von der Semantik der [[Dimensionstypen|Dimension]] ab.

## Namenskonventionen

| Prefix | Bedeutung |
|--------|-----------|
| `fact_` | Faktentabelle |
| `dim_` | Dimensionstabelle |
| `agg_` | Voraggregierte Tabelle (vgl. [[Performance-Optimierung]]) |
| `mart_` | Denormalisierte View (vgl. [[BI-Kompatibilität]]) |

## Weiterführend

- [[dbt-Modellierung]] — Umsetzung mit dbt
- [[Performance-Optimierung]] — Partitionierung, Clustering
- [[Query-Strategien]] — Materialisierung vs. dynamisch

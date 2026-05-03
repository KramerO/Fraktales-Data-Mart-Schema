# dbt-Modellierung

#fdm #implementierung #dbt

Die Umsetzung des FDM mit [dbt](https://www.getdbt.com/) nutzt die natürliche Schichtung von dbt-Modellen: **staging → intermediate → marts**.

## Projektstruktur

```
models/
├── staging/
│   ├── stg_sales.sql
│   └── stg_events.sql
├── intermediate/
│   └── int_customer_behavior.sql
└── marts/
    └── sales/
        ├── fact_sales.sql
        ├── dim_customer.sql
        └── fact_customer_behavior.sql
```

Jede Rekursionsebene des FDM liegt im `marts/`-Ordner. Die Zugehörigkeit zu einem Mart wird über Unterordner ausgedrückt.

## Beispiel-Modelle

### fact_sales.sql

```sql
SELECT
    sale_id,
    customer_id,
    product_id,
    revenue,
    quantity,
    sale_date
FROM {{ ref('stg_sales') }}
```

### fact_customer_behavior.sql

```sql
SELECT
    {{ dbt_utils.generate_surrogate_key(['customer_id', 'channel', 'event_date']) }} AS behavior_id,
    customer_id,
    COUNT(*) AS visits,
    SUM(CASE WHEN converted THEN 1 ELSE 0 END) AS conversions,
    channel,
    event_date
FROM {{ ref('stg_events') }}
GROUP BY customer_id, channel, event_date
```

## Materialisierung

| Modell-Typ | Empfohlene Materialisierung |
|-----------|----------------------------|
| `fact_*` | `table` oder `incremental` |
| `dim_*` | `table` |
| `agg_*` | `table` (vgl. [[Performance-Optimierung]]) |
| `mart_*` | `view` (Default); bei hoher Abfragefrequenz `table` — vgl. [[Query-Strategien#Hybrid\|Entscheidungsmatrix]] |

## Tests und Dokumentation

```yaml
# schema.yml
models:
  - name: fact_customer_behavior
    description: "Sub-Fact: Kundenverhalten auf Event-Ebene"
    columns:
      - name: customer_id
        tests:
          - not_null
          - relationships:
              to: ref('dim_customer')
              field: customer_id
```

Die `relationships`-Tests sichern die **referenzielle Integrität** zwischen Rekursionsebenen — ein kritischer Aspekt des [[Strukturregeln|Regelwerks]].

## Weiterführend

- [[Physisches Design]] — zugrunde liegendes Schema
- [[Query-Strategien]] — Abfragemuster
- [[Governance]] — Ownership pro Modell

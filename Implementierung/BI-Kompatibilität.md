# BI-Kompatibilität

#fdm #implementierung #bi

## Problem

BI-Tools (Looker, Tableau, Power BI, Metabase) erwarten **flache Modelle**: ein Fakt mit direkt verknüpften Dimensionen. Die rekursive Struktur des FDM ist für diese Tools nicht nativ navigierbar.

## Lösung: Generierte Views

Für jede relevante Kombination aus Fakt und expandierten Dimensionen wird eine flache View generiert:

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

Diese View entspricht dem Ergebnis von $\text{expand}(\text{dim\_customer})$ auf `fact_sales` — siehe [[Operatoren#Expansion|Expansion]].

## Namenskonvention

```
mart_{fact_name}_flat
mart_{fact_name}_{dimension_name}
```

Beispiele:
- `mart_sales_flat` — vollständig expandiert
- `mart_sales_customer` — nur Customer-Ebene expandiert

## Automatische View-Generierung

Statt alle Views manuell zu pflegen, werden sie aus der Schema-Definition (vgl. [[Nächste Schritte#Priorität 1 DSL zur Modellbeschreibung|DSL]]) oder der [[Governance#Schema Registry|Schema Registry]] generiert.

### Welche Pfade werden generiert?

Nicht jede kombinatorische Möglichkeit wird materialisiert. Stattdessen gilt ein **Whitelist-Ansatz:**

1. **Ebene-1-Expansion** (Default) — Jeder Fakt bekommt eine View, die alle direkten Dimensionen und deren Sub-Facts (1 Ebene tief) einschließt
2. **Annotierte Pfade** — Modellierer markieren in der DSL explizit, welche tieferen Expansionspfade als View generiert werden sollen
3. **Nutzungsgetrieben** — Query-Logs identifizieren häufig genutzte Pfade, die als View kandidieren (vgl. [[Query-Strategien#Hybrid|Hybrid-Strategie]])

### Kombinatorische Explosion vermeiden

Bei einem Fakt mit $n$ Dimensionen, von denen $k$ erweiterte oder komponierte Dimensionen sind, existieren theoretisch $2^k - 1$ nicht-triviale Expansionskombinationen.

**Begrenzungsstrategie:**

| Maßnahme | Wirkung |
|----------|---------|
| Ebene-1-Default | Eine kombinierte View pro Fakt (alle Ebene-1-Expansionen in einer View) |
| Whitelist statt Blacklist | Nur explizit gewünschte Pfade |
| Max Views pro Fakt | Hartes Limit (Default: 5), konfigurierbar |

### Namenskonvention bei tiefer Rekursion

```
mart_{root_fact}_{dim1}[_{dim2}]
```

| Tiefe | Beispiel | Name |
|-------|---------|------|
| 0 | `fact_sales` + primitive Dims | `mart_sales_flat` |
| 1 | + `fact_customer_behavior` | `mart_sales_customer` |
| 2 | + `fact_behavior_detail` | `mart_sales_customer_detail` |

**Regel:** Maximal 3 Segmente nach `mart_`. Bei tieferer Rekursion wird ein Alias in der DSL vergeben.

## Semantic Layer

Alternativ oder ergänzend zu Views: Definition im **Semantic Layer** (z.B. dbt Metrics, Looker LookML, Cube.js):

- Metriken mit [[Strukturregeln#Aggregierbarkeit|Aggregierbarkeitsklassifikation]]
- Dimensions-Hierarchien inkl. Sub-Fact-Beziehungen
- Automatische Join-Pfade

Dies ist der empfohlene Ansatz für Teams, die bereits einen Semantic Layer nutzen.

## Weiterführend

- [[Performance-Optimierung]] — View-Performance
- [[Query-Strategien]] — wann Views vs. dynamisch
- [[Governance]] — wer pflegt die Views

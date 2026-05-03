# Strukturregeln

#fdm #konzept #regeln

Die Strukturregeln definieren Invarianten, die jedes gültige FDM-Schema erfüllen muss. Sie verhindern fehlerhafte Rekursion und garantieren analytische Korrektheit.

## Depth-Limit

> **Pflicht-Constraint**: Jedes FDM-Schema muss ein explizites Depth-Limit $n_{\max}$ definieren.

$$\text{depth}(\text{Mart}(f)) \leq n_{\max}$$

**Default:** $n_{\max} = 3$

Dies verhindert unkontrollierte Rekursion, insbesondere wenn mehrere Teams unabhängig Dimensionen erweitern.

**Begründung:** Ohne hartes Limit ist Zyklusfreiheit in der Praxis schwer durchzusetzen. Ein optionales Limit verlagert die Verantwortung auf den Modellierer — bei domänenübergreifender Zusammenarbeit ein Risiko.

### Mart-spezifisches Limit

Das Depth-Limit kann **pro Mart** überschrieben werden, solange es das globale Maximum nicht übersteigt:

$$n_{\text{mart}}(f) \leq n_{\max}$$

Dies erlaubt einfachen Marts ein niedrigeres Limit (z.B. $n_{\text{mart}} = 2$), während komplexe Marts das volle Budget nutzen.

```yaml
# Beispiel in der DSL
global_depth_limit: 4

marts:
  - name: sales
    depth_limit: 3       # einfacher Mart, braucht nicht alle 4 Ebenen
  - name: supply_chain
    depth_limit: 4       # komplexe Wertschöpfungskette, nutzt volles Limit
  - name: reporting
    # kein Override → erbt global_depth_limit: 4
```

**Regel:** Überschreibung ist nur **nach unten** möglich. Kein Mart darf `global_depth_limit` überschreiten. Pro Dimension bleibt das Limit nicht überschreibbar — das verhindert, dass einzelne Teams die Komplexität unkontrolliert treiben.

## Zyklusfreiheit

Der gerichtete Graph $G = (D, E)$ mit $E = \{(d, d') \mid d' \in \text{dims}(f'),\; f' \in \text{subFacts}(d)\}$ muss **azyklisch** sein.

(Hilfsfunktionen $\text{dims}$ und $\text{subFacts}$ sind im [[Formales Metamodell#Hilfsfunktionen|Metamodell]] definiert.)

Zusammen mit dem Depth-Limit ergibt sich eine strikte DAG-Struktur (Directed Acyclic Graph).

## Granularität

Das FDM unterscheidet zwei Granularitätsbegriffe:

### Entity-Granularität

Die Aggregationsebene einer Entität. Wird pro Dimension als **Grain-Level** deklariert.

$$\text{entityGrain}: D \to \text{String}$$

Beispiele: `customer`, `segment`, `event`, `transaction`.

**Regel:** Sub-Facts müssen mindestens gleich oder feiner granular sein als ihre referenzierende Dimension:

$$\text{entityGrain}(f') \preceq \text{entityGrain}(d) \quad \text{für alle } f' \in \text{subFacts}(d)$$

Beispiel: Wenn `dim_customer` auf Kundenebene aggregiert, darf `fact_customer_behavior` nicht auf Segmentebene aggregiert sein — wohl aber auf Event-Ebene.

Die Ordnung $\preceq$ ist **domänenspezifisch** und wird in der Schema-Definition deklariert (z.B. `event ≺ customer ≺ segment`).

### Temporale Granularität

Die zeitliche Auflösung eines Fakts. Formal über $\text{temporalGrain}: F \to T$ definiert — siehe [[Formales Metamodell#Temporale Semantik]].

## Aggregierbarkeit

Metriken müssen explizit klassifiziert werden:

| Typ | Beschreibung | Beispiel |
|-----|-------------|---------|
| **additiv** | Über alle Dimensionen summierbar | Umsatz, Menge |
| **semi-additiv** | Nur über bestimmte Dimensionen summierbar | Kontostand (nicht über Zeit) |
| **nicht-additiv** | Nicht summierbar | Durchschnittspreis, Ratio |

### Propagation über Rekursionsstufen

Die Additivität einer Metrik kann sich über Sub-Facts verändern. Die folgende Algebra formalisiert das Verhalten.

#### Propagationsfunktion

Sei $\alpha: M \to \{\text{A}, \text{SA}, \text{NA}\}$ die Additivitätsklassifikation (Additiv, Semi-Additiv, Nicht-Additiv). Bei einem fraktalen Join $f \bowtie^* d$ gilt für eine Metrik $m$ aus einem $f' \in \text{subFacts}(d)$:

$$\alpha'(m) = \begin{cases} \text{A} & \text{wenn } \alpha(m) = \text{A} \land \text{joinType}(d, f') = \text{1:1} \\ \text{SA} & \text{wenn } \alpha(m) = \text{A} \land \text{joinType}(d, f') = \text{1:N} \\ \text{NA} & \text{wenn } \alpha(m) = \text{SA} \land \text{joinType}(d, f') = \text{1:N} \\ \alpha(m) & \text{sonst (SA bei 1:1 bleibt SA, NA bleibt NA)} \end{cases}$$

**Kernregel:** Additivität kann über Rekursionsstufen nur **gleich bleiben oder sinken**, nie steigen.

$$\text{A} \geq \text{SA} \geq \text{NA}$$

#### Join-Typ-Erkennung

Der Join-Typ zwischen Dimension und Sub-Fact bestimmt die Propagation:

| Join-Typ | Bedingung | Auswirkung |
|----------|-----------|------------|
| **1:1** | Sub-Fact hat gleiche Granularität wie Dimension | Additivität bleibt erhalten |
| **1:N** | Sub-Fact ist feiner granular (mehrere Zeilen pro BK) | Additiv → Semi-Additiv |

#### Beispiel

```
fact_sales (revenue: A)
  └── dim_customer (1:N zu fact_customer_behavior)
        └── fact_customer_behavior (visits: A)
```

- `revenue` bleibt **additiv** (eigene Metrik von `fact_sales`, nicht propagiert)
- `visits` wird im Kontext von `fact_sales` zu **semi-additiv**: ein Kunde hat N Behavior-Einträge, `SUM(visits)` über `fact_sales`-Zeilen würde Visits mehrfach zählen

#### Komponierte Dimensionen

Bei [[Dimensionstypen#Komponierte Dimension|komponierten Dimensionen]] mit mehreren Sub-Facts gilt die Propagation **pro Sub-Fact-Pfad unabhängig**. Die resultierende Additivität einer View, die mehrere Sub-Facts kombiniert, ist das **Minimum** über alle Pfade:

$$\alpha_{\text{view}}(m) = \min_{p \in \text{Pfade}} \alpha_p(m)$$

#### Signalisierung im Semantic Layer

| Situation | Signalisierung |
|-----------|---------------|
| Metrik wird herabgestuft (A → SA) | **Warnung** im Semantic Layer mit Angabe des verursachenden Join-Pfads |
| Nicht-additive Metrik in Aggregation verwendet | **Blockierung** — Abfrage wird abgelehnt oder erfordert explizite Aggregationsfunktion |
| Metrik über komponierten Pfad mit gemischter Additivität | **Warnung** + Anzeige des restriktivsten Pfads |

## Weiterführend

- [[Formales Metamodell]] — mathematische Grundlage
- [[Dimensionstypen]] — wie sich Regeln auf Typen auswirken
- [[Governance]] — organisatorische Durchsetzung

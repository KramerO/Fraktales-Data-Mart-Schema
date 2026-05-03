# Operatoren

#fdm #konzept #operatoren

Das FDM definiert drei Kernoperatoren, die auf der [[Formales Metamodell#Rekursive Definition eines Data Mart|rekursiven Mart-Struktur]] arbeiten.

## Expansion

Löst eine Dimension in ihre vollständigen Sub-Marts auf:

$$\text{expand}(d) = \bigcup_{f' \in \text{subFacts}(d)} \text{Mart}(f')$$

Für **primitive Dimensionen** ($\text{subFacts}(d) = \emptyset$) gilt:

$$\text{expand}(d) = \emptyset$$

Die Expansion ist die grundlegende Operation, um analytische Tiefe freizulegen. Sie wird durch das [[Strukturregeln#Depth-Limit|Depth-Limit]] begrenzt.

## Projektion

Schränkt einen Mart auf eine Teilmenge seiner Metriken ein:

$$\pi_{M'}(\text{Mart}(f)) \quad \text{mit } M' \subseteq \text{metrics}^*(\text{Mart}(f))$$

wobei $\text{metrics}^*$ alle Metriken des Marts einschließlich seiner Sub-Facts umfasst:

$$\text{metrics}^*(\text{Mart}(f)) = \text{metrics}(f) \cup \bigcup_{d \in \text{dims}(f)} \bigcup_{f' \in \text{subFacts}(d)} \text{metrics}^*(\text{Mart}(f'))$$

Die Projektion respektiert die [[Strukturregeln#Aggregierbarkeit|Additivitätsklassifikation]] — nicht-additive Metriken, die eine Aggregation erfordern würden, werden ausgeschlossen oder mit Warnung versehen.

## Fraktaler Join

Verbindet einen Fakt mit der expandierten Struktur einer seiner Dimensionen:

$$f \bowtie^* d = f \bowtie d \bowtie \text{expand}(d)$$

Für **primitive Dimensionen** ($\text{expand}(d) = \emptyset$) reduziert sich der fraktale Join zum regulären Dimension-Join:

$$f \bowtie^* d = f \bowtie d \quad \text{wenn } \text{subFacts}(d) = \emptyset$$

Der fraktale Join ist die zentrale Abfrageoperation im FDM. Er entspricht dem physischen Pattern eines mehrstufigen Joins über die Sternschema-Kette (vgl. [[Physisches Design#Beispiel-Join|Beispiel-Join]]).

### Beispiel

```
fact_sales ⋈* dim_customer
  = fact_sales ⋈ dim_customer ⋈ fact_customer_behavior
```

## Komposition

Operatoren lassen sich kombinieren:

$$\pi_{M'}(f \bowtie^* d)$$

Dies projiziert das Ergebnis eines fraktalen Joins auf eine Metrik-Teilmenge — die typische BI-Abfrage.

## Weiterführend

- [[Query-Strategien]] — physische Umsetzung der Operatoren
- [[BI-Kompatibilität]] — Abbildung auf flache BI-Modelle
- [[Formales Metamodell]] — theoretische Grundlage

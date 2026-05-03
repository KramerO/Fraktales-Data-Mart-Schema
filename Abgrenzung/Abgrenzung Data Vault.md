# Abgrenzung Data Vault

#fdm #abgrenzung #datavault

Data Vault 2.0 und FDM adressieren beide das Problem komplexer, sich verändernder analytischer Strukturen — aber mit grundlegend verschiedenen Philosophien.

## Strukturvergleich

| Aspekt | Data Vault | FDM |
|--------|-----------|-----|
| **Grundbausteine** | Hub, Link, Satellite | Fact, Dimension, Sub-Fact |
| **Rekursion** | Implizit über Link-Ketten | Explizit über `subFact`-Relation |
| **Historisierung** | Kernfeature (Satellites mit Load-Date) | Nicht eingebaut, orthogonal lösbar |
| **Granularität** | Einheitlich (Business Key-Ebene) | Variabel pro Ebene (vgl. [[Strukturregeln#Granularität]]) |
| **Schemaevolution** | Additiv (neue Satellites) | Additiv (neue Sub-Facts) |
| **Aggregation** | Nicht im Modell, gehört in Marts | Kernbestandteil ([[Strukturregeln#Aggregierbarkeit]]) |

## Philosophischer Unterschied

- **Data Vault** modelliert **Beziehungen zwischen Entitäten** (Hubs) über Links. Die analytische Struktur entsteht erst in der Mart-Schicht.
- **FDM** modelliert **analytische Zusammenhänge direkt**. Die Rekursion ist eine analytische Aussage: "Diese Dimension hat eigene, tiefere Metriken."

## Wann Data Vault, wann FDM?

### Data Vault bevorzugen, wenn:
- **Historisierung** ein Kernbedarf ist (Audit Trail, bitemporale Daten)
- Die Quellenlandschaft **heterogen und volatil** ist (viele Quellen, häufige Schema-Änderungen)
- Der Fokus auf **Raw Vault** liegt (Rohdaten erstmal landen, später analytisch aufbereiten)

### FDM bevorzugen, wenn:
- Die analytische Struktur **bereits klar** ist und modelliert werden soll
- **Mehrere analytische Ebenen** mit eigenen Metriken existieren
- Integration mit **BI-Tools** und **Semantic Layers** im Vordergrund steht (vgl. [[BI-Kompatibilität]])

## Kombination

Data Vault und FDM schließen sich nicht aus:

```
Raw Vault (Data Vault)
  → Business Vault (Data Vault)
    → Fraktale Marts (FDM)
```

Data Vault als Integrationsschicht, FDM als analytische Präsentationsschicht. Dies kombiniert die Stärken beider Ansätze.

## Weiterführend

- [[Abgrenzung Activity Schema]] — weiterer Vergleich
- [[Zielsetzung]] — Positionierung des FDM
- [[Einsatzszenarien]] — Entscheidungshilfe

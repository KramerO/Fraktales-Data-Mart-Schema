# Erweiterungen

#fdm #einsatz #erweiterungen

Mögliche Erweiterungen des FDM über das relationale Kernmodell hinaus.

## Graph-basierte Implementierung

Die rekursive Struktur des FDM lässt sich nativ als **Property Graph** abbilden:

- **Knoten:** Fakten, Dimensionen
- **Kanten:** `hasDim`, `hasMetric`, `subFact`
- **Traversal:** Graph-Queries statt rekursive SQL-Joins

Vorteile gegenüber relationaler Implementierung:
- Natürliche Navigation der Rekursion
- Effiziente Pfadabfragen
- Schema-Flexibilität

Technologien: Neo4j, Amazon Neptune, Apache TinkerPop

## API-basierter Zugriff

Ein **GraphQL-Layer** über dem FDM ermöglicht:

```graphql
query {
  factSales {
    revenue
    customer {
      region
      behavior {
        visits
        conversions
      }
    }
  }
}
```

Die verschachtelte GraphQL-Struktur bildet die fraktale Rekursion direkt ab. Die [[Strukturregeln#Depth-Limit|Depth-Limit]]-Regel wird als max query depth im GraphQL-Server durchgesetzt.

## Integration mit Feature Stores

Für ML-Pipelines: Sub-Facts als **Feature Groups** im Feature Store registrieren.

| FDM-Konzept | Feature Store |
|-------------|--------------|
| Sub-Fact | Feature Group |
| Business Key | Entity Key |
| Metrik | Feature |
| Granularität | Feature Freshness |

Technologien: Feast, Tecton, SageMaker Feature Store

## DSL zur Modellbeschreibung

Eine domänenspezifische Sprache zur deklarativen Definition von FDM-Schemas — siehe [[Nächste Schritte]].

## Weiterführend

- [[Nächste Schritte]] — Priorisierung der Erweiterungen
- [[Physisches Design]] — aktuelles relationales Design
- [[Formales Metamodell]] — theoretische Basis für Erweiterungen

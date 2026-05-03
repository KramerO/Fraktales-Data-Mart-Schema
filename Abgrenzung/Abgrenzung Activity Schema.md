# Abgrenzung Activity Schema

#fdm #abgrenzung #activityschema

Das Activity Schema (nach Ahmed Elsamadisi) verfolgt einen radikal anderen Ansatz als sowohl Sternschema als auch FDM: **ein einziges, breites Ereignis-Modell** statt strukturierter Fakten und Dimensionen.

## Strukturvergleich

| Aspekt | Activity Schema | FDM |
|--------|----------------|-----|
| **Grundbaustein** | Activity (Entität + Zeitstempel + Aktivitätstyp) | Fact + Dimension + Sub-Fact |
| **Struktur** | Flach, eine Tabelle | Rekursiv, verschachtelte Sternschemas |
| **Modellierung** | Event-zentriert | Fakt-zentriert |
| **Joins** | Selbst-Joins (Activity ⋈ Activity) | Fraktale Joins ([[Operatoren#Fraktaler Join]]) |
| **Granularität** | Immer Event-Ebene | Variabel pro Ebene |
| **Schema-Komplexität** | Minimal (1-2 Tabellen) | Proportional zur analytischen Tiefe |

## Philosophischer Unterschied

- **Activity Schema** sagt: "Alles ist ein Event. Struktur entsteht zur Abfragezeit."
- **FDM** sagt: "Analytische Struktur existiert und soll explizit modelliert werden."

Activity Schema optimiert für **Flexibilität und Einfachheit**, FDM für **analytische Ausdruckskraft und Governance**.

## Wann Activity Schema, wann FDM?

### Activity Schema bevorzugen, wenn:
- Die Domäne **event-getrieben** ist (Clickstreams, IoT, Logs)
- Das Team **klein** ist und wenig Governance-Overhead will
- **Schnelle Iteration** wichtiger ist als formale Modellierung
- Die Analysen primär **zeitbasierte Sequenzen** betreffen (Funnels, Journeys)

### FDM bevorzugen, wenn:
- **Verschiedene analytische Ebenen** mit eigenen Metriken und Aggregationsregeln existieren
- **Domänenübergreifende Teams** zusammenarbeiten (Governance ist kritisch)
- Die Analysen über **strukturierte Beziehungen** laufen, nicht über Zeitsequenzen
- [[Strukturregeln#Aggregierbarkeit|Aggregierbarkeit]] explizit kontrolliert werden muss

## Kombination

Activity Schema kann als **Eingabe** für FDM dienen:

```
Activity Stream (Activity Schema)
  → Aggregation + Strukturierung
    → Fraktale Marts (FDM)
```

Events werden zu strukturierten Sub-Facts aggregiert. Dies ist ein natürliches Pattern für Organisationen, die rohe Events sammeln und analytisch aufbereiten.

## Weiterführend

- [[Abgrenzung Data Vault]] — weiterer Vergleich
- [[Einsatzszenarien]] — Entscheidungshilfe
- [[Zielsetzung]] — Positionierung des FDM

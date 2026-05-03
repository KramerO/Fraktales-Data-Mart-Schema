# Fraktales Data-Mart-Schema (FDM)

> Rekursives Datenmodell zur strukturierten Abbildung komplexer analytischer Domänen.

Das **Fraktale Data-Mart-Schema (FDM)** ist ein Modellierungsansatz für analytische Datenarchitekturen, der klassische Star-/Snowflake-Schemata um ein **rekursives, selbstähnliches Strukturprinzip** erweitert. Ziel ist es, fachliche Komplexität konsistent und beliebig tief zu modellieren — ohne Verlust von Performance, Governance oder BI-Kompatibilität.

Dieses Repository ist ein **Obsidian-Vault** mit Konzept, formalem Metamodell, Implementierungs­leitfaden, Governance-Regeln und Abgrenzung zu verwandten Ansätzen (Data Vault, Activity Schema).

---

## Einstieg

Hauptnavigation: **[`00 - Index.md`](./00%20-%20Index.md)** — Map of Content für den gesamten Vault.

## Vault-Struktur

| Ordner | Inhalt |
|--------|--------|
| **`Konzept/`** | Zielsetzung, formales Metamodell, Strukturregeln, Dimensionstypen, Operatoren |
| **`Implementierung/`** | Physisches Design, dbt-Modellierung, Query-Strategien, Performance, BI-Kompatibilität |
| **`Governance/`** | Regeln, Versionierung, Konfliktauflösung |
| **`Einsatz/`** | Einsatzszenarien und Erweiterungen |
| **`Abgrenzung/`** | Vergleich zu Data Vault und Activity Schema |

## Nutzung

### Mit Obsidian (empfohlen)

1. [Obsidian](https://obsidian.md) installieren
2. Repository klonen:
   ```bash
   git clone https://github.com/KramerO/Fraktales-Data-Mart-Schema.git
   ```
3. In Obsidian: *Open folder as vault* → den geklonten Ordner auswählen
4. Mit `00 - Index.md` starten — Wiki-Links (`[[...]]`) navigieren durch den Vault

### Ohne Obsidian

Alle Inhalte sind reines Markdown. Wiki-Links werden auf GitHub nicht aufgelöst, aber alle Dateien sind über die Ordnerstruktur erreichbar.

## Status

Aktiver Entwurf. Konzept und Metamodell sind stabil; DSL-Spezifikation, Query-Engine und Referenz­implementierung sind in Arbeit (siehe `Roadmap` im Index).

## Beitrag

Issues und Pull Requests sind willkommen — insbesondere zu:

- Anwendungsfällen und Einsatzszenarien
- Abgrenzung zu weiteren Modellierungs­ansätzen
- Implementierungs­erfahrungen mit dbt / Query-Engines

## Lizenz

[Apache License 2.0](./LICENSE) — die Inhalte dürfen frei verwendet, modifiziert und weitergegeben werden, sofern die Lizenz­bedingungen eingehalten werden.

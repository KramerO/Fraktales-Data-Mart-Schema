# Zielsetzung

#fdm #konzept

Das Fraktale Data-Mart-Schema (FDM) erweitert klassische Sternschema-Ansätze um **Selbstähnlichkeit** und **Komponierbarkeit**. Es ist ein rekursives Datenmodell zur strukturierten Abbildung komplexer analytischer Domänen.

## Kernziele

1. **Rekursive Modellierung** — Analytische Zusammenhänge lassen sich auf jeder Ebene als eigenständiges Sternschema abbilden. Jede Dimension kann optional einen eigenen Fakt tragen.
2. **Skalierbarkeit ohne strukturellen Bruch** — Neue analytische Tiefe entsteht durch Komposition, nicht durch Schema-Migration.
3. **Domänengetriebene Datenarchitektur** — Jede Domäne besitzt und pflegt ihre eigenen Sub-Facts (vgl. [[Governance]]).
4. **Plattformintegration** — Das Modell ist unabhängig von spezifischen Technologien implementierbar (vgl. [[Physisches Design]], [[dbt-Modellierung]]).

## Abgrenzung

FDM ist **kein Ersatz** für das klassische Sternschema, sondern eine Erweiterung für Szenarien, in denen flache Modelle an ihre Grenzen stoßen. Für einfache Dashboards und statische Reports bleibt das Sternschema die bessere Wahl (vgl. [[Einsatzszenarien]]).

Zur Positionierung gegenüber anderen Ansätzen siehe [[Abgrenzung Data Vault]] und [[Abgrenzung Activity Schema]].

## Weiterführend

- [[Formales Metamodell]] — mathematische Fundierung
- [[Strukturregeln]] — Invarianten und Constraints
- [[00 - Index|Index]]

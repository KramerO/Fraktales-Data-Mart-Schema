# Governance

#fdm #governance

Governance im FDM adressiert drei Dimensionen: Ownership, Versionierung und semantische Konsistenz.

## Ownership

Jede Dimension und jeder Sub-Fact gehört einer **Domäne**. Die Domäne ist verantwortlich für:

- Schema-Definition und -Evolution
- Datenqualität
- SLA-Einhaltung (Freshness, Completeness)

**Regel:** Kein Sub-Fact darf ohne expliziten Owner existieren. Dies ist besonders kritisch bei [[Dimensionstypen#Komponierte Dimension|komponierten Dimensionen]], deren Sub-Facts aus verschiedenen Domänen stammen können.

### Ownership-Matrix

| Ebene | Owner | Verantwortung |
|-------|-------|---------------|
| `fact_sales` | Sales Analytics | Schema, Metriken |
| `dim_customer` | Customer Domain | Attribute, BK |
| `fact_customer_behavior` | Customer Domain | Sub-Fact, Granularität |

## Versionierung

Schema-Versionen werden **pro Substruktur** gepflegt — nicht global. Dies erlaubt unabhängige Evolution der Rekursionsebenen.

```
fact_sales          v3.1
├── dim_customer    v2.0
│   └── fact_customer_behavior  v1.4
└── dim_product     v5.2
    └── fact_product_returns    v1.0
```

**Kompatibilitätsregel:** Ein Sub-Fact-Upgrade darf die Schnittstelle zur übergeordneten Dimension nicht brechen (Business Key muss stabil bleiben).

## Semantische Konsistenz

Metriken werden **zentral** definiert, auch wenn sie in verschiedenen Sub-Facts vorkommen.

- Ein Semantic Layer (vgl. [[BI-Kompatibilität#Semantic Layer|Semantic Layer]]) dient als Single Source of Truth
- Gleiche Metrik-Namen in verschiedenen Sub-Facts müssen identisch definiert sein
- [[Strukturregeln#Aggregierbarkeit|Additivitätsklassifikation]] (A/SA/NA) wird zentral gepflegt

## Konflikterkennung bei Multi-Team-Evolution

Wenn mehrere Teams unabhängig Sub-Facts erweitern, können Konflikte entstehen. Das FDM adressiert dies über eine **Schema Registry**, eine **Konflikterkennung** und einen **Breaking Change Workflow**.

### Schema Registry

Jeder Sub-Fact wird in einer zentralen **Schema Registry** registriert — analog zu Event-Schema-Registries (z.B. Confluent Schema Registry). Die Registry enthält:

- Fakt-Name, Owner, Version
- Business Keys und deren Typen
- Metriken mit [[Strukturregeln#Aggregierbarkeit|Additivitätsklassifikation]]
- Temporale Granularität (vgl. [[Formales Metamodell#Temporale Semantik]])

**Technisch:** Die Registry kann als dbt-`schema.yml`-Dateien, als zentrales YAML-Repository, oder als dedizierter Service implementiert werden.

### Konflikttypen und Erkennung

| Konflikttyp | Erkennung | Auflösung |
|-------------|-----------|-----------|
| **Metrik-Kollision** — gleicher Name, unterschiedliche Definition | Registry-Validierung bei Registration | Umbenennen oder zentrale Abstimmung |
| **BK-Inkompatibilität** — Sub-Fact referenziert geänderten Business Key | CI/CD-Check gegen Registry | Breaking Change blockieren, Migration erzwingen |
| **Granularitätsverletzung** — Sub-Fact gröber als Dimension | Automatische Prüfung gegen [[Strukturregeln#Granularität]] | Registration ablehnen |
| **Depth-Limit-Überschreitung** — neuer Sub-Fact überschreitet $n_{\max}$ | Graph-Analyse bei Registration | Registration ablehnen |
| **Temporale Inkompatibilität** — Sub-Fact hat gröbere temporale Granularität als übergeordneter Fakt | Prüfung gegen [[Formales Metamodell#Kompatibilitätsregel\|temporale Kompatibilitätsregel]] | Registration ablehnen |

### Breaking Change Propagation

Ein Breaking Change in einem Sub-Fact wird **nach oben propagiert**:

```
fact_product_returns v1.0 → v2.0 (BK geändert)
  ↑ bricht
dim_product v5.2
  ↑ erfordert Update
fact_sales v3.1
```

**Regel:** Kein Breaking Change darf ohne explizite Zustimmung aller Upstream-Owner deployed werden. Die Registry erzwingt dies über einen **Approval-Workflow**:

1. Team meldet Breaking Change an
2. Registry identifiziert alle betroffenen Upstream-Pfade
3. Upstream-Owner erhalten Notification und müssen approven
4. Erst nach vollständiger Approval wird die neue Version registriert

## Integration mit Data Mesh

Das FDM ist natürlich kompatibel mit Data Mesh:

- **Domain Ownership** → Ownership pro Sub-Fact
- **Data as a Product** → jeder Sub-Fact als Datenprodukt
- **Self-serve Platform** → [[dbt-Modellierung]] als Modellierungsstandard innerhalb der Plattform
- **Federated Governance** → zentrale Metriken, dezentrale Schemas

Siehe [[Einsatzszenarien]] für weitere Kontext.

## Weiterführend

- [[Strukturregeln]] — technische Constraints
- [[dbt-Modellierung]] — Tests und Dokumentation
- [[Einsatzszenarien]] — wo Governance besonders wichtig ist

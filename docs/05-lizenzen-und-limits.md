# Lizenzen & Nutzungslimits

Diese Recherche wurde separat vom ursprünglichen Deep-Research-Durchlauf durchgeführt (Web-Suche, nicht adversarial verifiziert) — Stand: Juli 2026. Bei sicherheits- oder budgetkritischen Entscheidungen die verlinkten Quellen selbst gegenprüfen.

## Übersicht

| Komponente | Lizenz | Kosten Self-Hosting | Nutzungslimit in der freien Variante |
|---|---|---|---|
| **Matrix Synapse** (roher Homeserver, eigenes Deployment) | Apache 2.0 | keine | keines — technisch nicht limitiert |
| **Element Server Suite Community** (Elements eigenes Bundle: Synapse + Element + Tooling/Helm-Chart) | AGPLv3 | keine | ausgelegt für **1–100 Nutzer**; darüber verweist Element auf die kostenpflichtige „Synapse Pro" / ESS Pro |
| **Authentik** (Core) | AGPLv3 | keine | **kein Nutzerlimit**; Enterprise-Tarif (ab $5/internem Nutzer/Monat) bringt nur Zusatzfeatures (Remote Access/RAC, KI-Risk-Detection, SLA-Support, erweiterte Audit-Logs), keine Freischaltungssperre für Kernfunktionen (SSO, MFA, LDAP, SAML, OIDC) |
| **MISP** | AGPLv3 | keine | kein Limit |
| **MinIO** | AGPLv3 | ⚠️ Projekt seit Dez. 2025 in Maintenance Mode | Community Edition verlor im Mai 2025 die Admin-Konsole (IAM/Policy/Replikation nur noch Enterprise) — **nicht mehr für neue Projekte empfohlen** |
| **AWS S3 / KMS** | kein Software-Lizenzmodell — AWS-Managed-Service | nutzungsbasiert (Storage-GB, API-Calls, KMS-Key-Nutzung) | kein "Freemium"-Limit im klassischen Sinn, reine Verbrauchsabrechnung |

## Einordnung für 10-50 Nutzer

Bei der in dieser Dokumentation angenommenen Teamgröße (10-50 Nutzer) bewegt ihr euch **komfortabel innerhalb** der kostenlosen Grenzen aller empfohlenen Komponenten:

- Roher Synapse-Homeserver (Docker-Compose-Deployment, wie in [03-poc-anleitung.md](03-poc-anleitung.md) beschrieben): kein lizenzbedingtes Limit, unabhängig von der Nutzerzahl.
- Element Server Suite Community: 10-50 Nutzer liegen deutlich unter der 100-Nutzer-Grenze.
- Authentik Community: kein Limit, unabhängig von der Nutzerzahl.
- MISP: kein Limit.

## MinIO — warum von der ursprünglichen Empfehlung gestrichen

Die ursprüngliche Architektur-Empfehlung in [02-architektur.md](02-architektur.md) nannte MinIO als mögliche Alternative zu AWS S3 für self-hosted Object-Storage. Diese Einschätzung ist überholt:

- Mai 2025: MinIO entfernt die Web-Admin-Konsole aus der Community Edition — zentrale Verwaltungsfunktionen (Policy-Management, Replikations-Konfiguration, Lifecycle-Management) wandern vollständig ins kommerzielle Enterprise-Produkt.
- Dezember 2025: MinIO setzt das Open-Source-Projekt offiziell in „Maintenance Mode" — keine neuen Features, Issues/PRs werden nicht mehr regulär bearbeitet, selbst kritische Sicherheits-Fixes nur noch „nach Ermessen".

Für Organisationen mit bestehendem AWS-Account (wie in diesem Projekt) ist die pragmatische Konsequenz: natives AWS S3 + KMS nutzen statt eine faktisch aufgegebene Software selbst zu betreiben. Für einen providerunabhängigen self-hosted Object-Storage-Bedarf wären aktuell Ceph oder SeaweedFS die gängigen Alternativen — beide nicht Teil dieser Recherche, vor Einsatz eigenständig prüfen.

## Quellen

- [Element Server Suite Community](https://element.io/en/server-suite/community)
- [Scaling to millions of users requires Synapse Pro (Element-Blog)](https://element.io/blog/scaling-to-millions-of-users-requires-synapse-pro/)
- [Synapse Pro | Enterprise Matrix Server (Element)](https://element.io/en/server-suite/synapse-pro)
- [authentik Pricing](https://goauthentik.io/pricing/)
- [authentik Enterprise Dokumentation](https://docs.goauthentik.io/enterprise/)
- [MinIO License (GitHub)](https://github.com/minio/minio/blob/master/LICENSE)
- [MinIO users complain after admin UI removed from Community Edition (Blocks & Files, Juni 2025)](https://www.blocksandfiles.com/ai-ml/2025/06/19/minio-users-complain-after-admin-ui-removed-from-community-edition/1610856)
- [MinIO in Maintenance Mode: Open Source Alternatives (Dezember 2025)](https://bizety.com/2025/12/06/minio-in-maintenance-mode-open-source-alternatives/)
- [MinIO's Community Edition: The End? (Dezember 2025)](https://pepitedata.com/2025/12/15/minio-community-edition-end-agpl/)

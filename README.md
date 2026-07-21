# Ice-IR-Plattform

Dokumentation zu einer sicheren, 2FA-geschützten Kommunikations- und Datenaustausch-Plattform für Teilnehmer einer dauerhaften Incident-Response/CERT-Struktur (10-50 Nutzer). Recherche-, Architektur- und POC-Dokumentation — kein Betriebs-Repo, keine Zugangsdaten.

## Ausgangsfrage

Kann eine sichere 2FA-Plattform für Chat-, Nachrichten- und Datenaustausch während eines Incident-Response-Einsatzes self-hosted auf AWS EC2 gebaut werden — mit dem zwingenden Nebenziel, dass sie **out-of-band** läuft (also unabhängig von einer möglicherweise kompromittierten Firmen-/Vereinsinfrastruktur) und DSGVO-/EU-Datenresidenz-konform ist?

## TL;DR

- Eine **einzelne EC2-Instanz reicht nicht** für Out-of-Band-IR-Betrieb. AWS selbst empfiehlt für Forensik/IR-Arbeit einen **komplett separaten, isolierten Account + eigene VPC** — siehe [02-architektur.md](docs/02-architektur.md).
- Empfohlener Software-Stack: **Matrix/Synapse + Element** (E2EE, verifizierbares Cross-Signing) hinter **Authentik als OIDC-Identity-Provider** (SSO + 2FA/FIDO2), **MISP** für strukturierten IOC-Austausch mit TLP-2.0-Tagging, **AWS S3 + KMS** für verschlüsselte Forensik-Dateien.
- Alle empfohlenen Kernkomponenten sind Open Source ohne Lizenzkosten und decken 10-50 Nutzer komfortabel im kostenlosen Rahmen ab (Details inkl. Nutzungslimits: [05-lizenzen-und-limits.md](docs/05-lizenzen-und-limits.md)). **MinIO wird nicht mehr empfohlen** — das Projekt ist seit Dezember 2025 in Maintenance Mode.
- Für den Dauerbetrieb ist eine Alternative zu AWS (z.B. ein separates Hetzner-Cloud-Projekt in der EU) meist deutlich günstiger bei gleicher DSGVO-Konformität — AWS lohnt sich vor allem, wenn bereits ein dedizierter Out-of-Band-Account existiert.
- Ein Proof-of-Concept lässt sich mit überschaubarem Aufwand auf vorhandener Proxmox-Infrastruktur hinter einer bestehenden Authentik-Instanz aufbauen, siehe [03-poc-anleitung.md](docs/03-poc-anleitung.md).
- **Update:** Der POC läuft produktiv, SSO-only (kein Passwort-Fallback mehr), extern erreichbar über eine eigene Subdomain mit Let's-Encrypt-TLS. Migration auf Matrix Authentication Service (MAS) abgeschlossen, inkl. QR-Code-Geräteverknüpfung — siehe [06-mas-migration-und-qr-login.md](docs/06-mas-migration-und-qr-login.md).
- **Update:** Security-Audit durchgeführt (Versionsabgleich + Live-Konfigurationsprüfung), mehrere Findings identifiziert und gehärtet — siehe [07-security-audit-und-haertung.md](docs/07-security-audit-und-haertung.md).

## Inhalt

| Dokument | Inhalt |
|---|---|
| [docs/01-analyse-und-quellen.md](docs/01-analyse-und-quellen.md) | Recherche-Analyse: was belegt ist (mit Quellen-URLs), was widerlegt/unbelegt blieb |
| [docs/02-architektur.md](docs/02-architektur.md) | Architektur-Empfehlung, Mermaid-Diagramme, AWS vs. Alternativen, Kosten |
| [docs/03-poc-anleitung.md](docs/03-poc-anleitung.md) | Schritt-für-Schritt-Anleitung für einen Proof-of-Concept |
| [docs/04-schlussfolgerungen.md](docs/04-schlussfolgerungen.md) | Schlussfolgerungen, Risiken/Fallstricke, offene Punkte für Nachrecherche |
| [docs/05-lizenzen-und-limits.md](docs/05-lizenzen-und-limits.md) | Lizenzkosten und Nutzungslimits je Komponente, inkl. MinIO-Statusänderung Dez. 2025 |
| [docs/06-mas-migration-und-qr-login.md](docs/06-mas-migration-und-qr-login.md) | Update: Migration auf Matrix Authentication Service (MAS), QR-Code-Login, SSO-only-Umstellung |
| [docs/07-security-audit-und-haertung.md](docs/07-security-audit-und-haertung.md) | Security-Audit: Versionsabgleich, Konfigurations-Findings, Härtungsmaßnahmen, Lessons Learned |

## Methodik

Die Recherche wurde mit einem mehrstufigen Multi-Agent-Verfahren durchgeführt: Themenzerlegung in 5 Suchrichtungen → parallele Web-Recherche → Extraktion falsifizierbarer Einzelaussagen aus 19 Quellen → adversariale 3-Stimmen-Verifikation jeder Aussage (mind. 2 von 3 Gegenstimmen widerlegen eine Aussage) → Synthese. Von 25 tiefenverifizierten Kernaussagen wurden 17 bestätigt und 8 explizit widerlegt — die widerlegten Aussagen sind bewusst dokumentiert, um zu zeigen, welche plausibel klingenden Annahmen sich NICHT bestätigen ließen (siehe [01-analyse-und-quellen.md](docs/01-analyse-und-quellen.md)).

## Interaktive Fassung

Die ursprüngliche, interaktive Fassung dieses Berichts (inkl. Mermaid-Diagrammen) liegt als privates Claude-Artefakt vor: https://claude.ai/code/artifact/d6c0a54e-fa67-49ea-be17-e04856880741 — Zugriff nur für den Ersteller/nach expliziter Freigabe. Die hier im Repo abgelegten Dokumente sind eine bereinigte, für Veröffentlichung geeignete Fassung ohne interne Infrastruktur-Details.

## Hinweis

Dieses Repository enthält ausschließlich Analyse-/Architektur-Dokumentation. Keine Zugangsdaten, Tokens, Zertifikate, internen Hostnamen/IPs oder sonstigen sensiblen Betriebsdaten werden hier veröffentlicht.

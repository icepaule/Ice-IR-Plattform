# Analyse & Quellen

Recherche-Grundlage: 101 Subagenten, 5 parallele Suchrichtungen, 19 ausgewertete Quellen, 87 extrahierte Einzelaussagen, davon 25 tiefenverifiziert (adversariale 3-Stimmen-Prüfung, mind. 2/3 Gegenstimmen widerlegen eine Aussage). Ergebnis: 17 bestätigt, 8 widerlegt, 0 unentschieden.

## 1. Bestätigte Kernaussagen (mit Quellen)

### Out-of-Band-Architektur — AWS-eigene Best Practice

> "AWS empfiehlt einen separaten, dedizierten AWS-Account (in AWS Organizations, eigene Forensik-OU) für Incident-/Forensik-Arbeit — vollständig isoliert vom Rest der Organisation, um Kontaminationsrisiko zu vermeiden. Die zugehörige VPC soll restriktiv, isoliert und auditierbar sein, ohne geteilte Workloads."

- **Quelle:** [AWS Security Blog — Forensic Investigation Environment Strategies in the AWS Cloud](https://aws.amazon.com/blogs/security/forensic-investigation-environment-strategies-in-the-aws-cloud/)
- **Verifikation:** 3-0 (einstimmig bestätigt)
- **Bedeutung:** Beantwortet die Ausgangsfrage direkt — eine einzelne EC2-Instanz im bestehenden AWS-Account ist NICHT das, was AWS selbst für Out-of-Band-IR empfiehlt. Produktivbetrieb braucht Konto-Ebenen-Isolation.

### Sicherer Datenzugriff — AWS-eigene Best Practice

> Zugriff auf forensische Daten sollte über kurzlebige STS-Temporary-Credentials (Standard 1h, AssumeRole + Session-Policies) statt statischer IAM-User-Keys erfolgen. S3-Storage: Customer-Managed KMS Keys (CMK) für Encryption-at-Rest, TLS-Erzwingung via Bucket-Policy (`aws:SecureTransport`). AWS: "using a static IAM user secret access key isn't a best practice" für diesen Anwendungsfall.

- **Quelle:** [AWS Security Blog — A framework for securely collecting forensic artifacts into S3 buckets](https://aws.amazon.com/blogs/security/a-framework-for-securely-collecting-forensic-artifacts-into-s3-buckets/)
- **Verifikation:** 3-0 (zwei Einzelaussagen, je einstimmig bestätigt)

### Matrix E2EE — verifizierbares Vertrauensmodell

> Matrix implementiert eine Drei-Schlüssel-Cross-Signing-Architektur (Master-Key, Self-Signing-Key, User-Signing-Key): Ein Nutzer wird einmal verifiziert, danach werden automatisch alle von diesem Nutzer signierten Geräte vertraut — statt paarweiser Geräteverifikation.

- **Quelle:** [Matrix.org Docs — Cross-Signing](https://matrix.org/docs/older/e2ee-cross-signing/)
- **Verifikation:** 3-0, zusätzlich durch JS-/iOS-/Android-SDK-Dokumentation und die ursprüngliche MSC1680-Proposal korroboriert
- **Bedeutung:** Relevant für die Anforderung "E2EE-Messaging mit verifizierbarer Identität" — für ein IR-Team ist nachvollziehbare, auditierbare Geräteverifikation neuer Mitglieder wichtig.

### MISP — strukturierter IOC-Austausch

> MISP ist eine ausgereifte, aktiv gepflegte Open-Source-Threat-Intelligence-Plattform für strukturierte IOC-Speicherung, automatische Korrelation und Export/Sync in Standardformaten (STIX, OpenIOC) zu IDS/SIEM und weiteren MISP-Instanzen.

- **Quellen:** [misp-project.org](https://www.misp-project.org/), [misp-project.org/features](https://www.misp-project.org/features/)
- **Verifikation:** 3-0, korroboriert durch GitHub, Wikipedia, CIRCL

> MISPs eigene Compliance-Dokumentation begründet rechtmäßigen CSIRT-zu-CSIRT-Informationsaustausch mit Recital 49 der DSGVO (Interesse an Netz- und Informationssicherheit); in einem Peer-to-Peer-MISP-Sharing-Netzwerk gilt jeder Teilnehmer als eigenständig verantwortlicher Data Controller, es gibt keinen zentralen Controller.

- **Quelle:** [misp-project.org/compliance/GDPR](https://www.misp-project.org/compliance/GDPR/)
- **Verifikation:** gemischt — "Peer als eigenständiger Controller" 3-0 bestätigt; "Recital 49 als Rechtsgrundlage" nur 2-1 (mittlere Konfidenz)
- **⚠️ Rechts-Nuance:** Ein Recital (Erwägungsgrund) der DSGVO ist Auslegungshilfe, keine eigenständig bindende Rechtsgrundlage. Die eigentliche Rechtsgrundlage für das Sharing ist Art. 6(1)(f) DSGVO (berechtigtes Interesse), Recital 49 stützt diese Auslegung. **Vor produktivem Einsatz mit echten Personenbezugsdaten: juristische Prüfung einholen.**

### TLP 2.0 — Sensitivitäts-Label, KEIN Verschlüsselungsstandard

> TLP (Traffic Light Protocol) ist der De-facto-internationale Standard für Sharing-Sensitivitäts-Kennzeichnung (TLP:RED/AMBER/GREEN/CLEAR), verwendet von CSIRTs, PSIRTs, ISACs und Regierungsbehörden. Formal verwaltet und versioniert durch die TLP Special Interest Group (TLP-SIG) von FIRST.org. TLP 1.0 (Aug. 2016) → TLP 2.0 (Aug. 2022, aktuell). **TLP ist explizit KEIN Verschlüsselungs- oder technischer Handhabungsstandard** — es kennzeichnet nur die Sensitivität; E2EE/Zugriffskontrolle müssen separat auf der Kommunikationsplattform implementiert werden.

- **Quellen:** [FIRST.org — TLP-SIG](https://www.first.org/global/sigs/tlp/), [CISA — TLP 2.0 User Guide (PDF)](https://www.cisa.gov/sites/default/files/2023-02/tlp-2-0-user-guide_508c.pdf)
- **Verifikation:** 3-0 (fünf Einzelaussagen zusammengeführt, alle einstimmig bestätigt), korroboriert durch CERT-EU und Wikipedia
- **Praktische Konsequenz:** TLP als Tagging-Konvention einbauen (z.B. Raumnamen-Präfix `TLP:AMBER — ...` in Matrix, Event-Tags in MISP), aber nicht als Zugriffsschutz verlassen — das übernimmt E2EE + IdP-Zugriffskontrolle.

### RFC 2350 — CSIRT-Kommunikationsstandard

> RFC 2350 (IETF-Standard zur CSIRT-Selbstbeschreibung, weiterhin aktiv) verlangt, dass CSIRTs mindestens einen PGP-Key für sichere Kommunikation mit Konstituenten und Peer-Teams pflegen, und definiert die drei Kern-Sicherheitsziele für CSIRT-Kommunikation: Vertraulichkeit, Integrität, Authentizität — erreichbar u.a. via PGP oder PEM.

- **Quelle:** [IETF — RFC 2350](https://datatracker.ietf.org/doc/html/rfc2350)
- **Verifikation:** 3-0, als weiterhin aktuelle Praxis bestätigt durch CERT-EU-, Red-Hat- und CSIRT@IPB-Dokumente (2023-2025), die weiterhin dem RFC-2350-Format inkl. PGP-Key-Veröffentlichung folgen

### NIST — E-Mail ungeeignet für Out-of-Band-Authentifizierung

> NIST SP 800-63B schließt E-Mail explizit als Out-of-Band-Authentifizierungskanal aus (Begründung: reiner Passwort-Zugriff, Abfangrisiko, DNS-Spoofing/Rerouting-Angriffe).

- **Quellen:** [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html), [Cygnvs — Out-of-Band Crisis Communications](https://www.cygnvs.com/resources/insights/out-of-band-crisis-communications-how-to-coordinate-when-your-systems-are-compromised)
- **Verifikation:** 2-1 — der NIST-Originaltext wurde unabhängig verifiziert; die spezifische Anwendung auf "Incident Responder" ist eine plausible Erweiterung des Blog-Autors, keine direkte NIST-Aussage zu IR-Teams
- **Bedeutung:** Stützt die Architekturentscheidung, 2FA/Identity-Infrastruktur für die IR-Plattform komplett getrennt vom (potenziell kompromittierten) Firmen-Mailsystem aufzubauen — inklusive einer eigenständigen Identity-Provider-Instanz statt Wiederverwendung der Haupt-Instanz.

## 2. Explizit widerlegte Annahmen

Diese Aussagen klingen plausibel, wurden aber bei der adversarialen Verifikation widerlegt (mind. 2 von 3 unabhängigen Gegenstimmen) — sie sind **nicht** Teil der Empfehlung:

| Aussage | Quelle | Abstimmung |
|---|---|---|
| "AWS empfiehlt explizit Out-of-Band-Tools wie Amazon WorkMail/Amazon Chime für IR-Kommunikation" | [AWS Security Blog](https://aws.amazon.com/blogs/security/forensic-investigation-environment-strategies-in-the-aws-cloud/) | 0-3 widerlegt |
| "CryptPad ist eine Ende-zu-Ende-verschlüsselte, Zero-Knowledge Office-Suite (Server kann nicht entschlüsseln)" | [cryptpad.org](https://cryptpad.org/) | 0-3 widerlegt |
| "CryptPad unterstützt Self-Hosting und anonymes, account-freies Teilen von Dokumenten" | [cryptpad.org](https://cryptpad.org/) | 0-3 widerlegt |
| "ArmorText ist eine speziell für Out-of-Band-IR gebaute Plattform, die auch bei kompromittiertem Firmennetz unabhängig funktioniert" | [armortext.com/incident](https://armortext.com/incident/) | 0-3 widerlegt |
| "Bei ArmorText können weder interne Admins noch der Anbieter selbst auf Nutzerkommunikation zugreifen" | [armortext.com/incident](https://armortext.com/incident/) | 0-3 widerlegt |
| "Out-of-Band-IR-Kanäle sollten unabhängige Authentifizierung und andere Infrastruktur als Produktivsysteme nutzen, mit klaren Aktivierungskriterien" | [nhimg.org](https://nhimg.org/faq/how-should-organisations-set-up-out-of-band-communications-for-incident-response/) | 1-2 widerlegt |
| "Für verteilte/Hochrisiko-Organisationen: ein Text-Kanal + ein Sprach-Kanal + ein Offline-Kontaktverzeichnis, damit Kanäle unabhängig ausfallen" | [nhimg.org](https://nhimg.org/faq/how-should-organisations-set-up-out-of-band-communications-for-incident-response/) | 0-3 widerlegt |
| "Matrix SSSS speichert verschlüsselte Cross-Signing-Keys server-seitig, geschützt durch Passphrase (PBKDF2, 500.000 Iterationen), unzugänglich für Server-Admins" | [matrix.org](https://matrix.org/docs/older/e2ee-cross-signing/) | 1-2 nicht bestätigt |

**Einordnung:** Die widerlegten Aussagen stammen überwiegend aus Vendor-Marketing (CryptPad, ArmorText) oder einer nicht-autoritativen Blog-Quelle (nhimg.org). Sie sollten **nicht** als Grundlage für Sicherheitsentscheidungen zitiert werden, ohne unabhängig nachgeprüft zu werden.

## 3. Unbeantwortete Fragen (keine ausreichenden Quellen gefunden)

- Konkrete, verifizierte Kosten-/Instanzgrößen-Schätzung für einen 10-50-Nutzer-Stack auf EC2 in eu-central-1 (Compute, Storage, Backup, Monitoring, DDoS-Schutz)
- Verifizierte Evidenz zu Implementierungsdetails von self-hosted 2FA/MFA (TOTP vs. FIDO2/WebAuthn) und konkreten Integrationsmustern Authentik/Keycloak ↔ Matrix/Element
- Primärquellen-belegte Dokumentation, welche konkreten Out-of-Band-Plattformen reale CERTs/CSIRTs (CERT-Bund, DFN-CERT, FIRST-Mitgliedsteams) tatsächlich einsetzen — über die RFC-2350/TLP-Baseline hinaus
- Belastbarer, verifizierter Feature-Vergleich Matrix/Element vs. Wire (OSS) vs. Rocket.Chat vs. Mattermost speziell für IR-relevante Kriterien (forensische Audit-Logs, Self-Destruct-Nachrichten, Geräteverifikation, Betriebsaufwand)

## 4. Alle ausgewerteten Quellen

| URL | Typ | Suchrichtung |
|---|---|---|
| https://aws.amazon.com/blogs/security/forensic-investigation-environment-strategies-in-the-aws-cloud/ | Primärquelle (AWS) | Standards & Best Practices |
| https://aws.amazon.com/blogs/security/a-framework-for-securely-collecting-forensic-artifacts-into-s3-buckets/ | Primärquelle (AWS) | Sicherer Dateiaustausch |
| https://matrix.org/docs/older/e2ee-cross-signing/ | Primärquelle (Matrix.org) | Messaging-Stacks |
| https://element.io/blog/verifying-your-devices-is-becoming-mandatory-2/ | Blog (Element) | Messaging-Stacks |
| https://www.misp-project.org/ | Primärquelle (MISP-Projekt) | Dateiaustausch/IOC-Sharing |
| https://www.misp-project.org/features/ | Primärquelle (MISP-Projekt) | Dateiaustausch/IOC-Sharing |
| https://www.misp-project.org/compliance/GDPR/ | Primärquelle (MISP-Projekt) | Dateiaustausch/IOC-Sharing |
| https://www.circl.lu/doc/misp/best-practices/ | Primärquelle (CIRCL) | Dateiaustausch/IOC-Sharing |
| https://cryptpad.org/ | Sekundärquelle (Hersteller) | Dateiaustausch/IOC-Sharing |
| https://www.first.org/global/sigs/tlp/ | Primärquelle (FIRST.org) | CERT/CSIRT-Praxis |
| https://datatracker.ietf.org/doc/html/rfc2350 | Primärquelle (IETF) | CERT/CSIRT-Praxis |
| https://www.cisa.gov/sites/default/files/2023-02/tlp-2-0-user-guide_508c.pdf | Primärquelle (CISA) | CERT/CSIRT-Praxis |
| https://armortext.com/incident/ | Blog (Vendor) | Standards & Best Practices |
| https://nhimg.org/faq/how-should-organisations-set-up-out-of-band-communications-for-incident-response/ | Blog | Standards & Best Practices |
| https://www.cygnvs.com/resources/insights/out-of-band-crisis-communications-how-to-coordinate-when-your-systems-are-compromised | Blog | Standards & Best Practices |
| https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2023/05_DER_Detektion_und_Reaktion/DER_4_Notfallmanagement_Edition_2023.pdf | Primärquelle (BSI) | Standards & Best Practices |
| https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/it-grundschutz-kompendium_node.html | Primärquelle (BSI) | Standards & Best Practices |
| https://forensicspot.com/topics/incident-response-and-management/sans-picerl-model | Blog | Standards & Best Practices |
| https://codeattack.io/en/blog/self-hosted-infrastructure-2026 | Blog | AWS EC2 Kosten/Alternativen |
| https://gartsolutions.com/hetzner-vs-aws/ | Blog | AWS EC2 Kosten/Alternativen |

Hinweis: Die beiden BSI-Quellen (IT-Grundschutz-Kompendium, Baustein DER.4 Notfallmanagement) lieferten Kontext zu Notfallhandbuch-/Kommunikationsplan-Anforderungen, wurden aber nicht in die 25 tiefenverifizierten Kernaussagen aufgenommen (Budget-Priorisierung der Verifikationsstufe) — bei Bedarf gezielt nachrecherchieren.

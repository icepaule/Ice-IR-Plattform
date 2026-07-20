# Schlussfolgerungen

## Beantwortung der Ausgangsfrage

**"Kann ich das in AWS als EC2-Instanz bauen?"** — Technisch ja, aber eine **einzelne** EC2-Instanz im bestehenden AWS-Account entspricht nicht dem, was AWS selbst für Out-of-Band-Forensik-/IR-Infrastruktur empfiehlt. AWS' eigene Sicherheits-Dokumentation empfiehlt einen separaten, isolierten Account mit restriktiver, auditierbarer VPC (siehe [01-analyse-und-quellen.md](01-analyse-und-quellen.md), [02-architektur.md](02-architektur.md)). Verfügt eine Organisation bereits über einen dedizierten Out-of-Band-Account, reduziert sich der verbleibende Architektur-Aufwand auf eine korrekt restriktive VPC-Konfiguration.

**Lizenzkosten:** Der empfohlene Software-Stack (Matrix/Synapse, Element, Authentik, MISP, optional MinIO) ist vollständig Open Source ohne Lizenzkosten. Laufende Kosten entstehen ausschließlich durch Infrastrukturverbrauch (Compute, Storage, ggf. genutzte AWS-Managed-Services wie S3/KMS).

## Zentrale Erkenntnisse

1. **Out-of-Band ist eine Architekturfrage, keine Produktfrage.** Es gibt keine "Out-of-Band-Software" im eigentlichen Sinne — Out-of-Band entsteht durch Konto-/Netz-Isolation plus eigenständige Identity-Infrastruktur, nicht durch die Wahl eines bestimmten Chat-Tools. Vendor-Marketing, das "echtes Out-of-Band" als Produkteigenschaft verspricht (siehe widerlegte ArmorText-Aussagen in [01-analyse-und-quellen.md](01-analyse-und-quellen.md)), sollte kritisch geprüft werden.

2. **TLP kennzeichnet, verschlüsselt aber nicht.** Ein häufiges Missverständnis: TLP (Traffic Light Protocol) wird oft mit einem technischen Sicherheitsstandard verwechselt. Es ist ausschließlich eine Sensitivitäts-Kennzeichnungskonvention — die eigentliche technische Absicherung (E2EE, Zugriffskontrolle) muss die Plattform separat leisten.

3. **E-Mail-basierte Auth-Recovery ist ein Out-of-Band-Antipattern.** NIST schließt E-Mail explizit als Out-of-Band-Auth-Kanal aus — daraus folgt, dass die Identity-/2FA-Infrastruktur für die IR-Plattform nicht auf dieselbe E-Mail-Infrastruktur zurückgreifen sollte wie der Alltagsbetrieb.

4. **Nicht jede plausible Quelle hält der Prüfung stand.** Von 25 tiefenverifizierten Aussagen wurden 8 explizit widerlegt, darunter mehrere Vendor-Marketing-Claims (CryptPad "Zero-Knowledge", ArmorText "Out-of-Band"). Das unterstreicht: Sicherheitsrelevante Produktversprechen sollten vor Einsatz unabhängig geprüft werden, nicht aus Werbematerial übernommen werden.

## Risiken und Fallstricke (Zusammenfassung)

Siehe Detail-Ausführung in [02-architektur.md](02-architektur.md), Abschnitt "Wichtige Sicherheitsrisiken und Fallstricke":

- Schlüsselverwaltung (Cross-Signing-/Recovery-Keys)
- Recovery-Prozess bei 2FA-Geräteverlust
- Metadaten-Leaks trotz E2EE
- Federation-Deaktivierung nicht vergessen
- DDoS-Schutz bei öffentlicher Erreichbarkeit
- TLP nicht mit Zugriffsschutz verwechseln
- Backup-Schlüsselverwaltung getrennt vom System

## Offene Punkte für Nachrecherche

Diese Teilfragen der ursprünglichen Recherche blieben ohne ausreichend verifizierte Quellenlage (siehe [01-analyse-und-quellen.md](01-analyse-und-quellen.md), Abschnitt 3):

- Konkrete, verifizierte Kostenschätzung für 10-50 Nutzer auf EC2 in eu-central-1
- Implementierungsdetails self-hosted 2FA/FIDO2 speziell in Kombination Authentik/Keycloak ↔ Matrix
- Primärquellen-belegte Praxisbeispiele realer CERTs/CSIRTs zu ihrer konkreten Plattformwahl
- Belastbarer Feature-Vergleich Matrix/Element vs. Wire (OSS) vs. Rocket.Chat vs. Mattermost für IR-spezifische Kriterien

## Empfohlene nächste Schritte

1. VPC-Konfiguration im bestehenden AWS-OOB-Account gegen die AWS-Forensik-Best-Practices prüfen (isoliert, restriktiv, auditierbar, kein Shared-Workload)
2. Proof-of-Concept gemäß [03-poc-anleitung.md](03-poc-anleitung.md) auf vorhandener Infrastruktur aufsetzen und den 2FA-/Verifikations-Flow end-to-end testen
3. Bei Bedarf gezielte Nachrecherche zu den oben genannten offenen Punkten (insbesondere FIDO2-Integrationsdetails und Kostenkalkulation mit aktuellem AWS Pricing Calculator)
4. Migration der validierten POC-Konfiguration in die isolierte AWS-Produktivumgebung

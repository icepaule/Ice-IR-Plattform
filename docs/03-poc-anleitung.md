# Schritt-für-Schritt-Anleitung: Proof-of-Concept

Ziel des POC: den empfohlenen Software-Stack (Matrix/Synapse + Element hinter einem bestehenden Authentik-Identity-Provider) auf vorhandener Virtualisierungs-Infrastruktur evaluieren — Software-Auswahl, 2FA-Flow und Bedienung testen, bevor eine isolierte Produktivumgebung aufgesetzt wird.

**Wichtiger Hinweis:** Dieser POC bildet **keine** echte Netzwerk-Isolation vom Alltagsbetrieb ab (siehe [02-architektur.md](02-architektur.md)) — er dient ausschließlich der Software-/UX-Evaluierung. Für den produktiven Out-of-Band-Einsatz ist eine komplett separate Umgebung (anderer Account/Provider) notwendig.

## Voraussetzungen

- Ein Proxmox-Host (oder vergleichbarer Hypervisor) mit ausreichend freiem RAM (Richtwert: ≥ 8 GB frei für die POC-VM)
- Eine bereits laufende Authentik-Instanz als Identity-Provider (Version mit OIDC-Provider-Unterstützung)
- Ein eigenes, vom Produktivnetz getrenntes VLAN/Netzsegment auf dem Hypervisor (empfohlen, auch im POC-Maßstab schon zur Gewöhnung an das Out-of-Band-Prinzip)

## Sizing (Richtwert für 10-50 Nutzer im POC-Maßstab)

| Ressource | Wert |
|---|---|
| vCPU | 4 |
| RAM | 8 GB |
| Storage | 60 GB SSD |
| Betriebssystem | Debian oder Ubuntu (minimal) |

## Schritte

### 1. VM anlegen

Neue VM auf dem Hypervisor anlegen (Sizing siehe oben), eigenes internes Netz/VLAN zuweisen, kein Routing zum Produktiv-LAN.

### 2. Basis-Stack installieren

Postgres + Matrix Synapse per Docker-Compose installieren (analog zu bestehenden Compose-basierten Diensten). Wichtig in der Synapse-Konfiguration (`homeserver.yaml`):

```yaml
federation_domain_whitelist: []
```

Federation deaktivieren — sonst ist der Homeserver nicht isoliert.

### 3. Element Web als Frontend

Element Web als statisches Frontend vor den Homeserver stellen (eigener Reverse Proxy, eigenes TLS-Zertifikat, eigene Domain/Subdomain).

### 4. OIDC-Provider in Authentik anlegen

In der bestehenden Authentik-Instanz eine **neue, eigenständige** OIDC-Provider/-Application anlegen (z.B. „IR-Matrix"). Bewusst **nicht** denselben Flow/Client wie für andere Dienste wiederverwenden — Ziel: ein Incident, der andere Teile der Authentik-Konfiguration betrifft, soll den IR-Zugang nicht mit gefährden.

### 5. Synapse mit Authentik verbinden

In `homeserver.yaml` den `oidc_providers`-Block auf die neue Authentik-Application zeigen lassen (Client-ID, Client-Secret, Discovery-URL). TOTP/FIDO2-Erzwingung für diese spezifische Application im Authentik-Login-Flow aktivieren.

### 6. Geräteverifikation testen

Für alle Test-Accounts in Element die Cross-Signing-/Geräteverifikation durchführen (siehe [01-analyse-und-quellen.md](01-analyse-und-quellen.md) zum Cross-Signing-Modell) und den kompletten Login- und Verifikations-Flow end-to-end durchspielen.

### 7. Optional: MISP-Instanz ergänzen

Für den Test des IOC-Austausch-Workflows eine zweite kleine VM/Container mit MISP aufsetzen, TLP-Tagging und STIX-Export testen.

## Was danach zu prüfen ist

- Funktioniert die 2FA-Erzwingung zuverlässig auch bei neuen Geräten/Sessions?
- Ist die Geräteverifikation für Nutzer verständlich genug, um sie im Ernstfall unter Zeitdruck korrekt durchzuführen?
- Wie verhält sich der Recovery-Prozess bei Verlust eines 2FA-Geräts?
- Skaliert die Konfiguration ohne größere Anpassungen auf eine isolierte Produktivumgebung (siehe [02-architektur.md](02-architektur.md))?

## Nächster Schritt nach erfolgreichem POC

Migration der validierten Konfiguration auf eine vollständig isolierte Produktivumgebung (separates Konto/Projekt bei einem Cloud-Anbieter, eigene Netz-Isolation, eigene Backup-Strategie) — siehe Architektur-Empfehlung in [02-architektur.md](02-architektur.md).

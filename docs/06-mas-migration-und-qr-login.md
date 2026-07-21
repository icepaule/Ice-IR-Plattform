# MAS-Migration und QR-Code-Login

Update nach dem ersten produktiven POC-Betrieb: Der ursprüngliche Aufbau (Synapse mit klassischem `oidc_providers`-Delegationsmodus, siehe [03-poc-anleitung.md](03-poc-anleitung.md)) wurde auf **Matrix Authentication Service (MAS)** migriert.

## Warum MAS

Der klassische `oidc_providers`-Mechanismus in Synapse delegiert nur den Login selbst an den externen Identity-Provider, der komplette OAuth2/OIDC-Autorisierungs-Stack (Token-Ausstellung, Geräte-Autorisierung, Session-Handling) bleibt Teil von Synapse. Für zwei praxisrelevante Anforderungen reicht das nicht mehr aus:

- **QR-Code-Login zur Geräteverknüpfung** (Anmeldung auf einem Zweitgerät durch Scannen eines QR-Codes auf einem bereits angemeldeten Gerät) erfordert [MSC4108](https://github.com/matrix-org/matrix-spec-proposals/pull/4108). MSC4108 setzt seinerseits [MSC3861](https://github.com/matrix-org/matrix-spec-proposals/pull/3861) (delegierte OAuth2-Autorisierung über einen dedizierten Auth-Service) voraus — ein reines Synapse-Feature-Flag ohne MAS führt zu einem Konfigurationsfehler und startet den Homeserver nicht.
- Für eine spätere Produktivumgebung (siehe [02-architektur.md](02-architektur.md)) ist MAS ohnehin der von den Matrix-Maintainern empfohlene Weg — der klassische `oidc_providers`-Mechanismus gilt als Übergangslösung.

## Migrationsablauf (Kurzfassung)

1. **Backup vor der Migration**: vollständiger Snapshot der VM/des Containers sowie ein Datenbank-Dump, um im Fehlerfall verlustfrei zurückrollen zu können.
2. **MAS als zusätzlichen Dienst** neben Postgres/Synapse/Element in die bestehende Compose-Umgebung aufnehmen (eigenes Image, eigene Konfigurationsdatei, eigene Datenbank auf derselben Postgres-Instanz).
3. **Migrationswerkzeug `syn2mas`** (offizielles Element-Tool) in drei Stufen ausführen:
   - `check` — validiert, dass Synapse- und MAS-Konfiguration zueinander passen
   - `dry-run` — simuliert die Migration ohne Schreibzugriff
   - `migrate` — echte Migration bestehender Nutzer-Accounts und aktiver Sessions, **ohne** Zwang zur Neuanmeldung
4. **Eigene Subdomain für MAS** einrichten (eigener OAuth2/OIDC-Endpunkt-Namespace, damit keine Pfadkollision mit dem Element-Web-Frontend auf derselben Domain entsteht), inkl. eigenem TLS-Zertifikat und Split-DNS-Eintrag analog zur bestehenden Element/Synapse-Domain.
5. **Reverse-Proxy-Kompatibilitätsschicht**: die Endpunkte `/_matrix/client/*/login`, `/logout` und `/refresh` werden an MAS geroutet, alle übrigen Matrix-Client-API-Aufrufe bleiben bei Synapse. Das ist die von den Matrix-Maintainern dokumentierte Übergangsroute, bis ein Homeserver-Release MAS vollständig intern terminiert.
6. **Identity-Provider-Konfiguration anpassen**: die Redirect-URI in der Authentik-Application zeigt danach auf den MAS-Callback statt auf den vorherigen Synapse-Callback.
7. **`oidc_providers`-Block erst nach erfolgreicher Migration entfernen** (siehe Fallstricke unten).

## Verifikation nach der Migration

- Alle Dienste (Datenbank, Homeserver, Web-Client, MAS) melden sich als gesund (`healthy`).
- Bestehende, bereits angemeldete Sitzungen funktionieren ohne erzwungene Neuanmeldung weiter.
- Die OIDC-Discovery- bzw. MSC2965-Auth-Metadata-Endpunkte von MAS liefern korrekt einen `device_authorization_endpoint` — das ist die Voraussetzung für den QR-Code-Login.
- Ein realer Login-Versuch von einem mobilen Client (iPhone/Android) über den neuen `/upstream/authorize`-Endpunkt wurde erfolgreich beobachtet.

## Fallstricke (praxisrelevant für Nachbauten)

- **Passwort-Schema-Mismatch**: MAS' Konfigurationsvalidierung (`syn2mas check`) bricht ab, wenn das dort konfigurierte Passwort-Schema nicht exakt zum `password_config.pepper`-Wert in Synapses `homeserver.yaml` passt. Bei deaktiviertem Passwort-Login (SSO-only, siehe unten) beide Werte explizit auf einen übereinstimmenden leeren Zustand setzen, statt sie implizit undefiniert zu lassen.
- **`oidc_providers` darf während der echten Migration noch nicht entfernt sein**: `syn2mas migrate` schlägt fehl, wenn bereits über den alten Provider verknüpfte Nutzer-Accounts in der Datenbank existieren, der Provider in der Konfiguration aber schon entfernt wurde. Reihenfolge: erst migrieren, dann aufräumen.
- **MSC4108 nicht isoliert aktivieren**: das reine Synapse-Flag ohne MAS führt zu einem Startfehler des Homeservers (kurzer Ausfall im hier dokumentierten Testlauf, sofort per Konfigurations-Rollback behoben). MSC4108 ausschließlich im Rahmen einer vollständigen MAS-Migration aktivieren.

## SSO-only-Login (kein Passwort-Fallback)

Zusätzlich zur MAS-Migration wurde der klassische Passwort-Login vollständig deaktiviert — Anmeldung ist ausschließlich über den Identity-Provider (SSO + 2FA) möglich:

- Serverseitig: Passwort-Login im Homeserver deaktiviert.
- Clientseitig: der Web-Client leitet ohne Zwischenschritt direkt zum SSO-Login weiter (kein sichtbarer Passwort-Login-Button mehr).
- Feingranulare Zugriffskontrolle im Identity-Provider: externe Test-Accounts sehen im Identity-Provider-Dashboard ausschließlich die Kachel für diese Plattform, nicht die übrigen im selben Identity-Provider verwalteten Dienste (Gruppen-basierte Policy-Bindings, siehe [02-architektur.md](02-architektur.md) zum Prinzip der eigenständigen IdP-Konfiguration).

## Offene Punkte

- Aktuell teilt sich der Identity-Provider-Client für diese Plattform noch den allgemeinen Standard-Authentifizierungs-Flow der bestehenden Identity-Provider-Instanz. Vor einem echten Produktiv-/Out-of-Band-Einsatz (siehe [02-architektur.md](02-architektur.md)) sollte dafür ein dedizierter, eigenständiger Flow angelegt werden, damit eine Störung an anderer Stelle der Identity-Provider-Konfiguration diesen Login-Pfad nicht mit gefährden kann.
- End-to-End-Test des QR-Code-Device-Linking durch reale Nutzer auf mobilen Geräten steht noch aus.

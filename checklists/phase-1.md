# Phase-1-Checkliste — Manueller Update-Hinweis

## App-Vorbereitung

- [ ] App hat eine stabile App-ID (eindeutiger Name)
- [ ] App kann die eigene installierte Version lesen
- [ ] App kann die aktuelle Plattform erkennen (macos / windows / linux)
- [ ] Manifest-URL ist festgelegt und dokumentiert

## Manifest

- [ ] Manifest liegt auf einem HTTPS-Server
- [ ] `latestVersion` ist korrekt gesetzt
- [ ] `minimumVersion` ist korrekt gesetzt
- [ ] `downloadUrl` funktioniert und zeigt auf den richtigen Installer
- [ ] `changelog` ist vorhanden und aktuell
- [ ] `forceUpdate` ist korrekt gesetzt
- [ ] `publishedAt` ist gesetzt

## Update-Logik

- [ ] Versionsvergleich funktioniert numerisch (nicht als String)
- [ ] Update-Dialog wird nur angezeigt wenn wirklich eine neue Version existiert
- [ ] `latestVersion > installierte Version` → Update-Dialog
- [ ] `installierte Version < minimumVersion` → Pflicht-Update-Dialog
- [ ] Netzwerkfehler führen nicht zum App-Absturz
- [ ] Timeout für Netzwerk-Request ist gesetzt (max. 30 Sekunden)
- [ ] Update-Prüfung läuft nicht auf dem UI-Thread (kein Blockieren beim Start)

## Update-Dialog

- [ ] Aktuelle und neue Version werden angezeigt
- [ ] Changelog ist lesbar dargestellt
- [ ] "Jetzt herunterladen" öffnet die Download-URL im Browser
- [ ] "Später" ist vorhanden (außer bei Pflicht-Updates)
- [ ] Bei Pflicht-Updates: kein "Später", klare Erklärung

## Release-Prozess

- [ ] Prozess für das Aktualisieren des Manifests ist dokumentiert
- [ ] Wer darf das Manifest aktualisieren, ist definiert
- [ ] Neues Manifest nach Update-Test veröffentlicht

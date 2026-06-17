# Android-Checkliste

## Für Google-Play-Apps

- [ ] Update-Prüfung ist kein direkter APK-Download (für Play-Store-Apps)
- [ ] Eigenes Backend für Mindestversionscheck vorhanden
- [ ] Nutzer wird bei `minimumVersion`-Unterschreitung zum Play Store geleitet
- [ ] Play-Store-URL (`play.google.com/store/apps/details?id=...`) ist korrekt

## Play Core API (In-App-Updates, empfohlen)

- [ ] Play Core Library eingebunden
- [ ] Flexible Updates (Hintergrund) oder sofortige Updates implementiert
- [ ] Nutzer kann flexible Updates nach Abschluss aktivieren
- [ ] Sofortige Updates werden bei kritischen Fixes genutzt

## Eigenes Backend

- [ ] Manifest-URL ist über HTTPS erreichbar
- [ ] Netzwerkfehler werden still behandelt
- [ ] Timeout für Netzwerk-Requests gesetzt (max. 10 Sekunden)

## APK Side-Loading (falls relevant)

- [ ] Nur für Enterprise oder Beta-Tests
- [ ] Nutzer muss "Unbekannte Quellen" manuell aktivieren
- [ ] APK ist signiert (gleicher Keystore wie Play-Store-Version)
- [ ] SHA-256 der APK wird vor Installation geprüft
- [ ] Installer-Berechtigung (`REQUEST_INSTALL_PACKAGES`) vorhanden

## Signierung

- [ ] Keystore ist sicher gespeichert (nicht im Repository)
- [ ] Keystore-Passwort ist in CI/CD als Secret hinterlegt
- [ ] Play App Signing ist aktiviert (empfohlen)

## Privacy

- [ ] Keine persönlichen Daten im Update-Request
- [ ] `INTERNET`-Berechtigung in `AndroidManifest.xml`
- [ ] Datenschutzerklärung beschreibt Update-Prüfung

# iOS-Checkliste

## Für App-Store-Apps

- [ ] Update-Prüfung ist kein Installer-Download (App Store Policy)
- [ ] Eigenes Backend für Mindestversionscheck vorhanden
- [ ] Nutzer wird bei `minimumVersion`-Unterschreitung zum App Store geleitet
- [ ] Store-URL (`apps.apple.com/app/id...`) ist korrekt und getestet

## Remote-Konfiguration

- [ ] Manifest-URL ist über HTTPS erreichbar
- [ ] Netzwerkfehler werden still behandelt (kein App-Absturz)
- [ ] Timeout für Netzwerk-Requests gesetzt (max. 10 Sekunden)
- [ ] Maintenance-Modus wird dem Nutzer verständlich erklärt

## App Store Regeln

- [ ] Kein Download und Ausführen von nativem Code
- [ ] Kein OTA-Update von Swift/Objective-C Code
- [ ] JavaScript-Bundle-Updates (React Native): Store-Richtlinien gelesen
- [ ] In-App-Purchases nur über Apple-System

## Privacy & Datenschutz

- [ ] Keine persönlichen Daten im Update-Request (nur Version + Plattform)
- [ ] Keine Geräte-IDs im Update-Request ohne Nutzererlaubnis
- [ ] Datenschutzerklärung beschreibt Update-Prüfung

## TestFlight (Beta)

- [ ] TestFlight-Builds haben eigene Manifest-URL (falls nötig)
- [ ] Beta-Nutzer haben Mindestversions-Hinweise
- [ ] Beta-Ablaufdaten sind bekannt (TestFlight-Builds laufen nach 90 Tagen ab)

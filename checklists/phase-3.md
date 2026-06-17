# Phase-3-Checkliste — Automatischer Updater

Alle Punkte aus Phase-2-Checkliste müssen erfüllt sein.

## Voraussetzungen

- [ ] Release-Prozess ist stabil und dokumentiert
- [ ] Alle Builds werden vor Release getestet
- [ ] Code-Signierung ist vorhanden und aktiv (plattformabhängig)
- [ ] Integritätsprüfung (SHA-256 und/oder Signatur) existiert
- [ ] Rollback-Strategie ist schriftlich definiert

## Signierung

- [ ] macOS: App ist code-signiert mit Apple Developer Certificate
- [ ] macOS: App ist notarisiert (notarytool)
- [ ] Windows: Installer ist signiert (empfohlen: EV-Zertifikat)
- [ ] Linux: Paket-signierung nach gewähltem Format

## Installationsprozess

- [ ] Installation erfolgt erst nach erfolgreicher Verifikation
- [ ] Kein Ausführen von Remote-Skripten
- [ ] Kein Ersetzen von Systemdateien ohne explizite Notwendigkeit
- [ ] Admin-Rechte werden nur angefordert wenn wirklich nötig
- [ ] Installationsfehler werden erkannt und behandelt

## Rollback

- [ ] Backup der aktuellen Version vor Installation erstellt
- [ ] Fehler nach Installation werden erkannt (Absturz, Start schlägt fehl)
- [ ] Automatischer Rollback bei erkanntem Fehler
- [ ] Nutzer wird über Rollback informiert

## Nutzer-Einwilligung

- [ ] Update-Modell ist definiert (immer fragen / opt-in / automatisch)
- [ ] Nutzer kann Auto-Updates deaktivieren (außer bei kritischen Sicherheits-Patches)
- [ ] Nutzer wird mindestens benachrichtigt bevor ein Update installiert wird

## Monitoring

- [ ] Erfolgreiche und fehlgeschlagene Updates werden geloggt
- [ ] Logs enthalten keine Secrets oder persönliche Daten
- [ ] Ungewöhnliche Update-Fehler können erkannt und analysiert werden

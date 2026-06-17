# Phase-2-Checkliste — Geführter Download

Alle Punkte aus Phase-1-Checkliste müssen erfüllt sein.

## Zusätzliche Manifest-Felder

- [ ] `sha256` ist im Manifest für jede Plattform vorhanden
- [ ] `fileSize` ist im Manifest angegeben
- [ ] SHA-256 wurde nach dem Build berechnet und verifiziert

## Download-Implementierung

- [ ] Download läuft im Hintergrund (kein UI-Freeze)
- [ ] Fortschrittsbalken zeigt aktuellen Stand
- [ ] Abbrechen des Downloads ist möglich
- [ ] Unterbrochene Downloads können erneut gestartet werden
- [ ] Download-Ziel ist ein sicheres temporäres Verzeichnis
- [ ] Abgebrochene Downloads hinterlassen keine unvollständigen Dateien

## Verifizierung

- [ ] SHA-256 der heruntergeladenen Datei wird lokal berechnet
- [ ] SHA-256 wird mit dem Manifest verglichen
- [ ] Bei Abweichung: Datei löschen, Fehlermeldung anzeigen
- [ ] Nutzer kann bei Verifizierungsfehler erneut versuchen

## Nutzer-Feedback

- [ ] Dateiname und Dateigröße werden vor dem Download angezeigt
- [ ] Fortschritt in Prozent und verbleibende Zeit sichtbar
- [ ] Nach Download: klare Handlungsaufforderung
- [ ] Netzwerkfehler: verständliche Fehlermeldung mit Retry-Option
- [ ] Zu wenig Speicher: frühzeitig prüfen und erklären

## Sicherheit

- [ ] Kein Schreiben in schreibgeschützte Systemverzeichnisse
- [ ] Keine Admin-Rechte für den Download selbst
- [ ] Download-URL kommt aus dem verifizierten Manifest (nicht aus Nutzer-Input)

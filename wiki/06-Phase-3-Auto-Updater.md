# 06. Phase 3 — Automatischer Updater

Automatische Updates sollten erst hinzugefügt werden wenn der Release-Prozess stabil und getestet ist.

Ein Auto-Updater der schlecht implementiert ist, ist gefährlicher als gar kein Updater.

---

## Voraussetzungen für Phase 3

Folgendes muss erfüllt sein bevor Phase 3 begonnen wird:

- [ ] Phase 1 und Phase 2 laufen stabil
- [ ] Release-Prozess ist dokumentiert und reproduzierbar
- [ ] Builds sind signiert (macOS: Notarization, Windows: Code-Signing)
- [ ] Integritätsprüfung existiert (Checksummen oder Signaturen)
- [ ] Rollback-Strategie ist definiert
- [ ] Update-Fehler werden behandelt und geloggt
- [ ] Nutzer-Einwilligung ist geregelt

---

## Ablauf eines sicheren Auto-Updaters

```text
App erkennt neues Update
↓
Manifest laden und Signatur prüfen
↓
Nutzer wird informiert (außer bei Hintergrund-Updates)
↓
Installer herunterladen
↓
SHA-256 und Signatur prüfen
↓
Installer sicher ausführen
↓
App neugestartet
↓
Fehlerfall: Rollback zur vorherigen Version
```

---

## Nutzer-Einwilligung

Es gibt drei Modelle:

| Modell | Beschreibung | Wann geeignet |
|--------|-------------|---------------|
| Immer fragen | Nutzer bestätigt jedes Update | Konservative Apps |
| Opt-in still | Nutzer kann aktivieren | Standard-Einstellung |
| Vollautomatisch | Updates ohne Interaktion | Kritische Sicherheits-Updates |

**Empfehlung:** Vollautomatisch nur für kritische Sicherheits-Updates. Für alles andere mindestens eine Benachrichtigung.

---

## Signierung

### macOS

```text
App muss code-signiert sein
→ Apple Developer Account nötig
→ Notarisierung via notarytool
→ Gatekeeper-Prüfung beim Nutzer
```

### Windows

```text
EV Code-Signing-Zertifikat empfohlen
→ SmartScreen-Filter
→ Installer-Signierung (z.B. mit signtool)
```

### Linux

```text
Paketspezifische Signierung
→ .deb: GPG-Signatur
→ Flatpak: Signierung im Flatpak-System
→ AppImage: optionale Signatur
```

---

## Rollback

Ein guter Auto-Updater hat eine Rollback-Strategie:

```text
Vor der Installation:
- Backup der aktuellen Version erstellen

Bei Fehler nach der Installation:
- Fehler erkennen (Absturz, Start schlägt fehl)
- Backup wiederherstellen
- Nutzer informieren
- Fehler loggen
```

---

## Was niemals tun

- Keine Remote-Skripte ausführen ohne vorherige Verifikation
- Keine Binärdateien ersetzen ohne Checksummen- oder Signaturprüfung
- Keine Admin-Rechte anfordern die nicht wirklich nötig sind
- Kein stilles Update das der Nutzer nicht deaktivieren kann (außer bei kritischen Sicherheits-Patches)

---

## Checkliste Phase 3

→ Vollständige Checkliste: [`checklists/phase-3.md`](../checklists/phase-3.md)

---

## Nächster Schritt

→ Weiter mit [`07-Sicherheit.md`](07-Sicherheit.md)

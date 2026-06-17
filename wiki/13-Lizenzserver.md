# 13. Lizenzserver

Ein Lizenzserver ist erst dann relevant wenn kommerzielle Lizenzierung eine konkrete Anforderung ist.

**Nicht zu früh implementieren.** Ein Lizenzserver ist komplex, wartungsintensiv und sicherheitskritisch.

---

## Wann ein Lizenzserver?

Ein Lizenzserver wird nötig wenn:
- Updates an Lizenzkauf gebunden sind
- Nur zahlende Kunden bestimmte Versionen bekommen
- Lizenzen ablaufen und erneuert werden müssen
- Geräte-basierte Lizenzierung nötig ist
- Staged Rollouts nach Abonnementstufe kontrolliert werden

---

## Lizenzserver-Architektur

```text
Nutzer kauft Lizenz
↓
Lizenzserver stellt Lizenzschlüssel aus
↓
App aktiviert sich beim ersten Start
↓
App prüft bei Update: Lizenz gültig?
  ├── Ja → Download-URL bereitstellen
  └── Nein → Update verweigern oder auf Kaufseite weiterleiten
```

---

## Lizenzprüfung im Update-Flow

```json
{
  "app": "example-premium-app",
  "platform": "macos",
  "licenseRequired": true,
  "updateCheckUrl": "https://api.example.com/updates/check",
  "activationUrl": "https://api.example.com/licenses/activate"
}
```

Die App sendet bei der Update-Prüfung:
- App-ID
- Installierte Version
- Plattform
- Lizenzschlüssel (verschlüsselt oder als Hash)

Der Server antwortet mit:
- Update verfügbar: ja/nein
- Download-URL (falls Lizenz gültig)
- Lizenzstatus
- Ablaufdatum

---

## Wichtige Sicherheitsregeln

**Nie prüfbare Lizenzvalidierung nur client-seitig.**

Alles was nur in der App passiert, kann umgangen werden. Die echte Validierung findet auf dem Server statt.

**Lizenzschlüssel nicht als Plaintext in Logs.**

Lizenzschlüssel sind wie Passwörter zu behandeln.

**Offline-Modus planen.**

Was passiert wenn der Lizenzserver kurzzeitig nicht erreichbar ist?

Optionen:
- Kurze Toleranzzeit (z.B. 72 Stunden Offline-Betrieb erlaubt)
- Letztes bekanntes Lizenzstatus-Ergebnis cachen
- Klare Fehlermeldung wenn Lizenz nicht geprüft werden kann

---

## Lizenzmodelle

| Modell | Beschreibung |
|--------|-------------|
| Lifetime | Einmaliger Kauf, Updates für eine Hauptversion |
| Subscription | Monatlich/jährlich, laufende Updates |
| Per-Device | Lizenz an bestimmte Geräte gebunden |
| Floating | Pool von Lizenzen für Teams |
| Trial | Zeitlich begrenzte Vollversion |

---

## Erst implementieren wenn

- Zahlungssystem existiert
- Lizenzmodell definiert ist
- Support-Prozess für Lizenzprobleme existiert
- Backup-Prozess für Lizenzserver existiert

---

## Nächster Schritt

→ Weiter mit [`14-FAQ.md`](14-FAQ.md)

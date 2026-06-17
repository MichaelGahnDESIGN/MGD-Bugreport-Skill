# 03. Architektur

Eine gute Update-Architektur beginnt einfach.

```text
Anwendung
↓ lädt
Manifest (JSON)
↓ zeigt auf
Installer oder Paket
```

---

## Komponenten eines Update-Systems

| Komponente | Beschreibung |
|-----------|-------------|
| Update-Client | Code in der App der prüft, lädt und installiert |
| Manifest | JSON-Datei mit Versions- und Download-Infos |
| Release-Speicher | Server, CDN oder Hosting wo Installer liegen |
| Release-Prozess | Dokumentierter Ablauf für neue Releases |

---

## Was die App wissen muss

Jede App die Updates prüft, braucht:

```text
- App-ID         (eindeutiger Name der App)
- Installierte Version
- Plattform      (macos / windows / linux / android / ios)
- Update-Kanal   (stable / beta — optional)
- Manifest-URL
```

---

## Phase-1-Architektur (Empfehlung)

```text
App startet
↓
App liest installierte Version
↓
App lädt latest.json via HTTPS
↓
Versionsvergleich
  ├── latestVersion > installierte Version → Update-Dialog anzeigen
  └── installierte Version < minimumVersion → Pflicht-Update-Dialog
↓
Nutzer klickt "Herunterladen"
↓
Browser öffnet downloadUrl
```

---

## Phase-2-Architektur

```text
[Phase 1] + App lädt Installer selbst herunter
↓
Fortschrittsanzeige
↓
Checksummen-Verifikation
↓
"Installer öffnen" oder automatisch starten
```

---

## Phase-3-Architektur

```text
[Phase 2] + App installiert ohne Browser-Schritt
↓
Signaturprüfung
↓
Sichere Installation
↓
Neustart
↓
Rollback wenn Fehler
```

---

## Statisches JSON vs. API

### Statisches JSON

```text
App
↓ GET
https://updates.example.com/example-app/macos/latest.json
↓
Vergleich direkt in der App
```

Vorteile: Kein Server, günstig, einfach.

### API-Endpunkt

```text
App
↓ POST mit {appId, version, platform, channel}
https://api.example.com/updates/check
↓
Antwort mit {updateAvailable, downloadUrl, ...}
```

Vorteile: Lizenzprüfung, Rollout-Kontrolle, Statistiken.

---

## Wo Installer speichern?

Optionen von einfach bis komplex:

| Option | Geeignet für |
|--------|-------------|
| Eigenes Webhosting | Frühe Projekte, kleine Nutzerzahl |
| GitHub Releases | Open-Source-Projekte |
| CDN (Cloudflare, Bunny etc.) | Viele Nutzer, schnelle Downloads |
| Eigener S3-kompatibler Speicher | Volle Kontrolle, skalierbar |

**Niemals** private Repositories als Update-Quelle verwenden wenn dafür ein Access-Token in der App eingebettet werden müsste.

---

## Nächster Schritt

→ Weiter mit [`04-Phase-1-Manuelle-Updates.md`](04-Phase-1-Manuelle-Updates.md)

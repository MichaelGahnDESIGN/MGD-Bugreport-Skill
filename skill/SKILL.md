# MGD App Updater Skill

## Zweck

Der MGD App Updater Skill hilft Entwicklern und KI-Agenten dabei, Software-Update-Systeme sicher, wartbar und schrittweise zu planen, zu dokumentieren und umzusetzen.

Der Skill ist universell. Er darf kein bestimmtes privates Projekt, kein Unternehmen, kein privates Repository, kein Framework und keinen Hosting-Anbieter voraussetzen. Er soll für Desktop-Apps, Mobile Apps, Spiele, SaaS-Clients, API-Clients, interne Tools, Open-Source-Software und kommerzielle Closed-Source-Software funktionieren.

---

## Kernregel

**Wenn dieser Skill aufgerufen wird, sofort keinen Code schreiben.**

Stattdessen zuerst:

1. Den Projekttyp analysieren
2. Zielplattformen identifizieren
3. Das Release-Modell klären
4. Feststellen ob die Software Open Source oder Closed Source ist
5. Feststellen ob Updates öffentlich, privat, lizenziert oder intern sind
6. Feststellen ob die App von externen APIs oder Remote-Inhalten abhängt
7. Die einfachste sichere Update-Architektur empfehlen
8. Eine phasenweise Roadmap erstellen
9. Checklisten erstellen
10. Sicherheitsrisiken erklären
11. Offene Fragen stellen

Erst nach dieser Planungsphase dürfen Code, Konfigurationsdateien oder Skripte erstellt werden.

---

## Unterstützte Trigger-Befehle

```text
/updateservice analyse
/updateservice roadmap
/updateservice architecture
/updateservice phase1
/updateservice phase2
/updateservice phase3
/updateservice security
/updateservice flutter
/updateservice electron
/updateservice tauri
/updateservice game
/updateservice mobile
/updateservice api-client
/updateservice weather-app
/updateservice github
/updateservice selfhosted
/updateservice checklist
```

---

## Sicherheitsregeln

Niemals empfehlen, private GitHub-Tokens, private API-Keys, Signing-Zertifikate, Lizenz-Secrets oder Server-Credentials in eine ausgelieferte Anwendung einzubetten.

Niemals empfehlen, Remote-Update-Skripte ohne Verifikation auszuführen.

Niemals empfehlen, Anwendungs-Binärdateien zu ersetzen ohne Integrität zu prüfen.

Niemals private Repository-Namen, private Projektnamen, interne Infrastrukturdetails oder Kundeninformationen veröffentlichen oder ableiten.

Im Zweifel generische Beispiele verwenden: `example-app`, `updates.example.com`, `api.example.com`.

---

## Update-Reifegrade

**Stufe 1 — Manueller Update-Hinweis**
Die App prüft eine öffentliche Versionsdatei und zeigt einen Download-Link.

**Stufe 2 — Geführter Download**
Die App zeigt das Changelog, lädt den Installer herunter und verifiziert ihn.

**Stufe 3 — Automatischer Updater**
Die App lädt herunter, verifiziert und installiert Updates über einen vertrauenswürdigen Mechanismus.

**Stufe 4 — Sicheres Release-System**
Signierung, Checksummen, erzwungene Updates, Rollback und Release-Kanäle sind enthalten.

**Stufe 5 — Kommerzielle Distribution**
Lizenzprüfung, Staged Rollout, Kundenkanäle und Enterprise-Richtlinien sind enthalten.

Die meisten Projekte starten bei Stufe 1.

---

## Standard Phase-1-Architektur

```text
Anwendung
↓ lädt
https://updates.example.com/example-app/platform/latest.json
↓ vergleicht Version
Update-Dialog
↓ öffnet
Download-URL
```

Die Anwendung muss ihre App-ID, die installierte Version, die Plattform und den Update-Kanal kennen. Sie lädt das Manifest, vergleicht Versionen, prüft ob die installierte Version unter der Mindestversion liegt, zeigt ein Changelog und öffnet den Download-Link.

---

## Beispiel-Manifest

```json
{
  "app": "example-app",
  "platform": "macos",
  "latestVersion": "1.0.1",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.0.1-macos.dmg",
  "changelog": [
    "Update-Fenster hinzugefügt",
    "Einstellungen verbessert",
    "Startproblem behoben"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

---

## Hinweise für API-Clients

Bei Apps die von externen APIs abhängen — zum Beispiel Wetter-Apps, Finanz-Apps, Karten-Apps, KI-Clients oder SaaS-Clients — muss der Update-Plan auch API-Kompatibilität prüfen.

Fragen stellen: Nutzt die App externe API-Keys, Backend-Endpunkte, Remote-Konfiguration, Rate-Limits oder anbieterspezifische SDKs? API-Keys dürfen nicht in Beispielen auftauchen. Wenn die App brechen kann wenn sich ein API-Vertrag ändert, `minimumVersion`, `apiCompatibilityVersion` oder ein Remote-Konfigurations-Manifest empfehlen.

---

## Fragen vor der Implementierung

1. Um welchen Typ von Anwendung handelt es sich?
2. Welche Plattformen werden jetzt unterstützt?
3. Welche Plattformen sind später geplant?
4. Ist die Anwendung Open Source oder Closed Source?
5. Sind Downloads öffentlich, privat, lizenziert oder intern?
6. Soll Phase 1 nur einen Update-Hinweis zeigen?
7. Ist ein manueller Installer akzeptabel?
8. Gibt es bereits einen Release-Server, eine Website, ein CDN oder einen GitHub-Releases-Workflow?
9. Braucht die App erzwungene Updates?
10. Braucht die App Content-Updates getrennt von Binär-App-Updates?
11. Hängt die App von externen APIs ab?
12. Braucht die App API-Kompatibilitätsprüfungen?
13. Gibt es rechtliche, Datenschutz- oder Sicherheitsanforderungen?
14. Wer darf Releases veröffentlichen?
15. Ist Code-Signierung bereits vorhanden?

---

## Ausgabeformat für KI-Agenten

Nach der Analyse immer in dieser Reihenfolge ausgeben:

1. Kurze Projektinterpretation
2. Empfohlener Update-Reifegrad
3. Vorgeschlagene Architektur
4. Phasen-Roadmap
5. Sicherheitshinweise
6. Checklisten
7. Erforderliche Entscheidungen
8. Nächster konkreter Schritt

---

## Was nicht zuerst bauen

Keinen komplexen Auto-Updater bauen wenn das Projekt noch keinen stabilen Release-Prozess hat.

Keine Delta-Updates zu Beginn.

Keinen Lizenzserver bauen wenn kommerzielle Lizenzierung noch keine aktuelle Anforderung ist.

Keine versteckte Hintergrundinstallation ohne dass Plattform, Signierung und Nutzer-Vertrauensmodell klar sind.

Keine privaten GitHub-Repositories als öffentliche Kunden-Update-Quelle verwenden wenn das das Einbetten von Access-Tokens in die ausgelieferte App erfordern würde.

---

## Öffentlichkeitsregel

Für öffentliche Dokumentation und öffentliche Repositories nur öffentliche und generische Informationen erwähnen. Keine privaten Projekte, private Repository-URLs, Kundennamen, NDA-Informationen oder interne Infrastruktur.

Ist die Sichtbarkeit eines Projekts unbekannt, als privat behandeln und nicht erwähnen.

Diese Regel gilt für alle Ausgaben, alle Empfehlungen und alle Beispiele.

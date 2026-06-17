# 14. Häufige Fragen (FAQ)

---

## Allgemein

### Brauche ich einen eigenen Update-Server?

Nicht unbedingt. Eine statische JSON-Datei auf normalem Webhosting reicht für Phase 1.

Erst wenn Lizenzprüfung, Rollout-Kontrolle oder Statistiken nötig sind, ist eine eigene API sinnvoll.

### Soll ich zuerst einen Auto-Updater bauen?

Normalerweise nein. Mit einem manuellen Update-Hinweis (Phase 1) starten. Das ist einfacher, sicherer und in den meisten Projekten ausreichend.

### Kann ich GitHub Releases als Update-Quelle nutzen?

Ja, für öffentliche Open-Source-Apps. Für Closed-Source-Apps mit privatem Repository: nur wenn kein Access-Token in der App eingebettet werden muss.

---

## Sicherheit

### Reichen Checksummen?

Checksummen erkennen Beschädigungen und einfache Manipulationen. Für ernsthafte Sicherheit ist zusätzlich Code-Signierung nötig.

### Was wenn der Update-Server kompromittiert wird?

Bei Code-Signierung: Der Angreifer kann keine signierten Binärdateien erstellen ohne den Private Key. Die App würde manipulierte Downloads ablehnen.

Ohne Signierung: Checksummen helfen nur wenn das Manifest selbst nicht manipuliert wird.

### Muss ich HTTPS erzwingen?

Ja. HTTP ermöglicht Man-in-the-Middle-Angriffe die Downloads durch Malware ersetzen.

---

## Technisch

### Wie vergleiche ich Versionsnummern richtig?

Nicht als String — `"1.10.0"` wäre als String kleiner als `"1.9.0"`.

Semver aufteilen und numerisch vergleichen:
```text
"1.10.0" → [1, 10, 0]
"1.9.0" → [1, 9, 0]
Vergleich: 10 > 9 → 1.10.0 ist neuer
```

### Wie oft soll die App auf Updates prüfen?

**Beim App-Start** ist der Standard. Nicht bei jedem API-Request oder im Sekundentakt.

Mögliche Strategien:
- Nur beim Start
- Beim Start + alle 24 Stunden im Hintergrund
- Manuell auf Nutzer-Anfrage

### Was wenn der Update-Server nicht erreichbar ist?

Die App soll still scheitern. Kein Absturz, kein Blockieren des Starts, keine Fehlermeldung wenn der Nutzer gerade kein Internet hat.

---

## Mobile

### Was ist mit mobilen Apps?

Binär-Updates gehen fast immer über App Stores (Apple App Store, Google Play).

Das eigene Backend kann trotzdem nützlich sein für:
- Mindestversions-Prüfung und Weiterleitung zum Store
- Content-Updates
- Remote-Konfiguration
- API-Kompatibilitätsprüfungen

### Kann ich Updates außerhalb des App Stores ausliefern?

Bei iOS: Nein, außer für interne Enterprise-Distribution.
Bei Android: Ja (Side-Loading), aber mit Einschränkungen und Sicherheitsrisiken.

---

## Spieleentwicklung

### Brauche ich für ein Spiel zwei separate Update-Systeme?

Nicht unbedingt, aber empfohlen. Client-Updates (Programm) und Content-Updates (Level, Assets) haben verschiedene Frequenzen und Größen.

### Wie groß darf ein Content-Patch sein?

So klein wie möglich. Delta-Patching (nur geänderte Dateien) reduziert die Größe für Nutzer signifikant. Erst bei wirklichem Bedarf implementieren.

---

## Nächster Schritt

→ Weiter mit [`17-Glossar.md`](17-Glossar.md)

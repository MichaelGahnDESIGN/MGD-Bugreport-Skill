# 11. GitHub Releases

GitHub Releases ist eine nützliche Option für öffentliche Open-Source-Tools und für interne Test-Releases.

---

## Wann GitHub Releases nutzen?

**Geeignet:**
- Öffentliche Open-Source-Projekte
- Interne Entwickler-Tools
- Beta-Releases für Tester
- Apps deren Code bereits öffentlich ist

**Nicht geeignet:**
- Kommerzielle Apps mit privatem Quellcode wenn dafür Tokens in der App eingebettet werden müssen
- Apps die privates Source-Hosting als Update-Quelle brauchen

---

## Empfohlenes Muster für Open-Source-Apps

```text
Öffentliches Repository
↓
Release erstellen (git tag + gh release create)
↓
Installer hochladen als Release-Asset
↓
SHA-256 berechnen und in Release-Notes oder eigenem Manifest eintragen
↓
App lädt Manifest (eigene URL oder GitHub API)
```

---

## Manifest via GitHub Releases API

GitHub bietet eine API um Release-Informationen abzurufen:

```text
GET https://api.github.com/repos/{owner}/{repo}/releases/latest
```

Antwort enthält `tag_name`, `assets` mit Download-URLs und `body` für Release-Notes.

**Vorteil:** Kein eigener Update-Server nötig.

**Nachteil:** Abhängigkeit von GitHub-Verfügbarkeit, Rate-Limits ohne Token.

---

## Empfohlenes Muster für Closed-Source-Apps

Für Closed-Source-Apps einen getrennten öffentlichen Update-Endpunkt nutzen:

```text
Privates Repository (Code bleibt privat)
↓
Build-Pipeline (CI/CD)
↓
Installer auf eigenen Server hochladen (kein GitHub nötig)
↓
Öffentliches Manifest aktualisieren
↓
App lädt öffentliches Manifest
```

**Warum:** Ein privates GitHub-Repository als direkte Update-Quelle würde erfordern, dass ein Personal Access Token in die ausgelieferte App eingebettet wird. Das ist ein Sicherheitsrisiko — der Token könnte extrahiert werden.

---

## GitHub Actions für automatisierte Releases

```yaml
# Beispiel-Workflow-Schritt (vereinfacht)
name: Release
on:
  push:
    tags:
      - 'v*'
jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: make build
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: dist/*
```

---

## Nächster Schritt

→ Weiter mit [`12-Self-Hosted-Updates.md`](12-Self-Hosted-Updates.md)

## GitHub-Integration

Die GitHub-Integration ist eine optionale Erweiterung des Feedback-Hubs. Sie ermöglicht die automatische Erstellung von GitHub Issues aus eingegangenen Bug-Reports und Ideen.

> **Hinweis:** Diese Funktion ist optional. Der Agent MUSS die folgenden Fragen stellen, bevor eine Integration implementiert wird.

---

### Fragen, die der Agent stellen MUSS

1. Soll GitHub für die Projektverwaltung genutzt werden?
2. In welchem Repository sollen Issues erstellt werden? (z. B. `example-org/example-app`)
3. Ist das Repository **öffentlich** oder **privat**?
4. Sollen Bug-Reports **automatisch** als Issues erstellt werden oder nur manuell?
5. Sollen Ideen ebenfalls automatisch als Issues erstellt werden?
6. Welche **Labels** sollen verwendet werden? (Standard-Labels empfohlen, siehe unten)
7. Sollen GitHub-Webhooks genutzt werden, um den Status in der App zu aktualisieren?

---

### Ablauf: Bug → GitHub Issue

```mermaid
graph TD
    A[Nutzer meldet Bug] --> B[Report wird im Backend gespeichert]
    B --> C{Automatisch oder manuell?}
    C -->|Automatisch| D[Backend ruft GitHub API auf]
    C -->|Manuell| E[Admin klickt „GitHub Issue erstellen"]
    E --> D
    D --> F[GitHub Issue wird erstellt]
    F --> G[Issue-URL wird im Backend gespeichert]
    G --> H[Link erscheint in der Admin-Oberfläche]
```

---

### Empfohlenes Label-System

#### Typ-Labels
| Label | Beschreibung |
|-------|-------------|
| `bug` | Fehler in der Software |
| `idea` | Feature-Wunsch oder Verbesserungsvorschlag |
| `feedback` | Allgemeines Nutzer-Feedback |
| `help` | Hilfe-Anfrage |

#### Prioritäts-Labels
| Label | Bedeutung |
|-------|-----------|
| `priority:low` | Geringer Handlungsbedarf |
| `priority:medium` | Normaler Handlungsbedarf |
| `priority:high` | Hoher Handlungsbedarf |
| `priority:critical` | Sofortiger Handlungsbedarf |

#### Status-Labels
| Label | Bedeutung |
|-------|-----------|
| `status:new` | Neu eingegangen, noch nicht bearbeitet |
| `status:in-progress` | Wird aktuell bearbeitet |
| `status:fixed` | Behoben und getestet |

---

### GitHub Issue erstellen — API-Beispiel

```http
POST https://api.github.com/repos/example-org/example-app/issues
Authorization: Bearer GITHUB_TOKEN
Content-Type: application/json
```

```json
{
  "title": "[Bug] App stürzt beim Öffnen der Kamera ab",
  "body": "**Beschreibung:**\nDie App stürzt reproduzierbar ab, wenn die Kamera-Funktion geöffnet wird.\n\n**Schritte zum Reproduzieren:**\n1. App öffnen\n2. Auf „Kamera" tippen\n3. App stürzt ab\n\n**Erwartetes Verhalten:**\nKamera-View wird angezeigt.\n\n**Technische Daten:**\n- Gerät: iPhone 14 Pro\n- OS: iOS 17.2\n- App-Version: 2.3.1\n- Build: 231\n\n*Automatisch erstellt über Feedback-Hub*",
  "labels": ["bug", "priority:high", "status:new"]
}
```

**Antwort (gekürzt):**
```json
{
  "id": 1234567890,
  "number": 42,
  "html_url": "https://github.com/example-org/example-app/issues/42",
  "title": "[Bug] App stürzt beim Öffnen der Kamera ab",
  "state": "open"
}
```

---

### Webhook: GitHub → App (Status-Sync)

Wenn ein GitHub Issue geschlossen wird, kann der Bug-Status in der App automatisch aktualisiert werden:

```json
// GitHub sendet POST an: https://api.example.com/webhooks/github
{
  "action": "closed",
  "issue": {
    "number": 42,
    "html_url": "https://github.com/example-org/example-app/issues/42",
    "title": "[Bug] App stürzt beim Öffnen der Kamera ab"
  }
}
```

Das Backend sucht den Bug anhand der Issue-URL und setzt den Status auf `Behoben`.

---

### Sicherheitshinweis

> **KRITISCH:** Der GitHub-Token darf **niemals** in der Client-App gespeichert oder verwendet werden.

- Der Token gehört **ausschließlich** ins Backend
- Die Client-App kommuniziert **nur mit dem eigenen Backend** (`api.example.com`)
- Das Backend agiert als **Proxy** zur GitHub API
- Den Token als Umgebungsvariable speichern, niemals in den Quellcode committen

```
# .env (Backend)
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
GITHUB_REPO=example-org/example-app
```

---

→ Weiter mit [wiki/13-Datenschutz.md](13-Datenschutz.md)

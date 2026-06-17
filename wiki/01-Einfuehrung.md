# 01. Einführung

Der MGD App Updater Skill ist ein praktischer Leitfaden für die Planung von Software-Update-Systemen mit KI-Agenten.

Er ist keine fertige Bibliothek und kein Framework. Er ist ein Planungs-Skill, der einem Agenten hilft, **zuerst die richtige Update-Strategie zu wählen** — bevor Code geschrieben wird.

---

## Die Grundidee

```text
Installierte App
↓
Update-Manifest prüfen
↓
Update-Dialog zeigen
↓
Download oder Installation
```

Ein nützliches erstes Update-System kann sehr einfach sein: Die App prüft eine JSON-Datei, vergleicht Versionen und zeigt einen Download-Link.

---

## Warum Planung wichtig ist

Update-Systeme können Software auf dem Computer eines Nutzers ersetzen. Das macht sie sicherheitskritisch.

Ein schlecht implementierter Updater kann:
- zu einem Angriffspunkt für Dritte werden
- Nutzerdaten beschädigen
- Installationen in einem unbenutzbaren Zustand hinterlassen
- durch fehlerhafte Signierungsprüfung kompromittiert werden

Dieser Skill sorgt dafür, dass die erste Implementierung einfach und sicher ist — und bereitet spätere Sicherheitsmaßnahmen wie Checksummen, Signaturen, erzwungene Updates und Rollback schrittweise vor.

---

## Wie dieser Skill aufgebaut ist

| Bereich | Inhalt |
|---------|--------|
| `skill/SKILL.md` | Kern-Anweisungen für KI-Agenten |
| `wiki/` | Vollständiges Handbuch (diese Dateien) |
| `examples/` | Konkrete Beispiele pro Plattform |
| `checklists/` | Fertige Checklisten zum Abhaken |
| `templates/` | JSON-Templates und Release-Prozesse |

---

## Für wen ist dieser Skill?

**Entwickler** die ein Update-System einbauen wollen, aber nicht wissen wo sie anfangen sollen.

**KI-Agenten** (Claude Code, ChatGPT Codex, Cursor, Windsurf, Gemini CLI) die ein Projekt analysieren und eine Update-Roadmap erstellen sollen.

**Indie-Entwickler** die ohne Update-Erfahrung ein professionelles Ergebnis erreichen wollen.

---

## Nächster Schritt

→ Weiter mit [`02-Grundlagen.md`](02-Grundlagen.md)

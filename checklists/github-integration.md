## Checkliste GitHub-Integration

- [ ] GitHub-Repository ausgewählt (öffentlich oder privat?)
- [ ] Entschieden: Bugs als Issues automatisch? Ideen als Issues automatisch?
- [ ] GitHub-Token erstellt (Fine-grained PAT mit issues:write)
- [ ] Token nur im Backend gespeichert (Umgebungsvariable)
- [ ] Token niemals im Client-Code oder Repository
- [ ] Labels angelegt: bug, idea, feedback, priority:low, priority:medium, priority:high, priority:critical
- [ ] Milestone für aktuelle Version erstellt
- [ ] Webhook konfiguriert (Issue closed → Status in eigener DB aktualisieren)
- [ ] Rate-Limiting berücksichtigt (GitHub: 5000 Requests/Stunde)
- [ ] Fallback wenn GitHub nicht erreichbar (Report trotzdem speichern)
- [ ] Duplikat-Prüfung (gleicher Titel/Beschreibung → kein zweites Issue)

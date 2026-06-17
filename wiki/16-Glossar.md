## Glossar

Begriffsdefinitionen für den MGD-Bugreport-Skill und verwandte Konzepte, von A bis Z.

---

**Admin-Bereich**
Die Verwaltungsoberfläche für eingegangene Reports, Ideen, Feedbacks und Hilfe-Anfragen. Nur für autorisierte Teammitglieder zugänglich. Bietet Funktionen wie Statusverwaltung, Zuweisung und GitHub-Sync.

---

**Anhang**
Eine Datei, die an einen Report angehängt wird — typischerweise ein Screenshot, ein Log-File oder ein Video. Anhänge werden separat gespeichert und im Admin-Bereich angezeigt.

---

**Bug**
Ein Fehler in der Software, der dazu führt, dass die Anwendung sich nicht wie erwartet verhält. Bugs können Abstürze, fehlerhafte Anzeigen, falsche Berechnungen oder Sicherheitsprobleme umfassen.

---

**Bug-Report**
Ein formalisierter Fehlerbericht, der eine strukturierte Beschreibung des Fehlers, Schritte zur Reproduktion, das erwartete und tatsächliche Verhalten sowie technische Daten (Gerät, OS, App-Version) enthält.

---

**DSGVO**
Datenschutz-Grundverordnung. EU-weite Verordnung (in Kraft seit Mai 2018), die den Umgang mit personenbezogenen Daten regelt. Für den Feedback-Hub relevant bei der Verarbeitung von Kontaktdaten, Screenshots und technischen Gerätedaten.

---

**Feature-Flag**
Ein konfigurierbarer Schalter im Code oder in der Konfiguration, der bestimmte Funktionen aktiviert oder deaktiviert — ohne einen neuen Build zu erstellen. Wird genutzt, um den Feedback-Hub nur für bestimmte Nutzergruppen oder Umgebungen zu aktivieren.

---

**Feedback-Hub**
Das zentrale Menü, das alle Feedback-Typen bündelt. Der Nutzer öffnet es über den Floating Button und wählt dann zwischen: Bug melden, Idee einreichen, Feedback geben oder Hilfe anfordern.

---

**Floating Button**
Ein schwebender, immer sichtbarer Button, der dauerhaft über der App-Oberfläche liegt. Er öffnet den Feedback-Hub und hat den höchsten Z-Index im gesamten UI, sodass er niemals von anderen Elementen verdeckt wird.

---

**GitHub-Sync**
Die automatische oder manuelle Erstellung von GitHub Issues aus eingegangenen Bug-Reports oder Ideen. Der Sync-Status (verknüpfte Issue-Nummer, offen/geschlossen) wird im Admin-Bereich angezeigt.

---

**html2canvas**
Eine JavaScript-Bibliothek, die den aktuellen DOM-Zustand einer Webseite als Canvas-Element rendert und so einen Screenshot ermöglicht. Limitierungen: Cross-Origin-Ressourcen und iFrames anderer Domains werden nicht erfasst.

---

**Issue**
Eine Aufgabe oder ein Ticket in GitHub. Im Kontext des Feedback-Hubs bezeichnet „Issue" einen automatisch erstellten GitHub-Eintrag, der einem Bug-Report oder einer Idee entspricht.

---

**Mindestversion**
Die älteste App-Version, die noch vom Feedback-Hub unterstützt wird. Wichtig für die Kompatibilität der API-Endpunkte und das Format der gesendeten Daten.

---

**Modal**
Ein Dialogfenster, das über der eigentlichen Anwendung erscheint und die Interaktion mit dem Hintergrund blockiert. Der Feedback-Hub öffnet sich als Modal, wenn der Floating Button betätigt wird.

---

**Opt-out**
Die Möglichkeit für den Nutzer, eine aktivierte Funktion abzuwählen. Im Kontext des Feedback-Hubs: Technische Daten und automatische Screenshots müssen per Opt-out abwählbar sein (DSGVO-Anforderung).

---

**Privacy Manifest**
Eine von Apple verpflichtend eingeführte Konfigurationsdatei (`PrivacyInfo.xcprivacy`) für iOS-Apps seit iOS 17. Sie deklariert, welche Arten von Daten die App erfasst, zu welchem Zweck und ob sie mit der Identität des Nutzers verknüpft werden.

---

**Rate-Limiting**
Die Begrenzung der Anzahl von Anfragen, die ein Nutzer oder eine IP-Adresse in einem bestimmten Zeitraum senden darf. Schützt den Feedback-Hub vor Spam und missbräuchlichen Einreichungen.

---

**Screenshot**
Eine Bildschirmaufnahme des aktuellen App-Zustands zum Zeitpunkt des Feedbacks. Hilft Entwicklern, den Kontext eines Fehlers besser zu verstehen. Muss vor dem Senden auf sensible Inhalte geprüft werden.

---

**Schweregrad**
Die Klassifizierung der Dringlichkeit eines Bugs. Übliche Stufen: `low` (gering), `medium` (mittel), `high` (hoch), `critical` (kritisch). Beeinflusst die Priorität der Bearbeitung.

---

**Webhook**
Ein automatischer HTTP-Aufruf, den ein Dienst (z. B. GitHub) an einen anderen Dienst (z. B. das eigene Backend) sendet, wenn ein bestimmtes Ereignis eintritt. Wird genutzt, um den Bug-Status in der App zu aktualisieren, wenn ein GitHub Issue geschlossen wird.

---

**Z-Index**
Eine CSS- bzw. UI-Eigenschaft, die die Überlagerungsreihenfolge von Elementen bestimmt. Ein höherer Z-Index bedeutet, dass ein Element über anderen angezeigt wird. Der Floating Button muss stets den höchsten Z-Index im gesamten UI besitzen.

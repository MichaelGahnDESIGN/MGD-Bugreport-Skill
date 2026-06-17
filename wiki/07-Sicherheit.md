# 07. Sicherheit

Update-Systeme sind sicherheitskritisch. Sie können Software auf dem Rechner des Nutzers ersetzen. Ein angreifbarer Updater ist eine direkte Remote-Code-Execution-Schwachstelle.

---

## Die goldenen Regeln

### 1. Niemals private Tokens in die App einbetten

Alles was in einer App ausgeliefert wird, kann extrahiert werden.

Das gilt für:
- GitHub Personal Access Tokens
- API-Keys und API-Secrets
- Signing-Zertifikate und Private Keys
- Lizenz-Server-Secrets
- Datenbank-Credentials
- Interne Server-Adressen

**Lösung:** Update-Server öffentlich machen. Für private Downloads: signierte temporäre URLs vom eigenen Server generieren.

### 2. Immer HTTPS

Manifeste und Downloads immer über HTTPS ausliefern.

HTTP ermöglicht Man-in-the-Middle-Angriffe die Downloads durch Malware ersetzen können.

### 3. Checksummen prüfen

Vor dem Ausführen eines heruntergeladenen Installers die Checksumme prüfen.

```text
SHA-256 aus Manifest: a3f5b2c8...
SHA-256 der heruntergeladenen Datei: lokal berechnen
Unterschied → Datei verwerfen
```

### 4. Signaturen prüfen (Phase 3+)

Checksummen prüfen Integrität. Signaturen prüfen Authentizität.

Ein Angreifer der die Manifest-Datei kontrolliert, kann auch die Checksumme manipulieren. Code-Signierung (Apple Developer, Windows Code-Signing) verhindert das.

### 5. Keine Remote-Skripte ausführen

Niemals:
- Shell-Skripte von einem Update-Server herunterladen und ausführen
- PowerShell-Befehle aus Remote-Quellen ausführen
- Bash-Skripte aus dem Internet starten

Nur verifizierte, signierte Binärdateien und Installer ausführen.

### 6. Update-URLs aus Manifest nicht blind vertrauen

Die Download-URL aus einem Manifest sollte auf eine bekannte Domain zeigen.

Empfehlung: Erlaubte Domains im App-Code hart kodieren und prüfen.

```text
Erlaubt: https://updates.example.com/...
Nicht erlaubt: https://evil.example.com/...
```

---

## Angriffsvektoren und Gegenmaßnahmen

| Angriff | Gegenmaßnahme |
|---------|--------------|
| Manifest-Manipulation (DNS-Spoofing) | HTTPS + Certificate Pinning |
| Download-Austausch (MITM) | HTTPS + SHA-256 Checksumme |
| Signatur-Bypass | Code-Signierung + Notarization |
| Eingebettete Secrets | Keine Secrets in die App |
| Rollback-Angriff (alte Version erzwingen) | `minimumVersion` im Manifest |
| Server-Kompromittierung | Signing Key getrennt vom Server |

---

## Signing Key Management

Der Private Key für Code-Signierung darf niemals:
- Im Repository liegen
- In CI/CD-Logs auftauchen
- In der App eingebettet sein

Empfehlung: Key in einem Hardware-Sicherheitsmodul oder einem Secret-Management-System (z.B. Vault) speichern.

---

## Checkliste Sicherheit

- [ ] HTTPS für alle Manifest-URLs
- [ ] HTTPS für alle Download-URLs
- [ ] Keine privaten Tokens in der App
- [ ] Checksummen (SHA-256) für Downloads
- [ ] Signierung für Phase 3+
- [ ] Download-Domain whitelist in der App
- [ ] Timeout für Netzwerk-Requests
- [ ] Fehler werden geloggt aber keine Secrets geloggt
- [ ] `minimumVersion` schützt vor Rollback-Angriffen

---

## Nächster Schritt

→ Weiter mit [`08-Plattformen.md`](08-Plattformen.md)

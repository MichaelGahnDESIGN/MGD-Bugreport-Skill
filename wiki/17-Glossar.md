# 15. Glossar

Begriffe die in diesem Skill und in der Update-Entwicklung häufig vorkommen.

---

## A

**App-ID**
Eindeutiger Bezeichner einer App. Wird im Manifest verwendet um sicherzustellen dass die App ihr eigenes Update lädt.

**AppImage**
Portables Linux-App-Format das keine Installation erfordert und auf den meisten Distributionen läuft.

**Auto-Updater**
Ein Update-Mechanismus der Updates ohne Nutzerinteraktion herunterlädt und installiert (Phase 3).

---

## C

**CDN (Content Delivery Network)**
Netzwerk von Servern das Dateien schnell weltweit ausliefert. Nützlich für Update-Downloads.

**Changelog**
Liste der Änderungen in einer neuen Version. Sollte im Manifest und im Update-Dialog angezeigt werden.

**Checksum / Checksumme**
Kurzwert der aus einer Datei berechnet wird. Erlaubt die Überprüfung ob eine Datei vollständig und unverändert ist. Empfohlen: SHA-256.

**Code-Signing / Code-Signierung**
Digitale Signatur einer Datei oder App mit einem Zertifikat. Erlaubt dem Betriebssystem den Herausgeber zu verifizieren.

**Content-Update**
Update das Spielinhalte, Konfiguration, Assets oder Daten ändert — nicht das Programm selbst.

---

## D

**Delta-Update**
Update das nur die Änderungen zwischen zwei Versionen enthält statt der vollständigen Dateien. Reduziert Download-Größe.

**Download-URL**
Direkte URL zum Installer oder Paket im Update-Manifest.

---

## F

**Feature-Flag**
Schalter in der Remote-Konfiguration der Funktionen ein- oder ausschaltet ohne ein App-Update.

**Force Update / Pflicht-Update**
Update das erzwungen wird weil die installierte Version nicht mehr unterstützt wird.

**Flatpak**
Distributionsunabhängiges Linux-Paketformat mit Sandbox-Isolation.

---

## G

**Gatekeeper**
macOS-Sicherheitssystem das Apps vor dem ersten Start auf Code-Signierung und Notarisierung prüft.

---

## I

**Integrity Check / Integritätsprüfung**
Prüfung ob eine heruntergeladene Datei unverändert und vollständig ist. Meist via SHA-256.

---

## L

**latest.json**
Name der Manifest-Datei für Phase-1-Update-Systeme. Enthält aktuelle Version, Mindestversion und Download-URL.

**Lizenzserver**
Server der prüft ob ein Nutzer berechtigt ist bestimmte Versionen herunterzuladen.

---

## M

**Manifest**
JSON-Datei auf einem Server die Informationen über verfügbare Updates enthält.

**minimumVersion**
Älteste Version die noch unterstützt wird. Nutzer darunter werden zum Update aufgefordert (Pflicht).

---

## N

**Notarization / Notarisierung**
Apple-Prozess bei dem eine App von Apple automatisch auf Schadcode geprüft wird. Pflicht für öffentliche macOS-Apps.

---

## R

**Remote-Konfiguration**
Konfigurationsdatei auf einem Server die die App beim Start lädt. Erlaubt Änderungen ohne App-Update.

**Rollback**
Wiederherstellung der vorherigen App-Version wenn ein Update fehlschlägt.

**Rollout**
Schrittweise Veröffentlichung eines Updates an einen wachsenden Prozentsatz der Nutzer.

---

## S

**Semantic Versioning (Semver)**
Versionierungsschema `MAJOR.MINOR.PATCH`. MAJOR: Breaking Changes, MINOR: neue Funktionen, PATCH: Bugfixes.

**SHA-256**
Kryptographische Hash-Funktion. Erzeugt einen 64-stelligen Hex-String der eine Datei eindeutig identifiziert.

**SmartScreen**
Windows-Sicherheitsfunktion die unbekannte oder nicht signierte Dateien vor dem Ausführen warnt.

**Staged Rollout**
Schrittweise Freigabe eines Updates (z.B. zuerst 10%, dann 50%, dann 100% der Nutzer).

---

## U

**Update-Kanal**
Kategorie von Update-Releases. Typisch: `stable`, `beta`, `nightly`.

**Update-Manifest**
Siehe Manifest.

---

## V

**Version-String**
Textdarstellung einer Versionsnummer, z.B. `"1.2.3"`.

---

*Für Ergänzungen: Pull Request oder Issue auf GitHub.*

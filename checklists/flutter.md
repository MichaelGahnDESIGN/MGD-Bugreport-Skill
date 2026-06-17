## Checkliste Flutter

- [ ] http oder dio Paket eingebunden
- [ ] Platform.isIOS / Platform.isAndroid Abfrage implementiert
- [ ] Screenshot-Lösung plattformspezifisch (kein Aufruf auf iOS ohne Zustimmung)
- [ ] SystemInfo via dart:io Platform Klasse
- [ ] FeedbackButton als immer-oberste-Ebene implementiert (Overlay oder Stack)
- [ ] iOS: Privacy Manifest erstellt (PrivacyInfo.xcprivacy)
- [ ] iOS: App Store Review Guidelines §5.1 geprüft
- [ ] Android: Keine MediaProjection ohne User-Permission
- [ ] Offline-Zwischenspeicherung (SharedPreferences oder Hive)
- [ ] Lokalisierung des Formulars (AppLocalizations)

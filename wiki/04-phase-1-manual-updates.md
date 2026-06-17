# 04. Phase 1: Manual Updates

Phase 1 is the recommended start for most projects.

The app checks whether a new version exists and shows a download link. The user installs the update manually.

## Flow

```text
User starts app
↓
App checks latest.json
↓
New version found
↓
Dialog shows changelog
↓
User clicks download
↓
Browser opens installer URL
```

## Checklist

- [ ] App can read its installed version.
- [ ] App knows its platform.
- [ ] Manifest URL exists.
- [ ] Manifest uses HTTPS.
- [ ] Manifest contains latest version.
- [ ] Manifest contains minimum version.
- [ ] Manifest contains download URL.
- [ ] App compares versions correctly.
- [ ] Optional updates can be postponed.
- [ ] Forced updates are clearly explained.
- [ ] Network errors do not crash the app.

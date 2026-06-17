# 03. Architecture

A good update architecture starts simple.

```text
Application
↓ fetches
Manifest
↓ points to
Installer or package
```

## Static JSON or API?

Static JSON is enough for many early projects.

Use an API when updates depend on license status, customer groups, rollout channels, languages, regions or platform-specific rules.

## Recommended components

The app needs an update client, a manifest format, a release storage location and a documented release process.

Do not ship private access tokens in the app. If users need private downloads, use a server-side authorization flow or signed temporary URLs.

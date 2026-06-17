# Example: PHP Update API

A small PHP endpoint can return update information.

Example response:

```json
{
  "app": "example-app",
  "platform": "windows",
  "latestVersion": "1.0.1",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.0.1-windows.exe",
  "forceUpdate": false
}
```

Validate app ids, platforms and channels. Do not expose private paths or secrets.

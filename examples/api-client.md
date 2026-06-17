# Example: API Client

This example applies to applications that depend on external APIs or backend services.

Examples include weather apps, finance apps, map apps, AI clients, booking apps, shop clients and SaaS companion apps.

## Example manifest

```json
{
  "app": "example-api-client",
  "platform": "windows",
  "latestVersion": "1.4.0",
  "minimumVersion": "1.2.0",
  "apiCompatibilityVersion": "api-v3",
  "remoteConfigUrl": "https://updates.example.com/example-api-client/config.json",
  "downloadUrl": "https://updates.example.com/example-api-client/releases/example-api-client-1.4.0-windows.exe",
  "forceUpdate": false
}
```

## Good first implementation

Check version, check compatibility, show changelog, open update link and handle errors gracefully.

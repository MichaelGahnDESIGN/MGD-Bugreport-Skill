# Example: Weather App

A weather app is a good example for an API-based app.

It may depend on a weather provider, API keys, rate limits, location permissions, backend proxy endpoints, remote configuration and data format compatibility.

## Recommended phase 1 flow

```text
Weather app starts
↓
Reads installed version
↓
Checks update manifest
↓
Checks optional remote configuration
↓
Warns if installed version is too old
↓
Shows update notice
```

## Questions for Codex or Claude Code

- Is the app mobile, desktop or web?
- Is it distributed through an app store or direct download?
- Does it call the weather provider directly or through a backend proxy?
- Are API keys safely stored?
- Can an API contract change break older clients?
- Should update checks happen on startup, manually or both?

# MGD App Updater Skill

**MGD App Updater Skill** is a universal AI-agent skill for planning safe update systems for apps, programs, games and API-based clients.

It is made for ChatGPT Codex, Claude Code, Cursor and other coding agents. The skill helps an agent analyse a project before writing code and then guides it through a clear update roadmap.

The project is intentionally neutral. It can be used for open-source and closed-source projects, Flutter, Electron, Tauri, native desktop apps, mobile apps, games, SaaS clients and self-hosted software.

## Why this skill exists

Many apps eventually need updates. At first this sounds simple, but it quickly raises important questions.

How does the app know that a new version exists? Where are installers stored? What happens when an old version no longer works with an API? How are downloads verified? Should updates be optional or mandatory?

This skill helps answer those questions step by step.

## Philosophy

Plan before implementation.

Security before comfort.

Start simple, then grow safely.

Most projects should not begin with a complex auto-updater. A good first update system can simply check a public JSON file, compare versions, show a changelog and open a download link.

## Typical use cases

This skill works for desktop apps, mobile apps, weather apps, finance apps, map apps, AI clients, SaaS clients, shop clients, games, internal tools and self-hosted software.

## Skill file

The main skill lives here:

```text
skill/SKILL.md
```

## Example prompt for Codex or Claude Code

```text
Use the MGD App Updater Skill. Analyse my project and create a phase 1 update roadmap. Do not write code yet. First explain the architecture, risks, checklist and open questions.
```

## Suggested commands

```text
/updateservice analyse
/updateservice roadmap
/updateservice architecture
/updateservice phase1
/updateservice phase2
/updateservice phase3
/updateservice security
/updateservice flutter
/updateservice electron
/updateservice tauri
/updateservice game
/updateservice mobile
/updateservice api-client
/updateservice weather-app
/updateservice selfhosted
/updateservice checklist
```

## Maturity levels

| Level | Name | Description |
| --- | --- | --- |
| 1 | Manual update notice | App checks a version file and opens a download link. |
| 2 | Guided download | App shows changelog, downloads the installer and verifies it. |
| 3 | Automatic updater | App downloads, verifies and installs updates safely. |
| 4 | Secure release system | Signing, checksums, forced updates and rollback strategy. |
| 5 | Commercial delivery | License checks, staged rollout and enterprise channels. |

## Minimal update manifest

```json
{
  "app": "example-app",
  "platform": "macos",
  "latestVersion": "1.0.1",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.0.1-macos.dmg",
  "changelog": [
    "Added update check",
    "Improved settings",
    "Fixed startup issue"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

## Important security rule

Never put private GitHub tokens, API secrets, signing keys, license secrets or server credentials into a shipped app. Anything shipped to users must be treated as extractable.

## Documentation

Start with `wiki/01-introduction.md` and then continue with the phase and platform checklists.

## License

MIT License. See `LICENSE`.

## Imprint

See `IMPRESSUM.md`.

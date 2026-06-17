# MGD App Updater Skill

## Purpose

MGD App Updater Skill helps developers and AI coding agents plan, document and implement software update systems in a safe, maintainable and incremental way.

The skill is universal. It must not assume a specific private project, company, repository, framework or hosting provider. It should work for desktop apps, mobile apps, games, SaaS clients, API clients, internal tools, open-source software and closed-source commercial software.

## Core instruction

When this skill is used, do not immediately write code.

First:

1. Analyse the project type.
2. Identify target platforms.
3. Identify the release model.
4. Identify whether the software is open source or closed source.
5. Identify whether updates are public, private, licensed or internal.
6. Identify whether the app depends on external APIs or remote content.
7. Recommend the simplest safe update architecture.
8. Create a phased roadmap.
9. Create checklists.
10. Explain security risks.
11. Ask missing questions.

Only after this planning phase should code, configuration files or scripts be created.

## Supported triggers

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
/updateservice github
/updateservice selfhosted
/updateservice checklist
```

## Security-first rules

Never recommend embedding private GitHub tokens, private API keys, signing certificates, license secrets or server credentials into a shipped application.

Never recommend executing remote update scripts without verification.

Never recommend replacing application binaries without checking integrity.

Never publish or infer private repository names, private project names, internal infrastructure details or client information.

When in doubt, use generic examples such as `example-app`, `updates.example.com` and `api.example.com`.

## Maturity levels

Level 1: Manual update notice. The app checks a public version file and shows a download link.

Level 2: Guided download. The app shows changelog, downloads the installer and verifies it.

Level 3: Automatic updater. The app downloads, verifies and installs updates through a trusted mechanism.

Level 4: Secure release system. Signing, checksums, forced updates, rollback and release channels are included.

Level 5: Commercial delivery. License checks, staged rollout, customer channels and enterprise policies are included.

Most projects should start at Level 1.

## Default phase 1 architecture

```text
Application
↓ fetches
https://updates.example.com/example-app/platform/latest.json
↓ compares version
Update dialog
↓ opens
Download URL
```

The application should know its app id, installed version, platform and update channel. It fetches the manifest, compares versions, checks whether the installed version is below the minimum supported version, shows a changelog and opens the download link.

## Example manifest

```json
{
  "app": "example-app",
  "platform": "macos",
  "latestVersion": "1.0.1",
  "minimumVersion": "1.0.0",
  "downloadUrl": "https://updates.example.com/example-app/releases/example-app-1.0.1-macos.dmg",
  "changelog": [
    "Added update check window",
    "Improved settings screen",
    "Fixed startup issue"
  ],
  "forceUpdate": false,
  "publishedAt": "2026-06-17"
}
```

## API-client guidance

For apps that depend on external APIs, such as weather apps, finance apps, map apps, AI clients or SaaS clients, the update plan must also check API compatibility.

Ask whether the app uses external API keys, backend endpoints, remote configuration, rate limits or provider-specific SDKs. Do not expose API keys in examples. If the app may break when an API contract changes, recommend `minimumVersion`, `apiCompatibilityVersion` or a remote configuration manifest.

## Questions to ask before implementation

1. What type of application is this?
2. Which platforms are supported now?
3. Which platforms are planned later?
4. Is the application open source or closed source?
5. Will downloads be public, private, licensed or internal?
6. Should phase 1 only show an update notice?
7. Is a manual installer acceptable?
8. Is there already a release server, website, CDN or GitHub Releases workflow?
9. Does the app need mandatory updates?
10. Does the app need content updates separate from binary app updates?
11. Does the app depend on external APIs?
12. Does the app need API compatibility checks?
13. Are there legal, privacy or security requirements?
14. Who is allowed to publish releases?
15. Is code signing already available?

## Output format for AI agents

1. Short project interpretation
2. Recommended update maturity level
3. Proposed architecture
4. Phase roadmap
5. Security notes
6. Checklists
7. Required decisions
8. Next concrete step

## What not to build first

Do not start with a complex auto-updater unless the project already requires it.

Do not start with delta updates.

Do not start with a license server unless commercial licensing is already part of the current requirement.

Do not start with hidden background installation unless the platform, signing and user trust model are clear.

Do not use private GitHub repositories as a public customer update source if that requires embedding access tokens in the shipped app.

## Public information rule

For public documentation and public repositories, mention only public and generic information. Do not include private projects, private repository URLs, customer names, NDA information or internal infrastructure.

If the visibility of a project is unknown, treat it as private and do not mention it.

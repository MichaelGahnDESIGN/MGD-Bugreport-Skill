# 02. Basics

There are two common update types.

## App updates

App updates replace the program itself. Examples are `.dmg`, `.exe`, `.msi`, `.AppImage`, mobile app releases or game client builds.

## Content updates

Content updates change data loaded by the app. Examples are game levels, quests, translations, templates, remote configuration, feature flags or API compatibility metadata.

Keep app updates and content updates separate whenever possible.

## Versioning

Use predictable versions such as `1.0.0`, `1.0.1` and `1.1.0`.

A useful manifest contains `latestVersion` and `minimumVersion`.

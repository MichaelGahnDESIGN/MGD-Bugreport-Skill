# 11. GitHub Releases

GitHub Releases can be useful for public open-source tools and internal testing.

For private commercial apps, do not ship private GitHub tokens inside the app.

Recommended pattern for private source code:

```text
Private repository
↓
Build release
↓
Publish installer to controlled public update endpoint
↓
App checks public manifest
```

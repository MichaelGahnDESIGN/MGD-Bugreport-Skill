# 07. Security

Update systems are security-sensitive.

## Golden rules

Do not embed private tokens in shipped apps.

Do not store signing keys in the repository.

Do not execute remote scripts without verification.

Do not accept update files without integrity checks.

Do not trust user-controlled update URLs.

Serve manifests and downloads through HTTPS.

Anything shipped inside an app can be extracted.

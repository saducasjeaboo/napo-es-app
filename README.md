# Napo ES — Android apps

Build configuration for the **Napo Elementary School** Android apps:

| App | Package | Download asset |
|---|---|---|
| Napo ES (public site) | `je.site.napoelementaryschool.twa` | `napo-es.apk` |
| NES Admin (admin panel) | `je.site.napoelementaryschool.admin` | `nes-admin.apk` |

Both are Trusted Web Activities: a thin native shell around
`https://napoelementaryschool.site.je` using the visitor's own Chrome engine, so
the site's service worker keeps working (visited pages stay readable offline,
and `offline.php` / `admin/offline.php` show when nothing is cached).

## How a build happens

`.github/workflows/android.yml` runs on every push to `main` and on `v*` tags:

1. installs JDK 17 + the Android SDK,
2. builds both TWAs with Bubblewrap using the release keystore from repo secrets,
3. signs them, prints the signing SHA-256,
4. attaches `napo-es.apk`, `nes-admin.apk` and `assetlinks.json` to the release.

Download links (stable — always the newest release):

- https://github.com/OWNER/napo-es-app/releases/latest/download/napo-es.apk
- https://github.com/OWNER/napo-es-app/releases/latest/download/nes-admin.apk

## Required secrets

| Secret | Meaning |
|---|---|
| `ANDROID_KEYSTORE_B64` | base64 of `android.keystore` |
| `KEYSTORE_PASSWORD` | keystore password |
| `KEY_PASSWORD` | key password (same value) |
| `KEY_ALIAS` | key alias (`napo`) |

Losing the keystore means Android refuses to install an update over an existing
install (signature mismatch) — back it up.

## Site verification (removes the address bar)

Upload `assetlinks.json` to `https://napoelementaryschool.site.je/.well-known/assetlinks.json`.
It must list both package names with the SHA-256 printed by the build.

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

Minimum Android version: 7.0 (API 24), required by `androidx.browser` 2.7.4.

## Download links

Stable — always the newest release:

- https://github.com/saducasjeaboo/napo-es-app/releases/latest/download/napo-es.apk
- https://github.com/saducasjeaboo/napo-es-app/releases/latest/download/nes-admin.apk
- https://github.com/saducasjeaboo/napo-es-app/releases/latest/download/assetlinks.json

The school site's `install.php` and `admin/install-app.php` render these from
`APP_APK_URL` / `APP_ADMIN_APK_URL` in `config/config.php`.

## How a build happens

`.github/workflows/android.yml` runs on every push to `main` and on demand:

1. installs JDK 17 + the Android SDK,
2. builds both TWAs with Bubblewrap using the release keystore from repo secrets,
3. signs them and prints the signing SHA-256,
4. attaches `napo-es.apk`, `nes-admin.apk` and `assetlinks.json` to a new release
   tagged `app-v<run_number>` (marked latest).

### Build-time workarounds

These exist because of how the hosted runner and the school's host behave:

- `android-actions/setup-android` is pinned to `platform-tools`; its default
  package list still asks for `tools`, which no longer exists in the SDK repo.
- Bubblewrap only accepts an SDK that has a `tools` or `bin` folder at its root,
  so the workflow symlinks `$ANDROID_HOME/bin` to the versioned `cmdline-tools`.
- Bubblewrap reads the version name from the manifest's `appVersion` key and
  blocks on a prompt unless it equals the current `appVersionCode`; the workflow
  writes both from the run number and answers the "regenerate project?" prompt.
- The icon and web-manifest URLs point at copies in `assets/`, because the school
  host answers automated requests with its `aes.js` challenge page instead of the
  file. Chrome still reads the real `/manifest.json` from the site at runtime.
  If the site's manifest changes materially (name, colours, start URL), copy the
  new file into `assets/` again.

## Required secrets

| Secret | Meaning |
|---|---|
| `ANDROID_KEYSTORE_B64` | base64 of `android.keystore` |
| `KEYSTORE_PASSWORD` | keystore password |
| `KEY_PASSWORD` | key password (same value) |
| `KEY_ALIAS` | key alias (`napo`) |

Losing the keystore means Android refuses to install an update over an existing
install (signature mismatch) — back it up.

## Signing certificate

SHA-256: `F6:CE:D3:AF:75:FE:5A:76:07:FE:9A:B2:3B:99:0E:CC:33:B9:9A:D6:90:BE:AD:45:21:B0:78:54:6B:4E:9C:53`

## Site verification (removes the address bar)

Upload the release's `assetlinks.json` to
`https://napoelementaryschool.site.je/.well-known/assetlinks.json`. It lists both
package names with the fingerprint above. Until it is uploaded the apps work but
show Chrome's address bar inside the app.

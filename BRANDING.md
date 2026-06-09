<!-- SPDX-License-Identifier: GPL-2.0-or-later -->
# Guru Desktop — branding fork notes

This is a rebrand fork of the **Nextcloud Desktop client**, pinned at upstream tag
`v33.0.3`. It is the V0 sync client for the Guru platform
(server side: `sync.guru.beanguru.com`). Product working name: **Guru**
(will become **Ortura** — see the sweep section below).

> Scope of the fork is deliberately tiny: **name, icon, default server URL, brand
> colour, bundle id.** Sync engine, FileProvider, conflict resolution, and the
> updater are **unmodified** upstream. If sync misbehaves, fix the *server's*
> protocol response, not this client.

## Repo / branch model

| Remote | URL |
|---|---|
| `upstream` | `https://github.com/nextcloud/desktop.git` |
| `origin` | `git@github.com:bgbjako/desktop.git` (our fork) |

| Branch | Purpose |
|---|---|
| `main` | upstream baseline, pinned at `v33.0.3` (rebase target for future upstream bumps) |
| `guru` | our rebrand commits on top of `v33.0.3` — **this is what we build/ship** |

## Every brand change lives in two places

### 1. `NEXTCLOUD.cmake` — all brand strings (grep `# BRAND`)

Run `grep -n '# BRAND' NEXTCLOUD.cmake` to see every overridden line. Current set:

| Variable | Stock value | Guru value |
|---|---|---|
| `APPLICATION_NAME` | `Nextcloud` | `Guru` |
| `APPLICATION_SHORTNAME` | `Nextcloud` | `Guru` |
| `APPLICATION_EXECUTABLE` | `nextcloud` | `guru` |
| `APPLICATION_DOMAIN` | `nextcloud.com` | `guru.beanguru.com` |
| `APPLICATION_VENDOR` | `Nextcloud GmbH` | `BeanGuru` |
| `APPLICATION_UPDATE_URL` | nextcloud updater URL | `https://guru.beanguru.com/` — inert (V0 disables auto-update at build) but **must be non-empty**: `config.h.in` uses `#cmakedefine` and `theme.cpp` returns it unconditionally, so `""` fails to compile |
| `APPLICATION_SERVER_URL` | *(empty)* | `https://sync.guru.beanguru.com` |
| `APPLICATION_SERVER_URL_ENFORCE` | `ON` (unchanged) | `ON` — client may **only** connect to the URL above (BeanGuru-only V0 build) |
| `APPLICATION_REV_DOMAIN` | `com.nextcloud.desktopclient` | `com.beanguru.guru` (macOS bundle id) |
| `LINUX_PACKAGE_SHORTNAME` | `nextcloud` | `guru` |
| `NEXTCLOUD_BACKGROUND_COLOR` | `#0082c9` | `#3575B8` (BeanGuru brand blue — wizard header) |

`APPLICATION_ICON_NAME` is left as `${APPLICATION_SHORTNAME}`, so it resolves to
`Guru` and the build looks for `theme/colored/Guru-*.svg` (below).

**Deliberately NOT changed:** `THEME_CLASS` (`NextcloudTheme`) is a C++ class name,
not user-visible — renaming it needs a matching source rename, out of scope for a
minimal rebrand. `APPLICATION_VIRTUALFILE_SUFFIX` is left `nextcloud` (VFS is not
used in V0).

### 2. `theme/` — icon assets (BeanGuru mark, interim until Ortura)

| File | Used by | Notes |
|---|---|---|
| `theme/colored/Guru-icon.svg` | macOS `.icns`, Windows `.ico`, Linux PNGs | App/Dock icon — BeanGuru mark (white) on a brand-blue tile |
| `theme/colored/Guru-sidebar.svg` | macOS Finder sidebar (template) | BeanGuru mark, monochrome (macOS tints it) |
| `theme/colored/Guru-w10startmenu.svg` | Windows start-menu tile PNGs | BeanGuru mark, white |
| `theme/guru.VisualElementsManifest.xml` | Windows start tile | Renamed from `nextcloud.VisualElementsManifest.xml`; references the generated `*-Guru-w10startmenu.png` |

These reuse the **BeanGuru logo mark** (extracted from the main app) on a brand-blue
tile — legit enough for the team; swapped for final Ortura art at the rename. The tray/sync-status overlay glyphs
(`theme/white/`, `theme/black/` `state-*`) are upstream and **left as-is** for V0.

The original `Nextcloud-*` assets are left in `theme/colored/` untouched (inert
once `APPLICATION_ICON_NAME` points at `Guru`).

## How to build (macOS)

From `admin/osx/mac-crafter`:

```bash
swift run mac-crafter build --app-name Guru --disable-auto-updater
```

- `--app-name Guru` **must match** `APPLICATION_NAME` — mac-crafter copies
  `<app-name>.app` out of the build image, so a mismatch fails the copy.
- `--disable-auto-updater` — V0 ships without Sparkle.
- Add `--full-rebuild` after changing `NEXTCLOUD.cmake` cache vars
  (e.g. `APPLICATION_SERVER_URL`) so a stale CMake cache doesn't keep the old value.
- Output: `admin/osx/mac-crafter/product/Guru.app` (unsigned, ad-hoc). For a `.dmg`
  use the `create-dmg` subcommand. Unsigned → recipients open via right-click → Open.

Windows `.exe` is produced via the fork's CI (KDE Craft on a Windows runner) — see
Phase 2 notes / the spec.

**Windows packaging gotcha:** the installer's app name, install folder, Start Menu
shortcut, and the exe the shortcut launches do **not** come from `NEXTCLOUD.cmake` —
they come from the upstream **`desktop-client-blueprints/nextcloud-client.py`**, which
hardcodes `displayName = "Nextcloud"`, `defines["appname"] = "nextcloud"`,
`defines["company"]`, and `applicationExecutable = "nextcloud"`. Left as-is, the
shortcut is named "Nextcloud" and targets `nextcloud.exe` while our build ships
`guru.exe` — so the app installs but can't be found or launched. The CI patches these
four lines to `Guru` / `guru` / `BeanGuru` in the "Patch upstream blueprint" step
(`.github/workflows/guru-windows.yml`).

A **second file in the same blueprint dir — `blacklist.txt` — is even more important:**
its last line strips every `bin/*.exe` except a hardcoded allow-list,
`bin/(?!(nextcloud|nextcloudcmd|QtWebEngineProcess)).*\.exe`. Because our exes are
renamed to `guru.exe` / `gurucmd.exe`, they are *not* allow-listed and the packager
**deletes the main app binary from the image** — the installer ships the Guru libs +
branding but no runnable app. The same CI step rewrites the allow-list to
`(guru|gurucmd|QtWebEngineProcess)`. macOS is unaffected (mac-crafter reads the cmake
values directly and doesn't use this blacklist).

## The Ortura rename sweep (when it lands)

Everything is concentrated so the rename is a one-place change:

1. `NEXTCLOUD.cmake`: change the `# BRAND`-tagged values — `Guru`→`Ortura`,
   `guru`→`ortura`, `com.beanguru.guru`→`com.ortura.desktop` (or final id),
   domain/vendor as decided. (`grep -n '# BRAND' NEXTCLOUD.cmake`.)
2. `theme/colored/`: rename `Guru-icon.svg` / `Guru-sidebar.svg` /
   `Guru-w10startmenu.svg` → `Ortura-*` and drop in final art.
3. `theme/guru.VisualElementsManifest.xml` → `theme/ortura.VisualElementsManifest.xml`
   (filename must equal `APPLICATION_EXECUTABLE`), update the PNG refs inside.
4. `.github/workflows/guru-windows.yml` — the "Patch upstream blueprint" step rewrites
   the blueprint's `displayName` / `appname` / `company` / `applicationExecutable`
   **and** the `blacklist.txt` exe allow-list (see the Windows packaging gotcha above).
   Update both replacement targets to the Ortura values — the brand strings *and* the
   `(guru|gurucmd|QtWebEngineProcess)` allow-list — or the Windows installer regresses
   (broken "Nextcloud" shortcut and/or a missing app binary).
5. Build with `--app-name Ortura`.

That's the whole surface. See the platform spec `NEXTCLOUD-TRANSITION-SPEC.md`
(in the main app repo) for the why and the GA path (code-signing, auto-update).

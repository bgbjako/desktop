<!-- SPDX-License-Identifier: GPL-2.0-or-later -->
# Ledge Desktop — branding fork notes

This is a rebrand fork of the **Nextcloud Desktop client**, pinned at upstream tag
`v33.0.3`. It is the V0 sync client for the Guru/Ledge platform
(server side: `sync.guru.beanguru.com`). Product working name: **Ledge**
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
| `ledge` | our rebrand commits on top of `v33.0.3` — **this is what we build/ship** |

## Every brand change lives in two places

### 1. `NEXTCLOUD.cmake` — all brand strings (grep `# LEDGE`)

Run `grep -n '# LEDGE' NEXTCLOUD.cmake` to see every overridden line. Current set:

| Variable | Stock value | Ledge value |
|---|---|---|
| `APPLICATION_NAME` | `Nextcloud` | `Ledge` |
| `APPLICATION_SHORTNAME` | `Nextcloud` | `Ledge` |
| `APPLICATION_EXECUTABLE` | `nextcloud` | `ledge` |
| `APPLICATION_DOMAIN` | `nextcloud.com` | `guru.beanguru.com` |
| `APPLICATION_VENDOR` | `Nextcloud GmbH` | `BeanGuru` |
| `APPLICATION_UPDATE_URL` | nextcloud updater URL | `https://guru.beanguru.com/` — inert (V0 disables auto-update at build) but **must be non-empty**: `config.h.in` uses `#cmakedefine` and `theme.cpp` returns it unconditionally, so `""` fails to compile |
| `APPLICATION_SERVER_URL` | *(empty)* | `https://sync.guru.beanguru.com` |
| `APPLICATION_SERVER_URL_ENFORCE` | `ON` (unchanged) | `ON` — client may **only** connect to the URL above (BeanGuru-only V0 build) |
| `APPLICATION_REV_DOMAIN` | `com.nextcloud.desktopclient` | `com.beanguru.ledge` (macOS bundle id) |
| `LINUX_PACKAGE_SHORTNAME` | `nextcloud` | `ledge` |
| `NEXTCLOUD_BACKGROUND_COLOR` | `#0082c9` | `#3575B8` (BeanGuru brand blue — wizard header) |

`APPLICATION_ICON_NAME` is left as `${APPLICATION_SHORTNAME}`, so it resolves to
`Ledge` and the build looks for `theme/colored/Ledge-*.svg` (below).

**Deliberately NOT changed:** `THEME_CLASS` (`NextcloudTheme`) is a C++ class name,
not user-visible — renaming it needs a matching source rename, out of scope for a
minimal rebrand. `APPLICATION_VIRTUALFILE_SUFFIX` is left `nextcloud` (VFS is not
used in V0).

### 2. `theme/` — placeholder icon assets (V0 = minimal, not final art)

| File | Used by | Notes |
|---|---|---|
| `theme/colored/Ledge-icon.svg` | macOS `.icns`, Windows `.ico`, Linux PNGs | App/Dock icon — blue rounded square + white "L" |
| `theme/colored/Ledge-sidebar.svg` | macOS Finder sidebar (template) | Monochrome "L" |
| `theme/colored/Ledge-w10startmenu.svg` | Windows start-menu tile PNGs | White "L" |
| `theme/ledge.VisualElementsManifest.xml` | Windows start tile | Renamed from `nextcloud.VisualElementsManifest.xml`; references the generated `*-Ledge-w10startmenu.png` |

These are **placeholders** — a recolored square with an "L", chosen over polished
art because the Ortura rename is imminent. The tray/sync-status overlay glyphs
(`theme/white/`, `theme/black/` `state-*`) are upstream and **left as-is** for V0.

The original `Nextcloud-*` assets are left in `theme/colored/` untouched (inert
once `APPLICATION_ICON_NAME` points at `Ledge`).

## How to build (macOS)

From `admin/osx/mac-crafter`:

```bash
swift run mac-crafter build --app-name Ledge --disable-auto-updater
```

- `--app-name Ledge` **must match** `APPLICATION_NAME` — mac-crafter copies
  `<app-name>.app` out of the build image, so a mismatch fails the copy.
- `--disable-auto-updater` — V0 ships without Sparkle.
- Add `--full-rebuild` after changing `NEXTCLOUD.cmake` cache vars
  (e.g. `APPLICATION_SERVER_URL`) so a stale CMake cache doesn't keep the old value.
- Output: `admin/osx/mac-crafter/product/Ledge.app` (unsigned, ad-hoc). For a `.dmg`
  use the `create-dmg` subcommand. Unsigned → recipients open via right-click → Open.

Windows `.exe` is produced via the fork's CI (KDE Craft on a Windows runner) — see
Phase 2 notes / the spec.

## The Ortura rename sweep (when it lands)

Everything is concentrated so the rename is a one-place change:

1. `NEXTCLOUD.cmake`: change the `# LEDGE`-tagged values — `Ledge`→`Ortura`,
   `ledge`→`ortura`, `com.beanguru.ledge`→`com.ortura.desktop` (or final id),
   domain/vendor as decided. (`grep -n '# LEDGE' NEXTCLOUD.cmake`.)
2. `theme/colored/`: rename `Ledge-icon.svg` / `Ledge-sidebar.svg` /
   `Ledge-w10startmenu.svg` → `Ortura-*` and drop in final art.
3. `theme/ledge.VisualElementsManifest.xml` → `theme/ortura.VisualElementsManifest.xml`
   (filename must equal `APPLICATION_EXECUTABLE`), update the PNG refs inside.
4. Build with `--app-name Ortura`.

That's the whole surface. See the platform spec `NEXTCLOUD-TRANSITION-SPEC.md`
(in the main app repo) for the why and the GA path (code-signing, auto-update).

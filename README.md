# Godot Build Action

This GitHub Action automates building Godot projects for multiple platforms using official Godot binaries and export templates.

## Highlights

- Uses official Godot 4.x releases (or preview builds on demand)
- Auto-detects latest stable 4.x when GODOT_VER isn’t provided
- Android export support (SDK/NDK/JDK setup)
- Optional Blender install for .blend imports
- Headless import with Xvfb and configurable timeout
- Optional itch.io upload via butler
- Caches Godot binary and export templates for faster builds
- Outputs version name/code detected from export_presets.cfg

## Quick start

```yaml
name: Build Game

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build (Windows)
        uses: mlm-games/godot-build-action@v1
        with:
          EXPORT_PRESET_NAME: "Windows Desktop"
```

## Full example (matrix + Android + caching + artifacts)

```yaml
name: Build Game

on:
  workflow_dispatch:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        preset: ["Windows Desktop", "Linux/X11", "macOS", "Android arm64"]

    steps:
      - uses: actions/checkout@v4

      - name: Build
        id: build
        uses: mlm-games/godot-build-action@v1
        with:
          EXPORT_PRESET_NAME: ${{ matrix.preset }}
          PROJECT_DIR: game                # if your project is in a subfolder
          GODOT_VER: ""                    # auto-detect latest stable 4.x
          DEBUG_MODE: "false"              # or "true"
          GODOT_PREVIEW_BUILDS: "false"    # set "true" to use preview builds
          EXPORT_DIR: builds
          INSTALL_BLENDER: "false"         # or "true" if your project has .blend files
          BLENDER_VERSION: "4.4.3"
          IMPORT_TIMEOUT: "60"
          VERBOSE_IMPORT: "true"
          BUTLER_UPLOAD: "false"           # set "true" to upload to itch
          BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
          ITCH_USER_SLASH_GAME: yourname/yourgame

      - name: Upload builds
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.preset }}-build
          path: ${{ steps.build.outputs.build }}
```

## Inputs

| Input | Description | Required | Default |
|------|-------------|----------|---------|
| `EXPORT_PRESET_NAME` | Name of the preset in `export_presets.cfg` to use (e.g., “Windows Desktop”, “Linux/X11”, “macOS”, “Android arm64”) | Yes | — |
| `GODOT_VER` | Godot version (e.g., `4.2.2-stable`). If empty, the latest stable 4.x is used | No | `""` |
| `PROJECT_DIR` | Path to the Godot project in the repo | No | `.` |
| `DEBUG_MODE` | Use `--export-debug` instead of `--export-release` | No | `false` |
| `GODOT_PREVIEW_BUILDS` | Use preview builds from `godot-builds` | No | `false` |
| `EXPORT_DIR` | Directory where builds are exported | No | `builds` |
| `INSTALL_BLENDER` | Install Blender (for .blend imports) | No | `false` |
| `BLENDER_VERSION` | Blender version to install | No | `4.4.3` |
| `IMPORT_TIMEOUT` | Import timeout in minutes | No | `60` |
| `VERBOSE_IMPORT` | Verbose import logs | No | `true` |
| `BUTLER_UPLOAD` | Upload built files to itch.io using butler | No | `false` |
| `BUTLER_CREDENTIALS` | Butler API key | No | `""` |
| `ITCH_USER_SLASH_GAME` | Itch target as `user/game` | No | `""` |
| `RELEASE_KEYSTORE` | Base64/ASCII-armored keystore (for Android signing). Pass encrypted content; decrypted with `KEYSTORE_PASSPHRASE` | No | `""` |
| `KEYSTORE_PASSPHRASE` | Passphrase used to decrypt `RELEASE_KEYSTORE` | No | `""` |
| `KEY_ALIAS` | Android signing key alias | No | `""` |
| `KEY_PASSWORD` | Android signing key password | No | `""` |

Tip: For Android exports, the action sets/updates the Godot editor settings to use the installed JDK and Android SDK/NDK.

## Outputs

| Output | Description |
|--------|-------------|
| `build` | Absolute path to the build output directory |
| `version_name` | Version name parsed from export presets |
| `version_code` | Version code parsed from export presets |

## Requirements

- Godot 4.x project with export presets configured
- Export preset names must match exactly
- For Android, a preset like “Android arm64” (or any preset name containing “Android”)

## Common issues

- Export Templates not found: ensure your project’s Godot version matches the templates. If auto-detection fails or rate-limit hits, set `GODOT_VER` explicitly.
- Android export failures: check that the preset exists; accept Android SDK licenses; ensure version codes/names are present in `export_presets.cfg`.
- Web/HTML5 export: produced directory is zipped automatically before butler upload.

## License

GPL-3.0-only. See LICENSE. (Will change to MIT on request, but it shouldn't matter here)

## Credits

- Thanks to the Godot team and community for official binaries and templates.
- Inspired by workflows from various open-source Godot projects.
- [yeslayla](https://github.com/yeslayla) -> special mention for the initial reference yml

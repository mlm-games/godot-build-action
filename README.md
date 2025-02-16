# Godot Build Action

This GitHub Action automates building Godot projects for multiple platforms. It uses the official Godot binaries and export templates directly from Godot's releases.

## Features

- Uses official Godot builds
- Automatically uses the recommended stable version from godotengine.org
- Supports custom Godot versions
- Debug mode support
- Custom project directory support
- Exports using your project's export presets

## Usage

### Basic Example

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
      
      - name: Build
        uses: mlm-games/build-godot-action@v1
        with:
          BINARY_NAME: my-game
          EXPORT_PRESET_NAME: windows
```

### Full Example with All Options

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
    strategy:
      matrix:
        platform: [windows, linux, mac]
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        uses: mlm-games/build-godot-action@v1
        with:
          BINARY_NAME: my-game
          EXPORT_PRESET_NAME: ${{ matrix.platform }}
          GODOT_VER: 4.2.1  # Optional: specific version
          PROJECT_DIR: game  # Optional: if project is in subdirectory
          DEBUG_MODE: true   # Optional: for debug builds
          
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.platform }}-build
          path: ${{ github.workspace }}/game/builds/
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `BINARY_NAME` | Name of the exported binary | Yes | - |
| `EXPORT_PRESET_NAME` | Name of the preset in `export_presets.cfg` to use | Yes | - |
| `GODOT_VER` | Specific Godot version to use | No | Latest recommended version |
| `PROJECT_DIR` | Location of Godot project in repository | No | "." |
| `DEBUG_MODE` | Whether to use `--export-debug` | No | false |

## Outputs

| Output | Description |
|--------|-------------|
| `build` | Path to the build output directory |

## Prerequisites

1. Your Godot project must have export presets configured
2. The export preset names must match what you specify in the action
3. Your project should be Godot 4.x compatible 

## Export Preset Configuration

1. Open your project in Godot
2. Go to Project > Export
3. Add presets for your target platforms
4. Make sure the preset names match what you specify in the action

## Common Issues

### Export Templates Not Found

This usually means there's a mismatch between your project's Godot version and the export templates. Make sure:
- Your project is using Godot 4.x
- The `GODOT_VER` matches your project version (if specified)

### Export Preset Not Found

This happens when the preset name in your workflow doesn't match the ones in your project. To fix:
1. Open your project in Godot
2. Check the exact names of your export presets
3. Use those exact names in your workflow

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the GPL 3.0 License - see the LICENSE file for details.

## Credits

- [yeslayla](github.com/yeslayla) for the initial reference yml

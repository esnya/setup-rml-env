# Setup RML Environment

This GitHub Action sets up and caches the Resonite game files and ResoniteModLoader for mod development. It's designed to work like other `setup-*` actions, focusing on environment preparation while leaving the actual build process to the user.

## Features

- 🎮 **Automated Resonite Installation**: Downloads and installs Resonite via SteamCMD
- 📦 **Smart Caching**: Caches Resonite installation to speed up subsequent runs
- 🔧 **ResoniteModLoader Setup**: Automatically installs ResoniteModLoader and dependencies
- 🔥 **ResoniteHotReloadLib Support**: Optional installation of ResoniteHotReloadLib for hot-reload development
- ✅ **Environment Verification**: Validates that all components are properly installed
- 🏷️ **Flexible Versioning**: Support for specific ResoniteModLoader and ResoniteHotReloadLib versions
- 📊 **Rich Outputs**: Provides paths and cache status for downstream steps

## License

This project/action is licensed under the [MIT License](./LICENSE).

## Inputs

| **Input**         | **Description**                                                                         | **Default**                           | **Required** |
|-------------------|-----------------------------------------------------------------------------------------|---------------------------------------|--------------|
| `resonite-path`   | Path to the directory where Resonite will be installed                                  | `${{ github.workspace }}/Resonite` | No           |
| `steam-login`     | Login credentials for SteamCMD in the format `"<username> <password>"`                  |                                       | Yes          |
| `app-id`          | Steam App ID for Resonite                                                               | `2519830`                             | No           |
| `cache-key`       | Custom cache key for the Resonite installation                                          | Auto-generated from app-id            | No           |
| `rml-version`     | ResoniteModLoader version to install (`latest` or specific version like `v2.0.0`)       | `latest`                              | No           |
| `hot-reload-lib`  | Whether to install ResoniteHotReloadLib (`true` or `false`)                             | `false`                               | No           |
| `hot-reload-version` | ResoniteHotReloadLib version to install (`latest` or specific version like `v2.1.1`) | `latest`                              | No           |

## Outputs

| **Output**        | **Description**                                                                  |
|-------------------|----------------------------------------------------------------------------------|
| `resonite-path`   | Path to the installed Resonite directory                                        |
| `libraries-path`  | Path to the Resonite Libraries directory                                        |
| `rml-libs-path`   | Path to the ResoniteModLoader libraries directory                               |
| `cache-hit`       | Whether the cache was hit for Resonite installation (`true` or `false`)        |

## Example Usage

### Basic Setup (Release Build)

```yaml
name: Build Resonite Mod

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Setup MSBuild
        uses: microsoft/setup-msbuild@v2

      - name: Setup Resonite Environment
        id: setup-resonite
        uses: esnya/setup-rml-env@v2
        with:
          steam-login: ${{ secrets.STEAM_LOGIN }}

      - name: Restore dependencies
        run: dotnet restore

      - name: Build mod (Release)
        run: dotnet build --configuration Release --no-restore

      - name: Install mod
        run: dotnet msbuild -target:Install -property:Configuration=Release

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: mod-artifacts
          path: ./bin/Release/*.dll
```

### Advanced Setup with Custom Configuration

```yaml
name: Advanced Resonite Mod Build

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Setup MSBuild
        uses: microsoft/setup-msbuild@v2

      - name: Setup Resonite Environment
        id: setup-resonite
        uses: esnya/setup-rml-env@v2
        with:
          resonite-path: ${{ github.workspace }}/MyModProject
          steam-login: ${{ secrets.STEAM_LOGIN }}
          hot-reload-lib: true
          hot-reload-version: v3.0.0-RML
          cache-key: my-custom-cache-key-v1

      - name: Use setup outputs
        run: |
          echo "Resonite installed at: ${{ steps.setup-resonite.outputs.resonite-path }}"
          echo "Libraries path: ${{ steps.setup-resonite.outputs.libraries-path }}"
          echo "Cache hit: ${{ steps.setup-resonite.outputs.cache-hit }}"

      - name: Restore dependencies
        run: dotnet restore

      - name: Build mod
        run: dotnet build --configuration Debug --no-restore

      - name: Install mod
        run: dotnet msbuild -target:Install -property:Configuration=Debug
```

### Debug Build with Hot Reload

```yaml
name: Debug Build with Hot Reload

on:
  push:
    branches: [develop, feature/*]

jobs:
  debug-build:
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Setup MSBuild
        uses: microsoft/setup-msbuild@v2

      - name: Setup Resonite Environment with Hot Reload
        id: setup-resonite
        uses: esnya/setup-rml-env@v2
        with:
          steam-login: ${{ secrets.STEAM_LOGIN }}
          hot-reload-lib: true
          hot-reload-version: latest

      - name: Restore dependencies
        run: dotnet restore

      - name: Build mod (Debug with Hot Reload)
        run: dotnet build --configuration Debug --no-restore

      - name: Install mod for development
        run: dotnet msbuild -target:Install -property:Configuration=Debug
```

## Migration from v1

If you're migrating from the previous `build-rml-mod` action, here are the key changes:

### What Changed

- **Action renamed**: `build-rml-mod` → `setup-rml-env`
- **Repository renamed**: `build-rml-mod` → `setup-rml-env`
- **Focus changed**: Now only handles environment setup, not building
- **Build system**: Uses MSBuild for .NET Framework 4.7.2 compatibility
- **Input names**: `project` → `resonite-path`, removed build-specific inputs
- **New outputs**: Added outputs for paths and cache status
- **Verification**: Added automatic verification of installed components

### Migration Steps

1. **Update action reference**:

   ```yaml
   # Before
   uses: esnya/build-rml-mod@v1

   # After
   uses: esnya/setup-rml-env@v2
   ```

2. **Update inputs**:

   ```yaml
   # Before
   with:
     project: ${{ github.workspace }}/MyProject
     configuration: Release
     target-dir: ./bin/Release
     steam-login: ${{ secrets.STEAM_LOGIN }}

   # After
   with:
     resonite-path: ${{ github.workspace }}/Resonite
     steam-login: ${{ secrets.STEAM_LOGIN }}
   ```

3. **Add build steps**:

   ```yaml
   # Add these steps after the setup action
   - name: Setup .NET
     uses: actions/setup-dotnet@v4
     with:
       dotnet-version: '9.0.x'

   - name: Setup MSBuild
     uses: microsoft/setup-msbuild@v2

   - name: Restore dependencies
     run: dotnet restore

   - name: Build mod
     run: dotnet build --configuration Release --no-restore

   - name: Install mod
     run: dotnet msbuild -target:Install -property:Configuration=Release
   ```

## ResoniteHotReloadLib Support

This action optionally supports installing [ResoniteHotReloadLib](https://github.com/Nytra/ResoniteHotReloadLib), which enables hot-reload functionality for mod development.

### Version Compatibility

- **v3.0.0-RML and later**: Includes both `ResoniteHotReloadLib.dll` and `ResoniteHotReloadLibCore.dll`
- **Earlier versions**: Only includes `ResoniteHotReloadLib.dll`

The action automatically detects the version and downloads the appropriate files.

### Usage

```yaml
- name: Setup .NET
  uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '9.0.x'

- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2

- name: Setup Resonite Environment with Hot Reload
  uses: esnya/setup-rml-env@v2
  with:
    steam-login: ${{ secrets.STEAM_LOGIN }}
    hot-reload-lib: true
    hot-reload-version: latest  # or specific version like 'v3.0.0-RML'

- name: Build Debug mod with Hot Reload
  run: |
    dotnet restore
    dotnet build --configuration Debug --no-restore
    dotnet msbuild -target:Install -property:Configuration=Debug
```

## Requirements

- **Runners**: Windows runners only (due to Resonite being Windows-specific)
- **Secrets**: Steam login credentials must be stored in repository secrets
- **Permissions**: Runner needs write access for caching

## Tips

- **Steam Credentials**: Store your Steam username and password in repository secrets as `STEAM_LOGIN` in the format `"username password"`
- **Caching**: The action automatically caches Resonite installation. Cache duration follows GitHub's cache policies
- **Version Pinning**: For reproducible builds, consider pinning `rml-version` and `hot-reload-version` to specific versions rather than using `latest`
- **.NET Framework 4.7.2**: This action targets .NET Framework 4.7.2 projects, but uses .NET 9.0 SDK's `dotnet` CLI for building
- **Hot Reload Development**: Enable `hot-reload-lib: true` for Debug builds to enable faster mod development iterations
- **Configuration Strategy**: Use Release builds for production/releases, Debug builds with hot-reload for development
- **Install Target**: Ensure your `.csproj` file has an `Install` target defined for automatic mod installation
- **Parallel Jobs**: Multiple jobs can share the same cache key for the same Resonite version
- **MSBuild Compatibility**: Uses MSBuild which is included with GitHub Actions Windows runners

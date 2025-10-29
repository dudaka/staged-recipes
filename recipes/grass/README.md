# GRASS GIS Conda Recipe

This recipe builds GRASS GIS for multiple platforms with platform-specific configurations.

## Directory Structure

```
grass/
├── meta.yaml                      # Platform dispatcher (uses Jinja2 includes)
├── linux/
│   ├── meta.yaml                 # Linux-specific recipe
│   ├── build.sh                  # Linux-specific build script
│   └── conda_build_config.yaml   # Linux-specific configurations
├── osx/
│   ├── meta.yaml                 # macOS-specific recipe
│   ├── build.sh                  # macOS-specific build script
│   └── conda_build_config.yaml   # macOS-specific configurations
├── win/
│   ├── meta.yaml                 # Windows-specific recipe (placeholder)
│   ├── bld.bat                   # Windows-specific build script (placeholder)
│   └── conda_build_config.yaml   # Windows-specific configurations
└── README.md                      # This file
```

## Key Design

**Complete Separation**: Each platform has its own `meta.yaml`, build script, and configuration. This allows:

- Different dependencies per platform
- Different build configurations
- Independent maintenance without affecting other platforms

The root `meta.yaml` acts as a dispatcher that includes the appropriate platform-specific recipe using Jinja2's `{% include %}` directive.

## Platform-Specific Details

### Linux

- Uses GCC compiler
- OpenMP enabled (`_openmp_mutex` dependency)
- GNU linker flags (`--no-as-needed`) for libiconv linking
- Full X11 support with xorg packages

### macOS

- Uses Clang compiler
- OpenMP via llvm-openmp (different from Linux)
- macOS-specific linker flags (no `--no-as-needed`)
- Uses `DYLD_LIBRARY_PATH` instead of `LD_LIBRARY_PATH`
- Different sed behavior (creates .bak files with `-i`)
- Build script configured with `--without-openmp` by default

### Windows

Currently a placeholder. Windows support requires:

1. CMake-based build (GRASS has experimental CMake support)
2. Visual Studio compiler configuration
3. Different set of dependencies (no X11, different graphics libraries)
4. Implementing `win/bld.bat` build script

## Building the Package

To build for a specific platform:

```bash
# Linux
conda build . --variants "{target_platform: linux-64}"

# macOS
conda build . --variants "{target_platform: osx-64}"
conda build . --variants "{target_platform: osx-arm64}"
```

## How It Works

The root `meta.yaml` file uses Jinja2 `{% include %}` directives to include the appropriate platform-specific recipe:

```yaml
{% if target_platform.startswith('linux') %}
{% include 'linux/meta.yaml' %}
{% elif target_platform.startswith('osx') %}
{% include 'osx/meta.yaml' %}
{% elif target_platform.startswith('win') %}
{% include 'win/meta.yaml' %}
{% endif %}
```

**How conda-build processes this:**

1. Conda-build reads the root `meta.yaml`
2. Based on `target_platform` (e.g., `linux-64`, `osx-64`, `osx-arm64`, `win-64`)
3. The appropriate platform-specific `meta.yaml` is included
4. That platform's `build.sh` or `bld.bat` is executed

**Examples:**

- Building for `linux-64`: Uses `linux/meta.yaml` → runs `linux/build.sh`
- Building for `osx-64` or `osx-arm64`: Uses `osx/meta.yaml` → runs `osx/build.sh`
- Building for `win-64`: Uses `win/meta.yaml` → runs `win/bld.bat`

## Modifying Platform-Specific Builds

Each platform has complete independence. To make changes:

- **Linux-specific changes**: Edit `linux/meta.yaml` and/or `linux/build.sh`
- **macOS-specific changes**: Edit `osx/meta.yaml` and/or `osx/build.sh`
- **Windows-specific changes**: Edit `win/meta.yaml` and/or `win/bld.bat`

**Examples of platform-specific changes:**

- Different dependencies: Edit the platform's `meta.yaml` requirements section
- Different build flags: Edit the platform's `build.sh` or `bld.bat`
- Different version pins: Edit the platform's `conda_build_config.yaml`

**Important:** Changes to one platform do NOT affect others. There's no shared code between platforms.

## Testing

After building, test the package:

```bash
grass --version
grass --tmp-project EPSG:4326 --exec g.version -rge
```

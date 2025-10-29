# Platform-Specific meta.yaml Architecture

## How It Works

```
┌─────────────────────────────────────┐
│   Root: meta.yaml (Dispatcher)      │
│                                     │
│  Uses Jinja2 {% include %} to      │
│  select platform-specific recipe    │
└──────────┬──────────────────────────┘
           │
           │ Checks target_platform
           │
    ┌──────┴──────┬─────────────┐
    ▼             ▼             ▼
┌────────┐   ┌────────┐   ┌────────┐
│ linux/ │   │  osx/  │   │  win/  │
├────────┤   ├────────┤   ├────────┤
│ meta.yaml   │ meta.yaml   │ meta.yaml
│ build.sh │  │ build.sh │  │ bld.bat│
│ conda_   │  │ conda_   │  │ conda_ │
│ build_   │  │ build_   │  │ build_ │
│ config   │  │ config   │  │ config │
└────────┘   └────────┘   └────────┘
```

## Build Flow

1. User runs: `conda build . --variants "{target_platform: linux-64}"`
2. conda-build reads root `meta.yaml`
3. Jinja2 evaluates: `target_platform.startswith('linux')` → True
4. Includes `linux/meta.yaml` (entire file contents)
5. Executes `linux/build.sh` as specified in `linux/meta.yaml`
6. Uses dependencies from `linux/meta.yaml`

## Benefits of This Approach

✅ **Complete Independence**: Each platform has its own full recipe
✅ **No Conditional Logic**: No need for `# [linux]` selectors everywhere
✅ **Easy to Read**: Each platform recipe is self-contained
✅ **Easy to Maintain**: Changes to one platform don't risk breaking others
✅ **Clear Dependencies**: Each platform explicitly lists what it needs
✅ **Platform-Specific Docs**: Can add comments specific to each platform

## When to Edit What

| What You Want to Change | File to Edit                                         |
| ----------------------- | ---------------------------------------------------- |
| Linux dependencies      | `linux/meta.yaml`                                    |
| Linux build process     | `linux/build.sh`                                     |
| macOS dependencies      | `osx/meta.yaml`                                      |
| macOS build process     | `osx/build.sh`                                       |
| Windows dependencies    | `win/meta.yaml`                                      |
| Windows build process   | `win/bld.bat`                                        |
| Version or source URL   | Update in each platform's `meta.yaml`                |
| Add new platform        | Create new directory with `meta.yaml` + build script |

## Example: Adding a Linux-Only Dependency

Edit `linux/meta.yaml`:

```yaml
requirements:
  host:
    - python
    - gdal
    - my-linux-only-lib # ← Add here
```

This won't affect macOS or Windows builds at all!

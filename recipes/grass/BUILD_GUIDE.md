# Building GRASS for Different Platforms

## Quick Reference Commands

### Build for Linux (64-bit)
```bash
conda build . --variants "{target_platform: linux-64}"
```

### Build for macOS Intel (64-bit)
```bash
conda build . --variants "{target_platform: osx-64}"
```

### Build for macOS Apple Silicon (ARM64)
```bash
conda build . --variants "{target_platform: osx-arm64}"
```

### Build for Windows (64-bit) - When implemented
```bash
conda build . --variants "{target_platform: win-64}"
```

## Testing Locally

After building, install and test:

```bash
# Install the locally built package
conda install -c local grass

# Test it
grass --version
grass --tmp-project EPSG:4326 --exec g.version -rge
```

## Checking Which Recipe Will Be Used

You can preview what conda-build will do by rendering the recipe:

```bash
# See the rendered recipe for Linux
conda render . --variants "{target_platform: linux-64}"

# See the rendered recipe for macOS
conda render . --variants "{target_platform: osx-64}"
```

This will show you exactly which files are being included and what the final recipe looks like.

## Common Build Issues

### Issue: Wrong platform recipe is being used

**Solution**: Make sure you're specifying the `target_platform` variant:
```bash
conda build . --variants "{target_platform: osx-64}"
```

### Issue: Build fails with "file not found"

**Solution**: Check that the platform-specific files exist:
```bash
ls -la linux/  # Should have meta.yaml and build.sh
ls -la osx/    # Should have meta.yaml and build.sh
ls -la win/    # Should have meta.yaml and bld.bat
```

### Issue: Changes to meta.yaml not taking effect

**Solution**: Clear conda-build cache and try again:
```bash
conda build purge
conda build . --variants "{target_platform: linux-64}"
```

## Debugging

Enable verbose output to see what's happening:

```bash
conda build . --variants "{target_platform: linux-64}" --debug
```

See the full build log:
```bash
conda build . --variants "{target_platform: linux-64}" 2>&1 | tee build.log
```

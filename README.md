# Android PRoot Builder

Compile PRoot, proot-loader, and talloc from Termux source code to generate binaries that can run on Android.

## Features

- ✅ **Source Persistence**: Uses Docker volumes; reuses source after the initial clone without needing to re-download.
- ✅ **Incremental Compilation**: Only recompiles modified files.
- ✅ **Package Name Independent**: No package name replacement is performed.
- ✅ **Multi-Architecture Support**: arm64-v8a and x86_64.
- ✅ **Real-Time Progress Display**: Shows detailed progress information for source downloading and compilation.
- ✅ **Mainland China Acceleration**: Uses Alibaba Cloud/Tencent Cloud mirrors by default, suitable for users in mainland China.

## Quick Start

```powershell
# Build all architectures (clones source on first run, reuses subsequently)
.\build-proot.ps1

# Build only arm64 (physical devices)
.\build-proot.ps1 -Arch arm64

# Statically link talloc into libproot.so (no runtime dependency on libtalloc.so.2)
.\build-proot.ps1 -Arch arm64 -TallocLink static

# Build and copy to your Android project (requires specifying the project root directory)
.\build-proot.ps1 -CopyToJniLibs -CopyToAssets -AndroidProjectRoot "D:\path\to\YourAndroidProject"
```

## Artifacts

```text
output/
├── arm64/
│   ├── libproot.so           # PRoot main program
│   ├── libproot-loader.so    # ELF loader (64-bit)
│   ├── libproot-loader32.so  # ELF loader (32-bit compat)
│   └── libtalloc.so.2        # talloc memory library (output only when TallocLink=shared)
└── x86_64/
    ├── libproot.so
    ├── libproot-loader.so
    └── libtalloc.so.2        # output only when TallocLink=shared
```

## Build Modes

| Mode | Image | Source | Description |
|------|------|------|------|
| `incremental` | Reused | Reused | Default, fastest |
| `rebuild` | Rebuilt | Reused | Updates NDK and other dependencies |
| `clean` | Rebuilt | Rebuilt | Completely starts from scratch |

```powershell
# Incremental build (default)
.\build-proot.ps1

# Rebuild image but keep source code
.\build-proot.ps1 -Mode rebuild

# Completely clean and rebuild
.\build-proot.ps1 -Mode clean

# Only re-clone source (forces preparation from scratch)
.\build-proot.ps1 -ResetSource
```

## Project Integration

After the build is complete, you need to copy the artifacts to your project:

```powershell
# Auto-copy (recommended, requires specifying the Android project root directory)
.\build-proot.ps1 -CopyToJniLibs -CopyToAssets -AndroidProjectRoot "D:\path\to\YourAndroidProject"

# If using -TallocLink static, -CopyToAssets is generally not needed (reduces APK size)
.\build-proot.ps1 -CopyToJniLibs -TallocLink static -AndroidProjectRoot "D:\path\to\YourAndroidProject"

# Manual copy
Copy-Item output\arm64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\arm64-v8a\
Copy-Item output\arm64\libtalloc.so.2 <AndroidProjectRoot>\app\src\arm64\assets\proot\arm64-v8a\

Copy-Item output\x86_64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\x86_64\
Copy-Item output\x86_64\libtalloc.so.2 <AndroidProjectRoot>\app\src\x86_64\assets\proot\x86_64\
```

## Cleanup

```powershell
# Delete output files
.\clean.ps1 -RemoveOutput

# Delete Docker images
.\clean.ps1 -RemoveImages

# Delete source code (will be re-cloned on the next build)
.\clean.ps1 -RemoveSource

# Clean all
.\clean.ps1 -All
```

## Docker Resources

The build process uses the following Docker resources:

| Resource | Name | Purpose |
|------|------|------|
| Image | `proot-builder:arm64` | arm64 build environment |
| Image | `proot-builder:x86_64` | x86_64 build environment |
| Volume | `proot-builder-source` | Persists source code (shared across all architectures) |

```powershell
# View resources
docker images | Select-String "proot-builder"
docker volume ls | Select-String "proot-builder"
```

## Technical Notes

### Source Origins

| Component | Repository | Description |
|------|------|------|
| proot | [termux/proot](https://github.com/termux/proot) | Official Termux optimized version for Android |
| proot-loader | termux/proot (src/loader) | ELF loader |
| talloc | Same as above | Memory allocation library |

### Compilation Options

- `PROOT_UNBUNDLE_LOADER=1`: Uses an external loader
- Android NDK r27c + API 28
- PIE (Position Independent Executable)
- **Mainland China Mirror Acceleration**:
  - Ubuntu packages: Alibaba Cloud mirror
  - Android NDK: Tencent Cloud → Alibaba Cloud → Official source (auto-fallbacks)
  - GitHub source: ghproxy mirror → Official source (auto-fallbacks)

### Dockerfile Layer Optimization

To avoid re-downloading the NDK (633MB) when modifying dependencies, the Dockerfile uses an optimized layer structure:

```text
Layer 1: Base image + mirror replacement (rarely changes)
Layer 2: Basic tools (curl, unzip) (rarely changes)
Layer 3: Download NDK (largest file, placed early) ← Cached here
Layer 4: Other dependencies (may be adjusted)
Layer 5: Copy scripts (changes frequently)
```

This ensures that even if dependency packages are modified, the NDK will not be re-downloaded. See [DOCKERFILE-OPTIMIZATION.md](DOCKERFILE-OPTIMIZATION.md) for details.

### Why compile it yourself?

1. **Issue Debugging**: Add logging and debug symbols.
2. **Version Control**: Synchronize upstream fixes.
3. **Android Compatibility**: Include Android 16+ patches.

## Troubleshooting

### Build is stuck with no progress display

**Fixed!** The script now shows real-time progress:

```text
[INFO] Cloning proot source...
  Repository: https://github.com/termux/proot.git
  Branch: master
  Cloning into '/build/src/proot'...
  remote: Enumerating objects: 1234, done.
  remote: Counting objects: 100% (1234/1234), done.
  ...

[INFO] Compiling talloc...
  Using 8 parallel tasks
  CC talloc.c
  CC talloc_stack.c
  LD libtalloc.so.2
[OK] talloc compilation complete
```

If you still cannot see the output, you can run the test script to verify:

```powershell
.\test-progress.ps1
```

### Docker is not running

```powershell
docker version
# If it fails, start Docker Desktop
```

### Slow NDK download

The script will automatically try the Tencent Cloud mirror. If it is still slow, you can manually download the NDK and place it in `/opt/android-ndk`.

### Build fails

```powershell
# View the complete log
docker logs proot-builder-build-arm64

# Enter the container for debugging
docker run -it --rm -v proot-builder-source:/build/src proot-builder:arm64 /bin/bash
```

## Detailed Documentation

For a complete compilation guide, please refer to `proot-compilation-guide.md`.

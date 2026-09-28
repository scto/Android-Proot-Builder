# Android PRoot Builder

Compile PRoot, proot-loader, and talloc from Termux source code to generate binaries that can run on Android.

## Features

- ✅ Source Persistence: Uses Docker volumes; reuses source after the initial clone without needing to re-download.
- ✅ Incremental Compilation: Only recompiles modified files.
- ✅ Package Name Independent: No package name replacement is performed.
- ✅ Multi-Architecture Support: arm64-v8a and x86_64.
- ✅ Real-Time Progress Display: Shows detailed progress information for source downloading and compilation.
- ✅ Mainland China Acceleration: Uses Alibaba Cloud/Tencent Cloud mirrors by default, suitable for users in mainland China.
- 
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


Artifacts
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


Build Modes
Mode
Image
Source
Description
incremental
Reused
Reused
Default, fastest
rebuild
Rebuilt
Reused
Updates NDK and other dependencies
clean
Rebuilt
Rebuilt
Completely starts from scratch

# Incremental build (default)
.\build-proot.ps1

# Rebuild image but keep source code
.\build-proot.ps1 -Mode rebuild

# Completely clean and rebuild
.\build-proot.ps1 -Mode clean

# Only re-clone source (forces preparation from scratch)
.\build-proot.ps1 -ResetSource


Project Integration
After the build is complete, you need to copy the artifacts to your project:
# Auto-copy (recommended, requires specifying the Android project root directory)
.\build-proot.ps1 -CopyToJniLibs -CopyToAssets -AndroidProjectRoot "D:\path\to\YourAndroidProject"

# If using -TallocLink static, -CopyToAssets is generally not needed (reduces APK size)
.\build-proot.ps1 -CopyToJniLibs -TallocLink static -AndroidProjectRoot "D:\path\to\YourAndroidProject"

# Manual copy
Copy-Item output\arm64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\arm64-v8a\
Copy-Item output\arm64\libtalloc.so.2 <AndroidProjectRoot>\app\src\arm64\assets\proot\arm64-v8a\

Copy-Item output\x86_64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\x86_64\
Copy-Item output\x86_64\libtalloc.so.2 <AndroidProjectRoot>\app\src\x86_64\assets\proot\x86_64\


Cleanup
# Delete output files
.\clean.ps1 -RemoveOutput

# Delete Docker images
.\clean.ps1 -RemoveImages

# Delete source code (will be re-cloned on the next build)
.\clean.ps1 -RemoveSource

# Clean all
.\clean.ps1 -All


Docker Resources
The build process uses the following Docker resources:
Resource
Name
Purpose
Image
proot-builder:arm64
arm64 build environment
Image
proot-builder:x86_64
x86_64 build environment
Volume
proot-builder-source
Persists source code (shared across all architectures)

# View resources
docker images | Select-String "proot-builder"
docker volume ls | Select-String "proot-builder"


Technical Notes
Source Origins
Component
Repository
Description
proot
termux/proot
Official Termux optimized version for Android
proot-loader
termux/proot (src/loader)
ELF loader
talloc
Same as above
Memory allocation library

Compilation Options
PROOT_UNBUNDLE_LOADER=1: Uses an external loader
Android NDK r27c + API 28
PIE (Position Independent Executable)
Mainland China Mirror Acceleration:
Ubuntu packages: Alibaba Cloud mirror
Android NDK: Tencent Cloud → Alibaba Cloud → Official source (auto-fallbacks)
GitHub source: ghproxy mirror → Official source (auto-fallbacks)
Dockerfile Layer Optimization
To avoid re-downloading the NDK (633MB) when modifying dependencies, the Dockerfile uses an optimized layer structure:
Layer 1: Base image + mirror replacement (rarely changes)
Layer 2: Basic tools (curl, unzip) (rarely changes)
Layer 3: Download NDK (largest file, placed early) ← Cached here
Layer 4: Other dependencies (may be adjusted)
Layer 5: Copy scripts (changes frequently)


This ensures that even if dependency packages are modified, the NDK will not be re-downloaded. See DOCKERFILE-OPTIMIZATION.md for details.
Why compile it yourself?
Issue Debugging: Add logging and debug symbols.
Version Control: Synchronize upstream fixes.
Android Compatibility: Include Android 16+ patches.
Troubleshooting
Build is stuck with no progress display
Fixed! The script now shows real-time progress:
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


If you still cannot see the output, you can run the test script to verify:
.\test-progress.ps1


Docker is not running
docker version
# If it fails, start Docker Desktop


Slow NDK download
The script will automatically try the Tencent Cloud mirror. If it is still slow, you can manually download the NDK and place it in /opt/android-ndk.
Build fails
# View the complete log
docker logs proot-builder-build-arm64

# Enter the container for debugging
docker run -it --rm -v proot-builder-source:/build/src proot-builder:arm64 /bin/bash


Detailed Documentation
For a complete compilation guide, please refer to proot-compilation-guide.md.

# Android PRoot Builder

从 Termux 源码编译 PRoot、proot-loader 和 talloc，生成可在 Android 上运行的二进制文件。

## 特性

- ✅ **源码持久化**: 使用 Docker volume，首次克隆后复用，无需重复下载
- ✅ **增量编译**: 只重新编译修改过的文件
- ✅ **包名无关**: 不做包名替换
- ✅ **多架构支持**: arm64-v8a 和 x86_64
- ✅ **实时进度显示**: 显示源码下载、编译的详细进度信息
- ✅ **国内加速**: 默认使用阿里云/腾讯云镜像，适合中国大陆用户

## 快速开始

```powershell
# 构建所有架构（首次运行会克隆源码，后续复用）
.\build-proot.ps1

# 只构建 arm64（真机）
.\build-proot.ps1 -Arch arm64

# 将 talloc 静态链接进 libproot.so（运行时不再依赖 libtalloc.so.2）
.\build-proot.ps1 -Arch arm64 -TallocLink static

# 构建并复制到你的 Android 项目（需要指定项目根目录）
.\build-proot.ps1 -CopyToJniLibs -CopyToAssets -AndroidProjectRoot "D:\\path\\to\\YourAndroidProject"

```

## 产物

```
output/
├── arm64/
│   ├── libproot.so           # PRoot 主程序
│   ├── libproot-loader.so    # ELF loader (64-bit)
│   ├── libproot-loader32.so  # ELF loader (32-bit compat)
│   └── libtalloc.so.2        # talloc 内存库（仅 TallocLink=shared 时输出）
└── x86_64/
    ├── libproot.so
    ├── libproot-loader.so
    └── libtalloc.so.2        # 仅 TallocLink=shared 时输出
```

## 构建模式

| 模式 | 镜像 | 源码 | 说明 |
|------|------|------|------|
| `incremental` | 复用 | 复用 | 默认，最快 |
| `rebuild` | 重建 | 复用 | 更新 NDK 等依赖 |
| `clean` | 重建 | 重建 | 完全从头开始 |

```powershell
# 增量构建（默认）
.\build-proot.ps1

# 重建镜像但保留源码
.\build-proot.ps1 -Mode rebuild

# 完全清理后重建
.\build-proot.ps1 -Mode clean

# 只重新克隆源码（强制从头准备）
.\build-proot.ps1 -ResetSource
```

## 项目集成

构建完成后，需要将产物复制到项目中：

```powershell
# 自动复制（推荐，需指定 Android 项目根目录）
.\build-proot.ps1 -CopyToJniLibs -CopyToAssets -AndroidProjectRoot "D:\\path\\to\\YourAndroidProject"

# 如果使用 -TallocLink static，一般不需要 -CopyToAssets（可减少 APK 体积）
.\build-proot.ps1 -CopyToJniLibs -TallocLink static -AndroidProjectRoot "D:\\path\\to\\YourAndroidProject"

# 手动复制
Copy-Item output\arm64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\arm64-v8a\
Copy-Item output\arm64\libtalloc.so.2 <AndroidProjectRoot>\app\src\arm64\assets\proot\arm64-v8a\

Copy-Item output\x86_64\libproot*.so <AndroidProjectRoot>\app\src\main\jniLibs\x86_64\
Copy-Item output\x86_64\libtalloc.so.2 <AndroidProjectRoot>\app\src\x86_64\assets\proot\x86_64\
```

## 清理

```powershell
# 删除输出文件
.\clean.ps1 -RemoveOutput

# 删除 Docker 镜像
.\clean.ps1 -RemoveImages

# 删除源码（下次构建会重新克隆）
.\clean.ps1 -RemoveSource

# 全部清理
.\clean.ps1 -All
```

## Docker 资源

构建过程使用以下 Docker 资源：

| 资源 | 名称 | 用途 |
|------|------|------|
| 镜像 | `proot-builder:arm64` | arm64 构建环境 |
| 镜像 | `proot-builder:x86_64` | x86_64 构建环境 |
| Volume | `proot-builder-source` | 持久化源码（所有架构共享） |

```powershell
# 查看资源
docker images | Select-String "proot-builder"
docker volume ls | Select-String "proot-builder"
```

## 技术说明

### 源码来源

| 组件 | 仓库 | 说明 |
|------|------|------|
| proot | [termux/proot](https://github.com/termux/proot) | Termux 官方 Android 优化版 |
| proot-loader | termux/proot（src/loader） | ELF 加载器 |
| talloc | 同上 | 内存分配库 |

### 编译选项

- `PROOT_UNBUNDLE_LOADER=1`: 使用外部 loader
- Android NDK r27c + API 28
- PIE (Position Independent Executable)
- **国内镜像加速**:
  - Ubuntu 包: 阿里云镜像
  - Android NDK: 腾讯云 → 阿里云 → 官方源（自动降级）
  - GitHub 源码: ghproxy 镜像 → 官方源（自动降级）

### Dockerfile 层级优化

为了避免修改依赖包时重新下载 NDK（633MB），Dockerfile 采用了优化的层级结构：

```
第 1 层：基础镜像 + 换源（几乎不变）
第 2 层：基础工具（curl, unzip）（几乎不变）
第 3 层：下载 NDK（最大文件，放前面）← 缓存在这里
第 4 层：其他依赖包（可能调整）
第 5 层：复制脚本（经常变）
```

这样即使修改了依赖包，也不会重新下载 NDK。详见 [DOCKERFILE-OPTIMIZATION.md](DOCKERFILE-OPTIMIZATION.md)

### 为什么需要自己编译？

1. **问题调试**: 添加日志、调试符号
2. **版本控制**: 同步上游修复
3. **Android 兼容性**: 加入 Android 16+ 补丁

## 故障排除

### 构建卡住没有进度显示

**已修复！** 脚本现在会显示实时进度：

```
[INFO] 克隆 proot 源码...
  仓库: https://github.com/termux/proot.git
  分支: master
  Cloning into '/build/src/proot'...
  remote: Enumerating objects: 1234, done.
  remote: Counting objects: 100% (1234/1234), done.
  ...

[INFO] 编译 talloc...
  使用 8 个并行任务
  CC talloc.c
  CC talloc_stack.c
  LD libtalloc.so.2
[OK] talloc 编译完成
```

如果仍然看不到输出，可以运行测试脚本验证：

```powershell
.\test-progress.ps1
```

### Docker 未运行

```powershell
docker version
# 如果失败，启动 Docker Desktop
```

### NDK 下载慢

脚本会自动尝试腾讯云镜像，如仍然慢，可手动下载 NDK 放到 `/opt/android-ndk`。

### 编译失败

```powershell
# 查看完整日志
docker logs proot-builder-build-arm64

# 进入容器调试
docker run -it --rm -v proot-builder-source:/build/src proot-builder:arm64 /bin/bash
```

## 详细文档

完整的编译指南请参考 `proot-compilation-guide.md`

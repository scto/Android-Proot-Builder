# PRoot Compilation Guide

This document explains how to compile PRoot and its dependencies from Termux source code to support the Linux environment for Android PRoot Builder.

## Overview

Android PRoot Builder uses PRoot to provide Linux environment support on Android. PRoot is a user-space implementation of `chroot` and `mount --bind`, allowing Linux programs to run on Android without requiring root privileges.

### Components Description

| Component | Filename | Purpose |
|------|--------|------|
| PRoot Main Program | `libproot.so` | Core program, provides rootfs isolation and system call translation |
| ELF Loader (64-bit) | `libproot-loader.so` | Loads and executes ELF executables |
| ELF Loader (32-bit) | `libproot-loader32.so` | Runs 32-bit programs on 64-bit systems |
| talloc Library | `libtalloc.so.2` | Memory allocation library (Runtime dependency and output when `TallocLink=shared`; default not output and no need to distribute when `TallocLink=static`) |

### Why compile it yourself?

1. **Android Compatibility Fixes**: seccomp/clone3 compatibility issues exist on Android 16+
2. **Issue Debugging**: Ability to add logs and debug symbols
3. **Version Synchronization**: Get bug fixes from Termux upstream at any time
4. **Custom Modifications**: Customize PRoot behavior according to specific needs

## Compilation Environment

### Prerequisites

- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- PowerShell 5.1+ or PowerShell 7+
- ~5GB disk space (NDK + build cache)
- Internet connection (to download NDK and source code)

### Target Platforms

| Architecture | Android ABI | Description |
|------|-------------|------|
| aarch64 | arm64-v8a | Most Android phones/tablets |
| x86_64 | x86_64 | Emulators and some Chromebooks |

### Android API Level

The compilation target is **API 28 (Android 9.0)**, ensuring broad device compatibility.

## Quick Compilation

### One-Click Build

```powershell
# Build all architectures (clones source on first run, reuses subsequently)
\.\\build-proot.ps1

# Build only arm64 (physical devices)
\.\\build-proot.ps1 -Arch arm64

# Statically link talloc into libproot.so (no runtime dependency on libtalloc.so.2)
\.\\build-proot.ps1 -Arch arm64 -TallocLink static

# Build and automatically integrate into the project
\.\\build-proot.ps1 -CopyToJniLibs -CopyToAssets

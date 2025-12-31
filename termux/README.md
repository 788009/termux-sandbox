# Termux in Termux

[← Back to Root](../#supported-environments)

`termux-sandbox` facilitates the creation of clean, segregated Termux instances, offering a robust set of management tools.

**Content**: [Features](#features) | [Requirements](#requirements) | [Installation](#installation) | [Usage](#usage)

## Features

- **Native Execution**  
  Runs directly on the system without emulation or proot overhead.

- **Independent Environments**  
  Create multiple sandboxes for different projects or experiments.

- **Safe Defaults**  
  The sandbox starts minimal and does not modify the host Termux installation unless explicitly invoked.

- **Root-Aware Userspace**  
  Includes a preloadable UID shim to allow Termux packages to operate correctly under real root privileges.

- **Seamless Networking**  
  Out-of-the-box internet access. Automatically configures DNS (`resolv.conf`) and network interfaces, so tools like `pkg`, `pip`, and `git` work immediately without manual host patching.

- **Host Access**  
  - **Safe mode (default)**: No access to `/sdcard` or the host Android system.
  - **Unrestricted mode**: Optional flags to map `/sdcard` and `/host_root` for advanced tasks.

- **Duplicate, Export, and Import**  
  Allows for easy sandbox duplication, backup (exporting), and environment recovery/sharing (importing), significantly streamlining setup and maintenance.

## Requirements

- Android device with root access.  
  (Magisk or KernelSU recommended; other root solutions may work but are not guaranteed)
- Architecture: ARM64 (aarch64) or ARM (armhf).
- A working Termux installation.

> [!NOTE]
> 
> For non-root users, consider using [Yonle/termux-proot](https://github.com/Yonle/termux-proot) instead.

> [!IMPORTANT]
> 
> **Note on kernel compatibility**
> 
> This project relies on Linux mount namespaces.
> 
> If you encounter the following error when running under `tsu`:
> 
> ```
> unshare: Operation not permitted
> ```
> 
> it indicates that mount namespaces are not available to user processes on your device, due to kernel configuration, SELinux policy, or vendor-specific restrictions. In this case, the sandbox cannot function on that device.
> 
> You can also verify compatibility by running this command in `tsu`:
> 
> ```bash
> unshare --mount /bin/true
> ```
> 
> If it returns an error, your kernel does not support the required isolation features.

## Installation

### 1. Install basic dependencies

```bash
pkg update
pkg install tsu curl zip clang util-linux -y
```

> [!NOTE]
> 
> * `clang` is required to compile `fake_uid.c` for UID spoofing.
> * `util-linux` provides `unshare`. On some systems it may already be present.
> * `tsu` is used to maintain Termux environment variables (like `$PATH`) while operating with root privileges. Using standard `su` or `su -i` may cause dependency resolution failures.

### 2. Install the sandbox manager script

> [!WARNING]
> 
> The script must be installed inside the Termux (host) prefix. Do not install it into `/bin`, which is read-only on Android.

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/latest/termux-sandbox" -o "$PREFIX/bin/termux-sandbox"
chmod +x "$PREFIX/bin/termux-sandbox"
```

## Usage

> [!WARNING]
> 
> Requires `tsu` environment.

> [!TIP]
> 
> If no name is specified for the `create`, `enter`, and `delete` commands, the actual name used will be `default`.

### View the version

```bash
termux-sandbox version
termux-sandbox --version
```

### Create a sandbox

#### 1. Default creation

When no source is specified, the script automatically downloads the official Termux bootstrap matching your device's architecture (ARMHF or ARM64).

The current default version is [this release](https://github.com/termux/termux-packages/releases/tag/bootstrap-2025.11.30-r1%2Bapt.android-7).

```bash
termux-sandbox create
# or with name
termux-sandbox create mydevbox
```

#### 2. Using a custom bootstrap

You can use the `--source` option to provide a local file path or a direct download URL for a custom bootstrap archive (must be `.zip`).

```bash
# Using a local file
termux-sandbox create mysandbox --source /path/to/mybootstrap.zip
# Using a direct URL
termux-sandbox create mysandbox --source https://example.com/custom_bootstrap.zip
```

> [!TIP]
> 
> The default bootstrap is not always the latest. You can find the latest version and more official bootstraps [here](https://github.com/termux/termux-packages/releases).

> [!NOTE]
> 
> **Note on Architectures**
> 
> When using a custom source, ensure the archive is built for the **correct architecture** (`ARMHF` or `ARM64`) of your device, as the script will skip the smart architecture detection.

### Enter a sandbox (two modes)

#### 1. Safe mode (recommended)

By default, the sandbox is isolated. It cannot see your SD card or host system files.

```bash
termux-sandbox enter
# or with name
termux-sandbox enter mysandbox
```

#### 2. Unrestricted mode (advanced)

Use the `-b` or `--bind` flag to mount host directories (`/sdcard` and `/host_root`).

```bash
termux-sandbox enter -b
# or with name
termux-sandbox enter mysandbox -b
```

> [!CAUTION]
> 
> **Warning for Unrestricted Mode**
> 
> When using `-b`, you have **Real Root Access** to your physical file system.
> 
>   * Deleting files in `/sdcard` will permanently delete them from your phone.
>   * Modifying `/host_root` can **BRICK** your device.

> [!WARNING]
>
> Run `apt update` before using `apt` inside the sandbox.

### Exiting the sandbox

Press `Ctrl` + `D` to exit. If stuck, reboot your device (and feel free to open an issue).

### View sandbox list and status

```bash
termux-sandbox list
```

### Remove a sandbox

```bash
termux-sandbox delete
# or with name
termux-sandbox delete mysandbox
```

> [!TIP]
> 
> Always use the `termux-sandbox delete` command. This ensures that lock files and temporary scripts are cleaned up correctly.
> 
> This tool uses Private Mount Namespaces. This means mounts created inside the sandbox are invisible to the host system. While manual deletion of the sandbox directory is safe, using the built-in delete command is still the preferred method to prevent state inconsistencies.

### Rename a sandbox

```bash
termux-sandbox rename mysandbox yoursandbox
```

### Duplicate a sandbox

```bash
termux-sandbox duplicate mysandbox mysandbox2
```

### Export a sandbox

```bash
termux-sandbox export mysandbox mysandbox.tar.gz
```

This command exports the complete sandbox environment, including all its metadata.

### Import a sandbox

```bash
termux-sandbox import mysandbox.tar.gz
# or skip verifying metadata with --force or -f
termux-sandbox import mysandbox.tar.gz -f
```

## Implementation Overview

* **Private Mount Namespaces (`unshare`)** are used to isolate the file system hierarchy. Mounts created inside the sandbox are invisible to the host, preventing accidental host data loss during cleanup.
* **Chroot** provides a minimal root filesystem based on the Termux bootstrap.
* An **`LD_PRELOAD` library** overrides a small set of system calls to report a non-root UID, allowing `apt`, `pkg`, and other Termux tools to operate normally.
* **Network Namespace Sharing** allows the sandbox to use the host's network connection directly.
* **Automatic DNS Configuration** generates a standard Linux `/etc/resolv.conf` by auto-detecting DNS connectivity, bypassing Android-specific DNS properties that often fail inside chroots.
* **Smart Exporting**: When exporting, APT caches and runtime files (busybox, entry scripts, mount points) are automatically excluded to minimize the archive size.

---

[Back to Root](../README.md#supported-environments)
# Termux Sandbox

[中文](https://github.com/788009/termux-sandbox/blob/main/README_zh.md)

Termux Sandbox provides isolated, clean Termux environments that run with native performance inside an existing Termux installation.  
It is designed for testing scripts, building software, or keeping the main environment minimal without relying on proot or container runtimes.

**Note**: This tool only provides **environment** isolation, not **security** isolation. Therefore, it **must not** be used for malicious script testing.

![example-enter.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-enter.jpg?raw=true)

<details>
<summary>More images</summary>

![example-create.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-create.jpg?raw=true)

![example-use.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-use.jpg?raw=true)

</details>

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
  - **Safe mode (default):** No access to `/sdcard` or the host Android system.
  - **Unrestricted mode:** Optional flags to map `/sdcard` and `/host_root` for advanced tasks.

- **Duplicate, Export, and Import**
  Allows for easy sandbox duplication, backup (exporting), and environment recovery/sharing (importing), significantly streamlining setup and maintenance.

## Requirements

- Android device with root access (Magisk or KernelSU; others may be compatible)
- Architecture: ARM64 (aarch64)
- Current Termux installation
- BusyBox static binary (provided below)

**Note**: **For non-root users**, use [Yonle/termux-proot](https://github.com/Yonle/termux-proot) instead.

## Installation

### 1. Install basic dependencies

```bash
pkg update
pkg install tsu curl zip clang -y
```

*(`clang` is required to compile `fake_uid.c` for UID spoofing.)*

### 2. Install the sandbox manager script

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/v1.0/termux-sandbox" -o $PREFIX/bin/termux-sandbox
chmod +x $PREFIX/bin/termux-sandbox
```

## Usage

<details>
<summary>Click to expand</summary>

**Requires `tsu` environment.**

If no name is specified for the `create`, `enter`, and `delete` commands, the actual name used will be `default`.

### Create a new sandbox

```bash
termux-sandbox create
# or with name
termux-sandbox create mysandbox
```

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
termux-sandbox enter -b mysandbox
```

**Warning for Unrestricted Mode:**
When using `-b`, you have **Real Root Access** to your physical file system.

  * Deleting files in `/sdcard` will permanently delete them from your phone.
  * Modifying `/host_root` can **BRICK** your device.

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

**CRITICAL WARNING: READ BEFORE DELETING**
 
1.  **ALWAYS use the `termux-sandbox delete` command.**
2.  **NEVER manually delete** the sandbox directory (e.g., using `rm` or a file manager) while the > > > sandbox is active.
      * *Why?* Even in Safe Mode, system directories (`/system`, `/dev`) are mounted. Deleting the folder manually will attempt to delete actual system files, which can **BRICK** your device or cause data loss.
3.  **Uninstalling Termux?**
      * Please **reboot your device** first. This guarantees that all sandbox mounts are disconnected before the Termux app data is wiped by Android.

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

### Import a sandbox

```bash
termux-sandbox import mysandbox.tar.gz
```

</details>

## Implementation Overview

* **Mount namespaces** are used to isolate the environment while allowing selected host paths to be rebound.
* **Chroot** provides a minimal root filesystem based on the Termux bootstrap.
* An **`LD_PRELOAD` library** overrides a small set of system calls to report a non-root UID, allowing `apt`, `pkg`, and other Termux tools to operate normally.
* The design avoids modifying host Termux and keeps each sandbox self-contained.
* **Network Namespace Sharing** allows the sandbox to use the host's network connection directly.
* **Automatic DNS Configuration** generates a standard Linux `/etc/resolv.conf` (auto-detecting DNS connectivity), bypassing Android's specific DNS properties that often fail inside chroots.
* When exporting and importing, APT caches will be automatically cleaned, and environment-irrelevant content such as `busybox`, `entry.sh`, and mount points will be excluded to minimize the export file size.

## Credits

* Static BusyBox binary provided by [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64).
* Termux bootstrap packages from the [official Termux project](https://github.com/termux/termux-packages).

## License

MIT License

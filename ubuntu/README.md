# Ubuntu in Termux

[← Back to Root](../#supported-environments)

`ubuntu-sandbox` facilitates the creation of clean, segregated Ubuntu instances within Termux ([and potentially other distributions](#2-using-a-custom-source)), offering a robust set of management tools.

**Content**: [Features](#features) | [Requirements](#requirements) | [Installation](#installation) | [Usage](#usage) | [Advanced Usage](#advanced-usage) | [Implementation Overview](#implementation-overview)

## Features

- **Native Execution** Runs directly on the system without emulation or `proot` overhead, ensuring zero performance loss.

- **Independent Environments** Create multiple sandboxes for different projects or experiments.

- **Safe Defaults** The sandbox remains minimal and isolated. It does not modify or "pollute" the host Termux environment unless explicitly invoked.

- **Seamless Networking** Out-of-the-box internet access. Automatically configures DNS (`resolv.conf`) and network interfaces, so tools like `apt`, `pip`, and `git` work immediately without manual host patching.

- **Host Access** - **Safe mode (default)**: No access to `/sdcard` or the host Android system files.
  - **Unrestricted mode**: Optional flags to map `/sdcard` and `/host_root` for advanced tasks.

- **Duplicate, Export, and Import** Allows for easy sandbox duplication, backup (exporting), and environment recovery/sharing (importing), significantly streamlining setup and maintenance.

## Requirements

- Android device with root access.  
  (Magisk or KernelSU recommended; other root solutions may work but are not guaranteed)
- Architecture: ARM64 (aarch64) or ARM (armhf).
- A working Termux installation.

> [!NOTE]
> 
> For non-root users, consider using [proot-distro](https://github.com/termux/proot-distro) instead.

> [!IMPORTANT]
> 
> **Note on kernel compatibility**
> 
> This project relies on Linux mount namespaces.
> 
> If you encounter the following error when running under `sudo bash`:
> 
> ```
> unshare: Operation not permitted
> ```
> 
> It indicates that mount namespaces are not available to user processes on your device due to kernel configuration, SELinux policy, or vendor restrictions. In this case, the sandbox cannot function.
> 
> You can verify compatibility by running this command in `sudo bash`:
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
pkg install sudo curl zip util-linux -y
```

> [!NOTE]
> 
> * `util-linux` provides the `unshare` tool. On some systems, it may already be present.
> * `sudo` is used to maintain Termux environment variables (like `$PATH`) while operating with root privileges. Using standard `su` or `su -i` may cause dependency resolution failures.

### 2. Install the ubuntu-sandbox script

> [!WARNING]
> 
> The script must be installed inside the Termux (host) prefix (`$PREFIX`). Do not install it into the system `/bin` directory, which is read-only on Android.

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/latest/ubuntu-sandbox" -o "$PREFIX/bin/ubuntu-sandbox"
chmod +x "$PREFIX/bin/ubuntu-sandbox"
```

## Usage

> [!WARNING]
> 
> Requires `sudo bash` environment.

> [!TIP]
> 
> If no name is specified for the `create`, `enter`, and `delete` commands, the actual name used will be `default`.

### View the version

```bash
ubuntu-sandbox version
ubuntu-sandbox --version
```

### Create a sandbox

#### 1. Default creation

When no source is specified, the script downloads the official Ubuntu Base and automatically detects the architecture (ARMHF or ARM64).

The current default source is [this release](https://cdimage.ubuntu.com/ubuntu-base/releases/24.04.1/release/), version `24.04.1` LTS.

```bash
ubuntu-sandbox create
# or with name
ubuntu-sandbox create mydevbox
```

#### 2. Using a custom source

Use the `--source` option to provide a local file path or a direct download URL for a custom Ubuntu Base (must be `.tar.gz`).

```bash
# Using a local file
ubuntu-sandbox create mysandbox --source /path/to/mybase.tar.gz
# Using a direct URL
ubuntu-sandbox create mysandbox --source https://example.com/custom_base.tar.gz
```

> [!TIP]
> 
> The default Base is not always the latest. You can find newer or other official versions [here](https://cdimage.ubuntu.com/ubuntu-base/releases/).

> [!NOTE]
> 
> When using a custom source, ensure the Base architecture (`ARMHF`/`ARM64`) matches your device.

> [!TIP]
> 
> **Explore More Distributions**
> 
> Although named `ubuntu-sandbox`, **testing with** other Linux rootfs images (such as Debian, Arch Linux ARM, etc.) via the `--source` flag **is encouraged**. Please open an issue if you encounter any problems or have successful experiences to share.

### Enter a sandbox (two modes)

#### 1. Safe mode (recommended)

By default, the sandbox is isolated. It cannot see your SD card or host system files.

```bash
ubuntu-sandbox enter
# or with name
ubuntu-sandbox enter mysandbox
```

#### 2. Unrestricted mode (advanced)

Use the `-b` or `--bind` flag to mount host directories (`/sdcard` and `/host_root`).

```bash
ubuntu-sandbox enter -b
# or with name
ubuntu-sandbox enter mysandbox -b
```

> [!CAUTION]
> 
> **Warning for Unrestricted Mode**
> 
> When using `-b`, you have **Real Root Access** to your physical file system.
> * Deleting files in `/sdcard` will permanently delete them from your phone.
> * Modifying `/host_root` can **BRICK** your device.

> [!WARNING]
> Run `apt update` before using `apt` inside the sandbox.

### Exiting the sandbox

Press `Ctrl` + `D` to exit. If stuck, reboot your device (and feel free to open an issue).

### View sandbox list and status

```bash
ubuntu-sandbox list
```

### Remove a sandbox

```bash
ubuntu-sandbox delete
# or with name
ubuntu-sandbox delete mysandbox
```

> [!TIP]
> 
> Always use the `ubuntu-sandbox delete` command. This ensures that lock files and temporary scripts are cleaned up correctly.
> 
> This tool uses Private Mount Namespaces. Mounts created inside the sandbox are invisible to the host system. While manual deletion of the sandbox directory is generally safe, using the built-in delete command is preferred to prevent state inconsistencies.

### Rename a sandbox

```bash
ubuntu-sandbox rename mysandbox yoursandbox
```

### Duplicate a sandbox

```bash
ubuntu-sandbox duplicate mysandbox mysandbox2
```

### Export a sandbox

```bash
ubuntu-sandbox export mysandbox mysandbox.tar.gz
```

This command exports the complete sandbox environment, including all its metadata.

### Import a sandbox

```bash
ubuntu-sandbox import mysandbox.tar.gz
# or skip verifying metadata with --force or -f
ubuntu-sandbox import mysandbox.tar.gz -f
```

### Uninstall `ubuntu-sandbox`

```bash
ubuntu-sandbox uninstall
```

## Advanced Usage

### Running Xfce4 Desktop via Termux:X11

#### Environment Setup

Install Termux:X11 from the same source as your Termux app (e.g., GitHub or F-Droid). Then, install `x11-repo` in the Termux host environment:

```bash
pkg update
pkg install x11-repo -y
```

Install dependencies inside the sandbox:

```bash
apt update
apt install xfce4 dbus-x11 -y
```

> [!NOTE]
> 
> Ubuntu Base provides a minimal environment and does not include `dbus-x11`. Since `xfce4` relies on `dbus-launch` from the `dbus-x11` package to initialize sessions, it must be installed manually.

#### Start Services

Start the Termux:X11 service in the Termux host:

```bash
termux-x11 :0 -ac
```

Inside the sandbox (running in `-b|--bind` mode):

```bash
mkdir -p /tmp/.X11-unix
mount --bind /host_root/data/data/com.termux/files/usr/tmp/.X11-unix /tmp/.X11-unix
export DISPLAY=:0
xfce4-session
```

It is recommended to save the above commands as a script, allowing you to start the desktop with a single command.

> [!NOTE]
> 
> - `-ac` flag: Disables X11 access control, ensuring processes within the isolated environment can connect to the host's display service.
> - `-b|--bind` mode: Required to access the `.X11-unix` socket outside the sandbox.
> - Mounting `.X11-unix`: X11 relies on Unix Domain Sockets for communication. By mounting this directory, Xfce4 inside the sandbox can communicate with Termux:X11 via the host's socket file.

> [!TIP]
> 
> **About GPU Acceleration**
> 
> Starting the Xfce4 desktop using the method above will utilize `llvmpipe`, meaning the CPU handles all rendering computations. While it is **possible** to access the device's GPU within the sandbox, the implementation details vary significantly across different hardware brands. As such, these steps are not covered in detail here; please feel free to experiment if needed.

## Implementation Overview

* **Private Mount Namespaces (`unshare`)** are used to isolate the file system hierarchy. Mounts created inside the sandbox are invisible to the host, preventing accidental host data loss during cleanup.
* **Chroot** provides a minimal root filesystem based on the Termux bootstrap.
* **Network Namespace Sharing** allows the sandbox to use the host's network connection directly.
* **Automatic DNS Configuration** generates a standard Linux `/etc/resolv.conf` by auto-detecting DNS connectivity, bypassing Android-specific DNS properties that often fail inside chroots.
* **Smart Exporting**: When exporting, APT caches and runtime files (busybox, entry scripts, mount points) are automatically excluded to minimize the archive size.

---

[Back to Root](../#supported-environments)
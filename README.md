# Termux Sandbox

[中文](https://github.com/788009/termux-sandbox/blob/main/README_zh.md)

Termux Sandbox provides isolated, clean Termux environments that run with native performance inside an existing Termux installation.  
It is designed for testing scripts, building software, or keeping the main environment minimal without relying on proot or container runtimes.

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

- **Root-Aware Userspace**  
  Includes a preloadable UID shim to allow Termux packages to operate correctly under real root privileges.

- **Host Access**  
  - `/sdcard` – direct access to shared storage  
  - `/host_root` – unrestricted mirror of the Android root filesystem  

- **Safe Defaults**  
  The sandbox starts minimal and does not modify the host Termux installation unless explicitly invoked.

## Requirements

- Android device with root access (Magisk or KernelSU; others may be compatible)
- Architecture: ARM64 (aarch64)
- Current Termux installation
- BusyBox static binary (provided below)

## Installation

### 1. Install basic dependencies

```bash
pkg update
pkg install tsu curl zip clang -y
```

### 2. Install the sandbox manager script

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/v1.0/termux-sandbox" \
     -o $PREFIX/bin/termux-sandbox
chmod +x $PREFIX/bin/termux-sandbox
```

### 3. Install a static BusyBox binary

```bash
mkdir -p ~/.suroot
curl -L 'https://github.com/788009/termux-sandbox/releases/download/v1.0/busybox_arm64' \
     -o ~/.suroot/busybox_arm64
chmod 755 ~/.suroot/busybox_arm64
```

## Usage

**Requires `tsu` environment.**

### Create a new sandbox

```bash
termux-sandbox create
# or with name
termux-sandbox create dev
```

### Enter a sandbox

```bash
termux-sandbox enter
# or with name
termux-sandbox enter dev
```

### Remove a sandbox

```bash
termux-sandbox delete
# or with name
termux-sandbox delete dev
```

## Inside the Sandbox

Once inside, the file system layout is standard Linux/Termux, with two special additions:

| Path | Description |
| :--- | :--- |
| `/sdcard` | Direct access to your internal storage. |
| `/host_root` | A **recursive mirror** of your real Android system. You can see `/data`, `/apex`, `/vendor` etc. inside here. |

> **Warning:** You are **Root** inside the sandbox.
> *   Deleting files in `/sdcard` deletes them from your phone.
> *   Deleting files in `/host_root` **WILL BRICK** your phone.

## Implementation Overview

* **Mount namespaces** are used to isolate the environment while allowing selected host paths to be rebound.
* **Chroot** provides a minimal root filesystem based on the Termux bootstrap.
* An **LD_PRELOAD library** overrides a small set of system calls to report a non-root UID, allowing `apt`, `pkg`, and other Termux tools to operate normally.
* The design avoids modifying host Termux and keeps each sandbox self-contained.

## Credits

* Static BusyBox binary provided by [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64).
* Termux bootstrap packages from the [official Termux project](https://github.com/termux/termux-packages).

## License

MIT License

# Termux Sandbox

[![Termux](https://img.shields.io/badge/Termux-Rolling-green.svg)](https://termux.dev/)
[![Root Required](https://img.shields.io/badge/Root-Required-red.svg)](https://github.com/788009/termux-sandbox)

**Termux Sandbox** allows you to run **clean, isolated, and native** Termux environments inside your existing Termux installation.

Unlike `proot` solutions, this uses **Native Chroot**, **Mount Namespaces**, and **LD_PRELOAD** techniques to provide a high-performance environment with real Root privileges, perfect for testing dangerous scripts, compiling software, or keeping your main environment clutter-free.

## ✨ Features

*   **⚡ Native Performance:** Zero emulation overhead. Compiles and runs code at full native speed.
*   **📦 Multi-Instance:** Create multiple independent sandboxes (`default`, `dev`, `test`...).
*   **🛡️ Fake UID Injection:** Custom `libfakeuid.so` allows Termux packages (apt, dpkg) to run correctly under Root by spoofing UID 10000.
*   **🧠 Recursive Host Access:**
    *   `/host_root`: A perfect mirror of your Android Root filesystem (using recursive mounting).
    *   `/sdcard`: Direct, reliable access to internal storage via `/data/media/0`.
*   **🌐 Smart Networking:** Automatic DNS fallback (Google -> 114DNS) and Hosts injection ensures `pip`, `pkg`, and `curl` work immediately.

## 🛠️ Prerequisites

*   **Android Device with Root Access** (Magisk/KernelSU).
*   **Termux App**.

## 📥 Installation

You can install everything using the commands below.

### 1. Install Dependencies
```bash
pkg update
pkg install tsu curl zip clang
```

### 2. Install the Script
Download the script to your bin directory and make it executable:
```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/v1.0/termux-sandbox" -o $PREFIX/bin/termux-sandbox
chmod +x $PREFIX/bin/termux-sandbox
```

### 3. Install Busybox (Required for Chroot)
The sandbox requires a static busybox binary placed in the Root Home directory (`~/.suroot`). Run this command to download and install it automatically:

```bash
curl -L 'https://github.com/788009/termux-sandbox/releases/download/v1.0/busybox_arm64' -o /data/data/com.termux/files/home/.suroot/busybox_arm64 && chmod 755 /data/data/com.termux/files/home/.suroot/busybox_arm64
```

*(This places the binary safely in the root home, separate from your main Termux environment.)*

## 📖 Usage

### Create a Sandbox
Create a fresh, clean environment.
```bash
# Create 'default' sandbox
termux-sandbox create

# Create a named sandbox (e.g., for python testing)
termux-sandbox create py-test
```

### Enter a Sandbox
Drop into the root shell of your isolated environment.
```bash
termux-sandbox enter
# OR
termux-sandbox enter py-test
```

### Delete a Sandbox
Completely remove a sandbox and free up space.
```bash
termux-sandbox delete py-test
```

## 📂 Inside the Sandbox

Once inside, the file system layout is standard Linux/Termux, with two special additions:

| Path | Description |
| :--- | :--- |
| `/sdcard` | Direct access to your internal storage. |
| `/host_root` | A **recursive mirror** of your real Android system. You can see `/data`, `/apex`, `/vendor` etc. inside here. |

> **⚠️ Warning:** You are **Root** inside the sandbox.
> *   Deleting files in `/sdcard` deletes them from your phone.
> *   Deleting files in `/host_root` **WILL BRICK** your phone.

## ❓ FAQ

### How it works

1.  **Chroot:** Changes the root directory to `~/.termux-sandbox/{name}`.
2.  **Mount Namespaces:** Uses `mount --rbind` to bring your Android Kernel directories (`/dev`, `/proc`, `/sys`) and partitions into the sandbox.
3.  **LD_PRELOAD:** Loads a compiled C library that intercepts `getuid()` calls. It tells applications "You are user 10000" even though you are Root, preventing permission errors common in Termux Chroots.

### Why not use `proot`?

Proot relies on `ptrace` to intercept system calls, which is slower and can have compatibility issues with multithreaded apps. This script uses native Linux `chroot`, which is faster and more stable but requires Root.

## ❤️ Credits & Attribution

*   **Busybox Static**: The `busybox_arm64` binary used in this project is provided by **EXALAB**.
    *   Source: [EXALAB/Busybox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64)
    *   It is used here to ensure a stable, static environment for the initial chroot process.
*   **Google Gemini**: Generate the whole `termux-sandbox` and the draft of this `README.md`.

## 📜 License

MIT License
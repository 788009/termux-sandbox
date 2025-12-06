# Termux Sandbox

Termux Sandbox 能够在现有的 Termux 安装中运行**隔离、纯净且具备原生性能**的 Termux 环境。  
本工具专为脚本测试、软件编译或维持主环境整洁而设计，无需依赖 `proot` 或容器运行时。

**注意**：本工具提供的是**环境**隔离。虽然默认模式下限制了对宿主文件的访问，但它**并非安全虚拟机**，请勿用于分析复杂的恶意软件。

![example-enter.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-enter.jpg?raw=true)

<details>
<summary>更多图片</summary>

![example-create.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-create.jpg?raw=true)

![example-use.jpg](https://github.com/788009/termux-sandbox/blob/main/images/example-use.jpg?raw=true)

</details>

## 特性

  - **原生执行**  
    直接在系统上运行，无模拟或 `proot` 开销，性能零损耗。

  - **独立环境**  
    支持创建多个相互独立的沙盒，便于区分不同的项目或实验。

  - **Root 权限适配**  
    内置 UID 伪装机制，确保 `apt`、`pkg` 等 Termux 软件包在真实的 Root 权限下也能正常运行。

  - **宿主资源映射**
    - **安全模式（默认）：** 无法访问 `/sdcard` 或宿主 Android 系统文件。
    - **无限制模式：** 可通过参数开启 `/sdcard` 和 `/host_root` 的完整映射。

  - **无侵入设计**  
    沙盒保持极简与隔离，除非显式操作，否则绝不修改或污染原本的 Termux 环境。

  - **复制、导出与导入**
    支持沙盒的复制、备份（导出），以及环境恢复与共享（导入），大幅简化了环境的配置和维护流程。

## 要求

  - 拥有 Root 权限的 Android 设备（Magisk 或 KernelSU，其他方案不保证）。
  - ARM64 (aarch64) 架构。
  - 已安装 Termux 应用。
  - BusyBox 静态二进制文件（下方提供）。

**若没有 Root 权限**，可以尝试 [Yonle/termux-proot](https://github.com/Yonle/termux-proot)。

## 安装

### 1. 安装基础依赖

```bash
pkg update
pkg install tsu curl zip clang -y
```

*（`clang` 用于编译 `fake_uid.c`，实现 UID 欺骗）*

### 2. 安装 termux-sandbox 脚本

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/v1.0/termux-sandbox" -o $PREFIX/bin/termux-sandbox
chmod +x $PREFIX/bin/termux-sandbox
```

## 使用方法

<details>
<summary>点击展开</summary>

**需要 `tsu` 环境**

`create`、`enter` 和 `delete` 命令若未指定名称，则实际名称为 `default`。

### 创建新沙盒

```bash
termux-sandbox create
# 或者指定名称
termux-sandbox create mysandbox
```

### 进入沙盒（两种模式）

#### 1. 安全模式（推荐）

默认情况下，沙盒是隔离的。你看不到 SD 卡或宿主系统文件。

```bash
termux-sandbox enter
# 或
termux-sandbox enter mysandbox
```

#### 2. 无限制模式（高级）

使用 `-b` 或 `--bind` 参数来挂载宿主目录（`/sdcard` 和 `/host_root`）。

```bash
termux-sandbox enter -b
# 或
termux-sandbox enter --b mysandbox
```

**无限制模式警告：**
当使用 `-b` 参数时，你拥有对物理文件系统的 **真实 Root 权限**。

  * 删除 `/sdcard` 中的文件会将其从手机中彻底删除。
  * 修改 `/host_root` 中的系统文件可能会导致手机**变砖**。


### 退出沙盒

按下 `Ctrl` + `D` 退出。

若因为某些原因无法退出沙盒，请重启设备（也欢迎提 issue）。

### 查看沙盒列表与状态

```bash
termux-sandbox list
```

### 删除沙盒

```bash
termux-sandbox delete
# 或者指定名称
termux-sandbox delete mysandbox
```

**警告：关于删除沙盒**

1.  **永远使用 `termux-sandbox delete` 命令来删除沙盒。**
2.  **切勿手动删除沙盒目录**（例如使用 `rm` 命令或文件管理器）。
      * *为什么？* 即使在安全模式下，`/system` 和 `/dev` 等系统目录依然处于挂载状态。手动删除目录会直接尝试删除手机的真实系统文件，这会导致手机**变砖**或系统崩溃。
3.  **关于卸载 Termux**
      * 请务必先**重启手机**。重启是强制断开所有挂载的唯一绝对安全的方法，防止安卓系统在卸载清理数据时误删底层文件。

### 重命名沙盒

```bash
termux-sandbox rename mysandbox yoursandbox
```

### 复制沙盒

```bash
termux-sandbox duplicate mysandbox mysandbox2
```

### 导出沙盒

```bash
termux-sandbox export mysandbox mysandbox.tar.gz
```

### 导入沙盒

```bash
termux-sandbox import mysandbox.tar.gz
```

</details>

## 实现原理

  * 利用 **Mount namespaces**（挂载命名空间）隔离环境，同时允许将特定的宿主路径重新绑定 (rebind) 到沙盒内。
  * 利用 **Chroot** 提供基于 Termux bootstrap 的最小化根文件系统。
  * 利用 **`LD_PRELOAD` 库** 拦截并覆盖少量的系统调用，向应用程序报告非 Root UID，从而使 `apt`、`pkg` 等 Termux 工具能正常运行。
  * 该设计完全避免了修改宿主 Termux 环境，确保每个沙盒都是自包含的。
  * 导出和导入时，会自动清理 APT 缓存，以及排除 `busybox`、`entry.sh`、挂载点等环境无关内容，尽量减小导出体积。

## 致谢

  * 静态 BusyBox 二进制文件由 [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64) 提供。
  * Termux bootstrap 包来自 [Termux 官方项目](https://github.com/termux/termux-packages)。

## 许可证

MIT License
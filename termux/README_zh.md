# Termux in Termux

[← 返回主页](../README_zh.md#支持的环境)

`termux-sandbox` 用于在 Termux 中创建全新且隔离的 Termux 环境，并提供完善的管理功能。

**目录**：[特性](#特性) | [运行要求](#运行要求) | [安装](#安装) | [使用方法](#使用方法) | [实现原理](#实现原理)

## 特性

- **原生执行**  
  直接在系统上运行，无模拟或 `proot` 开销，性能零损耗。

- **独立环境**  
  支持创建多个相互独立的沙盒，便于区分不同的项目或实验。

- **无侵入设计**  
  沙盒保持极简与隔离，除非显式操作，否则绝不修改或污染原本的 Termux 环境。

- **Root 权限适配**  
  内置 UID 伪装机制，确保 `apt`、`pkg` 等 Termux 软件包在真实的 Root 权限下也能正常运行。

- **网络连接**  
  开箱即用的网络连接。自动配置 DNS (`resolv.conf`) 和网络接口，确保 `pkg`、`pip` 和 `git` 等工具无需手动修补 host 即可直接工作。

- **宿主资源映射**  
  - **安全模式（默认）**： 无法访问 `/sdcard` 或宿主 Android 系统文件。
  - **无限制模式**： 可通过参数开启 `/sdcard` 和 `/host_root` 的完整映射。

- **复制、导出与导入**  
  支持沙盒的复制、备份（导出），以及环境恢复与共享（导入），大幅简化了环境的配置和维护流程。

## 运行要求

- 拥有 Root 权限的 Android 设备（Magisk 或 KernelSU，其他方案不保证）。
- ARM64 (aarch64) 或 ARM (armhf) 架构。
- 已安装 Termux 应用。

> [!NOTE]
> 
> 非 Root 用户可以尝试 [Yonle/termux-proot](https://github.com/Yonle/termux-proot)。

> [!IMPORTANT]
> 
> **关于内核兼容性的说明**
> 
> 本项目依赖 Linux 挂载命名空间（Mount Namespaces）。
> 
> 如果在 `tsu` 环境下运行时遇到以下错误：
> 
> ```
> unshare: Operation not permitted
> ```
> 
> 这表明由于内核配置、SELinux 策略或厂商限制，你的设备不支持向用户进程开放挂载命名空间。在这种情况下，沙盒无法在该设备上运行。
> 
> 你也可以在 `tsu` 环境中运行以下命令来预先验证兼容性：
> 
> ```bash
> unshare --mount /bin/true
> ```
> 
> 如果该命令返回错误，则说明你的内核不支持所需的隔离特性。

## 安装

### 1. 安装基础依赖

```bash
pkg update
pkg install tsu curl zip clang util-linux -y
```

> [!NOTE]
> 
>   * `clang` 用于编译 `fake_uid.c` 以实现 UID 欺骗。
>   * `util-linux` 提供 `unshare` 工具，在某些系统中可能已经预装。
>   * `tsu` 用于在以 Root 权限操作时保持 Termux 的环境变量（如 `$PATH`）。使用标准的 `su` 或 `su -i` 可能会导致依赖解析失败。


### 2. 安装 termux-sandbox 脚本

> [!WARNING]
> 
> 脚本必须安装在 Termux 宿主环境的前缀（`$PREFIX`）路径内。切勿安装到系统的 `/bin` 目录，该目录在 Android 上是只读的。

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/latest/termux-sandbox" -o "$PREFIX/bin/termux-sandbox"
chmod +x "$PREFIX/bin/termux-sandbox"
```

## 使用方法

> [!WARNING]
>
> 需要 `tsu` 环境

> [!TIP]
> 
> `create`、`enter` 和 `delete` 命令若未指定名称，则实际名称为 `default`。

### 查看版本

```bash
termux-sandbox version
termux-sandbox --version
```

### 创建沙盒

#### 1. 使用默认源

若没有指定 bootstrap（引导文件）来源，脚本会下载 Termux 官方的 bootstrap，且会自动判断当前架构（ARMHF/ARM64）。

当前的默认源来自[该 release](https://github.com/termux/termux-packages/releases/tag/bootstrap-2025.11.30-r1%2Bapt.android-7)。

```bash
termux-sandbox create
# 或者指定名称
termux-sandbox create mydevbox
```

#### 2. 使用自定义源

使用 `--source` 加上本地路径或下载链接来使用自定义 bootstrap（必须是 `.zip` 格式）。

```bash
# 使用本地路径
termux-sandbox create mysandbox --source /path/to/mybootstrap.zip
# 使用下载链接
termux-sandbox create mysandbox --source https://example.com/custom_bootstrap.zip
```

> [!TIP]
> 
> 当前的默认 bootstrap 并不总是最新版本，你可以在[这里](https://github.com/termux/termux-packages/releases)找到最新版本和其他官方版本。

> [!NOTE]
> 
> 使用自定义源时应自行确保 bootstrap 架构（`ARMHF`/`ARM64`）与你的设备匹配。

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

> [!CAUTION]
> 
> **无限制模式警告**
> 
> 当使用 `-b` 参数时，你拥有对物理文件系统的**真实 Root 权限**。
> 
>   * 删除 `/sdcard` 中的文件会将其从手机中彻底删除。
>   * 修改 `/host_root` 中的系统文件可能会导致手机**变砖**。

> [!WARNING]
>
> 若需要在沙盒内使用 `pkg`，请务必先 `pkg update`。

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

> [!TIP]
> 
> 务必始终使用 `termux-sandbox delete` 命令。这能确保正确清理锁文件和临时脚本。
> 
> 本工具使用了私有挂载命名空间 (Private Mount Namespaces)。这意味着在沙盒内部创建的挂载点对宿主系统是不可见的。虽然手动删除沙盒目录可以认为安全，但仍推荐使用内置的 delete 命令，以防止状态不一致。

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

这条命令会导出带有元数据（metadata）的完整沙盒环境。

### 导入沙盒

```bash
termux-sandbox import mysandbox.tar.gz
# 使用 --force 或 -f 以跳过元数据检查
termux-sandbox import mysandbox.tar.gz -f
```

### 卸载 `termux-sandbox`

```bash
termux-sandbox uninstall
```

## 实现原理

* **私有挂载命名空间 (`unshare`)** 用于隔离文件系统层级。沙盒内创建的挂载点对宿主不可见，从而防止在清理过程中意外导致宿主数据丢失。
* 利用 **Chroot** 提供基于 Termux bootstrap 的最小化根文件系统。
* 利用 **`LD_PRELOAD` 库**拦截并覆盖少量的系统调用，向应用程序报告非 Root UID，从而使 `apt`、`pkg` 等 Termux 工具能正常运行。
* **共享网络命名空间**允许沙盒直接复用宿主机的网络连接。
* **自动 DNS 配置**通过自动检测 DNS 连通性生成标准的 Linux `/etc/resolv.conf`，绕过了 Android 特有 DNS 属性在 Chroot 环境中经常失效的问题。
* 导出和导入时，会自动清理 APT 缓存，以及排除 `busybox`、`entry.sh`、挂载点等环境无关内容，尽量减小导出体积。

---

[返回主页](../README_zh.md#支持的环境)
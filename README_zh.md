# Termux Sandbox

Termux Sandbox 能够在现有的 Termux 安装中运行**隔离、纯净且具备原生性能**的 Termux 环境。  
该工具专为脚本测试、软件编译或维持主环境整洁而设计，无需依赖 `proot` 或容器运行时。

**注意**：该工具仅提供**环境**隔离，没有**权限**隔离，因此**严禁**用于恶意脚本测试。

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
      - `/sdcard` – 直接访问内部存储。
      - `/host_root` – 完整映射 Android 根文件系统（System, Data, Apex 等）。

  - **无侵入设计**  
    沙盒保持极简与隔离，除非显式操作，否则绝不修改或污染原本的 Termux 环境。

## 需求

  - 拥有 Root 权限的 Android 设备（Magisk 或 KernelSU，其他方案不保证）。
  - ARM64 (aarch64) 架构。
  - 已安装 Termux 应用。
  - BusyBox 静态二进制文件（下方提供）。

## 安装

### 1. 安装基础依赖

```bash
pkg update
pkg install tsu curl zip clang -y
```

### 2. 安装 termux-sandbox 脚本

```bash
curl -L "https://github.com/788009/termux-sandbox/releases/download/v1.0/termux-sandbox" -o $PREFIX/bin/termux-sandbox
chmod +x $PREFIX/bin/termux-sandbox
```

### 3. 安装静态 BusyBox

```bash
mkdir -p ~/.suroot
curl -L 'https://github.com/788009/termux-sandbox/releases/download/v1.0/busybox_arm64' -o ~/.suroot/busybox_arm64
chmod 755 ~/.suroot/busybox_arm64
```

## 使用方法

**需要 `tsu` 环境**

### 创建新沙盒

```bash
termux-sandbox create
# 或者指定名称
termux-sandbox create dev
```

### 进入沙盒

```bash
termux-sandbox enter
# 或者指定名称
termux-sandbox enter dev
```

### 删除沙盒

```bash
termux-sandbox delete
# 或者指定名称
termux-sandbox delete dev
```

## 特殊路径

进入沙盒后，文件系统布局为标准的 Linux/Termux 结构，并包含两个特殊路径：

| 路径 | 描述 |
| :--- | :--- |
| `/sdcard` | 直接访问手机的内部存储。 |
| `/host_root` | 真实 Android 系统的**递归镜像**。你可以在此查看到 `/data`, `/apex`, `/vendor` 等目录。 |

> **警告：** 你在沙盒内拥有 **Root** 权限。
>
>   * 删除 `/sdcard` 中的文件会将其从手机中彻底删除。
>   * 删除 `/host_root` 中的文件 **会导致手机变砖**。

## 实现原理

  * 利用 **Mount namespaces**（挂载命名空间）隔离环境，同时允许将特定的宿主路径重新绑定 (rebind) 到沙盒内。
  * 利用 **Chroot** 提供基于 Termux bootstrap 的最小化根文件系统。
  * 利用 **LD\_PRELOAD 库** 拦截并覆盖少量的系统调用，向应用程序报告非 Root UID，从而使 `apt`、`pkg` 等 Termux 工具能正常运行。
  * 该设计完全避免了修改宿主 Termux 环境，确保每个沙盒都是自包含的。

## 致谢

  * 静态 BusyBox 二进制文件由 [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64) 提供。
  * Termux bootstrap 包来自 [Termux 官方项目](https://github.com/termux/termux-packages)。

## 许可证

MIT License
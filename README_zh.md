# Termux Sandbox

Termux Sandbox 能够在现有的 Termux 中运行**隔离、纯净且具备原生性能**的 Linux 环境，目前支持运行全新的 Termux 和 Ubuntu。  
本工具专为脚本测试、软件编译或维持主环境整洁而设计，无需依赖 `proot` 或容器运行时。

> [!CAUTION] 
> 本工具提供的是**环境**隔离。虽然默认模式下限制了对宿主文件的访问，但它**并非安全虚拟机**，请勿用于分析复杂的恶意软件。

![example-enter.jpg](images/example-enter.jpg?raw=true)

<details>
<summary>更多图片（以 Termux in Termux 为例）</summary>

![example-create.jpg](images/example-create.jpg?raw=true)

![example-use.jpg](images/example-use.jpg?raw=true)

</details>

## 核心特色

1. **Chroot 带来的原生性能**：使用 Chroot 而非 Proot 创建隔离环境，实现以原生效率在沙盒内执行操作，但也因此要求 Root 环境。
2. **自动化环境管理**：自动处理原生 Chroot 环境、网络配置、挂载点以及权限设置。
3. **类似 Docker 的导出与导入**：支持像 Docker 一样轻松备份、分享或迁移你的整个沙盒环境。

## 支持的环境

请根据需求选择对应的沙盒环境：

- [**Termux** in Termux](termux/README_zh.md)
- [**Ubuntu** in Termux](ubuntu/README_zh.md) (或许也兼容其他发行版)

## 致谢

* 静态 BusyBox 二进制文件由 [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64) 提供。
* Termux bootstrap 包来自 [Termux 官方项目](https://github.com/termux/termux-packages)。
* Ubuntu Base 包来自 [Ubuntu 官方项目](https://cdimage.ubuntu.com/ubuntu-base)。

## 许可证

MIT License
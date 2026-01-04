# Termux Sandbox

[中文](README_zh.md)

Termux Sandbox provides native-performance Linux environments within Termux, bypassing the overhead of Proot while automating the tedious setup of traditional Chroot (networking, mounts, and permissions).

It is the ideal "one-click" solution for high-performance software building or maintaining clean, isolated sandboxes for Termux and Ubuntu (and more).

> [!CAUTION] 
> This tool only provides **environment** isolation, not **security** isolation. Therefore, it **must not** be used for malicious script testing.

![example-enter.jpg](images/example-enter.jpg?raw=true)

<details>
<summary>More images (take Termux in Termux as an example)</summary>

![example-create.jpg](images/example-create.jpg?raw=true)

![example-use.jpg](images/example-use.jpg?raw=true)

</details>

## Core Features

1. **Native Performance via Chroot**: Leveraging Chroot instead of Proot to create isolated environments, achieving operations with native efficiency—though root access is required as a result.
2. **Automated Environment Management**: Automatically handles the native Chroot environment, network configurations, mount points, and permissions.
3. **Docker-like Export/Import**: Easily backup, share, or migrate your entire sandbox environment with Docker-like export and import capabilities.

## Supported Environments

Please choose a sandbox environment based on your needs:

- [**Termux** in Termux](termux/README.md)
- [**Ubuntu** in Termux](ubuntu/README.md) (potentially compatible with other distributions)

## Credits

* Static BusyBox binary provided by [EXALAB/BusyBox-static](https://github.com/EXALAB/Busybox-static/blob/main/busybox_arm64).
* Termux bootstrap packages from the [official Termux project](https://github.com/termux/termux-packages).
* Ubuntu Base packages from the [official Ubuntu project](https://cdimage.ubuntu.com/ubuntu-base)

## License

MIT License

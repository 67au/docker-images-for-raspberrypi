# Docker Images For Raspberry Pi

适用于 Raspberry Pi 64-bit OS 的 Docker 镜像合集

## 项目背景

该项目来源于日常在 Raspberry Pi 上开发/部署项目遇到问题的解决方法。使用过 Raspberry Pi 3B+ ，也使用过 Raspberry Pi 4B ，在使用过程中遇到了很多问题，不限于下面：

1. 使用 sudo make install ，安装程序/依赖后导致系统组件故障
2. 使用 64-bit OS 时，不支持使用相机程序
3. 使用 Arch Linux Arm 时，不便于使用 debian package
4. 使用 Arch Linux Arm 时， linux-aarch64 不支持运行 32-bit 的用户程序
5. 使用 Debian / Raspberry Pi OS / Ubuntu 时，没有最新的依赖/程序

关于第 1 点的解决方法是 docker / [systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn) ，使用轻量级的虚拟化环境去运行相关依赖的程序，或者是编写依赖/项目的 PKGBUILD 使用 Arch Linux 的 makepkg 构建软件包，达到管理依赖的目的；关于第 2 点的解决方法是安装 32-bit OS 运行相机程序；关于第 3 点的解决方案是安装 Debian-based OS，或者是使用 [debtap](https://github.com/helixarch/debtap) / 自行编写 PKBUILD 转换；关于第 4 点的解决方法是更换内核或者使用 qemu-user 运行 32-bit 的用户程DD

综合上面方案，总结出满意的方案：使用 Aarch64 架构的 Arch Linux Arm，使用 docker / systemd-nspawn 运行 32-bit 的 Debian 容器来运行相机程序或安装 deb 软件包，同时使用 btrfs 的快照功能备份 /var/lib/{docker,machines} 

所以本项目作为一个储存 Dockerfile 的仓库

## 镜像目录

- [raspberrypi-userland](docker/raspberrypi-userland/README.md)

## 使用方法

### 安装 aarch64 架构的 Arch Linux Arm 

安装的方法可以依照 [这里](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4) ，可选使用 btrfs 文件系统，推荐布局参考 [这里](https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout)  ， 使用方法参考 [这里](https://wiki.archlinux.org/title/Btrfs) 

### 更换内核

安装 [linux-rasapberry4](https://archlinuxarm.org/packages/aarch64/linux-raspberrypi4) 替换 [linux-aarch64](https://archlinuxarm.org/packages/aarch64/linux-aarch64) ，linux-raspberry4 是 Raspberry Pi Foundation 提供的 [内核](https://github.com/raspberrypi/linux) ，支持 32-bit 和 64-bit 的用户程序。

```
pacman -Syu linux-raspberrypi4
```

安装内核后记得检查 `/boot/comline.txt` ，确保 `root` 、 `rootfstype` 、 `rootflags` 这三个参数正确

### 安装 Docker

安装与启用 Docker

```
pacman -Syu docker
systemctl enable --now docker
```

使用镜像加速器的方法可以参考 @yeasy 的《[Docker —— 从入门到实践](https://yeasy.gitbook.io/docker_practice/install/mirror)》

### 搭建私有 Registry [可选]

在使用 `docker-container` driver 的 builder 构建自定义镜像时，是无法使用本地储存的镜像，所以就需要搭建私有的 [Registry](https://hub.docker.com/_/registry) 储存本地镜像，让 builder 构建镜像时使用

```
docker run -d -p 5000:5000 --restart=always --name registry registry:2 --mount type=bind,src=<your directory>,dst=/var/lib/registry
```

### 构建镜像

现在可以构建需要的镜像了，详细可以查看 [镜像目录](#镜像目录) 

## 注意事项

1. 在 aarch64 架构的系统上使用 `buildx` 构建镜像时，需要指定 `--platform linux/arm64` ，确保作为 base image 时可以被正确识别
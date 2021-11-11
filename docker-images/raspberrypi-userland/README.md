# raspberrypi-userland

## 问题背景

在 aarch64 架构的 Arch Linux Arm 在安装 `alarm/raspberrypi-firmware` 后，却发现 `raspistill` 这个命令不存在。详细查看 raspberrypi/userland@f97b1af1b3e653f9da2c1a3643479bfd469e3b74 后，可以知道项目撤销了 64-bit 的 MMAL 与 MMAL_APP 的支持，因而需要使用 ARM 编译工具构建。

## 使用方法

### 启用相机

将下面内容添加到 `/boot/config.txt` 中

```
gpu_mem=128 
start_file=start4x.elf 
fixup_file=fixup4x.dat
```

将下面内容添加到 `/etc/modprobe.d/rpi-camera.conf` 中（启用 v4l2 驱动）

```
bcm2835-v4l2
options bcm2835-v4l2 max_video_width=3240 max_video_height=2464
```

详情可以参考 Arch Linux Arm 的 [wiki](https://archlinuxarm.org/wiki/Raspberry_Pi) 和 Raspberry Pi 官网的 [document](https://www.raspberrypi.com/documentation/accessories/camera.html#getting-started)

### 构建镜像

```
docker build . -t raspberrypi-userland:latest
```

### 测试容器

运行容器时需要映射 `/dev/vchiq` 与 `/dev/vcsm-cma` 这两个设备

```
docker run -it --rm \
    --name=rpi-userland \
    --device=/dev/vchiq \
    --device=/dev/vcsm-cma \
    --mount type=bind,src=$(pwd)/output,dst=/root/output \
    raspberrypi-userland:latest \
    /bin/sh -c "raspistill -v -o /root/output/test.jpg"
```

运行后看到容器 `raspistill` 详细的调试输出，当前文件夹会生成拍照生成的 test.jpg ，检查拍照结果是否异常

## 注意事项


---
title: Hi3518 摄像头模块逆向使用
published: 2026-05-02 11:34:38
description: 记录一次通过 UART、U-Boot、nmap 和 RTSP 把拆机 Hi3518 摄像头模块重新用起来的过程。
tags: [Hi3518, 摄像头, 逆向]
category: 折腾
draft: false
---

# Hi3518 摄像头模块逆向使用

很久之前，我在闲鱼上买过一个 Hi3518 摄像头模块。严格来说它不太像单独的开发模块，更像是从雄迈监控设备里拆出来的板子。今天突然想起来，就顺手试了一下能不能把它重新用起来。

这篇文章主要记录这次折腾过程：先找到 UART 串口，再从 U-Boot 里确认 IP 地址，最后通过 RTSP 拿到视频流。

![Hi3518 摄像头模块正面](https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown20260502183938512.png)

模块整体大概是这样：一个摄像头模组、一根电源线和一根网线。供电需要 12V 圆口电源，串口转接器是我后面自己接上去的。

![Hi3518 摄像头模块整体连接](https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown20260502184456145.png)

## 1. 先判断摄像头类型

海思 Hi3518 系列一般用于嵌入式 IP 摄像头。它和普通 USB 摄像头不太一样，主要特点是：

- 带硬件视频编码器（H.264/H.265），适合网络传输。
- 有专门的固件和操作系统（通常是 Linux）。
- 输出接口通常不是标准 USB，而是：
  - 以太网接口（RJ45）：直接作为 IP 摄像头接入网络。
  - CVBS/HDMI/BT.656 等视频接口：用于专业录像机或相关设备。
  - 串口调试接口（UART）：用于刷固件或调试。
- 并非即插即用的普通 UVC 摄像头。

所以这次的目标也很明确：不走 USB，而是通过网口把视频流拿到手。

## 2. 找到 UART 串口

想要拿到视频流，第一步是确认摄像头的 IP 地址。比较理想的方式是通过串口进入 Linux 系统，然后直接查看网络配置。因此我先尝试找出板子上的 UART 串口。

我参考了一篇同样关于 Hi3518 模组的博文，但它的 PCB 布局和我手上的这块不完全一样。最后还是靠万用表测了一下几个未接线端口的电压，大致确认了 UART 的位置。

![疑似 UART 焊盘位置](https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown20260502185127525.png)

UART 引脚如下图，串口波特率设置为 `115200`。

![UART 引脚连接方式](https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown20260502185631513.png)

## 3. 串口启动情况

本来以为接上 UART 之后就能顺利进入系统，但实际启动日志停在了 `booting the kernel` 附近：

```bash
U-Boot 2010.06-svn (Aug 31 2015 - 13:10:52)

DRAM:  256 MiB
Check spi flash controller v350... Found
Spi(cs1) ID: 0xC2 0x20 0x17 0xC2 0x20 0x17
Spi(cs1): Block:64KB Chip:8MB Name:"MX25L6406E"
envcrc 0xded2d67
ENV_SIZE = 0xfffc
In:    serial
Out:   serial
Err:   serial
Press Ctrl+C to stop autoboot
CFG_BOOT_ADDR:0x58040000
8192 KiB hi_sfc at 0:0 is now current device

### boot load complete: 1969140 bytes loaded to 0x82000000
### SAVE TO 80008000 !
## Booting kernel from Legacy Image at 82000000 ...
   Image Name:   linux
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    1969076 Bytes = 1.9 MiB
   Load Address: 80008000
   Entry Point:  80008000


load=0x80008000,_bss_end=80829828,image_end=801e8bb4,boot_sp=807c7168
   Loading Kernel Image ... OK
OK

Starting kernel ...

Uncompressing Linux... done, booting the kernel.
```

既然 Linux 没正常起来，就先进入 U-Boot 命令行看一下环境变量。输入 `printenv` 后，可以看到里面已经写好了网络相关配置：

```bash
hisilicon # printenv
bootcmd=setenv setargs setenv bootargs ${bootargs};run setargs;fload;bootm 0x82000000
bootdelay=1
baudrate=115200
bootfile="uImage"
da=mw.b 0x82000000 ff 1000000;tftp 0x82000000 u-boot.bin.img;sf probe 0;flwrite
du=mw.b 0x82000000 ff 1000000;tftp 0x82000000 user-x.cramfs.img;sf probe 0;flwrite
dr=mw.b 0x82000000 ff 1000000;tftp 0x82000000 romfs-x.cramfs.img;sf probe 0;flwrite
dw=mw.b 0x82000000 ff 1000000;tftp 0x82000000 web-x.cramfs.img;sf probe 0;flwrite
dc=mw.b 0x82000000 ff 1000000;tftp 0x82000000 custom-x.cramfs.img;sf probe 0;flwrite
up=mw.b 0x82000000 ff 1000000;tftp 0x82000000 update.img;sf probe 0;flwrite
ua=mw.b 0x82000000 ff 1000000;tftp 0x82000000 upall_verify.img;sf probe 0;flwrite
tk=mw.b 0x82000000 ff 1000000;tftp 0x82000000 uImage; bootm 0x82000000
dd=mw.b 0x82000000 ff 1000000;tftp 0x82000000 mtd-x.jffs2.img;sf probe 0;flwrite
ipaddr=192.168.1.10
serverip=192.168.1.107
netmask=255.255.255.0
osmem=43M
ethaddr=00:12:14:22:42:d4
HWID=8043420004048425
NID=0x0005
appSystemLanguage=SimpChinese
appVideoStandard=PAL
bootargs=mem=${osmem} console=ttyAMA0,115200 root=/dev/mtdblock1 rootfstype=cramfs mtdparts=hi_sfc:256K(boot),3520K(romfs),2560K(user),1280K(web),256K(custom),320K(mtd) earlyprintk=serial,ttyAMA0,115200
stdin=serial
stdout=serial
stderr=serial
verify=n
ver=U-Boot 2010.06-svn (Aug 31 2015 - 13:10:52)

Environment size: 1342/65532 bytes
```

这里能看到设备 IP 是 `192.168.1.10`，`serverip` 是 `192.168.1.107`。后面拉流主要用到的是设备 IP。

## 4. 扫描开放端口

确认 IP 之后，用 `nmap` 扫描一下常见端口，看看设备对外开放了哪些服务：

```bash
❯ sudo nmap -Pn -p 554,80,8899,34567 192.168.1.10
[sudo] password for tz:
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-02 19:16 +0800
Nmap scan report for 192.168.1.10
Host is up (0.000082s latency).

PORT      STATE SERVICE
80/tcp    open  http
554/tcp   open  rtsp
8899/tcp  open  ospf-lite
34567/tcp open  dhanalakshmi

Nmap done: 1 IP address (1 host up) scanned in 3.83 seconds
```

结果里比较关键的是：

- `80/tcp`：HTTP 服务。
- `554/tcp`：RTSP 服务。
- `8899/tcp` 和 `34567/tcp`：这类监控设备常见的私有服务端口。

我们要拿的是视频流，所以重点看 `554` 端口的 RTSP 服务。

## 5. 通过 RTSP 拉取视频流

使用 `ffplay` 直接访问 RTSP 地址：

```bash
ffplay -rtsp_transport tcp 'rtsp://192.168.1.10:554/user=admin_password=_channel=1_stream=0.sdp?real_stream'
```

命令跑起来之后，就能看到摄像头画面了。整体结论是：能用，但延迟比较明显，后面如果要长期使用，还需要继续调编码、码率或者传输方式。

![ffplay 播放 RTSP 视频流](https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown20260502193122649.png)

## 总结

这块拆机 Hi3518 摄像头模块最终可以通过网口重新使用起来，最小流程是：

- 找到 UART 串口，并确认波特率为 `115200`。
- 进入 U-Boot，通过 `printenv` 找到设备 IP。
- 用 `nmap` 确认 `554` 端口开放。
- 使用 `ffplay` 拉取 RTSP 视频流。

这次先做到能出画面为止。后续如果要继续优化，重点应该放在降低延迟和确认更稳定的 RTSP 地址格式上。

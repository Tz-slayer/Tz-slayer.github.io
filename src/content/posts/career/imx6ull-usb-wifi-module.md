---
title: IMX6ULL 使用 USB WIFI 模块
published: 2025-09-13 10:59:44
description: IMX6ULL 使用 USB WIFI 模块
tags: [学习]
category: 社畜
draft: false
---

# 1. 硬件介绍

USB WIFI 模块使用的是正点原子的，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20250913030519-1757732719633.png"/>

IMX6ULL 开发板使用的也是正点原子的，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20250913030716-1757732836225.png"/>

# 2. 使用

开发板上随便找个 USB 口插上，使用串口查看控制台打印出来的信息，

```cmd
[ 1658.663663] usb 1-1.1: new high-speed USB device number 4 using ci_hdrc
[ 1658.784106] bFWReady == _FALSE call reset 8051...
[ 1658.834375] RTL871X: rtw_ndev_init(wlan0)
[ 1659.444637] ==> rtl8188e_iol_efuse_patch
[ 1659.833032] IPv6: ADDRCONF(NETDEV_UP): wlan0: link is not ready
```

看到最后的 wlan0: link is not ready 心凉了一半，使用 lsusb 命令查看已连接的 USB 设备信息，可以看到设备的总线号、设备号和厂商 ID 以及产品 ID，

```cmd
root@ATK-IMX6U:~/start# lsusb
Bus 001 Device 006: ID 0bda:8179
Bus 001 Device 002: ID 1a40:0101
Bus 001 Device 001: ID 1d6b:0002
```

由于我插入的是从左往右第一个 USB 口，所以使用第一个 USB 信息来查看详细的信息，`-d` 参数明确厂商 ID 和产品 ID，`-v` 参数表示输出详细信息，

```cmd
root@ATK-IMX6U:~/start# lsusb -d 0bda:8179 -v

Bus 001 Device 006: ID 0bda:8179
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x0bda
  idProduct          0x8179
  bcdDevice            0.00
  iManufacturer           1 Realtek
  iProduct                2 802.11n NIC
  iSerial                 3 00ADA7200012
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           39
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower              500mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass       255
      bInterfaceSubClass    255
      bInterfaceProtocol    255
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
Device Qualifier (for other device speed):
  bLength                10
  bDescriptorType         6
  bcdUSB               2.00
  bDeviceClass            0
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  bNumConfigurations      1
can't get debug descriptor: Resource temporarily unavailable
Device Status:     0x0002
  (Bus Powered)
  Remote Wakeup Enabled
```

接口类型为 255（Vendor Specific，厂商自定义），说明该设备需要专用驱动程序（而非通用 USB 类驱动），这款 wifi 模块使用的芯片是 RTL8188EUS，先检查一下内核中是否自带驱动，如果没有自带驱动需要从外部下载相关驱动，

```bash
root@ATK-IMX6U:~/start# find /lib/modules/$(uname -r) -name "*8188*"
/lib/modules/4.1.15-g3dc0a4b/kernel/drivers/net/wireless/rtlwifi/rtl8188EUS
/lib/modules/4.1.15-g3dc0a4b/kernel/drivers/net/wireless/rtlwifi/rtl8188EUS/8188eu.ko
```

似乎找到了，这个 8188wu.ko 应该就是对应的驱动内核模块，使用 lsmod 命令查看到这个模块也被正确加载了，

```bash
root@ATK-IMX6U:~/start# lsmod | grep 8188eu
8188eu                795187  0
```

使用 ip link show 命令也能正常查看到 wlan0，所以难道前面的 not ready 其实只是正常的输出？

```bash
root@ATK-IMX6U:~# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: can0: <NOARP,ECHO> mtu 16 qdisc noop state DOWN mode DEFAULT group default qlen 10
    link/can
3: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
    link/ether 88:4b:55:27:22:41 brd ff:ff:ff:ff:ff:ff
4: eth1: <NO-CARRIER,BROADCAST,MULTICAST,DYNAMIC,UP> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
    link/ether 88:2e:bf:6f:90:54 brd ff:ff:ff:ff:ff:ff
5: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN mode DEFAULT group default
    link/sit 0.0.0.0 brd 0.0.0.0
10: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:ad:a7:20:00:12 brd ff:ff:ff:ff:ff:ff
```

## 3. 连接 WIFI

折腾了半天还是使用正点原子官方提供的脚本吧，

```bash title="alientek_usb_wifi_setup.sh"
root@ATK-IMX6U:~/shell/wifi# cat alientek_usb_wifi_setup.sh
#!/bin/sh
#正点原子@ALIENTEK
#USB WIFI脚本
#功能:脚本支持station模式、softap模式、bridge模式相互切换（若相互切换不成功，请重启板子）

#使用方法说明
usage_and_exit() {
        echo ""
        echo "*****************************************************************************************"
        echo "*usage: ${0##*/} [-m mode] [-i ssid] [-p psk] [-d device] [-e ethernet] [-h]*"
        echo "*                                                                                       *"
        echo "*station mode eg:${0##*/} -m station -i ssid -p psk -d wlan0                *"
        echo "*                                                                                       *"
        echo "*softap mode eg: ${0##*/} -m softap -d wlan0                                *"
        echo "*                                                                                       *"
        echo "*bridge mode eg:${0##*/} -m bridge -d wlan0 -e eth0                         *"
        echo "*eg: ${0##*/} -m mode mode=[ station | softap | bridge ]                    *"
        echo "*****************************************************************************************"
        echo ""
    return
}

#使用方法举例：
#source ./alientek_usb_wifi_setup.sh -m bridge -d wlan0 -e eth0
#source ./alientek_usb_wifi_setup.sh -m station -i ALIENTEK -p 15902020353 -d wlan0
#source ./alientek_usb_wifi_setup.sh -m softap -d wlan0


#版本信息
version() {
        echo "version 1.1"
        echo "作者:DZM@ALIENTEK"
}

#获取参数
while [ $# -gt 0 ]; do
  case $1 in
    --help | -h)
      usage_and_exit $0;break;;
    -m) shift; mode=$1; shift; ;;
    -i) shift; ssid=$1; shift; ;;
        -p) shift; psk=$1; shift; ;;
        -d) shift; device=$1; shift; ;;
        -e) shift; ethernet=$1; shift; ;;
    --version) version $0; break;;
    *) usage_and_exit; break; ;;
  esac
done

#判断参数
if [ -n "$mode" ]; then
echo "您的WIFI配置信息是："
echo "mode  : $mode"
fi

if [ -n "$ssid" ]; then
echo "ssid  : $ssid"
echo "psk   : $psk"
fi

if [ -n "$device" ]; then
echo "device: $device"
fi

if [ -n "$ethernet" ]; then
echo "ethernet  : $ethernet"
fi

#kill掉相关进程
processkill()
{
a=$(ps -aux |grep -E "hostapd" | grep -v grep | awk '{print $2}')
if [ -n "$a" ];then
        kill -9 $a
fi

a=$(ps -aux |grep -E "hostapd-rtl871xdrv" | grep -v grep | awk '{print $2}')
if [ -n "$a" ];then
        kill -9 $a
fi

a=$(ps -aux |grep -E "udhcpd" | grep -v grep  | awk '{print $2}')
if [ -n "$a" ];then
        kill -9 $a
fi

a=$(ps -aux |grep -E "wpa_supplicant" | grep -v grep | awk '{print $2}')
if [ -n "$a" ];then
        kill -9 $a
fi

a=$(ps -aux |grep -E "udhcpc" | grep -v grep | awk '{print $2}')
if [ -n "$a" ];then
        kill -9 $a
fi
}

#创建配置wifi的信息
touch /$PWD/wifi.conf
wifi_conf="/$PWD/wifi.conf"

#bridge Mode(桥接模式)
if [ "$mode" == "bridge" ]; then
        ifconfig $device down
        ifconfig $device up
        ifconfig $ethernet down
        ifconfig $ethernet up
        sleep 2
        if [ "$WIFI_MODE" == "station" ]; then
                wpa_supplicant -B -D wext -i $device -c /etc/wpa_supplicant.conf
        fi
        processkill
        sleep 1
        sync
    echo -ne "interface=wlan0\nssid=alientek_bridge\ndriver=rtl871xdrv\nchannel=6\nhw_mode=g\nignore_broadcast_ssid=0\n
auth_algs=1\nwpa=3\nwpa_passphrase=12345678\nwpa_key_mgmt=WPA-PSK\nwpa_pairwise=TKIP\nrsn_pairwise=CCMP\nbridge=br0" > $wifi_conf
        rm -rf /var/lib/misc/*
        touch /var/lib/misc/udhcpd.leases
        udhcpd -fS /etc/udhcpd.conf &
        ifconfig $device 0.0.0.0
        brctl addbr br0
        ifconfig $ethernet 0.0.0.0
        brctl addif br0 $ethernet
        brctl addif br0 $device
        ifconfig br0 192.168.1.39 netmask 255.255.255.0
        hostapd-rtl871xdrv $wifi_conf -B
        export WIFI_MODE=bridge
fi

#softap Mode（热点模式）
if [ "$mode" == "softap" ]; then
        processkill
        sleep 1
        sync
    echo -ne "interface=wlan0\nssid=alientek_softap\ndriver=rtl871xdrv\nchannel=6\nhw_mode=g\nignore_broadcast_ssid=0\n
auth_algs=1\nwpa=3\nwpa_passphrase=12345678\nwpa_key_mgmt=WPA-PSK\nwpa_pairwise=TKIP\nrsn_pairwise=CCMP" > $wifi_conf
        a=$(ifconfig |grep -E "br0" | grep -v grep | awk '{print $0}')
        if [ -n "$a" ];then
                brctl delif br0 $device
                brctl delif br0 $device
                ifconfig br0 down
        fi
        rm -rf /var/lib/misc/*
        touch /var/lib/misc/udhcpd.leases
        ifconfig $device up
        sleep 2
        ifconfig $device 192.168.1.38 netmask 255.255.255.0
        udhcpd -fS /etc/udhcpd.conf     &
        hostapd-rtl871xdrv $wifi_conf -B
        export WIFI_MODE=softap
fi

#station Mode（上网模式）
if [ "$mode" == "station" ]; then
        processkill
        sleep 1
        sync
        a=$(ifconfig |grep -E "br0" | grep -v grep | awk '{print $0}')
        if [ -n "$a" ];then
                brctl delif br0 $device
                brctl delif br0 $device
                ifconfig br0 down
        fi
    echo -ne "ctrl_interface=/var/run/wpa_supplicant\n
update_config=1\nnetwork={\nssid=\"$ssid\"\npsk=\"$psk\"\n}\n" > $wifi_conf
        rm -rf /var/lib/misc/*
        ifconfig eth0 down
        ifconfig eth1 down
        ifconfig wlan0 down
        ifconfig wlan0 up
        sleep 2
        wpa_supplicant -B -D wext -i $device -c $wifi_conf
        udhcpc -R -b -i wlan0
        route add default gw 192.168.1.1 dev $device
        export WIFI_MODE=station
fi

#删除wifi.conf
rm -rf /$PWD/wifi.conf
case $mode in
    softap|station|bridge)echo "WIFI设置$mode模式完成!"
    ;;

esac

#卸载环境变量
unset device
unset wifi_conf
unset a
unset ethernet
unset mode
unset ssid
unset psk

sync
```

连接 wifi 使用以下命令，`-i` 参数指定 wifi 名称，`-p` 参数指定密码。

```bash
source ./alientek_usb_wifi_setup.sh -m station -i ALIENTEK-YF -p 1590202**** -d wlan0
```
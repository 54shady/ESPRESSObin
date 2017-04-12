# ESPRESSObin

## SD卡启动系统

查看SD卡状态(下面是笔者主机信息)

	lsblk
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0 931.5G  0 disk
	├─sda1   8:1    0    50G  0 part
	├─sda2   8:2    0     1K  0 part
	├─sda5   8:5    0   673G  0 part /home
	├─sda6   8:6    0 517.7M  0 part /boot
	├─sda7   8:7    0   200G  0 part /
	└─sda8   8:8    0     8G  0 part [SWAP]
	sdb      8:16   1  14.9G  0 disk
	└─sdb1   8:17   1  14.9G  0 part

清除SD卡(这里假设是/dev/sdb)内所有数据,可以用lsblk查看SD卡状态

	sudo dd if=/dev/zero of=/dev/sdb bs=1M count=100

在SD上创建一个分区(sdb1)

	(echo n; echo p; echo 1; echo ''; echo ''; echo w) | sudo fdisk /dev/sdb

格式化分区为EXT4格式

	sudo mkfs.ext4 /dev/sdb1

把SD卡挂在到开发主机的/mnt/sdcard目录下

	sudo mkdir -p /mnt/sdcard
	sudo mount /dev/sdb1 /mnt/sdcard

将根文件系统(buildroot, ubuntu, yocto)解压到/mnt/sdcard/下(这里统一用rootfs.tar.gz表示)

	sudo tar -xvf rootfs.tar.gz -C /mnt/sdcard/

将内核和DTB拷贝到根文件系统的boot目录下

	sudo mkdir -p /mnt/sdcard/boot
	sudo cp Image /mnt/sdcard/boot/
	sudo cp marvell/armada-3720-community.dtb /mnt/sdcard/boot/

卸载SD卡

	sudo umount /mnt/sdcard

设置Uboot环境变量

	setenv image_name boot/Image
	setenv fdt_name boot/armada-3720-community.dtb
	setenv bootmmc 'mmc dev 0; ext4load mmc 0:1 $kernel_addr $image_name;ext4load mmc 0:1 $fdt_addr $fdt_name;setenv bootargs $console root=/dev/mmcblk0p1 rw rootwait; booti $kernel_addr - $fdt_addr'
	save

使用bootmmc启动开发板

	run bootmmc

## Ubuntu 14.04根文件系统制作

[ubuntu-base-14.04-core-arm64.tar.gz下载地址](http://cdimage.ubuntu.com/ubuntu-base/releases/14.04/release/)

将ubuntu core解压到本地

	mkdir ubunturootfs
	sudo tar xzvf ubuntu-base-14.04-core-arm64.tar.gz -C ubunturootfs/

修改默认启动级别(etc/init/rc-sysinit.conf)

	DEFAULT_RUNLEVEL=3

去除root用户登录密码(etc/passwd)

	root::0:0:root:/root:/bin/bash

创建一个串口初始化配置文件(etc/init/ttyMV0.conf)

	start on stopped rc or RUNLEVEL=[12345]
	stop on runlevel [!12345]
	respawn
	exec /sbin/getty -L 115200 ttyMV0 vt100 -a root

将内核和DTB拷贝到根文件系统的boot目录下(制作好的文件系统就可以直接拷贝到SD卡中使用)

	cp Image boot/
	cp armada-3720-community.dtb boot/

## 使用过程中遇到的问题和解决办法

Ubunt14.04网络配置

手动配置(开机不会在配置网络上卡吨)

	TBD

使用DHCP配置(添加下面内容到/etc/network/interfaces)

	auto wan
	iface wan inet dhcp

使用DHCP配置开机时配置始终不成功日志如下

	Waiting for network configuration...
	Waiting up to 60 more seconds for network configuration...
	Booting system without full network configuration...

进入系统后手动开启网卡,顺序不能颠倒,否则wan始终打不开

	ifconfig eth0 up
	ifconfig wan up

查看此时的路由表

	root@localhost:~# route
	Kernel IP routing table
	Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
	default         192.168.1.1     0.0.0.0         UG    0      0        0 wan
	192.168.1.0     *               255.255.255.0   U     0      0        0 wan

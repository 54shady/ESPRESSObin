# ESPRESSObin

## SD卡启动Buildroot系统

清除SD卡(这里假设是/dev/sdb)内所有数据,可以用lsblk查看SD卡状态

	sudo dd if=/dev/zero of=/dev/sdb bs=1M count=100

在SD上创建一个分区(sdb1)

	(echo n; echo p; echo 1; echo ''; echo ''; echo w) | sudo fdisk /dev/sdb

格式化分区为EXT4格式

	sudo mkfs.ext4 /dev/sdb1

把SD卡挂在到开发主机的/mnt/sdcard目录下

	sudo mkdir -p /mnt/sdcard
	sudo mount /dev/sdb1 /mnt/sdcard

将buildroot文件系统解压到/mnt/sdcard/下

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

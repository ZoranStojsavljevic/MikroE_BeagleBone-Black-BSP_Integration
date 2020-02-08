# am335x beaglebone-black-bsp
This repository is related to Texas Instruments Open Source Beaglebone Black referent development platform.

## am335x BeagleBone Black evaluation board setup

BeagleBone Black System Reference Manual

https://cdn-shop.adafruit.com/datasheets/BBB_SRM.pdf

element14 BeagleBone Black INDUSTRIAL System Reference Manual 

http://download.kamami.pl/p562276-BBBI_SRM_Rev%201.0%20VL.pdf

## Get u-boot

The latest u-boot version might not work out of the box.

	git clone http://git.denx.de/u-boot.git u-boot-bbb OR
	git clone git://git.denx.de/u-boot.git u-boot-bbb
	cd u-boot-bbb

The cloned version will match HEAD set to master branch (the following command is optional):

	git checkout -b <branch-name> # Take the current HEAD (the commit checked
	# out) and create a new branch called <branch-name>

## Installing arm cross compiler on the host (the host used is Fedora 31 distro)

To install on Fedora 31 gcc-arm-linux-gnu.x86_64 (which is not native x86_64 compiler), the following command is used:

	sudo dnf install gcc-arm-linux-gnu.x86_64

To install on Debian Buster, the following command is used:

	sudo apt-get install gcc-arm-linux-gnueabihf

## u-boot Build

Build u-boot using an ARM cross compiler, e.g. Fedora 31 gcc-arm-linux-gnu.x86_64:

To make .config file, the following command is required:

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 am335x-evm_defconfig

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 am335x-evm_defconfig

## Actual u-boot compilation:

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

## Actual u-boot compilation for the expert developers (part of porting effort) with no valid .dts tree present

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 EXT_DTB=<path/to/your/custom/built/dtb>

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 EXT_DTB=<path/to/your/custom/built/dtb>

## Place MLO and u-boot.img on SD card

For the BBB board case, two files are generated: MLO and u-boot.img.

In order to flash the SD card!

MLO should reside at offset 1024KB (1MB) of the SD card. To place it there (assuming ${DISK} is /dev/sdX):

	Flash the MLO image into the SD card:
	sudo dd if=./u-boot/MLO of=${DISK} bs=128k count=1 seek=1; sync

	Flash the u-boot.img image into the SD card:
	sudo dd if=./u-boot/u-boot.img of=${DISK} bs=384k count=2 seek=1; sync

Note - the sizes of MLO and u-boot.img on the SD card may vary, so, please, adjust this as needed.

The SD card device is typically something as /dev/sd<X> or /dev/mmcblk<X>. Note that there is a need for write permissions on the SD card for the command to succeed, so there is a need to su - as root, or use sudo, or do a chmod a+w as root on the SD card device node to grant permissions to users.

## Partitioning SD card

Here, two parttion are create: /dev/sdX1 for kernel, and /dev/sdX2 for rootfs.

Given example where /dev/sdX is /dev/sdb :

    echo "Create primary partition 1 for kernel"
    echo -e "n\np\n1\n\n+256M\nw\n"  | fdisk /dev/sdb

    echo "Create primary partition 2 for rootfs"
    echo -e "n\np\n2\n\n+8192M\nw\n"  | fdisk /dev/sdb

    echo "Formatting primary partition sdb1 for kernel"
    mkfs.vfat -F 32 /dev/sdb1

    echo "Formatting primary partition sdb2 for rootfs"
    mkfs.ext4 -F /dev/sdb2

## Making kernel using kernel.org vanilla (the latest stable upon writing this document) kernel 5.5.1

The kernel 5.5.1 source code is located @ the following location:

https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/

The file to be downloaded is the following: linux-5.5.1.tar.xz

The command to unpack the designated kernel is:

	tar -xvf inux-5.5.1.tar.xz
	cd linux-5.5.1/

To build the kernel, the following should be done:

Make proper .config:

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 omap2plus_defconfig

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 omap2plus_defconfig

Compile the kernel itself:

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

The kernel itself to be used is in the directory: .../linux-5.5.1/arch/arm/boot/ and it is called zImage:

	.../linux-5.5.1/arch/arm/boot/zImage

The .dtb file to be used is in the directory: .../linux-5.5.1/arch/arm/boot/dts/ and it is called am335x-boneblack.dtb

	.../linux-5.5.1/arch/arm/boot/dts/am335x-boneblack.dtb

The location on SD card both components should be placed is /dev/sdX1 mounted to some directory (example: /tmp/sdX1 (where the SD card itself is: /dev/sdX).

Assuming X=b, it looks like:

	mount /dev/sdb1 /tmp/sdb1

The following will happed after booting u-boot, and after booting kernel 5.5.1 from the SD card (initial boot @ [  0.000000]):

	Starting kernel ...

	[    0.000000] Booting Linux on physical CPU 0x0
	[ snap ]

## Making rootfs (using latest up to date BuildRoot distribution):

https://bootlin.com/doc/training/buildroot/buildroot-labs.pdf

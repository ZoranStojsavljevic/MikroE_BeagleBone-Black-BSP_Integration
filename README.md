## MASTER README.md - Overlay Examples

The BBB board Revision used is REV. A (0x0A5C).

	root@arm:/home/debian# /opt/scripts/tools/version.sh | grep eeprom
	eeprom:[A335BNLT0A5C2813BBBK4802]

The reason for that is the following:

https://github.com/ZoranStojsavljevic/MikroE_BeagleBone-Black_BSP-Integration/blob/master/BBB-debian_buster/overlay_examples/README.md

### People are highly encouraged to use overlays in BBB-debian_buster/overlay_examples/

Please, do note that the structure of this GitHub Repo has been changed significantly.

## Notes from the author
IMPORTANT: Please, read carefully notes from the author!

https://github.com/ZoranStojsavljevic/MikroE_BeagleBone-Black-BSP_Integration/blob/master/Notes_from_the_author.md

##  MikroE BeagleBone Black BSP Integration
This repository explains BeagleBone Black referent development platform (based upon Texas Instruments (TI)
armv7 A8 am335x) related to the MikroE CLICK mikroBUS HW and SW support design and integration.

https://www.mikroe.com/

### am335x BeagleBone Black evaluation board setup

BeagleBone Black System Reference Manual

https://cdn-shop.adafruit.com/datasheets/BBB_SRM.pdf

element14 BeagleBone Black INDUSTRIAL System Reference Manual

http://download.kamami.pl/p562276-BBBI_SRM_Rev%201.0%20VL.pdf

## Building BBB BSP (YOCTO and Buildroot)

In this repo there are various methods shown (as quick jump start to BBB Linux) how to build BBB BSP. Author
prefers to build YOCTO (core-image-minimal) as a full system with Kernel Development headers included, so the
author can use BBB as target development system for the native drivers and other testing purposes.

Author in most cases builds U-Boot as latest U-Boot provided by U-Boot community, and uses kernel.org kernel
tarballs as kernel build-up. YOCTO provides to author extensive root tree, but some users might preffer all
BSP elements (U-Boot and kernel) from the YOCTO deploy dir, NOT only root tree.

For the final development YOCTO tree is replaced by author with much more compact root tree from Buildroot
package (as much smaller and optimally scalable root tree footprint).

### Get U-Boot

The latest U-Boot version might not work out of the box.

	git clone http://git.denx.de/u-boot.git u-boot-bbb OR
	git clone git://git.denx.de/u-boot.git u-boot-bbb
	cd u-boot-bbb

The cloned version will match HEAD set to master branch (the following command is optional):

	git checkout -b <branch-name> # Take the current HEAD and create a new branch called <branch-name>

### Installing arm cross compiler on the host (the hosts used are Debian Buster and Fedora 31 distros)

To install on Debian Buster, the following command is used:

	sudo apt-get install gcc-arm-linux-gnueabihf

To install on Fedora 31 gcc-arm-linux-gnu.x86_64 (which is not native x86_64 compiler), the following command is used:

	sudo dnf install gcc-arm-linux-gnu.x86_64

### U-Boot Build

Build U-Boot using an ARM cross compiler, Debian Buster and Fedora 31:

To make .config file, the following command is required:

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 am335x_evm_defconfig

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 am335x_evm_defconfig

### Actual U-Boot compilation

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8

### Placing MLO and u-boot.img on SD card

For the BBB board case, two files are generated: MLO and u-boot.img .

In order to flash the SD card!

MLO should reside at offset 1024KB (1MB) of the SD card. To place it there (assuming ${DISK} is /dev/sdX):

	Flash the MLO image into the SD card:
	sudo dd if=./u-boot/MLO of=${DISK} bs=128k count=1 seek=1; sync

	Flash the u-boot.img image into the SD card:
	sudo dd if=./u-boot/u-boot.img of=${DISK} bs=384k count=2 seek=1; sync

Note - the sizes of MLO and u-boot.img on the SD card may vary, so, please, adjust this as needed.

The SD card device is typically something as /dev/sd<X> or /dev/mmcblk<X>. Note that there is a need for
write permissions on the SD card for the command to succeed, so there is a need to su - as root, or use
sudo, or do a chmod a+w as root on the SD card device node to grant permissions to users.

### Partitioning SD card

Here, two partions are created: /dev/sdX1 for kernel, and /dev/sdX2 for rootfs.

Given example where /dev/sdX is /dev/sdb :

	echo "Create primary partition 1 for kernel"
	echo -e "n\np\n1\n\n+256M\nw\n" | fdisk /dev/sdb

	echo "Create primary partition 2 for rootfs"
	echo -e "n\np\n2\n\n+8192M\nw\n" | fdisk /dev/sdb

	echo "Formatting primary partition sdb1 for kernel"
	mkfs.vfat -F 32 /dev/sdb1

	echo "Formatting primary partition sdb2 for rootfs"
	mkfs.ext4 -F /dev/sdb2

### Making kernel using kernel.org vanilla (the latest stable upon writing this document) kernel 5.5.5

The kernel 5.5.5 source code is located @ the following location:

https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/

The file to be downloaded is the following: linux-5.5.5.tar.xz

The command to unpack the designated kernel is:

	tar -xvf linux-5.5.5.tar.xz
	cd linux-5.5.5/

To build the kernel, the following should be done:

Make proper .config:

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 omap2plus_defconfig

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 omap2plus_defconfig

In order to make changes to the .config file, the following must be done!

Change kernel .config (original defconfig used to build .config is omap2plus_defconfig):

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 menuconfig

For systemd service to work, the following change must be done in .config

	  │ Symbol: CGROUPS [=y]
	  │ Type  : bool
	  │ Prompt: Control Group support
	  │   Location:
	  │ (1) -> General setup
	  │   Defined at init/Kconfig:805
	  │   Selects: KERNFS [=y]
	  │   Selected by [n]:
	  │   - SCHED_AUTOGROUP [=n]

They will appear as the following CONFIG options in the .config :

	CONFIG_CGROUPS=y
	CONFIG_KERNFS=y

For additional systemd service setup, please, read README file in the systemd sources
folder - it describes all options required to be enabled in kernel configuration.

Compile the kernel itself:

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8

The kernel itself to be used is in the directory: .../linux-5.5.5/arch/arm/boot/ and it is called zImage:

	.../linux-5.5.5/arch/arm/boot/zImage

The .dtb file to be used is in the directory: .../linux-5.5.5/arch/arm/boot/dts/ and it is called am335x-boneblack.dtb

	.../linux-5.5.5/arch/arm/boot/dts/am335x-boneblack.dtb

The location on SD card both components should be placed is /dev/sdX1 mounted to some directory (example: /tmp/sdX1
(where the SD card itself is: /dev/sdX).

Assuming X=b, it looks like:

	mount /dev/sdb1 /tmp/sdb1

## am335x Beaglebone-black Buildroot (making embedded Linux root tree)

### Making rootfs (using latest up to date Buildroot distribution):

https://bootlin.com/doc/training/buildroot/buildroot-labs.pdf

### Buildroot Build

Build Buildrool using an ARM cross compiler, Debian Buster and Fedora 31:

To make .config file, the following command is required:

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 beaglebone_defconfig

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 beaglebone_defconfig

### Buildroot customization

To customize .config file, the following command is required:

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 menuconfig

### Actual Buildroot compilation

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

	Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8

### The Buildroot output directory (location, the results are stored)

	.../buildroot/output/images
	[.../buildroot/output/images]$ ls -al
	total 158972
	drwxr-xr-x. 2 vuser vboxusers     4096 Feb 11 10:57 .
	drwxr-xr-x. 6 vuser vboxusers     4096 Feb 11 10:57 ..
	-rw-r--r--. 1 vuser vboxusers    58300 Feb 11 10:57 am335x-boneblack.dtb
	-rw-r--r--. 1 vuser vboxusers    57898 Feb 11 10:57 am335x-boneblue.dtb
	-rw-r--r--. 1 vuser vboxusers    56484 Feb 11 10:57 am335x-bone.dtb
	-rw-r--r--. 1 vuser vboxusers    56740 Feb 11 10:57 am335x-bonegreen.dtb
	-rw-r--r--. 1 vuser vboxusers    63027 Feb 11 10:57 am335x-evm.dtb
	-rw-r--r--. 1 vuser vboxusers    61679 Feb 11 10:57 am335x-evmsk.dtb
	-rw-r--r--. 1 vuser vboxusers 16777216 Feb 11 10:57 boot.vfat
	-rw-r--r--. 1 vuser vboxusers   107356 Feb 11 10:39 MLO
	-rw-r--r--. 1 vuser vboxusers 62914560 Feb 11 10:57 rootfs.ext2
	lrwxrwxrwx. 1 vuser vboxusers       11 Feb 11 10:57 rootfs.ext4 -> rootfs.ext2
	-rw-r--r--. 1 vuser vboxusers 46551040 Feb 11 10:57 rootfs.tar
	-rw-r--r--. 1 vuser vboxusers 79692288 Feb 11 10:57 sdcard.img
	-rw-r--r--. 1 vuser vboxusers   761880 Feb 11 10:39 u-boot.img
	-rw-r--r--. 1 vuser vboxusers      434 Feb 11 10:57 uEnv.txt
	-rw-r--r--. 1 vuser vboxusers  5712936 Feb 11 10:57 zImage
	[.../buildroot/output/images]$

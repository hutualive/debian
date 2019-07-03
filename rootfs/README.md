# bootstrap debian 10 root filesystem for armhf(armv7) on ubuntu 18.04
install the required pacakges -->

sudo apt install qemu-user-static binfmt-support debootstrap

mkdir rootfs

sudo debootstrap --foreign --no-check-gpg --include=ca-certificates --arch=armhf buster rootfs http://mirrors.huaweicloud.com/debian

sudo cp -av /usr/bin/qemu-arm-static ./rootfs/usr/bin

sudo cp -av /run/systemd/resolve/stub-resolv.conf ./rootfs/etc/resolv.conf

// copy the kernel modules to target

sudo cp -r ./lib/modules ./rootfs/lib

//copy the kernel firmware to target

sudo cp -r ./lib/firmware ./rootfs/lib

sudo chroot rootfs

--> entering into second stage

export LANG=C

export LANGUAGE=C

export LC_ALL=C

/debootstrap/debootstrap --second-stage --verbose

//add apt source list, for example deb http://mirrors.huaweicloud.com/debian buster main contrib non-free

nano /etc/apt/sources.list

apt update && apt upgrade -y && apt install vim sudo wpasupplicant -y

// change root password

passwd root

echo mp1 > /etc/hostname

echo 127.0.0.1	localhost > /etc/hosts

// add boot filesystem, for example /dev/mmcblk0p5	/	ext4	defaults,errors=remount-ro	0	1

nano /etc/fstab

// add network interface, for example auto eth0 iface eth0 inet dhcp

nano /etc/network/interfaces

// add /dev/ttySTM0 and /dev/ttyACM0 to /etc/securetty, otherwise you can't login with root

nano /etc/securetty 

apt autoclean

exit

--> finish second stage

sudo rm ./rootfs/etc/resolv.conf

sudo rm ./rootfs/usr/bin/qemu-arm-static

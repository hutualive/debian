# background
st introduced stm32mp1 series recently, a cortex-a7 dual core plus one cortex-m4 core processor. the official distribution is built by yocto, which is a complex auto-build tool, not easy to learn and customize for your own needs. so I'm thinking to bootstrap the coming debian 10(buster) to it.

they have introduced two discovery kits: dk1 and dk2. the key differences are dk1 without crypto related features and bt/wlan module on board, which is easier and cheaper to get access.

# quick start guide
in the end of process, you will have a complete sd card image, dd to the sd card, you will boot into your favorite debian 10 as root.

sudo dd if=./sd-dk1.img of=/dev/sdb bs=8M conv=fdatasync

//caution: check carefully your device file name, here for my case is /dev/sdb for my usb sd card reader.

//ps: now I still do not figure out a cloud storage service to upload my 1GB sd-dk1.img/sd-dk2.img file for you to download and try immediately.

# introduction
unlike your favorite raspberry pi board, st choose to use GPT(GUID partition table)as partition/boot mechanisim. the rom bootloader inside the SoC will look for the partition name fsbl1 for first stage bootloader. if failed, will look for fsbl2 as backup. st choose to use arm tf-a for fsbl1 and fsbl2. first stage bootloader will load second stage loader(ssbl), here is u-boot which is popular in embedded world. then second stage bootloader will load linux kernel and device tree blob. with these SoC/board specifi infrastructures put in place, you can bootstrap debian 10 as your root filesystem.

so below is the process step by step.

# populate a raw sd card image
create a 1GB raw sd card image file -->
sudo dd if=/dev/zero of=./sd-dk1.img bs=1M count=1024

use gdisk to create the partitions: fsbl1, fsbl2, ssbl, bootfs, rootfs -->
sudo gdisk ./sd-dk1.img

GPT reserve the first 17KB on sd card for protective MBR and 128 entries of GPT table. so you can only start with 17KB on-wards and you need adjust the sector alignment to 1 as gdisk default is 2048.

you need mark the bootfs partition as legacy bios bootable for u-boot to look for kernel and device tree.

below is the summary of partitions and filesystem type:
0-17K: protective MBR and GPT table
17K+256K: fsbl1 - linux reserved 8301
17K+256K+256K: fsbl2 - linux reserved 8301
17K+256K+256K+2M: ssbl - linux reserved 8301
17K+256K+256K+2M+64M: bootfs - linux filesystem 8300 with legacy bios bootable attribute
17K+256K+256K+2M+64M+rest of 1GB: rootfs - linux filesystem 8300

# mount sd card image as disk to populate
sudo losetup -Pf sd-dk1.img

//ps: here in my case the image file mounted as /dev/loop18 with 5 partitions  

# populate fsbl
inside the fsbl1 and fsbl2 directory, I uploaded st official tf-a firmware for dk1 and dk2 for your convenience:
sudo dd if=./fsbl1/tf-a-dk1.stm32 of=/dev/loop18p1 bs=1M conv=fdatasync  
sudo dd if=./fsbl2/tf-a-dk1.stm32 of=/dev/loop18p2 bs=1M conv=fdatasync

# populate ssbl
inside the ssbl directory, I uploaded u-boot firmware for dk1 and dk2 for your convenience: 
sudo dd if=./ssbl/u-boot-dk1.stm32 of=/dev/loop18p3 bs=1M conv=fdatasync

//ps: st use u-boot v2018.11 with their own patch, they enabled watchdog by default, so to use debian without periodically reset, I disabled the watchdog and re-compiled the firmware.

# populate bootfs
inside the bootfs directory, I uploaded the kernel, device tree blog and config file for dk1 and dk2 for your convenience:
sudo mkfs.ext4 -L bootfs /dev/loop18p4
rsync -avx ./bootfs/dk1 /dev/loop18p4

# populate rootfs
rootfs is big, you need follow the readme inside ./rootfs to bootstrap the debian 10 root filesystem before proceed below step:
sudo mkfs.ext4 -L rootfs /dev/loop18p5
rsync -avx ./rootfs/ /dev/loop18p5

# clean up and return to the quick start guide
sudo losetup -d /dev/loop18
sudo dd if=./sd-dk1.img of=/dev/sdb bs=8M conv=fdatasync

have fun :)

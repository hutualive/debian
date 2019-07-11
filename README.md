# background
st introduced stm32mp1 series recently, a cortex-a7 dual core plus one cortex-m4 core processor. the official distribution is built by yocto, which is a complex auto-build tool, not easy to learn and customize for your own needs. so I'm thinking to bootstrap the coming debian 10(buster) to it.

they have introduced two discovery kits: dk1 and dk2. the key differences are dk1 without crypto related features and bt/wlan module on board, which is easier and cheaper to get access.

# quick start guide
in the end of process, you will have a complete sd card image, dd to the sd card, you will boot into your favorite debian 10 as root.

sudo dd if=./sd-dk1.img of=/dev/sdb bs=8M conv=fdatasync

//caution: check carefully your device file name, here for my case is /dev/sdb for my usb sd card reader.

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

| size | name | type |
| :----: | :----: | :----: |
| 0-17K: | protective MBR and GPT table |
| 17K+256K: | fsbl1 | linux reserved 8301 |
| 17K+256K+256K: | fsbl2 | linux reserved 8301 |
| 17K+256K+256K+2M: | ssbl | linux reserved 8301 |
| 17K+256K+256K+2M+64M: | bootfs | linux filesystem 8300 with legacy bios bootable attribute |
| 17K+256K+256K+2M+64M+rest of 1GB: | rootfs | linux filesystem 8300 |

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

//ps: I'm using u-boot v2018.11 with st patch, st enabled watchdog by default, so to use debian without periodically reset, I disabled the watchdog and re-compiled the firmware. you can find the patch under ./ssbl/patch as reference and readme for how to compile.

# prepare raw bootfs

sudo mkfs.ext4 -L bootfs /dev/loop18p4

# prepare raw rootfs

sudo mkfs.ext4 -L rootfs /dev/loop18p5

# clean up and flash the sd card image

sudo losetup -d /dev/loop18

sudo dd if=./sd-dk1.img of=/dev/sdb bs=8M conv=fdatasync

# populate bootfs

insert the sd card into pc, you will have bootfs and rootfs as normal usb drive.

inside the bootfs directory, I uploaded the kernel, device tree blog and config file for dk1 and dk2 for your convenience:

rsync -avx ./bootfs/dk1 /media/dp/bootfs

//ps: I'm using the latest lts kernel version v4.19.56 from kernel.org with st patch and fragment. you can find the patch and frament under ./bootfs/patch as reference and readme for how to compile.

# populate rootfs

rootfs is big, you need follow the readme inside [./rootfs/README.md](./rootfs/README.md) to bootstrap the debian 10 root filesystem before proceed below step:

rsync -avx ./rootfs/rootfs/ /media/dp/rootfs

# wrap up
write back the whole system to raw sd card image for future replication

sudo dd if=/dev/sdb of=./sd-dk1.img bs=8M conv=fdatasync count=128  --> just dd the 1st 1GB, otherwise it's a long process for 16GB sd card make no sense

next time you just need refer to quick start guide to bring up debian 10 with one command.

have fun :)

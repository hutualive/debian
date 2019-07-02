# background
st introduced stm32mp1 series, a cortex-a7 dual core plus one cortex-m4 core processor. the official distribution is built by yocto, which is a complex auto-build tool, not easy to learn and customize. so I'm thinking to debootstrap the coming debian 10(buster)to it.

st introduced two discovery kits: dk1 and dk2. the difference are dk1 without crypto related features and bt/wlan module on board, which is easier and cheaper to get access.

# quick start guide
in the end of process, you will have a complete sd card image, dd to the sd card, you will boot into your favorite debian os with root.

sudo dd if=/sd-dk1.img of=/dev/sdb bs=8M conv=fdatasync

caution: check carefully your device file name, here for my case is /dev/sdb for my usb sd card reader.

ps: now I still do not figure out a cloud storage service to upload my 1GB sd-dk1.img/sd-dk2.img file for you to download and try immediately.

# introduction
unlike your favorite raspberry pi board, st choose to use GPT(GUID partition table)as partition/boot mechanisim. the rom bootloader inside the SoC will look for the partition name fsbl1 for first stage bootloader. if failed, will look for fsbl2 as backup. st choose to use arm tf-a for fsbl1 and fsbl2. first stage bootloader will load second stage loader(ssbl), here is u-boot which is popular in embedded world. then second stage bootloader will load linux kernel and device tree blob. with these SoC/board specifi infrastructure put in place, you can bootstrap debian as your root filesystem.

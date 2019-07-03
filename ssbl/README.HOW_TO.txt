Compilation of U-Boot:
1. Pre-requisite
2. Initialise cross-compilation via SDK
3. Prepare U-Boot source code
4. Compile U-Boot source code

1. Pre-requisite:
-----------------
OpenSTLinux SDK must be installed.

For U-Boot menuconfig you need to install:
* libncurses and libncursesw dev package
    - Ubuntu: sudo apt-get install libncurses5-dev libncursesw5-dev

2. Initialize cross-compilation via SDK:
---------------------------------------
* Source SDK environment:
    $> source <path to SDK>/environment-setup-cortexa7t2hf-neon-vfpv4-openstlinux_weston-linux-gnueabi

* To verify if you cross-compilation environment are put in place:
    $> set | grep CROSS
    CROSS_COMPILE=arm-openstlinux_weston-linux-gnueabi-

Warning: the environment are valid only on the shell session where you have
         sourced the sdk environment.

3. Prepare U-Boot source:
------------------------
extract the tarball and apply st patch:

    $> tar xfz v2018.11.tar.gz
    $> cd u-boot-2018.11
    $> for p in `ls -1 ../patch/*.patch`; do patch -p1 < $p; done

5. Compilation U-Boot source code:
----------------------------------
To compile U-Boot source code, first move to U-Boot source:
    $> cd u-boot-2018.11

5.1 Compilation for one target (one defconfig, one device tree)

    see <U-Boot source>/board/st/stm32mp1/README for details

    # make stm32mp15_<config>_defconfig
    # make DEVICE_TREE=<device tree> all

    example:

    a) trusted boot on dk1
	# make stm32mp15_trusted_defconfig
	# make DEVICE_TREE=stm32mp157a-dk1 all

    b) trusted boot on dk2
	# make stm32mp15_trusted_defconfig
	# make DEVICE_TREE=stm32mp157c-dk2 all

The generated binary files are available in ../build-${config}.


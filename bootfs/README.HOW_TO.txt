Compilation of kernel:
1. Pre-requisite
2. Initialise cross-compilation via SDK
3. Prepare kernel source code
4. Configure kernel source code
5. Compile kernel source code

1. Pre-requisite:
-----------------
OpenSTLinux SDK must be installed.

For kernel menuconfig, you need to install:
- libncurses and libncursesw dev package
    Ubuntu: sudo apt-get install libncurses5-dev libncursesw5-dev

- mkimage
    Ubuntu: sudo apt-get install u-boot-tools

2. Initialise cross-compilation via SDK:
----------------------------------------
Source SDK environment:
    $> source <path to SDK>/environment-setup-cortexa7t2hf-neon-vfpv4-openstlinux_weston-linux-gnueabi

To verify if your cross-compilation environment has been put in place correctly,
run the following command:
    $> set | grep CROSS
    CROSS_COMPILE=arm-openstlinux_weston-linux-gnueabi-

Warning: the environment is valid only on the shell session where you have
sourced the SDK environment.

3. Prepare kernel source:
-------------------------
extract the tarball.
    $> tar xfJ <kernel source>.tar.xz
    $> cd linux-4.19.56

apply the patch.
    $> for p in `ls -1 ../patch/*.patch`; do patch -p1 < $p; done

4. Configure kernel source code:
--------------------------------
* Configure on a build directory
    $> mkdir -p ../build
    $> make ARCH=arm O="$PWD/../build" multi_v7_defconfig fragment*.config

* apply the fragment by loop:
    $> for f in `ls -1 ../patch/fragment*.config`; do scripts/kconfig/merge_config.sh -m -r -O $PWD/../build $PWD/../build/.config $f; done
    $> yes '' | make ARCH=arm oldconfig O="$PWD/../build"

5. Compile kernel source code:
------------------------------
* Build kernel images (uImage and vmlinux) and device tree (dtbs)
    $> make ARCH=arm uImage vmlinux dtbs module LOADADDR=0xC2000040 O="$PWD/../build"

* Generate output build artifacts
    $> make ARCH=arm INSTALL_MOD_PATH="$PWD/../build/install_artifact" modules_install O="$PWD/../build"
    $> mkdir -p $PWD/../build/install_artifact/boot/
    $> cp $PWD/../build/arch/arm/boot/uImage $PWD/../build/install_artifact/boot/
    $> cp $PWD/../build/arch/arm/boot/dts/st*.dtb $PWD/../build/install_artifact/boot/


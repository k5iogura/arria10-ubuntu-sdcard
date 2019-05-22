# Ubuntu SD card building scripts for ARM CPU on Arria 10 SoC FPGAs

These scripts will build SD cards to boot Ubuntu on the hard ARM cores on
Intel Arria 10 FPGAs.  As well as the SD card image, they will build kernel, u-boot
and device tree pieces, as well as downloading a pre-built FPGA bitfile at
boot time.  Ubuntu and compiler images are downloaded as part of the build
process.

## Prerequisites

* A **Linux Intel machine** with the **Quartus** tools installed and on your PATH (tested with Ubuntu 16.04/18.04 and Quartus
  18.1).  In particular, the 'Intel SoC FPGA Embedded Development Suite Standard
  Edition'.  A licence for Quartus should not be necessary if you are not
  resynthesising the FPGA bitfiles.
* A Quartus project which has built a suitable bitfile (.sof) - this will be converted to the necessary .rbf as part of the build
* A Platform Designer (Qsys) design of the FPGA SoC that will be used to build the device tree
* An internet connection (to download necessary pieces)
* sudo access (since building SD card images requires the loopback device)

## Usage

./build_ubuntu_sdcard.sh \<path to FPGA project directory\> \<name of Quartus project\> \<name of Qsys project>

## Author

Theo Markettos

thunderclap+atm26 at cl.cam.ac.uk

# Tracing arria10 soc ubuntu github

## On RaspberryPi3 Model B+
Instead of Intel PC i use RaspberryPi3 with Rasbian strech as Host machine. Download repo. and work on it,  
```
  $ git clone https://github.com/k5iogura/arria10-ubuntu-sdcard

  $ cd arria10-ubuntu-sdcard
```

## Additional Prerequisite

```
  # apt install bison flex libncurses-dev bc  
```

- Flow  
```
ubuntu :
        $SCRIPT_PATH/fetch_ubuntu.sh
        $SCRIPT_PATH/configure_networking.sh mnt/2/
kernel :
        $SCRIPT_PATH/build_linux.sh
uboot :
        $SCRIPT_PATH/make_uboot.sh $FPGA_DIR/$FPGA_HANDOFF_DIR
        cp -a bsp/uboot_w_dtb-mkpimage.bin .
devicetree
bitfile
sdimage
tidy
```

#### Tracing fetch_ubuntu.sh
```
  $ wget http://cdimage.ubuntu.com/releases/16.04/release/ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img.xz

  # unxz ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img.xz
  # img=ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img

  # dev="$(losetup --show -f -P "$img")"

  # echo $dev
  /dev/loop0
```

#### Tracing configure_network.sh
```
  # vi /mn/2/etc/network/interfaces.d/60-ipv6dns.cfg

    iface eth0 inet6 auto
    dns-nameservers 2001:4860:4860::8888 2001:4860:4860::8844
```

#### Tracing build_linux.sh

  Setup CrossCompiler // so this repo for i386 intel PC  
```
  $ git clone https://github.com/altera-opensource/linux-socfpga
  $ cd linux-socfpga
  // git checkout socfpga-4.17 #use rel_socfpga-4.17_18.11.02_pr instead
  $ git checkout rel_socfpga-4.17_18.11.02_pr

  $ make socfpga_defconfig
  $ export ARCH=arm
  // make menuconfig if you need
  make zImage -j2
  cp -a arch/arm/boot/zImage ../
```

#### Tracing build_uboot.sh

#### Write sdimage
```
        sudo $SCRIPT_PATH/make_sdimage.py -f    \
                -P uboot_w_dtb-mkpimage.bin,num=3,format=raw,size=10M,type=A2 \
                -P mnt/2/*,num=2,format=ext3,size=1500M \
                -P zImage,socfpga.rbf,socfpga_arria10_socdk_sdmmc.dtb,num=1,format=vfat,size=500M \
                -s 2048M \
                -n sdimage.img
make_sdimage.py:
  mounted_fs = []
  losetup sdimage.img file
  fdisk to make partitions on loopback device
  losetup -d
  for all -P options
    do_partition() :
      losetup setup sdimage.img file
      mkfs.*

      copy_files_to_partition() :
        do_copy() :
          mp = '/tmp/' + time() + '_' + os.getpid()
          mkdir -p mp
          mount /dev/loop*p* mp
          mounted_fs.append(mp)
          copy all files into mount point with -at or -rt option
          umount mp

      losetup -d
      umount /mnt/*

/dev/loop0p1 :partition No.1 : vfat  500MB : zImage, rbf, dtb
/dev/loop0p2 :partition No.2 : ext3 1500MB : rootfs
/dev/loop0p3 :partition No.3 : A2     10MB : preloader image
```
**K.O 22.May,2019**  

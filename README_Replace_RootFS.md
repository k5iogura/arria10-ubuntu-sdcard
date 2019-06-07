# GSRD : How to replace root filesystem from angstrom to ubuntu server

## Prerequisites
- SDCard over 16GB  

## Download and extract partitions from GSRD and Ubuntu

GSRD  
```
  # wget http://releases.rocketboards.org/release/2018.10/gsrd/a10_gsrd/sdimage.tar.gz
  # tar xzf sdimage.tar.gz
  # losetup --show -f -P sdimage.img
    /dev/loop0
  # ls /dev/loop0*
    /dev/loop0  /dev/loop0p1  /dev/loop0p2  /dev/loop0p3
```
By df -T command,  
loop0p1 : vfat(FAT32)  
loop0p2 : Linux(ext2)  
loop0p3 : Unknown(A2)  

Copy or dd into work area,  
```
  # mkdir /mnt/1 /mnt/2
  # mkdir -p work/1 work/2 work/3
  # mount /dev/loop0p1 /mnt/1
  # cp -ar /mnt/1/* -t work/1
  # dd if=/dev/loop0p3 of=work/3/p3.bin bs=1
    20481+0 records in
    20481+0 records out
    10486272 bytes (10 MB, 10 MiB) copied, 0.0508722 s, 206 MB/s
  # umount /mnt/1 /mnt/2
  # losetup -d /dev/loop0
```

Ubuntu server  
```
  $ wget http://cdimage.ubuntu.com/releases/16.04/release/ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img.xz
  # unxz ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img.xz
  # losetup --show -f -P ubuntu-16.04.4-preinstalled-server-armhf+raspi2.img
    /dev/loop0
  # ls /dev/loop0`*`
  /dev/loop0  /dev/loop0p1  /dev/loop0p2
  # mount /dev/loop0p2 /mnt/2
  # cp -ar /mnt/2/* -t work/2
  # umount /mnt/2
  # losetup -d /dev/loop0

```

Data for New SDCard are ready in work directory.  

## GSRD SDCard Structure

- 1st partition for FAT32 
  zImage, \*.sdf, \*.dtb  

- 2nd partition for Linux root filesystem  
  /  

- 3rd partition for type A2  
  / u-boot 2nd boot-loader

## Format and make FileSystem on New SDCard
Insert SDCard to USB writer.  

- Use fdisk to make partitions  
```
  partition 1 as FAT32      500MB
  partition 2 as ext3      3000MB
  partition 3 as type a2     10MB
  partition 4 as ext3      remain about 11GB
```
- Use mkfs for partion 1, 2 and 4  
Example below, SDCard is recognized as /dev/sdb.  
```
  # mkfs -t vfat /dev/sdb1
  # mkfs -t ext3 /dev/sdb2
  # mkfs -t ext3 /dev/sdb4
```

## Copy and dd partitions
```
  # mount /dev/sdb1 /mnt/1
  # mount /dev/sdb2 /mnt/2
```
- cp zImage etc. in GSRD partition 1 to New SDCard partition 1  
```
  # cp work/1/* /mnt/1
```
- cp / in Ubuntu Server to New SDCard partition 2  
```
  # cp work/2/* /mnt/2
```

- dd from GSRD to New SDCard partition 3  
```
  # dd of=work/3/p3.bin of=/dev/sdb3 bs=1
```
```
  # umount /mnt/1 /mnt/2
```

## Setup Enviornment
  Pray and boot with New SDCard.  

- For DHCP Client
```
  # vi /etc/profile
  remove if setting line
```

- autologin with root at tty1  
```
  # systemctl edit getty@tty1.service  
  [Service]
  ExecStart=
  ExecStart=-/sbin/agetty --autologin root --noclear %I 38400 linux

```

- networkd  
To avoid 'A start job is running for wait for network to be configured'  
```
systemctl disable systemd-networkd-wait-online.service
systemctl mask systemd-networkd-wait-online.service
```



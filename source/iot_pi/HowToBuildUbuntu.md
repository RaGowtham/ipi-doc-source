title: How to Build Ubuntu
---

The procedure to describes how to create Ubuntu Root File System(RFS) image on **LEC-PX30 with Industrial Pi-SMARC**



## **Prerequisites**

1. Install QEMU on the development host PC with Ubuntu OS

   ```
   $ sudo apt-get install qemu-user-static
   ```

2. Download Ubuntu 18.04 base from http://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/   and choose this one: `ubuntu-base-18.04.1-base-arm64.tar.gz`

3. Create a temporary folder for decompression after download.

   ```
   $ mkdir temp
   ```

4. Extract the file to temp folder

   ```
   $ sudo tar -xpf ubuntu-base-18.04.1-base-arm64.tar.gz -C temp
   ```

   

## Configure the rootfs

1. Get your network ready:

   ```
   $ sudo cp -b /etc/resolv.conf temp/etc/resolv.conf
   ```

2. Prepare QEMU:

   ```
   $ sudo cp /usr/bin/qemu-aarch64-static temp/usr/bin/
   ```

3. Change root:

   ```
   $ sudo chroot temp
   ```

4. Update and upgrade:

   ```
   $ apt update
   $ apt upgrade
   ```

5. Install the required tools or utilities:

   ```
   $ apt install vim git sudo net-tools ifupdown kmod iputils-ping man wget bash-completion alsa-utils apt-utils usbutils locales i2c-tools netplan.io vnc4server lm-sensors usbmount gcc g++ cmake  can-utils sox v4l-utils xubuntu-desktop can-utils 
   ```

6. Add user name:

   ```
   $ useradd -s '/bin/bash' -m -G adm,sudo adlink
   ```

8. Set user’s password:

   ```
   $ passwd adlink
   ```

9. Set the root user’s password:

   ```
   $ passwd root
   ```

10. Add host name to `/etc/hostname`

       ```
    $ echo 'adlink' > /etc/hostname
       ```
    
11. Add host entry in `/etc/hosts`

       ```
    127.0.0.1 localhost
    127.0.0.1 adlink
       ```

12. After all the work is done, exit the chroot environment.

    ```
    $ exit
    ```



## Add I/O Interfaces to rootfs 



###    Ethernet & CAN Bus

1. Create a directory to copy the Ethernet and CAN kernel modules

    ```
    $ sudo mkdir temp/home/adlink/rockchip_test
    ```
    
2. Please click [here](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/Images/Ubuntu/UbuntuNecessaryFiles/Ubuntu-kernel-module_2020-02-24.zip) to download and copy  **smsc9500.ko,  smscusbnet.ko and mcp25xxfd.ko kernel modules** into rockchip_test folder from host PC. 

    ```
    $ sudo cp /your path/*.ko temp/home/adlink/rockchip_test/
    ```

3. Suggest to use Netplan to enables easily configuring networking on system. Please create a `01-network-manager-all.yaml` file under /etc/netplan

    ```
    $ sudo vim temp/etc/netplan/01-network-manager-all.yaml
    ```

    and add the following content, then save it.
    
    ```
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp4: true
          optional: true
        enx00800f117000:
          dhcp4: true
          optional: true
    ```

    **Note:** [Netplan](https://wiki.ubuntu.com/Netplan) enables easily configuring networking on a system via YAML files. Netplan processes the YAML and generates the required configurations for either NetworkManager or systemd-network the system’s renderer. Netplan [replaced ifupdown](http://blog.cyphermox.net/2017/06/netplan-by-default-in-1710.html) as the default configuration utility starting with Ubuntu 17.10 Artful. 



###    Audio

1. Please click [here](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/Images/Ubuntu/UbuntuNecessaryFiles/asound.state) to download **asound.state** file and copy to `/var/lib/alsa/`

   ```
   $ sudo cp asound.state temp/var/lib/alsa/
   ```



### Enable I/O Interfaces 

1. Please click [here](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/Images/Ubuntu/UbuntuNecessaryFiles/Load.sh) to download and copy **Load.sh** file to the `rockchip_test`folder. 

   **Note**: it is shell script to insert modules on every reboot 

   ```
   $ sudo cp temp/home/adlink/rockchip_test/Load.sh
   ```

2. Give execute permissions to script
   ```
   $ sudo chmod +x temp/home/adlink/rockchip_test/Load.sh
   ```

3. Please click [here](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/Images/Ubuntu/UbuntuNecessaryFiles/rc.local) to download and copy **rc.local** to `temp/etc/` 

   ```
   $ sudo cp rc.local temp/etc/
   ```

4. Give executable permissions.
   ```
   $ sudo chmod +x temp/etc/rc.local
   ```



### Add MRAA 

1. Please click [here](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/Images/Ubuntu/UbuntuNecessaryFiles/adlink-mraa-master.tar) to extract and copy the binaries, applications and libraries to respective folders:

   ```
   $ mkdir mraa
   $ tar -xvf adlink-mraa-master.tar -C mraa/
   $ sudo cp -r mraa/usr/bin/* ../temp/usr/bin/
   $ sudo cp -r mraa/usr/include/* ../temp/usr/include/
   $ sudo cp -r mraa/usr/lib/libmraa.so* ../temp/usr/lib/
   $ sudo cp -r mraa/usr/lib/pkgconfig/mraa.pc ../temp/usr/lib/pkgconfig/
   $ sudo cp -r mraa/usr/share/* ../temp/usr/share/
   ```

   



## Make the Root File System

Execute the commands below to make the rootfs.img. Notice that you need change the “count” value according to the size of the “temp” folder.

Below command will create new rootfs.img file of size 7.4 GB (6.9 GB)

    $ dd if=/dev/zero of=rootfs.img bs=1M count=7065 status=progress && sync
    $ sudo mkfs.ext4 rootfs.img
    $ mkdir rootfs
    $ sudo mount rootfs.img rootfs
    $ sudo cp -rfp temp/* rootfs/
    $ sudo umount rootfs/
    $ e2fsck -p -f rootfs.img

 The final root files system rootfs.img is ready.



## Add the created rootfs to Buildroot 

The procedure to describes how to attach **Ubuntu rootfs image** to LEC-PX30 Buildroot Linux

Before start to compile Buildroot, please Install the required packages on the development host PC.

```
$ sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo liblz4-tool genext2fs lib32stdc++6
```



1. Download [LEC-PX30 Buildroot SDK](https://hq0epm0west0us0storage.blob.core.windows.net/development/LEC-PX30/SDK/v1.0.8-20200125/px30_buildroot_es2_sdk_20200125.tar.gz) and extract it.

   **Note**: use `iPIsmarc-es2` branch and root for the building. please type `git branch` command under **px30_buildroot/kernel** to check the branch version.

2. Change directory to px30_buildroot.

   ```
   $ cd px30_buildroot
   ```

3. Create a temporary directory.

   ```
   $ mkdir ubunturootfs
   ```

4. Copy the created rootfs.img to ubunturootfs.

   ```
   $ cp /your path/rootfs.img ubunturootfs
   ```

5. Create a **parameter-ubuntu.txt** file under device/rockchip/px30 folder.

   ```
   $ cp device/rockchip/px30/parameter-buildroot.txt device/rockchip/px30/parameter-ubuntu.txt
   ```

6. Change directory to ubunturootfs folder and to check UUID of rootfs image by running below command and you will see the UUID as the below

   ```
   $ file ubunturootfs/rootfs.img
   ```
   
   <img align="center" src="HowToBuildUbuntu.assets/ubuntu_uuid.png"/>

7. Modify UUID you have to`parameter-ubuntu.txt` file

   ```
   $ vi device/rockchip/px30/parameter-ubuntu.txt
   ```

    ![img](HowToBuildUbuntu.assets/ubuntu_uuid_m.png)
   
   then, modify the rootfs size and alter the user data starting address as per requirement. In below we are changing rootfs szie to 7GB. ![img](HowToBuildUbuntu.assets/ubuntu_uuid_s.png)
   
8. Open BoardConfig_open.mk file and change the path for `parameter-ubuntu.txt` and `rootfs.img` files.

   ```
   $ sudo vi device/rockchip/px30/BoardConfig_open.mk
   ```
   
    ![img](HowToBuildUbuntu.assets/ubuntu_parameter.png)

9. Open rk3326.dtsi file and add new part UUID or device partition. In below added /dev/mmcblk1p8 partition.

   ```
   $ sudo vi kernel/arch/arm64/boot/dts/rockchip/rk3326-linux.dtsi
   ```
   
   ![img](HowToBuildUbuntu.assets/ubuntu_u-boot.png)

10. Change directory to `px30_buildroot`

       ```
      $ cd px30_buildroot
       ```


11. Run build.sh, it will generate update.img under rockdev folder.

       ```
      $ ./build.sh
       ```

###  
# arch-linux-vm-setup

Here are the steps for Arch Linux installation on a VM. 

## Create the VM and Boot

1. Download the Arch Linux ISO image (https://www.techspot.com/downloads/5571-arch-linux.html)
2. Create a Virtual Machine on VirtualBox (Make sure the VM and VirtualBox are on the same drive (both under C:/ or D:/))
3. I have used Fixed size virtual HDD to avoid potential issues with Dynamically allocating the memory (I am not sure this is required)
4. Start the VM
5. Select "Boot Arch Linux", the first option on setup menu. Then the system should start booting.

## Partition the Hard Disk

1. We will create 3 partitions on the virtual hard disk: The first one will be the primary root partition with half the size of your total mem. size (if you total mem. is 20G, this should be 10G). The second one will be the swap partition, which will be twice the initial RAM allocation (if you allocate 1024M of RAM, this will be 2048). The third will be the logical partition with the remaining memory.
2. Use __cfdisk__ command to create partitions
3. Select __dos__ as the label type
4. Press enter on Free space to create your first partition. Make it __primary__ and __bootable__.
5. Repeat the step 4, 2 times to create the other partitions.
6. Select __write__ to flush the changes. Type yes to confirm.

## Format the Partitions

Run the following commands to format the partitions you created in the previous step.

```
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda3 
mkswap /dev/sda2
```

Activate the swap by running this command

```
swapon /dev/sda2
```

### Mount the Primary Partition

```
mount /dev/sda1 /mnt
mkdir /mnt/home
mount /dev/sda3 /mnt/home
```

### Bootstrap Arch Linux

Bootstrap the system by running this command

```
pacstrap /mnt base linux linux-firmware
```

It can take a few minutes to complete.

After the installation generate the fstab file by typing:

```
genfstab /mnt >> /mnt/etc/fstab 
```

Change the system root to the Arch Linux installation directory:

```
arch-chroot /mnt /bin/bash
```

### Configure Language & Time Settings

1. To configure language settings we need to install __nano__ or a text editor. We will use __pacman__ (package manager) to download stuff.

```
pacman -S nano
```

2. Select your desired language configuration by deleting the # in front of it and pressing control + x and typing y to quit nano.
3. Activate it by running

```
locale-gen
```

4. Create the locale.conf by typing:

```
nano /etc/locale.conf
```

Add this line to the file:

__LANG=en_US.UTF-8__

5. Synchronize the zone information: Following command will give you a list of available zones.

``` ls /usr/share/zoneinfo ```

6. Select your zone, I will be using CET

``` ln –s /usr/share/zoneinfo/CET /etc/localtime ```

7. Synchronize the hardware clock

``` hwclock --systohc --utc ```

8. Set the root user password

``` passwd ```

### Setup Hostname and Networking

``` nano /etc/hostname ```

1. Type any name as your hostname, save and quit.

2. Install dhcpcd to setup DHCP client

``` pacman -S dhcpcd ```

3. Enable DHCP client

``` systemctl enable dhcpcd ```

### Install the Bootloader

1. Initiate the grub installation

``` pacman -S grub os-prober ```

2. Install the grub boot loader to the hard disk by typing 

``` grub-install /dev/sda ```

3. Configure it:

``` grub-mkconfig –o /boot/grub/grub.cfg ```

4. Exit from chroot and reboot the system

```
exit
reboot
```

### Boot into Installed System

1. After rebooting select "Boot existing OS" to boot Arch Linux
2. Login with your root name & password. 

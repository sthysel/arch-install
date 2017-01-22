# Installing arch

Instructions to install Arch of a .iso image to a target machine (metal or virtual).

First get a iso that you can use to boot into a VM or burn to a USB and boot a real machine.
The arch wiki lists a current .iso, get it from there. Write the iso to a USB drive if the target is
a real machine, else skip this step and continue below.


## Write iso to USB flash drive:

Be sure to use ```dmesg``` or similar to determine target drive
```
$ sudo dd if=archlinux-2017.01.01-dual.iso of=/dev/sdb1 bs=4M
```

If installing into vmware vm on a UEFI secured boot machine, vmware will fail, disable secure boot.

Boot image or iso, it will drop you into a root terminal, you will now:
- make some partitions, typically / and swap.
- format the partitions
- mount the / partition somwhere convenient in the live bootimage
- install arch base into / of the internet.
-

## Make partitions

Make a swap and / partition, some peasants like to have a /home partition, you
can safely ignore their pedantry.  Use ```cfdisk``` a nice curses clownsuit for
fdisk, or just use fdisk. I use ```cfdisk```

- Select label as ```dos```
- Use the TUI and create swap and root partition using the buttons. I first make
  the swap which is 2x the RAM size, and then the / partition, which is the
  rest of the physical disk.
- use swap type for the swap partition, and ext4 for the / partition
- make / bootable
- `write` changes and quit 


verify the partitions are to your liking:



## Format partitions

```
mkswap /dev/sda1
mkfs.ext4 /dev/sda2
```

Switch on swap partition: `swapon /dev/sda1/`


## Mount and install Arch base
Now mount the / partition and install Arch base on the / partition.

```
mount /dev/sda2 /mnt/
pacstrap /mnt base base-devel
```

That will take a while.


Generate the fstab file from the mounts: 
```
genfstab /mnt >> /mnt/etc/fstab
```

Verify `/etc/fstab/` is good: ```cat /mnt/etc/fstab```.

Now chroot to the newly installed Arch system:
```
arch-chroot /mnt /bin/bash
```

Fucking immediately install `vim` how the fuck can't that be part of the base
install ?

```
pacman -S vim
```

Set and generate the locale
```
vim /etc/locale.gen # en.US_UTF-8 UTF-8
locale-gen
```

Register the locale in ```/etc/locale.conf``` : ```vim /etc/locale.conf``` And
add  ```LANG=en_US.UTF-8```

# Timezone
Find your timezone in ```/usr/share/timezone``` and link it to ```/etc/localtime```

```
rm /etc/localtime
ln -s /usr/share/zoneinfo/Australia/Perth /etc/localtime
```

Set system and harware clock to UTC.
```
hwclock --systohc --utc 
```

# Hostname
Set hostname in ```/etc/hosts``` as well as ```/etc/hostname```.

# Get IP from DHCP server

```
systemctl enable dhcpcd
```

# Grubb

Now you need to enable booting the bootable partition we prepared earlier.

```
pacman -S grub os-prober
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

Now exit the chroot, unmout the partitions and reboot.
```
exit
umount /mnt
reboot
```




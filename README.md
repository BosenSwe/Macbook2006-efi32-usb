# How to create a 64-bit linux install USB stick that is bootable on a Mac with 32bit EFI (pre 2008 MacBook)

A usb drive with only 1 partition to load grub2 on usb-bootable machines with `Legacy BIOS`, `64bit UEFI` or `32bit UEFI`.

## Partition the drive and install grub2

**Warning: the usb drive will be formatted, save your data before proceeding!**

First of all, on you current installation, check if the folder `/usr/lib/grub/` exists and is not empty.
If it is empty or does not exist, make sure the package grub-common (or equivalent for your distribution) version 2 or higher is installed.
Depending on the system, `/usr/lib/grub/` will contain one or more of the following folders: `x86_64-efi`, `x86_64-efi-signed`, `i386-pc`, `i386-efi`, ...

The `x86_64-efi`, `i386-pc` and `i386-efi` folders need to be present in order to install the corresponding bootloader on the usb drive.

###### Install them using the package manager, for instance on Ubuntu :

`sudo apt install grub-efi-ia32-bin`

Now, find the device file for your usb drive. Here, the file is `/dev/sdX`. Replace `X` with the appropriate lower case letter(s) in the commands.

###### Make sure it's the right drive! (check the capacity and the partitions) :

`lsblk`  # Shows your drives 

###### Erase the USB-stick
`sudo dd if=/dev/zero of=/dev/sdX`
You can interrupt with ctrl-c after around 10 seconds

###### Open fdisk :

`sudo fdisk /dev/sdX`

###### Press the following keys (THIS WILL ERASE ALL DATA FROM THE SELECTED DRIVE!) :

`o` `<enter>` # Create a new empty DOS partition table

`n` `<enter>` # Create a new partition

`p` `<enter>` # Select primary partition type

`<enter>` # Set partition number to 1 (default)

`<enter>` # Start partition at the first possible sector (default)

`+50M` # Set partition end to first possible sector + 50 Megabyte (make the partion 50 Mb)

`t` `<enter>` # Change partition type

`ef` `<enter>` # Set partition type to EFI (FAT-12/16/32)

`a` `<enter>` # Enable the bootable flag on partition 1

`n` `<enter>` # Create a new partition

`p` `<enter>` # Select primary partition type

`<enter>` # Set partition number to 2 (default)

`<enter>` # Start partition at the first possible sector (default)

`<enter>` # Set partition end to last possible sector (default)

`w` `<enter>` # Write the partition table

###### Create a fresh filesystem in the newly created partition 1 :

`sudo mkfs.fat -F32 /dev/sdX1`

###### Mount the filesystem :

`sudo mount -o umask=000 /dev/sdX1 /mnt`

###### Install `/EFI/BOOT/BOOTIA32.EFI` and other grub files required for 32-bit UEFI :

`sudo grub-install --removable --boot-directory=/mnt/boot --efi-directory=/mnt --target=i386-efi /dev/sdX`

###### Create a grub.cfg file :

`touch /mnt/boot/grub/grub.cfg`


###### Download an Peppermint image and extract it to partition2 of the USB-stick:

*Note: make sure there is enough space on the usb drive.*

`sudo dd /path/to/Peppermint-10-20191210-amd64.iso /dev/sdX2 bs=1M`

###### Edit grub.cfg :

`nano /mnt/boot/grub/grub.cfg`

###### Write or paste something like this :

````
set root=(hd0,2)
configfile /boot/grub/grub.cfg
````


###### Save grub.cfg (in nano) :

`CTRL+O`

`<enter>`

`CTRL+X`

## Finish

###### Unmount the filesystem :

`sudo umount /mnt`

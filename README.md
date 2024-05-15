# Arch Linux Installation Guide.
## Overview
This guide will walk you through the steps needed to install a fully functional Arch Linux Desktop. 
The focus is on KDE, but steps for XFCE will also be provided. 
This will be a UEFI/systemd-boot install, not a BIOS/Grub install. 
The BTRFS filesystem is the focus, but EXT4 is included. Zram is used, but swap is also shown. 
I recommend using the official Arch Linux install guide for support.

https://wiki.archlinux.org/index.php/Installation_guide

## Download and create USB
### Download ISO image
The official Arch Linux download page. https://archlinux.org/download/

A direct link to the latest image. https://geo.mirror.pkgbuild.com/iso/latest/archlinux-x86_64.iso
### Create a Bootable USB
WARNING! All data on the USB device will be permanently lost. \
WARNING! Be sure to check the device size and mount points to make sure it is your USB. All other storage devices should be removed just to be safe. \
To find the name of your USB device, use the command `lsblk`.
```
lsblk
```
Be sure to know the name of your USB. It will be `/dev/sdX` where sdX is the name of your USB device. 
Be careful it is not your hard drive or SSD. 
The partition `part /` is the device with your system partition. That is NOT the device you want to write to.

We will use the `dd` command to write the image to the USB. Change the directory to wherever you downloaded the file.
```
cd ~/Downloads
dd bs=4M if=archlinux-x86_64.iso of=/dev/sdX conv=fsync oflag=direct status=progress
```
<br/><br/>
# Installation
You will need to boot your system from the USB drive. It may be as simply as rebooting with the USB left plugged in.
Or more likely you need to know how to tell your bios to boot from the USB device.
It may be something like pressing `Escape`, `F5`, `F8`, `F10` or `F11` during boot up. Search your device and boot from USB.
<br/><br/>
## Setup keyboard
If you are using a US keyboard you can skip this step.
### List available keyboard keymaps
```
localectl list-keymaps
```
Use the arrow keys or `j`, `k` to move up or down the list. To quit use the `q` key.
### Set keyboard keymap
Here is an example of setting the keyboard to the UK keymap.
```
loadkeys uk
```
<br/><br/>
## Change the Console Font
This is strictly an optional step.
### list available fonts.
```
ls /usr/share/kbd/consolefonts/ | more
```
### Set a font.
```
setfont ter-132b
```
<br/><br/>
## Make sure network device is enabled
```
ip link
```
<br/><br/>
## Setup Wireless Internet Connection
If you have a wired connection, or using a VM, you are probably automatically online.
### List Wifi adapter
If you need to setup wireless internet first find the name of your wifi DEVICE. On the left i should have a name for example `wlan0` and on the right `station`.
```
iwctl device list
```
This will be your DEVICE name. The name of your network you're connecting to is `"SSID"` in double quotations.
### Connect Wifi to internet
Replace DEVICE and SSID below with their names. You will be prompted for a password to your SSID.
```
iwctl station DEVICE connect "SSID"
```
<br/><br/>
## Check Wifi or Wired connection
Lets check if you're online.
```
ping archlinux.org
```
<br/><br/>
## Connect with SSH
### Set a root password
```
passwd
```
### Get host ip address
```
ip a
```
### Connect from guest machine
Using the ip address above as IP on the guest system that is on the same network.
```
ssh root@IP
```
<br/><br/>
## Make sure system clock is correct.
```
timedatectl
```
<br/><br/>
## Create Partitions
You will need to create some partitions to install Arch Ainux on.
<br/><br/>
WARNING!! This will permanently destroy some or all data on the device. Please for your own safety remove all unnesseary data storage devices. 
Please backup any data that is on the device you are about to partition that you do not wish to loose. Please check and double check that you are partitioning the correct device.
<br/><br/>
WARNING!! If you are Dual booting from the same device do not change the partition table type, do not change the EFI partition, do not change any partition needed for other systems. The only thing you can do is create a new root partition or optionally a new swap partition. Do not delete or change any other partition, you have been warned. Resizing a partition to make room is beyond the scope of these instructions.
### List devices
```
fdisk -l
```
or 
```
lsblk
```
### fdisk desired device
WARNING!! Changing `Disklable type` will destroy all exisitng data on device. If your dual booting read the above warnings. If Dual booting read the warnings above agian. You can only add a root partition or swap into existing space and nothing else. You have been warned again!
<br/><br/>
Replace sdX with your desired device it may be for example "sda", "vda" or "nvme0n1".
```
fdisk /dev/sdX
```
Print the partition table with `p`. If you don't see "Disklable type: gpt" then press `g` to set it.
<br/><br/>
Create the EFI Boot partition with `n`. Accept all the defaults until "Last sector" input `+512M` or `+1G`.\
Set this partition type as "EFI System" with `t` then `1`.
<br/><br/>
Optionally Create a swap partition with `n`. Accept all the defaults until "Last sector" and input something like `+4G`.\
Set this partition type as "Linux Swap" with `t` and `19`.
<br/><br/>
Create the root partition with `n` and accept all the defaults to the end.
<br/><br/>
WARNING!! This is the point of no return. When you press `w` that acually writes the partition table. Quitting before this will not change the partition table.
<br/><br/>
Print the partition table again with `p`. If it all looks good you can write it with `w` and quit with `q`.
<br/><br/>
## Format partitions
If your device is /dev/sdX then partition 1 will be /dev/sdX1. It is your job to know which partition you are formating. I am just giving examples.
### Format EFI/boot partiton
WARNING!! If you are dual booting do `NOT` format your EFI partition. Doing so will remove the existing data for your other operating systems. If you are dual booting skip this step.
<br/><br/>
Replace /dev/sdX1 with your EFI partition.
```
mkfs.fat -F32 /dev/sdX1
```
### If using swap partition (Legacy)
Replace /dev/sdx2 with your swap partition.
```
mkswap /dev/sdX2
```
<br/><br/>
## Create and mount EXT4 partition (Legacy)
I recommend using BTRFS not EXT4, however, there is instructions for both.
### Format your system partition
Replace /dev/sdX3 with your system partition.
```
mkfs.ext4 /dev/sdX3
```
### Mount root partition first to /mnt
Replace /dev/sdX3 with your root partition.
```
mount -o rw,noatime,discard,data=ordered /dev/sdX3 /mnt
```
### Mount boot partition, creating the directory if needed
Replace /dev/sdX1 with your EFI boot partition.
```
mkdir /mnt/boot
mount -t vfat -o rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro /dev/sdX1 /mnt/boot
```
### Mount optional swap partition (Legacy)
Replace /dev/sdX2 with your swap partition.
```
swapon /dev/sdX2
```
<br/><br/>
## Create and mount BTRFS partitions (Recommended)
I recommend using BTRFS not EXT4, however, there is instructions for both.
### Format your system partition
Replace /dev/sdX3 with your system partition.
```
mkfs.btrfs /dev/sdX3
```
### Mount root partition first to /mnt
Replace /dev/sdX3 with your root partition.
```
mount /dev/sdX3 /mnt
```
### Create subvolumes
```
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@.snapshots
```
### Unmount root partition
```
umount /mnt
```
### Mount root subvolume on /mnt
Replace /dev/sdX3 with your root partion.
```
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@ /dev/sdX3 /mnt
```
### Create needed directories in root subvolume
```
mkdir /mnt/{boot,home,var/log,var/cache/pacman/pkg,.snapshots}
```
### Mount rest of subvolumes
Replace /dev/sdX3 with the root partiton.
```
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@home /dev/sdX3 /mnt/home
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@log /dev/sdX3 /mnt/var/log
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@pkg /dev/sdX3 /mnt/var/cache/pacman/pkg
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@.snapshots /dev/sdX3 /mnt/.snapshots
```
### Mount EFI boot partition
Replace sdX1 with your EFI partition.
```
mount -t vfat -o rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro /dev/vda1 /mnt/boot
```
### Mount optional swap partition (Legacy)
Replace /dev/sdX2 with your swap partition.
```
swapon /dev/sdX2
```
<br/><br/>
## Update repository & install packages
### Enable ParallelDownloads
```
sed -i "s/^#Parallel/Parallel/" /etc/pacman.conf
```
### Run Reflector
Reflector prioritizes closer or faster servers for use with pacman.
```
reflector
```
### Sync pacman repository
```
pacman -Syy
```
### Install Arch Linux with KDE Plasma
Use pacstrap to install a full KDE Plasma system all at once.

    # pacstrap /mnt base linux-firmware linux grub efibootmgr iwd sudo nano vim \
    pacman-contrib htop base-devel xorg-server mesa-demos plasma plasma-wayland-session \
    kde-applications autopep8 chromium ctags firefox flake8 git hunspell-en_gb \
    pulseaudio-bluetooth python-jedi python-language-server python-mccabe python-numpy \
    python-pip python-pycodestyle python-pydocstyle python-pyflakes python-pygame \
    python-pylint python-rope python-wheel yapf


### Install Arch Linux with XFCE
Use pacstrap to install a full XFCE system all at once.

    # pacstrap /mnt base linux-firmware linux grub efibootmgr iwd sudo nano vim \
    pacman-contrib htop base-devel xorg-server mesa-demos xfce xfce-goodies lightdm \
    lightdm-gtk-greeter lightdm-gtk-greeter-settings file-roller gvfs network-manager-applet \
    bluez bluez-utils blueman atril galculator drawing geany geany-plugins xdg-user-dirs-gtk \
    pulseaudio pavucontrol adobe-source-sans-pro-fonts adobe-source-code-pro-fonts \
    gnu-free-fonts ttf-hack noto-fonts noto-fonts-emoji noto-fonts-cjk ttf-roboto autopep8 \
    chromium ctags firefox flake8 git hunspell-en_gb python-jedi python-language-server \
    python-mccabe python-numpy python-pip python-pycodestyle python-pydocstyle \
    python-pyflakes python-pygame python-pylint python-rope python-wheel yapf


### Generate fstab
Generate for your new system.

    # genfstab -U /mnt >> /mnt/etc/fstab

## Configure system in chroot environment
### Chroot
Use arch-chroot to change roots into new system.

    # arch-chroot /mnt

### Select your Locale
Uncomment your locale by removing the # infront of your locale. English UK is "en_GB.UTF-8".

    # nano /etc/locale.gen

### Generate locale

    # locale-gen

### Enable sudo for wheel
We will allow users who are part of the wheel group sudo privilage.

    # sed -i 's/# %wheel ALL=(ALL) NOPASSWD/%wheel ALL=(ALL) NOPASSWD/' /etc/sudoers

### Set keymap
We will set the keymap for the new systme. Replace "uk" with your keymap.

    # echo "KEYMAP=uk" > /etc/vconsole.conf

### Install grub
Install grub into the EFI partion with the command below.

    # grub-install --target=x86_64-efi --bootloader-id=archlinux --efi-directory=/boot/efi

### Configure grub
Grub will automatically create the Arch Linux boot options.

    # grub-mkconfig -o /boot/grub/grub.cfg

### Create hostname
You will need to choose a name that this machine will be called on the network.
An example would me "Chromebook". Replace MYHOST with your chosen hostname.

    # echo MYHOST > /etc/hostname

### Create hosts
We also need to create a hosts file. All 3 lines below create the hosts file.

    # echo "127.0.0.1       localhost" > /etc/hosts

    # echo "::1             localhost" >> /etc/hosts

Replace (both) MYHOST with your chosen hostname in the command below.

    # echo "127.0.0.1       MYHOST.localdomain MYHOST" >> /etc/hosts

### Create a user
We need to create a new user with sudo privialges. Replace MYUSER below with your desired new user name.

    # useradd -m MYUSER -G wheel

Give your new user a password.

    # passwd MYUSER

### Create a root password
Give the root user password too.

    # passwd

### Fix KDE fallback cursor
KDE will sometimes fallback to the Adwaita cursor. Change the default to breeze.

    # sed -i 's/Adwaita/breeze_cursors/' /usr/share/icons/default/index.theme

### Boot into new Arch Linux
We need to exit the chroot environment.

    # exit

Now we need to reboot into the new Arch Linux system. Be sure to remove the USB at shutdown and let it boot into the new system.

    # reboot

## Finale configuration
### Select a Time Zone
The  command piped into more displays only a single page. Pressing enter will let you scroll though the list.

    # timedatectl list-timezones | more

### Set Time Zone
Set your timezone. Replace Europe/London below with yours.
    # timedatectl set-timezone Europe/London

### Update system clock

    # timedatectl set-ntp true

### Set Locale
We will make sure your locale is set. Replace en_GB.UTF-8 with your locale.

    # localectl set-locale LANG=en_GB.UTF-8

### Set keyboard keymap
Double check your console keyboard is set. Replace uk with your keymap from before. The keymap is us by default.

    # localectl set-keymap uk

### See list of xll-keymap layouts
We will set the X11 keyboad keymap. This is different from the keymap above. The x11-keymap is us by default.

    # localectl list-x11-keymap-layouts

### Set x11-keymap
Replace gb with your x11-keymap. The x11-keymap is us by default.

    # localectl set-x11-keymap gb

### Set x11-keymap for SDDM
The KDE login manager SDDM will need it's own x11-keymap setting. Use the same x11-keymap, replacing gb below. The x11-keymap is us by default.

    # echo "setxkbmap gb" > /usr/share/sddm/scripts/Xsetup

### Enable required services
Enabling system services will start them during the next boot.
We need to enable Network Manager to user the internet.

    # systemctl enable NetworkManager

Enable bluetooth servie.

    # systemctl enable bluetooth

Enabling SDDM will make the system boot to the graphical login manager for KDE.

    # systemctl enable sddm

### Reboot
Now it is time to reboot into your new Arch Linux system.

    # reboot

# Other settings and fixes
## Pygame & Pygame Zero
### Install Pygame with pacman
If you haven't already installed pygame use the official Arch Linux packages.
Numpy is also required and we will use pip too.

    # sudo pacman -S --needed python-pygame python-numpy python-pip python-wheel

### Install Pygame Zero
The official release is currently broken and not compatable with the latest python. 
It also attempts to install a non working version of pygame.
We will use the pip command to pull and install the development version without letting it install the pygame package that is not compatable.
Please remember that this development version seems to require the `def update()` with `pass` or any other code in it to update the screen. 
Without this projects from the books will not display if you have not added the update call.

    # sudo pip install git+https://github.com/lordmauve/pgzero.git --no-deps --upgrade
    # sudo pip install pyfxr

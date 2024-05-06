# Arch Linux Installation Guide.
## Overview
This guide will walk you through the steps needed to install a fully functional Arch Linux Desktop.
The focus is KDE, but steps for XFCE will also be shown.
I use an install script to help me remember and automate most of the steps.
There are also some config and skel files included.
I also recommend using the official Arch Linux install guide for support.
I often find answers to my problems hidden there.

https://wiki.archlinux.org/index.php/Installation_guide

## Download and create USB
### Download ISO image
The official Arch Linux download page. https://archlinux.org/download/

A direct link to a mirror with the latest image. https://mirrors.edge.kernel.org/archlinux/iso/latest/

### Create a bootable USB
WARNING! All data on the USB device will be permanently lost.
WARNING! Be sure to check device size and mount points to make sure it is your USB.
To find the name of your USB device. Use the command lsblk.

    # lsblk

Be sure to know the name of your USB. `/dev/sdX` where X is the letter of your USB.
Be careful it is not your hard drive or ssd.
The partition `part /` is the device with your system partition.

We will use the cat command to write the image to the USB. Change directory to where ever you downloaded the file.

    # cd ~/Downloads
    # cat archlinux-*-x86_64.iso > /dev/sdX

# Installation
You will need to boot your system from the USB drive. It may simply need to reboot with the USB left in.
You most likely need to know how to tell your bios to boot from USB devices.
It may be something like pressing `escape`, `F5`, `F8`, `F10` or `F11` during boot up. Search your device and boot from USB.
## Setup keyboard
If you are using a US keyboard you can skip this step.
### List available keyboard keymaps
If you would like to see a list keyboard use this command.

    # ls /usr/share/kbd/keymaps/**/*.map.gz

### Set keyboard keymap
Here is an example of setting the keyboard to the UK keymap.

    # loadkeys uk

## Setup Internet Connection
### List Wifi adapter
If you need to setup wireless internet first find the name of your wife ADAPTER.

    # iwctl device list | grep station | cut -f 3 -d " "

This will be your ADAPTER name. The name of your network is your SSID.
### Connect Wifi to internet
Replace ADAPTER and SSID below with their names. You will be prompted for a password to your SSID.

    # iwctl station ADAPTER connect SSID

### Check Wifi or Wired connection
Lets check if you're online.

    # ping archlinux.org

### Update the system clock

    # timedatectl set-ntp true

## Create Partitions
You will need to create some partitions to install Arch Ainux on. This will permanently destroy any data on the device.
You have been warned. I will assume you have a device that you can create partitions on.
Again any existing partitions will be destroyed along with all data held in them. Get a list of partitions.

    # fdisk -l

Edit the device with fdisk. Replace X with the device letter you are willing to loose all data on.

    # fdisk /dev/sdX

## Format partitions
### Format EFI/boot partitoin
Replace /dev/sdX1 with your EFI partition.

    # mkfs.fat -F32 /dev/sdX1

### If using swap partition
Replace /dev/sdx2 with your swap partition.

    # mkswap /dev/sdX2

### Format your system partition
Replace /dev/sdX3 with your system partition.

    # mkfs.ext4 /dev/sdX3

## Mount partitions

    # mount /dev/sdX3 /mnt
    # mkdir /mnt/boot
    # mkdir /mnt/boot/efi
    # mount /dev/sdX1 /mnt/boot/efi

## Update repository & install packages
### Run Reflector
Reflector prioritizes closer or faster servers for use with pacman.

    # reflector

### Sync pacman repository

    # pacman -Syy

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

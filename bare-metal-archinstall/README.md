![Screenshot](screenshot.png)
# Installing Archlinux on bare-metal with archinstall.

https://wiki.archlinux.org/title/Installation_guide

## Create install USB stick.

Download the archlinux iso. \
https://archlinux.org/download/

To find the name of your USB device, use the command `lsblk`. \
The name of your USB. It will be `/dev/sdX` where sdX is the name of your USB device. 
Be careful it is not your hard drive or SSD. 
The partition `part /` is the device with your system partition. That is NOT the device you want to write to.
```
lsblk
```
Use the `dd` command to write the image to the USB. Change the directory to wherever you downloaded the file. Replace `/dev/sdX` with the name of your device.
```
cd ~/Downloads
dd bs=4M if=archlinux-x86_64.iso of=/dev/sdX conv=fsync oflag=direct status=progress
```
## Prepare host system.
You will need to boot your system from the USB drive. It may be something like pressing `Escape`, `F5`, `F8`, `F10` or `F11` during boot up.

If you are using a US keyboard you can skip this step. \
List available keymaps.
```
localectl list-keymaps
```
Here is an example of setting the UK keymap.
```
loadkeys uk
```
Set a password for the root user. SSH service is already running and accessable as root, but ssh a password set.
```
passwd
```
## Setup Wireless Connection (Optional)
If you have a wired connection, or using a VM, you are probably automatically online.

List Wifi adapter. For example `wlan0` and on the right `station`. This will be your DEVICE name.
```
iwctl device list
```
Scan Available Networks. Replcace DEVICE with your device name. The Network Name will be your SSID.
```
iwctl station DEVICE scan
```
Display the available SSID's.
```
iwctl station DEVICE get-networks
```
Connect Wifi to internet. Replace DEVICE and SSID below with their names. SSID needs to be in quotes. You will be prompted for a password to your SSID.
```
iwctl station DEVICE connect "SSID"
```
## Lets check if you're online.
```
ping archlinux.org
```
Get your IP address. For example 192.168.0.XXX.
```
ip a
```
## Connect the guest to the host system.
On the guest system log into the host using it's IP address and USERNAME. \
You will be asked to make a fingerprint then for the password
```
ssh USERNAME@IP
```
If there is already an existing fingerprint that needs to be removed.
```
ssh-keygen -R IP
```
## Install Archlinux
Run reflector to get fastest servers.
```
reflector
```
Allow Parallel downloads for pacman.
```
sudo sed -i 's/^#Parall/Parall/' /etc/pacman.conf
```
Update available package list.
```
pacman -Syy
```
Install Archlinux with btrfs and compression, systemd-boot, kde desktop and pipewire.
```
archinstall
```
Accept the chroot into the new system.

Enable sudo with NOPASSWD for wheel group.
```
sed -i 's/^# %wheel ALL=(ALL:ALL) NOPASSWD/%wheel ALL=(ALL:ALL) NOPASSWD/' /etc/sudoers
```
Remove user specific config. Replace USERNAME with your username.
```
rm /etc/sudoers.d/00_*
```
Set zram to twice the size of ram.
```
echo "compression-algorithm = zstd" > /etc/systemd/zram-generator.conf
echo "zram-size = ram * 2" >> /etc/systemd/zram-generator.conf
```
Allow Parallel downloads for pacman.
```
sudo sed -i 's/^#Parall/Parall/' /etc/pacman.conf
```
Disable sddm since the graphical login manager wont work with xrdp later.
```
systemctl disable sddm
```
Enable sshd to use ssh with the new installed system.
```
systemclt enable sshd
```
## Wireless setup (Option 1) Copy existing connection.
In order to reboot into the new install but also have it connect with the same IP address so that ssh keeps working we will disable NetworkManager and copy the existing settings over. This will let iwd automatically connect wihtout needing to login. But KDE will not show the network icon since NetworkManager is not running.

Disable NetworkManager as we will use iwd.
```
systemctl disbale NetworkManager
```
Enable iwd.
```
systemclt enable iwd
```
Enable systemd-networkd.
```
systemctl enable systemd-networkd
```
Enable systemd-resolved
```
systemclt enable systemd-resolved
```
Exit chroot.
```
exit
```
Copy configuration files over for systemd-networkd
```
cp /etc/systemd/network/* /mnt/etc/systemd/network
```
Create a simlink for systemd-resolved. Remove `/mnt/etc/resolv.conf` if you disable systemd-resolved later.
```
ln -sf ../run/systemd/resolve/stub-resolv.conf /mnt/etc/resolv.conf
```
Copy iwd login credentials.
```
cp /var/lib/iwd/* /mnt/var/lib/iwd
```
Reboot the host system.
```
reboot
```
## Wireless setup (Option 2)
In KDE NetworkManager will connect to wifi when logged in. But we want wifi connected on bootup. Using nmcli we will set up a connection that will do this. This will mean NetworkManager is still runnning for KDE. But this will mean we need to Interact with the host system again to set this up. This can't be done remotely.

After rebooting the host system login and then connect to wifi with nmcli. Replace SSID with the SSID used above. You will be asked for sudo password then your SSID password.
```
sudo nmcli device wifi connect "SSID" --ask
```
If you get an error about "secrets were asked but not given", try removing the existing NetworkManager saved login.
```
sudo nmcli con del "SSID"
```
Get you IP address like above. It may have changed.
```
ip a
```
## Setting up xrdp
Install needed packages to build xrdp xorgxrdp and pipewire-module-xrdp.
```
cd
sudo pacman -S --needed git fuse imlib2 nasm cmocka check xorg-server-devel
```
Build and install xrdp packages from aur.
```
cd
git clone https://aur.archlinux.org/xrdp.git
cd ~/xrdp
makepkg -i
```
Build and install xorgxrdp from aur.
```
cd
git clone https://aur.archlinux.org/xorgxrdp.git
cd ~/xorgxrdp
makepkg -i --skippgpcheck
```
Build and install pipewire-module-xrdp from aur.
```
cd
git clone https://aur.archlinux.org/pipewire-module-xrdp.git
cd ~/pipewire-module-xrdp
makepkg -i
```
Create an ~/.xinitrc so startx or xrdp will launch kde.
```
echo "export DESKTOP_SESSION=plasma" > ~/.xinitrc
echo "exec startplasma-x11" >> ~/.xinitrc
```
Enable xrdp service.
```
sudo systemctl enable xrdp
```
On Guest system install freerdp.
```
sudo pacman -S --needed freerdp
```
List available xfreerdp key layouts. LAYOUT will be a hex number.
```
xfreerdp3 /list:kbd
```
Check virt-manager for the IP address of the archlinux vm. USERNAME and PASSWD will be the ones you created for this VM.
```
xfreerdp3 /u:USERNAME /p:PASSWD /w:1366 /h:768 /v:IP /video /sound /rfx /network:lan /gfx /dynamic-resolution /bpp:32 /kbd:layout:LAYOUT
```
## Configure Archlinux
Install extra packages including firefox.
```
sudo pacman -S --needed pacman-contrib libva-utils mesa-demos compsize firefox
```
### Make firefox usable. Add extensions.
Ublock Origin \
Sponsor Block \
I still don't care about cookies

### In firefox about:config set
media.av1.enable False \
network.trr.default_provider_uri https://94.140.14.14/dns-query \
network.trr.mode 3

If you want a full featured KDE install add kde-applications.
```
sudo pacman -S --needed kde-applications-meta
```
Change default cursor from Adwaita to breeze. This will fix sddm and other places.
```
sudo sed -i 's/Adwaita/breeze_cursors/' /usr/share/icons/default/index.theme
```
Set a default keyboard for sddm if you don't want to see it set to US by default. \
Get a list of available x11-keymaps.
```
localectl list-x11-keymap-layouts
```
Set an x11-keymap for sddm. Replace gb with your choosen x11-keymap.
```
sudo bash -c "echo 'setxkbmap gb' >> /usr/share/sddm/scripts/Xsetup"
```
## Add Catppuccin color schemes for breeze.
Git color schemes from this repo. There are 3 dark and 1 light.
```
cd
git clone https://github.com/programmingrainbow/Archlinux
cd Archlinux
```
Create a local/share folder if needed and copy the color-schemes folder there.
```
mkdir -p ~/.local/share
cp -r color-schemes ~/.local/share
```
Set global theme to Breeze Dark or Breeze Light. Then in colors set one of the corresponding catppuccin color schemes.

## Use these mount points if repairing an install from iso.
You will need to change the `/dev/sda2` to the device you have btrfs installed to. Change `/dev/sda1` to your boot device.
```
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@ /dev/sda2 /mnt
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@home /dev/sda2 /mnt/home
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@log /dev/sda2 /mnt/var/log
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@.snapshots /dev/sda2 /mnt/.snapshots
mount -t vfat -o rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro /dev/sda1 /mnt/boot
```

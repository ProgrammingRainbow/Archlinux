# Installing Archlinux in virt-manager
## Setting up virt-manager
https://wiki.archlinux.org/title/Installation_guide \
https://archlinux.org/download/

# Create a qemu image of at least 15G
```
cd /var/lib/libvirt/images
sudo qemu-img create -f qcow2 archlinux.qcow2 15G
```

# In virt-manager Create a new VM with atleast 4G of ram and select the image created above.

# In virt-manager set Display Spice -> Listen type to None. Check OpenGL and select your GPU.

# In virt-manager set Video -> Mode to Virtio for virGL and check 3D acceleration, or QXL for llvmpipe.
```
  <model type="virtio" heads="1" primary="yes">
    <acceleration accel3d="yes"/>
    <resolution x="1920" y="1080"/>
  </model>
```
or
```
  <model type="qxl" ram="65536" vram="65536" vgamem="16384" heads="1" primary="yes">
    <resolution x="1920" y="1080"/>
  </model>
```

# Make sure virt-manager is using efi not mbr.
# Replace these lines in virt-managers xml for archlinux.
```
  <os>
    <type arch="x86_64" machine="pc-q35-9.0">hvm</type>
    <boot dev="hd"/>
  </os>
```
# With these lines.
```
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-9.0">hvm</type>
    <firmware>
      <feature enabled="no" name="enrolled-keys"/>
      <feature enabled="yes" name="secure-boot"/>
    </firmware>
    <loader readonly="yes" secure="yes" type="pflash">/usr/share/edk2/x64/OVMF_CODE.secboot.4m.fd</loader>
    <nvram template="/usr/share/edk2/x64/OVMF_VARS.4m.fd">/var/lib/libvirt/qemu/nvram/archlinux_VARS.fd</nvram>
  </os>
```
# Boot VM with ISO mounted.
# Run reflector to get fastest servers.
```
reflector
```
# Update available package list.
```
pacman -Syy
```
# Allow Parallel downloads for pacman.
```
sudo sed -i 's/^#Parall/Parall/' /etc/pacman.conf
```
# Install Archlinux with btrfs and compression, systemd-boot and pipewire.

# Dismount install ISO and reboot.

# Allow Parallel downloads for pacman.
```
sudo sed -i 's/^#Parall/Parall/' /etc/pacman.conf
```
# Install extra packages.
```
sudo pacman -S --needed pacman-contrib libva-utils mesa-demos compsize firefox
```
# Make firefox usable. Add extensions.
Ublock Origin
Sponsor Block
I still don't care about cookies
h264ify

# In firefox about:config set
media.av1.enable False
network.trr.default_provider_uri https://94.140.14.14/dns-query
network.trr.mode 3

# If you want a full featured KDE install.
```
sudo pacman -S --needed kde-applications-meta
```
# Add user to wheel group if not already in it. Enable sudo with NOPASSWD for wheel group.
```
groups
suod usermod -aG wheel pr
sudo sed -i 's/^# %wheel ALL=(ALL:ALL) NOPASSWD/%wheel ALL=(ALL:ALL) NOPASSWD/' /etc/sudoers
sudo rm /etc/sudoers.d/00_pr
```
# Change default cursor from Adwaita to breeze.
```
sudo sed -i 's/Adwaita/breeze_cursors/' /usr/share/icons/default/index.theme
```
# Set a default keyboard for sddm
```
sudo bash -c "echo 'setxkbmap gb' >> /usr/share/sddm/scripts/Xsetup"
```

# install needed packages to build xrdp xorgxrdp and pipewire-module-xrdp
```
cd
sudo pacman -S --needed git fuse imlib2 nasm cmocka check xorg-server-devel
```
```
git clone https://aur.archlinux.org/xrdp.git
cd ~/xrdp
sudo makepkg -i
```
```
cd ~/xorgxrdp
git clone https://aur.archlinux.org/xorgxrdp.git
sudo makepkg -i --skippgpcheck
```
```
cd ~/pipewire-module-xrdp
git clone https://aur.archlinux.org/pipewire-module-xrdp.git
sudo makepkg -i
```

# Create ~/.xinitrc so that startx or xrdp will stark kde.
```
echo "export DESKTOP_SESSION=plasma" > ~/.xinitrc
echo "exec startplasma-x11" >> ~/.xinitrc
```
# Enable xrdp service
```
sudo systemctl enable xrdp
```
# Disable sddm service
```
sudo systemctl disable sddm
```
# Reboot the VM.

# list available xfreerdp key layouts.
```
xfreerdp3 /list:kbd
```
```
xfreerdp3 /u:pr /p:secret /w:1366 /h:768 /v:192.168.122.59 /video /sound /rfx /network:lan /gfx /dynamic-resolution /bpp:32 /kbd:layout:0x00000809
```
# Use these mount points if repairing an install from iso.
```
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@ /dev/vda2 /mnt
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@home /dev/vda2 /mnt/home
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@log /dev/vda2 /mnt/var/log
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@pkg /dev/vda2 /mnt/var/cache/pacman/pkg
mount -o ssd,discard=async,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@.snapshots /dev/vda2 /mnt/.snapshots
mount -t vfat -o rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro /dev/vda1 /mnt/boot
```

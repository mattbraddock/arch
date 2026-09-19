# My Arch Installation

Download the image from [https://archlinux.org/download/](https://archlinux.org/download/)

Get drive name
```bash
ls -l /dev/disk/by-id/usb-*
```

Ensure it is not mounted with `lsblk`

Copy to the flash drive
```bash
dd bs=4M if=PATH/TO/archlinux-version-x86_64.iso of=/dev/disk/by-id/usb-FLASHDRIVENAME conv=fsync oflag=direct status=progress
```

Insert flash drive and boot into install

Verify boot mode if UEFI
```bash
ls /sys/firmware/efi/efivars
```

Check for network connectivity
```bash
ip link
ping archlinux.org
```

Update clock
```bash
timedatectl set-ntp true
```

Get device list and identify drive label (typically `/dev/sdX` or `/dev/nvme0nX`)
```bash
fdisk -l
```

Partition drive
```bash
fdisk /dev/XXX
```

Commands:
- `d` deletes partitions
- `g` creates a partition table
- `n` creates a new partition
- `t` changes the type of the partition
- `w` saves the partitions

First partition should be +512M and type 1. Second partition should be +8G and type 19. Third partition should be remaining size.

Format partitions
```bash
mkfs.fat -F32 /dev/XXX1
mkswap /dev/XXX2
swapon /dev/XXX2
mkfs.ext4 /dev/XXX3
```

Mount partitions
```bash
mount /dev/XXX3 /mnt
mkdir /mnt/boot
mount /dev/XXX1 /mnt/boot
```

Install base Linux
```bash
pacstrap /mnt base base-devel linux linux-firmware vim nano
```

Generate fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Chroot into installation
```bash
arch-chroot /mnt
```

Set up locale
```bash
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

Set up hostname (choose what `HOST` is)
```bash
echo HOST >> /etc/hostname
```

Edit `/etc/hosts` with the following
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   HOST.localdomain    HOST
```

Install bootloader
```bash
pacman -S efibootmgr intel-ucode linux-headers grub
grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Create root password, then create sudo user (choose what `USER` is)
```bash
passwd
useradd -m USER
passwd USER
groupadd sudo
usermod -aG wheel USER && usermod -aG sudo USER
```

Edit `visudo` by uncommenting `%wheel ALL=(ALL) ALL` then exit vim with `:wq`

Edit `/etc/pacman.conf` by uncommenting the two lines for multilib

Install the display manager
```bash
pacman -Syu && pacman -S sddm-kcm
systemctl enable sddm.service
```

Install essential programs
```bash
pacman -S git go labwc networkmanager openssh pacman-contrib xdg-user-dirs
systemctl enable NetworkManager.service
```

Install additional programs

Basics:
```bash
pacman -S brightnessctl cliphist conky ffmpeg firefox foot grim mako mousepad qt5-wayland qt5-graphicaleffects qt5-quickcontrols2 rofi slurp swaybg swayidle swaylock thunar waybar wlopm wlr-randr
```

Connectivity
```bash
pacman -S bluez bluez-utils cups nm-connection-editor
systemctl enable bluetooth.service
systemctl enable cups.service
```

Audio/Video
```bash
pacman -S pamixer pavucontrol-qt pulseaudio speech-dispatcher vlc
```

Archiving and file systems
```bash
pacman -S 7zip bzip3 dosfstools file-roller nfs-utils ntfs-3g unrar unzip zip
```

Developing
```bash
pacman -S lite-xl nodejs npm
```

Install fonts and language
```bash
pacman -S hunspll hunspell-en_us ttf-ibm-plex ttf-nerd-fonts-symbols woff2-font-awesome
```

Plugins
```bash
pacman -S cups-filters cups-pdf lite-xl-plugin-manager rofi-calc thunar-archive-plugin thunar-shares-plugin thunar-volman vlc-plugins-extra
```

Miscellaneous
```bash
pacman -S adw-gtk-theme gsimplecal wacomtablet
```

Set up user and install AUR helper
```bash
su - USER
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
cd .. && rm -rf yay
xdg-user-dirs-update
mkdir "${HOME}/.npm-packages"
npm config set prefix "${HOME}/.npm-packages"
gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3-dark'
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

Edit `.bashrc` with the following
```
NPM_PACKAGES="${HOME}/.npm-packages"
export PATH="$PATH:$NPM_PACKAGES/bin"
```

Press `CTRL+D` twice to return to installation medium, and use `reboot` to restart the system

After logging in, install other programs
```bash
sudo pacman -Sy discord \
                f3d \
                filezilla \
                font-manager \
                gimp \
                gparted \
                guvcview-qt \
                kicad \
                kicad-library \
                kicad-library-3d \
                libreoffice-fresh \
                mixxx \
                obs-studio \
                obsidian \
                pandoc \
                rpi-imager \
                steam \
                strawberry \
                syncthing \
                texlive-basic \
                texlive-binextra \
                texlive-fontsextra \
                texlive-fontsrecommended \
                texlive-latexextra \
                texlive-latexrecommended \
                texlive-mathscience \
                xournalpp
```

```bash
sudo yay -S bambustudio-appimage \
            chitubox-free-bin \
            diylc \
            gwenview-no-purpose \
            neofetch \
            numworks-epsilon \
            okular-no-purpose \
            openscad-snapshot-appimage \
            rofi-power-menu \
            ticemu \
            zoom
```

For emulation
```bash
sudo yay -S azahar-appimage-wayland \
            dolphin-emu \
            fceux \
            m64py \
            melonds \
            mgba-qt \
            ryujinx \
            sameboy
```

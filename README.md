# archlinux-installation-guide

## Installation

### Connect to the internet with iwctl

```bash
iwctl station wlan0 scan
```

```bash
iwctl station wlan0 connect <WIFIID>
```

```bash
iwctl station wlan0 show
```

### Update the system clock

```bash
timedatectl
```

### Partition the disks

Disk partitioning with complete data erasure

```bash
cfdisk -z /dev/nvme0n1
```

> choose GPT
>
> ```plaintext
> 1G - EFI System
> <rest>G - Linux filesystem
> ```

```bash
mkfs.fat -F 32 -n BOOT /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

### vconsole

Create `vconsole.conf` before `pacstrap`

```bash
mkdir -p /mnt/etc
```

```bash
nano /mnt/etc/vconsole.conf
```

> write
>
> ```plaintext
> KEYMAP=us
> ```

### Install packages

```bash
pacstrap -K /mnt base \
    base-devel linux \
    linux-headers \
    linux-firmware \
    amd-ucode \
    grub efibootmgr \
    zram-generator \
    networkmanager \
    neovim zsh
```

### Generate fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

```bash
arch-chroot /mnt
```

### Timezone setup

```bash
ln -sf /usr/share/zoneinfo/<REGION>/<CITY> /etc/localtime
```

```bash
hwclock --systohc
```

```bash
systemctl enable systemd-timesyncd
```

```bash
timedatectl status
```

### Localization setup

```bash
nvim /etc/locale.gen
```

> uncomment
>
> ```plaintext
> en_US.UTF-8 UTF-8
> ...
> ru_RU.UTF-8 UTF-8
> ```

```bash
locale-gen
nvim /etc/locale.conf
```

> write
>
> ```plaintext
> LANG=en_US.UTF-8
> ```

### Network configuration

```bash
nvim /etc/hostname
```

> write
>
> ```plaintext
> <HOSTNAME>
> ```

```bash
systemctl enable NetworkManager
```

### Initramfs

```bash
mkinitcpio -P
```

### Setup root password

```bash
passwd
```

### Setup grub

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Exit

```bash
exit
```

```bash
umount -R /mnt
```

```bash
shutdown now
```

## After installation

Login as `root`

### Connect to the internet

```bash
nmtui
```

### Setup user

```bash
useradd -m -G wheel -s /bin/zsh <USERNAME>
```

```bash
passwd <USERNAME>
```

```bash
EDITOR=nvim visudo
```

> uncomment
>
> ```plaintext
> %wheel ALL=(ALL:ALL) ALL
> ```

### Setup pacman in installed system

```bash
nvim /etc/pacman.conf
```

> uncomment
>
> ```plaintext
> Color
> ...
> [multilib]
> Include = ...
> ```

```bash
pacman -Syu
```

### Install base packages

```bash
pacman -S bluez bluez-utils \
    gvfs gvfs-mtp \
    gvfs-gphoto2 \
    sof-firmware \
    pipewire wireplumber \
    pipewire-audio pipewire-alsa \
    pipewire-pulse pipewire-jack \
    ttf-jetbrains-mono \
    noto-fonts noto-fonts-cjk \
    noto-fonts-emoji \
    noto-fonts-extra \
    otf-font-awesome \
    alacritty tmux \
    git eza bat \
    inxi htop tree \
    ripgrep fastfetch \
    unzip unrar openssh
```

### Enable zram

```bash
nvim /etc/systemd/zram-generator.conf
```

> write
>
> literally "ram / 2", not 8G
>
> ```plaintext
> [zram0]
> zram-size = ram / 2
> compression-algorithm = zstd
> ```

```bash
systemctl daemon-reload
```

```bash
systemctl start systemd-zram-setup@zram0.service
```

```bash
zramctl
```

### Enable base services

#### bluetooth

```bash
systemctl enable --now bluetooth
```

#### fstrim

```bash
systemctl enable --now fstrim.timer
```

```bash
fstrim -v /
```

### GNOME

```bash
pacman -S gdm gnome-shell \
    nautilus evince \
    file-roller gnome-keyring \
    gnome-control-center \
    gnome-tweaks \
    gnome-themes-extra \
    papirus-icon-theme \
    gnome-shell-extensions \
    gnome-shell-extension-appindicator \
    xdg-desktop-portal \
    xdg-desktop-portal-gnome
```

#### Enable GDM

```bash
systemctl enable --now gdm
```

## After logging in as \<USERNAME\>

### yay

```bash
git clone https://aur.archlinux.org/yay.git
```

```bash
cd yay
```

```bash
makepkg -si
```

### docker

```bash
sudo pacman -S docker \
    docker-compose \
    docker-buildx
```

```bash
sudo systemctl enable --now docker.socket
```

```bash
sudo usermod -aG docker <USERNAME>
```

```bash
newgrp docker
```

```bash
docker run hello-world
```

### GUI apps

From the official repositories

```bash
sudo pacman -S obs-studio \
    qbittorrent \
    vlc vlc-plugin-ffmpeg \
    v4l2loopback-dkms v4l2loopback-utils
```

From AUR

```bash
yay -S google-chrome \
    visual-studio-code-bin
```

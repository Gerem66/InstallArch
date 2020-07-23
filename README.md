# InstallArch

## Mettre le clavier en français :
loadkeys fr

## Partitionnement :
cfdisk
1 : 20GO    | Linux filesystem
2 : 512MO   | EFI System        | EF00
3 : 4GO     | Linux swap        | 8200
4 : ..GO    | Linux filesystem

## Formatage des partitions :
mkfs.ext4 /dev/sda1
mkfs.fat -F32 /dev/sda2
mkfs.ext4 /dev/sda4

## Partition swap :
mkswap /dev/sda3
swapon /dev/sda3

## Points de montage :
mount /dev/sda1 /mnt
mkdir /mnt/{boot,boot/efi,home}
mount /dev/sda2 /mnt/boot/efi
mount /dev/sda4 /mnt/home

## Choisir le miroir :
nano /etc/pacman.d/mirrorlist
>> alt-r pour tout remplacer

## Installer les outils de base :
pacstrap /mnt base base-devel pacman-contrib
pacstrap /mnt zip unzip p7zip nano vim mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion
pacstrap /mnt linux (linux-lts)

## Générer le fstab :
genfstab -U -p /mnt >> /mnt/etc/fstab

## Installer le chargeur de démarrage (grub)
pacstrap /mnt grub os-prober efibootmgr

## Réglages de l'OS :
arch-chroot /mnt
nano /etc/vconsole.conf
> KEYMAP=fr-latin9
> FONT=eurlatgr
nano /etc/locale.conf
> LANG=fr_FR.UTF-8
> LC_COLLATE=C
nano /etc/locale.gen
>> Vérifier que les lignes "fr_FR.UTF-8 UTF-8" et "en_US.UTF-8 UTF-8" sont décommentées
locale-gen
export LANG=fr_FR.UTF-8
nano /etc/hostname
>> NomDuPC

## Réglage du fuseau horaire :
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc

## Générer l'image du noyau :
mkinitcpio -p linux (linux-lts)

## Installation :
mount | grep efivars &> /dev/null || mount -t efivarfs efivarfs /sys/firmware/efi/efivars
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck

mkdir /boot/efi/EFI/boot
cp /boot/efi/EFI/arch_grub/grubx64.efi /boot/efi/EFI/boot/bootx64.efi

grub-mkconfig -o /boot/grub/grub.cfg
passwd root

pacman -Syy networkmanager
systemctl enable NetworkManager

exit
umount -R /mnt
reboot

## Environnement graphique et logiciels de base:
pacman -Syy ntp cronie
pacman -S gst-plugins-{base,good,bad,ugly} gst-libav
pacman -S xorg-{server,xinit,apps} xf86-input-{mouse,keyboard} xdg-user-dirs
pacman -S ttf-{bitstream-vera,liberation,freefont,dejavu} freetype2
pacman -S libreoffice-fresh-fr hunspell-fr
pacman -S chromium

useradd -m -g wheel -c 'Nom complet de l’utilisateur' -s /bin/bash nom-de-l’utilisateur
passwd nom-de-l’utilisateur

sudo nano /etc/sudoers
>> Décommenter la ligne qui suit "#Uncomment to allow members of group wheel to execute any command"

sudo pacman -S linux-headers (linux-lts-headers)
sudo pacman -Syu

## Gnome :
pacman -S gnome gnome-extra
## Deepin-Desktop : (fail)
sudo pacman -S gvfs-{afc,goa,google,gphoto2,mtp,nfs,smb}
sudo pacman -S deepin deepin-extra
sudo pacman -S lightdm-gtk-greeter lightdm-gtk-greeter-settings
## Deepin :
sudo pacman -S xorg xorg-server
sudo pacman -S deepin deepin-extra
sudo nano /etc/lightdm/lightdm.conf
>> Remplacer "#greeter-session=example-gtk-gnome" par "greeter-session=lightdm-deepin-greeter"

sudo localectl set-x11-keymap fr

## Startup Processes
systemctl enable cronie → *pour les tâches récurrentes*
systemctl enable bluetooth → *uniquement si on a du matériel bluetooth*
systemctl enable ntpd → *pour synchroniser l’heure en réseau.*
systemctl enable GDM (lightdm) → *pour démarrer l'environnement graphique au démarrage*

reboot

# Source :
* https://github.com/FredBezies/arch-tuto-installation/blob/master/install.md

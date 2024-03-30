# ARTIX LINUX RUNIT INSTALL WITH ENCRYPTION

## Install in .iso
Descarga el .iso de artix runit desde su pagina
Para bootear el usb 

```
lsblk (identificar usb)
umount /dev/sda (o el que salga en el paso de arriba)
mkfs.vfat -F 32 /dev/sda -I
dd if=distro.iso of=/dev/sda (o el nombre)
```

## USB Booteable
Entrar con 
```
root (user)
artix (passwd)
```

Luego
```
$ loadkeys la-latin1
```

## Wifi
```
connmanctl

enable wifi

scan wifi

services

agent on

connect (id obtained from services, el largo)

quit

ping artixlinux.org
```

## Drive Partition and Formating
```
lsblk (buscar nombre del disk)
fdisk /dev/nvme0n1
d 
d 
(usar d las veces que sean necesarias para eliminar particiones antiguas)

n (crear partición boot)
*enter*
*enter*
+1G (Last sector)


n (crear partición boot)
*enter*
*enter*
*enter*
(remover signature si sale "Y")

w 

[Paso no necesario para borrar todo bien de una particion primero usar dd if=/dev/urandom of=/dev/nvme0n1p2(nombre particion)]

lsblk
mkfs.fat -F 32 /dev/nvme0n1p1 (nombre de particion boot)
```

## Root Encryption
```
lsblk
cryptsetup luksFormat /dev/nvme0n1p2
YES
contraseña secreta
contraseña secreta

cryptsetup open /dev/nvme0n1p2 Alpha (abre la encriptación y se le asigna un nombre a elección)
contraseña secreta

mkfs.btrfs /dev/mapper/Alpha
```

## Mount Partitions
```
 mkdir /mnt/boot
 mount /dev/mapper/Alpha /mnt (importante el orden)
 mount /dev/nvme0n1p1 /mnt/boot/ (UEFI y BIOS)
 
```

## Install Packages
```
basestrap -i /mnt base base-devel runit elogind-runit linux linux-firmware grub networkmanager networkmanager-runit cryptsetup lvm2
lvm2-runit neovim vim efibootmgr(If UEFI)
```

```
fstabgen -U /mnt >> /mnt/etc/fstab        <- edit and verify, also set root, swap, home and etc..

Check the resulting fstab for errors before rebooting. Now, you can chroot into your new Artix system with:
```

## Chroot to /mnt
```
artix-chroot /mnt bash
```

## Settings

```
ln -sf /usr/share/zoneinfo/Brazil/West /etc/localtime
hwclock --systohc

vim /etc/locale.conf
export LANG="en_US.UTF-8"
export LC_COLLATE="C"
:wq

nvim /etc/locale.gen
/en_US (buscar el UTF-8 y descomentarlo)
:wq

echo "Artix" > /etc/hostname (ingresa como quieres llamar al host)

vim /etc/hosts
127.0.0.1        localhost
::1              localhost
127.0.1.1        Artix.localdomain  Artix (el nombre elegido antes)
:wq

ln -s /etc/runit/sv/NetworkManager /etc/runit/runsvdir/current (activar internet on boot nm)

passwd
clave super secreta de root

useradd -mG wheel user
passwd user
contraseña secreta de user

EDITOR=vim visudo (creo que larbs.sh lo hace solo)
uncomment %wheel ALL (...): ALL
:wq

vim /etc/runit/sv/agetty-tty1/conf (para autologin) (opcional)
GETTY_A..."... --autologin user" (nombre de user)
:wq

vim /etc/mkinitcpio.conf
HOOKS=(..... block encrypt lvm2 ...)
:wq
mkinitcpio -p linux

exit
lsblk -f >> /mnt/etc/default/grub
artix-chroot /mnt bash
vim /etc/default/grub (aqui subir lo que esta abajo lo importante son los uuid de root y su encriptcion)
LINUX_DEFAULT="... quiet cryptdevice=UUID=copiar aqui el UUID del crypto_LUKS:Alpha root=UUID=poner uuid del btrfs" (el nombre puede ser cualquiera)
descomentar .... BLE_CRYPTDISK=y
:wq

grub-install /dev/nvme0n1 (for LEGACY BIOS)
grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=GRUB  (for UEFI systems)
grub-mkconfig -o /boot/grub/grub.cfg

exit                           <- exit chroot environment
umount -R /mnt
reboot
```

## Post Install

### Install Larbs
```
$ pacman -Syu
$ pacman -S git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd

curl -LO larbs.xyz/larbs.sh
sh larbs.sh
```

### Larbs Config
```
$ nvim /etc/X11/xorg.conf.d/30-keyboard.conf
Section "InputClass"
    Identifier "keyboard"
    MatchIsKeyboard "on"
    Option "XKbLayout" "latam"
EndSection
:wq

$ pacman -S python-pywal

$ cd /etc/sudoers.d
rm 01 y larbs-temp

$ pacman -S alsa-utils

git clone https://github.com/NvChad/starter ~/.config/nvim && nvim
:MasonInstallAll
Lazy Sync
rm -rf ~/.config/nvim/.git

$ pacman -S ufw

$ pacman -S veracrypt

$ pacman -S clamav

(script de apagar en thinkpad)

virtmanager zzz
```


# Creare una Debian Stretch Live Custom persistente Sicura
- [Riferimento](https://francoconidi.it/creare-una-debian-stretch-live-custom-persistente-sicura/)

Guida su come creare una propria Debian Live personalizzata con partizione persistente in modalità UEFI. Ho utilizzato direttamente l’immagine [live ufficiale](https://www.debian.org/CD/live/) con firmware non-free. Questo metodo utilizza un’approccio diverso nella costruzione della live, ma l’obiettivo è quello di avere una usb bootable con i propri tools preferiti, e che ci permette di navigare in internet in sicurezza senza memorizzare dati su disco, e senza rinunciare ad una partizione dove poter stipare dati sensibili. Per cifrare la partizione persistente userò LUKS.

Pacchetti da installare:

    $ sudo apt install -y debootstrap grub-common grub-pc-bin grub-efi-amd64-bin efibootmgr syslinux squashfs-tools cryptsetup

Creazione Environment:

    $ sudo mkdir $HOME/debian_live
    $ sudo debootstrap --arch=amd64 --variant=minbase stretch $HOME/debian_live/chroot http://ftp.it.debian.org/debian/
    $ sudo chroot $HOME/debian_live/chroot
    # echo "debian-live" > /etc/hostname
    # echo 'deb https://ftp.it.debian.org/debian/ stretch main contrib non-free' > /etc/apt/sources.list
    # apt update

Installazione kernel:

    # apt-cache search linux-image
    # apt install -y linux-image-4.9.0-4-amd64 linux-headers-4.9.0-4-amd64

Installazione dei pacchetti personalizzati (nel mio caso solo quelli sotto)

    # apt install -y mate-desktop-environment task-laptop xorg xinit xserver-xorg-input-evdev xserver-xorg-input-libinput xserver-xorg-input-kbd live-boot systemd-sysv network-manager net-tools wireless-tools nano gparted x11-xserver-utils x11-utils pciutils usbutils ntfs-3g rsync dosfstools syslinux firefox-esr chromium xserver-xorg-input-synaptics vpnc network-manager-gnome network-manager-vpnc-gnome network-manager-openvpn-gnome gtkterm vsftpd putty openssh-client firmware-linux-nonfree firmware-iwlwifi firmware-linux-free vlc terminator synaptic p7zip-full rar unrar zip ssh wget curl mesa-utils dnsmasq grub2-common grub-efi-amd64 grub-pc-bin xkb-data keyboard-configuration tzdata locales cryptsetup encfs wireshark-gtk aircrack-ng nmap zenmap
    # apt clean

Password per root:

    # passwd root
    # exit

Comprimere il tutto in uno squash filesystem:

    $ sudo mkdir -p $HOME/debian_live/image/
    $ sudo mksquashfs $HOME/debian_live/chroot $HOME/debian_live/image/filesystem.squashfs -noappend -e boot

Copiare kernel e initramfs fuori chroot:

    $ sudo cp $HOME/debian_live/chroot/boot/vmlinuz-4.9.0-4-amd64 $HOME/debian_live/image/vmlinuz-4.9.0-4-amd64
    $ sudo cp $HOME/debian_live/chroot/boot/initrd.img-4.9.0-4-amd64 $HOME/debian_live/image/initrd.img-4.9.0-4-amd64

Preparazione chiavetta usb: (nel mio caso /dev/sda di 16G)

Negli step successivi verranno create 3 partizioni:

1. partizione in fat32 EFI
2. partizione in fat32 che contiene il sistema Linux
3. partizione in ext4 persistente criptata con LUKS
```
$ sudo fdisk -l
$ sudo umount /dev/sda*
$ sudo dd count=1 bs=512 if=/dev/zero of=/dev/sda
$ sudo parted -s -- /dev/sda mktable gpt mkpart efi 1 100M set 1 boot on
$ sudo parted -s -- /dev/sda mkpart system 100 2G
$ sudo parted -s -- /dev/sda mkpart persistence 2G 16G
$ sudo mkdir -p /mnt/{usb,efi,persistence}
$ sudo mkfs.fat -F32 -n efi /dev/sda1
$ sudo mkfs.fat -F32 -n system /dev/sda2
$ sudo mkfs.ext4 -L persistence /dev/sda3
```
Montaggio delle partizioni EFI e di Sistema:

    $ sudo mount /dev/sda1 /mnt/efi/
    $ sudo mount /dev/sda2 /mnt/usb/

Installazione di Grub-EFI:

    $ sudo grub-install --target=x86_64-efi --efi-directory=/mnt/efi --boot-directory=/mnt/usb/boot --removable --recheck

Copiare il sistema nella seconda partizione:

    $ sudo rsync -rv $HOME/debian_live/image/ /mnt/usb/live/

Creare il file grub.cfg ed incollare dentro:

    $ sudo nano /mnt/usb/boot/grub/grub.cfg

    set default="0"
    set timeout=3
    menuentry "Debian Custom Live" {
        linux /live/vmlinuz-4.9.0-4-amd64 boot=live persistence persistence-encryption=luks
        initrd /live/initrd.img-4.9.0-4-amd64
    }

Creazione della partizione persistente criptata con LUKS:

    $ sudo cryptsetup --verbose --verify-passphrase luksFormat /dev/sda3

rispondere YES

    $ sudo cryptsetup luksOpen /dev/sda3 live
    $ sudo mkfs.ext4 -L persistence /dev/mapper/live
    $ sudo  mount /dev/mapper/live /mnt/usb
    $ sudo su
    # echo "/ union" > /mnt/usb/persistence.conf
    # umount /mnt/usb/
    # cryptsetup luksClose live
    # umount /dev/sda*
    # exit

sopra non abbiamo fatto altro che creare una cartella live che verrà montata in /dev/mapper/, ed aprire e chiudere la partizione persistente. Per accedere alla partizione da adesso in poi bisognerà digitare una password, dopodichè si potranno mettere i files importanti da portarsi dietro.

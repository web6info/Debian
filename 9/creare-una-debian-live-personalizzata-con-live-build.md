# Creare una Debian Live personalizzata con live build
- [Riferimento](https://francoconidi.it/creare-una-debian-live-personalizzata-con-live-build/)

Metodo per costruirsi una propria distribuzione Linux personalizzata basata su Debian basata su [debootstrap](https://wiki.debian.org/Debootstrap).
Leggere prima la documentazione su [live-config](https://manpages.debian.org/jessie/live-config-doc/live-config.7.en.html), e [live-build](https://manpages.debian.org/stretch/live-build/live-build.7.en.html).

Alcuni esempi di config:
- [Parrot](https://nest.parrotsec.org/parrot-build/parrot-build/blob/master/auto/config)
- https://gist.github.com/detly/54e07c56f68b9d2ed066
```
# apt install -y debootstrap grub-common grub-pc-bin grub-efi-amd64-bin efibootmgr syslinux squashfs-tools live-build live-tools live-boot
$ mkdir live; cd live
$ lb config --distribution stretch --binary-images iso-hybrid --architectures amd64 --archive-areas "main contrib non-free" --debian-installer-gui "true" debian-installer "live" --mirror-bootstrap https://ftp.it.debian.org/debian/ --mirror-binary https://ftp.it.debian.org/debian --bootappend-live "boot=live components timezone=Europe/Rome locales=en_GB.UTF-8 keyboard-layouts=it hostname=Debian-Custom username=user noeject autologin"
```
per installare un desktop environment e tutti i programmi di cui si necessita, bisogna editare il file `live.list.chroot`:
```
$ nano config/package-lists/live.list.chroot
```
ed incollare dentro il tutto:
```
mate-desktop-environment task-laptop xorg xinit xserver-xorg-input-evdev xserver-xorg-input-libinput xserver-xorg-input-kbd
```
volendo si può anche cambiare l'immagine di boot, proprio come ho fatto io. Serve un immagine di 640×480 dal nome splash.png, crearsela oppure scaricare la mia. Ipotizzando che il file splash.png si trova nella home:

    $ cd config; mkdir bootloaders
    $ sudo cp -r /usr/share/live/build/bootloaders/isolinux bootloaders/
    $ sudo rm bootloaders/isolinux/splash.svg
    $ sudo cp $HOME/splash.png bootloaders/isolinux/splash.png
    $ cd ..
    $ sudo lb build

a questo punto non resta che provare la nostra Debian Stretch personalizzata, tramite VirtualBox oppure direttamente da chiavetta usb:

    $ sudo su
    # dd if=live-image-amd64.hybrid.iso of=/dev/sdX bs=4M status=progress

con questa configurazione utente e password sono le solite:

    user=user
    password=live


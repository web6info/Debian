## Abilitare/disabilitare touchpad da tastiera

[Riferimento](https://nitstorm.github.io/blog/disabling-touchpad-laptop-linux/)

```bash
# apt update
# apt install xinput
```
Mostrare la lista dei dispositivi di input
```bash
$ xinput list
⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
⎜   ↳ PixArt HP X500 USB Optical Mouse        	id=9	[slave  pointer  (2)]
⎜   ↳ SynPS/2 Synaptics TouchPad              	id=12	[slave  pointer  (2)]
⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
```
Creare lo script `/usr/local/bin/enable-touchpad` contenente il seguente codice:
```bash
#!/bin/bash

xinput set-prop `xinput list | grep -i 'DELL0768:00 06CB:7E92' | cut -f 2 | grep -oE '[[:digit:]]+'` "Device Enabled" 1
```
Creare lo script `/usr/local/bin/disable-touchpad` contenente il seguente codice:
```bash
#!/bin/bash

xinput set-prop `xinput list | grep -i 'DELL0768:00 06CB:7E92' | cut -f 2 | grep -oE '[[:digit:]]+'` "Device Enabled" 0
```
In impostazioni->tastiera creare due sequenze di tasti (per es. `ctrl+1` e `ctrl+2`) per lanciare gli script creati sopra.

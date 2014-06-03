Aggiungere nuovi fonts
======
Abbiamo una mole di fonts all'interno di una cartella e vogliamo che siano installati sul nostro sistema Ubuntu.
Copiamo la cartella dei fonts `<dirfonts>` nella cartella di sistema:
```
# cp <dirfonts> /usr/share/fonts
```
Oppure si possono copiare i fonts nella cartella nascosta `.fonts` presente nella propria home
```
$ cp <dirfonts> ~/.fonts
```
Infine bisogna lanciare il seguente comando
```
# fc-cache -fv
```

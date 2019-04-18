# Introduction

Cette documentation est rédigée par F4HWN Armel. Elle vise à accompagner les Radio Amateurs désireux de découvrir le RRF et de mettre en œuvre un noeud ou hotspot de type Spotnik.

# Partie Hardware

## Spotnik Beta sur CRT FP00

Raccordement d'un Spotnik Beta sur CRT FP00 par F1GWX Olivier. Le CRT FP00 est un pocket bibande VHF (1W/5W) et UHF (1W/4W). Il offre un bon compromis au Baofeng UV5R tout en respectant les normes techniques, environnementales, etc. (norme CE, Directive RoHS, etc.).  

![alt text](https://github.com/armel/RRF4Noobs/blob/master/img/spotnik-crt-fp00.jpg)

## Spotnik Beta sur CRT Space U ou V

Raccordement d'un Spotnik Beta sur CRT Space U ou V par F1GWX Olivier. Les CRT Space U ou Space V sont des Transceivers mobiles offrant 5W, 10W ou 17W de puissance. Très compactes, ils sont déjà utilisés sur plusieurs noeuds du RRF (F1GWX, F5NKP, F4HWN, etc.). 

![alt text](https://github.com/armel/RRF4Noobs/blob/master/img/spotnik-crt-space.jpg)

# Partie Software

## Téléchargement de l'image Spotnik

Télécharger l'image Spotnik sur le site [FTP](ftp://rrf.f5nlg.ovh/) de F5NLG. Il existe différentes images en fonction de la carte que vous allez utiliser (Orange Pi Zero, Raspberry, etc.).

Etant sur Orange Pi Zero, j'ai téléchargé la [version 1.9](ftp://rrf.f5nlg.ovh/spotnik-1.9_opi.img.7z).

## Préparation de la carte micro SD

Mon poste de travail étant sous Macos, j'ai utilisé l'excellente application [ApplePi-Baker](https://www.tweaking4all.com/hardware/raspberry-pi/macosx-apple-pi-baker/) afin de formater la carte micro SD et copier l'image Spotnik. 

Commençons par lancer l'application ApplePi-Backer et formater la carte micro SD. Pour se faire, depuis l'application, 

- sélectionner la carte SD cible
- cliquer sur le bouton "Prep for NOOBS"

Une fois cette étape terminée, nous allons copier l'image Spotnik. L'image étant au format 7z après téléchargement via le site FTP, je recommande de la décompresser au format .img avec l'application [The Unarchiver](https://theunarchiver.com/). Sous Windows, vous pouvez utiliser [7-Zip](https://7-zip.fr.softonic.com/) ou [WinRAR](https://www.win-rar.com/).


Puis,

- sélectionner l'image spotnik-1.9_opi.img
- cliquer sur le bouton "Restore Backup"

Patientez. Par défaut, la carte sera éjectée à l'issue de la copie.

Vous pouvez alors l'insérer dans le logement prévu à cet effet, situé sous la carte Orange Pi Zero.

## Premier démarrage

Vous pouvez maintenant raccorder votre Orange Pi Zero à un câble RJ45 et l'alimenter via un câble micro USB. L'Orange Pi Zero va booter, pour la première fois !

## Adresse IP

À l'aide d'un scanner de réseau local, relevé l'IP du Spotnik. J'ai utilisé [LanScan](https://www.iwaxx.com/lanscan/). Vous pouvez aussi utiliser l'application [Angry IP Scanner](https://angryip.org/download/) qui est multiplateforme. 

Vous pouvez ensuite vous connecter en SSH, via l'application Terminal, en utilisant le login _root_ et le mot de passe _spotnik_. Sous Windows, téléchargez [PuTTy](https://www.putty.org/).

## Extension du système de fichiers

Par défaut, l'image déployée sur la carte micro SD n'utilise pas 100% de l'espace disponible. La commande `df -h` vous permettra de vérifier qu'environ 1,5G sont occupés pour environ 1,9G d'espace disque, alors que, par exemple, la carte micro SD fait 8G.

```
Sys. de fichiers Taille Utilisé Dispo Uti% Monté sur
/dev/mmcblk0p1     1,9G    1,5G  387M  80% /
udev                10M       0   10M   0% /dev
tmpfs               99M    4,5M   95M   5% /run
tmpfs              248M       0  248M   0% /dev/shm
tmpfs              5,0M    4,0K  5,0M   1% /run/lock
tmpfs              248M       0  248M   0% /sys/fs/cgroup
tmpfs              248M       0  248M   0% /var/spool/svxlink
tmpfs              248M     24K  248M   1% /tmp
tmpfs               50M       0   50M   0% /run/user/0
```

Une première opération consiste à étendre le système de fichiers afin qu'il utilise l'ensemble de l'espace. Pour ce faire, une fois connecté en SSH sur le Spotnik, procédez de la façon suivante:

- exécuter la commande `/etc/init.d/resize2fs start`
- patienter 2 ou 3 minutes
- rebooter
- exécuter de nouveau la commande `/etc/init.d/resize2fs start`

Voilà, la commande `df -h` vous permettra de vérifier que désormais, l'ensemble de l'espace disponible sur la carte micro SD est utilisable.

```
Sys. de fichiers Taille Utilisé Dispo Uti% Monté sur
/dev/mmcblk0p1     7,1G    1,5G  5,6G  21% /
udev                10M       0   10M   0% /dev
tmpfs               99M    4,5M   95M   5% /run
tmpfs              248M       0  248M   0% /dev/shm
tmpfs              5,0M    4,0K  5,0M   1% /run/lock
tmpfs              248M       0  248M   0% /sys/fs/cgroup
tmpfs              248M       0  248M   0% /var/spool/svxlink
tmpfs              248M     24K  248M   1% /tmp
tmpfs               50M       0   50M   0% /run/user/0
```


## FAQ

### Après paramétrage depuis le GUI d'un Spotnik Delta, le CTCSS n'est pas fonctionnel

Pour la programmation du SA818, préférer l'utilisation d'un script Shell plutôt que le GUI. 

```
cd /usr/local/sbin
cp 818cli-prog 818-f4hwn
```

Editer le fichier, en changeant la fréquence ligne 15.

```
diff 818cli-prog 818-f4hwn
15c15
<    group = '432.9750,432.9750,0002,5,0002'
---
>    group = '434.9750,434.9750,0002,5,0002'
```

Attention au code CTCSS et voir la table de correspondance dans le fichier README.txt

Et rebootez.

### Après basculement sur le salon TEC, le Spotnik retourne sur le salon RRF au bout de 20 minutes

Editer la ligne 40 du fichier ```/etc/spotnik/restart.tec``` comme ceci,

```
< sleep 20m
----
> sleep 120m
```

Et voilà, on restera 2 heures (120 minutes) sur le salon TEC avant de rebasculer sur le salon RRF.

### Comment vérifier et régler mon audio ?

Plusieurs moyens de contrôle et outils sont à votre disposition. 

Un bon réflexe est de commencer par basculer le link sur le Perroquet. Vous pouvez le faire, soit via la commande DTMF (95#), soit via l'outil ```spot``` en CLI (menu 8).

En mode Perroquet, vous allez pouvoir vous écouter.

C'est le bon moment de jouer sur les paramètres:

- de votre Transceiver (en particulier, le gain micro),
- de votre Spotnik.

Concernant les réglages du Spotnik, toujours depuis l'outil ```spot``` en CLI, vous allez pouvoir jouer avec Alsamixer (menu 11).

Ensuite, vous pouvez aussi vous écouter sur le Dashboard. C'est également un bon moyen de contrôler la qualité de sa modulation. Attention, il est préférable de basculer sur le salon Technique afin de ne pas perturber le RRF avec vos tests. Il est inutile d'en faire profiter l'ensemble du réseau ;) 

Vous pouvez rejoindre le salon Technique soit via la commande DTMF (98#), soit via l'outil ```spot``` en CLI (menu 3). Passez en émission et écoutez vous sur le Dashboard du [salon Technique](http://rrf.f5nlg.ovh:82/). Si d'autres Radio Amateurs sont présents sur le salon Technique, ils pourront également vous faire un report.  

Pour finir, il existe un autre excellent moyen de s'écouter sur le RRF en utilisant le [WebSDR du F4KJI](http://www.f4kji.fr/index6m.html) du Radio Club de Strasbourg. Sur 50.520 MHz, vous pourrez écouter le RRF et vous écouter également.

### Comment désactiver les commandes DTMF

Editer le fichier ```/etc/spotnik/svxlink.cfg``` et modifier le paramètre ```DTMF_DEC_TYPE```. Par défaut, il est initialisé avec la valeur ```INTERNAL```. Il suffit de changer la valeur en ```NONE```.

```
< DTMF_DEC_TYPE=INTERNAL
----
> DTMF_DEC_TYPE=NONE
```
### Comment modifier le fuseau horaire

Par défaut, votre Spotnik est réglé sur l'heure Francaise. Il suffit de modifier le ```/etc/localtime```. Par exemple, pour la Guadeloupe:

```cp /usr/share/zoneinfo/America/Guadeloupe /etc/localtime```

### Comment démarrer l'interface Web

Par les commandes suivantes:

```
cd /opt/spotnik/gui
make start
```

Normalement, cette opération est réalisée au démarrage de votre Spotnik, via le fichier ```/etc/rc.local```.



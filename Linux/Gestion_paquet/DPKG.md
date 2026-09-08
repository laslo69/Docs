L’outil _Debian Package_ (`dpkg`) est l’utilitaire de base pour installer, configurer, maintenir et supprimer des paquets logiciels sur les systèmes basés sur Debian. L’opération la plus simple consiste à installer un paquet `.deb`

`dpkg` possède des base de données sur les paquets installé, dans `/var/lib/dpkg`

La base de donnée `/var/lib/dpkg/status` permet de voir, dans les paquets installé, leur états actuels

## Installer un paquet

La plupart du temps, un paquet pourra dépendre d’autres paquets pour fonctionner comme prévu. Par exemple, un éditeur d’images pourra avoir besoin de bibliothèques pour ouvrir des fichiers JPEG, ou un autre programme pourra avoir besoin d’un widget toolkit comme Qt ou GTK pour son interface utilisateur

`dpkg` va vérifier si ces dépendances sont installées sur votre système, et il ne pourra pas installer le paquet si elles ne sont pas présentes

Dans ce cas, `dpkg` affichera la liste des paquets manquants. Cependant, il ne peut pas résoudre les dépendances par lui-même

Avant d’installer un paquet, `dpkg` va vérifier si une version précédente existe déjà sur le système. Si c’est le cas, le paquet sera mis à niveau vers la nouvelle version. Autrement, une nouvelle copie sera installée

`dpkg -i` et `dpkg --install` ont le même effet

```bash
dpkg -i openshot-qt_2.4.3+dfsg1-1_all.deb
(Reading database ... 269630 files and directories currently installed.)
Preparing to unpack openshot-qt_2.4.3+dfsg1-1_all.deb ...
Unpacking openshot-qt (2.4.3+dfsg1-1) over (2.4.3+dfsg1-1) ...
**dpkg: dependency problems prevent configuration of openshot-qt:**
 openshot-qt depends on fonts-cantarell; however:
  Package fonts-cantarell is not installed.
 openshot-qt depends on python3-openshot; however:
  Package python3-openshot is not installed.
 openshot-qt depends on python3-pyqt5; however:
  Package python3-pyqt5 is not installed.
 openshot-qt depends on python3-pyqt5.qtsvg; however:
  Package python3-pyqt5.qtsvg is not installed.
 openshot-qt depends on python3-pyqt5.qtwebkit; however:
  Package python3-pyqt5.qtwebkit is not installed.
 openshot-qt depends on python3-zmq; however:
  Package python3-zmq is not installed.

**dpkg: error processing package openshot-qt (--install):**
 dependency problems - leaving unconfigured
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for gnome-menus (3.32.0-1ubuntu1) ...
Processing triggers for desktop-file-utils (0.23-4ubuntu1) ...
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for man-db (2.8.5-2) ...
Errors were encountered while processing:
 openshot-qt
```

Certaines dépendances sont absentes comme indiqué , OpenShot dépend des paquets 

- `fonts-cantarell`
- `python3-openshot`
- `python3-pyqt5`
- `python3-pyqt5.qtsvg`
- `python3-pyqt5.qtwebkit`
- `python3-zmq`

Tous ces composants doivent être installés avant que l’installation d’OpenShot puisse réussir

Dans le cas ou, un dossier contient plusieurs fichiers .deb, il est possible de les installer en même temps avec le mode récursif

`dpkg -R /path/to/dir`

## Supprimer paquets

Pour supprimer un paquet, passez le paramètre `-r` à `dpkg`, suivi du nom du paquet, `--remove` est aussi valide. Les fichiers de configuration sont conservé sauf si la commande `-P` ou `--purge` est utilisé

L’opération de suppression effectue également un contrôle des dépendances, et un paquet ne peut être supprimé que si tous les autres paquets qui en dépendent sont également supprimés

```bash
# dpkg -r unrar
(Reading database ... 269630 files and directories currently installed.)
Removing unrar (1:5.6.6-2) ...
Processing triggers for man-db (2.8.5-2) ...
```

Si vous essayez de faire `dpkg -r p7zip`, vous obtiendrez un message d’erreur

Vous pouvez forcer `dpkg` à installer ou supprimer un paquet, même si les dépendances ne sont pas respectées, en ajoutant le paramètre `--force`

Cependant, une telle démarche laissera très probablement le paquet installé, voire votre système, dans un état dégradé

```bash
dpkg -r p7zip
dpkg: dependency problems prevent removal of p7zip:
 winetricks depends on p7zip; however:
  Package p7zip is to be removed.
 p7zip-full depends on p7zip (= 16.02+dfsg-6).

dpkg: error processing package p7zip (--remove):
 dependency problems - not removing
Errors were encountered while processing:
 p7zip
```

Lorsqu’un paquet est supprimé, les fichiers de configuration correspondants restent en place sur le système. Si vous voulez supprimer _tout_ ce qui est associé au paquet, utilisez l’option `-P` (_purge_) au lieu de `-r`

```bash
dpkg -P p7zip
```

## Reconfigurer un paquet installé

Lorsqu’un paquet est installé, il y a une étape de configuration appelée post-install où un script est exécuté pour configurer tout ce qui est nécessaire au bon fonctionnement du logiciel, comme les permissions, le placement des fichiers de configuration, etc...

Des questions peuvent également être posées à l’utilisateur afin de définir ses préférences relatives au fonctionnement du logiciel

Parfois, en raison d’un fichier de configuration endommagé ou malformé, vous pouvez souhaiter restaurer les paramètres d’un paquet à son état initial. Ou alors, vous souhaitez peut-être modifier les réponses que vous avez fournies aux questions de configuration initiale. Pour ce faire, invoquez la commande `dpkg-reconfigure` suivie du nom du paquet

Cette commande va sauvegarder les anciens fichiers de configuration, décompresser les nouveaux vers les bons répertoires et exécuter le script _post-install_ fourni par le paquet, comme si le paquet avait été installé pour la première fois.

```bash
dpkg-reconfigure tzdata
```

## Mettre à jour un paquet

Pour mettre à jour un paquet, il faut installer avec la commande `dpkg -i <packet>` le même paquet mais, avec un version supérieur

Normalement, dpkg va :

- Vérifier qu'une ancienne version est déjà installée
- Remplacer et supprime les anciens fichiers binaires
- Conserve généralement vos fichiers de configuration personnalisés
- Il écrit les nouveaux fichiers de la version supérieure


## Lister paquet installé

Pour lister les paquets installé sur le système, `dpkg -l` ou `dpkg --list`.

Les paquets installé commence avec, pour valeur dans la première colonne : `ii`

```bash
dpkg -l
```

Si on veut récupérer uniquement les noms

`dpkg -l | awk '{print $2}'`

## Obtenir informations sur les paquets

Pour obtenir des informations sur un paquet `.deb`, telles que sa version, son architecture, son mainteneur, ses dépendances et autres, utilisez la commande `dpkg` avec le paramètre `-I`, suivi du nom de fichier du paquet à inspecter

```bash
dpkg -I /root/PacketTracer_9.01_64bits.deb 
 new Debian package, version 2.0.
 size 400458388 bytes: control archive=1616 bytes.
     371 bytes,     9 lines      control
    2038 bytes,    90 lines   *  postinst             #!/bin/bash
     725 bytes,    45 lines   *  postrm               #!/bin/bash
     742 bytes,    39 lines   *  preinst              #!/bin/sh
     123 bytes,     8 lines   *  prerm                #!/bin/bash
 Package: PacketTracer
 Version: 9.0.1
 Section: base
 Priority: optional
 Architecture: amd64
 Depends:coreutils,libc-bin,libfuse2,libzstd1,libpcre2-dev,sudo,xdg-utils,gtk-update-icon-cache,libc6 (>= 2.31),libstdc++6 (>= 9.3.0),bash (>=4.4)
 Maintainer: Packet Tracer Support <packettracersupport@external.cisco.com>
 Description: Cisco PacketTracer 9.0.1 installation package
```

## Afficher les paquets installés et leur contenu

Pour obtenir une liste de tous les paquets installés sur votre système, utiliser l’option `--get-selections`, comme dans `dpkg --get-selections`

Vous pouvez également obtenir une liste de tous les fichiers installés par un paquet spécifique en passant le paramètre `-L NOMDUPAQUET` à `dpkg`, `dpkg -listfiles` est aussi valide

```bash
dpkg -L unzip 
/.
/usr
/usr/bin
/usr/bin/funzip
/usr/bin/unzip
/usr/bin/unzipsfx
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/lib
/usr/lib/mime
/usr/lib/mime/packages
/usr/lib/mime/packages/unzip
/usr/share
/usr/share/doc
/usr/share/doc/unzip
/usr/share/doc/unzip/BUGS
/usr/share/doc/unzip/History.600.gz
/usr/share/doc/unzip/ToDo
/usr/share/doc/unzip/changelog.Debian.gz
/usr/share/doc/unzip/copyright
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/funzip.1.gz
/usr/share/man/man1/unzip.1.gz
/usr/share/man/man1/unzipsfx.1.gz
/usr/share/man/man1/zipgrep.1.gz
/usr/share/man/man1/zipinfo.1.gz
/usr/share/doc/unzip/changelog.gz
```

## Savoir à quel paquet appartient un fichier

Il est parfois nécessaire de savoir à quel paquet appartient un fichier donné dans votre système. Pour ce faire, vous pouvez utiliser l’outil `dpkg-query` suivi du paramètre `-S` et du chemin d’accès au fichier en question.

`dpkg --search <path/to/bin` est aussi valide

```bash
dpkg-query -S /usr/bin/uname 
coreutils: /usr/bin/uname
```

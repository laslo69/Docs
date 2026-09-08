Rechercher un paquet
Savoir quel paquet fournis un fichier

`RPM` est un gestionnaire de paquet pour la famille de distribution Red hat, qui comme `dpkg`, ne gère pas les dépendances.

les base de données concernant les paquets `rpm` est à `/var/lib/rpm`

## Installer, mettre à jour et supprimer des paquets

base de donnée des logiciels installé dans /var/lib/rpm

L’opération la plus basique consiste à installer un paquet

```bash
rpm -i PAQUET
```

Où `PAQUET` est le nom du paquet `.rpm` que vous souhaitez installer.

La plupart du temps, un paquet dépendra d’autres paquets pour fonctionner comme prévu. À titre d’exemple, un éditeur d’images pourra recourir à certaines bibliothèques pour gérer les fichiers JPG, ou un logiciel pourra avoir besoin d’un kit de développement de widgets comme Qt ou GTK pour son interface utilisateur

`rpm` va vérifier si ces dépendances sont installées sur votre système, et va échouer à installer le paquet dans le cas contraire

Dans ce cas, `rpm` affichera la liste des composants manquants. Cependant, il n’est pas capable de résoudre les dépendances par lui-même

Dans l’exemple ci-dessous, l’utilisateur a essayé d’installer un paquet pour l’éditeur d’images GIMP, mais certaines dépendances étaient manquantes

```bash
rpm -i gimp-2.8.22-1.el7.x86-64.rpm
error: Failed dependencies:
	babl(x86-64) >= 0.1.10 is needed by gimp-2:2.8.22-1.el7.x86_64
	gegl(x86-64) >= 0.2.0 is needed by gimp-2:2.8.22-1.el7.x86_64
	gimp-libs(x86-64) = 2:2.8.22-1.el7 is needed by gimp-2:2.8.22-1.el7.x86_64
	libbabl-0.1.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgegl-0.2.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimp-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpbase-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpcolor-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpconfig-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpmath-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpmodule-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpthumb-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpui-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpwidgets-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libmng.so.1()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libwmf-0.2.so.7()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libwmflite-0.2.so.7()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
```

C’est à l’utilisateur de trouver les paquets `.rpm` avec les dépendances correspondantes et de les installer

S’il existe une version précédente d’un paquet sur le système, vous pouvez passer à une version plus récente en utilisant l’option `-U` pour la mettre à niveau :

```bash
rpm -U PAQUET
```

`rpm -U` suit une logique 

- Recherche si le paquet est déja installé, si oui il le met à jour, sinon il l'installe
- Supprime les anciens fichiers
- Fichiers de config sauvegardé avec extension `.rpmsave`

`-F` permet de mettre à jour un paquet SEULEMENT si il est installé

Dans les deux cas, vous pouvez ajouter l’option `-v` pour obtenir un affichage détaillé (plus d’informations sont fournies pendant l’installation) et `-h` pour obtenir des signes dièse (`#`) qui s’affichent en guise de barre de progression

Plusieurs options peuvent être combinées, de sorte que `rpm -i -v -h` est identique à `rpm -ivh`

Pour supprimer un paquet installé, passez l’option `-e` ( “erase” ) à `rpm`, suivie du nom du paquet que vous souhaitez supprimer, rpm ne gère pas les dépendances lors de l'installation et de la suppression

```bash
rpm -e wget
```

Si un paquet installé dépend du paquet en cours de suppression, vous obtiendrez un message d’erreur

```bash
rpm -e unzip
error: Failed dependencies:
	/usr/bin/unzip is needed by (installed) file-roller-3.28.1-2.el7.x86_64
```

Pour venir à bout de l’opération, vous devrez d’abord supprimer les paquets qui dépendent de celui que vous souhaitez supprimer (dans l’exemple ci-dessus, `file-roller`). Vous pouvez fournir plusieurs noms de paquets en argument à `rpm -e` pour supprimer plusieurs paquets à la fois

## Afficher la liste des paquets installés

Pour obtenir une liste de tous les paquets installés sur votre système, utilisez la commande `rpm -qa` (comme dans “query all”)

```bash
rpm -qa
selinux-policy-3.13.1-229.el7.noarch
pciutils-libs-3.5.1-3.el7.x86_64
redhat-menus-12.0.2-8.el7.noarch
grubby-8.28-25.el7.x86_64
hunspell-en-0.20121024-6.el7.noarch
dejavu-fonts-common-2.33-6.el7.noarch
xorg-x11-drv-dummy-0.3.7-1.el7.1.x86_64
libevdev-1.5.6-1.el7.x86_64
[...]
```

## Obtenir des informations sur les paquets

Pour obtenir des informations sur un paquet _installé_, comme son numéro de version, son architecture, sa date d’installation, le nom du mainteneur du paquet, une description sommaire, etc., utilisez `rpm` avec l’option `-qi` (comme “query info”), suivie du nom du paquet

```bash
rpm -qi unzip
Name        : unzip
Version     : 6.0
Release     : 19.el7
Architecture: x86_64
Install Date: Sun 25 Aug 2019 05:14:39 PM EDT
Group       : Applications/Archiving
Size        : 373986
License     : BSD
Signature   : RSA/SHA256, Wed 25 Apr 2018 07:50:02 AM EDT, Key ID 24c6a8a7f4a80eb5
Source RPM  : unzip-6.0-19.el7.src.rpm
Build Date  : Wed 11 Apr 2018 01:24:53 AM EDT
Build Host  : x86-01.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.info-zip.org/UnZip.html
Summary     : A utility for unpacking zip files
Description :
The unzip utility is used to list, test, or extract files from a zip
archive. Zip archives are commonly found on MS-DOS systems. The zip
utility, included in the zip package, creates zip archives. Zip and
unzip are both compatible with archives created by PKWARE(R)'s PKZIP
for MS-DOS, but the programs' options and default behaviors do differ
in some respects.

Install the unzip package if you need to list, test or extract files from
a zip archive.
```

Pour obtenir une liste des fichiers contenus dans un paquet _installé_, utilisez l’option `-ql` (comme “query list”) suivie du nom du paquet

```bash
rpm -ql unzip

/usr/bin/funzip
/usr/bin/unzip
/usr/bin/unzipsfx
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/share/doc/unzip-6.0
/usr/share/doc/unzip-6.0/BUGS
/usr/share/doc/unzip-6.0/LICENSE
/usr/share/doc/unzip-6.0/README
/usr/share/man/man1/funzip.1.gz
/usr/share/man/man1/unzip.1.gz
/usr/share/man/man1/unzipsfx.1.gz
/usr/share/man/man1/zipgrep.1.gz
/usr/share/man/man1/zipinfo.1.gz
```

Si vous souhaitez obtenir des informations ou une liste de fichiers d’un paquet qui n’a _pas_ encore été installé, ajoutez simplement l’option `-p` aux commandes ci-dessus, suivie du nom du fichier RPM (`FICHIER`)

Donc `rpm -qi PAQUET` devient `rpm -qip FICHIER`, et `rpm -ql PAQUET` devient `rpm -qlp FICHIER`, comme indiqué ci-dessous.

```bash
rpm -qip atom.x86_64.rpm
Name        : atom
Version     : 1.40.0
Release     : 0.1
Architecture: x86_64
Install Date: (not installed)
Group       : Unspecified
Size        : 570783704
License     : MIT
Signature   : (none)
Source RPM  : atom-1.40.0-0.1.src.rpm
Build Date  : sex 09 ago 2019 12:36:31 -03
Build Host  : b01bbeaf3a88
Relocations : /usr
URL         : https://atom.io/
Summary     : A hackable text editor for the 21st Century.
Description :
A hackable text editor for the 21st Century.

rpm -qlp atom.x86_64.rpm

/usr/bin/apm
/usr/bin/atom
/usr/share/applications/atom.desktop
/usr/share/atom
/usr/share/atom/LICENSE
/usr/share/atom/LICENSES.chromium.html
/usr/share/atom/atom
/usr/share/atom/atom.png
/usr/share/atom/blink_image_resources_200_percent.pak
/usr/share/atom/content_resources_200_percent.pak
/usr/share/atom/content_shell.pak
```

## Savoir à quel paquet appartient un fichier

Pour savoir quel paquet installé possède un fichier, utilisez l’option `-qf` (pensez à “query file”) suivie du chemin complet du fichier

```bash
rpm -qf /usr/bin/unzip

unzip-6.0-19.el7.x86_64
```

## Voir les dépendances

Lors de la commande `rpm -q`, si ont ajoute le paramètre `--requires`, une liste de librairie apparait.

Ces librairies sont celles qui sont nécessaires pour l'utilisation du paquet

```bash
rpm -q --requires nmap
libc.so.6()(64bit)
libc.so.6(GLIBC_2.11)(64bit)
libc.so.6(GLIBC_2.14)(64bit)
libc.so.6(GLIBC_2.15)(64bit)
libc.so.6(GLIBC_2.2.5)(64bit)
libc.so.6(GLIBC_2.3)(64bit)
libc.so.6(GLIBC_2.3.2)(64bit)
libc.so.6(GLIBC_2.3.4)(64bit)
libc.so.6(GLIBC_2.33)(64bit)
libc.so.6(GLIBC_2.34)(64bit)
libc.so.6(GLIBC_2.38)(64bit)
libc.so.6(GLIBC_2.4)(64bit)
libc.so.6(GLIBC_2.7)(64bit)
libc.so.6(GLIBC_2.8)(64bit)
libc.so.6(GLIBC_ABI_DT_RELR)(64bit)
libcrypto.so.3()(64bit)
libcrypto.so.3(OPENSSL_3.0.0)(64bit)
libgcc_s.so.1()(64bit)
libgcc_s.so.1(GCC_3.0)(64bit)
libgcc_s.so.1(GCC_3.3.1)(64bit)
libm.so.6()(64bit)
libm.so.6(GLIBC_2.2.5)(64bit)
libm.so.6(GLIBC_2.29)(64bit)
libm.so.6(GLIBC_2.38)(64bit)
libpcap.so.1()(64bit)
libpcre2-8.so.0()(64bit)
libssl.so.3()(64bit)
libssl.so.3(OPENSSL_3.0.0)(64bit)
libstdc++.so.6()(64bit)
libstdc++.so.6(CXXABI_1.3)(64bit)
libstdc++.so.6(CXXABI_1.3.8)(64bit)
libstdc++.so.6(CXXABI_1.3.9)(64bit)
libstdc++.so.6(GLIBCXX_3.4)(64bit)
libstdc++.so.6(GLIBCXX_3.4.11)(64bit)
libstdc++.so.6(GLIBCXX_3.4.15)(64bit)
libstdc++.so.6(GLIBCXX_3.4.20)(64bit)
libstdc++.so.6(GLIBCXX_3.4.21)(64bit)
libstdc++.so.6(GLIBCXX_3.4.29)(64bit)
libstdc++.so.6(GLIBCXX_3.4.30)(64bit)
libstdc++.so.6(GLIBCXX_3.4.9)(64bit)
libz.so.1()(64bit)
nmap-ncat = 4:7.92-5.el10
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(PayloadIsZstd) <= 5.4.18-1
rtld(GNU_HASH)
```

L'option `--provides` affiche ce que le paquet installe

```bash
rpm -q --provides nmap
nmap = 4:7.92-5.el10
nmap(x86-64) = 4:7.92-5.el10
```

Le paramètre `--changelog` permet de fournir un historique du paquet

## rpm2cpio

`rpm2cpio` est une commande qui sert à convertir un paquet `.rpm` en `.cpio`

Permet de récupérer le contenu d'un paquet RPM sur son disque sans l'installer via le gestionnaire de paquets du système, inspecter son contenu et de récupérer un fichier isolé

```bash
`rpm2cpio paquet.rpm | cpio -idmv`
```

- `-i` : extrait les fichiers
- `-d` : crée les répertoires si nécessaire
- `-m` : conserve les dates de modification
- `-v` : affiche les détails en mode verbose

autre options intéressante :

- `-c` Créer une archive à partir d'une liste de fichier dans l'input standard
- `-u` remplace les fichiers inconditionnelement

Ce format est limité aux fichiers de moins de 4 Go et tend à être remplacé par `rpm2archive` sur les systèmes récents


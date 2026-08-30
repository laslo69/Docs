APT ( Advanced Package Tool ) est un système de gestion des paquets qui comprend un ensemble d’outils et simplifie considérablement l’installation, la mise à jour, la suppression et la gestion des paquets. APT offre des fonctionnalités telles que la recherche avancée et la résolution automatique des dépendances

APT n’est pas un "remplacement" de `dpkg`, il faut l’imaginer comme un "frontal" qui facilite les opérations et vient combler les lacunes des fonctionnalités de `dpkg`, comme la résolution des dépendances

APT travaille de concert avec les dépôts de logiciels qui contiennent les paquets disponibles à l’installation. Ces dépôts peuvent être un serveur local ou distant, ou (moins courant) un disque CD-ROM

Les commandes `apt-get` et `apt` ont le même effet

## Cache local

Lorsque vous installez ou mettez à jour un paquet, le fichier `.deb` correspondant est téléchargé dans un répertoire de cache local avant que le paquet ne soit installé

Par défaut, ce répertoire est `/var/cache/apt/archives`. Les fichiers partiellement téléchargés sont copiés dans `/var/cache/apt/archives/partial/`

Au fur et à mesure que vous installez et mettez à jour des paquets, le répertoire de cache peut devenir assez volumineux

Pour récupérer de l’espace, vous pouvez vider le cache en utilisant la commande `apt-get clean` ou `apt clean`. Cette opération supprime le contenu des répertoires `/var/cache/apt/archives` et `/var/cache/apt/archives/partial/`

## Actualiser les dépots

Avant d’installer ou de mettre à jour un logiciel avec APT, il est recommandé d’actualiser l’index des paquets afin de récupérer les informations sur les nouveaux paquets et les paquets mis à jour

`apt-get update` ou `apt update`

```bash
apt update 
Hit:1 http://deb.debian.org/debian trixie InRelease
Get:2 http://security.debian.org/debian-security trixie-security InRelease [43.4 kB]
Get:3 http://deb.debian.org/debian trixie-updates InRelease [47.3 kB]                      
Get:4 http://security.debian.org/debian-security trixie-security/main Sources [226 kB]   
Get:5 http://security.debian.org/debian-security trixie-security/main i386 Packages [229 kB]
Get:6 http://security.debian.org/debian-security trixie-security/main amd64 Packages [251 kB]
Get:7 http://security.debian.org/debian-security trixie-security/main Translation-en [152 kB]
Fetched 953 kB in 0s (2,695 kB/s)                                
3 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

## Mise à jour paquets

APT peut être utilisé pour mettre à jour automatiquement tous les paquets installés vers les dernières versions disponibles dans les dépôts. Cela se fait à l’aide de la commande `apt-get upgrade` ou `apt upgrade`. Avant de l’invoquer, pensez à mettre à jour l’index des paquets avec `apt-get update`

```bash
apt upgrade 
The following packages were automatically installed and are no longer required:
  linux-image-6.12.100+deb13-amd64  python3-proton-vpn-local-agent
Use 'apt autoremove' to remove them.

Upgrading:
  linux-doc-6.12  linux-image-amd64  linux-libc-dev

Installing dependencies:
  linux-image-6.12.107+deb13-amd64

Summary:
  Upgrading: 3, Installing: 1, Removing: 0, Not Upgrading: 0
  Download size: 150 MB
  Space needed: 189 MB / 396 GB available
```

La liste récapitulative en bas de l’affichage indique le nombre de paquets qui seront mis à niveau, le nombre de paquets qui seront installés, retirés ou conservés, la taille totale du téléchargement et l’espace disque supplémentaire nécessaire pour mener à bien l’opération

## Installer et supprimer des paquets

Pour installer un paquet, utilisation de la commande `apt-get install` ou `apt install` suivi du nom du paquet à installer

le paramètre `-s` peut simuler l'opération

```bash
root@debian:~ apt install gnumeric
The following packages were automatically installed and are no longer required:
  linux-image-6.12.100+deb13-amd64  python3-proton-vpn-local-agent
Use 'apt autoremove' to remove them.

Installing:
  gnumeric

Installing dependencies:
  docbook-xml          libevdocument3-4t64        libyelp0
  evince               libevview3-3t64            lp-solve
  evince-common        libgnome-desktop-3-20t64   pxlib1
  fonts-dejavu         libgoffice-0.10-10-common  sgml-data
  fonts-dejavu-extra   libgoffice-0.10-10t64      yelp
  gnome-desktop3-data  libgsf-1-114               yelp-xsl
  gnumeric-common      libgsf-1-common
  gnumeric-doc         libxkbregistry0

Suggested packages:
  docbook           docbook-xsl             libgsf-1-dev  opensp
  docbook-defguide  nautilus-sendto         perlsgml
  docbook-dsssl     gnumeric-plugins-extra  w3-recs

Summary:
  Upgrading: 0, Installing: 23, Removing: 0, Not Upgrading: 3
  Download size: 24.0 MB
  Space needed: 85.2 MB / 396 GB available

Continue? [Y/n] 
```

Pour supprimer un paquet, `apt-get remove <paquet>` ou `apt remove <paquet>`

```bash
apt remove tree 
The following packages were automatically installed and are no longer required:
  linux-image-6.12.100+deb13-amd64  python3-proton-vpn-local-agent
Use 'apt autoremove' to remove them.

REMOVING:
  tree

Summary:
  Upgrading: 0, Installing: 0, Removing: 1, Not Upgrading: 3
  Freed space: 132 kB
```

Sachez que lors de l’installation ou de la suppression de paquets, APT va effectuer une résolution automatique des dépendances

Cela signifie que tout paquet supplémentaire requis par le paquet que vous installez sera également installé, et que les paquets qui dépendent du paquet que vous supprimez seront également supprimés. APT va toujours afficher ce qui va être installé ou supprimé avant de vous demander si vous voulez continuer

Notez que lorsqu’un paquet est retiré, les fichiers de configuration correspondants sont laissés en place sur le système. Pour supprimer le paquet _et_ tout fichier de configuration, utilisez le paramètre `purge` au lieu de `remove` ou le paramètre `remove` avec l’option `--purge

```bash
apt purge <paquet>
ou
apt remove --purge <paquet>
```

## Rechercher des paquets

L’outil `apt-cache` peut être utilisé pour effectuer des opérations sur l’index des paquets, telles que la recherche d’un paquet spécifique ou la liste des paquets qui contiennent un fichier particulier

Pour effectuer une recherche, invoquez `apt-cache search` ou `apt search` suivi de la chaîne de caractères à rechercher. Le résultat sera une liste de chaque paquet qui contient le motif recherché, soit dans son nom de paquet, soit dans sa description ou dans les fichiers fournis

```bash
apt search p7zip

liblzma-dev/stable 5.8.1-1+deb13u1 amd64
  XZ-format compression library - development files

liblzma5/stable,now 5.8.1-1+deb13u1 amd64 [installed]
  XZ-format compression library

p7zip/stable,stable 16.02+transitional.1 all
  transitional package

p7zip-full/stable,stable 16.02+transitional.1 all
  transitional package

python3-bcj/stable 1.0.3+ds-1+b1 amd64
  BCJ (Branch-Call-Jump) filter for Python

python3-ppmd/stable 0.5.0-4 amd64
  PPMd compression/decompression library
```

Dans l’exemple ci-dessous, l’entrée `liblzma5 - XZ-format compression library` ne semble pas correspondre au motif recherché. Cependant, si nous affichons les informations complètes avec la description pour le paquet en invoquant le paramètre `show`, nous retrouvons la chaîne de caractères recherchée. `apt show` ou `apt-cache show`

```bash
apt show liblzma5

Package: liblzma5
Version: 5.8.1-1+deb13u1
Priority: optional
Section: libs
Source: xz-utils
Maintainer: Sebastian Andrzej Siewior <sebastian@breakpoint.cc>
Installed-Size: 445 kB
Depends: libc6 (>= 2.34)
Breaks: liblzma2 (<< 5.1.1alpha+20110809-3~)
Homepage: https://tukaani.org/xz/
Tag: role::shared-lib
Download-Size: 309 kB
APT-Manual-Installed: yes
APT-Sources: http://deb.debian.org/debian trixie/main amd64 Packages
Description: XZ-format compression library
 XZ is the successor to the Lempel-Ziv/Markov-chain Algorithm
 compression format, which provides memory-hungry but powerful
 compression (often better than bzip2) and fast, easy decompression.
 .
 The native format of liblzma is XZ; it also supports raw (headerless)
 streams and the older LZMA format used by lzma. (For 7-Zip's related
 format, use the p7zip package instead.)
```

## Afficher le contenu d’un paquet et rechercher des fichiers

`apt-file` peut être utilisé pour effectuer d’autres opérations dans l’index des paquets, comme afficher le contenu d’un paquet ou repérer un paquet qui contient un fichier donné, cet outil peut ne pas être installé par défaut sur votre système

Après l’installation, vous devrez mettre à jour le cache de paquets utilisé pour `apt-file`

```bash
apt-file update
```

Pour afficher le contenu d’un paquet, utilisez `apt-file list` suivi du nom du paquet

```bash
apt-file list unrar

unrar: /usr/bin/unrar-nonfree
unrar: /usr/share/doc/unrar/changelog.Debian.gz
unrar: /usr/share/doc/unrar/copyright
unrar: /usr/share/man/man1/unrar-nonfree.1.gz
```

Vous pouvez rechercher un fichier dans tous les paquets en utilisant le paramètre `search` suivi du nom du fichier

```bash
apt-file search libSDL2.so
libsdl2-dev: /usr/lib/x86_64-linux-gnu/libSDL2.so
```

## Reparer dependances cassees

Il est possible d’avoir des "dépendances cassées" sur un système. Cela signifie qu’un ou plusieurs paquets installés dépendent d’autres paquets qui n’ont pas été installés ou qui ne sont plus présents

Cela peut se produire suite à une erreur APT ou à cause d’un paquet installé manuellement

Cette opération vise à réparer  les paquets cassés en installant les dépendances manquantes, afin de s’assurer que tous les paquets soient à nouveau cohérents.

```bash
apt-get install -f
ou
apt install -f
```

## Mise à jour distribution

La commande `apt dist-upgrade` ou `apt-get dist-upgrade` permet de faire une mise à jour complète de l'OS, généralement utilisé pour passer sur une nouvelle version

```bash
apt-get dist-upgrade
ou
apt dist-upgrade
```

## Gérer les dépots

APT utilise une liste de sources pour savoir où récupérer les paquets. Cette liste est conservée dans le fichier `sources.list` situé dans le répertoire `/etc/apt`

Ce fichier peut être édité directement avec un éditeur de texte comme `vi`, `pico` ou `nano`, ou avec des outils graphiques comme `aptitude` ou `synaptic`

exemple d'un sourcelist

```bash
deb http://us.archive.ubuntu.com/ubuntu/ disco main restricted universe multiverse
```

La syntaxe comprend le type d’archive, l’URL, la distribution et un où plusieurs composants

- `Type d’archive` : Un dépôt peut contenir des paquets contenant des logiciels prêts à l’emploi (paquets binaires de type `deb`) ou le code source de ces logiciels (paquets sources de type `deb-src`). L’exemple ci-dessus fournit des paquets binaires.
- `URL` : L’URL du dépôt
- `Distribution` : Le nom (ou nom de code) de la distribution pour laquelle les paquets sont fournis. Un dépôt peut héberger des paquets pour plusieurs distributions. Dans l’exemple ci-dessus, `disco` est le nom de code pour Ubuntu 19.04 _Disco Dingo_
- `Composants` : Chaque composant représente un ensemble de paquets. Ces composants peuvent varier selon les différentes distributions de Linux

Par exemple, sur Ubuntu et ses déclinaisons, on aura
	- `main` : contient des paquets _open source_ officiellement pris en charge
	- `restricted` : contient des logiciels propriétaires officiellement pris en charge, comme les pilotes de cartes graphiques
	- `universe` : contient des logiciels _open source_ maintenus par la communauté
	- `multiverse` : contient des logiciels propriétaires non pris en charge ou autrement protégés par un brevet

Sur Debian, les principaux composants sont :
- `main` : contient les paquets conformes aux principes du logiciel libre selon Debian (DFSG pour _Debian Free Software Guidelines_), qui ne dépendent pas de logiciels extérieurs à ce domaine pour fonctionner. Les paquets inclus ici sont considérés comme faisant partie de la distribution Debian
- `contrib` : contient les paquets conformes aux DFSG, mais qui dépendent d’autres paquets qui ne sont pas dans `main`.
- `non-free` : contient les paquets qui ne sont pas conformes aux DFSG
- `security` : contient les mises à jour de sécurité
- `backports` : contient des versions plus récentes de paquets qui sont dans `main`. Le cycle de développement des versions stables de Debian est assez long (environ deux ans), et cela permet aux utilisateurs d’obtenir les paquets les plus récents sans avoir à modifier le dépôt principal `main`

## Ajout dépot

La deuxième méthode est recommandé pour, l'ajout de dépot autre que ceux fournis par l'organisation derrière la prise en charge de la distribution linux utilisé.

Pour ajouter un nouveau dépôt pour obtenir des paquets, vous pouvez simplement ajouter la ligne correspondante (généralement fournie par le mainteneur du dépôt) à la fin de `sources.list`, enregistrer le fichier et recharger l’index du paquet avec `apt-get update`. Une fois que c’est fait, les paquets du nouveau dépôt seront disponibles à l’installation

Le répertoire `/etc/apt/sources.list.d` vous permet d’ajouter des fichiers avec des dépôts supplémentaires exploitables par APT, et sans modifier le fichier principal `/etc/apt/sources.list`. Il s’agit là de simples fichiers texte avec la même syntaxe que celle décrite ci-dessus et l’extension de fichier `.list`

ajout de : `/etc/apt/sources.list.d/buster-backports.list`

```bash
deb http://deb.debian.org/debian buster-backports main contrib non-free
deb-src http://deb.debian.org/debian buster-backports main contrib non-free
```


`dnf` est l’outil de gestion de paquets utilisé sous Fedora, c’est un fork de `yum`. De ce fait, les commandes et les paramètres se ressemblent beaucoup

`dnf` dispose d’un système d’aide intégré, qui affiche des informations détaillées (comme les paramètres supplémentaires) pour chaque commande, gère les dépendances

## Recherche de paquet

Avant d’installer un paquet, vous devez connaître son nom. Pour ce faire, vous pouvez effectuer une recherche avec `yum search MOTIF`, où `MOTIF` est le nom du paquet recherché. Le résultat est une liste de paquets dont le nom ou le résumé contient le motif de recherche spécifié

```bash
yum search 7zip

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
=========================== N/S matchyutr54ed: 7zip ============================
p7zip-plugins.x86_64 : Additional plugins for p7zip
p7zip.x86_64 : Very high compression ratio file archiver
p7zip-doc.noarch : Manual documentation and contrib directory
p7zip-gui.x86_64 : 7zG - 7-Zip GUI version

  Name and summary matches only, use "search all" for everything.
```

## Installer des paquets

Pour installer un paquet à l’aide de `yum`, utilisez la commande `yum install PAQUET`, où `PAQUET` est le nom du paquet. `yum` va récupérer le paquet et les dépendances correspondantes depuis un dépôt en ligne et installer le tout sur votre système

```bash
yum install p7zip

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
Resolving Dependencies
--> Running transaction check
---> Package p7zip.x86_64 0:16.02-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================
 Package        Arch            Version               Repository     Size
==========================================================================
Installing:
 p7zip          x86_64          16.02-10.el7          epel          604 k

Transaction Summary
==========================================================================
Install  1 Package

Total download size: 604 k
Installed size: 1.7 M
Is this ok [y/d/N]:
```
## Supprimer des paquets

Pour supprimer un paquet installé, utilisez `yum remove PAQUET`, où `PAQUET` est le nom du paquet que vous souhaitez supprimer

## Mettre à jour des paquets

Pour mettre à jour un paquet installé, utilisez `yum update PAQUET`, où `PAQUET` est le nom du paquet que vous souhaitez mettre à jour

Si vous ne fournissez aucun nom de paquet en argument, vous pouvez mettre à jour tous les paquets du système pour lesquels une mise à jour est disponible

```bash
yum update wget

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
Resolving Dependencies
--> Running transaction check
---> Package wget.x86_64 0:1.14-18.el7 will be updated
---> Package wget.x86_64 0:1.14-18.el7_6.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================
 Package     Arch          Version                   Repository      Size
==========================================================================
Updating:
 wget        x86_64        1.14-18.el7_6.1           updates        547 k

Transaction Summary
==========================================================================
Upgrade  1 Package

Total download size: 547 k
Is this ok [y/d/N]:
```

Pour vérifier si une mise à jour est disponible pour un paquet spécifique, utilisez `yum check-update PAQUET`. De manière similaire, si vous omettez le nom du paquet, `yum` vérifiera les mises à jour pour chacun des paquets installés sur le système

## Savoir quel paquet fournit un fichier donné

Dans un exemple précédent, nous avons montré une tentative d’installation de l’éditeur d’images `gimp`, qui a échoué pour cause de dépendances non satisfaites. Certes, `rpm` montre quels fichiers sont manquants, mais il ne liste pas le nom des paquets qui les fournissent

Par exemple, l’une des dépendances manquantes était `libgimpui-2.0.so.0`. Pour voir quel paquet la fournit, vous pouvez utiliser `yum whatprovides` suivi du nom du fichier recherché

```bash
yum whatprovides libgimpui-2.0.so.0

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
2:gimp-libs-2.8.22-1.el7.i686 : GIMP libraries
Repo        : base
Matched from:
Provides    : libgimpui-2.0.so.0
```

## Savoir à quel paquet appartient un fichier

Cela fonctionne également pour les fichiers déjà présents sur votre système. Par exemple, si vous souhaitez savoir d’où vient le fichier `/etc/hosts`, vous pouvez utiliser

```bash
yum whatprovides /etc/hosts

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
setup-2.8.71-10.el7.noarch : A set of system configuration and setup files
Repo        : base
Matched from:
Filename    : /etc/hosts
```

## En savoir plus sur un paquet

Pour obtenir des informations sur un paquet, comme sa version, son architecture, sa description, sa taille et plus encore, utilisez `yum info PAQUET` où `PAQUET` est le nom du paquet pour lequel vous voulez en savoir plus

```bash
yum info firefox

Last metadata expiration check: 0:24:16 ago on Sat 21 Sep 2019 02:39:43 PM -03.
Installed Packages
Name         : firefox
Version      : 69.0.1
Release      : 3.fc30
Architecture : x86_64
Size         : 268 M
Source       : firefox-69.0.1-3.fc30.src.rpm
Repository   : @System
From repo    : updates
Summary      : Mozilla Firefox Web browser
URL          : https://www.mozilla.org/firefox/
License      : MPLv1.1 or GPLv2+ or LGPLv2+
Description  : Mozilla Firefox is an open-source web browser, designed
             : for standards compliance, performance and portability.
```

## Obtenir une liste de tous les paquets installés sur le système

`dnf list --installed`

## Afficher le contenu d’un paquet

`dnf repoquery -l PAQUET`

## Gérer les dépôts DNF

Tout comme `yum` et `zypper`, `dnf` fonctionne avec des dépôts de logiciels (_repos_). Chaque distribution dispose d’une liste de dépôts par défaut, et les administrateurs peuvent ajouter ou supprimer des dépôts en cas de besoin

Pour obtenir une liste de tous les dépôts disponibles, utilisez `dnf repolist`. Pour répertorier uniquement les dépôts activés, ajoutez l’option `--enabled`, et pour afficher uniquement les dépôts désactivés, utilisez l’option `--disabled`

```bash
dnf repolist

Last metadata expiration check: 0:20:09 ago on Sat 21 Sep 2019 02:39:43 PM -03.
repo id                    repo name                                      status
*fedora                    Fedora 30 - x86_64                             56,582
*fedora-modular            Fedora Modular 30 - x86_64                        135
*updates                   Fedora 30 - x86_64 - Updates                   12,774
*updates-modular           Fedora Modular 30 - x86_64 - Updates              145
```

Pour ajouter un dépôt, invoquez `dnf config-manager --add_repo URL`, où `URL` est l’URL complète du dépôt. Pour activer un dépôt, utilisez `dnf config-manager --set-enabled ID_DEPOT`.

De même, pour désactiver un dépôt, utilisez `dnf config-manager --set-disabled ID_DEPOT`. Dans les deux cas, `ID_DEPOT` est l’ID unique du dépôt, que vous pouvez obtenir en invoquant `dnf repolist`. Les dépôts ajoutés sont activés par défaut.

Les dépôts sont stockés dans des fichiers `.repo` dans le répertoire `/etc/yum.repos.d/` et utilisent exactement la même syntaxe que `yum`

## Gérer les dépôts YUM

Pour `yum`, les dépôts de paquets (“repos”) figurent dans le répertoire `/etc/yum.repos.d/`. Chaque dépôt est représenté par un fichier `.repo`, comme `CentOS-Base.repo`

Des dépôts supplémentaires peuvent être ajoutés par l’utilisateur en ajoutant un fichier `.repo` dans le répertoire mentionné ci-dessus, ou à la fin de `/etc/yum.conf`. Ceci étant dit, la procédure recommandée pour ajouter ou gérer les dépôts de paquets passe par l’outil `yum-config-manager`

Pour ajouter un dépôt, utilisez le paramètre `--add-repo` suivi de l’URL d’un fichier `.repo`

```bash
yum-config-manager --add-repo

https://rpms.remirepo.net/enterprise.remi.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://rpms.remirepo.net/enterprise/remi.repo
grabbing file https://rpms.remirepo.net/enterprise/remi.repo to /etc/yum.repos.d/remi.repo
repo saved to /etc/yum.repos.d/remi.repo
```

Pour obtenir une liste de tous les dépôts disponibles, utilisez `yum repolist all`. Vous obtiendrez un résultat semblable à celui-ci

```bash
yum repolist all

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
repo id                       repo name                    status
updates/7/x86_64              CentOS-7 - Updates           enabled:  2,500
updates-source/7              CentOS-7 - Updates Sources   disabled
```

Les dépôts désactivés (`disabled`) seront ignorés lors de l’installation ou de la mise à jour des logiciels. Pour activer ou désactiver un dépôt, invoquez l’outil `yum-config-manager` en renseignant l’identifiant du dépôt.

Dans l’exemple ci-dessus, l’identifiant du dépôt est indiqué dans la première colonne (`repo id`) de chaque ligne. Utilisez uniquement la partie avant le premier `/`, de sorte que l’identifiant pour le dépôt `CentOS-7 - Updates` est `updates`, et non pas `updates/7/x86_64`.

```bash
yum-config-manager --disable updates
```

La commande ci-dessus va désactiver le dépôt `updates`. Pour le réactiver, utilisez :

```bash
yum-config-manager --enable updates
```

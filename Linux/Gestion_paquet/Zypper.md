
Zypper est le gestionnaire de paquet de la distribution `OpenSUSE`

Zypper conserve une trace des dépots qui ont servi de source d'installation, quand un paquet est mis à jour, le dépot source d'installation est consulté, même si un autre dépot propose une meilleur version

Les fichiers de configuration dépot se trouve dans `/etc/zypp/repos.d` avec une extension .repo, les fichiers de configuration `/etc/zypp/zypp.conf` et `/etc/zypp/zypper.conf`

Zypper peut utiliser des variables comme `$arch`, `$basearch` pour la recherche de paquet dans les dépots

Zypper utilise un système de priorité pour les dépots ( entre 1 et 200 ), plus la priorité est basse, plus le dépot est prioritaire, ce qui permet de régler les conflits si 2 dépots fournissent le même paquet

## Actualiser depots

`zypper` fonctionne avec des dépôts contenant des paquets et des métadonnées. Ces métadonnées doivent être rafraîchies de temps en temps, afin que le gestionnaire soit au courant des derniers paquets disponibles

Pour mettre à jour les dépots : `zypper refresh`

```bash
zypper refresh
Rafraichissement du service 'openSUSE'.
Le dépôt 'https://download.opensuse.org/distribution/leap/16.0/repo/oss/x86_64'
est à jour.
Le dépôt 'repo-openh264 (16.0)' est à jour.                    Le dépôt 'repo-oss (16.0)' est à jour.                                          
Tous les dépôts ont été rafraichis.
```

`zypper` dispose d’une fonctionnalité de rafraîchissement automatique qui peut être activée individuellement pour chaque dépôt. Autrement dit, certains dépôts peuvent être rafraîchis automatiquement avant une requête ou l’installation d’un paquet, alors que d’autres devront être rafraîchis manuellement

Pour ce faire, il faut modifier la configuration du dépot cible

```bash
zypper modifyrepo -f <alias depot>
ou
zypper modifyrepo --refresh <alias depot>
```

Pour enlever le rafraichissement automatique

```bash
zypper modifyrepo -F <alias depot>
ou
zypper modifyrepo --no-refresh <alias depot>
```

## Rechercher un paquet

Pour rechercher un paquet, utilisez le paramètre `search` ou `se` suivie du nom du paquet

```bash
zypper se gnumeric

Loading repository data...
Reading installed packages...

S | Name           | Summary                           | Type
--+----------------+-----------------------------------+--------
  | gnumeric       | Spreadsheet Application           | package
  | gnumeric-devel | Spreadsheet Application           | package
  | gnumeric-doc   | Documentation files for Gnumeric  | package
  | gnumeric-lang  | Translations for package gnumeric | package
```

La commande `search` peut également être utilisée pour obtenir une liste de tous les paquets installés sur le système avec le paramètre `-i` sans nom de paquet

Pour voir si un paquet spécifique est installé, ajoutez le nom du paquet à la fin de la commande, la requête suivante va rechercher parmi les paquets installés tous ceux qui contiennent “firefox” dans leur nom

```bash
zypper se -i firefox
```

Pour rechercher uniquement parmi les paquets non installés, ajoutez l’option `-u` à l’opérateur `se`

```bash
zypper se -u firefox
Rafraichissement du service 'openSUSE'.
Chargement des données du dépôt...
Lecture des paquets installés...

S  | Name                               | Summary                       | Type
---+------------------------------------+-------------------------------+-------
   | firefox-esr-branding-openSUSE      | openSUSE branding of Mozill-> | paquet
   | MozillaFirefox                     | Le navigateur web Mozilla F-> | paquet
   | MozillaFirefox-branding-openSUSE   | openSUSE branding of Mozill-> | paquet
   | MozillaFirefox-branding-upstream   | Upstream branding for Firefox | paquet
   | MozillaFirefox-devel               | Paquet de développement pou-> | paquet
   | MozillaFirefox-translations-common | Traductions communes pour F-> | paquet
   | MozillaFirefox-translations-other  | Traductions supplémentaires-> | paquet
```

## Installer, mettre à jour et supprimer des paquets

Pour installer un paquet logiciel, utilisez le parametre `install` ou `in` suivie du nom du paquet

`zypper` peut également être utilisé pour installer un paquet RPM depuis le disque local tout en essayant de satisfaire ses dépendances avec les paquets en provenance des dépôts. Pour ce faire, il suffit de fournir le chemin complet du paquet au lieu d’un nom de paquet, comme `zypper in /home/john/nouveaupaquet.rpm`

```bash
zypper in unrar

Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following NEW package is going to be installed:
  unrar

1 new package to install.
Overall download size: 141.2 KiB. Already cached: 0 B. After the operation, additional 301.6 KiB will be used.
Continue? [y/n/v/...? shows all options] (y): y
Retrieving package unrar-5.7.5-lp151.1.1.x86_64
                                     (1/1), 141.2 KiB (301.6 KiB unpacked)
Retrieving: unrar-5.7.5-lp151.1.1.x86_64.rpm .......................[done]
Checking for file conflicts: .......................................[done]
(1/1) Installing: unrar-5.7.5-lp151.1.1.x86_64 .....................[done]
```

Pour mettre à jour les paquets installés sur le système, utilisez `zypper update` ou `zypper up`, cette opération va afficher une liste de paquets à installer/mettre à jour avant de vous demander si vous voulez continuer

```bash
zypper update 
Rafraichissement du service 'openSUSE'.
Chargement des données du dépôt...
Lecture des paquets installés...

Les 14 paquets suivants vont être mis à jour :
  dracut gzip hwinfo libarchive13 libhd25 python313-pip qemu-guest-agent rsync
  vim vim-data vim-data-common wget wget-lang xxd

14 paquets à mettre à jour.

Taille du téléchargement de paquet :    21,6 MiB

Modification de la taille d'installation des paquets :
              |      68,4 MiB  requis par les paquets qui seront installés
   257,2 KiB  |  -   68,2 MiB  libérés par les paquets qui seront supprimés

Back-end:  classic_rpmtrans
Continuer ? [o/n/v/...? affiche toutes les options] (o):
```

Si vous souhaitez seulement afficher la liste des mises à jour disponibles sans rien installer, vous pouvez utiliser `zypper list-updates`

```bash
zypper list-updates 
Rafraichissement du service 'openSUSE'.
Chargement des données du dépôt...
Lecture des paquets installés...
Aucune mise à jour trouvée.
```

Pour supprimer un paquet, utilisez la commande `remove` ou `rm` suivie du nom du paquet. Gardez à l’esprit que la suppression d’un paquet entraîne la suppression de tous les autres paquets qui en dépendent.

```bash
zypper rm unrar
Lecture des paquets installés...
'unrar' n'a pas été trouvé parmi les noms de paquets. Essai parmi les capacités.
Résolution des dépendances des paquets...

Le paquet suivant va être SUPPRIMÉ :
  unrar_wrapper

1 paquet à supprimer.

Modification de la taille d'installation des paquets :
              |         0 B    requis par les paquets qui seront installés
   -61,5 KiB  |  -   61,5 KiB  libérés par les paquets qui seront supprimés

Back-end:  classic_rpmtrans
Continuer ? [o/n/v/...? affiche toutes les options] (o):
```

## Security update

Zypper à une commande pour installer uniquement, les mise à jour de sécurité. `zypper patch`

## Upgrade distribution

Pour mettre à jour la distribution, utilisation de la commande `zypper dist-upgrade` ou `zypper dup`

Le paramètre `allow-vendor-change` permet d'autoriser le changement de source pour la mise à jour

## Savoir quel paquet fournit un fichier

Pour savoir quel paquet contient un fichier donné, utilisez la commande `search` suivie de l’option `--provides` et du nom du fichier (ou de son chemin complet)

```bash
zypper se --provides /usr/lib64/libgimpmodule-3.0.so.0
Rafraichissement du service 'openSUSE'.
Chargement des données du dépôt...
Lecture des paquets installés...

S  | Name          | Summary                                        | Type
---+---------------+------------------------------------------------+-------
i  | libgimp-3_0-0 | The GNU Image Manipulation Program - Libraries | paquet
```

Pour être plus précis, `zypper se --provide --match-exact`

Au passage, la commande au dessus est un alias à `zypper what-provides <paquet>`

## Obtenir des informations sur les paquets

Pour voir les métadonnées associées à un paquet, utilisez la commande `zypper info` suivie du nom du paquet

Indiquera le dépôt d’origine, le nom du paquet, la version, l’architecture, le fabricant, la taille installée, s’il est installé ou non, le statut (s’il est à jour), le paquet source ainsi qu’une description

```bash
zypper info gimp
Rafraichissement du service 'openSUSE'.
Chargement des données du dépôt...
Lecture des paquets installés...


Informations sur paquet gimp :
------------------------------
Dépôt                     : https://download.opensuse.org/distribution/leap/16.0/repo/oss/x86_64
Nom                       : gimp
Version                   : 3.0.8-bp160.6.1
Architecture              : x86_64
Fabricant                 : openSUSE
Taille une fois installé  : 45,6 MiB
Installé                  : Oui
État                      : à jour
Paquet source             : gimp-3.0.8-bp160.6.1.src
URL en amont              : https://www.gimp.org/
Résumé                    : Programme de traitement d'images, libre et extensible
Description               : 
    The GIMP is an image composition and editing program, which can be
    used for creating logos and other graphics for Web pages. The GIMP
    offers many tools and filters, and provides a large image
    manipulation toolbox, including channel operations and layers,
    effects, subpixel imaging and antialiasing, and conversions, together
    with multilevel undo. The GIMP offers a scripting facility, but many
    of the included scripts rely on fonts that we cannot distribute.
```


## Contenu dépot

```bash
cat /etc/zypp/repos.d/openSUSE\:repo-non-oss.repo 
# Repository 'openSUSE:repo-non-oss' is maintained by the 'openSUSE' service.
# Manual changes may be overwritten by a service refresh.
# See also 'man zypper', section 'Services'.
[openSUSE:repo-non-oss]
name=repo-non-oss (${releasever})
enabled=1
autorefresh=1
baseurl=http://cdn.opensuse.org/distribution/leap/${releasever}/repo/non-oss/$basearch
type=rpm-md
gpgkey=http://cdn.opensuse.org/distribution/leap/${releasever}/repo/non-oss/$basearch/repodata/repomd.xml.key
service=openSUSE
```

## Gérer les dépôts

`zypper` peut également être utilisé pour gérer les dépôts

Pour voir une liste de tous les dépôts enregistrés sur votre système, utilisez `zypper repos` ou `zypper lr`

`Zypper lr -E` permet de lister uniquement les dépots actif

```bash
zypper repos
Rafraichissement du service 'openSUSE'.
Les priorités des dépôts sont sans effet. Tous les dépôts activés partagent la même priorité.

# | Alias                                | Name   | Enabled | GPG Check | Refresh
--+--------------------------------------+--------+---------+-----------+--------
1 | https-download.opensuse.org-9d7344f2 | http-> | Oui     | (r ) Oui  | Oui
2 | openSUSE:repo-non-oss                | repo-> | Non     | ----      | ----
3 | openSUSE:repo-non-oss-debug          | repo-> | Non     | ----      | ----
4 | openSUSE:repo-openh264               | repo-> | Oui     | (r ) Oui  | Oui
5 | openSUSE:repo-oss                    | repo-> | Oui     | (r ) Oui  | Oui
6 | openSUSE:repo-oss-debug              | repo-> | Non     | ----      | ----
7 | openSUSE:repo-oss-source             | repo-> | Non     | ----      | ----
```

Dans la colonne `Enabled`, notez que certains dépôts sont activés, alors que d’autres ne le sont pas

Pour changer, activer un dépot avec la commande `modifyrepo` suivie de l’option `-e` ( enable ) ou `-d` ( disable ) et l’alias du dépôt en question (qui figure dans la deuxième colonne dans l’affichage ci-dessus). `zypper mr` est un alias à la commande `zypper modifyrepo`

```bash
zypper modifyrepo -e openSUSE\:repo-non-oss
Le dépôt 'openSUSE:repo-non-oss' a été activé avec succès.
```

```bash
zypper modifyrepo -d openSUSE\:repo-non-oss
Le dépôt 'openSUSE:repo-non-oss' a été désactivé avec succès.
```

Pour modifier la priorité d'un dépot

```bash
zypper mr -p 90 packman 
La priorité du dépôt 'packman' a été réglée à 90.
```

## Ajouter et supprimer des dépôts

Pour ajouter un nouveau dépôt logiciel pour `zypper`, utilisez la commande `addrepo` suivie de l’URL et du nom du dépôt, vous pouvez activer le rafraîchissement automatique avec l’option `-f` en même temps

Les dépôts ajoutés sont activés par défaut, mais vous pouvez ajouter et désactiver un dépôt d’une traite en utilisant l’option `-d`

```bash
zypper addrepo -f http://packman.inode.at/suse/openSUSE_Leap_15.1/ packman
Ajout du dépôt 'packman' ..................................................[fait]
Le dépôt 'packman' a été ajouté avec succès

URI                          : http://packman.inode.at/suse/openSUSE_Leap_15.1/
Activé                       : Oui
Vérification GPG             : Oui
Rafraichissement automatique : Oui
Priorité                     : 99 (priorité par défaut)

Les priorités des dépôts sont sans effet. Tous les dépôts activés partagent la même priorité.
```

Pour supprimer un dépôt, utilisez la commande `removerepo`, suivie du nom du dépôt (Alias)

```bash
zypper removerepo packman 
Suppression du dépôt 'packman' ............................................[fait]
Le dépôt 'packman' a été supprimé.
```

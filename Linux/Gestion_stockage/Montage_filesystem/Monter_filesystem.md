
Quand une partition et un filesystem sont crée, il devient nécessaire de monter la partition pour la rendre accessible à l'utilisation

Pour ce faire, il existe plusieurs méthodes :

- montage manuel
- montage automatique

## Montage manuel

En montant manuellement une partition, il faut supposé que l'ont veut monter un périphérique de stockage type clé USB, disque USB, carte SD, etc....

### Montage filesystem

Pour monter la partition, il faut dans un premier temps créer un répertoire qui va accueillir le système de fichier, habituellement sous le répertoire `/mnt`

```bash
mkdir /mnt/HDD1
```

Ensuite il faut repérer le stockage à monter avec la commande `lsblk`

```bash
root@debian:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0     11:0    1 1024M  1 rom  
sda    254:0    0   20G  0 disk 
├─sda1 254:1    0 18,9G  0 part /
├─sda2 254:2    0    1K  0 part 
└─sda5 254:5    0  1,1G  0 part [SWAP]
sdb    254:16   0   20G  0 disk 
└─sdb1 254:17   0   20G  0 part 
```

Dans le cas ou, c'est un disque dur ou SSD, `sda` est le disque principal et `sda1` est la partition de boot

`sdb1` est donc la première partition de mon deuxième disque que je veut monter, pour ce faire il faut utiliser la commande `mount`

Il faut spécifier la partition cible et non le disque  dur complet

```bash
mount /dev/sdb1 /mnt/HDD1
```

il est possible de définir le type de filesystem

```bash
mount -t ext4 /dev/sdb1 /mnt/HDD1
```

La commande `mount` peut être utilisée avec un certain nombre de paramètres

- `-a` : Monter tous les systèmes de fichiers répertoriés dans le fichier `/etc/fstab`
- `-o` ou `--options` : Passer une liste d’options de montage ( voir option de `fstab` ) séparées par des virgules à la commande `mount`, modifie la façon dont le système de fichiers sera monté
- `-r` ou `-ro` : Monter le système de fichiers en lecture seule.
- `-w` ou `-rw` : Monter le système de fichiers en lecture/écriture

### Demonter filesystem

Pour démonter un système de fichiers, utilisez la commande `umount` suivie du nom du périphérique ou du point de montage. Si l’on considère l’exemple précédent, les deux commandes ci-dessous sont interchangeables

```bash
umount /dev/sdb1
umount /mnt/HDD1
```

## Montage automatique

Le montage automatique d'un filesystem peut être utile dans le cas ou, le filesystem est le `swap`, une partition d'un disque dur interne.

Le montage automatique, comme son nom l'indique, se fait automatiquement au boot et permet d'automatiser la tâche

### Modification fstab

Le fichier `/etc/fstab` contient les spécifications des systèmes de fichiers qui peuvent être montés. Il s’agit là d’un fichier texte dans lequel chaque ligne décrit un système de fichiers destiné à être monté, avec six champs par ligne dans l’ordre suivant

```bash
FILESYSTEM MOUNTPOINT TYPE OPTIONS DUMP PASS
```

- `FILESYSTEM` : Le périphérique qui contient le système de fichiers à monter. Au lieu du périphérique, vous pouvez spécifier l’UUID ou l’étiquette de la partition, ce que nous allons voir un peu plus loin.
- `MOUNTPOINT` : L’endroit où le système de fichiers sera monté.
- `TYPE` : Le type de système de fichiers.
- `OPTIONS` : Les options de montage qui seront passées à la commande `mount`.
- `DUMP` : Indique si les systèmes de fichiers ext2, ext3 ou ext4 doivent être pris en compte pour la sauvegarde par la commande `dump`. En règle générale, ce paramètre est égal à zéro, ce qui signifie qu’ils doivent être ignorés.
- `PASS` : Lorsqu’il a une valeur non nulle, il définit l’ordre dans lequel les systèmes de fichiers seront vérifiés au démarrage. En général, il est égal à zéro

Par exemple

```bash
/dev/Sda1  /mnt/HDD1  ext4  defaults 0 1
ou
UUID=dcefdf6a-2f65-4475-a299-4aa7a6b43ed4	/mnt/HDD1	ext4	defaults 0	1
LABEL=Test /mnt/HDD1 ext4 defaults 0 2
```

Pour connaitre l'UUID de la partition que l'ont veut monter automatiquement via `fstab`, la commande `lsblk`  avec le paramètre `-f` et la partition associé, permet de fournir cette information

L'avantage d'utiliser l'UUID est que, cet élément reste unique, même si le port USB qui sert à connecté le périphérique, n'est pas le même, ce qui évite de modifier sans cesse `fstab` pour adapter.

Utiliser le label est valide aussi, en alternative au UUID

```bash
lsblk -f /dev/sda1

NAME FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda1 ext4   TEST      6e2c12e3-472d-4bac-a257-c49ac07f3761   64,9G    33% /
```

Ici, `̀6e2c12e3-472d-4bac-a257-c49ac07f3761` représente l'UUID

La sortie standard de `lsblk` contient cependant des informations intéressantes

- `NAME` : Nom du périphérique qui contient le système de fichiers.
- `FSTYPE` : Type du système de fichiers.
- `LABEL` : Étiquette du système de fichiers.
- `UUID` : Identifiant universel unique (UUID) attribué au système de fichiers.
- `FSAVAIL` : L’espace disponible dans le système de fichiers.
- `FSUSE%` : Pourcentage d’utilisation du système de fichiers.
- `MOUNTPOINT` : L’endroit où le système de fichiers est monté

### Options fstab

Les options de montage spécifiées dans `OPTIONS` sont une liste de paramètres séparés par des virgules, qui peuvent être génériques ou spécifiques à un système de fichiers

- `atime` et `noatime` : Par défaut, à chaque fois qu’un fichier est lu, les informations de temps d’accès sont mises à jour. Désactiver ceci (avec `noatime`) peut accélérer les entrées-sorties sur le disque
- `auto` et `noauto` : Indique si le système de fichiers peut (ou non) être monté automatiquement avec `mount -a`.
- `defaults` : Cette option va passer les options `rw`, `suid`, `dev`, `exec`, `auto`, `nouser` et `async` à `mount`.
- `dev` et `nodev` : Indique si les périphériques de type caractère ou de type bloc seront interprétés dans le système de fichiers monté.
- `exec` et `noexec` : Permet ou empêche l’exécution de binaires sur le système de fichiers.
- `user` et `nouser` : Permet (ou non) à un utilisateur normal de monter le système de fichiers.
- `group` : Permet à un utilisateur de monter le système de fichiers s’il appartient au même groupe que celui qui est propriétaire du périphérique correspondant.
- `owner` : Permet à un utilisateur de monter le système de fichiers s’il est propriétaire du périphérique concerné.
- `suid` et `nosuid` : Autorise ou non les bits SETUID et SETGID à prendre effet.
- `ro` et `rw` : Monte un système de fichiers en lecture seule ou en écriture.
- `sync` et `async` : Indique si toutes les opérations d’entrées-sorties vers le système de fichiers doivent être effectuées de manière synchrone ou asynchrone. `async` est habituellement la valeur par défaut. La page de manuel de `mount` prévient que l’utilisation de `sync` sur un média avec un nombre limité de cycles d’écriture peut raccourcir la durée de vie du périphérique

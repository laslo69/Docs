
Quand une partition et un filesystem sont crée, il devient nécessaire de monter la partition pour la rendre accessible à l'utilisation

Pour ce faire, il existe plusieurs méthodes :

- montage manuel
- montage automatique

## Montage manuel

En montant manuellement une partition, il faut supposé que l'ont veut monter un périphérique de stockage type clé USB, disque USB, carte SD, etc....

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

## Montage automatique


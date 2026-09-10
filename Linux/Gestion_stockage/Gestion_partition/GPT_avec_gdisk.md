
L’outil `gdisk` est l’équivalent de `fdisk` pour les disques avec des partitions GPT. En fait, l’interface est inspirée de `fdisk`, avec une invite interactive et les mêmes commandes (ou des commandes très similaires) et n'enregistre pas automatiquement les modifications, `w` pour écrire les modifications et `q` pour quitter sans enregistrer ( si non enregistré avant )

Un disque `GPT` peut avoir jusqu'à 128 partitions, ce qui permet de justifier que l'utilisation de partition étendu n'est pas nécessaire

## Créer une partition

La commande pour créer une partition est `n`, comme dans `fdisk`

La principale différence est qu’en plus du numéro de partition et du premier et dernier secteur (ou de la taille), vous pouvez également spécifier le type de partition pendant la création

Les partitions GPT supportent beaucoup plus de types que les partitions MBR. Pour consulter la liste de tous les types supportés en utilisant la commande `l`

```bash
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-41943006, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-41943006, default = 41940991) or {+-}size{KMGTP}: +10G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8300
Changed type of partition to 'Linux filesystem'
```

## Supprimer une partition

Pour supprimer une partition, tapez `d` et le numéro de la partition. Contrairement à `fdisk`, la première partition ne sera pas sélectionnée automatiquement si c’est la seule sur le disque

Sur les disques GPT, les partitions peuvent être facilement réorganisées, ou "triées", pour éviter les écarts dans la séquence de numérotation. Pour ce faire, il suffit d’utiliser la commande `s`

Par exemple, prenons un disque avec la table de partitionnement suivante

```bash
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1024.0 MiB  8300  Linux filesystem
   2         2099200         2361343   128.0 MiB   8300  Linux filesystem
   3         2361344         2623487   128.0 MiB   8300  Linux filesystem
```

Si vous supprimez la deuxième partition, la table deviendra

```bash
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1024.0 MiB  8300  Linux filesystem
   3         2361344         2623487   128.0 MiB   8300  Linux filesystem
```

Si vous utilisez la commande `s`, ce sera

```bash
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1024.0 MiB  8300  Linux filesystem
   2         2361344         2623487   128.0 MiB   8300  Linux filesystem
```

la troisième partition est devenue la seconde, mais il y'aura toujours un écart entre la fin de la première partition et le début de la seconde

## Afficher la table de partitionnement en cours

La commande `p` est utilisée pour afficher la table de partitionnement en cours

```bash
Command (? for help): p
Disk /dev/sdb: 3903578 sectors, 1.9 GiB
Model: DataTraveler 2.0
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): AB41B5AA-A217-4D1E-8200-E062C54285BE
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 3903544
Partitions will be aligned on 2048-sector boundaries
Total free space is 1282071 sectors (626.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1024.0 MiB  8300  Linux filesystem
   2         2623488         3147775   256.0 MiB   8300  Linux filesystem
```

L’espace libre est affiché sur la dernière ligne ( ligne : Total free space is 1282071 sectors ( 626.0MiB), il n’est donc pas nécessaire d’avoir un équivalent de la commande `F` de fdisk

Chaque disque possède un identifiant de disque unique (GUID), représenté par un nombre hexadécimal de 128 bits, attribué de manière aléatoire lors de la création de la table de partitionnement. Étant donné qu’il existe 3,4 × 1038 valeurs possibles pour ce nombre, les chances que deux disques aléatoires disposent du même GUID sont très faibles

Le GUID peut être utilisé pour identifier les systèmes de fichiers à monter au démarrage (et à quel endroit), ce qui évite d’utiliser le chemin d’accès au périphérique (comme `/dev/sdb`)

## Changer type partition

Tout comme `fdisk`, il est possible de modifier le type de partition en utilisant l'option `t`

```bash
Command (? for help): t
Using 1
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): L
Type search string, or <Enter> to show all codes: 
0700 Microsoft basic data                0701 Microsoft Storage Replica         
0702 ArcaOS Type 1                       0c01 Microsoft reserved                
2700 Windows RE                          3000 ONIE boot                         
3001 ONIE config                         3900 Plan 9                            
4100 PowerPC PReP boot                   4200 Windows LDM data                  
4201 Windows LDM metadata                4202 Windows Storage Spaces            
7501 IBM GPFS                            7f00 ChromeOS kernel                   
7f01 ChromeOS root                       7f02 ChromeOS reserved                 
7f03 ChromeOS firmware                   7f04 ChromeOS mini-OS                  
7f05 ChromeOS hibernate                  8200 Linux swap                        
8300 Linux filesystem                    8301 Linux reserved                    
8302 Linux /home                         8303 Linux x86 root (/)                
8304 Linux x86-64 root (/)               8305 Linux ARM64 root (/)              
8306 Linux /srv                          8307 Linux ARM32 root (/)              
8308 Linux dm-crypt                      8309 Linux LUKS                        
830a Linux IA-64 root (/)                830b Linux x86 root verity             
830c Linux x86-64 root verity            830d Linux ARM32 root verity           
830e Linux ARM64 root verity             830f Linux IA-64 root verity           
8310 Linux /var                          8311 Linux /var/tmp                    
8312 Linux user's home                   8313 Linux x86 /usr                    
8314 Linux x86-64 /usr                   8315 Linux ARM32 /usr                  
Press the <Enter> key to see more codes, q to quit: q

Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'
```

## Redimenssionner partition

Avec `gdisk`, il n'est pas possible de redimensionner une partition

Il faut obligatoirement la détruire et refaire une nouvelle partition, bien penser à faire un backup des fichiers sur la partition.

La suppression de la partition supprime également les fichiers de la partition

## Les options de récupération

Les disques GPT enregistrent des copies de sauvegarde de l’en-tête GPT et de la table de partitionnement, ce qui facilite la récupération des disques au cas où ces données sont endommagées

`gdisk` fournit des fonctionnalités pour faciliter ces tâches de récupération, accessibles avec la commande `r`

Vous pouvez reconstruire un en-tête GPT ou une table de partitionnement endommagés avec `b` et `c` respectivement, ou utiliser l’en-tête et la table pour reconstruire une sauvegarde avec `d` et `e`. Vous pouvez également convertir un MBR en GPT avec `f` et faire l’inverse avec `g`, parmi d’autres opérations

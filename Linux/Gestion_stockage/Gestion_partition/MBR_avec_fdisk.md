
La table de partitionnement est stockée sur le premier secteur d’un disque ( cylinder 0, head 0, sector 0 ) appelé boot sector, avec un chargeur d’amorçage comme GRUB sur les systèmes Linux, pour rendre le disque amorçable, la première partition doit être une partition primaire

En revanche, le MBR présente une série de limitations gênantes sur les systèmes modernes, comme l’impossibilité d’adresser des disques d’une taille supérieure à 2 To, et la limite maximale de 4 partitions primaires par disque et 1 partition étendue

Petit tips, pour savoir si des partitions sont membres d'une partition étendu, un disque `MBR` ne peut avoir max, que 4 partitions, automatiquement, toutes partitions égale à 5 ou supérieur est une `partition logique`

Sur un disque MBR, vous pouvez avoir deux types de partitions : 

- primaire
- étendue

Une manière de contourner cette limitation consiste à créer une partition étendue qui sert de conteneur pour les partitions logiques. Vous pourriez avoir, par exemple, une partition primaire, une partition étendue occupant le reste de l’espace disque et cinq partitions logiques à l’intérieur de celle-ci

L’outil standard pour gérer les partitions MBR sous Linux est `fdisk`, un programme interactif, piloté par un menu. Pour l’utiliser, tapez `fdisk` suivi du nom du périphérique associé au disque à modifier

```bash
fdisk /dev/sdb
```

Au lancement, `fdisk` affiche un message d’accueil ainsi qu’un avertissement, et attend vos commandes, par défaut, `fdisk` n'enregistre pas les modifications automatiquement.

Pratique pour tester sans altérer le disque, pour enregistrer, saisir le paramètre `w`, pour quitter sans enregistrer : `q`

```bash
fdisk /dev/sdb

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

## Creer une partition

Pour créer une partition, utilisez la commande `n`

Par défaut, les partitions seront créées au début de l’espace non alloué sur le disque. Vous devrez indiquer le type de partition (primaire ou étendue), le premier secteur et le dernier secteur

Pour le premier secteur, vous pouvez généralement accepter la valeur par défaut proposée par `fdisk`, à moins que vous n’ayez besoin d’une partition qui commence à un secteur donné

Au lieu de spécifier le dernier secteur, vous pouvez très bien préciser une taille suivie des lettres `K`, `M`, `G`, `T` ou `P` (Kilo, Méga, Giga, Téra ou Péta)

Ainsi, si vous souhaitez créer une partition de 1 Go, vous pouvez spécifier `+1G` comme `Last sector`, et `fdisk` va dimensionner la partition en fonction

```bash
Commande (m pour l'aide) : n
Type de partition
   p   primaire (0 primaire, 0 étendue, 4 libre)
   e   étendue (conteneur pour partitions logiques)
Sélectionnez (p par défaut) : p
Numéro de partition (1-4, 1 par défaut) : 
Premier secteur (2048-41943039, 2048 par défaut) : 
Dernier secteur, +/-secteurs ou +/-taille{K,M,G,T,P} (2048-41943039, 41943039 par défaut) : +5G

Une nouvelle partition 1 de type « Linux » et de taille 5 GiB a été créée.
```

## Supprimer partition

Pour supprimer une partition, utilisez la commande `d`, `fdisk` vous demandera le numéro de la partition que vous voulez effacer, sauf s’il n’y a qu’une seule partition sur le disque. Dans ce cas, cette partition sera sélectionnée et supprimée immédiatement

Notez que si vous supprimez une partition étendue, toutes les partitions logiques qu’elle contient seront également supprimées

```bash
Commande (m pour l'aide) : d
Numéro de partition (1,2, 2 par défaut) : 2

La partition 2 a été supprimée.
```

## Attention aux écarts

Gardez à l’esprit que lorsque vous créez une nouvelle partition avec `fdisk`, la taille maximale sera limitée à la quantité maximale d’espace contigu non alloué sur le disque et ne peut pas être ré-organiser. Admettons, par exemple, que vous ayez la table de partitionnement suivante

```bash
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdd1          2048 1050623 1048576  512M 83 Linux
/dev/sdd2       1050624 2099199 1048576  512M 83 Linux
/dev/sdd3       2099200 3147775 1048576  512M 83 Linux
```

Vous supprimez alors la partition 2 et vérifiez l’espace libre

```bash
Commande (m pour l'aide) : d
Numéro de partition (1,2, 2 par défaut) : 2

La partition 2 a été supprimée.

Command (m for help): F
Unpartitioned space /dev/sdd: 881 MiB, 923841536 bytes, 1804378 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

  Start     End Sectors  Size
1050624 2099199 1048576  512M
3147776 3903577  755802  369M
```

En additionnant la taille de l’espace non alloué, nous disposons théoriquement de 881 Mo. Or, voyez ce qui se passe lorsque nous essayons de créer une partition de 700 Mo

```bash
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2,4, default 2): 2
First sector (1050624-3903577, default 1050624):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-2099199, default 2099199): **+700M**
Value out of range.
```

Cela est dû au fait que le plus grand espace contigu non alloué sur le disque est le bloc de 512 Mo qui appartenait à la partition 2. Votre nouvelle partition ne peut pas "déborder" vers la partition 3 pour utiliser une partie de l’espace non alloué qui se trouve après

## Afficher table partition MBR

La commande `p` est utilisée pour afficher la table de partitionnement en cours. Le résultat ressemble à ceci :

```
Command (m for help): **p**
Disk /dev/sda: 111.8 GiB, 120034123776 bytes, 234441648 sectors
Disk model: CT120BX500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x97f8fef5

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1            4096 226048942 226044847 107.8G 83 Linux
/dev/sda2       226048944 234437550   8388607     4G 82 Linux swap / Solaris
```

Voici la signification de chacune des colonnes :

- `Device` : Le périphérique affecté à la partition
- `Boot` : Indique si la partition est amorçable (`bootable`) ou non
- `Start` : Le secteur où commence la partition
- `End` : Le secteur où se termine la partition
- `Sectors` : Le nombre total de secteurs dans la partition. Multipliez-le par la taille d’un secteur pour obtenir la taille de la partition en octets
- `Size` : La taille de la partition au format "humainement lisible". Dans l’exemple ci-dessus, les valeurs sont exprimées en gigaoctets
- `Id` : La valeur numérique qui représente le type de la partition
- `Type` : La description du type de partition

## Vérifier l’espace non alloué

Si vous souhaitez connaître la quantité d’espace libre sur le disque, vous pouvez utiliser la commande `F` pour afficher l’espace non alloué

```bash
Commande (m pour l'aide) : F
Espace non partitionné /dev/vdb : 15 GiB, 16105078784 octets, 31455232 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets

   Début      Fin Secteurs Taille
10487808 41943039 31455232    15G
```

La ligne " Espace non partitionnée " donne la taille de stockage non utilisé du disque

## Changer le type de partition

Il est parfois nécessaire de changer le type de partition, notamment lorsqu’il s’agit de disques qui seront utilisés avec d’autres systèmes d’exploitation et d’autres plateformes

Pour ce faire, utilisez la commande `t` suivie du numéro de la partition que vous souhaitez modifier

Le type de partition doit être spécifié par son code hexadécimal correspondant, et vous pouvez afficher une liste de tous les codes disponibles en utilisant la commande `l`

```bash
Commande (m pour l'aide) : t
Partition 1 sélectionnée
Code Hexa ou synonyme (taper L pour afficher tous les codes) :
```

en tapant `L`, une liste de type de partition apparait

```bash
00 Vide             27 TFS WinRE masqu  82 partition d'éch  c1 DRDOS/sec (FAT-
01 FAT12            39 Plan 9           83 Linux            c4 DRDOS/sec (FAT-
02 root XENIX       3c récupération Pa  84 OS/2 cachée ou   c6 DRDOS/sec (FAT-
03 usr XENIX        40 Venix 80286      85 Linux étendue    c7 Syrinx         
04 FAT16 <32M       41 PPC PReP Boot    86 NTFS volume set  da Données non-FS 
05 Étendue          42 SFS              87 NTFS volume set  db CP/M / CTOS / .
06 FAT16            4d QNX4.x           88 Linux plaintext  de Dell Utility   
07 HPFS/NTFS/exFAT  4e 2e partie QNX4.  8e LVM Linux        df BootIt         
08 AIX              4f 3e partie QNX4.  93 Amoeba           e1 DOS access     
09 Amorçable AIX    50 OnTrack DM       94 Amoeba BBT       e3 DOS R/W        
0a Gestionnaire d'  51 OnTrack DM6 Aux  9f BSD/OS           e4 SpeedStor      
0b W95 FAT32        52 CP/M             a0 IBM Thinkpad hi  ea Amorçage Linux 
0c W95 FAT32 (LBA)  53 OnTrack DM6 Aux  a5 FreeBSD          eb BeOS fs        
0e W95 FAT16 (LBA)  54 OnTrackDM6       a6 OpenBSD          ee GPT            
0f Étendue W95 (LB  55 EZ-Drive         a7 NeXTSTEP         ef EFI (FAT-12/16/
10 OPUS             56 Golden Bow       a8 UFS Darwin       f0 Linux/PA-RISC b
11 FAT12 masquée    5c Priam Edisk      a9 NetBSD           f1 SpeedStor      
12 Compaq diagnost  61 SpeedStor        ab Amorçage Darwin  f4 SpeedStor      
14 FAT16 masquée <  63 GNU HURD ou Sys  af HFS / HFS+       f2 DOS secondaire 
16 FAT16 masquée    64 Novell Netware   b7 BSDI fs          f8 EBBR protégé   
17 HPFS/NTFS masqu  65 Novell Netware   b8 partition d'éch  fb VMware VMFS    
18 AST SmartSleep   70 DiskSecure Mult  bb Boot Wizard mas  fc VMware VMKCORE 
1b W95 FAT32 masqu  75 PC/IX            bc Acronis FAT32 L  fd RAID Linux auto
1c W95 FAT32 masqu  80 Minix ancienne   be Amorçage Solari  fe LANstep        
1e W95 FAT16 masqu  81 Minix / Linux a  bf Solaris          ff BBT            
24 NEC DOS        

Synonymes:
   linux          - 83
   swap           - 82
   extended       - 05
   uefi           - EF
   raid           - FD
   lvm            - 8E
   linuxex        - 85
```

Ne confondez pas le type de partition avec le système de fichiers utilisé

Même si au début il y avait une corrélation entre les deux, vous ne pouvez plus supposer que c’est le cas aujourd’hui. Une partition Linux, par exemple, peut contenir n’importe quel système de fichiers natif de Linux, comme ext4 ou ReiserFS

```bash
Code Hexa ou synonyme (taper L pour afficher tous les codes) :82
Type de partition « Linux » modifié en « Linux swap / Solaris ».
```

la partition est désormais une partition capable de faire du `swap`

## Redimenssionner partition

Pour redimenssionner une partition, l'option `e` est utile

Noter qu'il faut spécifier le format avec la lettre correspondant à la taille ( G pour giga, etc...)

```bash
Commande (m pour l'aide) : e
Partition 1 sélectionnée

Nouvelle <taille>{K,M,G,T,P} en octets ou <taille>S en secteurs (20G par défaut): 10G

La partition 1 a été redimensionnée.

Commande (m pour l'aide) : p
Disque /dev/vdb : 20 GiB, 21474836480 octets, 41943040 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
Type d'étiquette de disque : dos
Identifiant de disque : 0x25e3ceb4

Périphérique Amorçage Début      Fin Secteurs Taille Id Type
/dev/vdb1              2048 20973567 20971520    10G 82 partition d'échange Linu
```


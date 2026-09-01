
Afin de contrôler la machine, le composant principal du système d’exploitation - le noyau - doit être chargé par un programme appelé le chargeur de démarrage, qui est lui-même chargé par un firmware préinstallé comme le BIOS ou l’UEFI

Le chargeur de démarrage peut être personnalisé pour transmettre des paramètres au noyau, par exemple la partition qui contient le système de fichiers racine ou le mode dans lequel le système d’exploitation doit s’exécuter

 Une fois chargé, le noyau poursuit le processus de démarrage en identifiant et en configurant le matériel. Au terme de ce processus, le noyau lance l’utilitaire chargé de démarrer et de gérer les services du système

## Boot BIOS

Le BIOS est un programme stocké dans une puce de mémoire non volatile attachée à la carte mère, exécuté à chaque fois que l’ordinateur est mis sous tension

Ce type de programme est appelé firmware et son emplacement de stockage est distinct des autres périphériques de stockage que le système peut avoir. Le BIOS part du principe que les 440 premiers octets du premier périphérique de stockage - suivant l’ordre défini dans l’interface de configuration du BIOS - constituent la première phase du chargeur de démarrage

Les 512 premiers octets d’un périphérique de stockage sont appelés le MBR ( Master Boot Record ) pour les périphériques de stockage qui utilisent le schéma de partition DOS standard et, en plus de la première phase du chargeur de démarrage, contient la table de partitions

1) Test POST
2) BIOS active les composants basiques ( clavier, sortie vidéo, stockage )
3) Bios exécute la première phase du chargeur de démarrage à partir du MBR ( les 440 premier octets du périphérique défini)
4) La première phase du chargeur de démarrage appelle la deuxième parti du chargeu de démarrage( chargement options démarrage, kernel )
5) Initialisation systeme : Ie kernel prend en charge le CPU, détecte et configure le systeme
6) Le kernel ouvre un disque memoire initial ( initramfs), qui contient un systeme de fichiers utilisé en fichiers racines temporaire lors du demarrage. Fournis les modules pour acceder au vrai systeme de fichiers racine
7) Des que le systeme de fichiers racine est disponible, le kernel monte tous les systemes de fichiers dans /etc/fstab et execute le premier programme ( utilitaire init, pid = 1)

## Boot UEFI

UEFI se distingue du BIOS sur plusieurs points essentiels.

Tout comme le BIOS, UEFI est également un micrologiciel, mais il est capable d’identifier des partitions et de lire un grand nombre de systèmes de fichiers qui s’y trouvent et ne s'appuie pas sur MBR, prend en compte uniquement les paramètres stockés dans la NVRAM

Ces définitions indiquent l’emplacement des programmes compatibles UEFI, appelés applications EFI, qui seront exécutés automatiquement ou invoqués à partir d’un menu de démarrage

Les applications EFI peuvent être des chargeurs d’amorçage, des sélecteurs de systèmes d’exploitation, des outils de diagnostic et de réparation système, etc. Ils doivent figurer dans une partition classique de périphérique de stockage et dans un système de fichiers compatible ( système de fichier compatibles : FAT12,16,32)

La partition qui contient les applications EFI est appelée EFI System Partition ou simplement ESP. Cette partition ne doit en aucun cas être partagée avec d’autres systèmes de fichiers du système comme le système de fichiers racine ou les systèmes de fichiers contenant les données des utilisateurs. Le répertoire EFI dans la partition ESP contient les applications référencées par les entrées enregistrées dans la NVRAM

1) Test POST
2) UEFI active les composants basiques ( clavier, sortie vidéo, stockage )
3) UEFI lit les paramètres stockés dans la NVRAM pour exécuter l'application EFI ( généralement un chargeur de démarrage )
4) UEFI lit le EFI bootloader dans la partition ESP, qui charge le kernel
5) Initialisation systeme : Ie kernel prend en charge le CPU, détecte et configure le systemeaspects fondamentaux du systeme
6) Le kernel ouvre un disque memoire initial ( initramfs), qui contient un systeme de fichiers utilisé en fichiers racines temporaire lors du demarrage. Fournis les modules pour acceder au vrai systeme de fichiers racine
7) Des que le systeme de fichiers racine est disponible, le kernel monte tous les systemes de fichiers dans /etc/fstab et execute le premier programme ( utilitaire init, pid = 1)

UEFI ne prend en compte que les paramètres stockés dans la NVRAM attaché à la carte mère

Ces définitions indiquent l’emplacement des programmes compatibles UEFI, appelés _applications EFI_, qui seront exécutés automatiquement ou invoqués à partir d’un menu de démarrage

Les applications EFI peuvent être des chargeurs d’amorçage, des sélecteurs de systèmes d’exploitation, des outils de diagnostic et de réparation système, etc. Ils doivent figurer dans une partition classique de périphérique de stockage et dans un système de fichiers compatible ( généralement FAT 12,16,32 )

En l'abscence d'entrée NVRAM, c'est `/boot/efi/EFI/BOOT/bootx64.efi` qui sera exécuté

arborescence de `/BOOT/`

- grubx64.efi : bootloader
- shimx64.efi : bootloader secure boot
- grub.cfg : Configuration de grub2
- BOOTX64.efi : fichier lu pour ajout NVRAM
- mmx64.efi : Executable signé secure boot avec clé machine, passe le relais à GRUB2
## Partition boot

Sous Linux, les fichiers nécessaires au processus de démarrage sont généralement stockés sur une partition de démarrage, montée sous le système de fichiers racine et communément appelée `/boot`

Une partition de démarrage n’est pas nécessaire sur les systèmes actuels dans la mesure où les chargeurs de démarrage comme GRUB permettent généralement de monter le système de fichiers racine et de rechercher les fichiers nécessaires à l’intérieur d’un répertoire `/boot`, mais il s’agit là d’une bonne pratique qui permet de séparer les fichiers nécessaires au processus de démarrage du reste du système de fichiers

Cette partition est généralement la première sur le disque

Cela tient au fait que le BIOS des PC IBM d’origine s’adressait aux disques en utilisant des cylindres, des têtes et des secteurs (CHS pour _Cylinder/Head/Sector_), avec un maximum de 1024 cylindres, 256 têtes et 63 secteurs, ce qui donne une taille maximale de disque de 528 Mo (504 Mo sous MS-DOS)

Ce qui signifie que tout ce qui dépasse cette limite ne serait pas accessible, à moins qu’un autre système d’adressage de disque comme LBA, ( Logical Block Addressing ) ne soit utilisé

Ainsi, pour une compatibilité maximale, la partition `/boot` est généralement située au début du disque et se termine avant le cylindre 1024 (528 Mo), ce qui garantit que la machine sera toujours capable de charger le noyau. La taille recommandée pour cette partition sur une machine actuelle est de 300 Mo

D’autres raisons pour une partition `/boot` séparée sont le chiffrement et la compression, étant donné que certaines méthodes peuvent ne pas encore être supportées par GRUB 2, ou si vous avez besoin de formater la partition racine du système (`/`) en utilisant un système de fichiers non supporté

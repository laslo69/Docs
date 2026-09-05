
GRUB 2 est une réécriture complète de GRUB qui se veut plus propre, plus sûre, plus robuste et plus puissante. Parmi les nombreux avantages par rapport à GRUB Legacy figurent un fichier de configuration beaucoup plus flexible (avec beaucoup plus de commandes et d’instructions conditionnelles, comme un langage de script), une conception plus modulaire et une meilleure localisation/internationalisation

On trouve également la prise en charge de thèmes et de menus de démarrage graphiques avec écrans de démarrage, la possibilité de démarrer les ISO de LiveCD directement depuis le disque dur, une meilleure prise en charge des architectures non-x86, une prise en charge universelle des UUID (ce qui facilite l’identification des disques et des partitions) et bien plus encore

## Demarrer depuis GRUB2

Vous pouvez utiliser le shell GRUB 2 pour démarrer le système au cas où une mauvaise configuration dans une entrée de menu le ferait échouer.

La première chose à faire est de trouver l’emplacement de la partition de démarrage. Vous pouvez le faire avec la commande `ls`, qui vous affichera une liste des partitions et des disques que GRUB 2 a trouvés

```bash
ls
(proc) (hd0) (hd0,msdos1)
```

Il n’y a qu’un seul disque `(hd0)` avec une seule partition : `(hd0,msdos1)`

Les disques et partitions répertoriés varieront selon votre système. Dans notre exemple, la première partition de `hd0` est appelée `msdos1` parce que le disque a été partitionné en utilisant le schéma de partitionnement MBR. S’il était partitionné en GPT, le nom serait `gpt1`

Pour démarrer Linux, nous avons besoin d’un noyau et d’un disque mémoire initial (initrd)

 Examinons le contenu de `(hd0,msdos1)`, Vous pouvez ajouter le paramètre `-l` à `ls` pour obtenir un affichage détaillé, comparable à ce que vous obtiendriez dans un terminal Linux

```bash
grub> ls (hd0,msdos1)/
lost+found/ swapfile etc/ media/ bin/ boot/ dev/ home/ lib/ lib64/ mnt/ opt/ proc/ root/ run/ sbin/ srv/ sys/ tmp/ usr/ var/ initrd.img initrd.img.old vmlinuz cdrom/
```

Notez que nous avons un noyau (`vmlinuz`) et des images initrd (`initrd.img`) directement dans le répertoire racine. Dans le cas contraire, nous pouvons vérifier le contenu de `/boot` avec `list (hd0,msdos1)/boot/`

À présent, définir la partition de démarrage

```bash
grub> set root=(hd0,msdos1)
```

Chargez le noyau Linux avec la commande `linux`, suivie du chemin vers le noyau et de l’option `root=` pour indiquer au noyau où se trouve le système de fichiers racine du système d’exploitation.

En spécifiant `/vmlinuz`, un lien symbolique est utilisé vers le noyau considéré comme actuel

```bash
grub> linux /vmlinuz root=/dev/sda1
```

Pour définir un kernel précis

```bash
grub> linux /vmlinuz-6.12.0-211.42.1.el10_2.x86_64 root=/dev/sda1
```

Charger le disque mémoire initial ( initrd )

```bash
grub> initrd /initrd.img
```

Même principe pour le initrd, possible de définir une version précise mais doit correspondre à la même version que le kernel

```bash
grub> initrd /initrd.img-6.12.0-...
```

## GRUB2 rescue shell

En cas de défaillance du démarrage, GRUB 2 peut charger un shell de secours

La procédure pour démarrer un système depuis ce shell est presque la même que pour démarrer depuis GRUB.

Cependant, vous devrez charger quelques modules de GRUB 2 pour que tout fonctionne

Une fois que vous avez identifié la partition de démarrage, utilisez la commande `set prefix=`, suivie du chemin complet vers le répertoire contenant les fichiers de GRUB 2, habituellement `/boot/grub` pour les distributions basé sur `debian`, `/boot/grub2/` pour les distributions de la famille `Red hat`

```bash
grub rescue> set prefix=(hd0,msdos1)/boot/grub
```

A présent, chargez les modules `normal` et `linux` à l’aide de la commande `insmod`

Les modules se trouve ( du moins sur ma VM oracle 10) dans le répertoire `/boot/grub2/i386-pc/`, une grande liste de modules est disponibles et finissent par l'extension `.mod`

```bash
grub rescue> insmod normal
grub rescue> insmod linux
```

Ensuite, définissez la partition de démarrage, chargez le noyau Linux, le disque mémoire initial essayez de démarrer avec `boot`, comme pour démarrer depuis GRUB.

si UUID utilisé, remplacer :

`root=/dev/sda1` par `root=UUID=xxxx-xxxx-xxxx`

```bash
grub rescue> set root=(hd0,msdos1)
grub rescue> linux /vmlinuz root=/dev/sda1
grub rescue> initrd /initrd.img
gros rescue> boot
```

## Installer GRUB2

Si vous avez un système non amorçable, démarrez en utilisant un live CD ou un disque de secours, repérez la partition de démarrage de votre système, montez cette partition et exécutez la commande

GRUB 2 peut être installé à l’aide de la commande `grub-install`

Le premier disque d’un système est généralement le périphérique de démarrage et vous devrez éventuellement savoir s’il y a une partition de démarrage sur le disque. On peut le faire avec la commande `fdisk` pour lister les partitions

```bash
fdisk -l /dev/sda
Disk /dev/sda: 111,8 GiB, 120034123776 bytes, 234441648 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x97f8fef5

Device     Boot    Start       End   Sectors   Size Id Type
/dev/sda1  *        2048   2000895   1998848   976M 83 Linux
/dev/sda2        2002942 234440703 232437762 110,9G  5 Extended
/dev/sda5        2002944  18008063  16005120   7,6G 82 Linux swap / Solaris
/dev/sda6       18010112 234440703 216430592 103,2G 83 Linux
```

Le `*` signifie que la partition possède le **flag bootable** dans la table de partitions MBR, sous la colonne `Boot`. N'existe pas dans un partitionage GPT+UEFI

Maintenant, créez un répertoire temporaire sous `/mnt` et montez la partition en-dessous, l'objectif est de monter le disque de l'autre pc sur un système fonctionnel et ré-installer la partition `/boot` et fournir un secteur d'amorçage fonctionnel :

```bash
mkdir /mnt/tmp
mount /dev/sda1 /mnt/tmp
```

Ensuite, exécutez `grub-install` en le faisant pointer vers le périphérique de démarrage (et non la partition) ainsi que le répertoire où la partition de démarrage est montée

Si votre système dispose d’une partition de démarrage dédiée

```bash
grub-install --boot-directory=/mnt/tmp /dev/sda
```

Si vous effectuez l’installation sur un système qui n’a pas de partition de démarrage, mais seulement un répertoire `/boot` sur le système de fichiers racine, pointez `grub-install` vers celui-ci.

```bash
grub-install --boot-directory=/boot/ /dev/sda
```

## configuration GRUB2

La configuration de grub va lire les fichiers

- `/boot/grub/device.map`
- `/etc/default/grub`
- Tous les fichiers dans `/boot/grub.d/`

Le ficier `device.map` contient la numérotation des disques, en sont absence, la numérotation des disques dépend de l'ordre détecté dans le `BIOS`.

Il est possible de l'éditer manuellement ou de le générer automatiquement avec la commande `grub-mkdevicemap`

```bash
grub-mkdevicemap
(hd0)   /dev/disk/by-id/nvme-WDS500G3XHC-00SJG0_193686801877
(hd1)   /dev/disk/by-id/ata-ST8000VN004-2M2101_WKD07X8B
(hd2)   /dev/disk/by-id/ata-CT1000BX500SSD1_2515E9B43DC4
```

Le fichier de configuration par défaut pour GRUB 2 est `/boot/grub/grub.cfg` ou `/boot/grub2/grub.cfg` selon la distribution. Ce fichier est généré automatiquement et il n’est pas recommandé de le modifier à la main au risque de casser le système

Pour apporter des modifications à la configuration de GRUB2, vous devez éditer le fichier `/etc/default/grub` puis lancer la commande `update-grub` pour générer un fichier valide.

`update-grub` est généralement un raccourci vers `grub-mkconfig -o /boot/grub/grub.cfg` ou `grub2-mkconfig -o /boot/grub2/grub.cfg`, les deux commandes produisent le même résultat

Il y a une série d’options dans le fichier `/etc/default/grub` qui contrôlent le comportement de GRUB 2, comme le noyau par défaut au démarrage, le délai d’attente, les paramètres supplémentaires de la ligne de commande, etc

Les plus importantes sont :

`GRUB_DEFAULT=`

L’entrée de menu par défaut au démarrage. Il peut s’agir d’une valeur numérique (comme `0`, `1`, etc.), du nom d’une entrée de menu (comme `debian` ou `saved`), qui est utilisée en conjonction avec `GRUB_SAVEDEFAULT=`, voir l’explication ci-dessous. N’oubliez pas que les entrées de menu commencent à zéro, donc la première entrée de menu est `0`, la seconde est `1`, etc

`GRUB_SAVEDEFAULT=`

Si cette option est définie à `true` et que `GRUB_DEFAULT=` est défini à `saved`, alors l’option de démarrage par défaut sera toujours la dernière sélectionnée dans le menu de démarrage

`GRUB_TIMEOUT=`

Le délai en secondes avant que l’entrée de menu par défaut ne soit sélectionnée. S’il est défini à `0`, le système démarrera l’entrée par défaut sans afficher le menu. S’il est défini à `-1`, le système attendra que l’utilisateur choisisse une option, peu importe le temps que cela prendra

`GRUB_CMDLINE_LINUX=`

Cette liste énumère les options en ligne de commande qui seront ajoutées aux entrées du noyau Linux

`GRUB_CMDLINE_LINUX_DEFAULT=`

Par défaut, deux entrées de menu sont générées pour chaque noyau Linux, une avec les options par défaut et une entrée pour la récupération

`GRUB_ENABLE_CRYPTODISK=`

Si la valeur est `y`, les commandes comme `grub-mkconfig`, `update-grub` et `grub-install` vont rechercher les disques chiffrés et ajouter les commandes nécessaires pour y accéder au démarrage

## Gerer entrees menu GRUB2

Lorsque `update-grub` est exécuté, GRUB 2 va rechercher les noyaux et les systèmes d’exploitation sur la machine et générer les entrées de menu correspondantes dans le fichier `/boot/grub/grub.cfg`

De nouvelles entrées peuvent être ajoutées manuellement aux fichiers de script dans le répertoire `/etc/grub.d/`

Ces fichiers doivent être exécutables, et ils sont traités en ordre numérique par `update-grub`

Ainsi, `05_debian_theme` est traité avant `10_linux` et ainsi de suite. Les entrées de menu personnalisées sont généralement ajoutées au fichier `40_custom`

La syntaxe de base pour une entrée de menu est

```bash
menuentry "Default OS" {
    set root=(hd0,1)
    linux /vmlinuz root=/dev/sda1 ro quiet splash
    initrd /initrd.img
}
```

La première ligne commence toujours par `menuentry` et se termine par `}`. Le texte entre guillemets sera affiché comme titre d’entrée dans le menu de démarrage de GRUB 2.

Le paramètre `set root` définit le disque et la partition où se situe le système de fichiers racine du système d’exploitation. Notez que dans GRUB 2 les disques sont numérotés à partir de zéro, donc `hd0` est le premier disque (`sda` sous Linux), `hd1` le second, et ainsi de suite

Quant aux partitions, elles sont numérotées à partir de un. Dans l’exemple ci-dessus, le système de fichiers racine est situé sur le premier disque (`hd0`), la première partition (`,1`), ou `sda1`

Dans l’exemple ci-dessus, nous avons spécifié la partition racine (`root=/dev/sda1`) en passant trois paramètres au noyau : la partition racine doit être montée en lecture seule (`ro`), la plupart des messages de journalisation doivent être désactivés (`quiet`) et un écran de démarrage doit être affiché (`splash`)

Au lieu de spécifier directement le périphérique et la partition, vous pouvez également demander à GRUB 2 de rechercher un système de fichiers avec une étiquette ou un UUID (_Universally Unique Identifier_) spécifique. Pour ce faire, utilisez le paramètre `search --set=root` suivi du paramètre `--label` et de l’étiquette du système de fichiers à rechercher, ou `--fs-uuid` suivi de l’UUID du système de fichiers

Pour trouver l'UUID d'un système de fichier

```bash
ls -l /dev/disk/by-uuid
total 0
lrwxrwxrwx 1 root root 10 nov  4 08:40 3e0b34e2-949c-43f2-90b0-25454ac1595d -> ../../sda5
lrwxrwxrwx 1 root root 10 nov  4 08:40 428e35ee-5ad5-4dcb-adca-539aba6c2d84 -> ../../sda6
lrwxrwxrwx 1 root root 10 nov  5 19:10 56C11DCC5D2E1334 -> ../../sdb1
lrwxrwxrwx 1 root root 10 nov  4 08:40 ae71b214-0aec-48e8-80b2-090b6986b625 -> ../../sda1
```

Dans l’exemple, l’UUID pour `/dev/sda1` est `ae71b214-0aec-48e8-80b2-090b6986b625`. Si vous souhaitez le définir comme périphérique racine pour GRUB 2, la commande sera : `search --set=root --fs-uuid ae71b214-0aec-48e8-80b2-090b6986b625`

Quand on utilise la commande `search`, il est courant d’ajouter le paramètre `--no-floppy` pour que GRUB ne perde pas de temps à chercher sur des disquettes floppy

La ligne `linux` indique où se trouve le noyau du système d’exploitation (dans ce cas, le fichier `vmlinuz` à la racine du système de fichiers). Après cela, vous pouvez passer les paramètres en ligne de commande au noyau.

Il est possible de spécifier une version du kernel en fournissant le chemin complet : `linux /vmlinuz/vmlinuz-6.12.100+deb13-amd64`

Dans ce cas, le initrd correspondant doit aussi être fournis

## Interaction GRUB2

Lorsque vous démarrez un système avec GRUB 2, vous verrez un menu d’options. Utilisez les touches fléchées pour sélectionner une option et Entrée pour confirmer et démarrer l’entrée sélectionnée

Si vous voyez juste un compte à rebours, mais pas de menu, appuyez sur Maj pour afficher le menu

Pour éditer une option, sélectionnez-la avec les touches fléchées et appuyez sur e. Cela affichera une fenêtre d’édition avec le contenu `menuentry` associé à cette option, tel que défini dans `/boot/grub/grub.cfg`

Après avoir édité une option, tapez Ctrl+X ou F10 pour démarrer, ou Échap pour revenir au menu

Pour accéder au shell GRUB 2, appuyez sur C dans l’écran de menu (ou Ctrl+C) dans la fenêtre d’édition). Vous verrez une invite de commande comme ceci : `grub >`

Tapez `help` pour voir une liste de toutes les commandes disponibles, ou appuyez sur Échap pour quitter le shell et revenir à l’écran de menu

ce menu n’apparaîtra pas si `GRUB_TIMEOUT` est paramétré à `0` dans `/etc/default/grub`

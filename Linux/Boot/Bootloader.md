Le chargeur de démarrage le plus populaire pour Linux dans l’architecture x86 est GRUB ( Grand Unified Bootloader )

Dès qu’il est appelé par le BIOS ou par l’UEFI, GRUB affiche une liste de systèmes d’exploitation disponibles au démarrage et la possibilité de choisir sur lequel on veut boot

Il peut arriver que la liste n’apparaisse pas automatiquement, mais elle peut être invoquée en appuyant sur `Maj` pendant que GRUB est appelé par le BIOS. Avec les systèmes UEFI, la touche `Échap` doit être utilisée à la place

## GRUB

## Passer paramètre

Lors du démarrage, il est possible de passer des paramètres au kernel. C'est paramètres ne sont valide que pour ce boot uniquement

La plupart des paramètres du noyau suivent le schéma `option=valeur`. Voici quelques-uns des paramètres les plus pertinents du noyau :

- `acpi`

Active/désactive le support de l’ACPI. `acpi=off` désactivera le support de l’ACPI ( Advanced Configuration and Power Interface ). En somme, la gestion de l'alimentation

- `init`

Définit un initialiseur système alternatif. À titre d’exemple, `init=/bin/bash` définira le shell Bash comme initialiseur. Cela signifie qu’une session shell sera lancée juste après le processus de démarrage du noyau.

- `systemd.unit`

Définit la cible _systemd_ à activer. Par exemple, `systemd.unit=graphical.target`. Systemd accepte également les niveaux d’exécution numériques tels que définis pour _SysV_. Pour activer le niveau d’exécution 1, par exemple, il suffit d’inclure le chiffre `1` ou la lettre `S` (pour “single”) comme paramètre du noyau.

- `mem`

Définit la quantité de RAM disponible pour le système. Ce paramètre est utile pour les machines virtuelles afin de limiter la quantité de RAM disponible pour chaque système invité. L’utilisation de `mem=512M` limitera à 512 mégaoctets la quantité de RAM disponible pour un système invité donné.

- `maxcpus`

Limite le nombre de processeurs (ou cœurs de processeur) visibles pour le système dans les machines à multiprocesseurs symétriques. C’est également valable pour les machines virtuelles. Une valeur de `0` désactive le support des machines multiprocesseurs et a le même effet que le paramètre de noyau `nosmp`. Le paramètre `maxcpus=2` limitera à deux le nombre de processeurs disponibles pour le système d’exploitation. Avec `htop`, les processeurs qui ne sont pas pris en charge sont considéré comme 'offline'

- `quiet`

Masque la plupart des messages de démarrage.

- `vga`

Sélectionne un mode vidéo. Le paramètre `vga=ask` affichera une liste des modes disponibles au choix.

- `root`

Définit la partition racine, distincte de celle pré-configurée dans le chargeur de démarrage. Par exemple, `root=/dev/sda3`.

- `rootflags`

Options de montage pour le système de fichiers racine.

- `ro`

Effectue le montage initial du système de fichiers racine en lecture seule.

- `rw`

Permet l’écriture dans le système de fichiers racine lors du montage initial.

## Ajouter paramètre

Lors du démarrage de la machine, il est possible de modifier le démarrage de l'OS en modifiant les paramètres du bootloader

Dans la fenêtre de sélection ( démarrage, options avancées, etc...), en appuyant sur la touche `e`, il est possible d'ajouter une valeur

L'ajout du paramètre se fait dans la ligne qui commence par `linux`

exemple avec l'ajout de `maxcpus`

```bash
linux /boot/vmlinuz-6.12.101+deb13-amd64 root=UUID=41ecdc02-a293-4223-bdfe-21023478e614 ro splash quiet maxcpus=1
```

Une fois que le système d’exploitation tourne, les paramètres du noyau utilisés pour le chargement de la session en cours sont disponibles en lecture dans le fichier `/proc/cmdline`.

en utilisant un outils comme `htop` par exemple, sur les 2 processeurs alloué à la VM, 1 est assigné avec la mention `offline`

## Contenu bootloader

Le contenu de la partition `/boot` peut varier en fonction de l’architecture système ou du chargeur d’amorçage en vigueur, mais sur un système de type x86, vous trouverez généralement les fichiers ci-dessous.

La plupart d’entre eux sont nommés avec un suffixe `-VERSION` qui représente la version du noyau Linux correspondant. Par conséquent, un fichier de configuration pour le noyau Linux en version `4.15.0-65-generic` serait appelé `config-4.15.0-65-generic`

### Fichier config

Ce fichier, généralement appelé `config-VERSION` (voir l’exemple ci-dessus), contient les paramètres de configuration du noyau Linux. Ce fichier est généré automatiquement lors de la compilation ou de l’installation d’un nouveau noyau et ne doit pas être modifié directement par l’utilisateur

### System.map

Ce fichier est une table de recherche qui fait correspondre les noms de symboles (comme les variables ou les fonctions) à leur position correspondante en mémoire. Il sert à déboguer un type de défaillance du système connue sous le nom de _kernel panic_, étant donné qu’il permet à l’utilisateur de savoir quelle variable ou fonction a été appelée lorsque la défaillance s’est produite. Comme le fichier `config`, le nom est généralement `System.map-VERSION` (par exemple `System.map-4.15.0-65-generic`)

### Noyau Linux

C’est le noyau du système d’exploitation à proprement parler. Le nom est généralement `vmlinux-VERSION` (par exemple `vmlinux-4.15.0-65-generic`). Vous trouverez également le nom `vmlinuz` au lieu de `vmlinux`, le `z` final indiquant que le fichier a été compressé.

### Disque mémoire initial ( Init )

Il est généralement appelé `initrd.img-VERSION` et contient un système de fichiers racine minimal chargé dans un disque RAM ( d'ou `initrd` ), qui contient à son tour les outils et les modules du noyau nécessaires pour que le noyau puisse monter le système de fichiers racine proprement dit

### Fichiers chargeur de démarrage

Sur les systèmes avec GRUB installé, ceux-ci sont généralement rangés dans `/boot/grub` et comprennent le fichier de configuration de GRUB (`/boot/grub/grub.cfg` pour GRUB 2 ou `/boot/grub/menu.lst` dans le cas de GRUB Legacy), les modules (dans `/boot/grub/i386-pc`), les fichiers de traduction (dans `/boot/grub/locale`) et les polices (dans `/boot/grub/fonts`)


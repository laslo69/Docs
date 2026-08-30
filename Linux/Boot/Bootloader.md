Le chargeur de démarrage le plus populaire pour Linux dans l’architecture x86 est GRUB ( Grand Unified Bootloader )

Dès qu’il est appelé par le BIOS ou par l’UEFI, GRUB affiche une liste de systèmes d’exploitation disponibles au démarrage et la possibilité de choisir sur lequel on veut boot

Il peut arriver que la liste n’apparaisse pas automatiquement, mais elle peut être invoquée en appuyant sur `Maj` pendant que GRUB est appelé par le BIOS. Avec les systèmes UEFI, la touche `Échap` doit être utilisée à la place

## Passer paramètre

Lors du démarrage, il est possible de passer des paramètres au kernel. C'est paramètres ne sont valide que pour ce boot uniquement

La modification des paramètres du noyau n’est pas nécessaire en général, mais elle peut s’avérer utile pour détecter et résoudre les problèmes liés au système d’exploitation.

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

Les paramètres du noyau doivent être ajoutés au fichier `/etc/default/grub` à la ligne `GRUB_CMDLINE_LINUX` pour les rendre persistants après chaque redémarrage

exemple avec l'ajout de `maxcpus`

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash maxcpus=4"
```

Un nouveau fichier de configuration pour le chargeur d’amorçage doit être généré à chaque fois que `/etc/default/grub` change, ce qui est effectué par la commande `grub-mkconfig -o /boot/grub/grub.cfg`. Une fois que le système d’exploitation tourne, les paramètres du noyau utilisés pour le chargement de la session en cours sont disponibles en lecture dans le fichier `/proc/cmdline`.

Sur debian/ubuntu/mint :

```bash
update-grub
ou
grub-mkconfig -o /boot/grub/grub.cfg
```

Sur fedora/Alma,Rocky : 

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

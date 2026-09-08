
La version originale de GRUB (_Grand Unified Bootloader_) maintenant connue sous le nom de _GRUB Legacy_ a été développée en 1995 dans le cadre du projet GNU Hurd et a ensuite été adoptée comme chargeur de démarrage par défaut de nombreuses distributions Linux, en remplacement d’alternatives antérieures telles que LILO

GRUB Legacy n’est plus développé activement (la dernière version en date était la 0.97, en 2005), et aujourd’hui la plupart des distributions Linux majeures installent GRUB 2 comme chargeur de démarrage par défaut

## Installer GRUB Legacy

Pour installer GRUB Legacy sur un disque depuis un système en marche, nous utiliserons la commande `grub-install`

La commande de base est `grub-install /dev/PERIPHERIQUE` où `PERIPHERIQUE` est le disque sur lequel vous souhaitez installer GRUB Legacy et non pas la partition comme dans `/dev/sda1`.

```bash
grub-install /dev/sda
```

Par défaut, GRUB va copier les fichiers nécessaires dans le répertoire `/boot` sur le périphérique spécifié

Si vous souhaitez les copier vers un autre répertoire, utilisez le paramètre `--boot-directory=`, suivi du chemin complet vers l’endroit vers lequel les fichiers doivent être copiés

```bash
grub-install --boot-directory=/boot/ /dev/sda
```

## GRUB Legacy rescue shell

Si vous ne pouvez pas démarrer le système pour une raison quelconque et que vous devez réinstaller GRUB Legacy, vous pouvez le faire à partir du shell GRUB sur un disque de démarrage GRUB Legacy

Depuis le shell GRUB (tapez `c` dans le menu de démarrage pour accéder à l’invite `grub>`), la première étape consiste à définir le périphérique de démarrage, qui contient le répertoire `/boot`

Par exemple, si ce répertoire se trouve dans la première partition du premier disque, la commande sera

```bash
grub> root (hd0,0)
```

Si vous ne savez pas quel périphérique contient le répertoire `/boot`, vous pouvez demander à GRUB de le rechercher avec la commande `find`

```bash
grub> find /boot/grub/stage1
(hd0,0)
```

Ensuite, définissez la partition de démarrage comme indiqué ci-dessus et utilisez la commande `setup` pour installer GRUB Legacy dans le MBR et copier les fichiers nécessaires sur le disque

```bash
grub> setup (hd0,0)
```

Une fois terminé, redémarrez le système et il devrait démarrer normalement

## Gerer entrees menu GRUB

Les entrées de menu et les paramètres de GRUB Legacy sont stockés dans le fichier `/boot/grub/menu.lst`

Il s’agit d’un fichier texte simple avec une liste de commandes et de paramètres, qui peut être édité directement avec votre éditeur de texte préféré

Une entrée de menu comporte au moins trois commandes

- La première, `title`, définit le titre du système d’exploitation dans l’écran de menu
- La seconde, `root`, indique à GRUB Legacy le périphérique ou la partition à partir de laquelle il doit démarrer.
- La troisième entrée, `kernel`, spécifie le chemin complet vers l’image du noyau qui doit être chargée lorsque l’entrée correspondante est sélectionnée

Vous devrez éventuellement spécifier l’emplacement du disque mémoire initial pour le système d’exploitation à l’aide du paramètre `initrd`. 

Le chemin complet vers le fichier `initrd` peut être spécifié comme pour le paramètre `kernel`, il faut juste que les 2 fichiers correspondent

Contrairement à GRUB 2, dans GRUB Legacy, les disques _et_ les partitions sont numérotés à partir de zéro. Ainsi, la commande `root (hd0,0)` va définir la partition de démarrage comme la première partition (`0`) du premier disque (`hd0`)

```bash
title Ma distribution Linux
root (hd0,0)
kernel /vmlinuz root=/dev/hda1
initrd /initrd.img
```

Vous pouvez omettre l’instruction `root` si vous spécifiez le périphérique de démarrage en tête du chemin à la commande `kernel`

```bash
kernel (hd0,0) /vmlinuz root=/dev/hda1
```

GRUB Legacy a une conception modulaire, où des modules généralement stockés sous forme de fichiers `.mod` dans `/boot/grub/i386-pc` peuvent être chargés pour ajouter des fonctionnalités supplémentaires, comme le support de matériel, de systèmes de fichiers ou de nouveaux algorithmes de compression inhabituels

Les modules sont chargés en utilisant la commande `module`, suivie du chemin complet vers le fichier `.mod` correspondant. Gardez à l’esprit que, comme pour les noyaux et les images initrd, ce chemin est relatif au périphérique spécifié dans la directive `root`

L’exemple ci-dessous va charger le module `915resolution`, requis pour régler correctement la résolution du framebuffer sur les systèmes équipés de chipsets vidéo Intel de la série 800 ou 900

```bash
module /boot/grub/i386-pc/915resolution.mod
```

## Passer de GRUB à GRUB2

Il est possible de passer à GRUB2 depuis une distribution avec GRUB en utilisant la commande `update-from-grub-legacy`

Le fichier `menu.lst` sera contrôlé et la nouvelle configuration de GRUB mise en place selon le contenu du fichier `menu.lst`
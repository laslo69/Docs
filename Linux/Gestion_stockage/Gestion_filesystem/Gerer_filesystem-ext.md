
Le système de fichiers ext ( Extended Filesystem ) a été le premier système de fichiers pour Linux. Au fil des années, il a été remplacé par de nouvelles versions appelées ext2, ext3 et ext4. Cette dernière version est actuellement le système de fichiers par défaut pour de nombreuses distributions Linux

La différence majeur entre `ext2` et `ext3-4` est, que `ext2` ne possède par de journalisation du filesystem, en cas d'interruption du système, la journalisation permet la récupération instantanée en rejouant ou annulant les transactions du journal, qui est beaucoup plus rapide qu'un filesystem sans journalisation

Il est cependant possible, de convertir une partition `ext2` en `ext3`

## Créer un filesystem

Les outils `mkfs.ext2`, `mkfs.ext3` et `mkfs.ext4` sont utilisés pour créer des systèmes de fichiers ext2, ext3 et ext4. En fait, tous ces "outils" n’existent qu’en tant que liens symboliques vers un autre programme appelé `mke2fs`

`mke2fs` modifie ses paramètres par défaut en fonction du nom par lequel il est invoqué. Ainsi, ils ont tous le même comportement et les mêmes paramètres de ligne de commande

Pour créer un filesystem ext2,3 ou 4, il est possible d'utiliser 3 méthodes :

- mkfs.ext4 /dev/sdx, méthode la plus simple
- mkfs -t ext4 /dev/sdx, méthode plus générique
- mke2fs -t ext4 /dev/sdx, ouitls historique capable de créer uniquement des extensions en format ext2, 3 ou 4

La forme d’utilisation la plus simple est, pour créer un système de fichiers ext4 dans la partition `/dev/sdb1`

```bash
mkfs.ext4 /dev/sdb1
ou
mkfs -t ext4 /dev/sdb1
ou
mke2fs -t ext4 /dev/sdb1
```

### Paramètres mkfs

Si appellé avec la commande `mkfs -t`

- `-t, --type` : spécifie le format du filesystem

### Paramètres mke2fs

`mke2fs` supporte toute une panoplie de paramètres et d’options en ligne de commande. En voici quelques-uns parmi les plus importants

- `-b SIZE` : Fixe la taille des blocs de données dans le périphérique à `SIZE` (taille), qui peut être de 1024, 2048 ou 4096 octets par bloc
- `-j` : Créer le système de fichier avec un journal ext3, Si l'option -J n'est pas indiqué, des paramètres par défaut seront utilisés pour le dimensionnement du journal
- `-L VOLUME_LABEL` : Fixe le nom du volume à celui spécifié dans `VOLUME_LABEL`
- `-n` : Simule la création du système de fichiers et affiche ce qui se passerait si elle était exécutée sans l’option `n`. Considérez-la comme un mode "test" qui permet de bien vérifier les choses avant d’effectuer des changements sur le disque
- `-q` : Mode silencieux. `mke2fs` s’exécutera normalement, mais n’affichera rien dans le terminal
- `-t TYPE` : Indique le type de filesystem qui doit être crée
- `-U ID` : Définit l’UUID de la partition à la valeur spécifiée par ID. Au lieu d’un ID, vous pouvez également spécifier des paramètres comme `clear` pour supprimer l’UUID du système de fichiers, `random` pour utiliser un UUID généré de manière aléatoire, ou `time` pour générer un UUID basé sur l’heure
- `-v` : Le mode verbeux permet d’afficher beaucoup plus d’informations que d’habitude pendant l’opération. Utile pour le débogage

## Redimensionner filesystem

Pour redimensionner un système de fichiers `ext4` sous Linux, on utilise l'outil `resize2fs` associé à la modification de la partition

### Augmenter un filesystem

Un agrandissement du filesystem peut se faire à chaud, pendant que le filesystem est monté et utilisé, il est fortement recommandé de démonter la partition avant d'opérer une modification au risque de perte de donnée

Par défaut, si aucun paramètre pour donner le volume à ajouter est spécifié, le filesystem va prendre la totalité de la place disponible de la partition

Il faut au préalable, augmenter la taille de la partition via `fdisk` ou `gdisk`, 

```bash
resize2fs /dev/sdb1
```

### Réduire un filesystem

Point important !, `fdisk` et `gdisk` ne savent pas gérer une partition avec un filesystem, la réduction d'une partition est destructive, ce qui insinu que toutes données sortant de la limite du filesystem sont perdu. La méthode la plus simple et de supprimé la partition et la créer à nouveau à la taille voulu

Pour réduire un filesystem, l'opération doit impérativement se faire à froid pour éviter une perte de données

il faut donc démonter le filesystem avec la command `umount`

ensuite, faire un contrôle du filesystem et ensuite réduire la taille occupé par le filesystem

```bash
e2fsck -f /dev/sdb1
resize2fs /dev/vdb1 10G
```

### Paramètre resize2fs

- `-M` : Reduire le filesystem pour minimiser sa taille autant que possible en tenant en compte des fichiers enregistres
- `-p` : affiche une barre de progression pour chaque phase de resize2fs lors d'un redimensionnement à froid


Un lien physique est un deuxième nom pour le fichier d’origine. Il ne s’agit pas d’un doublon, mais d’une entrée supplémentaire dans le système de fichiers qui pointe vers le même inode sur le disque

Un inode est une structure de données qui stocke les attributs d’un objet (comme un fichier ou un répertoire) dans un système de fichiers. Parmi ces attributs figurent les droits d’accès, les propriétaires et les blocs du disque sur lesquels les données de l’objet sont stockées. Il s’agit d’une entrée dans un index, d’où le nom qui vient de index node

Contrairement aux liens symboliques, vous pouvez créer des liens physiques vers des fichiers uniquement, et le lien et la cible doivent résider dans le même système de fichiers

## Créer lien physique

La commande pour créer un lien physique sous Linux est `ln`

La cible doit déjà exister (c’est le fichier vers lequel le lien pointera), et si la cible n’est pas dans le répertoire courant, ou si vous voulez créer le lien ailleurs, vous devez impérativement spécifier le chemin complet vers ce fichier

```bash
ln target.txt /home/carol/Documents/hardlink
```

va créer un fichier nommé `hardlink` dans le répertoire `/home/carol/Documents/`, lié au fichier `target.txt` dans le répertoire courant

## Gérer les liens physiques

Les liens physiques sont des entrées du système de fichiers avec des noms différents mais qui pointent vers les mêmes données sur le disque

Tous ces noms se valent et peuvent être utilisés pour faire référence à un fichier. Si vous modifiez le contenu de l’un des noms, le contenu de tous les autres noms pointant vers ce fichier change puisque tous les noms pointent vers les mêmes données

Si vous supprimez l’un des noms, les autres noms continueront à fonctionner

En effet, lorsque vous "supprimez" un fichier, les données ne sont pas réellement effacées du disque. Le système supprime simplement l’entrée de la table du système de fichiers pointant vers l'_inode_ correspondant aux données sur le disque

Mais si vous avez une deuxième entrée pointant vers le même _inode_, vous pouvez toujours accéder aux données

Vous pouvez vérifier en utilisant l’option `-i` de `ls`, pour afficher les `inodes`

```bash
ls -li
total 224
3806696 -r--r--r-- 2 carol carol 111702 Jun  7 10:13 hardlink
3806696 -r--r--r-- 2 carol carol 111702 Jun  7 10:13 target.txt
```

Le nombre qui précède les droits d’accès est le numéro de l'inode

le fichier `hardlink` et le fichier `target.txt` ont le même numéro (`3806696`) ? C’est parce que l’un est un lien physique de l’autre

Mais lequel est l’original et lequel est le lien ? Impossible de le savoir, dans la mesure où les deux sont identiques sur le plan pratique

Notez que chaque lien physique pointant vers un fichier augmente le compte de liens (_link count_) du fichier. C’est le nombre situé juste après les droits d’accès dans l’affichage de `ls -l`

Par défaut, chaque fichier a un compte de liens de `1` (les répertoires ont un compte de `2`), et chaque lien physique pointant vers lui augmente d’un cran ce compte. Ce qui explique le nombre de liens `2` pour les fichiers de la liste ci-dessus

## Déplacer et supprimer des liens physiques

Étant donné que les liens physiques sont traités comme des fichiers normaux, ils peuvent être supprimés avec `rm` et renommés ou déplacés dans le système de fichiers avec `mv`

Et comme un lien physique pointe vers le même _inode_ que la cible, il peut être déplacé librement, sans risquer de "casser" le lien
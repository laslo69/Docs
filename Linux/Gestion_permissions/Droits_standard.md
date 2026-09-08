
En tant que système multi-utilisateurs, Linux doit disposer d’un mécanisme permettant de savoir à qui appartient chaque fichier et si tel ou tel utilisateur est autorisé à effectuer des actions sur un fichier

Cela permet de garantir la confidentialité des utilisateurs qui voudraient protéger le contenu de leurs fichiers, et aussi d’assurer la collaboration en rendant certains fichiers accessibles à plusieurs utilisateurs

Pour ce faire, un système de permissions à trois niveaux est mis en place. Chaque fichier sur le disque appartient à un utilisateur et à un groupe d’utilisateurs et possède trois jeux de permissions : 

- un pour son propriétaire
- un pour le groupe qui détient le fichier
- un pour tous les autres

Dans l’affichage de `ls -l`, les droits d’accès aux fichiers sont indiqués juste après le type de fichier, sous la forme de trois groupes de trois caractères chacun, dans l’ordre `r`, `w` et `x`, un tiret `-` représente l’absence d’une permission

***

## Droits fichier

 Droits d’accès aux fichiers

- `r` : Signifie read avec une valeur octale de `4` (ne vous inquiétez pas, nous en parlerons prochainement). Cela correspond à la permission d’ouvrir un fichier et d’en lire le contenu.
- `w` : Signifie write avec une valeur octale de `2`. Cela équivaut au droit de modifier le contenu d’un fichier.
- `x` : Signifie execute avec une valeur octale de `1`. Cela signifie que le fichier peut être lancé comme un exécutable ou un script

***

## Droits dossier

- `r` : Signifie read  avec une valeur octale de `4`. Cela représente le droit de lire le contenu du répertoire, comme les noms des fichiers. En revanche, cela n’implique pas forcément la permission de lire le contenu des fichiers eux-mêmes.
- `w` : Signifie write avec une valeur octale de `2`. Cela signifie que l’on peut créer ou supprimer des fichiers dans le répertoire en question. Notez bien que vous ne pouvez pas effectuer ces changements avec les seules permissions _write_, mais qu’il vous faut également la permission `x` pour modifier le répertoire.
- `x` : Signifie execute avec une valeur octale de `1`. Cela signifie que l’on a la permission d’entrer dans un répertoire, mais pas d’en afficher les fichiers (pour cela, il faut `r`).


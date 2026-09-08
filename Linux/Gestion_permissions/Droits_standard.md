
En tant que système multi-utilisateurs, Linux doit disposer d’un mécanisme permettant de savoir à qui appartient chaque fichier et si tel ou tel utilisateur est autorisé à effectuer des actions sur un fichier, permet de garantir la confidentialité des utilisateurs qui voudraient protéger le contenu de leurs fichiers, et aussi d’assurer la collaboration en rendant certains fichiers accessibles à plusieurs utilisateurs

## Explication droits

Pour ce faire, un système de permissions à trois niveaux est mis en place. Chaque fichier sur le disque appartient à un utilisateur et à un groupe d’utilisateurs et possède trois jeux de permissions : 

- Propriétaire
- Groupe
- Others

Le premier des trois chiffres du jeu de permissions représente les droits de l’utilisateur (`u`), le deuxième ceux du groupe (`g`) et le troisième ceux de tous les autres (`o`)

Dans l’affichage de `ls -l`, les droits d’accès aux fichiers sont indiqués juste après l'inode, sous la forme de trois groupes de trois caractères chacun, dans l’ordre `r`, `w` et `x`, un tiret `-` représente l’absence d’une permission

Les droits peuvent s'écrire de 2 manières mais aboutissent au même résultat

- octal
- symbolique

### Droits fichier

 Droits d’accès aux fichiers

- `r` : Signifie read avec une valeur octale de `4`, correspond à la permission d’ouvrir un fichier et d’en lire le contenu.
- `w` : Signifie write avec une valeur octale de `2`, équivaut au droit de modifier le contenu d’un fichier.
- `x` : Signifie execute avec une valeur octale de `1`, signifie que le fichier peut être lancé comme un exécutable ou un script

### Droits dossier

- `r` : Signifie read  avec une valeur octale de `4`, représente le droit de lire le contenu du répertoire, comme les noms des fichiers. En revanche, cela n’implique pas forcément la permission de lire le contenu des fichiers eux-mêmes.
- `w` : Signifie write avec une valeur octale de `2`, signifie que l’on peut créer ou supprimer des fichiers dans le répertoire en question. Notez bien que vous ne pouvez pas effectuer ces changements avec les seules permissions `write`, mais qu’il vous faut également la permission `x` pour modifier le répertoire.
- `x` : Signifie execute avec une valeur octale de `1`, signifie que l’on a la permission d’entrer dans un répertoire, mais pas d’en afficher les fichiers (pour cela, il faut `r`).

## Modes permissions

La commande `chmod` est utilisée pour modifier les droits d’accès d’un fichier. Elle prend au moins deux paramètres : le premier décrit les droits à modifier, le second désigne le fichier ou le répertoire dans lequel la modification sera effectuée

seul le propriétaire du fichier ou l’administrateur du système (`root`) peut modifier les droits d’un fichier

### Mode Octal

En mode octal, les permissions sont spécifiées différemment, sous forme d’une valeur à trois chiffres

Lorsqu’une valeur de permission est impaire, le fichier est forcément exécutable

Chaque permission a une valeur correspondante, et elles sont spécifiées dans l’ordre suivant : 

- lecture (`r`) : `4`
- l’écriture (`w`) :`2`
- exécution (`x`) : `1`
- pas de permissions : `0`

Ainsi, une permission de `rwx` correspondrait à `7` (`4+2+1`), `r-x` à `5` (`4+0+1`), etc...

`660` équivaut à rw- rw- ---

```bash
chmod 660 text.txt
ls -l text..txt
-rw-rw---- 1 carol carol 765 Dec 20 21:25 text.txt
```

### Mode Symbolique

En mode symbolique, le(s) premier(s) caractère(s) indique(nt) les droits que vous allez modifier : 

- ceux de l’utilisateur (`u`)
- du groupe (`g`)
- des autres (`o`)
- de tout le monde (`a`).

Ensuite, il vous faut indiquer à la commande ce qu’elle doit faire : 

- accorder un droit (`+`)
- révoquer un droit (`-`)
- lui donner une valeur spécifique (`=`).

Enfin, vous indiquez le droit sur lequel vous souhaitez agir : 

- lecture (`r`)
- écriture (`w`)
- exécution (`x`).

```bash
chmod u+x text.txt
ls
-rwxrw---- 1 carol carol 765 De 20 21:25 text.txt
```

il est possible de modifier plusieurs droits en même temps. Dans ce cas, il faut les séparer par une virgule (`,`)

```bash
chmod ug+rw-x,o-rw text.txt
ls
-rw-rw---- 1 carol carol 765 De 20 21:25 text.txt
```


## Permissions par defaut

Lors de la création d'un fichier, par défaut un permission est établie

```bash
touch testfile
```

Jetons un coup d’œil sur les permissions de ce fichier

```bash
ls -lh testfile
-rw-rw-r-- 1 carol carol 0 jul 13 21:55 testfile
```

Les permissions sont `rw-r—​r--` : lecture et écriture pour l’utilisateur, et lecture pour le groupe et les autres, ou `644` en mode octal

Pareil pour un dossier, mais les permissions sont différentes

```bash
mkdir testdir
ls -lhd testdir
drwxrwxr-x 2 carol carol 4,0K jul 13 22:01 testdir
```

Les permissions sont maintenant `rwxr-xr-x` : lecture, écriture et exécution pour l’utilisateur, lecture et exécution pour le groupe et les autres, ou `755` en mode octal

Ces permissions par défaut doivent bien venir de quelque part mais, où?

Elles proviennent du `user mask` ou `umask`, qui définit les permissions par défaut pour chaque fichier créé

```bash
umask 
0002
umask -S
u=rwx,g=rwx,o=rx
```

par défaut, `umask` à une valeur de 0002

Ce sont là les mêmes droits que ceux accordés à notre répertoire de test dans l’un des exemples ci-dessus, pourtant, les fichiers et dossiers créer ne possède pas les mêmes droits

cela n’a pas de sens de définir par défaut des droits d’exécution globaux pour tout le monde sur n’importe quel fichier, les répertoires en revanche ont besoin de droits d’exécution (sinon vous ne pouvez pas y entrer), mais les fichiers n’en ont pas, donc ils ne les obtiennent pas. D’où le `rw-w-​r--`

En plus d’afficher les droits par défaut, `umask` peut également être utilisé pour les modifier dans la session en cours de l’interpréteur de commandes

```bash
umask u=rwx,g=rwx,o=
```

Chaque nouveau répertoire va hériter des droits `rwxrwx---`, et chaque fichier `rw-rw----` (car ils n’ont pas les droits d’exécution)

Il est possible de calculer le umask

Prendre en valeur par défaut:

- 666 pour les fichiers
- 777 pour les dossiers

umask va retirer la valeur demandé, à la valeur par défaut des fichiers ou dossier

par exemple:

Un fichier que je veut avec les droits `644` ( soit lecture écriture/lecture/lecture) aura une valeur de umask `022` ( 666 - 022 ) va supprimer de la valeur par défaut la valeur octal 2 ( correspond à écriture )

|Valeurs|Droits pour les fichiers|Droits pour les répertoires|
|---|---|---|
|`0`|`rw-`|`rwx`|
|`1`|`rw-`|`rw-`|
|`2`|`r--`|`r-x`|
|`3`|`r--`|`r--`|
|`4`|`-w-`|`-wx`|
|`5`|`-w-`|`-w-`|
|`6`|`---`|`--x`|
|`7`|`---`|`---`|

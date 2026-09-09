
Liens symboliques, également appelés soft links, pointent vers le chemin d’accès d’un autre fichier. Si vous supprimez le fichier vers lequel pointe le lien ( appelé cible ), ce dernier existera toujours, mais il ne fonctionnera plus, car il pointera désormais vers rien

## Creer line symbolique

La commande utilisée pour créer un lien symbolique est `ln`, mais avec l’ajout de l’option `-s`

syntaxe :

```bash
ln -s target.txt /home/carol/Document/softlink
```

Cette opération va créer un fichier nommé `softlink` dans le répertoire `/home/carol/Documents/`, pointant vers le fichier `target.txt` dans le répertoire courant

Vous pouvez créer des liens symboliques vers des fichiers aussi bien que des répertoires, même sur des partitions différentes. Il est assez facile de repérer un lien symbolique dans l’affichage de la commande `ls`

```bash
ls -lh
total 112K
-rw-r--r-- 1 carol carol 110K Jun  7 10:13 target.txt
lrwxrwxrwx 1 carol carol   12 Jun  7 10:14 softlink -> target.txt
```

le premier caractère avant les droits d’accès au fichier `softlink` est `l`, ce qui signifie qu’il s’agit d’un lien symbolique, par ailleurs juste après le nom du fichier, le nom de la cible vers laquelle pointe le lien, en l’occurrence le fichier `target.txt`

Les droits d’accès héritent toujours des mêmes permissions que la cible. L'affichage du lien symbolique peut afficher rwxrwxrwx, mais le fichier original est -rw-r--r--. Même si le lien est en 777, l'héritage fait en sorte que le fichier soit en 644

## Déplacer et supprimer lien symbolique

Les liens symboliques peuvent être supprimés avec `rm` et déplacés ou renommés avec `mv`. Cependant, il faut faire très attention lors de leur création, pour éviter de "casser" le lien lorsqu’on le déplace depuis son emplacement d’origine

Lorsque vous créez des liens symboliques, sachez qu’à moins qu’un chemin d’accès ne soit complètement spécifié, l’emplacement de la cible est interprété comme étant relatif à l’emplacement du lien, ce qui peut poser des problèmes quand on déplace le lien ou le fichier vers lequel il pointe

Un exemple permet de mieux comprendre ce principe. Supposons que vous ayez un fichier nommé `original.txt` dans le répertoire courant, et que vous vouliez créer un lien symbolique appelé `softlink` vers ce fichier

```bash
ln -s original.txt softlin
```

Apparemment, tout va bien. Vérifions avec `ls`

```bash
ls -lh
total 112K
-r--r--r-- 1 carol carol 110K Jun  7 10:13 original.txt
lrwxrwxrwx 1 carol carol   12 Jun  7 19:23 softlink -> original.txt
```

Le lien est construit : `softlink` pointe vers (`→`) `original.txt`. Maintenant, voyons ce qui se passe lorsque vous déplacez le lien vers le répertoire parent et que vous essayez d’afficher son contenu en utilisant la commande `less`

```bash
mv softlink ../
less ../soflink
../softlink: No such file or directory
```

Comme le chemin vers `original.txt` n’a pas été spécifié, le système part du principe qu’il se situe dans le même répertoire que le lien. À partir du moment où ce n’est plus le cas, le lien ne fonctionne plus

Pour éviter cette situation, il suffit de spécifier le chemin d’accès complet à la cible lors de la création du lien

```bash
ln -s /home/carol/Documents/original.txt softlink
```

Ainsi, quel que soit l’endroit où vous déplacez le lien, il fonctionnera toujours, étant donné qu’il pointe vers l’emplacement absolu de la cible

```bash
ls -lh
total 112K
lrwxrwxrwx 1 carol carol   40 Jun  7 19:34 softlink -> /home/carol/Documents/original.txt
```

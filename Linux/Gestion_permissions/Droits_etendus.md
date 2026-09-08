
En dehors des droits de lecture, d’écriture et d’exécution pour l’utilisateur, le groupe et les autres, chaque fichier peut comporter trois autres droits d’accès étendus qui peuvent modifier le fonctionnement d’un répertoire ou l’exécution d’un programme

Il est possible, tout comme avec les permissions standard, de passer plusieurs paramètres en même temps

Par exemple, de passer le SUID et SGID

```bash
chmod 6755 Sample_Directory
ou
chmod u,g+s Sample_Directory
```

## Sticky Bit

Le _sticky bit_, a la valeur octale `1`, en mode symbolique, il est représenté par un `t` dans les permissions des utilisateurs " others "

Le sticky bit s’applique aux répertoires et n’a aucun effet sur les fichiers normaux, il empêche les utilisateurs de supprimer ou de renommer un fichier dans un répertoire à moins qu’ils ne soient propriétaires de ce fichier ou de ce répertoire

- Le `t` minuscule signifie que le sticky bit est activé et que les autres ont les droits d’exécution
- Le `T` majuscule signifie que le _sticky bit_ est activé et que les autres n’ont pas les droits d’exécution
- Le `t` ou `T` remplace le droit d'exécution `x` pour les utilisateurs `others`

Pour activer le sticky bit  (valeur `1`) pour le répertoire `Another_Directory` en mode octal avec les permissions `755`, la commande serait

```bash
chmod 1755 Another_Directory
ou
chmod +t Another_Directory
ls -l Another_Directory
drwxr-xr-t 2 carol carol 4096 Dec 20 18:46 Sample_Directory/
22027395 drwx-----T  2 root root 4.0K Sep  8 23:06  test
```

Pour retirer le `sticky bit`

```bash
chmod 0755 Another_Directoy
ou
chmod -t Another_Directory
```

## SUID

SUID, également connu sous le nom de Set User ID, a une valeur octale `4` et une valeur symbolique `s` sur les droits de user, il s’applique aux fichiers et n’a aucun effet sur les répertoires

un `S` majuscule indique que l'utilisateur qui possède le fichier n'a pas les droits d'exécution, les fichiers avec le bit SUID présentent un `s` à la place du `x` au niveau des droits de l’utilisateur

Le processus s’exécutera avec les privilèges de l'utilisateur propriétaire du fichier

```bash
chmod 4755 test.sh
ou
chmod u+s test.sh
ls -l test.sh
-rwsr-xr-x 1 carol carol 33 Dec 11 10:36 test.sh
-rwSr-xr-x 1 root root 33 Dec 11 10:36 test2.sh
```

Pour supprimer le `SUID`

```bash
chmod 0755 test.sh
ou
chmod u-s test.sh
```

## SGID

SGID, également connu sous le nom de Set GID ou Set Group ID bit, a la valeur octale `2` et une valeur symbolique de `s`, ce bit s’applique aux fichiers exécutables ou aux répertoires.

- Un `s` minuscule sur le deuxième lot de permissions signifie que le groupe propriétaire du fichier a des droits d’exécution et que le bit SGID est activé.
- Un `S` majuscule signifie que le groupe propriétaire du fichier n’a pas les droits d’exécution (`-`)

Dans le cas des fichiers, le processus s’exécutera avec les privilèges du groupe propriétaire du fichier. Lorsqu’il est appliqué à un répertoire, il fait en sorte que chaque fichier ou répertoire créé à l’intérieur hérite du groupe du répertoire parent

```bash
chmod 4755 test.sh
ou
chmod g+s test.sh
ls -l test.sh
-rwxr-sr-x 1 carol root     33 Dec 11 10:36 test.sh
```


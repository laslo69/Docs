
Les bibliothèques partagées, également connues sous le nom shared objects sont des bouts de code compilés et réutilisables comme des fonctions ou des classes, et qui sont utilisés de manière récurrente par différents programmes

- /etc/ld.so.conf.d/ : contient les fichiers *.conf
- /etc/ld.so.conf.d/mylib.conf : contient une ou plusieurs références vers des dossiers de librairies ( /lib/mylib)
- le dossier ciblé contient les librairies partagé nécessaire pour le programme ( /lib/mylib -> /lib/mylib/lib1.so.1, /lib/mylib/lib2.so.1)

Pour construire un fichier exécutable à partir du code source d’un programme, deux étapes importantes sont nécessaires. Pour commencer, le compilateur transforme le code source en code machine qui est stocké dans ce qu’on appelle des fichiers objets

Ensuite, l’éditeur de liens ( linker ) combine les fichiers objets et les lie aux bibliothèques afin de générer le fichier exécutable final

Cette édition des liens peut se faire de manière statique ou dynamique. Selon la méthode choisie, nous parlerons de bibliothèques statiques ou, dans le cas d’une édition de liens dynamique, de bibliothèques partagées. Expliquons leurs différences

## Bibliotheque statique

Une bibliothèque statique est intégrée au programme au moment de la création du lien.

Une copie du code de la bibliothèque est embarquée dans le programme pour en faire partie. Ainsi, le programme ne dépend pas de la bibliothèque au moment de l’exécution car il contient déjà le code de la bibliothèque.

L’absence de dépendances peut être considérée comme un avantage, étant donné que vous n’avez pas à vous soucier de la disponibilité des bibliothèques utilisées.

L’inconvénient, c’est que les programmes liés statiquement sont plus lourds

## Bibliotheque dynamique

Dans le cas des bibliothèques partagées, l’éditeur de liens veille simplement à ce que le programme référence correctement les bibliothèques

En revanche, l’éditeur de liens ne copie aucun code de la bibliothèque dans le fichier du programme. Au moment de l’exécution, la bibliothèque partagée doit cependant être disponible pour satisfaire les dépendances du programme

C’est donc une approche économique de la gestion des ressources du système qui permet de réduire la taille des fichiers des programmes en ne chargeant qu’une seule copie de la bibliothèque en mémoire, même si elle est utilisée par plusieurs programmes

## Convention nommage

Le nom d’une bibliothèque partagée ( soname ) respecte un schéma composé de trois éléments :

- Nom de la bibliothèque (normalement préfixé par `lib`)
- `so` (qui signifie “shared object”)
- Numéro de version de la bibliothèque

Exemple : `libpthread.so.0`

En comparaison, les noms des bibliothèques statiques se terminent en `.a`, 

Exemple : `libpthread.a`

La `glibc` ( GNU C library ) est un bon exemple de bibliothèque partagée, sur un système Debian GNU/Linux 9.9, son fichier est nommé `libc.so.6`. Ces noms de fichiers plutôt génériques sont normalement des liens symboliques qui pointent vers le fichier réel contenant une bibliothèque, dont le nom contient le numéro de version exact

```bash
ls -l /lib/x86_64-linux-gnu/libc.so.6
lrwxrwxrwx 1 root root 12 feb  6 22:17 /lib/x86_64-linux-gnu/libc.so.6 -> libc-2.24.so
```

Cette façon de référencer les fichiers de bibliothèques partagées nommés selon une version spécifique par des noms de fichiers plus généraux constitue une pratique courante

Emplacements habituels des bibliothèques partagées dans un système Linux

- `/lib`
- `/lib32`
- `/lib64`
- `/usr/lib`
- `/usr/local/lib`

## Chercher dependance d'un executable

Pour rechercher les bibliothèques partagées requises par un programme spécifique, utilisez la commande `ldd` suivie du chemin absolu vers le programme

Le résultat indique le chemin du fichier de la bibliothèque partagée ainsi que l’adresse mémoire hexadécimale à laquelle il est chargé

```bash
ldd /usr/bin/git
    linux-vdso.so.1 =>  (0x00007ffcbb310000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f18241eb000)
	libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f1823fd1000)
	libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007f1823db6000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f1823b99000)
	librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f1823991000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f18235c7000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f182445b000)
```

Si une bibliothéque est manquante, le messa `-> not found` sera affiché

De même, nous utilisons `ldd` pour rechercher les dépendances d’un objet partagé

```bash
ldd /lib/x86_64-linux-gnu/libc.so.6
	/lib64/ld-linux-x86-64.so.2 (0x00007fbfed578000)
	linux-vdso.so.1 (0x00007fffb7bf5000)
```

Avec l’option `-u` (ou `--unused`), `ldd` affiche les dépendances directes inutilisées (si elles existent)

```bash
ldd -u /usr/bin/git
Unused direct dependencies:
	/lib/x86_64-linux-gnu/libz.so.1
	/lib/x86_64-linux-gnu/libpthread.so.0
	/lib/x86_64-linux-gnu/librt.so.1
```

La présence de dépendances inutilisées est liée aux options utilisées par l’éditeur de liens lors de la construction du binaire. Même si le programme n’a pas besoin d’une bibliothèque inutilisée, il a quand même été lié et étiqueté comme `NEEDED` (requis) dans les informations sur le fichier objet

`ldd` lit dans l'ordre :

- `LD_LIBRARY_PATH`
- `/etc/ld.so.cache/`
- `/lib` et `/usr/lib`

Vous pouvez explorer ce sujet en utilisant des commandes comme `readelf` ou `objdump`

- `readelf` : Afficher les informations sur les fichiers ELF ( Executable and Linkable Format )
- `objdump` : Afficher les informations des fichiers objets

## Configuration des chemins de bibliothèques partagées

Les références contenues dans les programmes liés dynamiquement sont résolues par le chargeur de liens dynamiques (`ld.so` ou `ld-linux.so`) lorsque le programme est exécuté

Le chargeur de liens dynamiques recherche les bibliothèques dans une série de répertoires. Ces répertoires sont spécifiés par le chemin des bibliothèques

Le chemin des bibliothèques est configuré dans le répertoire `/etc`, à savoir dans le fichier `/etc/ld.so.conf` et, plus couramment de nos jours, dans les fichiers qui se trouvent dans le répertoire `/etc/ld.so.conf.d`

le premier ne comprend qu’une seule ligne `include` pour les fichiers `*.conf`

```bash
cat /etc/ld.so.conf
include /etc/ld.so.conf.d/*.conf
```

Le répertoire `/etc/ld.so.conf.d` contient des fichiers `*.conf`

```bash
cat /etc/ld.so.conf.d/
fakeroot-x86_64-linux-gnu.conf  x86_64-linux-gnu.conf
i386-linux-gnu.conf             zz_i386-biarch-compat.conf
libc.conf       
```

Ces fichiers `*.conf` doivent inclure les chemins absolus vers les répertoires des bibliothèques partagées :

```bash
cat /etc/ld.so.conf.d/x86_64-linux-gnu.conf
# Multiarch support
/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu
```

- `/etc/ld.so.conf` pointe vers tous les fichiers .conf (*.conf) dans le dossier `/etc/ld.so.conf.d/`
- Les fichiers dans le dossier `/etc/ld.so.conf.d/` pointent vers les librairies utilisé par le linker

La commande `ldconfig` se charge de lire ces fichiers de configuration, de créer l’ensemble des liens symboliques ci-dessus qui aident à localiser les différentes bibliothèques et enfin de mettre à jour le fichier cache `/etc/ld.so.cache`

Ainsi, `ldconfig` doit être exécuté chaque fois que des fichiers de configuration sont ajoutés ou mis à jour. Le cache se construit à partir de `ld.so.conf/` lors de l'exécution de la commande `ldconfig` pour charger le contenu de `/etc/ld.so.conf.d/` et va générer un lien symbolique vers la bibliothéque.

quelques options utiles pour `ldconfig` :

`-v`, `--verbose`

Afficher les numéros de version des bibliothèques, le nom de chaque répertoire et les liens qui sont créés

```bash
ldconfig -v
/usr/local/lib:
/lib/x86_64-linux-gnu:
    libnss_myhostname.so.2 -> libnss_myhostname.so.2
	libfuse.so.2 -> libfuse.so.2.9.7
	libidn.so.11 -> libidn.so.11.6.16
	libnss_mdns4.so.2 -> libnss_mdns4.so.2
	libparted.so.2 -> libparted.so.2.0.1
	(...)
```

`-p`, `--print-cache`

Afficher les listes de répertoires et de bibliothèques candidates stockées dans le cache actuel, notez comment le cache utilise le _soname_ pleinement qualifié des liens

```bash
ldconfig -p
1094 libs found in the cache `/etc/ld.so.cache'
    libzvbi.so.0 (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libzvbi.so.0
	libzvbi-chains.so.0 (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libzvbi-chains.so.0
	libzmq.so.5 (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libzmq.so.5
	libzeitgeist-2.0.so.0 (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libzeitgeist-2.0.so.0
	(...)
```

`-N` ne reconstruit pas le cache de `ldd`

`X` ne met pas à jour les liens symbolique

Si nous faisons un listing détaillé de `/lib/x86_64-linux-gnu/libfuse.so.2`, nous voyons la référence au fichier d’objet partagé réel `libfuse.so.2.9.7` stocké dans le même répertoire

```bash
ls -l /lib/x86_64-linux-gnu/libfuse.so.2
lrwxrwxrwx 1 root root 16 Aug 21  2018 /lib/x86_64-linux-gnu/libfuse.so.2 -> libfuse.so.2.9.7
```

En complément des fichiers de configuration décrits ci-dessus, la variable d’environnement `LD_LIBRARY_PATH` peut être utilisée pour ajouter temporairement de nouveaux chemins pour les bibliothèques partagées

Elle est constituée d’un ensemble de répertoires séparés par deux points (`:`) où les bibliothèques sont recherchées

 Par exemple, pour ajouter `/usr/local/mylib` au chemin des bibliothèques dans la session shell courante

```bash
LD_LIBRARY_PATH=/usr/local/mylib
echo $LD_LIBRARY_PATH
usr/local/mylib
```

Pour ajouter `/usr/local/mylib` au chemin des bibliothèques dans la session shell actuelle en l’exportant vers tous les processus enfants créés à partir de ce shell

```bash
export LD_LIBRARY_PATH=/usr/local/mylib
```

Pour rendre les changements permanents, vous pouvez écrire la ligne dans un des scripts d’initialisation de Bash tels que `/etc/bash.bashrc` ou `~/.bashrc`

Pour supprimer la variable d’environnement `LD_LIBRARY_PATH`, il suffit de taper

```bash
unset LD_LIBRARY_PATH
```

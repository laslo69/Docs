
## Fichier

- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- `/etc/gshadow`
- `/etc/default/useradd`
- `/etc/skel`
- `/etc/subgid`
- `/etc/subuid`
- `/etc/login.defs`

## Voir groupes

Pour voir dans quels groupes, l'utilisateur est présent

```bash
group
ou
id
```

la commande `group` va afficher la liste de groupe dont l'utilisateur est membre, mais uniquement le nom

La commande `id` va afficher  selon les paramètres, certaines informations

`id -g` affiche seulement le groupe de l'utilisateur ( seulement GID )
`id -G` affiche tout les groupes dont l'utilisateur est membres ( seulement GID )
`id -n` à utiliser avec `-g` ou `-G`, permet de ne donner que le nom du groupe

## Ajouter utilisateur

Lorsque vous utilisez la commande `useradd`, les informations sur les utilisateurs et les groupes stockées dans les bases de données de mots de passe et de groupes sont mises à jour pour le compte utilisateur nouvellement créé et, si cela est spécifié, le répertoire personnel du nouvel utilisateur est également créé. Un groupe portant le même nom que le nouveau compte utilisateur est également créé

```bash
useradd -c test -m -k /etc/skel/ -s /bin/bash -p $(openssl passwd -6 test) test
```

`-c, --comment`

Créer un nouveau compte utilisateur avec un commentaire personnalisé (par exemple le nom complet de l’utilisateur).

`-e, --expiredate`

Créer un nouveau compte utilisateur en précisant la date à laquelle il sera désactivé.

`-f, --inactive`

Créer un nouveau compte utilisateur en définissant le nombre de jours après l’expiration du mot de passe pendant lesquels l’utilisateur devra mettre à jour son mot de passe (faute de quoi le compte sera désactivé)

une valeur à 0 désactive l'expiration du mot de passe

`-g, --gid`

Créer un nouveau compte utilisateur avec un GID spécifique.

`-G, --groups`

Créer un nouveau compte utilisateur en l’ajoutant à plusieurs groupes secondaires.

`-k, --skel`

Créer un nouveau compte utilisateur en copiant les fichiers squelette `/etc/skel` par défaut depuis un répertoire donné (cette option n’est valide que si l’option `-m` ou `--create-home` est spécifiée). Si `-k` n'est pas spécifié, la valeur par défaut `/etc/skel` est utilisé mais peut être modifié avec la variable `SKEL` dans le fichier `/etc/default/useradd`

`-m, --create-home`

Créer un nouveau compte utilisateur avec son répertoire personnel (s’il n’existe pas).

`-M, --no-create-home`

Créer un nouveau compte utilisateur sans le répertoire personnel.

`-p, --password`

Defini un mot de passer pour le compte utilisateur crée, doit être encrypté. Sans cette option défini, le compte n'a pas de mot de passe et est bloqué. Dans le fichier `/etc/password`, l'utilisateur aura pour mot de passe `!`. Eviter cette option car lisible

Utilisation de `passwd` juste après pour éviter d'afficher le mot de passe dans un script ou, utilisation de openssl pour générer le mot de passe chiffré avec la commande `useradd -p $(openssl passwd -6 "motdepasse")`

`-r, --system` 

Crée un compte utilisateur système, sans information d'ancienneté de mot de passe. Pas de répertoire `/home/` crée pour l'utilisateur systeme

`-s, --shell`

Créer un nouveau compte utilisateur avec un shell de connexion spécifique.

## Modifier compte utilisateur

Parfois, il vous faut changer un paramètre d’un compte utilisateur existant, comme le nom d’utilisateur (_login name_), le shell de connexion, la date d’expiration du mot de passe et ainsi de suite.

```bash
cat /etc/passwd
test:x:1001:1001:test:/home/test:/bin/bash

usermod -c updated

cat /etc/passwd
test:x:1001:1001:updated:/home/test:/bin/bash
```

`-a, --append`

A utilise avec l'option `-G` pour ajouter l'utilisateur à un groupe, `-aG` est possible

`-c, --comment`

Met à jour le champ commentaire dans le fichier `/etc/passwd`, optionnel

`-d, --home`

Modifier le répertoire personnel du compte utilisateur spécifié. Si cette option est utilisée avec l’option `-m`, le contenu du répertoire personnel en cours sera déplacé vers le nouveau répertoire personnel, qui sera créé s’il n’existe pas déjà.

`-e, --expiredate`

Définir la date d’expiration du compte utilisateur spécifié, sous le format YYYY-MM-DD ( après date unix 1970-01-01)

`-f, --inactive`

Définir le nombre de jours après l’expiration du mot de passe pendant lesquels l’utilisateur devra mettre à jour son mot de passe (faute de quoi le compte sera désactivé).

`-G, --groups`

Ajouter des groupes secondaires au compte utilisateur spécifié. Chaque groupe doit exister et doit être séparé du suivant par une virgule, sans espace blanc intermédiaire. Si elle est utilisée seule, cette option supprime tous les groupes existants auxquels l’utilisateur appartient, tandis que si elle est utilisée avec l’option `-a`, elle ajoute simplement les nouveaux groupes secondaires aux groupes existants.

`-l, --login`

Modifier nom d’utilisateur (_login_) du compte spécifié.

`-L, --lock`

Verrouiller le compte utilisateur spécifié. Cette opération ajoute un point d’exclamation devant le mot de passe chiffré dans le fichier `/etc/shadow`, ce qui désactive l’accès avec un mot de passe pour cet utilisateur.

`-r, --remove`

Supprime l'utilisateur d'un groupe, utilisable uniquement avec l'option `-G`

`-s, --shell`

Modifier le shell de connexion du compte utilisateur spécifié.

`-u`

Modifier l’UID du compte utilisateur spécifié.

`-U, --unlock`

Déverrouiller le compte utilisateur spécifié. Cette opération supprime le point d’exclamation devant le mot de passe chiffré dans le fichier `/etc/shadow`.

## Supprimer compte utilisateur

Si vous souhaitez supprimer un compte utilisateur, vous pouvez utiliser la commande `userdel`

Plus précisément, cette commande met à jour les informations stockées dans les bases de données des comptes en supprimant toutes les entrées relatives à l’utilisateur spécifié

userdel 

`-f, --force` 

force la suppression de l'utilisateur

`-r, --remove`

les fichiers présent dans le répertoire de l'utilisateur et son répertoire `/home` seront supprimer. Tout les fichiers présent ailleurs seront à supprimer manuellement ou modificer l'appartenance

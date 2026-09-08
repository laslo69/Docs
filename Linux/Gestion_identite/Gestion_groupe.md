
Tout comme pour la gestion des utilisateurs, vous pouvez ajouter, modifier et supprimer des groupes en utilisant les commandes `groupadd`, `groupmod` et `groupdel` avec les privilèges de root

## Créer groupe

`groupadd`

`-g, --gid` : défini un GID de groupe au moment de la création
`-r, --system` crée un groupe systeme
`-U, --users` permet d'ajouter des utilisateurs au groupe au moment de sa création

## Modifier groupe

`groupmod`

`-a; --append` ajoute un utilisateur à une groups, à utiliser avec `-U`
`-g, --gid` modifie le GID
`-n, --new-name` renomme le groupe

## Supprimer groupe

`groupdel`

`-f, --force` forcer la suppression du groupe

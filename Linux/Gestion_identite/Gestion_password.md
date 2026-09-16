
## Changer mot de passe

La commande `passwd` est utilisée essentiellement pour modifier le mot de passe d’un utilisateur

Chaque utilisateur peut modifier son propre mot de passe, mais seul root a le droit de modifier le mot de passe de n’importe quel utilisateur

Ceci est dû au fait que la commande `passwd` a le bit SUID activé
ce qui signifie qu’elle s’exécute avec les privilèges du propriétaire du fichier (en l’occurrence root) sur le fichier `/etc/passwd`

```bash
passwd -S
root P 2026-07-30 0 99999 7 -1
```

Voici une description détaillée des sept champs

1)  Nom utilisateur
2) P : Mot de passe valide, L pour un mot de passe verrouille, NP mot de passe inexistant
3) Date du dernier changement de mot de passe
4) Durée minimale en jours entre chaque changement de mot de passe ( 0 indique que aucune limite n'est fixé )
5) Durée maximal en jours pendant lesquels, le mot de passe est valide
6) Periode d'avertissement en jours avant expiration du mot de passe
7) Periode d'inactivite du mot de passe en jours après expiration du mot de passe, -1 supprime l'inactivite du compte

- `-d, --delete` supprime le mot de passe, verrouille l'utilisateur
- `-e, --expire` annule la validité du mot de passe, oblige l'utilisateur à changer de mot de passe
- `-i, --inactive` verrouille un utilisateur après la durée de validité après le temps fournis après expiration du mot de passe
- `-l, --lock` verrouiller le mot de passe du compte indiqué, remplace le mot de passe par une valeur non valide ( ajout un `!` en début du mot de passe). Ne verrouille pas le compte et reste accessible via SSH avec connexion par clé
- `-S, --status` affiche les informations sur le statut du mot de passe d'un compte utilisateur, utilisable avec le paramètre `-a`.L pour un compte bloqué, NP si pas de mot de passe, P pour un password utilisable
- `-u, --unlock` deverouille le mot de passe, rétabli le mot de passe dans son état initial ( avant modification par `-l` )

## Validite mot de passe

chage est utilisée pour modifier les informations d’expiration du mot de passe d’un utilisateur

La commande `chage` est réservée à l’utilisateur root, à l’exception de l’option `-l` qui peut être utilisée par les utilisateurs ordinaires pour obtenir la liste des informations d’expiration du mot de passe de leur propre compte

`-d, --lastday` définit le dernier changement de mot  de passe pour un utilisateur
`-E, --expiredate` configure la date ou le nombre de jours à compter du 1er janvier 1970 à partir de laquelle le compte utilisateur ne sera plus accessible ( peut être exprimé au format YYYY-MM-DD), ex : chage -E $(date -d +180days %Y-%m-%d)
`-I, --inactive` configure le nombre de jours d'inactivité après que le mot de passe ait expiré, avant de verrouiller le compte
`-l, --list` Affiche les informations sur l'age des comptes
`-m, --mindays` configure le nombre de jours minimum avant qu'un utilisateur peut changer son mot de passe
`-M, --maxdays` configure le nombre de jours maximum avant que l'utilisateur soit forcer de changer de mot de passe
`W, --warndays` configure le nombre de jour avec avertiseement de changement de mot de passe, avant que le changement de mot de passe soit requis

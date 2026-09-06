
Cette commande est utilisée essentiellement pour modifier le mot de passe d’un utilisateur. Comme nous l’avons déjà dit, chaque utilisateur peut modifier son propre mot de passe, mais seul root a le droit de modifier le mot de passe de _n’importe quel_ utilisateur

Ceci est dû au fait que la commande `passwd` a le bit SUID activé
ce qui signifie qu’elle s’exécute avec les privilèges du propriétaire du fichier (en l’occurrence root)

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 42096 mag 17  2015 /usr/bin/passwd
```

`-d, --delete` supprime le mot de passe, verrouille l'utilisateur
`-e, --expire` annule la validité du mot de passe, oblige l'utilisateur à changer de mot de passe
`-i, --inactive` verrouille un utilisateur après la durée de validité après le temps fournis après expiration du mot de passe
`-l, --lock` verrouiller le mot de passe du compte indiqué, remplace le mot de passe par une valeur non valide ( ajout un `!` en début du mot de passe). Ne verrouille pas le compte et reste accessible via SSH avec connexion par clé
`-S, --status` affiche les informations sur le statut du mot de passe d'un compte utilisateur, utilisable avec le paramètre `-a`.L pour un compte bloqué, NP si pas de mot de passe, P pour un password utilisable
`-u, --unlock` deverouille le mot de passe, rétabli le mot de passe dans son état initial ( avant modification par `-l` )
`-w, --warndays` nombre de jours avant que le mot de passe soit changé, prévient l'utilisateur


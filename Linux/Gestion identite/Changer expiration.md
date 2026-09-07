
chage est utilisée pour modifier les informations d’expiration du mot de passe d’un utilisateur

La commande `chage` est réservée à l’utilisateur root, à l’exception de l’option `-l` qui peut être utilisée par les utilisateurs ordinaires pour obtenir la liste des informations d’expiration du mot de passe de leur propre compte

`-d, --lastday` définit le dernier changement de mot  de passe pour un utilisateur
`-E, --expiredate` configure la date ou le nombre de jours à compter du 1er janvier 1970 à partir de laquelle le compte utilisateur ne sera plus accessible ( peut être exprimé au format YYYY-MM-DD), ex : chage -E $(date -d +180days %Y-%m-%d)
`-I, --inactive` configure le nombre de jours d'inactivité après que le mot de passe ait expiré, avant de verrouiller le compte
`-l, --list` Affiche les informations sur l'age des comptes
`-m, --mindays` configure le nombre de jours minimum avant qu'un utilisateur peut changer son mot de passe
`-M, --maxdays` configure le nombre de jours maximum avant que l'utilisateur soit forcer de changer de mot de passe
`W, --warndays` configure le nombre de jour avec avertiseement de changement de mot de passe, avant que le changement de mot de passe soit requis

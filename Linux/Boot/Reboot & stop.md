
La commande `shutdown` agit comme un intermédiaire aux procédures SysV ou systemd, c’est-à-dire qu’elle exécute l’action requise en appelant l’action correspondante dans le gestionnaire de services utilisé par le système

La commande `shutdown` ajoute des fonctions supplémentaires au processus de mise hors tension : elle envoie automatiquement un avertissement à tous les utilisateurs connectés à leurs sessions shell et empêche toute nouvelle connexion

Après l’exécution de `shutdown`, tous les processus reçoivent le signal `SIGTERM`, suivi du signal `SIGKILL`, puis le système s’arrête ou change de niveau d’exécution. Par défaut, lorsqu’aucune des options `-h` ou `-r` n’est utilisée, le système passe au niveau d’exécution 1, c’est-à-dire au mode mono-utilisateur

Pour modifier les options par défaut pour `shutdown`, la commande devra être exécutée avec la syntaxe suivante :

```bash
shutdown [option] time [message]
```

Seul le paramètre `time` est requis. Le paramètre `time` définit le moment où l’action requise sera exécutée, en acceptant les formats suivants :

- `hh:mm` : Ce format spécifie le moment de l’exécution en heures et minutes
- `+m` : Ce format précise le nombre de minutes à attendre avant l’exécution
- `now` or `+0` : Ce format définit l’exécution immédiate
- message : Affiche un texte dans toutes les sessions de terminal des utilisateurs connectés

## Systemd


La commande `systemctl` peut également être utilisée pour arrêter ou redémarrer la machine sur les systèmes basés sur systemd

Pour redémarrer le système, la commande `systemctl reboot` doit être utilisée, pour arrêter le système, utilisez la commande `systemctl poweroff`, les deux commandes nécessitent les droits root, étant donné que les utilisateurs normaux ne peuvent pas effectuer ce genre d’opération

une liste d'option disponible pour systemctl :

- systemctl halt : stop le système sans éteindre la machine
- systemctl poweroff : stop le système et la machine
- systemctl reboot : redémarre la machine
- systemctl suspend : suspend l'exécution du système
- systemctl hibernate : système en hibernation

## SysV

L'implémentation SysV permet de restreindre l'accès au redémarrage via `Ctrl+Alt+Del` en :

- Ajoutant l'option `-a` à la ligne `ctrlaltdel` dans le fichier `/etc/inittab`
- Seuls les utilisateurs listés dans `/etc/shutdown.allow` pourront alors redémarrer le système avec cette combinaison

Par défaut, `shutdown` sans option `-h` (halt) ou `-r` (reboot) fait passer le système au niveau d'exécution 1 (mode mono-utilisateur), une notion typique de SysV


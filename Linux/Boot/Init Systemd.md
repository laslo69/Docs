
Systemd est un gestionnaire de système et de services moderne avec une couche de compatibilité pour les commandes et les niveaux d’exécution SysV

Le gestionnaire de services est le premier programme lancé par le noyau au cours du processus de démarrage, son PID (numéro d’identification du processus) est donc toujours `1`.

systemd a une structure parallèle, utilise les sockets et D-Bus pour l’activation des services, l’exécution de démons à la demande, la surveillance des processus avec _cgroups_, le support des instantanés, la récupération des sessions système, le contrôle des points de montage et un contrôle des services basé sur les dépendances

Actuellement, systemd constitue la boîte à outils la plus utilisée pour gérer les ressources et les services du système, qui sont désignés sous le nom units par systemd, Une unité est composée d’un nom, d’un type et d’un fichier de configuration correspondant

Les `units` sont stockés sous `/lib/systemd/system`

D'autre répertoires peuvent être utilisé

- `/usr/local/lib/systemd/system`
- `/usr/lib/systemd/system`

On distingue sept types d’unités systemd :

- `service` : Le type d’unité le plus courant, pour les ressources actives du système qui peuvent être initiées, interrompues et rechargées
- `socket` : Le type d’unité `socket` peut être un socket de système de fichiers ou un socket réseau. Toutes les unités `socket` ont une unité `service` correspondante, chargée lorsque le socket est sollicité
- `device` : Une unité `device` est associée à un périphérique matériel identifié par le noyau. Un périphérique ne sera considéré comme une unité systemd que s’il existe une règle udev à cet effet. Une unité `device` peut être utilisée pour résoudre les dépendances de configuration lorsque certains composants matériels sont détectés, étant donné que les propriétés de la règle udev peuvent être utilisées comme paramètres pour l’unité `device`
- `mount` : Une unité `mount` est une définition de point de montage dans le système de fichiers, similaire à une entrée dans `/etc/fstab`
- `automount` : Une unité `automount` est également une définition de point de montage dans le système de fichiers, mais montée automatiquement. Chaque unité `automount` dispose d’une unité `mount` correspondante, qui est lancée lors de l’accès au point de montage `automount`
- `target` : Une unité `target` est un regroupement d’autres unités, gérées comme une seule unité
- `snapshot` : Une unité `snapshot` est un état sauvegardé du gestionnaire systemd

La commande principale pour contrôler les unités systemd est `systemctl`. La commande `systemctl` est utilisée pour exécuter toutes les tâches liées à l’activation, la désactivation, l’exécution, l’interruption, la surveillance des unités, etc

Pour une unité fictive appelée `unit.service`, par exemple, les opérations `systemctl` les plus courantes seront :

Tableau comparatif entre `sysv init` et `systemd`

|          Service          |                    Systemctl                    |                   Description                    |
| :-----------------------: | :---------------------------------------------: | :----------------------------------------------: |
|    service name start     |              systemctl start name               |                Démarre le service                |
|     service name stop     |               systemctl stop name               |                Arrête le service                 |
|   service name restart    |             systemctl restart name              |               Redémarre le service               |
| service name condrestart  |           systemctl try-restart name            | Redémarre le service seulement si il est démarré |
|    service name reload    |              systemctl reload name              |            Recharge la configuration             |
|    service name status    | systemctl status name, systemctl is-active name |        Indique si le service est démarré         |
| service name --status-all |    systemctl list-units --type service -all     |      Affiche le status de tous les services      |

Pour l'activation de service, avec `systemctl`

- `--before` liste les services et les unités qui doivent démarrer **après** l'unité spécifiée 
- `--after`  liste les services qui doivent démarrer **avant** l'unité en question

|       chkconfig       |                                           systemctl                                           |              description               |
| :-------------------: | :-------------------------------------------------------------------------------------------: | :------------------------------------: |
|   chkconfig name on   |                                     systemctl enable name                                     |           Active un service            |
|  chkconfig name off   |                                    systemctl disable name                                     |          Désactive un service          |
| chkconfig --list name |                       systemctl status name, systemctl is-enabled name                        |       Indique l'état du service        |
|   chkconfig --list    | systemctl list-unit-files --type service, systemctl list-dependencies --before / --after name | Liste des services et leurs dépendance |

Si aucune autre unité du même nom n’existe dans le système, le suffixe après le point peut être omis. Si, par exemple, il n’y a qu’une seule unité `httpd` de type `service`, alors un simple `httpd` est suffisant comme paramètre d’unité pour `systemctl`

La commande `systemctl` peut également contrôler les cibles système (_system targets_). L’unité `multi-user.target`, par exemple, combine toutes les unités requises par l’environnement système multi-utilisateurs. Il est similaire au niveau d’exécution 3 dans un système basé sur SysV

La commande `systemctl isolate` bascule entre différentes cibles. Ainsi, pour basculer manuellement vers la cible `multi-user` :

```bash
systemctl isolate multi-user.target
```

Il existe des cibles correspondantes aux niveaux d’exécution SysV, en allant de `runlevelO.target` jusqu’à `runlevel6.target`. En revanche, systemd n’utilise pas le fichier `/etc/inittab`. Pour modifier la cible système par défaut, l’option `systemd.unit` peut être ajoutée à la liste des paramètres du noyau. Par exemple, pour utiliser `multi-user.target` comme cible standard, le paramètre du noyau sera `systemd.unit=multi-user.target`. Tous les paramètres du noyau peuvent être rendus persistants en modifiant la configuration du chargeur d’amorçage.

Une autre manière de changer la cible par défaut consiste à modifier le lien symbolique `/etc/systemd/system/default.target` de manière à ce qu’il pointe vers la cible souhaitée. La redéfinition du lien peut être effectuée par le biais de la commande `systemctl` :

```
systemctl set-default multi-user.target
```

De même, on peut déterminer la cible de démarrage par défaut du système avec la commande suivante :

```bash
systemctl get-default
graphical.target
```

Comme pour les systèmes avec SysV, la cible par défaut ne doit jamais pointer vers `shutdown.target`, puisqu’elle correspond au niveau d’exécution 0 (arrêt).

tableau comparatif `sysv init` et `systemd`

| Runlevel |             Target unit             |     Description     |
| :------: | :---------------------------------: | :-----------------: |
|    0     |  runlevel0.target, poweroff.target  | Exctinction machine |
|    1     |   runlevel1.target, rescue.target   |  Boot single user   |
|    2     | runlevel2.target, multi-user.target | console, multi-user |
|    3     | runlevel3.target, multi-user.target | console, multi-user |
|    4     | runlevel4.target, multi-user.target | console, multi-user |
|    5     | runlevel5.target, graphical.target  |   GUI, multi-user   |
|    6     |   runlevel6.target, reboot.target   |       Reboot        |

Les fichiers de configuration associés à chaque unité se trouvent dans le répertoire !

- `/etc/systemd/system`
- `/run/systemd/system`
- `/usr/lib/systemd/system`

La commande `systemctl list-unit-files` affiche la liste de toutes les unités disponibles et indique si elles sont activées au démarrage du système. L’option `--type` sélectionnera uniquement les unités pour un certain type, comme dans `systemctl list-unit-files --type=service` et `systemctl list-unit-files --type=target`.

Les unités actives ou ayant été actives pendant la session système en cours peuvent être listées avec la commande `systemctl list-units`. Comme pour l’option `list-unit-files`, la commande `systemctl list-units --type=service` sélectionnera uniquement les unités de type `service` et la commande `systemctl list-units --type=target` sélectionnera uniquement les unités de type `target`

Systemd est également chargé du déclenchement et de la réponse aux événements liés à l’alimentation. La commande `systemctl suspend` mettra le système en mode économique, en gardant les données actuelles en mémoire. La commande `systemctl hibernate` va copier toutes les données en mémoire sur le disque, de sorte que l’état actuel du système peut être récupéré après son extinction. Les actions associées à ces événements sont définies dans le fichier `/etc/systemd/logind.conf` ou dans des fichiers individuels à l’intérieur du répertoire `/etc/systemd/logind.conf.d/`.Cependant, cette fonctionnalité de systemd ne peut être utilisée que lorsqu’il n’y a aucun autre gestionnaire d’alimentation en cours d’exécution dans le système, comme le démon `acpid`


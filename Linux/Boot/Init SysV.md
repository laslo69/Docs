
Un gestionnaire de services basé sur la norme SysV init contrôle les daemons et les ressources qui seront disponibles en se basant sur le concept de runlevels

Les runlevels sont numérotés de 0 à 6 et définis par les mainteneurs des distributions pour répondre à des objectifs spécifiques. Les seules définitions de niveau d’exécution communes à toutes les distributions sont les niveaux d’exécution 0, 1 et 6 et ils sont généralement affectés aux objectifs suivants :

- Niveau 0 : Arrêt du système
- Niveau 1, s ou _single_ : Mode mono-utilisateur, sans réseau et autres fonctionnalités non-essentielles (pour la maintenance)
- Niveaux 2, 3 ou 4 : Mode multi-utilisateur. Les utilisateurs peuvent se connecter par la console ou par le réseau. Les niveaux 2 et 4 ne sont pas souvent utilisés
- Niveau 5 : Mode multi-utilisateur. Il est équivalent au niveau 3, avec la connexion en mode graphique
- Niveau 6 : Redémarrage du système

Le programme chargé de la gestion des niveaux d’exécution et des démons/ressources associés est `/sbin/init`, peut se trouver dans différent répertoire selon la distribution

- Redhat : `/etc/rc.d/rc.sysinit`
- Opensuse : `/etc/init.d/boot`
- Debian : `/etc/init.d/rcS`

Lors de l’initialisation du système, le programme `init`, PID 1, identifie le niveau d’exécution requis, défini par un paramètre du noyau ou dans le fichier `/etc/inittab`, charge les scripts correspondants qui y sont référencés pour le niveau d'exécution donné

Chaque niveau d’exécution peut être associé à une série de fichiers de services, généralement des scripts dans le répertoire `/etc/init.d/`. Comme tous les niveaux d’exécution ne se valent pas selon les différentes distributions Linux, une description succincte de la finalité du niveau d’exécution peut également être trouvée dans les distributions basées sur SysV

La syntaxe du fichier `/etc/innitab` respecte le format

```bash
id:niveaux:action:processus
```

- '`id` est un nom générique comportant jusqu’à quatre caractères, utilisé pour identifier l’entrée
- `niveaux` est une liste de numéros de niveaux d’exécution pour lesquels une action spécifiée doit être exécutée
- `action` définit comment `init` va exécuter le processus indiqué
- `processus`, les actions disponibles sont les suivantes :
	- `boot` : Le processus sera exécuté lors de l’initialisation du système. Le champ `niveaux` est ignoré.
	- `bootwait` : Le processus sera exécuté pendant l’initialisation du système et `init` attendra qu’il se termine pour continuer. Le champ `niveaux` est ignoré.
	- `sysinit` : Le processus sera exécuté après l’initialisation du système, quel que soit le niveau d’exécution. Le champ `niveaux` est ignoré.
	- `wait` : Le processus sera exécuté pour les niveaux d’exécution donnés et `init` attendra qu’il se termine pour continuer.
	- `respawn` : Le processus sera relancé s’il est interrompu.
	- `ctrlaltdel` : Le processus sera exécuté lorsque le processus `init` reçoit le signal `SIGINT`, déclenché lorsque la séquence de touches Ctrl+Alt+Del est actionnée.

Le niveau d’exécution par défaut - celui qui sera choisi si aucun autre n’est fourni comme paramètre du noyau - est également défini dans `/etc/inittab`, dans l’entrée `id:x:initdefault`

Le `x` est le nombre correspondant au niveau d’exécution par défaut. Ce nombre ne doit jamais être égal à `0` ou `6`, étant donné que cela entraînerait l’arrêt ou le redémarrage du système dès la fin du processus de démarrage

Un fichier `/etc/inittab` typique :

```bash
# Niveau d'exécution par défaut
id:3:initdefault:

# Script de configuration exécuté au démarrage
si::sysinit:/etc/init.d/rcS

# Action effectuée au niveau d'exécution S (mono-utilisateur)
~:S:wait:/sbin/sulogin

# Configuration pour chaque niveau d'exécution
l0:0:wait:/etc/init.d/rc 0
l1:1:wait:/etc/init.d/rc 1
l2:2:wait:/etc/init.d/rc 2
l3:3:wait:/etc/init.d/rc 3
l4:4:wait:/etc/init.d/rc 4
l5:5:wait:/etc/init.d/rc 5
l6:6:wait:/etc/init.d/rc 6

# Action effectuée à la combinaison de touches ctrl+alt+del
ca::ctrlaltdel:/sbin/shutdown -r now

# Activer les consoles pour les niveaux d'exécution 2 et 3
1:23:respawn:/sbin/getty tty1 VC linux
2:23:respawn:/sbin/getty tty2 VC linux
3:23:respawn:/sbin/getty tty3 VC linux
4:23:respawn:/sbin/getty tty4 VC linux

# Activer le port série en plus pour le niveau d'exécution 3
# terminals ttyS0 and ttyS1 (modem) consoles
S0:3:respawn:/sbin/getty -L 9600 ttyS0 vt320
S1:3:respawn:/sbin/mgetty -x0 -D ttyS1
```

La commande `telinit q` devra être exécutée après chaque modification du fichier `/etc/inittab`. 

`telinit` prend différent argument en charge pour savoir comment intéragir avec le fichier `/etc/inittab` si il à été modifié.

- `-q` ou `-Q` indique à `init` de relire `/etc/inittab`,de recharger le code exécutable de sa configuration et actualise init en RAM
- `-U` ou `-u` Force `init` à se ré-exécuter, remplçant son propre code en mémoire par le binaire sur le disque. Le programme `init` change de version en cours de route mais conserve son PID 1
- `-t` envoi une commande `SIGTERM` aux processus encore en cours de fonctionnement après un délai imparti. par exemple `telinit -t 10 0` envoi un `runlevel` 0 au bout de 10 secondes aux processus.

Les scripts utilisés par `init` pour configurer chaque niveau d’exécution sont rangés dans le répertoire `/etc/init.d/`

Chaque niveau d’exécution dispose d’un répertoire associé dans `/etc/`, nommé `/etc/rc0.d/`, `/etc/rc1.d/`, `/etc/rc2.d/`, etc., avec les scripts censés être exécutés au démarrage du niveau d’exécution correspondant

`rc` lit d'abord les fichiers avec `k` et ensuite les `s` et dans l'ordre numérique en partant de 0

exemple d'ordre logique de lecture

```bash
K20apache2
S10syslog
S40network
S55sshd
```

Étant donné qu’un même script peut être utilisé par différents niveaux d’exécution, les fichiers contenus dans ces répertoires ne sont que des liens symboliques vers les scripts réels dans `/etc/init.d/`

Par ailleurs, la première lettre du nom du lien dans le répertoire du niveau d’exécution indique si le service doit être démarré ou arrêté pour le niveau d’exécution correspondant suivi d'une valeur et d'un nom

La première lettre peut être `S` soit `K` :
- Un nom de lien commençant par la lettre `K` détermine que le service sera arrêté à l’entrée du niveau d’exécution ( kill ) `/etc/rc3.d/k20network`
- S’il commence par la lettre `S`, le service sera démarré lors de l’entrée dans le niveau d'exécution ( start ) `/etc/rc3.d/s10syslog`

- 20 indique le numéro de priorité du processus

- nom : indique le nom du service


Le répertoire `/etc/rc1.d/`, par exemple, comportera de nombreux liens vers des scripts réseau commençant par la lettre `K`, étant donné que le niveau d’exécution 1 est le niveau d’exécution mono-utilisateur, sans connectivité réseau

La commande `runlevel` indique le niveau d’exécution en cours pour le système. Cette commande affiche deux valeurs, la première est le niveau d’exécution précédent et la seconde correspond au niveau d’exécution actuel.

L'information est aussi disponible dans le fichier `/sbin/runlevel`

```bash
runlevel
N 3
```

La lettre `N` dans la sortie montre que le niveau d’exécution n’a pas changé depuis le dernier démarrage. Dans l’exemple, `runlevel 3` correspond au niveau d’exécution en cours du système

Le même programme `init` peut être utilisé pour basculer à chaud entre les niveaux d’exécution d’un système, sans qu’il soit nécessaire de redémarrer. La commande `telinit` peut également être utilisée pour basculer d’un niveau à l’autre. Par exemple, les commandes `telinit 1`, `telinit s` ou `telinit S` feront passer le système au niveau d’exécution 1

Pour démarrer un service manuellement, il faut citer explicitement le chemin complet du service à exécuter

```bash
/etc/init.d/sshd start
```

Cependant, certaines distribution comme `Red hat` ou `OpenSUSE` ont intégré un outils pour faciliter la gestion, par le biais de la commande `service`

```bash
service sshd start
```

`chkconfig` est un outil en ligne de commande utilisé dans les systèmes d'initiation traditionnels SysVinit (comme sur les anciennes versions de Red Hat, CentOS ou Fedora) pour gérer les services et leurs runlevels 

- Lister l'état des services pour tous les niveaux d'exécution : `chkconfig --list`
- Activer un service au démarrage (par exemple pour les niveaux 3 et 5) : `chkconfig --level 35 [nom_du_service] on`
- Désactiver un service au démarrage : `chkconfig [nom_du_service] off`
- Ajouter un nouveau service à la gestion de `chkconfig` : `chkconfig --add [nom_du_service]`


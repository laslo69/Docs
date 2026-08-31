
Upstart a été développé pour la distribution Ubuntu Linux afin de faciliter le démarrage parallélisant le processus de chargement des services du système. Ubuntu a cessé d’utiliser Upstart en 2015, date à laquelle la distribution est passée de Upstart à systemd.

Les scripts d’initialisation utilisés par Upstart se trouvent dans le répertoire `/etc/init/`. Les services du système peuvent être affichés avec la commande `initctl list`, qui indique également l’état actuel des services et, le cas échéant, leur PID.

Le gestionnaire de services est le premier programme lancé par le noyau au cours du processus de démarrage, son PID (numéro d’identification du processus) est donc toujours `1`.

```bash
# initctl list
avahi-cups-reload stop/waiting
avahi-daemon start/running, process 1123
mountall-net stop/waiting
mountnfs-bootclean.sh start/running
nmbd start/running, process 3085
passwd stop/waiting
rc stop/waiting
rsyslog start/running, process 1095
tty4 start/running, process 1761
udev start/running, process 1073
upstart-udev-bridge start/running, process 1066
console-setup stop/waiting
irqbalance start/running, process 1842
plymouth-log stop/waiting
smbd start/running, process 1457
tty5 start/running, process 1764
failsafe stop/waiting
```


Chaque action Upstart dispose de sa propre commande dédiée. Par exemple, la commande `start` peut être utilisée pour lancer un sixième terminal virtuel :

```bash
start tty6**
```

L’état actuel d’une ressource peut être vérifié avec la commande 
`status` :

```bash
status tty6
tty6 start/running, process 3282
```

Quant à l’interruption d’un service, elle se fait avec la commande `stop` :

```bash
# stop tty6
```

Upstart n’utilise pas le fichier `/etc/inittab` pour définir les niveaux d’exécution, par contre les commandes traditionnelles `runlevel` et `telinit` permettent toujours de vérifier les niveaux d’exécution et d’alterner entre eux.
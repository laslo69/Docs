il peut y avoir des erreurs au cours du processus de démarrage, mais elles ne sont pas forcément critiques au point de provoquer l’arrêt complet du système d’exploitation. Néanmoins, ces erreurs peuvent compromettre le comportement attendu du système

Toutes les erreurs génèrent des messages qui peuvent être utilisés pour de futures investigations, car ils contiennent des informations précieuses sur le moment et la manière dont l’erreur s’est produite

L’espace mémoire où le noyau stocke ses messages, y compris les messages de démarrage, est appelé kernel ring buffer

La sortie de `dmesg` peut compter des centaines de lignes

Les valeurs au début de chaque ligne correspondent au nombre de secondes par rapport à l’instant où le noyau a commencé à être chargé

Les commandes `dmesg -H` ou `dmesg --human` activeront la pagination par défaut

```bash
$ dmesg
[    5.262389] EXT4-fs (sda1): mounted filesystem with ordered data mode. Opts: (null)
[    5.449712] ip_tables: (C) 2000-2006 Netfilter Core Team
[    5.460286] systemd[1]: systemd 237 running in system mode.
[    5.480138] systemd[1]: Detected architecture x86-64.
[    5.481767] systemd[1]: Set hostname to <torre>.
[    5.636607] systemd[1]: Reached target User and Group Name Lookups.
[    5.636866] systemd[1]: Created slice System Slice.
[    5.637000] systemd[1]: Listening on Journal Audit Socket.
[    5.637085] systemd[1]: Listening on Journal Socket.
[    5.637827] systemd[1]: Mounting POSIX Message Queue File System...
[    5.638639] systemd[1]: Started Read required files in advance.
[    5.641661] systemd[1]: Starting Load Kernel Modules...
[    5.661672] EXT4-fs (sda1): re-mounted. Opts: errors=remount-ro
[    5.694322] lp: driver loaded but no devices found
[    5.702609] ppdev: user-space parallel port driver
[    5.705384] parport_pc 00:02: reported by Plug and Play ACPI
[    5.705468] parport0: PC-style at 0x378 (0x778), irq 7, dma 3 [PCSPP,TRISTATE,COMPAT,EPP,ECP,DMA]
[    5.800146] lp0: using parport0 (interrupt-driven).
[    5.897421] systemd-journald[352]: Received request to flush runtime journal from PID 1
```

Cependant, le tampon circulaire du noyau perd tous les messages lorsque le système est éteint ou lorsque la commande `dmesg --clear` est exécutée. Invoquée sans options, la commande `dmesg` affiche les messages actuels dans le tampon circulaire du noyau

## journalctl

Dans les systèmes basés sur systemd, la commande `journalctl` affichera les messages d’initialisation avec les options `-b`, `--boot`, `-k` ou `--dmesg`

La commande `journalctl --list-boots` affiche une liste de numéros de démarrage relatifs au démarrage en cours, leur empreinte d’identification et l’horodatage du premier et du dernier message correspondant

```bash
journalctl --list-boots
 -4 9e5b3eb4952845208b841ad4dbefa1a6 Thu 2019-10-03 13:39:23 -03—Thu 2019-10-03 13:40:30 -03
 -3 9e3d79955535430aa43baa17758f40fa Thu 2019-10-03 13:41:15 -03—Thu 2019-10-03 14:56:19 -03
 -2 17672d8851694e6c9bb102df7355452c Thu 2019-10-03 14:56:57 -03—Thu 2019-10-03 19:27:16 -03
 -1 55c0d9439bfb4e85a20a62776d0dbb4d Thu 2019-10-03 19:27:53 -03—Fri 2019-10-04 00:28:47 -03
  0 08fbbebd9f964a74b8a02bb27b200622 Fri 2019-10-04 00:31:01 -03—Fri 2019-10-04 10:17:01 -03
```

Les journaux d’initialisation précédents sont également conservés dans les systèmes basés sur systemd, de sorte que les messages des sessions précédentes du système d’exploitation peuvent toujours être inspectés

Si les options `-b 0` ou `--boot=0` sont fournies, alors les messages pour le démarrage en cours seront affichés. Les options `-b -1` ou `--boot=1` afficheront les messages de la précédente initialisation, en augmentant la vleur de `-b` ou `--boot`, il est possible de remonter sur des boots précédents

Les messages d’initialisation ainsi que les autres messages émis par le système d’exploitation sont stockés dans des fichiers à l’intérieur du répertoire `/var/log/`

Si une erreur critique se produit et que le système d’exploitation est incapable de poursuivre le processus d’initialisation après le chargement du noyau et de l’initramfs, un support de démarrage alternatif pourrait être utilisé pour démarrer le système et accéder au système de fichiers correspondant.

Ensuite, les fichiers en dessous de `/var/log/` peuvent être examinés pour déterminer les raisons possibles de l’interruption du processus de démarrage. Les options `-D` ou `--directory` de la commande `journalctl` peuvent être utilisées pour lire les logs dans des répertoires autres que `/var/log/journal/`, qui est l’emplacement par défaut des messages de journalisation de systemd. Étant donné que les messages du journal de systemd ne sont pas stockés au format texte brut, la commande `journalctl` est indispensable pour les lire
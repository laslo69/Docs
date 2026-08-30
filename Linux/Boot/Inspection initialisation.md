il peut y avoir des erreurs au cours du processus de démarrage, mais elles ne sont pas forcément critiques au point de provoquer l’arrêt complet du système d’exploitation. Néanmoins, ces erreurs peuvent compromettre le comportement attendu du système

Toutes les erreurs génèrent des messages qui peuvent être utilisés pour de futures investigations, car ils contiennent des informations précieuses sur le moment et la manière dont l’erreur s’est produite

L’espace mémoire où le noyau stocke ses messages, y compris les messages de démarrage, est appelé `kernel ring buffer`

## Kernel Buffer Ring

La sortie de `dmesg` affiche les messages acutels dans le `kernel buffer ring`, peut compter des centaines de lignes

Les valeurs au début de chaque ligne correspondent au nombre de secondes par rapport à l’instant où le noyau a commencé à être chargé

Les messages sont stockés dans `/var/log/`, les fichiers intéressants sont souvent `syslog`, `boot.log` et `messages`

Les commandes `dmesg -H` ou `dmesg --human` activeront la pagination par défaut

Certains paramètres peuvent être intéressant

- T : Améliore l'affichage du timestamp
- k : Affiche uniquement les messages kernel
- l : Affiche les messages selon le niveau de gravité ( warn,err,crit,emerg )

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

Dans les systèmes basés sur systemd, la commande `journalctl` affichera les messages d’initialisation `-k` pour les messages kernel ou `--dmesg`

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

Si les options `-b 0` ou `--boot=0` sont fournies, alors les messages pour le démarrage actuel seront affichés

Les options `-b -1` ou `--boot=1` afficheront les messages de la précédente initialisation, en augmentant la valeur de `-b` ou `--boot`, il est possible de remonter sur les boots précédents

Le paramètre `-k` permet de consulter les messages kernel contenu dans `journalctl`

```bash
journalctl -k
août 30 16:09:42 debian kernel: Linux version 6.12.101+deb13-amd64 (debian-kern>
août 30 16:09:42 debian kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-6.12.101>
août 30 16:09:42 debian kernel: BIOS-provided physical RAM map:
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x0000000000000000-0x0000000000>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x000000000009fc00-0x0000000000>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x00000000000f0000-0x0000000000>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x0000000000100000-0x000000003f>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x000000003ffdc000-0x000000003f>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x00000000b0000000-0x00000000bf>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x00000000fed1c000-0x00000000fe>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x00000000feffc000-0x00000000fe>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ff>
août 30 16:09:42 debian kernel: BIOS-e820: [mem 0x000000fd00000000-0x000000ffff>
août 30 16:09:42 debian kernel: NX (Execute Disable) protection: active
août 30 16:09:42 debian kernel: APIC: Static calls initialized
août 30 16:09:42 debian kernel: SMBIOS 2.8 present.
août 30 16:09:42 debian kernel: DMI: QEMU Standard PC (Q35 + ICH9, 2009), BIOS >
août 30 16:09:42 debian kernel: DMI: Memory slots populated: 1/1
août 30 16:09:42 debian kernel: Hypervisor detected: KVM
... Output Truncated
```

Le paramètre `-p` permet de filtrer par niveau de criticité ( emerg, alert, crit, err, warning, notice, info, debug )

## Examiner log d'un autre PC

Si une erreur critique se produit et que le système d’exploitation est incapable de poursuivre le processus d’initialisation après le chargement du noyau et de l’initramfs, un support de démarrage alternatif pourrait être utilisé pour démarrer le système et accéder au système de fichiers correspondant

Ensuite, les fichiers en dessous de `/var/log/` peuvent être examinés pour déterminer les raisons possibles de l’interruption du processus de démarrage

Les options `-D` ou `--directory` de la commande `journalctl` peuvent être utilisées pour lire les logs dans des répertoires autres que `/var/log/journal/`, qui est l’emplacement par défaut des messages de journalisation de systemd. Étant donné que les messages du journal de systemd ne sont pas stockés au format texte brut, la commande `journalctl` est indispensable pour les lire

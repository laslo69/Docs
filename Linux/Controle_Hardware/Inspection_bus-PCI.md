La commande `lspci` permet de lister les composants connecté et détecté sur le bus PCI de la carte mère, la sortie de commande peut donner une grande quantité d'information

```bash
$ lspci
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
04:04.0 Multimedia audio controller: VIA Technologies Inc. ICE1712 [Envy24] PCI Multi-Channel I/O Controller (rev 02)
04:0b.0 FireWire (IEEE 1394): LSI Corporation FW322/323 [TrueFire] 1394a Controller (rev 70)
... Output Truncated
```

Il est possible de spécifié un élément de la liste en particulier en ajoutant le paramètre `-s` pour cibler l'élement, et `-v` pour le mode verbose.

Les paramètres `-s` et `-v` ne peuvent pas être regroupé sous la form `-sv`

```bash
root@debian:~# lspci -s 04:02.0 -v
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
    Subsystem: Linksys WMP54G v4.1
    Flags: bus master, slow devsel, latency 32, IRQ 21
    Memory at e3100000 (32-bit, non-prefetchable) [size=32K]
    Capabilities: [40] Power Management version 2
    kernel driver in use: rt61pci
```

La sortie affiche maintenant plus de détails sur le périphérique à l’adresse `04:02.0`

Il s’agit d’une carte réseau dont le nom interne est `Ralink corp. RT2561/RT61 802.11g PCI`

L’entrée `Subsystem` est associée à la marque et au modèle de l’appareil - `Linksys WMP54G v4.1` - et peut servir à des fins de diagnostic

***

Le champs `kernel driver in use` permet de renseigner le module utilisé par le périphérique en question

Pour voir uniquement les caractéristiques kernel de l'interface.

Les options `-s` et `-k` ne peuvent pas être regroupé sous la forme `-sk`

```bash
root@debian:~# lspci -s 01:00.0 -k
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
    kernel driver in use: nvidia
    kernel modules: nouveau, nvidia_drm, nvidia
```

Pour le périphérique en question - une carte graphique NVIDIA - `lspci` nous indique à la ligne `kernel driver in use: nvidia` que le module en question est nommé `nvidia`, et tous les modules correspondants du noyau sont listés à la ligne `kernel modules : nouveau, nvidia_drm, nvidia`.

***

le paramètre `-t` permet d'afficher une arborescence logique de la sortie de la commande

```bash
lspci -t

-[0000:00]-+-00.0
           +-00.2
           +-01.0
           +-01.1-[01]----00.0
           +-01.3-[03-22]--+-00.0
           |               +-00.1
           |               \-00.2-[04-22]--+-00.0-[1f-20]----00.0-[20]--
           |                               +-01.0-[21]----00.0
           |                               \-04.0-[22]--
           +-02.0
           +-03.0
           +-03.1-[23-25]----00.0-[24-25]----00.0-[25]--+-00.0
           |                                            \-00.1
           +-04.0
           +-07.0
           +-07.1-[26]--+-00.0
           |            +-00.2
           |            \-00.3
           +-08.0
           +-08.1-[27]--+-00.0
           |            +-00.2
           |            \-00.3
           +-14.0
           +-14.3
           +-18.0
           +-18.1
           +-18.2
           +-18.3
           +-18.4
           +-18.5
           +-18.6
           \-18.7
```

***

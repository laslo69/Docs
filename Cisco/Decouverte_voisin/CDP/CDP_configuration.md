
## Configuration

Sur une topologie avec 3 switch

![cdp1](/Cisco/Decouverte_voisin/CDP/illustration/cdp1.png)

### SW1

Sur le switch1 :

- Désactivation CDP sur les interfaces FastEthernet 0/2 à 0/24
- Modification des timers
	- Hello 60s -> 10s
	- Holdtime 180s -> 60s
- Forcer CDP version 2

```bash
sw1(config)# int range fa0/2-24
sw1(config-if)# no cdp enable
sw1(config-if)# exit
sw1(config)# cdp timer 10
sw1(config)# cdp holdtime 60
sw1(config)# cdp advertise-v2
end
wr
```

Output pour vérifier l'état

```bash
SW1#sh cdp neighbors

Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge

S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone

Device ID Local Intrfce Holdtme Capability Platform Port ID
SW2    Fas 0/1          152         S      2960     Fas 0/1
```

## SW2

- Désactivation CDP sur les interfaces FastEthernet 0/3 à 0/24
- Modification des timers
	- Hello 60s -> 10s
	- Holdtime 180s ->  60
- Forcer CDP version 2

```bash
sw2(config)# int range fa0/3-24
sw2(config-if)# no cdp enable
sw2(config-if)# exit
sw2(config)# cdp timer 10
sw2(config)# cdp holdtime 60
sw2(config)# cdp advertise-v2
end
wr
```

Output pour vérifier l'état

```bash
SW2#sh cdp neighbors

Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge

S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone

Device ID Local Intrfce Holdtme Capability Platform Port ID

SW1       Fas 0/1       159         S      2960     Fas 0/1
SW3       Fas 0/2       161         S      2960     Fas 0/1
```

### SW3

- Désactivation CDP sur les interfaces FastEthernet 0/2 à 0/24
- Modification des timers
	- Hello 60s -> 10s
	- Holdtime 180s ->  60
- Forcer CDP version 2

```bash
sw3(config)# int range fa0/2-24
sw3(config-if)# no cdp enable
sw3(config-if)# exit
sw3(config)# cdp timer 10
sw3(config)# cdp holdtime 60
sw3(config)# cdp advertise-v2
end
wr
```

Output pour vérifier l'état

```bash
SW3#sh cdp neighbors

Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge

S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone

Device ID Local Intrfce Holdtme Capability Platform Port ID
SW2 Fas   0/1           158          S     2960     Fas 0/2
```

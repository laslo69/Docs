# Route statique IPv4

Le routage statique est une méthode fondamentale et incontournable de l’administration réseau, il consiste à insérer manuellement des chemins dans la table de routage d’un routeur ou d’un switch de niveau 3, pour définir explicitement l’acheminement des paquets IP

Une route statique peut être de différent type :

- Default
- Network
- Floating

## Default

Une route par défaut ( noté 0.0.0.0/0) est utilisée lorsque aucune route plus spécifique ne correspond à la destination du paquet. Elle est souvent appelée "gateway of last resort" (passerelle de dernier recours)

Elle permet, dans le cas ou un routeur n'est pas en mesure de, forward un paquet vers l'IP demandé, de le transférer vers une destination qui va servir à drop tous les paquets inconnus

```bash
R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.0.1
```

## Network

Une Network Route, est une route statique configuré pour atteindre un réseau distant normalement non accessible avec la table de routage de base.

La configuration de la route doit se faire sur chaque routeur pour pouvoir, dans un premier temps émettre, mais aussi recevoir la réponse

```bash
R1(config)#ip route 192.168.2.0 255.255.255.0 192.168.3.2
R2(config)#ip route 192.168.1.0 255.255.255.0 192.168.3.1
```

## Floating

Une route flottante est une route de secours avec une distance administrative plus élevée que la route principale. Elle est utilisée pour assurer la redondance en cas de panne du lien principal

Une distance administrative doit être saisie manuellement à la fin de la route flottante. Il peut être intéressant de mettre une route statique avec une distance administrative élevé, dans le cas ou un protocole de routage dynamique crée une route similaire, la route statique flottante peut prendre le relais en cas de panne du processus

```bash
R1(config)#no ip route 192.168.2.0 255.255.255.0 192.168.3.2
R1(config)#ip route 192.168.2.0 255.255.255.0 192.168.3.2 130
R2(config)#no ip route 192.168.1.0 255.255.255.0 192.168.3.1
R2(config)#ip route 192.168.1.0 255.255.255.0 192.168.3.1 130
```

## Configuration Cisco

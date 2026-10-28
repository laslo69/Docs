# Route statique IPv6

Le routage statique est une méthode fondamentale et incontournable de l’administration réseau, il consiste à insérer manuellement des chemins dans la table de routage d’un routeur ou d’un switch de niveau 3, pour définir explicitement l’acheminement des paquets IP

Une route statique peut être de différent type :

- Default
- Network
- Floating

Pour utiliser le routage IPv6 avec Cisco, il sera nécessaire d'activer le routage IPv6

```bash
ipv6 unicast-routing
```

## Default

Une route par défaut ( noté 0.0.0.0/0) est utilisée lorsque aucune route plus spécifique ne correspond à la destination du paquet. Elle est souvent appelée "gateway of last resort" (passerelle de dernier recours)

Elle permet, dans le cas ou un routeur n'est pas en mesure de, forward un paquet vers l'IP demandé, de le transférer vers une destination qui va servir à drop tous les paquets inconnus

une route par défaut vers `0.0.0.0` s’écrit en ipv6 ::/0 <interface sortie | ip>

```bash
R1(config)#ipv6 route ::/0 g0/2 fe80::2
R2(config)#ipv6 route ::/0 g0/1 fe80::1
```

## Network

Une Network Route, est une route statique configuré pour atteindre un réseau distant normalement non accessible avec la table de routage de base.

La configuration de la route doit se faire sur chaque routeur pour pouvoir, dans un premier temps émettre, mais aussi recevoir la réponse

Pour configurer une Network route, il existe plusieurs façon de l'écrire

- interface de sortie
- interface de sortie + link-local du voisin
- adresse ipv6 du voisin

```bash
ipv6 route 2001:db8:acad:1::/64 fa0/1
ipv6 route 2001:db8:acad:1::/64 fa0/1 fe80::2
ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::1/64
```

## Floating

Une route flottante est une route de secours avec une distance administrative plus élevée que la route principale. Elle est utilisée pour assurer la redondance en cas de panne du lien principal

Une distance administrative doit être saisie manuellement à la fin de la route flottante. Il peut être intéressant de mettre une route statique avec une distance administrative élevé, dans le cas ou un protocole de routage dynamique crée une route similaire, la route statique flottante peut prendre le relais en cas de panne du processus

```bash
R1(config)#ipv6 route 2001:db8:1ab:2::/64 g0/2 fe80::2 130
R2(config)#ipv6 route 2001:db8:1ab:1::/64 g0/1 fe80::1 130
```
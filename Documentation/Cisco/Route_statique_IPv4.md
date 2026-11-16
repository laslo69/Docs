# Cisco - Routage statique IPv4

## Visualiser table routage

Quelques commandes utile pour visualiser le routage statique

- show ip route : affiche l'ensemble de la table de routage
- show ip route static : affiche seulement les entrées statique
- show ip route 192.168.2.1 : affiche le chemin emprunté pour atteindre la destination

```bash
R1#sh ip route 
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S    192.168.4.0/24 [1/0] via 192.168.2.2
                    [1/0] via 192.168.1.2
     10.0.0.0/16 is subnetted, 1 subnets
C       10.0.0.0 is directly connected, FastEthernet2/10
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet1/0
C    192.168.3.0/24 is directly connected, FastEthernet0/1
S*   0.0.0.0/0 [1/0] via 10.0.0.1
```

## Configuration complète

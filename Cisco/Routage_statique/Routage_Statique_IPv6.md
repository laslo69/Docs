
Le routage statique IPv6 fonctionne de manière similaire à IPv4, mais avec des adresses IPv6 ( 128 bits ) et une syntaxe légèrement différente

Les routes statiques IPv6 se configurent de la même manière qu’en IPv4 en utilisant la commande ipv6 route

On conseillera d’utiliser des passerelles adressées en Link-local ce qui nécessitera de préciser l’interface de sortie et l’adresse du prochain saut

Pour utiliser le routage IPv6 avec Cisco, il sera nécessaire d'activer le routage IPv6

```bash
ipv6 unicast-routing
```

## Default Route

Pour configurer une default route, il faut spécifier, le réseau à atteindre, l’interface de sortie du routeur local et l’adresse IP du routeur voisin connecté au lien de sortie

une route par défaut vers `0.0.0.0` s’écrit en ipv6 ::/0 <interface sortie | ip>

```bash
R1(config)#ipv6 route ::/0 g0/2 fe80::2
R2(config)#ipv6 route ::/0 g0/1 fe80::1
```

## Network Route

Pour configurer une Network route, il existe plusieurs façon de l'écrire

- interface de sortie
- interface de sortie + link-local du voisin
- adresse ipv6 du voisin

```bash
ipv6 route 2001:db8:acad:1::/64 fa0/1
ipv6 route 2001:db8:acad:1::/64 fa0/1 fe80::2
ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::1/64
```

## Host Route

Une route hôte est une route vers une adresse IP spécifique (masque /128). Elle est utilisée pour atteindre un hôte particulier (ex. un serveur, un routeur distant).

Permet de diriger le trafic vers une machine critique ou pour tester la connectivité vers un hôte précis

```bash
routeur(config)#ipv6 route 2001:db8:01ab:0002::100/128 2001:db8:3::2
```

## Floating route

Une route flottante est une route de secours avec une distance administrative plus élevée que la route principale. Elle est utilisée pour assurer la redondance en cas de panne du lien principal.

Dans le cas ou, le lien principal tombe en panne, la route statique défini va permettre, à vitesse égale ou en service dégradé, de garantir une continuité de service.

Une distance administrative doit être saisie manuellement à la fin de la route flottante.

Il peut être intéressant de mettre une route statique avec une distance administrative élevé, dans le cas ou un protocole de routage dynamique crée une route similaire, la route statique flottante peut prendre le relais en cas de panne du processus.

```bash
R1(config)#ipv6 route 2001:db8:1ab:2::/64 g0/2 fe80::2 130
R2(config)#ipv6 route 2001:db8:1ab:1::/64 g0/1 fe80::1 130
```

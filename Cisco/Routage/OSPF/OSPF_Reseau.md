
OSPF possède plusieurs type de réseau.

Le type de réseau dépend principalement de l'architecture de l'infrastructure réseau et de comment l'administrateur réseau va configurer cette infrastructure

OSPF possède une configuration par défaut du type de réseau selon le type de câble détecter sur l'interface

- Ethernet : Type broadcast
- Serial : Point-to-Point

Entre les différents type de réseau OSPF, les timers ne sont pas identique, l'élection des rôles de routeurs peut ne pas exister et la découverte de voisins doit être fait manuellement

Il est possible de les modifier mais, il faut respecter une règle impérative.

Dead interval = hello interval * 4

Si cette règle n'est pas respecté, le routeur ne sera pas en mesure de former une adjacence OSPF

Les timers sont modifiable par interface, ce qui laisse la possibilité d'avoir des réglages différent par interface

### Point-to-point

Les routeurs sont connectés en direct entre eux et sont les seules adresses IP de leur segment ( sous entendre, masque /30 par exemple ), en point-to-point

En Point-to-Point, il n’y a pas d’élection de DR/BDR du fait de la présence de seulement 2 routeurs dans la configuration réseau du segment ce qui réduit le temps nécessaire et le nombre de message envoyé.

La découverte des voisins est automatique et la configuration simple.

Timers par défaut:

- Hello: 10s
- Dead: 40s

```bash
interface Serial0/0
ip ospf network point-to-point
```

### Point-to-multipoint

Pas d’élection de DR/BDR, ce type de réseau est un ensemble de connexion point-to-point du même segment réseau, la découverte des voisins en automatique est active.

L'élection des rôles de routeurs se fait, avec ce type de réseau

Timers par défaut:

- Hello: 30s
- Dead: 120s

```bash
interface GigabitEthernet0/0
ip ospf network point-to-multipoint
```

### Broadcast

Plusieurs routeurs partagent le même segment réseau, l'élection des rôles de routeurs à lieu

Timer par défaut:

- Hello: 10s
- Dead: 40s

```bash
interface GigabitEthernet0/0
ip ospf network broadcast
```

### Non-broadcast multi access ( NBMA )

Similaire au point-to-multipoint, l’élection des DR/BDR se fait toujours

La configuration de la des voisins doit être fait manuellement car les routeurs n’ont pas la capacité de faire du broadcast ou multicast pour la découverte automatique

Election de DR/BDR, les timers par défaut sont:

- Hello: 30s
- Dead: 120s

```bash
interface Serial0/0
ip ospf network non-broadcast
ip ospf neighbor 192.168.1.2
```

### Point-to-multipoint non broadcast

Similaire au point-to-multipoint mais pour les routeurs ne supportant pas le broadcast et/ou le multicast, il n'y a pas d’élection de DR/BDR, pas de découverte automatique des voisins, les voisins doivent être configuré manuellement

Timers par défaut:

- Hello: 30s
- Dead: 120s

```bash
interface Serial0/0
ip ospf network non-broadcast
ip ospf neighbor 192.168.1.2
ip ospf neighbor 192.168.1.3
```

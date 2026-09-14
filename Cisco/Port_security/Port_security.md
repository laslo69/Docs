
Port-security est une fonctionnalité des switch Cisco permettant de sécuriser un port d’accès (auquel est connecté une station de travail, un serveur,…) en contrôlant les adresses MAC source des trames Ethernet qui arrivent sur le port en question. En cas de violation une action sera prise

## Activer port security

Pour activer Port sécurity, le port ne dois pas être dynamique, il faut donc forcer manuellement sont type avec:

```bash
switchport mode access
switchport access vlan <vlan_id>
ou
switchport mode trunk
```

une fois le type défini, activer le service sur le port

```bash
switchport port-security
```

## Gestion MAC et Aging

### Gestion Mac Address

L'apprentissage des adresses MAC peut se faire de 2 manières:

- MAC statique
- MAC dynamique

Il faut d’abord définir le nombre d’adresse MAC maximum autoriser sur un interface, entre 1 et 132. La valeur par défaut est de 1

Par défaut, une adresse MAC apprise est libéré après un temps d'inactivité, paramètrable modifiable avec l'option `sticky`

```bash
switchport port-security maximum <1 – 132 >
```

Pour renseigner une adresse statique, l’adresse MAC doit être renseigné avec le format 0000.0000.0000

```bash
switchport port-security mac-adress 0123:4567:8910
```

Pour un apprentissage dynamique des adresse MAC, le paramètre sticky garder en mémoire les adresses MAC d'un port, même après redémarrage

```bash
switchport port-security mac-address sticky
```

Possible d'interdire une adresse MAC

```bash
switchport port-security mac-address forbidden 0123:4567:8910
```

### Aging

Les adresse MAC apprisent dynamiquement par port-security ont une durée de vie, qui peut être réglée avec des conditions.

Pour définir une durée de validité d’une adresse MAC, une fois le aging-time dépassé, l’adresse MAC n’est plus apprise par port-security

```bash
switchport port-security aging time <1-1440>
```

Le vieillissement d’une adresse MAC peut être fixé selon 2 méthodes additionnel:

- absolute : Toutes les adresses MAC dynamiques vieillissent après le même délai (configuré par aging time), indépendamment de leur activité. Utile dans un environnement ou, les rotations d’appareils sont fréquent, comme les clients émettent régulièrement des trames, le timers recommence à 0 tant que le client donne signe de vie
- inactivity : Une adresse MAC vieillit uniquement si elle n’est plus active pendant la durée configurée. Utile sur un infrastructure ou les appareils sont fixé et peu de rotation

```bash
switchport port-security aging absolute
ou
switchport port-security aging inactivity
```

Il est possible, de pouvoir activer le vieillissement des adresses MAC sur des adresses MAC enregistrer manuellement, mais rarement utilisé.

```bash
Switchport port-security aging static
```

## Mode Violation

Par défaut, Port-security ne permet d’apprendre que une seule adresse MAC.

Lors de la mise en place du mode de violation de port-security, il faut spécifier quel mode choisir et l’action qui en découle. Si une trame avec une adresse MAC non autorisé est détécté, une action doit être prise.

- Shutdown : Le port va se mettre en Err-Disable. Pour remettre l’interface en UP, il faut shutdown l’interface et ensuite la no shutdown. Le switch garde une trace de cette violation
- Restrict : Le port va dropper toutes les trames reçues par cette adresse MAC, il traitera toujours les trames avec une MAC autorisé mais ne va pas se mettre en Err-Disable. Le switch garde une trace de la violation
- Protect : Le port va dropper toutes les trames reçues par cette adresse MAC, il traitera toujours les trames avec une MAC autorisé mais ne va pas se mettre en Err-Disable. Le switch ne garde pas de trace de la violation

```bash
switchport mode access
switchport access vlan 1
switchport port-security
switchport port-security violation restrict
```

## Access & Trunk

Avec port-security, la gestion des VLAN demande une approche différente.

En mode access, un switch va forward les trames, un switch supérieur va donc enregistrer une seule adresse MAC qui est celle du port du switch inférieur, mais dans le cas d’un trunk, il faut bien prendre en compte, que les adresse MAC apprises par le switch inférieur sont transmis au switch supérieur

Il faut donc, bien penser à son infrastructure car, si un VLAN possède 2 clients, le switch supérieur va, détecter l’adresse MAC de l’interface qui connecte les 2 switchs  + les adresses MAC des clients

## Administration

### Maintenance

Afficher les interfaces protégé par port-security

```bash
Switch#sh port-security 
Secure Port MaxSecureAddr CurrentAddr SecurityViolation Security Action
               (Count)       (Count)        (Count)
--------------------------------------------------------------------
        Fa0/1       10          0                 0         Shutdown
        Fa0/2       10          0                 0         Shutdown
```

Affiche les détails d'un port

```bash
Switch#sh port-security int
Switch#sh port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 1400 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 10
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```

Afficher les adresses MAC sécurisées, sous entendu celle enregistré à la main

```bash
Switch#show port-security address 
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
   1    0123.4567.8910    SecureConfigured              Fa0/1        -
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```

### Dépannage

Si en mode shutdown et qu'une violation à lieu, le port en erreur se retrouvera en `err-dsabled`

Corriger le problème ( pas assez d'adresse MAC autorisé ou MAC flooding ), mettre le port en `shutdown` et le repasser en `no shutdown`

```bash
Switch#sh interfaces status 
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        err-disabled 1          auto    auto  10/100BaseTX
Fa0/2                        connected    1          a-full  a-100 10/100BaseTX
```

Contrôler les logs

```bash
show logging
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%PM-4-ERR_DISABLE: psecure-violation error detected on Fa0/1, putting Fa0/1 in err-disable state.
%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 00E0.F925.8132 on port FastEthernet0/1.
```

Voir si des violations ont été constaté, le champ `Security violation` doit s'incrémenter

```bash
Switch# sh port-security 
Secure Port MaxSecureAddr CurrentAddr SecurityViolation Security Action
               (Count)       (Count)        (Count)
--------------------------------------------------------------------
        Fa0/1       10          0                 0         Shutdown
        Fa0/2       10          0                 0         Shutdown
```

Remettre à 0 les adresse MAC appris et en mémoire sur une interface

```bash
clear port-security sticky interface fa0/1
```

Dans le cas d'un grand nombre d'adresse MAC dans la table, remise à 0 de la table ARP pour les entrées dynamique

```bash
clear mac address-table dynamic
```

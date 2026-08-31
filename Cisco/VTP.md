
VTP est un protocole de couche 2 qui utilise l'adresse MAC multicast `01:00:0C:CC:CC:CC`, va permettre, lors de la création de VLAN, de propager les VLANs créer sur d’autre switch d’un même domaine VTP via des liens trunk

L’objectif de VTP est de faciliter l’administration des VLAN en réduisant le nombre de manipulation à faire sur chaque switch, en centralisant la gestion des VLAN à partir d’un switch qui fera office de serveur VTP

L’utilisation de VTP peut simplifier l’administration du réseau et faire gagner du temps dans le cas de grande infrastructure mais, son utilisation peut comporter des risques en cas de mauvaises manipulations

Chaque interface qui inter-connectent des switchs membres du domaine VTP, doivent être en trunk et autoriser les VLANs associé, à pouvoir échanger via le ou les liens trunk

## Revision Number

Le numéro de révision de configuration est un nombre sur 32 bits qui indique le niveau de la révision lors d'échange VTP

Chaque périphérique VTP suit le numéro de révision de configuration VTP qui lui est assigné. La plupart des paquets VTP contiennent le numéro de révision de configuration VTP de l'expéditeur.

Ces informations sont utilisées afin de déterminer si les informations reçues sont plus récentes que la version actuelle.

Chaque fois qu'un VLAN est modifier dans un périphérique VTP, la révision de configuration est incrémentée de un

## Mode VTP

VTP possède 3 modes en plus du off ( VTP désactivé )

Attention, avant de mettre un switch en production, s’assurer que le numéro de Revision Number soit bien à 0. En cas d’utilisation d’un switch avec un numéro de révision supérieur au serveur VTP, l’entièreté de la base de donnée VLAN sera écrasé.

Pour remettre le numéro de release à 0, il faut passer le switch en VTP transparent avant de le passer en VTP server ou client et supprimer le fichier avec la commande : `delete flash:vlan.dat`

### VTP Server

En mode serveur, permet de créer, modifier, supprimer des VLAN, transmet les annonces VTP et se synchronise avec d’autres switch VTP. Il est possible d’avoir plusieurs VTP serveurs dans le même domaine.

Le lien doit être en trunk mais il faut aussi, autoriser les VLAN à transiter sur ce lien.

Pour que le serveur VTP puisse fonctionner, il faut créer un domaine VTP

ATTENTION : Un seul domaine VTP par switch

```bash
vtp mode server
vtp domaine <domaine>
vtp password <password>
```

VTP en mode client permet de traiter les annonces VTP pour se mettre à jour et les transmettre aux voisins

Il n’est pas possible de créer,modifier ou supprimer des VLAN

```bash
vtp mode server
vtp domaine <domaine>
vtp password <password>
```

### VTP Transparent

En mode transparent, il est possible de créer, modifier et supprimer des VLAN mais uniquement localement et ne seront pas transmis.

Le switch ne traite pas les messages VTP mais relais les annonces à ses voisins VTP.

```rust
vtp mode transparent
vtp domaine <domaine>
vtp password <password>
```

### VTP Off

En mode Off, VTP est simplement désactivé sur le périphérique

```bash
vtp mode off
```

## VTP Prunning

Lors de l'activation de VTP pruning sur le switch server en VTP v1/v2, l'ensemble du domaine VTP va mettre en place le pruning, VTP v3 nécessite de l'activer sur chaque switch

Le protocole VTP a intégré l’option de “pruning”, cette fonctionnalité permet de limiter la propagation des broadcasts aux VLAN qui sont présents sur chaque lien du réseau

Les switches de chaque VLAN échangent des messages pour déterminer quels VLAN sont présents et utilisé sur chaque lien et ajustent automatiquement leur configuration de pruning en conséquence.

Ainsi, lorsqu’un PC envoie un broadcast, il ne sera transmis qu’aux switches qui ont des interfaces appartenant au même VLAN, permettant d’optimiser la bande passante du réseau et de réduire la congestion de trafic inutile.

```rust
vtp prunning
```

## Version VTP

### Version 1

- Supporte la porté de VLAN de 1 à 1001
- Support du pruning
- Mot de passe en clair ou en MD5
- En mode transparent, contrôle le nom de domaine et numéro de révision avant de transférer le message VTP, seulement si les informations correspondent

### Version 2

- Support de Token-Ring
- Transmet le message, même si les TLV ne sont pas reconnu
- VTP v1 et v2 ne sont pas inter-opérable
- Si un switch utilise VTPv2, tous les switch vont utilisé VTPv2 si ils sont en capacité de le faire
- Ne transmet pas des messages VTP v1 ou v3

### Version 3

- Important dans la configuration, stipuler qui est le `primary server`
- Possibilité de " cacher le mot de passe " dans la running-config
- Support VLAN étendu 1 - 4095
- Support private VLAN
- Support RSPAN
- Mot de passe sauvegarde en crypté dans la configuration
- Seulement le VTP serveur peut modifié la configuration, si un switch est ajouté avec un numéro de révision supérieur, il ne peut pas modifié la configuration VTP
- Ajout des rôles VTP Server Primary et VTP server Secondary, seul le primary peut faire des modifications, les secondary servent en backup
- Primary server n'accepte pas les mises à jours d'autres serveurs VTP
- Support de VTP mode OFF
- Si le mot de passe est supprimé, le serveur vtp primary retourne en configuration initial, ne peut plus être fait en passant VTP en mode transparent
- Inter-opérable avec VTP v2
- Ne peut pas passer de V3 à V2 si des private-vlan sont actuellement pris en charge par VTP
- Si message reçu sur un port d'un VTP V2, VTP v3 va automatiquement se scaler sur la version du client
- VTP v3 à besoin de la configuration de spanning-tree extended system-id for spanning-tree
- VTP v3 ne relais pas les messages VTP v1/v2
- Changer VTP mode OFF à un autre mode, si des VLAN supérieur à 1005 existent, ils seront supprimé

## Sécurité

Quand un mot de passe est configuré avec la commande `show vtp password`, le mot de passe apparait en clair.

Il est recommandé, lors de la définition du mot de passe, de rajouter le paramètre `hidden`, va générer un mot de passe crypté permettant de le masque dans le fichier de configuration

```rust
switch(config)# vtp password <password> hidden
```

## Message VTP

Les messages VTP se font dans l'ordre

- Summary Advertisement
- Subset Advertisement
- Advertisement Request

### Summary Advertisement

Quand une modification des VLAN est apporté sur le Switch " VTP Server ", un message Summary Advertisement est envoyé aux clients pour les informer d'une mise à jour du fichier vlan.dat. Le Summary Advertisement est aussi, envoyé par défaut toutes les 5mn

Si aucune différence est constaté, aucun autre message ne sera envoyé.

### Subset Advertisement

Dans le cas ou une différence entre le Summary advertisement et l’état du client est constaté, le serveur VTP va émettre un Subset Advertisement pour mettre à jour le ficher vlan.dat du client.

Cette annonce va permettre de demander au serveur VTP de lui envoyé sa configuration actuelle pour pouvoir se mettre à jour.

### Advertisement Request

Messages envoyé par les clients VTP au serveur VTP pour, vérifié si les clients VTP n'ont pas d'information manquantes

## Configuration

### Configration VTPv2

#### SW1 - VTP Master

```rust
vlan 10
vlan 20
vlan 30
vlan 99
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
vtp version 2
vtp domain lab.lan
vtp mode server
vtp password lab hidden
vtp pruning
```

#### SW2 - VTP Slave

```rust
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
vtp version 2
vtp domain lab.lan
vtp mode client
vtp password lab hidden
```

### Configuration VTPv3

#### SW1 - VTP Server Primary

```rust
vlan 10
vlan 20
vlan 30
vlan 99
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
router(config)# vtp domain
router(config)# vtp version 3
router(config)# vtp password lab hidden
router(config)# vtp mode server
router(config)# vtp pruning
router# vtp primary
```

#### SW2 - VTP Server secondary

```rust
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
router(config)# vtp domain
router(config)# vtp version 3
router(config)# vtp password lab hidden
router(config)# vtp mode server
```

## Administration

### Maintenance

- Voir l’état de la configuration VTP avec la commande, show vtp status
- Voir si les VLANs attendus existent, show vlan
- Voir les messages VTP, show vtp counters
- Documentation

### Sécurité

- Utiliser un seul serveur vtp par domaine
- version 3 si possible
- mot de passe vtp
- Désactiver VTP si non utilisé
- Forcer le mode access sur les ports utilisateurs

### Dépannage

Si les vlans ne redescendent pas:

- Vérifier que VTP est bien configurer
- Que les ports connectant les switch du domaine vtp soient en mode trunk
- Port trunk soit bien en no shutdown

Voir les messages de dépannage avec les commandes:
- debug sw-vlan vtp events
- debug sw-vlan vtp packets

Vlan disparu
- Vérifier si un switch n’a pas un numéro révision supérieur, isoler le switch

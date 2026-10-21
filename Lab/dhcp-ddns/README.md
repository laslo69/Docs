# DHCP & DNS en haute disponibilite

## Projet

### Contexte

Dans le cadre de ce projet, j'ai mis en place une infrastructure réseau sous Linux associant un service DHCP à un service DNS dynamique (DDNS).

L'objectif est d'automatiser l'attribution des adresses IP aux clients et de maintenir à jour les enregistrements DNS correspondants, le tout en sécurisant les échange entre les DHCP et les DNS.

***Fichiers de configuration complet en annexe***

***

### Problematique

Comment automatiser l'attribution des adresses IP et la mise à jour des enregistrements DNS afin de simplifier l'administration d'un réseau ?

Comment faire en sorte, de sécuriser l'échange pour s'assurer qu'un serveur DNS non autorisé ne puisse pas se synchroniser avec les DNS existant ?

Si un des serveurs DHCP ou DNS tombe en panne, comment rendre l'infrastructure résiliante ?

***

### Difficulte & Observation

Avant de commencer, il y'a plusieurs points intéressants et important, qui m'ont fait perdre du temps bêtement, que je souhaite aborder

1) **Documentation kea & bind**

Avant d'entamer le lab, il à fallu prendre connaissance des documentations, pour repérer les options utiles et me les notés

Je ne sais que dire.... une documentation massive et brouillon pour kea-dhcp ce qui rend la compréhension très difficile

A l'inverse pour bind, la documentation est relativement pauvre ce qui peut induire, de passer à côté des options ou informations

2) **La configuration JSON**

Sur l'ancien DHCP du consortium ISC, la syntaxe était en bash. Syntaxe simple à lire, écrire, mais depuis que isc-dhcp est obsolète et remplacé par kea-dhcp, la syntaxe est passé en JSON, qui est une syntaxe que j'ai trouvé plus "lourde" et plus difficile à lire

3) **Kea-dhcp & apparmor**

Un autre détail important de kea-dhcp est, lors du boot de la machine ( en l'occurence, une VM pour ma part ), le service était continuellement en erreur, même si la syntaxe est bonne.

Le coupable est, après de longue recherche et test, AppArmor. Apparmor bloque le service kea-dhcp et nécessite de créer, que ce soit temporaire ou permanent, une autorisation pour que le service soit fonctionnel

En solution temporaire de fortune mais fonctionnel, j'ai ajouté une entrée dans cron pour que au boot de la machine, la commande soit automatiquement validé et le service fonctionnem

```bash
@reboot    apparmor_parser -R /etc/apparmor.d/usr.sbin.kea-dhcp4
```

4) **kea-dhcp & cle TSIG**

J'ai rencontré une autre difficulté en cours de route, lors de la génération de la clé TSIG pour kea-dhcp, le serveur DHCP n'a jamais voulu validé l'utilisation de la clé et se mettait constamment en erreur.

Alternative un peu moins sûre mais tolérable, est d'inscrire en solide la clé dans la configuration

5) **Répertoire `/etc/bind/`**

Il y'a un détail intéressant sur bind, ou plutôt sur le répertoire `/etc/bind`. Par défaut, bind est ***incapable*** de travailler dans ce répertoire sans avoir à casser les permissions Unix, c'est de ce fait, que les zones DNS sont enregistrés dans le dossier `/var/cache/bind/`

***

### Objectif

- Automatiser l'attribution des adresses IP via DHCP
- Configurer un service DNS pour la résolution des noms de machines
- Mettre en place les mises à jour DNS dynamiques
- Vérifier la cohérence entre les adresses IP attribuées et les enregistrements DNS.
- Valider le fonctionnement de l'infrastructure à l'aide de tests
- Reduction des SPOF
- Utilisation de clé TSIG pour sécuriser l'échange entre les DHCP et les DNS, mais aussi pour la réplication entre DNS

## Architecture & Environnement

- Hyperviseur : Proxmox
- Système d'exploitation : OPNsense, Debian
- Service DHCP :
    - kea-dhcp4-server
    - kea-dhcp-ddns-server
    - kea-ctrl-agent
- Service DNS : Bind9
- Réseau :
    - LAN local : 192.168.0.0/24
    - LAN 10    : 192.168.10.0/24
    - LAN 20    : 192.168.20.0/24

![topologie](./illustration/topologie.png)

## Configuration

### OPNsense

La première étape csonsite, à configurer les port sur OPNSense pour leur assigner un interface et une adresse IP

Ensuite mettre en place le relais DHCP, pour permettre à mon LAN client de communiquer avec le LAN où, les serveurs DHCP se situent et permettre de transmettre les messages DHCP.

![dhcp relay](./illustration/dhcp-relay.png)

***

### DHCP

Kea-dhcp enregistre touts les leases dans un fichier créer lors de l'installation, qui se situe dans : `/var/lib/kea/kea-leases4.csv` et est mis à jour dynamiquement

Les fichiers de configurations du serveur DHCP principale peuvent être copié sur l'autre serveur, pour être sûr d'avoir une configuration identique mais nécessiterons d'être adapté sur certains points

Note : La configuration du DHCP,DDNS et du control-agent se fait en `JSON`

***

#### Configuration DHCP

Le fichier `/etc/kea/kea-dhcp4.conf` étant déja existant lors de l'installation, il est recommandé de faire un copie du fichier de base avant de faire toutes modifications

La configuration du  bloc DHCP se compose de plusieurs sections:

- Timers
- Interface
- Gestion leases
- Gestion Sous-réseaux, pools & options DHCP

***

Eplications sur la configuration de chaque bloc avec un exemple

1) **Timers ( exprimé en secondes )**

La définissions des timers, va permettre de donner les directives de, quand demander une mise à jour du lease, sa durée de vie et dans le cas échéant, supprimer le lease obsolète du fichier de leases

- "valid-lifetime": 2000  : Durée validité du leases
- "renew-timer": 500      : Timer T1, délais avant l'émission d'un message DHCP RENEW afin de conserver le lease 
- "rebind-timer": 1000    : Timer T2, similaire à renew-timer mais se produit dans le cas ou, le serveur DHCP ne répond pas à la requête du "renew-timer"

***

2) **Interface**

Spécifie l'interface d'écoute pour la réception de trame DHCP, plusieurs interfaces peuvent être spécifié

Par défaut, l'interface écoute en " raw ", ce qui permet de traiter tous les messages DHCP, dans le cas où, le DHCP est un relais DHCP, il est nécessaire de mettre l'interface en écoute " UDP "

- "interfaces": [ "ens18" ] : Désigne l'interface d'écoute pour la réception de message DHCP

***

3) **Gestion leases**

Configure le comportement de kea-dhcp4, sur comment les leases doivent être stockés, ou, comment gérer la supppression des leases obsolète et le nombre d'erreurs autoriser avant que le fichier, ne soit plus lu

- "type": "memfile" : Désigne comment enregistrer les leases, "memfile" est le fichier `/var/lib/kea/kea-leases4.csv`
- "persist": true : Dans le cas d'un redémarrage, les leases restent enregistrer dans le fichiers
- "name": "/var/lib/kea/kea-leases4.csv" : Chemin du fichiers de lease
- "lfc-interval": 800 : Délai avant que le fichier contenant les leases soit nettoyer de tout les enregistrements anciens
- "max-row-errors": 100 : définit le nombre maximal de lignes corrompues ou erronées que Kea tolérera lors de la lecture d'un fichier de bail avant d'interrompre le chargement et de se fermer

**Note : L'utilisation du "memfile" est valide dans le cas d'un DHCP unique ou d'un ensemble de 2 DHCP en High-availability. Dans le cas ou 3 serveurs DHCP ou plus sont utilisés en cluster, il sera nécessaire de passer par une BDD**

***

4) **Gestion sous-réseaux**

Définission des réseaux, dont l'attribution des IP,option DHCP et pools, est géré par kea-dhcp

- "id": 1 : ID du réseau à déclarer, à incrémenter pour chaque nouveau réseau
- "subnet": "192.168.10.0/24" : Définir le réseau au format CIDR
- "pools": : Déclaration du/des pools réseau, plusieurs pools peuvent être déclaré tant que, non chevauchant
- "pool": "192.168.10.20 - 192.168.10.30"
- "option-data": Définition des options DHCP, met à jour pour les clients le fichier `/etc/resolv.conf`
- "name": "<name>" : Indiquer, soit le code de l'option, soit le nom de l'option
- "data": "<data>" : Indique la valeur correspondant à l'option 

```json
{
    "Dhcp4":
    {
        "valide-lifetime": 2000,
        "renew-timer": 500,
        "rebind-timer": 1000,
        "interfaces-config":
        {
            "interfaces": [ "ens18" ]
        },
        "lease-database":
        {
            "type": "memfile",
            "persist": true,
            "name": "/var/lib/kea/kea-leases4.csv",
            "lfc-interval": 800,
            "max-row-errors": 100
        },
        "subnet4":
        [
            {
                "id": 1,
                "subnet": "192.168.10.0/24",
                "pools":
                [
                    {
                        "pool": "192.168.10.20 - 192.168.10.30"
                    }
                ],
                "option-data":
                [
                    {
                        "name": "domain-name-servers",
                        "data": "192.168.10.5,192.168.10.6"
                    },
                    {
                        "name": "domain-name",
                        "data": "lab.lan"
                    },
                    {
                        "name": "domain-search",
                        "data": "lab.lan"
                    },
                    {
                        "name": "routers",
                        "data": "192.168.10.1"
                    }
                ]
            },
            {
                "id": 2,
                "subnet": "192.168.20.0/24",
                "pools":
                [
                    {
                        "pool": "192.168.20.10 - 192.168.20.40"
                    }
                ],
                "option-data":
                [
                    {
                        "name": "domain-name-servers",
                        "data": "192.168.10.5,192.168.10.6"
                    },
                    {
                        "name": "domain-name",
                        "data": "lab.lan"
                    },
                    {
                        "name": "domain-search";
                        "data": "lab.lan"
                    },
                    {
                        "name": "routers",
                        "data": "192.168.20.1"
                    }
                ],
                
            }
        ]
    }
}
```

***

#### Configuration DHCP-DDNS

La configuration du DHCP-DDNS se fait à travers 2 fichiers

- kea-dhcp4.conf
- kea-dhcp-ddns.conf

***

**kea-dhcp4.conf**

Dans le fichier `kea-dhcp4.conf`, la configuration du service kea-ddns va permettre d'indiquer, à kea-dhcp, comment interagir entre le DHCP et le service DDNS, mais aussi comment le NCR ( Name Change Request ) doit compléter le fonctionnement du ddns et la mise à jour des enregistrements DNS

- "enable-updates": true : Active globalement l'envoi des requêtes de mise à jour DNS par le serveur DHCP
- "server-ip": "127.0.0.1" : adresse IP sur laquelle le démon DDNS écoute
- "server-port": 53001 : Port réseau utilisé par Kea DDNS pour recevoir les demandes du serveur DHCP
- "sender-port": 0 : Port source utilisé par le serveur DHCP pour envoyer ses messages. 0 indique à Kea de laisser le système d'exploitation choisir un port éphémère libre de manière dynamique
- "ncr-protocol": "UDP" : Protocole réseau utilisé pour le transport des requêtes NCR
- "ncr-format": "JSON" : Format des données encapsulées dans les requêtes de mise à jour
- "ddns-send-updates": true : Confirme l'envoi des requêtes
- "ddns-override-client-update": true : Ignore la configuration ou la préférence du client
- "ddns-override-no-update": true :  : Si un client demande explicitement à ne pas publier son nom dans le DNS, le serveur ignore sa demande
- "ddns-qualifying-prefix": "lab.lan" : Nom de domaine ajouté automatiquement, à la fin du nom d'hôte
- "ddns-generated-prefix": "dhcp" : Préfixe pour générer un nom générique si le client ne fournis pas de non d'hôte valide ( souvent : dhcp-192.168.20.10 )
- "ddns-replace-client-name": "when-not-present" : Si le nom du client n'est pas fournis lors de la requête, kea-dhcp en génère un à partir du prefix + de l'IP attribué

```json
...
    "dhcp-ddns":
    {
        "enable-updates": true,
        "server-ip": "127.0.0.1",
        "server-port": 53001,
        "sender-port": 0,
        "ncr-protocol": "UDP",
        "ncr-format": "JSON"
        },
        "ddns-send-updates": true,
        "ddns-override-client-update": true,
        "ddns-override-no-update": true,
        "ddns-qualifying-prefix": "lab.lan",
        "ddns-generated-prefix": "dhcp",
        "ddns-replace-client-name": "when-not-present",
...
```

***

**kea-ddns.conf**

Le fichier `kea-ddns.conf` va permettre, de spécifier comment le service DHCP-DDNS va communiquer avec le serveur DNS pour pouvoir, ajouter/modifier des enregistrements

Le bloc "tsig-keys" permet de renseigner une clé TSIG, qui sera ajouté sur les serveurs DNS pour autoriser les ajouts DNS provenant de ce DHCP, via l'authentification par la clé TSIG

La section forward-ddns va permettre de spécifier, vers quel serveur pousser les modifications DNS, mais surtout en spécifiant la zone cible. 

La déclaration des zones, qu'elles soient directe comme inverse se fait de la même manière

Dans ce bloc, l'utilisation de la clé TSIG va permettre de signer les requêtes DNS provenant du DDNS en direction du DNS

La clé TSIG peut être générer avec la commande `tsig-keygen -a hmac-sha256 my-tsig-key > /etc/bind/key/ma-cle-ddns.key`

- "tsig-keys":    Déclaration de la clé TSIG
- "name": "ma-cle-ddns.key"     : Nom de la clé TSIG
- "algorithm": "hmac-sha256"    : Algorithme utilisé par la clé
- "secret": "HGj/s1alczSQMz8GAmVG/nwIsd0JiDxTX2whyXpk+MA="  : Secret de la clé

"forward-ddns" et "reverse-ddns" se déclare de la même façon

- "forward-ddns":
- "ddns-domains":
- "name": "lab.lan."    : Nom de la zone à mettre à jour
- "key-name": "ma-cle-ddns.key" : Déclaration de la clé pour signer les mise à jours DNS
- "dns-servers":        : Déclaration des serveurs DNS
- "ip-address": "192.168.10.5" : Adresse IP du serveur DNS
- "port": 53    : Port écoute du server DNS
- "reverse-ddns":
- "ddns-domains":
- "name": "10.168.192.in-addr.arpa.",
- "key-name": "ma-cle-ddns.key",
- "dns-servers":
- "ip-address": "192.168.10.5",
- "port": 53

```json
{
    "DhcpDdns":
    {
        "tsig-keys":
        [
            {
                "name": "ma-cle-ddns.key",
                "algorithm": "hmac-sha256",
                "secret": "HGj/s1alczSQMz8GAmVG/nwIsd0JiDxTX2whyXpk+MA="
            }
        ],
        "forward-ddns":
        {
            "ddns-domains":
            [
                {
                    "name": "lab.lan.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                }
            ]
        },
        "reverse-ddns":
        {
            "ddns-domains":
            [
                {
                    "name": "10.168.192.in-addr.arpa.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                },
                {
                    "name": "20.168.192.in-addr.arpa.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                }
            ]
        }
    }
}
```

***

#### Configuration reservation

Le bloc réservation, va permettre de créer les réservations pour des clients DHCP, et est assez simple à gérer

- "hw-address": "bv:24:11:37:b5:a2" : Spécifie l'adresse MAC de l'interface du client
- "ip-address": "192.168.20.10", : Désigne une adresse IP réservé pour le client avec l'adresse MAC correspondante
- "user-context": { "comment": "proxmox manager" } : User-context va servire, en commentaire pour simplifier la lecture des réservations

```json
...
"reservations":
                [
                    {
                        "hw-address": "bv:24:11:37:b5:a2",
                        "ip-address": "192.168.20.10",
                        "user-context": { "comment": "proxmox manager" }
                    },
                    {
                        "hw-address": "bc:24:11:1d:19:bd",
                        "ip-address": "192.168.20.11",
                        "user-context": { "comment": "px-pve-1" }
                    },                    {
                        "hw-address": "bc:24:11:95:4a:06",
                        "ip-address": "192.168.20.12",
                        "user-context": { "comment": "px-pve-2" }
                    },                    {
                        "hw-address": "nv:24:11:d1:32:8d",
                        "ip-address": "192.168.20.13",
                        "user-context": { "comment": "px-pve-3" }
                    },                    {
                        "hw-address": "bc:24:11:f4:a2:27",
                        "ip-address": "192.168.20.14",
                        "user-context": { "comment": "proxmox-backup" }
                    },                    {
                        "hw-address": "bc:24:11:34:bf:f6",
                        "ip-address": "192.168.20.15",
                        "user-context": { "comment": "proxmox-mail" }
                    }
                ]
...
```

***

#### Configuration Load-balance

La configuration du load-balance passe par des "hooks", qui peuvent fonctionner en 2 modes différent, qui est géré nativement par kea-dhcp, ce qui évite de passer par des outils intermédiaire

- fail-over
- load-balance

Dans le bloc " high-availability ", il faut spécifié, le mode de fonctionnement, les timers du heartbeat, le nom du serveur local et les pairs membres

- "this-server-name": "dhcp-ddns-1" : Décrit le nom du serveur actuel, si le fichier est copié, modifier le nom pour les autres serveurs
- "mode": "load-balancing" : Défini le mode de high availability
- "heartbeat-delay": 10000 : Définit le timer du heartbeat en `ms`
- "max-response-delay": 60000 : Délai imparti avant de considéré un pair, comme étant down
- "peers":
- "name": "dhcp-ddns-1" : Défini le nom du pair, même le serveur local
- "url": "http://192.168.10.5:8005/" : URL d'écoute de l'API de la haute disponibilité, le port doit être le même pour chaque pair sur chaque serveur
- "role": "primary" : Spécifie si le serveur est le primary ou secondary
- "auto-failover": true : Défini si le fail-over automatique doit être utilisé
- "user": "kea-api" : utilisateur défini dans `/etc/kea/kea-ctrl-agent`
- "password": "test" : mot de passe défini dans `/etc/kea/kea-api-password` et du chemin défini dans `/etc/kea/kea-ctrl-agent`

```json
...
"hooks-libraries":
        [
            {
                "library": "libdhcp_leases_cmds.so",
                "parameters": {}
            },
            {
                "library": "libdhcp_ha.so",
                "parameters": {
                    "high-availability": [
                        {
                            "this-server-name": "dhcp-ddns-1",
                            "mode": "load-balancing",
                            "heartbeat-delay": 10000,
                            "max-response-delay": 60000,
                            "peers":
                            [
                                {
                                    "name": "dhcp-ddns-1",
                                    "url": "http://192.168.10.5:8005/",
                                    "role": "primary",
                                    "auto-failover": true,
                                    "user": "kea-api",
                                    "password": "test"
                                },
                              {
                                    "name": "dhcp-ddns-2",
                                    "url": "http://192.168.10.6:8005/",
                                    "role": "secondary",
                                    "auto-failover": true,
                                    "user": "kea-api",
                                    "password": "test"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
...
```

***

#### DHCP control agent

Dans le fichier `/etc/kea/kea-ctrl-agent.conf`, il y'a 2 blocs à modifier

- "control-agent"
- "authentication"

Contrôle agent va servir à configurer comment l'API doit communiquer  et contrôler l'authentification des serveurs DHCP

- "http-host": "0.0.0.0" : Indique l'IP à écouté
- "http-port": 8000 : Indique le port utilisé ( 8000 par défaut )

```bash
{
...
"Control-agent": {
    "http-host": "0.0.0.0",
    "http-port": 8000,
    }
}
...
```

Ensuite, il faut configurer l'authentification

Dans le bloc client, il est attendu un fichier contenant le mot de passe et son chemin, que j'ai simplement appelé "kea-api-password"

```bash
...
"authentication":
{
    "type": "basic",
    "realm": "Kea Control Agent",
    "directory": "/etc/kea",
    "clients": [
        {
            "user": "kea-api",
            "password-file": "kea-api-password"
        }
    ]
},
...
```

Il faut bien penser à créer le fichier `/etc/kea/kea-api-password` dans `/etc/kea/` et écrire uniquement le mot de passe à l'intérieur

```bash
echo "test" > /etc/kea/kea-api-password
```

***

### DNS

#### Déclaration zone - Master

Le fichier peut être copié sur le DNS secondaire mais nécessite des légères modifications

Le fichier `named.conf.local` va servire à déclarer les zones DNS dont le serveur fait autorité, chaque zone doit être renseigné.

Dans le cas d'une haute disponibilité de bind, la designation de rôle est nécessaire pour désigner qui à quel rôle dans la relation master/slave

La génération d'une clé TSIG et l'utilisé pour le transfert de zone AXFR ou IXFR est recommandé pour pouvoir, par le biais d'une signature, de contrôler les serveurs DNS qui ont l'autorisation d'envoyer ou recevoir des mises à jours

Aussi, l'utilisation des clés DDNS généré sur les serveurs DHCP va permettre d'authentifier et d'autoriser les mises à jours provenant uniquement de ces serveurs, l'utilisation de clé distincte entre DHCP et DNS est recommandé

- include : Renseigne les clés DNS
- zone :    Défini le rôle, emplacement et autorisation du serveur sur la zone
    - type : Peux être soit master ou slave
    - file : Désigne l'emplacement logique de la zone
    - allow-update : Définie explicitement, quels serveurs à le droit de mettre à jour le DNS, ici par le biais d'une clé TSIG
    - allow-transfer :  Autorise le serveur DNS, à pouvoir faire un transfert de zone vers un serveur membre de la High-availability
    - notify :  Lors d'une mise à jour DNS, signale les membres de la haute-disponibilité du mise à jour, un échange IXFR à lieu pour mettre à jours les DNS slave, voir dans la configuration du DNS slave pour faire la jonction

```bash
include "/etc/bind/key/ma-cle-tsig.key";
include "/etc/bind/key/ma-cle-ddns.key";

zone "lab.lan"
{
    type master;
    file "/var/cache/bind/db.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;
};
zone "10.168.192.in-addr.arpa"
{
    type master;
    file "/var/cache/bind/db-reverse10.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;
};
zone "lab.lan"
{
    type master;
    file "/var/cache/bind/db-reverse20.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;    
};
```

#### Déclaration zone - Slave

Il y'a un détail important dans une relation Master/slave avec bind lors de la déclaration des zones

La déclaration est très similaire mais nécessite des corrections

- La déclaration des clés reste inchangé
- Le type de la zone doit être "slave"

Cependant, il faut ajouter 2 lignes

- masters { 192.168.10.5 key "ma-cle-tsig.key"; };
- allow-update-forwarding { key "ma-cle-tsig.key"; };

Pour expliquer l'utilité de ces lignes et l'importance

- masters : Bind ne sait pas faire du load-balancing, mais uniquement du fail-over. En désignant le serveur "master", le serveur sait à quelle addresse IP demander des mises à jours DNS.
- allow-update-forwarding : Imaginons que le DNS master tombe, mais que le serveur secondaire enregistre les nouvelles requêtes, quand le "master" reviens en état fonctionnel, il y'aura une incohérence dans les enregistrements. Cette options sert à signalé au "master", qu'il à un numéro de série supérieur et va donc lui transmettre les enregistrements pour le mettre à jour

```bash
include "/etc/bind/key/ma-cle-tsig.key";
include "/etc/bind/key/ma-cle-ddns.key";

zone "lab.lan"{
    type slave;
    masters { 192.168.10.5 key "ma-cle-tsig.key"; };
    file "/var/cache/bind/db.lab.lan";
    allow-transfer { key "ma-cle-ddns.key"; } ;
    allow-update-forwarding { ma-cle-tsig.key"; };
};
zone "10.168.192.in-addr.arpa"{
    type slave;
    masters { 192.168.10.5 key "ma-cle-tsig.key"; };
    file "/var/cache/bind/db.lab.lan";
    allow-transfer { key "ma-cle-ddns.key"; } ;
    allow-update-forwarding { ma-cle-tsig.key"; };
};
zone "20.168.192.in-addr.arpa"{
    type slave;
    masters { 192.168.10.5 key "ma-cle-tsig.key"; };
    file "/var/cache/bind/db.lab.lan";
    allow-transfer { key "ma-cle-ddns.key"; } ;
    allow-update-forwarding { ma-cle-tsig.key"; };
};
```

#### Création zone DNS

La création des zones doit se fare dans le répertoire `/var/cache/bind/`, comme désigné dans le fichier de configuration

**zone directe**

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        1          ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

**zone reverse LAN 10**

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        1          ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

**zone reverse LAN 20**

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        1          ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

#### Configuration DNS

Le fichier `named.conf.options` va permettre de définir des règles de fonctionnement du DNS et peut être copié sur chaque serveur DNS

Ces règles peuvent être du filtrage, comment rediriger une requête si elle est destiné à un réseau extérieur du domaine local, définir les interfaces d'écoute

- ACL : Ce bloc va permettre de définir, quels réseaux sont autorisé à envoyer des requêtes sur le serveur DNS
- Options : Contient différentes directives sur le fonctionnement du DNS
    - Directory : Renseigne le répertoire de travail de bind et contient les zones DNS
    - Version   : Dans le cas d'un scan réseau, le numéro de version de bind peut apparaitre. Ajouter une valeur fictive va permettre de masquer la version de bind
    - Forwarders : Renseigne des DNS externes, dans le cas ou une résolution DNS ne correspond à aucun enregistrement local
- Recursion : Va permettre d'autoriser ( dans le cas de "yes" ) de résoudre des noms de domaines dont le serveur, n'est pas le serveur d'autorité sur le domain en question
- listen-on { any; }    : Spécifie l'interface d'écoute des requêtes DNS
- allow-query { lan; }; : Utilisé avec une ACL bind, va permettre d'autoriser uniquement la ou les ACL renseigner, à envoyé des requêtes sur le serveur DNS

```bash
acl "lan"{
    192.168.10.0/24;
    192.168.20.0/24;
    localhost;
    localnets;
};
options {
    directory "/var/cache/bind";
    version "not available";
    forwarders {
        1.1.1.1;
        9.9.9.9;
    };
    recursion yes;
    listen-on { any; };
    allow-query { lan; };
};
```

## Test

serie de test

### DHCP

dhcp primary seul pour attribution ip

![dhcp-alone](./illustration/dhcp-alone.png)

dhcp en ha pour replication fichier lease

![dhcp-ha](./illustration/dhcp-ha.png)

dhcp 1 off pour voir si dhcp 2 attribue une ip

![dhcp-sync-1](./illustration/dhcp-sync-1.png)

![dhcp-sync-2](./illustration/dhcp-sync-2.png)

### DNS

dns 1 seul ajout dhcp client dans dns

dns 1 + 2 pour check axfr/ixfr

dns2 pour voir si dhcp ajoute bien dans secondary

dns1+2 pour voir si dns2 forward l'update vers le master

## Annexe

### DHCP

#### Kea-dhcp4

```json
{
    "Dhcp4":
    {
        "hooks-libraries":
        [
            {
                "library": "libdhcp_leases_cmds.so",
                "parameters": {}
            },
            {
                "library": "libdhcp_ha.so",
                "parameters": {
                    "high-availability": [
                        {
                            "this-server-name": "dhcp-ddns-1",
                            "mode": "load-balancing",
                            "heartbeat-delay": 10000,
                            "max-response-delay": 60000,
                            "peers":
                            [
                                {
                                    "name": "dhcp-ddns-1",
                                    "url": "http://192.168.10.5:8005/",
                                    "role": "primary",
                                    "auto-failover": true,
                                    "user": "kea-api",
                                    "password": "test"
                                },
                              {
                                    "name": "dhcp-ddns-2",
                                    "url": "http://192.168.10.6:8005/",
                                    "role": "secondary",
                                    "auto-failover": true,
                                    "user": "kea-api",
                                    "password": "test"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "dhcp-ddns":
        {
            "enable-updates": true,
            "server-ip": "127.0.0.1",
            "server-port": 53001,
            "sender-port": 0,
            "ncr-protocole": "UDP",
            "ncr-format": "JSON"
        },
        "ddns-send-updates": true,
        "ddns-override-client-update": true,
        "ddns-override-no-update": true,
        "ddns-qualifying-prefix": "lab.lan",
        "ddns-generated-prefix": "dhcp",
        "ddns-replace-client-name": "when-not-present",
        "valide-lifetime": 2000,
        "renew-timer": 500,
        "rebind-timer": 1000,
        "interfaces-config":
        {
            "interfaces": [ "ens18" ]
        },
        "lease-database":
        {
            "type": "memfile",
            "persist": true,
            "name": "/var/lib/kea/kea-leases4.csv",
            "lfc-interval": 800,
            "max-row-errors": 100
        },
        "subnet4":
        [
            {
                "id": 1,
                "subnet": "192.168.10.0/24",
                "pools":
                [
                    {
                        "pool": "192.168.10.20 - 192.168.10.30"
                    }
                ],
                "option-data":
                [
                    {
                        "name": "domain-name-servers",
                        "data": "192.168.10.5,192.168.10.6"
                    },
                    {
                        "name": "domain-name",
                        "data": "lab.lan"
                    },
                    {
                        "name": "domain-search",
                        "data": "lab.lan"
                    },
                    {
                        "name": "routers",
                        "data": "192.168.10.1"
                    }
                ]
            },
            {
                "id": 2,
                "subnet": "192.168.20.0/24",
                "pools":
                [
                    {
                        "pool": "192.168.20.10 - 192.168.20.40"
                    }
                ],
                "option-data":
                [
                    {
                        "name": "domain-name-servers",
                        "data": "192.168.10.5,192.168.10.6"
                    },
                    {
                        "name": "domain-name",
                        "data": "lab.lan"
                    },
                    {
                        "name": "domain-search";
                        "data": "lab.lan"
                    },
                    {
                        "name": "routers",
                        "data": "192.168.20.1"
                    }
                ],
                "reservations":
                [
                    {
                        "hw-address": "bv:24:11:37:b5:a2",
                        "ip-address": "192.168.20.10",
                        "user-context": { "comment": "proxmox manager" }
                    },
                    {
                        "hw-address": "bc:24:11:1d:19:bd",
                        "ip-address": "192.168.20.11",
                        "user-context": { "comment": "px-pve-1" }
                    },                    {
                        "hw-address": "bc:24:11:95:4a:06",
                        "ip-address": "192.168.20.12",
                        "user-context": { "comment": "px-pve-2" }
                    },                    {
                        "hw-address": "nv:24:11:d1:32:8d",
                        "ip-address": "192.168.20.13",
                        "user-context": { "comment": "px-pve-3" }
                    },                    {
                        "hw-address": "bc:24:11:f4:a2:27",
                        "ip-address": "192.168.20.14",
                        "user-context": { "comment": "proxmox-backup" }
                    },                    {
                        "hw-address": "bc:24:11:34:bf:f6",
                        "ip-address": "192.168.20.15",
                        "user-context": { "comment": "proxmox-mail" }
                    }
                ]
            }
        ]
    }
}
```

#### Kea-dhcp-ddns

Fichier `/etc/kea-dhcp-ddns.conf`, a copier sur chaque serveur kea-dhcp

```json
{
    "DhcpDdns":
    {
        "tsig-keys":
        [
            {
                "name": "ma-cle-ddns.key",
                "algorithm": "hmac-sha256",
                "secret": "HGj/s1alczSQMz8GAmVG/nwIsd0JiDxTX2whyXpk+MA="
            }
        ],
        "forward-ddns":
        {
            "ddns-domains":
            [
                {
                    "name": "lab.lan.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                }
            ]
        },
        "reverse-ddns":
        {
            "ddns-domains":
            [
                {
                    "name": "10.168.192.in-addr.arpa.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                },
                {
                    "name": "20.168.192.in-addr.arpa.",
                    "key-name": "ma-cle-ddns.key",
                    "dns-servers":
                    [
                        {
                            "ip-address": "192.168.10.5",
                            "port": 53
                        },
                        {
                            "ip-address": "192.168.10.6",
                            "port": 53
                        }
                    ]
                }
            ]
        }
    }
}
```

#### kea-ctrl-agent

Modifier la section "contro-agent" et "authentication" de `/etc/kea/kea-ctrl-agent` et modifier le port http

control-agent

```json
{
...
"Control-agent": {
    "http-host": "0.0.0.0",
    "http-port": 8000,
    ...
    }
}
```

authentication

```json
...
"authentication":
{
    "type": "basic",
    "realm": "Kea Control Agent",
    "directory": "/etc/kea",
    "clients": [
        {
            "user": "kea-api",
            "password-file": "kea-api-password"
        }
    ]
},
...
```

Créer dans `/etc/kea/` un fichier `kea-api-password` et écrire uniquement le mot de passe

```bash
test
```

### DNS

#### named.conf.local

Inclure les cles TSIG et renseigner les zones DNS

```bash
include "/etc/bind/key/ma-cle-tsig.key";
include "/etc/bind/key/ma-cle-ddns.key";

zone "lab.lan"
{
    type master;
    file "/var/cache/bind/db.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;
};

zone "10.168.192.in-addr.arpa"
{
    type master;
    file "/var/cache/bind/db-reverse10.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;
};

zone "lab.lan"
{
    type master;
    file "/var/cache/bind/db-reverse20.lab.lan";
    allow-update { key "ma-cle-ddns.key"; };
    allow-transfer { key "ma-cle-tsig.key; };
    notify yes;    
};
```

#### named.conf.options

```bash
acl "lan"{
    192.168.10.0/24;
    192.168.20.0/24;
    localhost;
    localnets;
};
options {
    directory "/var/cache/bind";
    version "not available";
    forwarders {
        1.1.1.1;
        9.9.9.9;
    };
    recursion yes;
    dnssec-validation auto;
    listen-on { any; };
    allow-query { lan; };
};
```

#### zone directe

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        2001062501 ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

#### zone reverse

LAN 10

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        2001062501 ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

LAN 20

```bash
$ORIGIN lab.lan 
$TTL 86400 
                @   	IN  	SOA 	dns1.lab.lan.	admin.lab.lan. (
                                        2001062501 ; serial                     
                                        21600      ; refresh after 6 hours                     
                                        3600       ; retry after 1 hour                     
                                        604800     ; expire after 1 week                     
                                        86400 )    ; minimum TTL of 1 day  
                                NS      dhcp-ddns-1.lab.lan.
                                NS      dhcp-ddns-2.lab.lan.
dhcp-ddns-1.lab.lan.    A               192.168.10.5
dhcp-ddns-2.lab.lan.    A               192.168.10.6
```

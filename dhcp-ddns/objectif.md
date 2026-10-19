# DHCP & DNS en haute disponibilité avec Kea et BIND

## Projet

### Contexte

Dans le cadre de ce projet, j'ai mis en place une infrastructure réseau sous Linux associant un service DHCP à un service DNS dynamique (DDNS).

L'objectif est d'automatiser l'attribution des adresses IP aux clients et de maintenir à jour les enregistrements DNS correspondants.

Un serveur DHCP en panne = plus aucun équipement ne peut rejoindre le réseau. Pas de réseau, pas de prod

### Problematique

Comment automatiser l'attribution des adresses IP et la mise à jour des enregistrements DNS afin de simplifier l'administration d'un réseau ?

### Objectif

- Automatiser l'attribution des adresses IP via DHCP.
- Configurer un service DNS pour la résolution des noms de machines.
- Mettre en place les mises à jour DNS dynamiques.
- Vérifier la cohérence entre les adresses IP attribuées et les enregistrements DNS.
- Valider le fonctionnement de l'infrastructure à l'aide de tests.
- Reduction des SPOF

## Architecture et Environnement technique

### Environnement utilisé

- Hyperviseur : Proxmox
- Système d'exploitation : OPNsense, Debian
- Service DHCP : kea-dhcp
- Service DNS : Bind9
- Réseau :
    - LAN local : 192.168.0.0/24
    - LAN 10 : 192.168.10.0/24
    - LAN 20 : 192.168.20.0/24

![topologie](./illustration/topologie.png)

## Installation et configuration

### OPNsense

Un des premier éléments à mettre en place, est le DHCP relay pour permettre à mon LAN clients de communiquer avec les serveurs DHCP sur un autre LAN à part

![dhcp relay](./illustration/dhcp-relay.png)

### DHCP

Kea-dhcp enregistre touts les leases dans un fichier créer lors de l'installation, qui se situe dans : `/var/lib/kea/kea-leases4.csv`

Le fichier de configuration de kea-dhcp contient des instructions pour mettre à jour le fichier automatiquement

kea-dhcp4.conf



kea-ddns.conf

load-balance

### DNS

namd.conf.local

namd.conf.options

creation zone

## Test

serie de test

### DHCP

dhcp primary seul pour attribution ip

dhcp en ha pour replication fichier lease

dhcp 1 off pour voir si dhcp 2 attribue une ip

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

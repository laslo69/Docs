Mise en place d'une infrastructure DHCP avec DNS dynamique (DDNS)

Type de projet : Administration système et réseau
Environnement : Linux
Statut : Projet terminé
Auteur : [Ton nom]
Date : [Date de réalisation]

1. Présentation du projet

1.1 Contexte

Dans le cadre de ce projet, j'ai mis en place une infrastructure réseau sous Linux associant un service DHCP à un service DNS dynamique (DDNS).

L'objectif est d'automatiser l'attribution des adresses IP aux clients et de maintenir à jour les enregistrements DNS correspondants.

1.2 Problématique

Comment automatiser l'attribution des adresses IP et la mise à jour des enregistrements DNS afin de simplifier l'administration d'un réseau ?

1.3 Objectifs

Automatiser l'attribution des adresses IP via DHCP.

Configurer un service DNS pour la résolution des noms de machines.

Mettre en place les mises à jour DNS dynamiques.

Vérifier la cohérence entre les adresses IP attribuées et les enregistrements DNS.

Valider le fonctionnement de l'infrastructure à l'aide de tests.

2. Architecture et environnement technique

2.1 Environnement utilisé

Élément

Description

Système d'exploitation

[Distribution et version Linux]

Service DHCP

[ISC DHCP Server, Kea DHCP ou autre]

Service DNS

[BIND9 ou autre]

Hyperviseur

[VirtualBox, Proxmox, VMware ou autre]

Domaine DNS

[domaine de laboratoire]

Réseau local

[Adresse réseau et masque]

2.2 Plan d'adressage IP

Équipement

Adresse IP

Rôle

Serveur DHCP/DNS

[IP fixe]

Attribution IP et résolution DNS

Client Linux

[IP attribuée]

Client DHCP

Passerelle

[Adresse IP]

Accès au réseau

2.3 Architecture réseau

[Insérer un schéma de l'infrastructure.]

Décrire le rôle de chaque machine ainsi que les échanges entre le client DHCP, le serveur DHCP et le serveur DNS.

3. Installation et configuration

3.1 Configuration du serveur DHCP

Objectif : attribuer automatiquement une configuration réseau aux clients.

Installation

[Présenter les paquets installés et les commandes utilisées.]

Configuration

[Présenter les paramètres essentiels :]

Interface réseau utilisée.

Plage d'adresses IP distribuées.

Masque de sous-réseau.

Passerelle par défaut.

Serveur DNS et domaine distribués aux clients.

Paramètres nécessaires aux mises à jour DNS dynamiques.

Vérification du service

[Présenter la commande utilisée pour vérifier l'état du service et les éventuelles erreurs.]

3.2 Configuration du serveur DNS

Objectif : permettre la résolution des noms de machines en adresses IP et, si configuré, la résolution inverse.

Installation

[Présenter les paquets installés.]

Configuration des zones DNS

Zone directe : association entre un nom et une adresse IP.

Zone inverse : association entre une adresse IP et un nom d'hôte.

Paramètres de mise à jour dynamique.

Autorisations et mécanisme d'authentification, le cas échéant.

Vérification du service

[Présenter les contrôles de configuration et les vérifications de l'état du service.]

3.3 Configuration du DDNS

Objectif : automatiser la mise à jour des enregistrements DNS en fonction de la configuration réseau des clients.

Présenter :

Le mécanisme de communication entre DHCP et DNS.

La méthode de mise à jour dynamique utilisée.

La gestion des enregistrements A et PTR.

Les autorisations nécessaires pour modifier les zones DNS.

Le comportement attendu lors de l'attribution ou du renouvellement d'un bail DHCP.

[Ajouter les extraits de configuration pertinents.]

3.4 Interaction entre les services

Décrire le déroulement d'une attribution d'adresse IP et la manière dont les enregistrements DNS sont créés ou actualisés.

[Insérer un schéma de fonctionnement si nécessaire.]

4. Tests et validation

Pour chaque test, présenter l'objectif, la commande exécutée, le résultat attendu et le résultat observé.

4.1 Vérification de l'attribution DHCP

Objectif : vérifier que le client reçoit automatiquement une configuration réseau.

Commandes utilisées :

[Ajouter les commandes adaptées au système client.]

Résultat attendu : le client obtient une adresse IP valide ainsi que les paramètres réseau prévus.

Résultat obtenu : [Décrire le résultat observé.]

[Insérer une capture d'écran.]

4.2 Vérification de la résolution DNS directe

Objectif : vérifier qu'un nom d'hôte est associé à la bonne adresse IP.

Commande :

[Ajouter la commande de résolution DNS.]

Résultat attendu : le nom d'hôte retourne l'adresse IP attendue.

Résultat obtenu : [Décrire le résultat observé.]

[Insérer une capture d'écran.]

4.3 Vérification de la résolution DNS inverse

Objectif : vérifier qu'une adresse IP peut être associée au nom d'hôte correspondant.

Commande :

[Ajouter la commande de résolution inverse.]

Résultat attendu : l'adresse IP retourne le nom d'hôte attendu, si la zone inverse est configurée.

Résultat obtenu : [Décrire le résultat observé.]

[Insérer une capture d'écran.]

4.4 Vérification des mises à jour DDNS

Objectif : vérifier que les enregistrements DNS sont mis à jour automatiquement selon le mécanisme configuré.

Procédure :

Obtenir ou renouveler un bail DHCP sur le client.

Vérifier l'adresse IP attribuée.

Interroger le serveur DNS pour contrôler l'enregistrement direct.

Vérifier l'enregistrement inverse, si applicable.

Contrôler les journaux des services en cas d'anomalie.

Résultat attendu : les enregistrements concernés correspondent à la configuration réseau actuelle du client.

Résultat obtenu : [Décrire les résultats observés.]

[Insérer une capture d'écran.]

4.5 Synthèse des tests

Test

Résultat

Commentaire

Attribution DHCP

[Validé / À corriger]

[Observation]

Résolution directe

[Validé / À corriger]

[Observation]

Résolution inverse

[Validé / Non applicable / À corriger]

[Observation]

Mise à jour DDNS

[Validé / À corriger]

[Observation]

5. Difficultés rencontrées et solutions

5.1 Problème rencontré

[Décrire le symptôme observé.]

5.2 Diagnostic

[Présenter les commandes, journaux et vérifications utilisés.]

5.3 Solution appliquée

[Expliquer la correction apportée et son intérêt.]

5.4 Validation après correction

[Présenter le résultat du nouveau test.]

6. Bilan du projet

6.1 Résultats obtenus

[Résumer les fonctionnalités effectivement mises en place et validées.]

6.2 Compétences développées

Administration de services réseau sous Linux.

Configuration et diagnostic DHCP.

Gestion des zones DNS.

Mise en œuvre des mises à jour DNS dynamiques.

Utilisation de commandes de diagnostic et de consultation des journaux.

Documentation technique et validation d'une infrastructure.

6.3 Améliorations possibles

Renforcer la sécurisation des mises à jour DNS.

Mettre en place une supervision des services.

Centraliser les journaux.

Étudier la redondance des services.

Automatiser les contrôles de fonctionnement.

7. Conclusion

Ce projet m'a permis de mettre en pratique des compétences en administration système et réseau sous Linux, en associant l'attribution automatique des adresses IP à la résolution DNS dynamique.

Les tests réalisés permettent de vérifier les fonctionnalités configurées et d'identifier les éventuels points d'amélioration.

8. Annexes

Extraits de configuration DHCP.

Extraits de configuration DNS.

Commandes de diagnostic utilisées.

Captures d'écran des tests.

Schéma de l'architecture réseau.
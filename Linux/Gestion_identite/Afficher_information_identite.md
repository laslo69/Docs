
Il existe plusieurs commandes pour lister les utilisateurs, groupes et, groupes auquels appartient un utilisateur

## Lister les utilisateurs

`cat /etc/password` permet de lister tout les utilisateurs local, si un mot de passe est défini, l'id de groupe principal, le shell utilisé, etc...

`getent` donne le même résultat mais peut aussi, lister les utilisateur d'un domaine active directory, d'un serveur LDAP et d'un serveur NIS

```bash
root@debian:~# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon:/usr/lib/dhcpcd:/bin/false
systemd-timesync:x:991:991:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:990:990:System Message Bus:/nonexistent:/usr/sbin/nologin
alex:x:1000:1000:alex,,,:/home/alex:/bin/bash
sshd:x:989:65534:sshd user:/run/sshd:/usr/sbin/nologin
```

D'ailleurs, il est possible de voir, si un mot de passe enregistré pour un utilisateur existe ou non, et si il est crypté ou non, en lisant le fichier `/etc/shadow`

```bash
root@debian:~# cat /etc/shadow
root:$y$j9T$g2SftyVGMiUzqackCTqKX/$uFArX1GFNBwXPxfR7Dez/N0dVaDLDM4H5odHSWL3QB/:20705:0:99999:7:::
daemon:*:20705:0:99999:7:::
bin:*:20705:0:99999:7:::
sys:*:20705:0:99999:7:::
sync:*:20705:0:99999:7:::
games:*:20705:0:99999:7:::
man:*:20705:0:99999:7:::
lp:*:20705:0:99999:7:::
mail:*:20705:0:99999:7:::
news:*:20705:0:99999:7:::
uucp:*:20705:0:99999:7:::
proxy:*:20705:0:99999:7:::
www-data:*:20705:0:99999:7:::
backup:*:20705:0:99999:7:::
list:*:20705:0:99999:7:::
irc:*:20705:0:99999:7:::
_apt:*:20705:0:99999:7:::
nobody:*:20705:0:99999:7:::
systemd-network:!*:20705:::::1:
dhcpcd:!:20705::::::
systemd-timesync:!*:20705:::::1:
messagebus:!*:20705::::::
alex:$y$j9T$/wIZ.9wS4JV5LZOXnKxkq.$bvC83/bwkBZIYEiuc31.qkBqXfCkdnNLbuJbxppyKS4:20705:0:99999:7:::
sshd:!*:20705::::::
root@debian:~# 
```

L'utilisateur `root` à un mot de passe crypté via sha-256/sha-512

Les utilisateurs avec un mot de passe `!` sont actif mais on ne peut pas se connecter dessus

## Lister les groupes

Pour lister les groupes, lire le fichier `/etc/group`

```bash
root@debian:~# cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:alex
floppy:x:25:alex
tape:x:26:
sudo:x:27:
audio:x:29:alex
dip:x:30:alex
www-data:x:33:
backup:x:34:
operator:x:37:
list:x:38:
irc:x:39:
src:x:40:
shadow:x:42:
utmp:x:43:
video:x:44:alex
sasl:x:45:
plugdev:x:46:alex
staff:x:50:
games:x:60:
users:x:100:alex
nogroup:x:65534:
systemd-journal:x:999:
systemd-network:x:998:
crontab:x:997:
input:x:996:
sgx:x:995:
clock:x:994:
kvm:x:993:
render:x:992:
netdev:x:101:alex
systemd-timesync:x:991:
messagebus:x:990:
_ssh:x:102:
alex:x:1000:
```

`getent` marche aussi avec les groupes

## Voir les groupes d'un utilisateur spécifique

Pour savoir à quels groupes appartient un utilisateur donné (par exemple `alex`), vous pouvez utiliser

 `groups alex` : Affiche simplement la liste des noms de groupes.

```bash
groups alex 
alex : alex cdrom floppy audio dip video plugdev users netdev
```

`id alex` : Plus complet, il affiche l'identifiant de l'utilisateur (UID), son groupe principal (GID) et tous ses groupes secondaires.

```bash
uid=1000(alex) gid=1000(alex) groupes=1000(alex),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),101(netdev)
```

`groups` ou `id` (sans nom) : Affiche les informations de l'utilisateur actuellement connecté

```bash
root@debian:~# groups 
root
root@debian:~# id
uid=0(root) gid=0(root) groupes=0(root)
root@debian:~# 
```
## Voir les membres d'un groupe spécifique

Après avoir fait la commande `chmod -aG test alex`

```bash
root@debian:~# getent group alex 
alex:x:1000:test
```

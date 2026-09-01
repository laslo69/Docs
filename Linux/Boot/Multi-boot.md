
GRUB peut être utilisé pour charger des systèmes d’exploitation non pris en charge, comme Windows, en utilisant un processus appelé chainloading  (multi-amorçage)

GRUB est chargé en premier, et lorsque l’option correspondante est sélectionnée, le chargeur de démarrage du système en question est chargé

Si aucun autre système n'est détecté, il faut alors installer le paquet `os-prober` et l'ajouter dans la configuration de `/etc/default/grub`

```bash
GRUB_DISABLE_OS_PROBER=false
```

Une entrée typique pour un double boot Windows ressemblerait à

```bash
title Windows XP
root (hd0,1)
makeactive
chainload +1
boot
```

Comme précédemment, `root (hd0,1)` spécifie le périphérique et la partition où se trouve le chargeur de démarrage du système d’exploitation que nous souhaitons charger. Dans cet exemple, la _seconde_ partition du premier disque

- `makeactive` : définit un fanion indiquant qu’il s’agit d’une partition active. Cela ne fonctionne que sur les partitions primaires DOS.
- `chainload +1` : indique à GRUB de charger le premier secteur de la partition de démarrage. C’est là que se trouvent généralement les chargeurs de démarrage.
- `boot` : va exécuter le chargeur de démarrage et charger le système d’exploitation correspondant.


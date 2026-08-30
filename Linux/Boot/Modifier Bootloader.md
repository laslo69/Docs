
Un nouveau fichier de configuration pour le chargeur d’amorçage doit être généré à chaque fois que `/etc/default/grub` change, ce qui est effectué par la commande `grub-mkconfig -o /boot/grub/grub.cfg`. 

Les paramètres du noyau doivent être ajoutés au fichier `/etc/default/grub` à la ligne `GRUB_CMDLINE_LINUX` pour les rendre persistants après chaque redémarrage

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash maxcpus=4"
```

Sur debian/ubuntu/mint :

```bash
update-grub
ou
grub-mkconfig -o /boot/grub/grub.cfg
```

Sur fedora/Alma,Rocky : 

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
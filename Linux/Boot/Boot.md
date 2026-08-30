## Boot BIOS

1) Test POST
2) BIOS active les composants basiques ( clavier, sortie vidéo, stockage )
3) Bios lit MBR (1ere phase du bootloader, 440 premier octets )
4) Bios lit la deuxième parti du bootloader ( chargement options démarrage, kernel )
5) Initialisation systeme : Ie kernel prend en charge le CPU, détecte et configure le systeme
6) Le kernel ouvre un disque memoire initial ( initramfs), qui contient un systeme de fichiers utilisé en fichiers racines temporaire lors du demarrage. Fournis les modules pour acceder au vrai systeme de fichiers racine
7) Des que le systeme de fichiers racine est disponible, le kernel monte tous les systemes de fichiers dans /etc/fstab et execute le premier programme ( utilitaire init, pid = 1)

## Boot UEFI

1) Test POST
2) UEFI active les composants basiques ( clavier, sortie vidéo, stockage )
3) UEFI lit la partition ESP ( généralement un chargeur de démarrage ) et contrôle sa validité
4) UEFI lit le EFI bootloader dans la partition ESP qui charge le kernel
5) Initialisation systeme : Ie kernel prend en charge le CPU, détecte et configure le systemeaspects fondamentaux du systeme
6) Le kernel ouvre un disque memoire initial ( initramfs), qui contient un systeme de fichiers utilisé en fichiers racines temporaire lors du demarrage. Fournis les modules pour acceder au vrai systeme de fichiers racine
7) Des que le systeme de fichiers racine est disponible, le kernel monte tous les systemes de fichiers dans /etc/fstab et execute le premier programme ( utilitaire init, pid = 1)

UEFI ne prend en compte que les paramètres stockés dans la NVRAM attaché à la carte mère

Ces définitions indiquent l’emplacement des programmes compatibles UEFI, appelés _applications EFI_, qui seront exécutés automatiquement ou invoqués à partir d’un menu de démarrage

Les applications EFI peuvent être des chargeurs d’amorçage, des sélecteurs de systèmes d’exploitation, des outils de diagnostic et de réparation système, etc. Ils doivent figurer dans une partition classique de périphérique de stockage et dans un système de fichiers compatible ( généralement FAT 12,16,32 )
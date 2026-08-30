Les commandes `lspci`, `lsusb` et `lsmod` agissent comme des frontaux pour lire les informations relatives au matériel stockées par le système d’exploitation. Ce type d’information est conservé dans des fichiers spéciaux dans les répertoires `/proc` et `/sys`

Ces répertoires sont des points de montage vers des systèmes de fichiers non présents dans une partition de périphérique, mais uniquement dans l’espace RAM utilisé par le noyau pour stocker la configuration d’exécution et les informations sur les processus en cours

Ces systèmes de fichiers ne sont pas destinés au stockage conventionnel de fichiers, ils sont donc appelés pseudo-systèmes de fichiers et n’existent que lorsque le système est en cours d’exécution

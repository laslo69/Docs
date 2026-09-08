
Lorsque vous ajoutez un nouveau compte utilisateur, même en créant son répertoire personnel, ce dernier est peuplé de fichiers et de dossiers copiés depuis le répertoire `/etc/skel`

L’idée derrière ceci est simple : un administrateur système veut ajouter de nouveaux utilisateurs avec les mêmes fichiers et répertoires dans leur répertoire personnel. Par conséquent, si vous voulez personnaliser les fichiers et les dossiers qui sont créés automatiquement dans le répertoire personnel des nouveaux comptes utilisateurs, vous devrez ajouter ces nouveaux fichiers et dossiers au répertoire squelette

```bash
ls /etc/skel

```

En ajoutant des fichiers, dossiers dans ce répertoie, chaque utilisateur créer aura automatiquement une copie de ces fichiers si, lors de la création les paramètres `-m` et `-k /etc/skel` sont transmis
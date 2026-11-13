# snayzOS

![Statut](https://img.shields.io/badge/statut-work%20in%20progress-orange)
![Version](https://img.shields.io/badge/version-v0.1-blue)
![Usage](https://img.shields.io/badge/usage-exp%C3%A9rimental-red)
![Architecture](https://img.shields.io/badge/architecture-x86%20(i386)-lightgrey)
![Langages](https://img.shields.io/badge/langages-C%20%7C%20NASM-informational)
![Boot](https://img.shields.io/badge/boot-GRUB%20Multiboot-green)
![Build](https://img.shields.io/badge/build-Windows%20%2B%20WSL-0078D6)
![Test](https://img.shields.io/badge/test-QEMU-ff6600)
![Licence](https://img.shields.io/badge/licence-non%20d%C3%A9finie-lightgrey)

**snayzOS** est un petit système d'exploitation expérimental développé
en C et en assembleur x86. Le projet fournit un noyau freestanding 32
bits (`i386`) qui démarre avec GRUB via Multiboot et affiche un
environnement graphique simple, un terminal intégré et quelques
informations matérielles.

> **État du projet :** prototype en cours de développement (v0.1).
> Plusieurs sous-systèmes sont encore des squelettes ; snayzOS n'est pas
> destiné à une utilisation quotidienne ni à être installé sur une
> machine réelle.

## Fonctionnalités

-   Démarrage du noyau 32 bits avec un en-tête Multiboot.
-   Bureau rudimentaire avec accès à quatre écrans :
    -   **Terminal** : interpréteur de commandes minimal.
    -   **Fichiers** : détection d'un disque ATA et affichage de son
        modèle et de sa capacité.
    -   **Paramètres** : informations sur le clavier et le mode
        d'affichage.
    -   **Informations** : version et caractéristiques du noyau.
-   Affichage via le framebuffer 32 bits fourni au démarrage, avec repli
    sur le mode texte VGA 80×25.
-   Lecture du clavier PS/2 par interrogation des ports d'E/S, avec une
    correspondance AZERTY de base.
-   Pilote ATA PIO : identification d'un disque et lecture d'un secteur
    en LBA28.
-   Image ISO amorçable par GRUB et image disque QCOW2 pour les essais
    dans QEMU.

## Commandes du terminal

Depuis l'écran d'accueil, appuyer sur `1` pour ouvrir le terminal.

  -----------------------------------------------------------------------
  Commande                            Description
  ----------------------------------- -----------------------------------
  `help`                              Affiche la liste des commandes.

  `clear`                             Réinitialise l'écran du terminal.

  `echo <texte>`                      Affiche le texte fourni.

  `info`                              Affiche des informations de base
                                      sur le noyau.

  `color <hex>`                       Modifie l'attribut de couleur du
                                      terminal à partir d'une valeur
                                      hexadécimale.

  `desktop`                           Retourne au bureau.

  `reboot`                            Arrête le processeur dans une
                                      boucle d'attente (ce n'est pas
                                      encore un redémarrage matériel).
  -----------------------------------------------------------------------

Depuis le bureau, les touches `1` à `4` ouvrent les écrans
correspondants. La touche `Échap` revient au bureau.

## Prérequis de compilation

Le `makefile` fourni cible actuellement un environnement Windows avec
des chemins d'outils configurés en dur. Il s'appuie notamment sur :

-   `i686-elf-gcc` et `i686-elf-ld` (toolchain croisée 32 bits) ;
-   NASM ;
-   QEMU (`qemu-system-i386`) et `qemu-img` ;
-   WSL avec `grub-mkrescue` et `cp` pour fabriquer l'ISO.

Les chemins définis au début du `makefile` doivent être adaptés à votre
installation. Les règles de création de répertoires et de nettoyage
utilisent également des commandes Windows (`mkdir`, `rmdir`, `del`) ; le
fichier n'est donc pas directement portable vers un shell Linux sans
adaptation.

## Compilation et exécution

Depuis le répertoire `snayzOS/` :

``` sh
make
```

La cible par défaut construit le noyau, prépare l'image ISO et crée
l'image disque virtuelle si elle n'existe pas.

Pour lancer l'ISO dans QEMU avec un disque IDE virtuel :

``` sh
make run
```

Pour lancer directement le noyau avec QEMU (sans ISO) :

``` sh
make run-direct
```

Cette dernière cible utilise `-display none -serial stdio` ; l'interface
graphique du noyau n'est donc pas visible dans cette configuration.

Pour supprimer les fichiers de compilation et l'ISO :

``` sh
make clean
```

> Selon votre environnement, adaptez les variables `LD`, `CC`, `NASM`,
> `QEMU` et `QEMU_IMG` dans le `makefile`. La fabrication de l'ISO
> dépend de WSL et de `grub-mkrescue`.

## Structure du dépôt

``` text
snayzOS/
├── src/
│   ├── c/
│   │   └── kmain.c          # Noyau, affichage, bureau, clavier et terminal
│   ├── drivers/
│   │   └── disk.c           # Pilote disque ATA PIO
│   ├── fs/
│   │   └── vfs.c            # Squelette du système de fichiers virtuel
│   ├── mem/
│   │   └── mem.c            # Squelette de gestion mémoire
│   └── nasm/
│       └── boot.s           # Point d’entrée assembleur et en-tête Multiboot
├── head/
│   ├── c/                   # En-têtes du noyau et police bitmap
│   ├── drivers/             # En-tête du pilote disque
│   ├── fs/                  # En-tête du VFS
│   └── mem/                 # En-tête mémoire
├── linker/
│   └── linkerscript.ld      # Script d’édition de liens
├── isodir/
│   └── boot/grub/grub.cfg   # Configuration de démarrage GRUB
├── doc/                     # Documents et licence de police
├── bin/                     # Objets et noyau compilé (générés)
├── makefile
├── snayzOS.iso              # Image ISO (générée)
└── snayzOS-disk.qcow2       # Disque virtuel (généré)
```

## Architecture technique

-   **Architecture cible :** x86 32 bits (`i386`).
-   **Langages :** C freestanding et assembleur NASM.
-   **Chargement :** GRUB / Multiboot.
-   **Édition de liens :** script dédié, chargement à partir de
    l'adresse `1 MiB`.
-   **Vidéo :** framebuffer Multiboot en 32 bits si disponible et
    compatible ; sinon VGA texte.
-   **Entrées :** clavier PS/2, lecture par polling.
-   **Stockage :** contrôleur ATA PIO, commandes IDENTIFY et READ
    SECTORS en LBA28.

## Limites actuelles

-   Le VFS (`src/fs/vfs.c`) et la gestion mémoire (`src/mem/mem.c`) ne
    sont pas encore implémentés.
-   L'écran Fichiers détecte le disque et permet la lecture de secteurs
    au niveau du pilote, mais aucun système de fichiers n'est monté et
    aucune écriture disque n'est proposée.
-   Il n'y a pas encore de gestionnaire d'interruptions clavier : les
    entrées sont lues en polling.
-   La prise en charge du clavier AZERTY est basique.
-   Les paramètres ne sont pas persistants.
-   La commande `reboot` ne réalise pas un redémarrage matériel.
-   Le noyau ne fournit pas encore de mécanisme d'installation sur
    disque dur.

## Pistes d'évolution

1.  Mettre en place une GDT et une IDT, puis gérer les interruptions
    matérielles.
2.  Implémenter un allocateur physique et un gestionnaire de mémoire
    virtuelle.
3.  Développer le VFS et un premier système de fichiers.
4.  Étendre le pilote disque et ajouter des tests de lecture robustes.
5.  Améliorer la gestion du clavier, du curseur et de l'interface.
6.  Ajouter des tests automatisés et documenter les conventions de
    compilation.

## Licence

Consulter les fichiers de licence présents dans `doc/`, notamment
`doc/DejaVu-font-license.txt` pour la police embarquée. Aucune licence
globale du projet n'est indiquée dans les fichiers examinés ; vérifier
les droits avant toute redistribution.

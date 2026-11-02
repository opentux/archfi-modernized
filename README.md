# archfi & archdi (Modernized Fork)

> **Arch Linux Fast Install & Desktop Install — Unofficial modernized rework**  
> *Forked from the original scripts created by [MatMoul](https://github.com/MatMoul).*

---

## ⚠️ Context & Disclaimer

Soyons honnêtes et clairs : ce projet est avant tout un **délire personnel**, un terrain de jeu et un exercice de nostalgie/modernisation. 

- **Archinstall existe désormais** : Depuis la création d'origine d'archfi/archdi, l'ISO officielle d'Arch Linux intègre son propre installateur guidé (`archinstall`).
- **Projet d'origine archivé** : Le créateur d'origine a cessé le développement il y a plusieurs années.
- **Aucune garantie dans le temps** : Ce fork est une réadaptation selon mes envies et besoins du moment. Le projet avance au gré du temps libre et de la motivation, sans promesse de suivi à long terme ni support officiel.

---

## 🎯 Ce qui a été fait (État actuel)

Le projet regroupe et modernise les deux scripts historiques dans une même arborescence unifiée (monorepo) :

### 1. `archfi` (Installation du système de base)
- **Monorepo & Détection locale** : `archdi` est rapatrié dans le même dossier et détecté automatiquement via `scriptdir`.
- **Prise en charge du matériel moderne** : Détection et nommage correct des partitions NVMe, eMMC et périphériques loop (`nvme0n1pX`).
- **Partitionnement moderne & Swap en RAM (zram)** :
  - Partition EFI / boot dimensionnée à **2 Go** par défaut (indispensable aujourd'hui pour accueillir plusieurs noyaux, initramfs de secours et UKIs).
  - Question interactive demandant si l'utilisateur souhaite créer ou non une partition swap sur le disque (afin de privilégier zram / swap en RAM). En l'absence de swap, la partition racine (`/`) occupe l'intégralité restante du disque.
- **Menu de formatage clarifié** : Séparation stricte entre le formatage de la partition BOOT/EFI (obligatoirement en FAT32 sous UEFI) et de la partition ROOT (accès direct à Btrfs, Ext4, XFS, F2FS avec descriptions claires), avec arrêt propre en cas d'annulation.
- **Options Btrfs optimisées** : Montage par défaut moderne (`rw,noatime,commit=120,space_cache=v2,compress=zstd:1`) et support revu des sous-volumes (`@`, `@home`, `@snapshots`, etc.).
- **Pacman QoL** : Activation de `ParallelDownloads`, de la couleur et du `ILoveCandy`.
- **Réseau & Wi-Fi clé en main** : Intégration et activation automatique de `NetworkManager.service` et `iwd` dès l'installation de base et lors du déploiement des bureaux, garantissant le bon fonctionnement immédiat de l'applet réseau et du Wi-Fi.
- **Noyaux, En-têtes & DKMS** : Choix du noyau avec installation intégrée des headers correspondants (`linux-headers`, `linux-lts-headers`, etc.) et de `dkms` dès l'étape pacstrap. Support du noyau ultra-performant **`linux-cachyos`** avec activation sécurisée des dépôts CachyOS (mode sans surcharge RAM en live boot, détection CPU v3/v4).
- **Optimisations CachyOS & Tweaks Système** :
  - Configuration interactive des règles I/O Schedulers udev (scheduler **`adios`** pour SSD/NVMe et `bfq` pour HDD).
  - Tweaks sysctl (`kernel.split_lock_mitigate=0`, `vm.max_map_count = 2147483642`).
  - Optimisation avancée de `mkinitcpio.conf` (modules NVMe + NVIDIA après installation des pilotes, compression ultra-rapide `lz4`, Plymouth avec thème `spinner`).
- **Création d'utilisateur & Sudoers propre** : Menu dédié à la création d'un compte utilisateur standard directement depuis `archfi`, avec ajout au groupe `wheel` et option pour créer un fichier nominatif sans mot de passe (`NOPASSWD`) dans `/etc/sudoers.d/<nomduuser>` (droits 0440).
- **Internationalisation & Français intégral (Console, X11, Wayland, Plasma & Apps)** :
  - Configuration complète de la locale système (`fr_FR.UTF-8 UTF-8` dans `locale.gen` et `locale.conf`).
  - Configuration native de **KDE Plasma 6** en Français via `/etc/xdg/plasma-localerc` (`LANGUAGE=fr_FR:fr`).
  - Installation automatique des packs de langue française pour les applications (`firefox-i18n-fr`, `libreoffice-fresh-fr`, `hunspell-fr`).
 
- **Traductions épurées** : Maintenance ciblée uniquement sur le Français et l'Anglais (suppression des fichiers de langues orphelins).

### 2. `archdi` (Post-installation & Environnement de bureau)
Un gros travail d'actualisation a déjà été effectué pour adapter le script aux standards actuels :
- **Clavier AZERTY natif dans les Display Managers & Bureaux** :
  - Détection automatique et configuration globale du clavier pour tous les gestionnaires de session (SDDM, GDM, LightDM, Plasma Login Manager) évitant de se retrouver en QWERTY à l'écran de connexion.
  - Configuration native de **KDE Plasma 6** (`kxkbrc` système et utilisateur), GNOME, XFCE, Cinnamon, MATE et Hyprland pour un clavier français fonctionnel dès le premier démarrage.
- **Listing des paquets dépoussiéré** : Nettoyage en profondeur des paquets abandonnés ou renommés dans les dépôts officiels.
- **Gestionnaire AUR `yay` & Déploiement sans blocage** :
  - Déploiement direct de **`yay`** (`yay-bin`) sans sous-menu superflu.
  - Résolution des blocages de mot de passe sudo lors de la compilation AUR en chroot (gestion `NOPASSWD` nominative ou temporaire).
  - Déploiement automatique de `yay` en amont si des paquets AUR (comme TUXEDO) sont sélectionnés.
- **Support PC Portables TUXEDO & Gestion d'énergie** :
  - Intégration du pack complet TUXEDO : `tuxedo-drivers-dkms` et `tuxedo-control-center-bin` (AUR via `yay`).
  - Détection automatique du/des noyau(x) installé(s) et installation transparente des headers et de `dkms`.
  - Résolution des conflits d'énergie : arrêt et masquage automatique (`systemctl mask`) des services concurrents (`power-profiles-daemon`, `tuned`, `tuned-ppd`, `tlp`, `auto-cpufreq`) pour confier la gestion exclusive à `tccd.service` (TUXEDO Control Center), avec option de restauration.

- **Pack CachyOS, Kernel, Schedulers & Optimisations Système** :
  - **Dépôt officiel CachyOS** : déploiement automatique avec détection d'architecture CPU (x86-64-v3, x86-64-v4, standard) et clés de confiance GPG.
  - **Noyau CachyOS & Settings** : installation de `linux-cachyos`, `linux-cachyos-headers` et `cachyos-settings`.
  - **Scheduler I/O `adios`** : règle udev dédiée `/etc/udev/rules.d/60-ioschedulers.rules` appliquant le scheduler `adios` pour SSD et NVMe, et `bfq` pour disques HDD.
  - **Tweaks Sysctl** : fichier `/etc/sysctl.d/99-splitlock.conf` (`kernel.split_lock_mitigate=0` anti-stutter et `vm.max_map_count = 2147483642` pour les jeux Steam/Proton/Unreal Engine).
  - **mkinitcpio & Boot Splash Plymouth** :
    - Injection automatique de `nvme` et des modules NVIDIA propriétaires/open (`nvidia`, `nvidia_modeset`, `nvidia_uvm`, `nvidia_drm`).
    - Intégration de Plymouth avec le thème moderne **`spinner`**.
    - Hooks modernes (`systemd autodetect microcode modconf kms block filesystems plymouth`).
    - Compression haute vitesse `lz4` (`COMPRESSION="lz4"`, `COMPRESSION_OPTIONS=(-1)`, `MODULES_DECOMPRESS="yes"`).
  - Possibilité de lancer le pack complet tout-en-un ou de choisir chaque composant individuellement.

- **Environnements de bureau modernes** : Prise en charge de **KDE Plasma 6** (avec Wayland et session moderne) et **GNOME** (générations récentes / GNOME 5x).
- **Dépôts Multilib & Tweaks Pacman** : Menu dédié pour activer d'un clic le dépôt `[multilib]` (essentiel pour le jeu/Steam et la compatibilité 32-bit), avec 10 téléchargements parallèles et `ILoveCandy`.
- **Pilotes GPU récents & Graphiques Hybrides (Optimus / Switch MUX)** :
  - Profils dédiés pour PC portables hybrides : **Intel + NVIDIA Open** et **AMD + NVIDIA Open**, configurant la stack graphique complète (`intel-media-driver`, `vulkan-intel`, `mesa` 32/64 bits) et l'outil de bascule PRIME (`nvidia-prime` / `prime-run`) pour exploiter le switch MUX ou l'Advanced Optimus.
  - **Détection dynamique des noyaux pour NVIDIA Open** : si un noyau alternatif est détecté (`linux-zen`, `linux-lts`, `linux-hardened`), le script installe automatiquement **`nvidia-open-dkms`** avec ses en-têtes associés, empêchant `pacman` de forcer l'installation du noyau `linux` standard.
  - Pilotes AMD Radeon, Intel Graphics et NVIDIA Propriétaire (`nvidia-dkms`) disponibles pour toutes configurations.
- **Audio moderne** : Intégration de la stack moderne **PipeWire** (avec `pipewire-pulse`, `wireplumber`) en remplacement du vieux PulseAudio.
- **Boîte à outils & Applications Populaires** : Sélection interactive enrichie :
  - **Navigateurs Web** : Firefox (avec pack de langue français automatique `firefox-i18n-fr`), Google Chrome (dépôt CachyOS / AUR), Vivaldi (avec codecs multimédia), Chromium.
  - **Gestionnaire de mots de passe** : Bitwarden.
  - **Multimédia** : VLC, SMPlayer (avec thèmes et skins), MPV.
  - **Monitoring & Outils CLI** : `btop`, `htop`, `fastfetch` (intégré et configuré dans `.bashrc` pour s'exécuter automatiquement à l'ouverture de chaque terminal interactif), `nvtop` (monitoring GPU en temps réel), `inxi`, `nmon`.
  - **Archives** : `p7zip`, `unrar`.

---

## 🚧 En cours & Pistes pour l'avenir (WIP & Roadmap)

Même si une bonne partie du décrassage est faite, il reste du pain sur la planche et plusieurs pistes de réflexion :

### Ce qui reste à finaliser
- **Peaufinage de `archdi`** : Finaliser les sous-menus restants pour l'amener là où je le souhaite (gestionnaires de connexion, profils audio/Bluetooth fins, suites logicielles).
- **Tests en conditions réelles** : Valider les différents scénarios d'installation (UEFI / BIOS, Btrfs vs Ext4, différents couples DE + Display Manager).
- **Nettoyage du code shell** : Poursuivre la simplification et l'unification du code entre les deux scripts.

### Idées & réflexions futures
- **Pack "Gaming" clé en main** :
  - Option d'installation regroupant l'écosystème gaming standard (Steam, GameMode, MangoHud, dépendances Vulkan/Wine, Proton-GE, etc.).
- **Virtualisation / Hyperviseurs Type 2** :
  - Menu dédié pour configurer facilement un hyperviseur de bureau au choix :
    - **QEMU / KVM** complet avec `virt-manager`, `libvirt`, `dnsmasq` et ponts réseau.
    - **VMware Workstation** (modules noyau, services systemd associés).

---

## 🚀 Utilisation (Quickstart)

### 1. Démarrer sur l'ISO Arch Linux
Démarrez votre machine avec une image d'installation récente d'Arch Linux et connectez-vous au réseau (ex: `iwctl` en Wi-Fi).

### 3. Installer 'git'
téléchargez et installer le package manquant avec pacman :

```bash
pacman -Sy git
```

### 3. Récupérer et lancer `archfi`
Clonez ou téléchargez les scripts dans votre environnement live :

```bash
git clone https://github.com/opentux/archfi-modernized.git
cd archfi-modernized
chmod +x archfi archdi
./archfi
```

Suivez les menus guidés pour partitionner votre disque, installer la base système et le chargeur d'amorçage.

### 3. Lancer `archdi` (si executé seul en post-installation)
Une fois le système de base en place (après avoir redémarré sur votre nouveau système) :

```bash
git clone https://github.com/opentux/archfi-modernized.git
cd archfi-modernized
chmod +x archdi
./archdi
```

Vous pourrez alors activer Multilib, installer vos pilotes GPU (`nvidia-open-dkms`, AMD...), déployer Plasma 6 ou GNOME, et sélectionner vos outils favoris (`nvtop`, `inxi`, etc.).

---

## 📜 Licence & Crédits

- Projet original conçu par [MatMoul](https://github.com/MatMoul) (archfi / archdi).
- Distribué sous licence **GPLv3**.

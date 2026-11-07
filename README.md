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
- **Partitionnement moderne & Swap en RAM (ZRAM)** :
  - Partition EFI / boot dimensionnée à **2 Go** par défaut (indispensable aujourd'hui pour accueillir plusieurs noyaux, initramfs de secours et UKIs).
  - Question interactive demandant si l'utilisateur souhaite créer ou non une partition swap sur le disque (afin de privilégier zram / swap en RAM). En l'absence de swap, la partition racine (`/`) occupe l'intégralité restante du disque.
  - **Activation automatique de ZRAM (`zram-generator`)** : pré-coché dans `pacstrap` dès qu'aucun swap disque n'est créé, avec configuration automatique en compression `zstd`, taille 100% RAM, priorité 100 et `vm.swappiness=180`.
- **Menu de formatage clarifié & Mode Lab Btrfs (`nodatacow,nodatasum`)** :
  - Séparation stricte entre le formatage de la partition BOOT/EFI (obligatoirement en FAT32 sous UEFI) et de la partition ROOT (accès direct à Btrfs, Ext4, XFS, F2FS avec descriptions claires), avec arrêt propre en cas d'annulation.
  - **Options Btrfs sur mesure (Standard vs Lab VM/Perf)** : choix entre `compress=zstd:1` et/ou `nodatacow` + `nodatasum` sur une base optimisée (`rw,noatime,ssd,commit=120,space_cache=v2`).
  - **Sous-volumes Btrfs avec ou sans Snapshots** : lorsque `nodatacow` est sélectionné pour un usage Lab, le mode **Sous-volumes sans snapshots** (`@`, `@home`, `@var_log`, `@pkg` — sans `@snapshots`) est automatiquement mis en tête et pré-sélectionné, tout en conservant le mode complet avec `@snapshots` ou le mode classique sans sous-volumes.
- **Pacman QoL & Sécurité `pacstrap`** : Activation de `ParallelDownloads`, de la couleur et du `ILoveCandy`, avec vérification stricte du code retour de `pacstrap` pour éviter toute poursuite sur un `/mnt` incomplet.
- **Réseau & Wi-Fi clé en main** : Intégration et activation automatique de `NetworkManager.service` et `iwd` dès l'installation de base et lors du déploiement des bureaux, garantissant le bon fonctionnement immédiat de l'applet réseau et du Wi-Fi.
- **Noyaux, En-têtes, DKMS & Mirrorlists CachyOS dédiés** : Choix du noyau avec installation intégrée des headers correspondants (`linux-headers`, `linux-lts-headers`, etc.) et de `dkms` dès l'étape pacstrap. Support du noyau ultra-performant **`linux-cachyos`** avec fichiers mirrorlists dédiés par architecture (`cachyos-v3-mirrorlist` sur `/repo/x86_64_v3/$repo`, `cachyos-v4-mirrorlist` sur `/repo/x86_64_v4/$repo` et `cachyos-mirrorlist`) et `SigLevel = PackageRequired`.
- **Optimisations CachyOS, Bootloaders Express & Kernel Cmdline automatique** :
  - Configuration interactive des règles I/O Schedulers udev (scheduler **`adios`** pour SSD/NVMe et `bfq` pour HDD).
  - Tweaks sysctl (`kernel.split_lock_mitigate=0`, `vm.max_map_count = 2147483642`).
  - Optimisation avancée de `mkinitcpio.conf` (modules NVMe + NVIDIA uniquement si déjà installés pour éviter les erreurs en `pacstrap`, compression ultra-rapide `lz4`, Plymouth avec thème `spinner`).
  - **Bootloader GRUB Express (1-clic)** et injection automatique des paramètres noyau `quiet splash` (Plymouth), `nvidia-drm.modeset=1 nvidia-drm.fbdev=1` (Wayland NVIDIA) et `rd.luks.name=...` (compatibilité double `sd-encrypt` systemd & `encrypt` standard) dans GRUB, `systemd-boot` et `rEFInd`.
- **Création d'utilisateur, Sudoers propre & Nettoyage automatique** : Menu dédié à la création d'un compte utilisateur standard directement depuis `archfi`, avec ajout au groupe `wheel`, option `NOPASSWD` dans `/etc/sudoers.d/<nomduuser>` (droits 0440), et suppression automatique des fichiers temporaires d'installation (`/root/archfi`, `/root/lng`, `99-installer-temp`) au démontage.
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
  - **Dépôt officiel CachyOS** : déploiement automatique avec détection d'architecture CPU (`x86_64_v3`, `x86_64_v4`, standard), mirrorlists dédiés (`cachyos-v3-mirrorlist`, `cachyos-v4-mirrorlist`) et clés de confiance GPG.
  - **Noyau CachyOS & Settings** : installation de `linux-cachyos`, `linux-cachyos-headers` et `cachyos-settings`.
  - **Scheduler I/O `adios`** : règle udev dédiée `/etc/udev/rules.d/60-ioschedulers.rules` appliquant le scheduler `adios` pour SSD et NVMe, et `bfq` pour disques HDD.
  - **Tweaks Sysctl** : fichier `/etc/sysctl.d/99-splitlock.conf` (`kernel.split_lock_mitigate=0` anti-stutter et `vm.max_map_count = 2147483642` pour les jeux Steam/Proton/Unreal Engine).
  - **ZRAM Swap, Ananicy-cpp, Sched-ext (`scx_lavd`) & Rate Mirrors** :
    - Configuration de `zram-generator` (`zstd`, 100% RAM, `vm.swappiness=180`).
    - Installation de **`ananicy-cpp`** + **`cachyos-ananicy-rules`** (démon de priorisation automatique CPU/I/O).
    - Support optionnel des ordonnanceurs CPU BPF **`scx-scheds`** (profil gaming/laptop **`scx_lavd`**).
    - Tri intelligent des miroirs Arch & CachyOS via **`cachyos-rate-mirrors`** (ou fallback `reflector`).
  - **mkinitcpio & Boot Splash Plymouth** :
    - Injection automatique de `nvme` et des modules NVIDIA propriétaires/open (`nvidia`, `nvidia_modeset`, `nvidia_uvm`, `nvidia_drm`).
    - Intégration de Plymouth avec le thème moderne **`spinner`** et ajout automatique de `quiet splash` dans le bootloader (GRUB, systemd-boot, rEFInd).
    - Hooks modernes (`systemd autodetect microcode modconf kms block filesystems plymouth`).
    - Compression haute vitesse `lz4` (`COMPRESSION="lz4"`, `COMPRESSION_OPTIONS=(-1)`, `MODULES_DECOMPRESS="yes"`).
  - Possibilité de lancer le pack complet tout-en-un (7 étapes) ou de choisir chaque composant individuellement.

- **Environnements de bureau modernes** : Prise en charge de **KDE Plasma 6** (avec Wayland et session moderne) et **GNOME** (générations récentes / GNOME 5x).
- **Dépôts Multilib & Tweaks Pacman** : Menu dédié pour activer d'un clic le dépôt `[multilib]` (essentiel pour le jeu/Steam et la compatibilité 32-bit), avec 10 téléchargements parallèles et `ILoveCandy`.
- **Pilotes GPU récents, Décodage Matériel VA-API & Graphiques Hybrides (Optimus / Switch MUX)** :
  - Profils dédiés pour PC portables hybrides : **Intel + NVIDIA Open** et **AMD + NVIDIA Open**, configurant la stack graphique complète (`intel-media-driver`, `vulkan-intel`, `mesa` 32/64 bits) et l'outil de bascule PRIME (`nvidia-prime` / `prime-run`) pour exploiter le switch MUX ou l'Advanced Optimus.
  - **Accélération matérielle vidéo (VA-API / NVDEC) & Services veille/énergie NVIDIA** :
    - Installation de `libva-nvidia-driver` (décodage vidéo matériel sous Wayland pour Firefox/Chrome/MPV), `libva-intel-driver` / `libva-mesa-driver` et `libva-utils`.
    - Configuration de `/etc/modprobe.d/nvidia.conf` (`modeset=1 fbdev=1`, `NVreg_PreserveVideoMemoryAllocations=1`) et injection dans les paramètres noyau du bootloader.
    - Activation des services `nvidia-suspend.service`, `nvidia-hibernate.service`, `nvidia-resume.service` et **`nvidia-powerd.service`** (Dynamic Boost pour GPU mobile).
  - **Détection dynamique des noyaux pour NVIDIA Open** : si un noyau alternatif est détecté (`linux-zen`, `linux-lts`, `linux-hardened`), le script installe automatiquement **`nvidia-open-dkms`** avec ses en-têtes associés, empêchant `pacman` de forcer l'installation du noyau `linux` standard.
  - Pilotes AMD Radeon, Intel Graphics et NVIDIA Propriétaire (`nvidia-dkms`) disponibles pour toutes configurations.
- **Audio moderne & Services Système étendus** :
  - Stack moderne **PipeWire** (`pipewire-pulse`, `wireplumber`) en remplacement du vieux PulseAudio.
  - Services pré-configurables : `NetworkManager`, `Bluetooth`, `CUPS`, `UFW` Firewall, `fstrim.timer` (SSD TRIM), **`zram-generator`**, **`paccache.timer`** (nettoyage automatique hebdomadaire du cache pacman), **Flatpak + Flathub** (avec intégration logithèque KDE Discover / GNOME Software), et option **Snapper + `snap-pac` + `btrfs-assistant`** (désactivée par défaut pour préserver les configurations Lab `nodatacow`).
- **Boîte à outils, Applications Populaires & Gaming** : Sélection interactive enrichie :
  - **Navigateurs Web** : Firefox (avec pack de langue français automatique `firefox-i18n-fr`), Google Chrome (dépôt CachyOS / AUR), Vivaldi (avec codecs multimédia), Chromium.
  - **Gestionnaire de mots de passe** : Bitwarden.
  - **Multimédia** : VLC, SMPlayer (avec thèmes et skins), MPV.
  - **Monitoring & Outils CLI** : `btop`, `htop`, `fastfetch` (intégré et configuré dans `.bashrc` pour s'exécuter automatiquement à l'ouverture de chaque terminal interactif), `nvtop` (monitoring GPU en temps réel), `inxi`, `nmon`.
  - **Gaming (Optionnel)** : Steam, Lutris, MangoHud + GOverlay (`mangohud`, `lib32-mangohud`, `goverlay`), Feral GameMode (`gamemode`, `lib32-gamemode`), et **`proton-cachyos`** (avec `umu-launcher` & `wine-cachyos`).
  - **Archives** : `p7zip`, `unrar`.

### 3. `stellaris16-plasma` & `stellaris16-gnome` (Full Auto — TUXEDO Stellaris 16)
Deux scripts dédiés 100% automatiques conçus sur mesure pour le **TUXEDO Stellaris 16 (Intel Core Ultra 9 275HX, 96 Go RAM, Samsung 990 Pro NVMe, NVIDIA RTX 5090 Laptop 175W avec Advanced Optimus / Switch MUX)**, sans modifier les scripts originaux `archfi` et `archdi` :
- **Étape interactive unique au départ** :
  1. Partitionnement et formatage identiques à `archfi` (options de montage Btrfs `compress=zstd:1` ou mode Lab `nodatacow,nodatasum` sans snapshots).
  2. Saisie immédiate après montage du `hostname` (`stellaris16`), du compte utilisateur + mot de passe (`sudo NOPASSWD`) et choix du bootloader (`GRUB EFI`, **`rEFInd EFI`** ou `systemd-boot`).
- **Exécution 100% automatique en 14 étapes sans interruption** :
  - **Français intégral (`fr_FR.UTF-8`) & Clavier AZERTY (`fr`)** avant et après (Live ISO, console, X11, Wayland, SDDM/GDM, Plasma/GNOME, Firefox FR, `hunspell-fr`).
  - **Synchronisation horaire** : fuseau `Europe/Paris` + horloge matérielle en `UTC` + `systemd-timesyncd`.
  - **Reflector dès le départ** : 15 miroirs HTTPS les plus rapides sur `Germany,France`, `ParallelDownloads = 10`, `Color`, `ILoveCandy` et dépôt `[multilib]` d'office.
  - **Pack Complet CachyOS & Outils Graphiques CachyOS** : dépôts `x86_64_v3`, noyau `linux-cachyos` + `linux-cachyos-headers` + `dkms` + `cachyos-settings`, démon `ananicy-cpp` + `cachyos-ananicy-rules`, scheduler I/O **`adios`** pour le Samsung 990 Pro NVMe, **ZRAM** (`zstd`, `swappiness=180`), **Plymouth (`spinner`)** + `mkinitcpio` (`lz4 -1`, modules `nvme` + `nvidia`), optimisations **Sysctl** (`kernel.split_lock_mitigate=0`, `vm.max_map_count = 2147483642`), ainsi que tous les outils graphiques CachyOS (**`cachyos-kernel-manager`**, **`cachyos-sysctl-manager`**, **`cachyos-hello`**, **`cachyos-packageinstaller`**, **`scx-manager`**).
  - **Spécificités `stellaris16-gnome` (GNOME & Gaming CachyOS au complet)** :
    - **Extensions GNOME & Thème Sombre unifié** : installation et activation d'office de **`gnome-shell-extension-appindicator`** (`appindicatorsupport@rgcjonas.gmail.com`), du gestionnaire d'extensions **`extension-manager`**, et de **`adw-gtk-theme`** avec activation automatique du thème sombre **`adw-gtk3-dark`** (`prefer-dark`) pour toutes les fenêtres récentes et anciennes (GTK3/GTK4).
    - **Multimédia & Navigateurs** : **`mpv`** (et `vlc`, sans `smplayer`), **`vivaldi`** (`+ vivaldi-ffmpeg-codecs`), `firefox` (`+ firefox-i18n-fr`), `google-chrome`.
    - **Pack Gaming CachyOS Complet, Wine CachyOS & Proton CachyOS** : installation de **`cachyos-gaming-meta`**, **`cachyos-gaming-applications`**, **`proton-cachyos`** (`+ proton-cachyos-slr`, `umu-launcher`), **`wine-cachyos`** (`+ winetricks`, `wine-mono`, `wine-gecko`), ainsi que les gestionnaires graphiques Wine/Proton (**`lutris`**, **`bottles`**, **`heroic-games-launcher-bin`**, **`protonup-qt`**, **`protonplus`**), `steam`, `gamescope`, `gamemode`, `mangohud`, `goverlay` et les tweaks Gaming (`ntsync`, limites Esync/Fsync, cache shaders NVIDIA 12 Go).
  - **GPU Hybride Intel 275HX + NVIDIA RTX 5090 175W Open + CUDA** : stack Intel (`intel-media-driver`, `vulkan-intel`, `intel-compute-runtime`, `vpl-gpu-rt`) + stack NVIDIA Open (`linux-cachyos-nvidia-open` / `nvidia-open-dkms`, `nvidia-utils` 32/64-bit, `nvidia-prime`, `libva-nvidia-driver`, `opencl-nvidia`, **`cuda`**) + service Dynamic Boost **`nvidia-powerd.service`** (175W) et services de veille NVIDIA.
  - **Support Complet TUXEDO + `yay`** : installation directe de `yay`, `tuxedo-drivers-dkms`, `tuxedo-control-center-bin`, masquage des services d'énergie concurrents (`power-profiles-daemon`, `tuned`, `tlp`, `auto-cpufreq`) et activation de `tccd.service` / `tccd-sleep.service`.
  - **Audio PipeWire + Codecs Multimédia + Polices de base + Services** : PipeWire complet, tous les codecs GStreamer/FFmpeg/AV1/x264/x265/DVD, polices complètes (Noto, DejaVu, Liberation, Ubuntu, Fira Code, JetBrains Mono, Roboto), **Bluetooth**, **CUPS + HPLIP**, **`fstrim.timer`**, et **`fastfetch`** automatique dans `.bashrc`.

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
chmod +x archfi archdi stellaris16-plasma stellaris16-gnome
./archfi
# Ou pour l'installation Full Auto dédiée TUXEDO Stellaris 16 :
# ./stellaris16-plasma
# ./stellaris16-gnome
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

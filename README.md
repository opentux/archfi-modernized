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
- **Noyaux, En-têtes & DKMS** : Choix du noyau avec installation intégrée des headers correspondants (`linux-headers`, `linux-lts-headers`, etc.) et de `dkms` dès l'étape pacstrap.
- **Traductions épurées** : Maintenance ciblée uniquement sur le Français et l'Anglais (suppression des fichiers de langues orphelins).

### 2. `archdi` (Post-installation & Environnement de bureau)
Un gros travail d'actualisation a déjà été effectué pour adapter le script aux standards actuels :
- **Listing des paquets dépoussiéré** : Nettoyage en profondeur des paquets abandonnés ou renommés dans les dépôts officiels.
- **Support PC Portables TUXEDO & Gestion d'énergie** :
  - Intégration du pack complet TUXEDO : `tuxedo-drivers-dkms` et `tuxedo-control-center-bin` (AUR).
  - Détection automatique du/des noyau(x) installé(s) et installation transparente des headers et de `dkms`.
  - Résolution des conflits d'énergie : arrêt et masquage automatique (`systemctl mask`) des services concurrents (`power-profiles-daemon`, `tuned`, `tuned-ppd`, `tlp`, `auto-cpufreq`) pour confier la gestion exclusive à `tccd.service` (TUXEDO Control Center), avec option de restauration.

- **Environnements de bureau modernes** : Prise en charge de **KDE Plasma 6** (avec Wayland et session moderne) et **GNOME** (générations récentes / GNOME 5x).
- **Dépôts Multilib & Tweaks Pacman** : Menu dédié pour activer d'un clic le dépôt `[multilib]` (essentiel pour le jeu/Steam et la compatibilité 32-bit), avec 10 téléchargements parallèles et `ILoveCandy`.
- **Pilotes GPU récents** : Intégration du pilote open-source officiel **`nvidia-open`** (architectures Turing, Ampere, Ada Lovelace et plus récentes) aux côtés des pilotes AMD, Intel et NVIDIA propriétaires (avec support DKMS automatisé).
- **Audio moderne** : Intégration de la stack moderne **PipeWire** (avec `pipewire-pulse`, `wireplumber`) en remplacement du vieux PulseAudio.
- **Boîte à outils système au choix** : Sélection interactive d'outils CLI et utilitaires de surveillance modernes :
  - `nvtop` (monitoring GPU en temps réel pour AMD/Intel/NVIDIA),
  - `inxi` (informations système complètes),
  - `nmon` (analyse des performances système),
  - `btop` & `fastfetch` (moniteurs et afficheurs système modernes).

---

## 🚧 En cours & Pistes pour l'avenir (WIP & Roadmap)

Même si une bonne partie du décrassage est faite, il reste du pain sur la planche et plusieurs pistes de réflexion :

### Ce qui reste à finaliser
- **Peaufinage de `archdi`** : Finaliser les sous-menus restants pour l'amener là où je le souhaite (gestionnaires de connexion, profils audio/Bluetooth fins, suites logicielles).
- **Tests en conditions réelles** : Valider les différents scénarios d'installation (UEFI / BIOS, Btrfs vs Ext4, différents couples DE + Display Manager).
- **Nettoyage du code shell** : Poursuivre la simplification et l'unification du code entre les deux scripts.

### Idées & réflexions futures
- **Tuning de performances (façon CachyOS ou équivalent)** :
  - Explorations autour de tweaks système / kernel / sysctl pour maximiser la réactivité.
  - *Réflexion sur les dépôts & noyaux CachyOS* : Les gains aux tests sont indéniables, mais la question se pose sur l'intégration. Si c'est pour faire du bricolage dans le script, autant laisser l'utilisateur l'ajouter proprement après coup. Je cherche la manière la plus propre et modulaire de l'amener, sinon cela restera optionnel/externe.
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

### 2. Récupérer et lancer `archfi`
Clonez ou téléchargez les scripts dans votre environnement live :

```bash
git clone https://github.com/opentux/archfi-modernized.git
cd archfi-modernized
chmod +x archfi
./archfi
```

Suivez les menus guidés pour partitionner votre disque, installer la base système et le chargeur d'amorçage.

### 3. Lancer `archdi` (Post-installation)
Une fois le système de base en place (après avoir redémarré sur votre nouveau système) :

```bash
git clone https://github.com/opentux/archfi-modernized.git
cd archfi-modernized
chmod +x archdi
./archdi
```

Vous pourrez alors activer Multilib, installer vos pilotes GPU (`nvidia-open`, AMD...), déployer Plasma 6 ou GNOME, et sélectionner vos outils favoris (`nvtop`, `inxi`, etc.).

---

## 📜 Licence & Crédits

- Projet original conçu par [MatMoul](https://github.com/MatMoul) (archfi / archdi).
- Distribué sous licence **GPLv3**.

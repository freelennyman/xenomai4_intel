.. prerequis.rst

Pré-requis & principes
======================

Contexte
--------

- Distribution : **Ubuntu 24.04 LTS** (amd64).
- Machine cible : **x86_64 Intel** (ex. laptop avec SSD **NVMe**, racine en **ext4**).
- Objectif : compiler et installer un noyau **EVL** (Xenomai 4), puis l’espace
  utilisateur **libevl**, et valider par des **mesures** de latence.

Concepts clés (vue d’ensemble)
------------------------------

- **EVL** (Embedded Virtual Linux) : cœur temps réel **out-of-band** (OOB) qui
  préempte Linux via la **pipeline IRQ** (Dovetail). Il traite les tâches
  critiques **avant** le noyau Linux (*in-band*).
- **/dev** (devtmpfs) : EVL expose des périphériques caractères
  (ex. ``/dev/evl/control``). Le montage automatique de **devtmpfs**
  crée ces nœuds au boot.
- **Chaîne de boot** : le noyau doit **voir** le disque racine (ex. NVMe)
  et **lire** le système de fichiers (ex. ext4) **dès le démarrage**. Pour éviter
  les blocages, on compile **en intégré (=y)** les pilotes **NVMe/PCI** et **EXT4**.

Paquets de build
----------------

Installer les outils nécessaires :

.. code-block:: bash

    sudo apt update
    sudo apt install -y build-essential bc bison flex libssl-dev libelf-dev \
        libncurses-dev dwarves git debhelper devscripts fakeroot \
        pkg-config meson ninja-build

Sources (GitLab)
----------------

.. code-block:: bash

    mkdir -p ~/rt && cd ~/rt
    # Noyau EVL (choisir un tag EVL stable)
    git clone --depth=1 --branch v6.12.30-evl5 https://gitlab.com/xenomai/xenomai4/linux-evl.git
    # Espace utilisateur libevl (r55 au moment du guide, compatible)
    git clone --depth=1 --branch r55 https://gitlab.com/xenomai/xenomai4/libevl.git

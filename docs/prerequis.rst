.. prerequis.rst

Pré-requis & principes
======================

Contexte
--------

- Distribution : **Ubuntu 24.04 LTS** (amd64).
- Machine cible : **x86_64 Intel** (ex. laptop avec SSD **NVMe**, racine en **ext4**).
- Objectif : compiler et installer un noyau **EVL** (Xenomai 4), puis l’espace
  utilisateur **libevl**, et valider par des **mesures** de latence.

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

.. libevl_userland.rst

Espace utilisateur — libevl & dépendances
=========================================

But
---

Installer l’**API utilisateur** et les **outils** EVL : ``evl``, ``latmus``, ``hectic``, etc.

Dépendance critique : libbpf
----------------------------

Sur Ubuntu, la ``libbpf`` des dépôts peut être **trop ancienne** (ex. 0.5).  
Or libevl requiert des APIs **≥ 0.6**. Installer **libbpf 1.x** depuis les sources :

.. code-block:: bash

    sudo apt install -y libelf-dev zlib1g-dev
    cd ~/rt
    git clone --depth=1 --branch v1.6.2 https://github.com/libbpf/libbpf.git
    cd libbpf/src
    make -j"$(nproc)"
    sudo make install PREFIX=/usr LIBDIR=/usr/lib/x86_64-linux-gnu
    sudo ldconfig
    pkg-config --modversion libbpf   # doit afficher 1.x

Compilation & installation de libevl
------------------------------------

.. code-block:: bash

    cd ~/rt/libevl
    rm -rf build && mkdir build && cd build


    meson setup -Dbuildtype=release -Dprefix=/usr -Duapi=~/rt/linux-evl -Dwerror=false ..
    meson compile
    sudo ninja install
    sudo ldconfig

Accès non-root aux devices EVL
------------------------------

.. code-block:: bash

    sudo groupadd -f evl
    sudo usermod -aG evl "$USER"

    cat <<'RULE' | sudo tee /etc/udev/rules.d/60-evl.rules
    SUBSYSTEM=="evl", KERNEL=="control", MODE="0660", GROUP="evl"
    SUBSYSTEM=="evl", KERNEL=="clone",   MODE="0660", GROUP="evl"
    SUBSYSTEM=="evl", KERNEL=="*/clone", MODE="0660", GROUP="evl"
    RULE

    sudo udevadm control --reload && sudo udevadm trigger
    # activer le groupe sans relogin
    newgrp evl

Vérifications
-------------

.. code-block:: bash

    which evl latmus hectic
    ls -l /dev/evl
    evl ps

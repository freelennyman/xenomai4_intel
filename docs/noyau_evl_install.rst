.. noyau_evl_install.rst

Noyau EVL (Xenomai 4) — Build & installation
============================================

Configuration minimale **indispensable**
----------------------------------------

Depuis l’arbre ``~/rt/linux-evl`` :

.. code-block:: bash

    cd ~/rt/linux-evl

    # Configuration de base : garantit x86_64 (ou partez d'une config pour votre infrastructure)
    
    make x86_64_defconfig

    # EVL + Dovetail + outils de bench
    ./scripts/config --enable EVL --enable DOVETAIL \
                     --enable EVL_LATMUS --enable EVL_HECTIC

    # /dev auto + initramfs lisible + audit config
    ./scripts/config --enable DEVTMPFS --enable DEVTMPFS_MOUNT \
                     --enable BLK_DEV_INITRD --enable RD_GZIP --enable RD_ZSTD \
                     --enable IKCONFIG --enable IKCONFIG_PROC

    # Chaîne de boot intégrée (ex. NVMe + ext4)
    ./scripts/config --enable BLK_DEV_NVME --enable NVME_PCI --enable EXT4_FS

    yes "" | make olddefconfig

Vérification rapide pour voir si tout est activé :

.. code-block:: bash

    grep -E '^CONFIG_(EVL|DOVETAIL|EVL_LATMUS|EVL_HECTIC|DEVTMPFS|BLK_DEV_NVME|NVME_PCI|EXT4_FS)=' .config

Compilation en paquets Debian
-----------------------------

.. code-block:: bash

    export LOCALVERSION=-evl
    export LOCALVERSION_AUTO=n   # noms de paquets sans suffixe -g<sha>, plus lisibles
    make -j"$(nproc)" bindeb-pkg

Les paquets sont produits **dans le dossier parent** (``~/rt``). Installer :

.. code-block:: bash

    cd ..
    # on remplace les étoiles par les images correspondantes après avoir fait le make pour les paquets Debian
    sudo dpkg -i linux-image-6.12.30-evl_*.deb linux-headers-6.12.30-evl_*.deb
    sudo update-grub

Redémarrage & contrôles
-----------------------

Au boot, choisir **Ubuntu, with Linux 6.12.30-evl**. Après login :

.. code-block:: bash

    uname -r                 # doit contenir 6.12.30-evl
    ls -l /dev/evl           # doit afficher au moins control / clone
    evl ps                   # doit répondre

Si ``/dev/evl`` n’apparaît pas, vérifier :

- **EVL** activé (``CONFIG_EVL=y``) dans ``/boot/config-$(uname -r)``.
- Ajouter au besoin ``evl.state=started`` dans ``/etc/default/grub`` :


Erreurs fréquentes et solutions (noyau)
---------------------------------------

**Si vous avez un blocage à “Loading initial ramdisk” au boot**

- Cause principale : pilotes **NVMe** ou **EXT4** manquants dans le noyau/ initramfs.
- Fix : s’assurer que ``CONFIG_BLK_DEV_NVME=y``, ``CONFIG_NVME_PCI=y``, ``CONFIG_EXT4_FS=y`` puis recompiler.


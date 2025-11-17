Installation de Xenomai 4 (EVL) sur Raspberry Pi 4
==================================================

Ce document décrit, étape par étape, comment compiler et installer
un noyau Linux EVL/Xenomai 4 sur un Raspberry Pi 4 (64 bits), puis
installer la bibliothèque utilisateur **libevl** et tester l’ensemble
avec l’outil ``latmus``.

L’exemple part d’un système de type Ubuntu pour Raspberry Pi, avec un
utilisateur standard (par exemple ``ubuntu``) ayant les droits ``sudo``.


1. Vérifier le noyau actuel
---------------------------

Avant de modifier quoi que ce soit, on note la version de noyau
actuellement utilisée. Cela permettra de revenir en arrière en cas de
problème et de vérifier que le nouveau noyau est bien pris en compte.

.. code-block:: bash

   uname -r          # Affiche uniquement la version du noyau
   uname -a          # Affiche toutes les infos sur le noyau en cours

Conserver ces informations quelque part (capture, note) peut être utile
pour la suite.


2. Récupérer les sources du noyau Raspberry Pi et EVL
-----------------------------------------------------

On commence par récupérer les sources du noyau Raspberry Pi et la branche
EVL/Xenomai 4 correspondante. L’option ``--depth=1`` limite l’historique
Git pour accélérer le clonage.

.. code-block:: bash

   # Noyau Raspberry Pi
   git clone --depth=1 https://github.com/raspberrypi/linux ~/linux

   # Noyau Linux EVL (Xenomai 4)
   git clone --depth=1 https://source.denx.de/Xenomai/xenomai4/linux-evl ~/linux-evl

On suppose ici que les deux dépôts sont clonés dans le répertoire
personnel (``~/``).


3. Préparer la configuration du noyau
-------------------------------------

On installe d’abord les dépendances nécessaires à la compilation du noyau
Linux (outils de build, bibliothèques, etc.).

.. code-block:: bash

   sudo apt update
   sudo apt install -y git bc bison flex libssl-dev make libncurses-dev

On se place ensuite dans les sources du noyau Raspberry Pi pour générer
une configuration de base adaptée au Raspberry Pi 4 (bcm2711).

.. code-block:: bash

   cd ~/linux

   # Charge une configuration par défaut pour Raspberry Pi 4 (aarch64)
   make bcm2711_defconfig

Cette étape crée un fichier ``.config`` de base dans ``~/linux``. On va
s’en servir comme point de départ pour la configuration du noyau EVL.


4. Copier et ajuster la configuration dans le noyau EVL
-------------------------------------------------------

L’idée est de partir de la configuration fonctionnelle du noyau
Raspberry Pi, puis d’y ajouter le support EVL/Xenomai 4.

.. code-block:: bash

   cd ~/linux-evl

   # On recopie la configuration générée dans ~/linux vers le noyau EVL
   cp ~/linux/.config ~/linux-evl/.config

Ensuite, on ouvre le menu de configuration pour ajuster quelques options.

.. code-block:: bash

   make menuconfig

Dans ``menuconfig`` :

1. **Changer le suffixe de version du noyau**

   Aller dans :

   - ``General setup`` → ``Local version - append to kernel release``

   Remplacer par exemple ``-v8`` par ``xeno-v8`` pour pouvoir identifier
   facilement le noyau EVL/Xenomai :

   - Mettre : ``-xeno-v8``

   Cela permettra d’obtenir plus tard une chaîne du type
   ``6.12.55xeno-v8+`` lors d’un ``uname -r``.

2. **Vérifier le support du fichier .config**

   S’assurer que l’option de support de la configuration du noyau est
   activée (cela permet notamment de retrouver la configuration utilisée
   directement depuis ``/proc/config.gz``).

   L’option correspondante est généralement :

   - ``General setup`` → ``Kernel .config support`` → **[Y]**

3. **Activer le cœur temps réel EVL**

   Dans ``menuconfig``, aller dans la section spécifique au noyau EVL,
   par exemple :

   - ``Kernel Features`` ou une section similaire → activer
     **EVL real-time core** (ou équivalent), afin d’intégrer le cœur
     temps réel EVL dans le noyau.

Après cela, on peut sauvegarder et quitter ``menuconfig``.

Pour être sûr que Xenomai/EVL est bien activé, vérifier (ou forcer) les
entrées suivantes dans ``.config`` :

.. code-block:: bash

   # Dans ~/linux-evl/.config, vérifier la présence des lignes :
   CONFIG_XENOMAI=y
   CONFIG_XENOMAI_CORE=y

Si nécessaire, les ajouter manuellement, puis relancer un
``make oldconfig`` pour vérifier la cohérence des options :

.. code-block:: bash

   make oldconfig


5. Compilation du noyau EVL pour Raspberry Pi 4
-----------------------------------------------

Une fois la configuration prête, on compile le noyau, les modules et les
Device Tree Blobs (DTBs). L’option ``-j4`` utilise 4 cœurs de compilation
(adapter selon le nombre de cœurs disponibles).

.. code-block:: bash

   cd ~/linux-evl

   make -j4 Image modules dtbs

Cette étape peut prendre un certain temps et nécessite de la mémoire
(disque et RAM). À la fin, on dispose d’un noyau EVL compilé, de ses
modules et des fichiers Device Tree associés.


6. Installation du noyau et des modules
---------------------------------------

On installe d’abord les modules du noyau dans le système :

.. code-block:: bash

   sudo make modules_install

Ensuite, on installe le noyau et les DTB dans la partition de boot
(ici ``/boot/firmware``, chemin standard sur Ubuntu pour Raspberry Pi) :

.. code-block:: bash

   # Copier les DTB (description matériel) Broadcom
   sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware

   # Copier l’image du noyau en remplaçant kernel8.img
   sudo cp arch/arm64/boot/Image /boot/firmware/kernel8.img

**Attention :** cette dernière commande remplace le noyau existant
``kernel8.img``. Il peut être prudent de faire une sauvegarde préalable
du fichier original (par exemple ``kernel8.img.bak``).


7. Configuration du boot (config.txt)
-------------------------------------

On configure maintenant le Raspberry Pi pour qu’il démarre sur ce nouveau
noyau EVL/Xenomai et applique l’overlay EVL.

.. code-block:: bash

   sudo nano /boot/firmware/config.txt

Ajouter (ou modifier) les lignes suivantes :

.. code-block:: text

   kernel=kernel8.img
   dtoverlay=evl

Sauvegarder puis quitter l’éditeur. Au prochain redémarrage, le Pi
chargera le noyau EVL.

Après redémarrage, vérifier que le nouveau noyau est bien actif :

.. code-block:: bash

   uname -a

Un résultat attendu ressemble à :

.. code-block:: text

   Linux raspberrypi 6.12.55xeno-v8+ #1 SMP PREEMPT EVL Thu Nov 13 07:01:35 GMT 2025 aarch64 GNU/Linux

La présence de ``xeno-v8`` et de ``EVL`` indique que le noyau temps réel
EVL est correctement chargé.


8. Installation de libevl (espace utilisateur)
----------------------------------------------

Pour utiliser les services EVL côté utilisateur, on installe la
bibliothèque **libevl**.

.. code-block:: bash

   # Récupération des sources
   git clone https://source.denx.de/Xenomai/xenomai4/libevl ~/libevl

   # Répertoire de build séparé
   mkdir ~/libevl-build
   cd ~/libevl-build

On installe la dépendance ``libbpf-dev``, nécessaire pour certaines
fonctionnalités de libevl :

.. code-block:: bash

   sudo apt-get update
   sudo apt-get install -y libbpf-dev

On configure ensuite la compilation via Meson. On indique le type de
build, le préfixe d’installation et le chemin vers les en-têtes UAPI du
noyau EVL (``~/linux-evl``) :

.. code-block:: bash

   meson setup \
     -Dbuildtype=release \
     -Dprefix=/usr/local \
     -Duapi=~/linux-evl \
     ~/libevl

Puis on compile :

.. code-block:: bash

   meson compile

Et on installe la bibliothèque :

.. code-block:: bash

   sudo ninja install


9. Configuration de l’édition de liens (ld.so)
----------------------------------------------

On s’assure que le chargeur de bibliothèques dynamique (``ld.so``) sait où
trouver ``libevl`` installée dans ``/usr/local``. On ajoute le répertoire
à la configuration de ``ld.so`` :

.. code-block:: bash

   echo "/usr/local/lib/aarch64-linux-gnu" | sudo tee -a /etc/ld.so.conf.d/libevl.conf

Puis on recharge la configuration :

.. code-block:: bash

   sudo ldconfig

On peut également exporter la variable d’environnement
``LD_LIBRARY_PATH`` pour les sessions shell (optionnel si ``ldconfig``
est correctement configuré) :

.. code-block:: bash

   export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
   echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
   source ~/.bashrc

**Remarque :** corriger toute faute de frappe éventuelle, par exemple
éviter ``SLD_LIBRARY_PATH`` qui est incorrect.


10. Test du système EVL avec latmus
-----------------------------------

À ce stade, le noyau EVL est installé et ``libevl`` est disponible.
On peut tester le fonctionnement en temps réel avec l’outil
**latmus** (installé dans ``/usr/local/bin``) :

.. code-block:: bash

   cd /usr/local/bin
   sudo ./latmus

L’exécution de ``latmus`` en tant que superutilisateur permet de mesurer
les latences du système EVL/Xenomai sur le Raspberry Pi 4. En observant
les résultats, on peut vérifier que le système répond bien aux attentes
en termes de temps réel dur.

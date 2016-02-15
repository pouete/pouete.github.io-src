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

Diffie-Hellman :
~~~~~~~~~~~~~~~~

Pour que deux ados puissent se parler entre eux dans la classe et pouvoir s'échanger des messages qu'eux seul peuvent lire. Il faut qu'ils se mettent d'accord sur une clé que meme une personne les écoutent ne pourra connaitre. C'est exactement ce que fait Diffie Hellman.
Imaginons un instant que ces clés sont des couleurs. Et que ces ados veulent échanger une couleur secrète dans une salle remplie de monde qui les écoutent.
Diffie-Hellman repose sur l'ếquivalent mathématique suivant :
Quand deux couleurs sont mélangées cela en donne une troisième. il est très difficile de prendre cette troisieme couleur et de revenir arrière pour connaitre les deux primaires utilisées.

Reprenons nos deux ados qui vont décider publiquement d'une couleur: la couleur publique.
Il vont ensuite chacun choisir une couleur qui leur est propre et la mélanger avec la couleur publique que nous appelerons premier mélange.
Bien sûr ce premier mélange est bien différent chez l'un et l'autre ado.
Ils s'envoient le premier mélange l'un l'autre, de manière publique, tout le monde peut la voir mais personne ne peut connaitre la couleur propre.
Il vont enfin mélanger ce premier mélange à leurs couleurs propre et cela va donner une quatrième couleur qui sera exactement la même chez l'un et chez l'autre.
Cela sera leur clé , et PERSONNE ne peux connaitre cette clé avec les données qui est leur clé secrete.


Pour pouvoir trouver la clé privée de chacun , il faudra tester toute les couleurs à disposition jusqu'a trouver la couleur utilisée.
C'est pareil pour Diffie-Hellman, il faudra tester un très grand nombre de chiffres ( un nombre de plus de 77 chiffres  ) avant de trouver le bon.


Voila pour la version facile de Diffie hellman. Dans les faits, les couleurs publiques sont en fait des groupes de nombres premiers sûrs bien définis.

premiers sûrs :
~~~~~~~~~~~~~~~

Les nombres premiers sûrs sont, pour faire simple, des chiffres qui complexifient grandement le calcul de la clé privée. Oui, parce que certain coréen se sont amusé à `simplifier les calculs avec des nombres premiers non surs <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.5296>`_.
Mais pas de problèmes ! openSSL utilise toujours des nombres premiers sûrs et même si ce n'est pas le cas, L'attaque est basée sur le fait que on utilise toujours la même clé privée partout tout le temps ! et ce n'as pas un problème ! puisque openSSL ne ferai jamais ça ! non ?


RFC 5114 :
~~~~~~~~~~

La RFC 5114 défini de nouveaux groupes de nombre premiers ( souvenez vous, les groupes sont des couleurs de départ ). Oui mais voilà, la plupart ne sont pas sûrs et les pires étant le groupe n*23 qui peuvent simplifier le calcul de la clé privée de 137 bits. Donc, si vous aviez choisi une clé de 224 bits, il ne reste que 44 à calculer.

Le dernier clou :
~~~~~~~~~~~~~~~~~

Ouais, bon, c'est pas trops grâve, OpenSSL gère tout ça et on à le droit à une nouvelle clé à chaque fois, ça fait perdre du temps, l'attaque des coréen n'est pas efficace dans ce cas !

SURPIIIISE , non.

Ben ouais, c'est la le problème: openSSL à une option activée par défaut : le fait de ne PAS regénerer la couleur secrète à chaque fois. Le hacker peut donc tranquillement attaquer la clé simplifiée.

Conclusion :
~~~~~~~~~~~~

Comment une petite erreur peut se transformer une catastrophe.
Encore ici, on voit qu'on a pas besoin d'exploitation complexe pour attaquer une connexion chiffrée.
Le correctif est déjà disponible et aura surement donné quelque sueurs froides à certain sysadmins de pars le monde.
Bon,dans le pire des cas, l'attaquant devra tout de même calculer une clé d'une taille de 2^44 ( nombre à 13 chiffres ), donc ça vas, aucun gouvernement ou entitée mafieuse n'a ce genre de `puissance de calcul <http://www.top500.org/lists/2015/11/>`_, n'est-ce-pas ?

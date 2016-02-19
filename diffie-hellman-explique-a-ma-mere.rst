Diffie Hellman :
################
:date: 2016-02-16 14:00
:slug: DH-a-ma-mpere
:category: Vulgarisation
:status: published
:tags: DH, Diffie hellman

Introduction :
~~~~~~~~~~~~~~

Le chiffrement sur internet est aujourd'hui partout et surtout vital. On me demande souvent comment fonctionne la cryptographie et ma réponse est souvent "ça marche, c'est tout".
La réponse suffit à beaucoup, mais certains me demandent plus. Les méthodes de chiffrement sont souvent facile à expliquer: partir du code césar par exemple et étendre à l'informatique, ça passait. Mais une partie était souvent très difficile à expliquer simplement: l'échange de clés.
Cet article est une explication "simplifiée" d'une technique d'échange de clés trouvée communément sur internet.

La clé privée :
~~~~~~~~~~~~~~~
C'est tout simplement le code secret qui va permettre de déchiffrer ou de chiffrer un message.  Dans notre cas, les deux communicants ont la même clé privée. Oui, mais voilà, internet est un lieu bruyant , et tout le monde peut voir et lire les conversations : c'est pourquoi on utilise le chiffrement.
Mais dans ce cas, comment communiquer une clé privée à un tiers sans que personne d'autre ne puisse connaitre cette clé ?
La réponse : Diffie Hellman

Diffie-Hellman expliqué à ma mère ( et mon père ) :
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Internet, c'est un peu comme une cour d'école, c'est bruyant, et tout le monde peut écouter tout le monde.

Alors, comment vont faire deux ados pour se parler entre eux et pouvoir s'échanger des messages qu'eux seuls peuvent lire.

Il faut qu'ils s'accordent sur une clé que personne ne peut connaitre même en les écoutant.

C'est exactement ce que fait Diffie Hellman. Imaginons un instant que ces clés soient des couleurs.

Ces ados veulent s'échanger une couleur secrète.

Diffie-Hellman repose sur l'équivalent mathématique suivant : Quand deux couleurs sont mélangées cela en donne une troisième. Il est très difficile de prendre cette troisième couleur et de revenir arrière pour connaitre les deux primaires utilisées.
Reprenons nos deux ados qui vont décider publiquement d'une couleur: la couleur publique.

Ils vont ensuite chacun choisir une couleur qui leur est propre et la mélanger avec la couleur publique que nous appellerons premier mélange. Bien sûr ce premier mélange est bien différent chez l'un et l'autre ado. Ils s'envoient le premier mélange l'un à l'autre, de manière publique, mais personne ne pourra connaitre la couleur propre.
Ils vont enfin mélanger ce premier mélange à leurs couleurs propres et cela va donner une quatrième couleur qui sera exactement la même chez l'un et chez l'autre.

Cela sera leur clé privée, et PERSONNE ne pourra connaitre cette clé avec les couleurs publiques qu'ils ont utilisées pour créer cette clé privée.

Afin de trouver la clé privée de chacun , il faudra tester toutes les couleurs à disposition jusqu'à trouver la couleur utilisée. Diffie-Hellman utilise le même principe, il faudra tester beaucoup de nombres ( un nombre de plus de 77 chiffres ) avant de trouver la clé privé.

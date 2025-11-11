src = https://youtu.be/NTgMJOsHpOs?si=z1DBcUK565-QDTdo  

# N'utilise plus RECHERCHEV

## Le problème de RECHERCHEV

RECHERCHEV va chercher la valeur spécifiée dans la toute **première colonne** de la plage sélectionnée, et nulle part ailleurs.  
Donc si on veut chercher une donnée située sur la droite de la plage et en récupérer une autre située sur la gauche, ça ne marchera pas.  

De plus, la suppression ou le déplacement d'une colonne dans la plage fournie à RECHERCHEV va casser son fonctionnement.  

Enfin, RECHERCHEV est une fonction qui est assez lente.  
Elle convient pour des petits tableaux, mais pas pour des milliers de lignes.

## RECHERCHEX


# Tableaux Croisés Dynamiques

## Pré-requis

- Les données doivent être organisées en **colonnes**, et non pas en lignes.  
- Les données à analyser doivent être dans un même tableau, donc dans un même onglet (feuille).
- Il faut définir des en-tête pour chaque colonne.
- Pas de lignes vides

## Créer un TCD

- sélectionner les données
    - se placer sur n'importe quelle cellule du tableau et faire Ctrl+A si vous voulez tout sélectionner
- aller dans le menu Insertion et cliquer sur Tableau croisé dynamique
- au début, le TCD est vide, c'est normal

## Remplir le TCD

Excel vous indique la marche à suivre pour générer un rapport (un TCD) :
- à droite de la feuille contenant le futur TCD, on a un panneau avec les "Champs de tableau croisé dynamique"
- il faut sélectionner les champs qui nous intéressent et les faire glisser vers la zone voulue
- on doit définir ce qu'on veut mettre en lignes, en colonnes, en valeurs

Exemple avec des données source contenant les bénéfices d'une boutique en ligne:
- dans la zone "lignes", on peut mettre les pays
- dans la zone "valeurs", on peut ajouter les bénéfices
ça va nous afficher les résultats pour chaque pays
- ensuite, on peut ajouter les types de produit dans la zone "colonnes"
ce qui va nous afficher le détail des bénéfices pour chaque type de produit

### Règles importantes

- L'ordre dans lequel on dispose les champs dans chaque zone est important
- La zone des valeurs est destinée à accueillir des données chiffrées
- Un TCD ne se met pas à jour tout seul, il faut faire un clic-droit sur le TCD et Actualiser
- par défaut, un TCD propose des filtres pertinents, aussi bien pour les lignes que pour les colonnes

## Menus propres au TCD

Quand on est sur une cellule de notre TCD, apparaissent 2 menus en haut du fichier Excel:
- Analyse du TCD
- Création

Si jamais le panneau latéral "champs de TCD" n'est pas visible, aller sur l'onglet "analyse du TCD" et cliquer sur "liste des champs".  


___
@11/26
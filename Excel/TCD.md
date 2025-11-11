src = https://youtu.be/hc1a1t4YBWM?si=lZlW1lfSIa5CAKjt  

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

## Règles importantes

- L'ordre dans lequel on dispose les champs dans chaque zone est important
- La zone des valeurs est destinée à accueillir des données chiffrées
- Un TCD ne se met pas à jour tout seul, il faut faire un clic-droit sur le TCD et Actualiser
- par défaut, un TCD propose des filtres pertinents, aussi bien pour les lignes que pour les colonnes

## Menus propres au TCD

Quand on est sur une cellule de notre TCD, apparaissent 2 menus en haut du fichier Excel:
- Analyse du TCD
- Création

Si jamais le panneau latéral "champs de TCD" n'est pas visible, aller sur l'onglet "analyse du TCD" et cliquer sur "liste des champs".  

## Zone "Valeurs"

Par défaut, Excel somme les valeurs ajoutées à cette zone. Mais ce comportement peut être modifié.  
- Pour cela, cliquer sur la liste déroulante à côté de l'élément à modifier (dans la zone des valeurs).
- Sélectionner "paramètres des champs de valeurs"
    - On peut alors modifier le type de calcul utilisé pour synthétiser les valeurs
    - ou bien transformer les valeurs brutes pour afficher par exemple des pourcentages
        - % du total de la ligne
        - % du total de la colonne
        - etc.

## Modification de la source de données

Si on modifie la source de données après la création du TCD, le TCD ne va pas s'actualiser tout seul.  
Il faut aller dans l'onglet "analyse dy TCD" et cliquer sur "changer la source des données".  
Il suffit de modifier la plage sélectionnée pour que le TCD prenne en compte les nouvelles données.  

### Astuce pour améliorer la sélection des données

Idéalement, ceci est à faire avant la création du TCD:
- Ctrl + A pour sélectionner l'ensemble de vos données source
- Onglet Insertion > clic sur Tableau

Cela va créer un tableau: 
- assurez-vous que la zone sélectionnée est la bonne
- et que la case "mon tableau comporte des en-têtes" est cochée

Une fois le tableau créé, un onglet "création de tableau" apparaît:
- nommez votre tableau comme vous le souhaitez, ce sera votre source de données
- vous pourrez ensuite utiliser ce nom comme référence lors de la création du TCD

La création d'un tableau vous évitera d'avoir à changer la source de données de votre TCD lorsque vous ajouterez des données dans votre tableau,
l'actualisation du TCD via un clic-droit suffira.

## Mise en forme d'un TCD

Cela se fait via l'onglet "Création". On peut y modifier:
- l'affichage des totaux généraux
- l'affichage des sous-totaux
- la disposition du rapport

## Ne pas partager des données confidentielles

- bien actualiser le TCD avant de procéder
- clic-droit sur le TCD > Options du TCD
- onglet Données > décocher les cases "enregistrer données source avec le fichier" et "activer affichage des détails"
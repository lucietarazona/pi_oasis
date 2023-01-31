Ce qui a changé dans la dernière version de mon node red 

Dans la partie recherche de salle :
* dans le form "Recherche de salle", la critère de la durée est en `Text`ET NON `Time`!!!!!!
* dans la fonction "id", la durée est convertie en minutes

L'identifiant de la salle a été centralisé, ce qui fait qu'il n'y a plus besoin d'aller changer l'id directement dans les noeuds comme avant : l'id de la salle est stocké dans la variable globale `global.id`, et une ligne du type `global.get("id")` permet de récupérer l'id dans les noeuds `function` où jusuq'à présent il fallait rentrer l'id à la main. 
Pour modifier l'id de la salle, il faut cliquer sur le bouton avec un logo "paramètres" sur l'écran d'accueil (celui où il y a l'emploi du temps et les données des capteurs). Le bouton emmène l'utilisateur sur une page où on lui demande un mot de passe. Celui-ci peut être configuré dans le node red mais pas depuis l'affichage, puisque dans l'idée on ne devrait pas le modifier. Le mot de passe est défini dans la partie `setup mdp`en haut à droite du flow. Quand l'utilisateur clique sur le bouton valider, s'il a renseigné le bon mot de passe il est redirigé vers une page où il peut changer l'identifiant de la salle : l'identifiant actuel sera par défaut écrit dans le champ de texte, il suffit de le supprimer, d'indiquer le nouvel identifiant, et de valider. On revient alors sur l'écran d'accueil (et l'identifiant a bien été actualisé : on peut le constater sur l'écran de couleur, on directement dans node red grâce à la partie `debug global`et plus particulièrement la ligne `check id`).
**Remarque :** L'état de la salle ne s'actualisera que lorsque le nouvel emploi du temps sera reçu. En attendant, l'écran de couleur sera bleu car le statut n'est pas défini.

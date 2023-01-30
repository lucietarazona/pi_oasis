Ceci est l'un des deux fichiers Node-Red qui ont été fusionnés pour obtenir l'interface utilisateur du PI Oasis. 

Cette version permet entre autres:
- d'afficher les **données** (température, concentration en CO2, humidité) mesurées par les capteurs
- d'afficher **l'emploi du temps** de la salle envoyé par le serveur
- de réaliser une **requête de salle** auprès du serveur
- d'afficher la **salle proposée** par le serveur en réponse à cette requête
- d'indiquer que l'on s'apprête à **occuper la salle** (dans le cas où elle est libre)
- d'indiquer **l'état d'occupation** de la salle
- de **changer l'écran affiché** selon si un utilisateur est proche de la tablette ou non

Les pincipaux ajouts dans la version finale du flow sont l'affichage d'une carte 3D de l'Ecole des Mines et l'affichage de la date et de l'heure.

### Gestion des statuts de la salle

La salle peut être dans trois états : **libre**, **occupée** ou **réservée**. L'état de la salle est stocké dans la variable globale *global.payload*, et est donc **accessible par tous les noeuds** sans avoir à tracer de liens explicites. Plusieurs boutons et éléments d'affichage voient leurs propriétés changer selon l'état de la salle.
J'ai établi un **ordre de priorité** dans le flow pour gérer les changements de statuts. Si la salle est réservée d'après l'emploi du temps, alors rien ne peut modifier le statut. Si la salle indiquée libre par l'emploi du temps, il est possible de l'occuper : le statut "occupée" est prioritaire sur le statut "libre" indiqué par l'emploi du temps.

###  Description des sections du flow 


- **Affichage selon la présence de l'utilisateur**
Deux écrans d'accueil coexistent. Le premier (**écran "basique"**) indique uniquement le nom de la salle et son statut par la couleur de l'arrière-plan : vert pour "libre", orange pour "occupée", rouge pour "réservée". De cette façon, l'utilisateur sait tout de suite, et de loin, le statut de la salle. Quand il s'approche de la tablette, l'affichage bascule sur **l'écran d'accueil** à proprement parler, avec toutes les informations liées à la date, l'emploi du temps, etc. 
Un script Python récupère les données de présence reçues par un capteur infrarouge, branché à la carte RaspberryPi. Un noeud *pythonshell* exécute ce script (:warning: écrire le chemin complet si le fichier .py n'est pas dans le même répertoire que le flow) et un noeud *switch* donne le comportement à suivre selon le résultat du programme : si on reçoit *False* (pas de présence), on affiche l'écran basique et on attend 5 secondes avant de relancer le programme ; si on reçoit *True*, on bascule sur l'écran détaillé et on attend une minute avant de relancer le programme. Les attentes sont gérées par des noeuds *delay*. 

- **Affichage des données des capteurs**
La température, l'humidité et la concentration en CO2 sont collectées par une carte Arduino, elle-même en communication avec la carte RaspberryPi qui pilote l'écran tactile. Un programme Python traite les données, les rendant récupérables par Node-Red. Les données arrivent simultanément les unes à la suite des autres, séparées par des virgules (une nouvelle série de données arrive toutes les 5 minutes). 
Le bloc *pythonshell* permet d'exécuter le code Python indiqué dans les réglages du bloc (:warning: écrire le chemin complet si le fichier .py n'est pas dans le même répertoire que le flow).
Le bloc *function* sépare les trois données et les renvoie chacune sur la sortie adéquate.

- **Demande de salle**
L'utilisateur remplit dans un formulaire les **critères** de sa recherche de salle. Le bloc *function* ajoute l'identifiant de la salle au message, puis la requête est envoyée au serveur.

- **Salles disponibles**
En réponse à la requête, le serveur renvoie une proposition de salle (rendue lisible par le bloc *json*). La proposition est affichée à l'écran. Si aucune salle remplissant les critères de recherche n'est disponible, l'information est aussi affichée sur l'écran.

- **Occuper la salle**
Si la salle est libre, l'utilisateur peut la prendre en cliquant sur le bouton *prendre la salle*. Il est redirigé vers une page où il lui est demandé d'indiquer la durée pendant laquelle il compte occuper la salle (notons qu'aucune sécurité n'a été mise en place pour éviter que l'utilisateur indique une durée qui empièterait sur un créneau réservé, la personne ayant réservé ayant toute légitimité à récupérer la salle à l'heure qu'elle a demandé). Pour faciliter le traitement, on demande à l'utilisateur d'indiquer la durée en utilisant deux chiffres pour les heures, deux points, puis deux chiffres pour les minutes.
La chaîne de caractères récupérées est convertie en un nombre de minutes par le noeud *function* (qui en même temps ajoute l'identifiant de la salle au message). Le message obtenu est envoyé au serveur par websocket.
Par ailleurs, un autre noeud *function* déclenche le passage de *global.payload* à "occupée", et envoie la durée d'occupation à un noeud *delay* qui déclenche le passage de *global.payload* à libre à la fin de la durée d'occupation indiquée par l'utilisateur.

- **Emploi du temps**
L'emploi du temps est envoyé par le serveur toutes les 15 minutes. Il est converti en string par un noeud *json* pour être traité par un noeud *function* qui le formate pour être lu par le noeud *table*. L'emploi du temps formaté est aussi envoyé à un autre noeud *function* qui compare les créneaux de réservation de l'emploi du temps à l'heure courante pour déterminer le statut de la salle.

### Points de vigilance
- Il est nécessaire d'envoyer l'identifiant de la salle au démarrage et à chaque requête 
:arrow_right: pour cela, il faut veiller pour chaque nouvelle tablette à indiquer la salle à laquelle elle correspond dans les noeuds *function* des fonctionnalités "identification", "demande de salle", et "récupération de salle", ainsi que sur le bouton de l'écran "basique".
- Bien indiquer le chemin pour accéder au fichier .py dans les noeuds *pythonshell*.
- Indiquer l'adresse du serveur dans les noeuds de communication *websocket in* et *websocket out*.

## Exercices du chapitre B_1 : Twig


1. Créer par la ligne de commande un contrôleur que vous appelerez `DefaultController`
  | Questions |
  |--|
  | Quels fichiers ont-ils été crées lors de l'exécution de la commande ? |
  | Dans quels dossiers de l'application ? |
  | Quelle est la strcture de classe `DefaultController` ? |
  | Que se paasse-t-il si vous cherchez à accéder à l'URL `localhost/<chemin-de-l-application>/public/index.html/default` |

2. En modifiant le gabarit créé par Symfony, nous voulons :
  - Modifier la mise en page globale en insérant un entête (`header`) et un pied de page (`footer`)
  - Le bloc `main` devra contenir une emplacement pour une barre latérale `aside` permettant d'afficher du contenu contextuel.
  - Ajouter des ressources web dans le dossier `public`
    * Définissez dans le dossier public un sous-dossier `css` pour mettre créer une feuille de styles;
    * Ajoutez une image d'entête et quelques autres images dans un sous-dossier `images`
  - Appelez ces ressources avec la fonction Twig `asset`

3. Nous voulons afficher dans cette page une liste de noms d'écrivains avec des titres de leurs livres; les livres ont un genre (roman, essai, etc.)
  - Cette liste sera contenue dans un tableau statique de la classe de contrôleur : chaque écrivain est associé à une liste de (titres de) livres et chaque livre a un genre ;
  - Les données de chaque écrivain seront affichées dans un bloc, qui sera lui-même structuré dans un fichier séparé chargé au moment de la création de la page.

# Première page

## Feuille de route

* Explication de l'architecture MVC pour Symfony
* Installation de Twig
* Dossier des templates
* Syntaxe de Twig
  * variables
  * filtres et fonctions
  * hiérarchie de gabarits & mise en page
  * inclusion de gabarits
  * contrôleurs secondaires
  * inclusion paresseuse
  * variables globales
* Présentation du profileur

## Résumé des syntaxes

### Console

| Commande | Explication |
|----|-----|
| make:controller | Crée une classe contrôleurs |

### Twig

| Commande | Explication |
|----|-----|
| {{ v1 }} | Variable |
| {% if ... %} | Instruction ou structure de contrôle |
| {% extends 'dossier/fichier' %} | Inclusion du gabarit dans une mise en page (“layout”) |
| {% include 'dossier/fichier' %} | Inclusion d'un fragment Twig dans le gabarit |
| {# v1 #} \| Commentaire |
| {{ v1 \| upper }} | Filtre : modifie la présentation de la valeur à gauche du “pipe” |
| {{ '\<h1>titre\</h1>' \| raw }} | Insère le texte _en tant que code HTML_  et non comme une simple chaîne de caractères |
| {{ asset(file) }} | Fonction |
| {{ render(path) }} | Fonction affichant dans la page la réponse d'uneroute ou d'un contrôleur secondaire |


## Notes

### Introduction

Créer une première page  pour une application Symfony nécessite de regarder plus loin que le code des pages, c'est-à-dire de prendre en compte le cycle complet des requêtes. Dans ce chapitre, nous allons donc présenter rapidement des fondements de ce cycle, sur lequel nous reviendrons en détail upeu plus loin.

Si jamais Twig n'est pas installé dans votre application, vous poouvez l'ajouter avec la commande :
```bash
# twig est un alias connu du serveurde “recipes” de Symfony.
composer require twig
```

### MVC

Symfony se base, comme beaucoup d'applications sur l'architecture appelée « **Modèle-Vue-Contrôleur**» ou **MVC**. Cette architecture se compose de l'interaction de trois grandes composantes :

1. Le **modèle**, qui a trait a l'organisation,le stockage et la gestion des données
2. Les **vues**, qui se chargent de la présentation des données, c'est-à-dire de l'interface utilisateur
3. Le(s) **contrôleur(s)**, qui organisent le traitement des demandes de l'utilisateur (des requêtes HTTP dans le cas d'une application web).

Cette description est néanmoins lacunaire, car elle omet des pans importants de l'architecture d'une application Symfony, en particulier le **routeur** et de **conteneur de services**. Son approche est centrée sur l'utilisateur, ses actions et ses besoins de lecture.

L'avantage de cette représentation est qu'elle permet de dissocier les différentes tâches qui concourent à la construction de l'application. Certaines personnes peuvent développer les vues, avec des compétences relativement limitées en programmation ; d'autres peuvent s'occuper du modèle de données et du stockage (base de données) ; d'autres encore peuvent écrire des bibliothèques de code métier etc. Toutes ces acticvités sont clairement scindées.


### Le cycle de la requête (version courte)

Pour revenir aux sources, le web est l'ensemble des **ressources** accessibles par des adresses appelées **URI** (pour **U**niform **R**esource **I**dentifier). Ce que l'on appelle “page” et donc une approximation et constitue plutôt la *représentation* de la ressource. Ce qui prime, en conséquence, est l'URI car c'est elle qui désigne la ressource.

Le travail de l'application est donc d'interpréter l'URI est d'en donner une représentation conforme à ce qu'attend l'utilisateur (ce pourrait être du texte, une description orale, une vidéo, des données structurées, etc. Il y a potentiellement une infinité de représentations possibles). Le cycle de la requête est le processus qui mène de l'un à l'autre.

La première tâche de Symfony est de lier une adresse (que l'on appellera une **route**) à un contrôleur, qui aura pour rôle de planifier et d'organiser le traitement et l'achèvement de la requête de l'utilisateur.

Pour créer le code de base nécessaire à tout cela, il existe une commande de la console :
```bash
symfony console make:controller
```
En exécutant cette commande, nous allons créer dans le dossier `src/Controller` un fichier (et une classe, donc) qui prendront le cycle de la requête en charge.

A un niveau superficiel, nous voyons que la classe contient une méthode (par défaut appelée `index`) et qui est très simple. Remarquons deux choses :

1. Dans le bloc de commentaires (un **DocBlock**, en fait, reconnaissable à l'ouverture `/**` et non `/*`) une **annotation** `@Route` qui contient le schéma de l'URL (et son nom). Le DocBlock est **lié fonctionnellement** à la méthode qui se trouve juste au-dessous, c'est-à-dire, pour nous, au contrôleur. Si j'écris cet URL dans la barre d'adresses de mon navigateur, c'est cette méthode qui sera exécutée.
2. Derrière le `return`, un appel à ma méthode `render`, qui prend en paramètre le chemin vers un « template », à l'interieur du dossier `templates`.

Donc, par ces quelques lignes, nous avons parcouru le cycle complet !

1. L'utilisateur envoie une requête et demande à accéder à une certaine ressource (URI)
2. l'URI est associé rigidement à un contrôleur qui traite la requête
3. Après ce traitement s'ensuit la construction d'une vue (par défaut une page HTML)
4. La vue est renvoyée à l'utilisateur via l'exécution du `return`

Nous avons là un code extrêmement concis (et que l'on pourrait encore compresser).

### Construction d'une vue

Comme présenté dans la **[A_Installation]**, les “pages” de l'application sont stockées dans le dossier `templates` situé à la racine du projet.

Symfony n'impose pas de format particulier pour la description des pages et, globalement, il est possible de les écrire en PHP. Néanmoins, l'outil privilégié de construction des pages (ou vues) reste le moteur de templates `Twig`, un autre projet de Sensio Labs.

Ce choix paraît quelquefois curieux, puisque PHP est déjà, en lui-même, un moteur de templates. `Twig` a toutefois des atouts :
* C'est un outil très léger et facilement extensible
* Il offre une syntaxe indépendante du langage de programmation (on peut utiliser Twig dans Django/Python, par exemple)
* Au-delà du rendu, il offre des connexions avec les autres composants de la plate-forme, dont Doctrine, dont nous pourrons tirer parti pour simplifier grandement l'écriture de certaines partie du code.

Nous sommes maintenant prêts à écrire une “page” que afficher du contenu à l'écran. Pour cela, nous avons besoin de la syntaxe de Twig. En premier lieu, tout code HTML statique est, par construction, du code Twig.

#### Variables
Afficher une variable `v1` s'écrit (les variables ne portent pas de marque distinctive comme le `$` de PHP) :
```twig
{{ v1 }}
```
C'est l'équivalent de la syntaxe PHP :
```php
<?= $v1 ?>
```
Les variables utilisées dans un template sont listées dans la méthode `render` à l'intérieur du contrôleur, sous forme d'un tableau associatif. C'est le deuxième argument passé à la méthode, le premier étant le nom du gabarit.

#### Instruction
Les instructions de Twig, s'écrivent :
```twig
{% instruction %}
```
On trouve en particulier les instructions : extends, block, if, for, include.
```twig
{% if v1 == 0 %}
<p class="warning">Atttention, votre vrariable est nullable</p>
{% else %}
{% endif %}
```
Toutes les instructions du langage sse terminent par la déclaration `{% end<_instruction_> %}`

#### Commentaires
Onpeut insérer des commentaires partout avec cette syntaxe :
```twig
{# Ceci est un commentaire #}
```

#### Synthèse
À partir de là, il est possible de traduire en Twig preque n'importe quelle page PHP. Il est à noter que l'on n'utilise pas de tableau (au sens où on ne les *construit* pas ; on peut les explorer bien sûr). Et les objets utilisent l'opérateur `.` classique plutôt que le `->` de PHP.

Exemple :
```html
<body>
  {# Début de l'entête de la page #}
  <header>
    <h1>Liste des œuvres de {{ auteur }}</h1>
  </header>
  <main>
    <ul>
    {%for o in oeuvres %}
      <li>{{ o.titre }}<li>
    {% endfor %}
    </ul>
  </main>
</body>
```

#### Hiérarchie de templates
Une particularité de Twig est d'autoriser « l'emboîtemement » des templates les uns dans les autres avec une technique de surcharge. Pour cela, il exites deux intructions `block` et `extends` qu qgissent de concert.

Un squelette Twig peut desfinir des blocks, qui sont juste, *a minima*, des emplacements vides destinés à recevoir du contenu :
```twig
{% block sidebar %}{% endblock %}
```
Ces `blocks` définissent donc une structure de la page. Généralement, on aura un squelette (quelquefois plusieurs) qui organisera la forme commune de toutes les pages du site. Ce squellete générique contient souvent assez peu de choses par lui-même.

D'autres pages peuvent “étendre” ce squellette générique, en remplissant les blocs. Pour cela, les “pages-filles” devront :
1. Déclarer quel squelette elles étendent
2. Donner (éventuellement) un contenu pour les blocs

L'étape 1 est réalisée en écrivant l'instruction suivante sur la première ligne du fichier :
```twig
{% extends '<chemin/du/fichier/du/squelette.html.twig>' %}
```
L'étape 2 se fait en déclarant simplement des blocs portant le même nom que dans le squellette, et englobant du contenu.

il est ànoter qu'un fichier contenant une instruction `extends` ne peut contenir **que des blocs**. Aucun code HTML ou autre ne peut être laissé « libre » en dehors d'un bloc.

Définissons la structure globale des pages :
```twig
{# une mise en page générique dans un fichier 'templates/default.html.twig' #}
<html>
  <head>
    {% block stylesheets %}{% endblock %}
  </head>
  <body>
    <header>
    </header>
    <main>
      {% block main %}{% endblock %}
    </main>
  </body>
</html>
```
Nous pouvons maitenant, pour une page donnée, remplir les blocs avec du contenu :
```twig
{# La racine du chemin est toujours `templates`, donc c'est implicite #}
{% extends 'default.html.twig'%}
{# La feuille de style, dont la racine du chemin est le dossier  `public` #}
{% block stylesheets %}
<link href="{{ asset('css/styles.css') }}" rel="stylesheet" />
{% endblock %}
{% block main %}
<section>
  <h2>Titre</h2>
</section>
{% endblock %}
```
Par défaut, le contenu du gabarit “enfant” **remplace** celui du contenu “parent”. Si vous voulez agréger les deux contenus, le premier à la suite du second, vous devrez ajouter l'instruction `parent()`.
```twig
{% block main %}
{{ parent() }}
<section>
  <h2>Titre</h2>
</section>
{% endblock %}
```

#### Inclusion de template
Inversement, il est possible d'inclure un gabarit dans un autre gabarit, comme nous le ferions avec l'instructino `include` de PHP. D'ailleurs, l'instruction est la même :
```twig
{# Inclut un gabarit depuis les dossier `templates\partials` #}
{% include 'partials/header.html' %}
```
Ces fragments Twig contiiennent éventuellement des variables ; il faut donc les leur passer avec le mot-clef `with` :
```twig
{% include 'partials/header.html' with {'v1': 10, 'v2': 'Nabuchodonosor'} %}
```
**N.B.** Le nom du gabarit peut être remplacé par n'importe quelle _expression_ valide.

Il existe aussi une version asynchrone `include` : `hinclude`, présentée [ici].

#### Fonctions et filtres
La syntaxe de Twig contient aussi des fonctions, ainsi que deodificateurs appelés “filtres”. Certains sont natifs, d'autres sont spécifiques à Symfony, et nos verrons plus tard q'uk est assez simple d'introduire des extensions pour ajouter nos propres éléments de syntaxe dans le répertoire des fonctions et des filtres.

##### Fonction
Une fonction ne se distingue en rien d'une fonction “normale” :
```twig
{{ path('page_information') }}
```
La fonction `path` rend l'URI qui correspond au nom d'une route. Un peu plus haut, nous avons utilisé la fonction `asset` pour construire l'URI correspondant au fchier d'une ressource web.

##### Filtre
Les filtres, quant à eux, sont une syntaxe spécifique à Twig, marquée par le symbole `|` (souvent appelé en anglais “pipe”, parce que c'est le symbole d'enchaînement des commandes UNIX).
```twig
{{ 'hello' | upper }}
```
Dans l'exemple ci-dessus, le mot sera affiché en majuscules.

#### Contrôleurs et routes secondaires
Dans un certain nombre de cas, les applications ont besoin de faire appel à des **contrôleurs secondaires** (on distingue alors la _requête maître_ des autres requêtes) ; c'est typiquement le cas lorsque l'on a besoin d'afficher de l'information contextuelle.

Dans ce cas, il existe la fonction `render`, qui peut être utilisée de deux manières différentes :
```twig
{# avec une route, la fonction `path` construit l'URI vers laquelle envoyer une requête #}
{{ render(path('page_information')) }}

{# avec un contrôleur, la méthode est exécutée directement #}
{{ render(controller('App\\DefaultController::index', {'id': 54} )) }}
```
La fonction `path` comme la fonction `controller` peuvent nécessiter des arguments, comme dans le deuxième exemple.

La différence entre les deux options tient dans le fait que `path` redéroule tout l'ensemble du cycle de la requête HTTP, alorque que `controller` se contente d'exécuter la méthode requise. Dans le cas général, cela n'aura pas beaucoup d'importance et l'on aura tendance à privilégier la solution la plus rapide, c'est-à-dire la deuxième.

## Exercices

* Création d'un premier contrôleur par la ligne de commande
  * Ce contrôleur s'appellera `DefaultController`
* Ecriture d'une page d'accueil pour le site
  * Ajouter une page d'information (Qui sommes-nous ?)
  * Cette page devra être accessible depuis la page d'accueil via un lien dont l'URL sera construit avec la fonction `path`
* Ajout de ressources web dans le dossier `public`
  * Définissez dans le dossier public un sous-dossier `css` pour mettre créer une feuille de styles
  * Appelez cette feuille avec la fonction Twig `asset`
* Construire une page qui affiche une liste de noms d'écrivains avec des titres de leurs livres; les livres ont un genre (roman, essai, etc.)
  * Cette liste sera contenue dans un tableau statique de la classe de contrôleur
  * Les titres sont cliquables et dans ce cas, j'affiche une autre page avec :
  * le titre du livre
  * un bloc qui affiche les livres du même genre (ce bloc sera appelé dynamiquement en faisant appel à un contrôleur  et à un gabarit d'affichage séparé)

## Ressources

* [Twig](https://twig.symfony.com/)
* [Création d'une page HTML](https://symfony.com/doc/current/page_creation.html)
* [Utilistion de Twig dans Symfony](https://symfony.com/doc/current/templates.html)
* [Variables globales](https://symfony.com/doc/current/templating/global_variables.html)
* [Chargement paresseux de fragments](https://symfony.com/doc/current/templating/hinclude.html)
* [Le profileur](https://symfony.com/doc/current/profiler.html)

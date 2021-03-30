# Aide-mémoire Twig

## Console

| Commande | Explication |
|----|-----|
| make:controller | Crée une classe contrôleurs |

## Contrôleurs

### Annotations

| Annotation | Attributs | Explication |
|---|---|---|
| @Route | ('/slug', name='<nom-de-la-route>')  | L'annotation `@Route` définit le schéma de l'uRL qui déclencje le contrôleur |
| @Template | ('templates/<nom-du-template>')  | L'annotation `@Template` définit le gabarit d'affichage de la ressource consultée. Elle évite de faire explicitement appel à la méthode `render` |

### Attributs (PHP 8)

| Annotation | Explication |
|---|---|
| #[Route('/slug', name='<nom-de-la-route>')]  | Nouvelle syntaxe native de PHP 8 remplaçant les annotations |

### Rendu

| Fonction | Arguments | Explication |
|---|---|---|
| render | string, array)  | Construit la vue envoyée en réponse àla requête |


## Twig

### Syntaxe de base

| Syntaxe | Explication |
|----|-----|
| {{ v1 }} | Variable |
| {% if ... %} ... {% elseif ... %} ... {% else ... %} ... {% endif %} | Conditionnelle |
| {% for variable in tableau %} ... {% endfor %} | Instruction ou structure de contrôle |
| {% extends 'dossier/fichier' %} | Inclusion du gabarit dans une mise en page (“layout”) |
| {% include 'dossier/fichier' %} | Inclusion d'un fragment Twig dans le gabarit |
| {# v1 #} \| Commentaire |
| {{ v1 \| upper }} | Filtre : modifie la présentation de la valeur à gauche du “pipe” |
| {{ '\<h1>titre\</h1>' \| raw }} | Insère le texte _en tant que code HTML_  et non comme une simple chaîne de caractères |

> **N.B.** Il est aussi possible d'injecter des variables globales, voire des services avec la notation {{ ... }}

### Exemples de fonctions Twig

| Syntaxe | Explication |
|----|-----|
| {{ asset(file) }} | Fonction |

### Extensions pour Symfony

| Syntaxe | Explication |
|----|-----|
| {{ asset(file) }} | Fonction |
| {{ render(path) }} | Fonction affichant dans la page la réponse d'une route ou d'un contrôleur secondaire |
| {{ render(controller) }} | Fonction affichant dans la page la réponse d'une route ou d'un contrôleur secondaire |
| {{ render_hinclude(controller) }} | Fonction affichant dans la page la réponse d'une route ou d'un contrôleur secondaire |

### La notation pointée de Twig

La notation `foo.bar` peut avoir plusieurs sens, qui sont recherchés dans l‘ordre suivant :

| Syntaxe | Explication |
|----|-----|
| $foo['bar'] | clef de tableau associatif |
| $foo->bar | propriété d‘objet |
| $foo->bar() | méthode d‘objet |
| $foo->getBar() | accesseur de la propriété `bar` |
| $foo->isBar() | méthode d‘essence selon la propriété `bar` |
| $foo->hasBar() | méthode de possession de la propriété `bar` |

### La variable globle `app`

| Variable | Explication | Renvoi |
|---|---|---|
| app.user | L'utilisateur (connecté) actuel | Sécurité |
| app.request | L'objet de la classe `Request` représentant le requête HTTP | Contrôleurs |
| app.session | L'objet de la classe `Session` contenant les données de la session courante | Contrôleurs |
| app.flashes | Un tableau contenant l'ensemblde des « messages Flash » | Twig |
| app.environment | Le nom de l'environnement courant (dev, prod, etc) | Configuration |
| app.debug | Valeur booléenne en fonction de l'actiovation di mode debug | Configuration |
| app.token | L'objet de la classe `TokenInterface` contenant le jeton de sécurité CSRF | Sécurité |

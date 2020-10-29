# Installation

## Feuille de route

* Les pré-requis de Symfony 5
  * PHP 7.4
  * Composer
  * La commande `symfony`
  * L'environnement système
    * curl
    * git
    * vim
* Politique de versionnage de Symfony
* Installation de Symfony Flex
  * (A voir)
  * Introduction à la notion de `recipe` (recette)
* Découverte de la structure des applications
* Affichage de la page par défaut
* Introduction à la ligne de commande

## Notes

### Pré-requis

#### PHP
Pour utiliser `Symfony 5`, vous devrez avoir installé PHP 7.2.

#### Composer
Vous devez aussi installer `Composer`qui est le gestionnaire de packages de PHP et que vous pouvez téléchharger que le site https://get-composer.org.

Composer est essentiel dans les applications PHP, quelles qu'elles soient, car il permet deux choses :
* l'installation (et la maintenance) des composants tiers;
* la prise en charge des stratégies d'auto-chargement des classes dans PHP.

Vous avez intérêt à l'installer dans un répertoire `système`(type `/usr/local/bin/` sous UNIX) pour pouvoir l'invoquer partout dans votre système de fichier. Sinon, vous serez obligés d'installer Composer pour chaque application.

La procédure est très simple :
```bash
# Récupération de l'installeur
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# Test pour vérifier que le fichier téléchargé n'est pas corrompu (grâce à un hash-code du fichier)
php -r "if (hash_file('sha384', 'composer-setup.php') === 'c31c1e292ad7be5f49291169c0ac8f683499edddcfd4e42232982d0fd193004208a58ff6f353fde0012d35fdd72bc394') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
# Exécution de l'installeur
# L'option --install-dir est utilisée pour déplacer le fichier exécutable `composer.phar` vers un dossier cible
# L'option --filename est utilise pour renommer `composer.phar`en `composer`plus simple à invoquer
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
# Suppression du fichier de l'installaeur
php -r "unlink('composer-setup.php');"
```

`Composer` permet d'exécuter un certain nombre de commandes dont les plus courantes sont :

* `create-project` : initialise un noveau projet, en ajoutant notamment des fichiers comme `composer.json` et les ressources d'auto-chargement des classes ;
* `require` : installe un package depuis un dépôt
* `install` : installe les packages listés dans le fichier `composer.json`
* `update` : fait les mises à jour si le fichier composer.json a été changé, en rapport avec le fichier `composer .lock`

Comme tous les gestionnaires de packages, l'intérêt de Composer est d'abord d'installer *en chaîne* les dépendances et dont de garantir que rien ne manque pour faire fonctionner une ressource.

Le dépôt principal (de référence) pour PHP est [Packagist](https://packagist.org), qui contient une foule de ressources (et sur lequel on puet naturellement aussi publier). Néanmoins `composer install` permet de charger des bibliothèques depuis n'importe quelle source, en particulier des dépôts publics Github.

#### Symfony
En parallèle à Composer, Symfony propose son propre outil `symfony` qui n'a toutefois pas la même vocation.

D'une part, il est centré sur Symfony, là où Composer est généraliste. Et il vise plutôt à la création et au déploiement d'environnement d'applications. Par exemple

* `new` : crée une nouvelle application (comme `composer create-project` mais avec des options différentes)
* `init` : crée un projet d'après un squelette défini sur un dépôt distant
* `deploy` : dépploie une application sur un serveur
* `console` : exécute une commande dans la console de Symfony

La commande `symfony` avait été dépréciée par Sensio Labs pendant un temps, mais elle est revenue et est maintenant documentée

#### Outils système

Pour écrire une application Symfony, il vous sera recommandé d'utiliser `git` (et éventuellement Github ou un autre système de partage).

En termes d'éditeur de code, les deux outils de prédilestion sont bien sûr `PHPStorm` et `VSCode`. Mais `Atom` peut éventuellement faire l'affaire (mais pas `Notepad++`). Et `vim` ou `emacs` restent des possibilités intéressantes.

Enfin, avoir une connaissance de base des commandes du shell UNIX est naturellement un gain de temps.

### Installation

Une fois la commande d'installation exécutée, Symfony peut vous demander quelques précisions. Celles-ci ont peu d'importances et peuvent être facilement modifiées par la suite dans les fichiers de configuraton.

Symfony peut s'installer sous deux modes différents :

* En tant que plate-forme de développement de site web
* En tant que noyau pour le développement d'applications spécifiques

Dans le premier cas,le code global du “framework” est installé avec ses quelques 30+ composants. La plupart sont utiles lorsque l'onveut développer des applications web courantes. Mais, évidemment, cela a un coût en termes d'espace disque  et de temps de calcul lors de l'initialisation de l'application. L'avantage est d'avoir un outil prêt à l'emploi.

Pour installer cette version, on utilisera :
```bash
# Composer
composer create-project symfony/website-skeleton <directory>
# Symfony
symfony new --full <directory>
```

Dans le second cas, Symfony n'installe que le strict minimum. Ce sera au développeut d'installer un par un les packages dont il a besoin... ou de d'écrire lui-même le code manquant.

Pour installer cette version, on utilisera :
```bash
# Composer
composer create-project symfony/skeleton <directory>
# Symfony
symfony new <directory>
```
#### Flex
La deuxième procédure est encouragée par Symfony depuis le passage à Symfony Flex, lors de la transition en Symfony 3 et Symfony 4. Flex apporte des amélioration essentielles dans la gestion des packages, lors de leur installation... et de leur suppression. Dans les versions antérieures, les développeurs étaient souvent amenés à faire des modifications manuelles dans le code ou les fchiers de configuration, et on courait toujours le risque de ne pas avoir complètement fait les modifications nécessaires.

Flex introduit la notion de `recipe` (recette) qui est associée au package et donne toutes les indications pour l'installation et la suppression des packages. Les “recipes” sont gérées par Symfony, même si l'on peut écrire les siennes propres (elles auront alors tout de même du mal à être diffusées). Onpeut en trouver la liste sur le site de [Flex](https://flex.symfony.com/).

Les recettes, garantissant la facilité d'emploi des packages, permettent donc de partir du “bare metal” et d'ajouter progressivment toutes les ressouces requises sans avoir à craindre de ne pas pouvoir les désinstaller. C'est donc une question de choix pour l'équipe de développement.

Ceci nous amène à considérer la structure même de la plate-forme Symfony, qui est en fait l'agrégation de composants indépendants les uns de autres, mais reliés entre eux par le jeu de l'injection de dépendance. Cela fait que c'est un outil très souple, dans lequel chaque brique peut être remplaée par une autre, **sous la seule condition que cette dernière respecte les mêmes spécifications**.

Nous développerons plus loin ces composants.

#### Structure des applications.
Quelle est la structure d'une application Symfony une fois installée ?

1. `vendor`

Le dossier `vendor` contient la base de code nécesaire au fonctionnement du framework. C'est-à-dire :
* le noyau de Symfony
* les composants
* les bibliothèques tierces (i.e. d'autres éditeurs que Symfony ou vous-mêmes)

C'est dans ce dossier que sont installés les packages demancés via Composer.

Il est évidemment fortement déconseillé de modifier le code présent à l'intérieur de ce dossier.

2. `var`

Ce dossier sert à maintenir les *caches* et les *logs* de l'application. En soi, ce n'est pas essentiel et son contenu peut être régulièrement effacé pour gagner de la place ou éviter les erreurs de cache. Ilsera recréé automatiquement.

3. `bin`

Le dossier `bin` contient des scripts PHP reconnus par la commande `symfony`, mais exécutables indépendamment.

Par défaut, il contient deux scripts :
* `console` : la console de Symfony
* `phpunit` : l'outil de tests unitaires de PHP

4. `config`

C'est le dossier de configuration, notamment des services.

5. `src`

Dans ce dossier, on mettra lecode *de* l'application.

Dans les versions récentes, Symfony a abandonné la logique de décomposition des applications en “**bundles**”. Ces derniers n'ont pas disparu, mais la logique est maintenant de considérer qu'une application est *un* “bundle” et que les autres, sibesoin est, sont des bibliothèques qui doivent, en tant que telles, être dans le dossier `vendor`. Cette philosophie n'est toutefois pas absolue. C'est plutôt ne recommandation des développeurs de la plate-forme.

En tant que “bundle”,le dossier `src` a lui-même une structure très codifiée. On y trouvera les dossiers suivants :
* Controller
* Entity
* Repository
* DepencyInjection
* ainsi qu'un fichier Kernel.php

Tous les éléments de cette structure seront approfondis plus tard.

6. `templates`

Le dossier `templates` contient tous les squelettes de mise en page qui seront utilisés par Twig (notamment) pour construire les pages HTML des vues. L'organisation de ce dossier est arbitraire, ainsi que les noms des fichiers. La seule règle est dans l'utilisation des suffixes :
* `.html.twig` pour les fichiers au format Twig
* `.html.php` pour les fichiers PHP
* etc.

Par défaut, il existe dans ce dossier un fichier `base.html.twig` qui est utilié paR Symfony lorsque rien n'a encore été développé.

7. `public`

Le dossier `public` est le seul qui doit être ouvert sur l'extérieur. C'est la racine de l'espace public du site.

Par défaut il contiendra le “contrôleur principal” de l'application, c'est-à-dire une Façade qui est le point d'entrée unique de l'application. Dans certains cas, on peut envisager plusieurs points d'accès. Le dossier ne devrait, en tout étatde cause, ne contenir aurcun autre code.

Ce qu'on y trouvera par ailleurs seront toutes les ressources web de l'application : feuilles de style, scripts , images, etc. Comme nous le verrons par la suite, on ne doit pas les y mettre “manuellement”, mais les importer grâce à une commade de la console (cf. `assets:install`).

8. Fichiers spéciaux

En plus de ces dossiers, on trouvera au moins un fichier `.env` qui contient les constantes d'environnement de l'application, qui font en fait partie des “super-globales” de PHP et qui sont accessibles comme telles.

Et pour finir les fichiers `composer.json` et `composer.lock` qui recensent les bibliothèques de l'application.


### Post-installation

Une fois la plate-forme installée, elle est immédiatement opérationnelle. On peut d'ailleurs consulter la page d'accueil dans un navigateur en exécutant le `index.php` du dossier `public`.

Cette page est spécifique (c'est indiqué en entête de la page) et nous ne la voyons *que* parce que rien n'a été défini pour le moment dans l'application. Il est donc inutile de chercher le code source de la page dans les dossier `src`, par exemple.

Au bas de la page, s'affiche un bandeau d'informations que l'on appelle le “profileur”. Il donne énormément d'informations sur l'exécution de la requête. Nous le voyons parce que nous sommes dans un “environnement” appelé `dev` pour développement.

Un “environnement” constituele cadre dans lequel l'application s'exécute. Il en exist trois par défaut :
* `dev` active toutes les fonctionnalités “développeur”, notamment les bibliothèques installés dans cet environnemen ;
* `prod` active les fonctionnalités du site lorsqu'il est déployé en “production”. Dans ce cas, les fonctionnalités `dev` sont inhibées ;
* `test` qui est un environnement activé automatiquement lors de l'exécution de procédures des tests autimatisés.


### Introduction à la ligne de commande

La console est un outil essentiel pour le développeur. Beaucoup d'opérations se font via le terminal qui accélèrent et facilitent le travail d'écriture. Le commandes sont accessibles depuis la racine de l'application via les instructions :
```sh
# symfony
symfony console
# shell
bin/console
```
Comme on peut le constater, les commandes sont *groupées* et dépendent généralement d'une bibliothèque particulière (comme `Doctrine`, par exemple ou le composant `Maker`).

Le  nom de chaque commande est divisé en deux ou trois segments divisés par le caractère ':'. Le premier segment représente le groupe ; le dernier représente l'action.

Chaque commande dispose d'une aide que l'on peut consulter avec l'option `--help`. Exemple :
```sh
symfony console make:entity --help
```

## Exercices

* Installation de Composer
* Installation de la commande `symfony`
* Installation de l'environnement de développement
* Affichage de la pagepar défaut de la plate-forme
* Inspecter les aides des commandes de la console avec l'option `--help`

## Ressources

* [Composer](https://get-composer.org)
* [La commande `symfony`](https://symfony.com/download)
* [Documentation Setup Symfony](https://symfony.com/doc/current/setup.html)
* [L'architecture en `bundle`](https://symfony.com/doc/current/bundles.html)
* [Les composants de Symfony](https://symfony.com/doc/current/components/index.html)

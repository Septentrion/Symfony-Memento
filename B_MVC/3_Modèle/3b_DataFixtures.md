# Utiliser des données factices

## Feuille de route

## Résumé des syntaxes

### Packages

| Package | Recipe | Explication |
|----|---|-----|
| **orm-fixtures** | * | Le bundle DataFixtures de Doctrine |
| **fzaninotto/faker** |   | Le générateur de données Faker de F. Zaninotto (obsolescent) |
| **fakerphp/faker** |   | FakerPHP, un fork de Faker maintenu |
| **hautelook/alice-bundle** |   | Alice, un autre générateur, une surcouche de FakerPHP |

### Commandes

| Commande | Explication |
|----|-----|
| **doctrine:fixtures:load** | Engengrement des données factices avec le bundle DataFixtures de Doctrine |
| **hautelook:fixtures:load** | Engengrement des données factices avec le bundle Alice |

### Fonctions

#### DataFixtures

| Fonction | Signature | Explication |
|----|----|---|
| **load** | (ObjectManager) | Fonction de la classe de fixtures qui définit les données à créer|
| **getDependencies** | () | Etablit les dépendances entre entités pour l'ordonnancement de la création de fixtures |
| **getGroups** | () | Définit de groupes pour segmenter la création de fixtures |


## Notes

### Introduction

Une fois le modèle du domaine de l'application établi, il est souvent utile de pouvoir travailler sur un jeu de données de manière à pouvoir écrire des requêtes, construire des vues, rôder des formulaires, créer des classes de test, etc. Autrement dit, constituer un jeu de données, même temporaire, est une étape importante du cycle de développement. Cela étant dit, écrire ces données manuellement est un travail long, fastidieux, et qui de surcroît peut devoir être recommencé plusieurs fois (e.g. en cas de modification du modèle du domaine).

Pour cette raison, il existe une bibliothèque de Doctrine qui permet d'engendre automatiquement autant de données que nécessaire en respectant le modèle, il s'agit de **DataFixtures**.

> **N.B.** : Si vous utilisez la version “_skeleton_” de Symfony et que vous devez installer la bibliothèque, il est recommandé de l'installer dans l'environnement de développement avec l'option `--dev` :
```bash
 composer require orm-fixtures --dev
```

### La bibliothèque DataFixtures

#### La classe de données factices

L'utilisation de Datafixtures est très simple puisqu'il suffit de créer des classes dans un sous-dossier `Datafixtures` du dossier applicatif `src`.

Cette classe s'écrit _a minima_ ainsi :
```php
namespace App\DataFixtures;

use Doctrine\Bundle\FixturesBundle\Fixture;

class AppFixtures extends Fixture
{ /* ... */ }
```
Cette classe hérite de la class `Fixture` de Doctrine.

Cette classe est très simple puisqu'elle n'exige qu'une seule méthode : `load`. Celle-ci dépend juste de l'`ObjectManager` de Doctrine :
```php
public function load(ObjectManager $manager);
```

La classe peut ainsi être réécrite de la manière suivante :
```php
namespace App\DataFixtures;

use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load (ObjectManager $om)
    {
        /* ... */
    }
}
```
La dépendance est ajoutée directement par Symfony grâce à l'**injection de dépendances**.

Le corps de la fonction `load` peut maintenant créer des entités d'une certane classe, assigner aux propriétés des vaeurs arbitraires et, grâce à l'ObjectManager` persister et enregistrer dans la base les entités nouvelles.

#### Lancer la création

Une fois les classes créées, il suffit de lancer le processus avec la commande dela console :
```bash
symfony console doctrine:fixtures:load
```

> **N.B.** : Par défaut (il existe d'autres manières de faire) le lancement de la commande efface toutes les données antérieurement contenue dans la base de données.

#### Variantes dans le processus

##### Dépendances entre entités

Dans la plupart des cas, le modèle du domaine comprends des associations entre entités et, pour la commodité du processus, il sera souhaitable d'ordonner la création des entités. Par exemple, si un `Author` écrit plusieurs `Book`, il serait préférable de créer d'abord des livres puis de les associer aux auteurs.

Dans ce cas, nous pouvons recourir à la méthode `getDependencies`, fournie par l'interface `Doctrine\Common\DataFixtures\DependentFixtureInterface`.
```php
// dans la classe AuthorFixtures
public function getDependencies()
{
    return [
        BookFixtures::class
    ]
}
```
La méthode `load` de `AuthorFixtures` aura la garantie que des entités `Book` existent et que l'on peut les lier à des `Author`.

##### Groupes de fixtures

Dans un certain nombre de cas, nous voudrons seulmement créer des données sur une sous-partie du domaine. Dans ce cas, il est possible de définir des groupes et de circonscrire le processus aux entités d'un groupe.

Pour cela `Datafixtures` disposie d'une méthode statique `getGroups` fournie par l'interface `Doctrine\Bundle\FixturesBundle\FixtureGroupInterface`.
```php
public static function getGroups() : array
{
    return [
        'groupDocuments',
        'groupUsers'
    ]
}

Il est maintenant possible de lancer le processus avec un groupe en option (voire plusieurs) :

```bash
symfony console doctrine:fixtures:load --group=groupDocuments
```

### Enrichir la description et la pertinence des données

`DataFixtures` fait l'essentiel du travail. Malheureusement, la seule méthode disponible est de créer des valeurs aléatoirement ou bien d'écrire des fonctions qui produiront des valeurs selon nos souhaits. Mais heureusement, des développeurs se sont déjà penchés sur la question et des outils existent pour faire exactement cela.

#### Faker

La bibliothèque utilisée taditionnellement est celle de François Zaninotto : Faker. Pour l'installer, il suffit d'exécutre la commande :
```bash
composer require fzaninotto/faker --dev
```

Faker fournit des “_Provider_” dont le rôle est de créer différents types de données, en fonction de paramètres particuliers (le pays, par exemple, si l'on cherche des villes ou des numéros de téléphone).

La manière la plus simple d'utiliser `Faker` (dabns Symfony) est en complément de `DataFixtures`. Par exemple :
```php
namespace App\DataFixtures;

// import de la fabrique (factory) de Faker pour engendrer les valeurs de différents Provider
use Faker\Factory as FakerFactory

use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load (ObjectManager $om)
    {
        // En passant par la _fabrique_, l'injection de dépendance ne peut pas être utilisée
        $faker = FakerFactory::create();
        $doc = new Book();
        // La date de publicaton est contrainte à être entre hier et ily a cent jours
        $doc->setPublishedAt($this->faker->dateTimeBetween('-100 days', '-1 days'));
        /* ... */
    }
}
```

La factory admet un paramètre de langue, nous permettant aindi de récupérer des données “localisées” :
```php
$faker = FakerFactory::create('en_US');
```
Toutes les données ne sont pas concernées ; le nombre et le type des données localisées dépend fortement des langues. Le rôle de cette fabrique est d'instancier un objet de la classe `Faker\Generator` (rien à voiravec les générateurs de PHP) qui fera concrètement le travail.

Comme vous pourrez le constater, les noms des propriétés ne sont jamais ambiguës. C'est ce qui permet au `Generator` de reconnaître les propriétés (comme `setPublishedAt`) dans l'exemple précédent.

Je vous renvoie à la documentation de `Faker` pour expérimenter toutes les options des `Provider` et de leurs différentes propriétés.

Si une donnée vous manque (ou une langue), il et très simple de créer son propre `Provider`. Il s'agit d'une simple classe PHP qui doit hériter de la classe `Faker\Base` ou une de ses sous-classes. Il faut juste implémenter les méthodes qui produiront les “bonnes” données. Pour que cette classe soit comprise par Faker, vous pourrez l'ajouter au `Generator` :

```php
$faker = FakerFactory::create('fr_FR');
// Noter au passage que les _Provider_ implémentent le design pattern `Visiteur`
$faker->addProvider(new App\Faker\Provider\fr_FR\MyProvider($faker))
```
Chaque classe doit naturellement être référée à une langue particulière.

On pourra par exemple créer une classe pour les immatriculations de voitures françaises :
```php
namespace App\Faker\Provider\fr_FR;

use Faker\Provider\Base;

class Immatriculation extends Base
{
    public function carImmatriculation()
    {
        return /* ... */;
    }
}
```

Le seul défaut de `Faker` est son développement a été maintenant abandonné par F. Zaninotto. La bibliothèque va être considérée comme obsolète par Symfony et nous allons dvoir trouver une alternatice pérenne.

#### FakerPHP

Il eiste un “fork” de `Faker`, maintenu en particulier par Andreas Möller et qui s'appelle `FakerPHP`. Les deux bibliothèques adoptent exacement la même logique. C'est la solution la plus évidente pouyr qui a déjà un peu l'habitude de travailler avec `Faker`. L'installation est quasiment identique :
```bash
composer require fakerphp/faker --dev
```


#### Alice

Dans un autre genre, `Alice` est une bibliothèque maintenue par Nelmio, un collectif auquel on doit quelques sources largement utilisées dans Symfony comme `NelmioApiDocBundle` (générateur de documentation technique). Avec Symfony, la solutin la plus simple est d'intaller le bundle `AliceBundle`, qui est une surcouche d'Alice :

Installation :
```bash
composer require --dev hautelook/alice-bundle
```

`Alice` repose sur des fichiers de description. C'est donc plutôt une meilleure méthode que les précédentes. Les fichiers de description peuvent être écrits dans plusieurs formats. Si nous prenons l'exemple de YAML, privilégié par Symfony, cela donne par exemple :
```yaml
App\Entity\Book:
    NotreDameDeParis:
        genre: Roman
        dateDePublication: 15/03/1831
        auteur: Victor Hugo
        pages: 940
        langue: fr
        editeur: Charles Gosselin
```
que l'on écrira dans un fichier `books.yaml` dans le dossier `fixtures` à la racine de l'application.

L'exemple précédent permet de créer _une_ entité dont le nom est NotreDameDeParis (en réalité peu importe), une instance de la classe `Book` et à laquelle nous avons assigné des valeurs particulières.

Si nous voulons créer une entité réellement factice et aléatoire, nous pouvons demander à `Alice` de s'appuyer sur `FakerPHP`. Ce qui donnerait :
```yaml
App\Entity\Book:
    NotreDameDeParis:
        genre: <genre()>
        dateDePublication:  <dateBetween('-100 year', '-1 year')>
        auteur: <name()>
        pages: <numberBetween(100, 900)>
        langue: fr_FR
        editeur: <name()>
```
Les chevrons ('<...>') indiquent qu'une fonction doit être exécutée pour engendrer la valeur et les fonctions en question sont en fait celles des `Provider` de `FakerPHP`. Dans notre exemple, nous avons introduit un `genre`, qui n'est pas connu de `FakerPHP`. Dans ce cas, nous devrions créer une classe correspondante définissnt la méthode en question, ainsi que nous l'avons vu précédemment pour `Faker`.

Si nous voulons maintenant produire une _série_ de livres, ilnous faut un peu changer l'étiquette :
```yaml
App\Entity\Book:
    book{1..100}:
        genre: <genre()>
        dateDePublication:  <dateBetween('-100 year', '-1 year')>
        auteur: <name()>
        pages: <numberBetween(100, 900)>
        langue: fr_FR
        editeur: <name()>
```
Ici, nous demandons à `Alice` de créer cent livres dont les étiquettes iront de `book1` à `book100`.

Ceci étant fait, il ne reste qu'à exécuter une commande de la console pour lancer l'engendrement des données :
```bash
symfony console hautelook:fixtures:load
```

`Alice` peut faire encore bien d'autres choses. Entre autres, il est possible :

* de gérer des propriétés optionnelles en définissant une probabilité d'existence suivie du caractère '`?`'.
```yaml
App\Entity\Book:
    book{1..100}:
        pages: 80%? <numberBetween(100, 900)>
```

* de gérer les relations entre entités en indiquant le nom de l'entité associée; dans l'exemple ci-dessous, nous choisissons 1) une constante (toujours le même auteur) ; 2) un auteur au hasard ; et 3) un auteur au hasard dans un intervalle restreint
```yaml
App\Antity\Author:
    author{1..20}:
        name: <name()>

App\Entity\Book:
    book{1..10}:
        auteur: '@author1'
    book{11..20}:
        auteur: '@author*'
    book{21..30}:
        auteur: '@author<numberBetween(5, 10)>'
```

Tous les détails sont dans les documentations  d'`Alice` et `AliceBundle`.

## Ressources

[DataFixtures dans Symfony](https://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html)
[Le code source de DataFixtures](https://github.com/doctrine/data-fixtures)
[Faker — François Zaninitto](https://github.com/fzaninotto/Faker)
[FakerPHP — Anfreas Möller](https://github.com/FakerPHP)
[Alice](https://github.com/nelmio/alice)
[Hautelook AliceBundle](https://github.com/hautelook/AliceBundle)

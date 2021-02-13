# Les entités et le modèle du domaine

## Feuille de route

## Résumé des syntaxes

### Commandes

| Commande | Explication |
|----|-----|
| make:entity | Création d'un entité |
| doctrine:create:database | Création une base de données |
| doctrine:schema:create | Création le schéma d'une base de données |
| doctrine:schema:update | Mise à jour le schéma d'une base de données |
| make:migration | Création une migration |
| doctrine:migration:migrate | Mise à jour le schéma d'une base de données via une migration |

### Annotations

| Annotation | Explication |
|----|-----|
| @ORM\Entity | Attributs de la classe d'entité |
| @ORM\OneToOne | Association 1-1 |
| @ORM\ManyToOne | Association N-1 |
| @ORM\OneToMany | Association 1-N (réciproque de la précédente) |
| @ORM\ManyToMany | Association N-N |
| @ORM\Column | Type de la propriété |
| @ORM\InheritanceType | Stratégie d'implentation des hiérarchies de classes en SQL  |
| @ORM\DiscriminatorColumn | Colonne SQL correspondant au type de l'entité  |
| @ORM\DiscriminatorMap | Liste des entités comprises dans la hiérarchies  |

## Notes

Les entités décrivent le modèle du domaine de l'application. Elles constituent le premier volet de la gestion du modèle par l'ORM (**O**bject **R**elational **M**apper ) Doctrine.

Dans la pratique ce sont juste des classes PHP dont le but est de permettre l'accès aux propriétés. Leur rôle est essentiellement déclaratif (ou descriptif) et on n'y implémentera jamais de code autre que la manipulation et la transformation strictes des propriétés.

Une entité reprend donc les grandes lignes de la programmation objet, et qui y ajoute des caractéristiques spécifiques à l'échange des données entre l'application et la base de données. Doctrine se servira en effet des déclarations dans les entités pour convertir les objets en leur équivalent relationnel et inversement.

Il est aussi à noter que Symfony s'appuie sur les entités pour créer et analyser les formulaires HTML, qui seront étudiés plus tard.

On retrouve dans la définition des entités beaucoup des attributs qui sont définis dans le modèle Entité/Association ainsi que **UML**.

Les entités **doivent** être stockées dans le dossier `src/Entity`, là où Symfony s'attend à les trouver.

### Création des entités

Pour créer une entité, le moyen le plus simple est d'utiliser la commande du composant Maker :

```bash
symfony console make:entity
```

Cette commande engage un dialogue dans lequel on vous demandera :
1. Le nom de l'entité
2. Le type est les caractéristiques des propriétés de l'entité
3. Des options (en fonction du type) qui serviront principalement à la création du schéma relationnel.

Selon la logique du modèle Entité/Association les propriétés des entités (ou concepts) sont décrites alternativement comme :
* des attributs, qui sont globalement des valeurs constantes, le plus souvent scalaires ;
* des associations, qui définissent les relations entre entités, et qui peuvent être de plusieurs types (1 - 1 1 - n ou n - n)
* des généralisations, qui permettent d'organiser les entités en hiérarchies, de manière analogue au modèle de la programmation objet.

#### Les attributs

Les attributs manipulés par Doctrine ne recouvrent pas exactement ceux de SQL. Par exemple, toutes les variantes des types d'entiers n'ont pas d'équivalents dans Doctrine..

On trouvera tous les types les plus simples, comme :
* `string`, `ascii_string` : équivalent au `VARCHAR` de SQL
* `text` : tous les types textes
* `integer`, `smallint`, `bigint` : tous les entiers
* `float` : tous les nombres à virgule flottante
* `decimal` : équivalent SQL
* `date`, `time`, `datetime` : équivalents à SQL
* `date_immutable`, `time_immutable`, `datetime_immutable` : variantes des précédents pour des valeurs non-mutables
* `datetimetz` : une date compte tenu du fuseau horaire
* `guid`: équivant à un `VARCHAR` de 32 caractères, généralement utilisé comme alternative aux clefs primaires auto-incrémentées (les GUID/UUID sont assurés d'être uniques)

Comme dans SQL, on peut définir des types binaires, qui sont toutefois très peu utilisés, voire considérés comme une mauvaise pratique :
* `blob`, `binary` : permettent de stocker dans la base de données des valeurs binaires arbitraires, comme des fichiers par exemple ;
* `object` : une propriété contenant un objet

De plus Doctrine reconnaît des données structurées, qui peuvent être stockées en tant aue telles dans la base de données :
* `array`, `simple_array`
* `json`, `json_array`

L'utilisation de champs JSON dans une base SQL soit toujours être décidée avec précaution. En effet, cela contrevient à la première forme normale (1FN) de SQL qui spécifie qu'une colonne doit touours contenir des données _atomiques_.

Doctrine se charge naturellement des la correspondance entre les types définis dans les entités et ceux inscrits dans la base de données. Si l'on utilise le processus “automatisé”, Doctrine choisira pour les tables SQL des types qui semblent lui convenir mais, dans les imites de la compatibilité, il est tout à fait possible de modifier la description de ces tables.

#### Les associations

Conformément au modèle Entité/Association, on peut définir trois types d'associations en fonction de leur cardinalité :
* `OneToOne` : spécifie une association dans laquelle chaque entité ne peut être liée qu'à une seule autre entité.
  * C'est une situation qui n'est pas très courante mais qui peut servir à definir des dépendances optionnelles (par exemple une Personne est liée à une entité Adresse mais celle-ci n'est pas obligatoire, on économise alors de l'espace disque) soit données temporaires (une Personne est liée à un Panier le temps de son achat en ligne — et le Panier n'appartient qu'à une sele Personne) ;
  * En SQL, on implémentera fréquemment cette association par le biais d'une clef étrangère entre les deux clefs primaires des deux tables liées, à moins de vouloir tracer un historique des associations
* `OneToMany`, `ManyToOne` : caractérise une association asymétrique dans laquelle une des entités est liée à plusieurs autres entités.
  * Le cas le plus emblématique est la relation Mère-Enfant(s), un Mère peut avoir plusieurs Enfant(s) (voire aucun) mais un Enfant ne peut avoir qu'une seule Mère. De mème un Disque contiendra plusieurs Plages, mais une Plage n'appartiendra qu'à un seul Disque.
  * En SQL, on ajoutera dans la table du côté N une colonne définissant une clef étrangère vers le coté 1.
  * On notera que, en UML, il existe deux manières de représenter les associations : les _agrégations_ et les _compositions_. S'il n'existe dans la pratique pas de différence essentielle entre les deux, on considérera que :
    * l'agrégation est une assoociation “optionnelle”, qui n'entre pas dnas l'essence de la description d'un objet (par exemple, une personne peut acheter des objets mais ces derniers ne définissent pas la personne) ; à ce titre l'impplémentation devrait plutôtprévoir une cardinalité `0..*`
    * la composition est une association qui ne peut pas être absente au risque de dénaturer l'objet (par exemple un disqueest défini par l'ensemble de ses plages, sans quoi il lui manque une partie intrinsèque de lui-même)
* `ManyToMany` : indique une association dans laquelle les entités peuvent être liées à un nombre arbitraire d'autres entités.
  * Typiquement, les réseaux sociaux sont de cette nature : toute personne est lié à plusieurs autres personnes, qui elles-mêmes sont liées à plusieurs personnes, etc. Dans un autre ordre une personne peut emprunter un livre dans une bibiothèque et ce livre sera emprunté succsivemment pas plusieurs personnes.
  * En SQL, on ne peut pas représenter directement ce type d'association ; on la décompose alors en deux associations 1-N tête-bêche autravers d'une table de jointure (ou de liaison)
  * En UML, l'aquivalent est la classe d'association ; celles-ci sont très fraquemment utilisées pour décrire les entités de Doctrine.

Une notion importante à comprendre est que les associations de Doctrine ne sont jamais strictement symétriques. Pour des raisons internes, Doctrine détermine toujours un _sens directeur_ et un _sens inverse_. Cette distinction a des effets non négligeables lors du traitement des données par l'application (notamment au travers des formulaires) et nous en verrons des exemples par la suite. Il est important de comprendre qu'un choix **doit** être fait, lors de la conception de l'application, qu'il est relativement définitif, et qu'il peut rendre le code plus ou moins synthétique. En première approxiamtion, on dira que les deux entités dans une association n'ont pas le même _poids_ et que le sens directeur doit aller de l'ntité la plus importante vers l'entité la plus annexe. A titre d'exemple, si l'on définit un disque pas l'ensemble de ses plages, on “sent” qu'il est légitime de définir le Disque comme l'entité la plus importante et la Plage comme l'entité annexe.

Dans le processus de la commande `make:entity`, la direction de des associations est définie implicitement par l'entité que vous êtes en train de créer.  C'est toujours d'elle que partiront les sens directeurs des associations que vous définirez pour les propriétés.

##### Comment identifier le sens directeur ?

Dans le code PHP de l'association (en admettant que l'on ait utilisé `make:entity`), on trouve une annotation telle que :
```php
/**
 * Identifiant du disque auquel appartient cette plage
 * @ORM\ManyToOne(target="CD", inversedBy="tracks")
 * @ORM\Column(type="integer")
 */
private $disqueID;
```
Cette annotation nous indique que la propriété `disqueID` est la référence d'une association de type `ManyToOne` vers l'entité `CD`. L'attribut `inversedBy` indique que `disqueID` est le point de départ du sens directeur de l'association. De l'autre côté, nous trouvons :
```php
/**
 * Liste des plages contenues dans ce CD
 * @ORM\OneToMany(target="Track", mappedBy="disqueID")
 */
private $tracks;
```
Le sens inverse de l'association est marqué par l'attribut `mappedBy`.

Nous voyons ici que les associations sont purement **déclaratives**. Il est tout à fait possible de revenir manuellement sur la structure de ces associations... à condition d'être précis dans les modifications !

Le mécanisme est identique pour les autres formes d'associations. Néanmoins, dans le cas du couple `ManyToOne/OneToMany`, le sens directeur est _toujours_, obligatoirement, le sens `ManyToOne`.

#### Les généralisations

Il est tout à fait possible de répliquer avec Doctrine des hiérarchies d'entités comme on le fait dans la programmation objet (même si cela est souvent — à juste titre — considéré comme une pratique douteuse).

Créer une hiérarchie n'est pas possible via une commande de Symfony. Cela doit nécessairement être fait manuellement. Comment faire ?
Admettons que nous voulions créer des `Document` qui seraient soit des `Livre`, soit des `DVD`, soit des `Journaux`, etc.

1. Dans un premier temps, créer les entités (on supposera qu'il existe un diagramme de classes UML qui les spécifie). Conformément à l'héritage de la POO, les entités ne contiennent que les propriétés qui leur sont spécifiques.
2. Dans la classe mère (ici `Document`), il faudra naturellement changer l'accessibilité de propriétés, qui sont `private` par défaut lorsque l'on utilise `make:entiy`
3. Dans les classes filles, on peut supprimer toute référence à `id`, qui sera héritée. Ceci n'est néanmoins moins pas indispensable, cela rend le code plus clair.
4. Maintenant vient la déclaration dun lien hiérarchique, ce qui se fait dans la classe mère :
```php
/**
 * [Document description]
 * @ORM\InheritanceType("SINGLE_TABLE")
 * @ORM\DiscriminatorColumn(name="documentType", type="integer")
 * @ORM\DiscriminatorMap("DOO" = "Document", "L01" = "Livre", "D02" = "DVD", "J03" = "Journal")
 */
class Document {
  /* ... */
}
```
* L'annotation `DiscriminatorColumn` indique qu'une colonne sera ajoutée dans la table SQL pour dire quel est le type de document de l'enregistrement considéré ;
* L'annotation `DiscriminatorMap` contient la liste de toutes valeurs possibles de cette colonne, c'est-à-dire toutes les sous-classes répertoriées de la classe mère ;
  * Nous remarquons que chaque classe est associé à une étiquette arbitraire
  * Nous remaquons aussi que la classe mère est elle-même incluse dans la liste
* Enfin, l'annotation `InheritanceType` indique comment cette hiérarchie sera implémentée dans la base SQL. Ici nous avons dux possibilités :
  * `SINGLE_TABLE` : Toutes les propriétés des différentes classes sont fusionnées dans une seule table ; l'avantage est de ne pas avoir à faire e jointure pour reconsituer un objet, mais l'on peut potentiellement avoir beaucoup de cellules vides
  * `JOINED` : Chaque entité correpond à une table et la reconstitution des objets se fait par le biais de clefs étrangères ; les avantages et inconvéients sont inversés, comme on s'en doute.

#### Remarques finales

L'avantage de la commande `make:entity` est qu'elle peut être exécutée autant de fois qu'il est nécessaire sur la même entité, pour ajouter de nouvelles propriétés. Cela sera souvnet très utile, évidemment, lorsque l'on souhaitera créer des associations... les entités liées devant déjà être connues au préalable.

En revanche, les cas de suppression de propriétés ne peuvent être réalisés que manuellement.

On remarquera que les propriétés ne sont pas correctement réordonnées après modificatino de l'entité ; on sera souvent amenés à le faire manuellement.

Enfin, l'amoncellement de méthodes `set...` et `get...` pourrait être remplacé par les méthodes magiques PHP `__set` et `__get`. Nous voyons à ce propos qu'une des limites de Doctrine est de nécessiter des accesseurs et de mutateurs pour toutes les propriétés, ce qui n'est pas vraiment dans l'esprit de SOLID et réintroduit le problème des variables globales que la programmation objet cherchait à éliminer.

Lorsque Symfony crée l'entité, il est indiqué de _deux_ classes sont engendrées :
* L'entité proprement dite (dans le dossier `src/Entity`)
* Une classe dite `Repository` (dans le dossier `src/Repository`)

Nous retrouvons celle-ci dans une annotation en entête de la classe d'entité :
```php
/**
 * @ORM\Entity(repositoryClass=NewsletterRepository::class)
 */
class Newsletter {
  /* ... */
}
```

Cette classe contiendra par la suite toutes les méthodes représentant des requêtes sur la base de données.


### Création du schéma de la base de données

Une fois les entités créées, nous pouvons demander à Doctrine d'engendrer le schéma de la base de données.

(**N.B.** : on peut aussi faire l'inverse, c'est-à-dire reconstituer les classes de l'application en fonction d'une base existante)

1. En premier lieu, il faut définir l'adresse de la base données. On la trouve dans le fichier de configuration `.env` qui est à la racine de l'installation
```bash
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=5.7
```
Symfony vous propose plusieurs exemples de définition pour MySQL, PostgreSQL ou sqlite.
2. Ensuite vous devez créer la base de données. Pour cela il existe deux méthodes (au moins) :
  * soit utiliser phpMyAdmin pour créer manuellement la base
  * soit utiliser la commande `symfony console doctrine:database:create`, qui fait la même chose en se basant sur la déclaration du ficher `.env` (on peut ajouter l'option `--if-not-exists`)
  ```bash
  symfony console doctrine:database:create --if-not-exists
  ```


### Gestion du schéma de la base

Pour créer/modifier les schéma de la base de données, il existe deux méthodes.

#### Méthode historique

Une fois la base créée, la première exécution est `doctrine:schema:create`. Cett commande, comme son nom l'indique, crée le schéma entier, sans tenir compte de ce qui existait déjà (potentiellement) dans la base de données. Ce commande peut être exécuté “à blanc” avec l'option `--dump-sql`.
```bash
symfony console doctrine:schema:create --dump-sql
```

Pour chaque modification des entités de l'application, vous pourrez maintenant exécuter la commande `doctrine:schema:update`. Celle-ci vous demandera _explicitement_ confirmation du mode d'exécution, soit “à blanc” (`--dump-sql`), soit effectivement (`--force`).
```bash
symfony console doctrine:schema:create --force
```

#### Méthode des migrations

Les versions récentes de Symfony intègrent un dispositif beaucoup plus perfectionné, les **migrations**.

La méthode historique à en effet une limite, c'est l'impossibilité de revenir en arrière. Toute modification du schéma ert définitive. Les migrations résolvent ceci avec une gestion de versions qui permet de conserver la trace de toutes les modifications successives.

A chaque étape de création/modification des entités, il est possible d'associer une classe de migration, par la commande `make:migration`.
```bash
symfony console make:migration
```
Cette commande produit une nouvelle classedans un dossier nommé `migrations`. Nous pouvons constater qu'une classe de migration contient principalement deux méthodes : `up` et `down`. Et chacune de ces méthodes sontient une liste de commandes SQL de modification du schéma de la base.

Comme on peut s'y attendre `up` permet de passer à la version suivante et `down` de revenir à la version précedente.

Exécuter une migration — c'est-à-dire globalement mettre à jour le schéma de la base — se fait généralement avec la commande `doctrine:migrations:migrate`.
```bash
symfony console doctrine:migrations:migrate
```
Par défaut, si vous modifiez la desctiption de vos entités (avec `make:entity`, par exemple) et que vous créez une nouvelle classe de migration, `doctrine:migrations:migrate` devrait restaurer la correspondance entre les classes des entités et les tables de la base de données.

Les migrations peuvent être employées de manière plus avancée (cf. mémento ad hoc)


## Exercices

* Diagrammes UML pour les claases de l'application
* Construction du modèle depuis la ligne de commande
* Construction des entités
  * Commande `make:entity`
  * Syntaxe des associations
* Entités hiérarchiques
  * Entêtes Doctrine `@ORM\DiscriminatorMap`
  * Modification du statut des clefs primaires -> protected
* Création d'une migration initiale

## Ressources

* [Mapping Objet-Relationel](https://www.wikiwand.com/fr/Mapping_objet-relationnel)
* [L'appariement hiérarchique de Doctrine](https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/inheritance-mapping.html#query-the-type)

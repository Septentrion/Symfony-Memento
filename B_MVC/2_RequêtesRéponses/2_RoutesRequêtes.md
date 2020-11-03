# Routes et requêtes

## Feuille de route

* Notion de route
  * Techniques de Symfony pour les routes
    * Fichier externe (YAML, JSON)
    * Annotations
* Syntaxe des routes
* Notion de contrôleur
  * Passage des paramètres de requête
* Notions de requête et de réponse
  * Présentation de la classe Request
  * Présentation de la classe Response (et des sous-classes)
* Utilisation des Annotations
  * Templating
  * ParameterConversion

## Résumé des syntaxes


## Notes

### Introduction

Le cycle de la requête dans la logique de l'architecture **MVC** commence, de manière sommaire, par l'exécution d'un **contrôleur**. Le contrôleur est la pièce centrale du dispositif. Il ne fait véritablement rien par lui-même mais se contente d'orchestrer le travail des diférents agents qui coopéreront pour délivrer à l'utilisateur l'information qu'il demande.

Le contrôleur,de manière encore une fois superficielle, requiert le **modèle** et, lorsque celui-ci a terminé, déclénche l'envoi d'une **réponse**, qui est, dans le cas général, une **vue**, c'est-à-dire — pour ce qui nous concerne — une page HTML ou un fragment de page HTML.

Globalement, cela donne le code suivant ;
```php
/**
 * @Route("/default", name="default")
 */
public function index(): Response
{
    return $this->render('default/index.html.twig', [
        /*
         *  Le tableau est envoyé à Twig pour affichage.
         *  Dans Twig, la variable porter le nom de la clef dans le tableau (authors)
         */
        'authors' => self::AUTHORS,
    ]);
}
```

Dans le **chapitre B_1**, nous avons déjà utilisé un contrôleur et nous voyons qu'il s'agit d'une méthode dans une classe de contrôleurs. Quelle méthode activer en fonction de lademande de l'utilisateur ? C'est ici qu'il faut introduire un nouveau composant dans l'architecture globale :le **routeur**, dont le rôle sera d'analyser la requête et de sélectionner un contrôleur.

**N.B.** : Nous voyons au passage que, déjà, l'architecture MVC est un peu insuffisante pour représenter la totalitéde l'application. C'est en premier lieu parce que MVC a été conçu pour les interfaces graphiques (et les vues multiples comme dans les applications de bureau).

### Notion de route

Dans l'univers des applications web, les routes constituent l'**API** ce l'application. C'est par ce moyen que l'utilisateur (qui peut tout à fait être une autre application) accède aux fonctionnalités de celle-là.

Comme nous sommes sur le web, une route prend la forme d'un **URI** (_voire d'un IRI_), telle que définie dans la [RFC 3986](https://tools.ietf.org/pdf/rfc3986.pdf) :
```
<schema>://<authority>/<path>?<query>#<fragment>
```
Dans Symfony, un URI s'écrira, par défaut, si le domaine pointe bien vers le dossier `public` :
```
https://<domain>/<route>?<query>#<fragment>
```
ou sinon, dans le cas général :
```
https://<adesse_IP>/<path>/public/index.php/<route>?<query>#<fragment>
```
**N.B.** : Dans les applications web, la forme des URI est essentielle et est travaillée pour définir une “politique d'URI”. Mais nous n'en discuterons pas ici.

Nous voyons que la route est la partie “interne” de l'URI. Elle peut, a priori, prendre une forme arbitraire, équivalent à chemin dans un système de fichiers, c'est-à-dire divisé en segments dontle séparateur est le caractère `/`.


#### Route et contrôleur dans Symfony

Dans Symfony, une route est liée à un et un seul contrôleur. Cette liaison peut être déclarée d'au moins deux manières différentes.

Comme nous le voyons dans l'exemple ci-dessus, la manière privilégiée par Symfony (comme très souvent) est d'utiliser des **annotations**. Ainsi les routes sont-elles liées quasi-syntaxiquement aux méthodes qu'elles vont déclencher. Cette annotation s'écrit :
```
@Route("/<schema>", name="<label>")
```
Les deux caractéristiques essentielles d'une route sont :
* son `schéma`, c'est-à-dire la structure de la route (e.g. `/user/profile`, `/book/25/update`, etc.)
* son `label`, qui est le nom par lequel l'application manipulera les routes (on n'écrit jamais _directement_ les URI dans le applications Symfony)

En appliquant cette méthode, les annotations doivent être contenues dans les `DocBlocks` qui **précèdent immédiatement** la déclaration de la function.

##### Déclarer les routes dans un fichier de configuration

Il existe une autre méthode pour déclarer les routes, c'est d'utiliser un fichier de configuration.

Commeexpliquer dansle **chapitre A**, il existe un dossier `config` qui contient tous les éléments de conguration des différents composants de l'application. A la racine, noustrouvonsun fichier `routes.yaml`. Par défaut, celui-ci contient une seule définition, commentée de surcroît :
```yaml
#index:
#    path: /
#    controller: App\Controller\DefaultController::index
```
On y retrouve :
* le **label**, ici `index`
* le **schéma**, ici la route racine vers la page d'accueil de l'application
* Naturellement, il faut préciser le nom du contrôleur sous la forme : <nom-qualifie-de-la-classe>::<methode>

La déclaration des routes peut également être paramétrée dans le fichier `config/routes/annotations.yaml`, où l'on voit que Symfony est averti que des routes peuvent exister sous formes d'annotations pour les classes du dossier `src/Controller`.

(_disclaimer_ : c'est plutôt celle que je recommanderais car elle découple le système des routes de son implémentation)

#### Attributs des routes

En dehors des attributs nécessaires (schéma et label), les routes admettent toute une série d'attributs optionnels qui enrichissent leur puissance.

##### Variables

Un schéma de route peut être composé de segments variables. Ceux-ci sont notés entre accolades.
```php
/**
 * @Route("/author/{id}", name="author_detail")
 */
public function index(int $id): Response
{
    return $this->render('default/index.html.twig', [
        /*
         *  Le tableau est envoyé à Twig pour affichage.
         *  Dans Twig, la variable porter le nom de la clef dans le tableau (authors)
         */
        'author' => self::AUTHORS[$id],
    ]);
}
```
Comme nous le voyons dans cet exemple, la variable `id` présente dans la route est introduite dans la signature de la fonction pous forme de variable PHP `$id`. Symfony fera automatiquement l'appariement en tes deux.

Version YAML :
```yaml
index:
    path: /author/{id}
    controller: App\Controller\DefaultController::index
```
Dès lors, il faudra e méfier car **plusieurs** routes peuvent correspondre à un même URI. Comme par exemple :
```
@Route("/author/{id}", name="author_detail")
@Route("/author/new", name="author_new")
```
Cette ambiguité pourra être résolue de différentes façons, comme nous allons le voir, mais cela nous mène à considérer comment Symfony règle ce cas en dehors de toute information complémentaire.

Les roues sont examinées par Symfony dans l'ordre où elles sont déclarées, c'est-à-dire au seind'une même classe dans l'ordre des lignes du fichier, et entre les différentes classes par ordre alphabétique. Le routeur de Symfony s'arrête dès qu'il trouve une route compatible avec l'URI de la requête. D'où il ressort que si l'on veut éviter les mauvaises interprétations, il faudra veiller à ordonner les routes de manière logique, en particulier des plus contraintes au plus génériques.

##### Méthodes HTTP

Il est possible de différencierles routes en fonction des méthodes HTTP auxquelles elles peuvent répondre. PHP et Apache sont relaivemnet limités de ce point de vue :
* PHP ne traite que GET et POST
* Apache ne considère que les méthodes liées à du contenu statique et ignore les autres.

Or il es recommandé d'exploiter les méthodes selon leur sémantique précise, telle que définie dans la [RFC 2616](https://tools.ietf.org/html/rfc2616) :

| Méthode | Usage |
|---|---|
| GET | Lit une représentation de la ressource |
| HEAD | Recueille les méta-données associées àune ressource (GET mais sons le corps de la réponse) |
| OPTIONS | Permet au client d'obtenir des informations et des contraintes associées àune ressource |
| PUT | Remplace une entité du serveur avec les données contenues dans le corps du message. Equivaut à un effacement puis une re-création |
| PATCH | Contrairement à PUT, autorise une modification partielle de l'entité• Equivaut à une mise à jour |
| POST | Vise à ajouter de l'information à une entité du serveur ; est souvent considérée comme la création d'une entité |
| DELETE | Efface une entité du serveur |
| TRACE | Equivaut à un “ping”, c'est-à-dire un écho ; peut être utilisé pour faire des tests |
| CONNECT | réservé pour se connecter via un proxy ou un tunnel (e.g. SSL)

Nous verrons plus tard comment indiquer à Symfony une méthode particulière. Mais du côté du routeur, il suffit d'ajouter un attribut à la définition de la route :
```
@Route("/author/{id}", name="author_detail", methods={"GET","HEAD"})
```
ou en YAML :
```yaml
index:
    path: /author/{id}
    controller: App\Controller\DefaultController::index
    methods: GET|HEAD
```
Une route peut arbitrairement reconnaître une ou plusieurs méthodes.

Un avantage de ceci est qu'il est possible d'utliser le même schéma d'URI pour plusieurs actions, même si ceci n'est pas toujours pratique dans un navigateur web.

##### Contraintes
Une autre manière de discriminer les routes est d'imposer des contraintes sur les variables. ceci se fait avec lattribut `requirements`, qui supporte des expressions régulières. Admettons que la variable `id` soit nécessairement un nombre, nous pourrions ajouter ceci :
```
@Route("/author/{id}", name="author_detail", methods={"GET","HEAD"}, requirements={"id"="[0-9]+"})
```
ou :
```yaml
index:
    path: /author/{id}
    controller: App\Controller\DefaultController::index
    methods: GET|HEAD
    requirements:
        id: "[0-9]"+
```
Grâce à cela, nous pouvons maintenant différencier les traitements à apporter à un message non seulement en fonction de sa forme ou de sa destination, mais aussi directement selon son contenu.

##### Valeurs par défaut

Il est possible d'associer une valeur par défaut à une variable et ceci peut se faire de deuw manière différentes. Imaginons que j'affiche la liste des auteurs par page (e.g. `authors/5` — avec `authors` au pluriel) et que je veuille autoriser la route `authors` affichaat la première page, je pourrais définir ma route ainsi  :
```
@Route("/authors/{page}", name="author_list", methods={"GET","HEAD"}, requirements={"page"="[0-9]+"}, defaults={"page"="1"} )
```
ou :
```yaml
index:
    path: /author/{id}
    controller: App\Controller\DefaultController::index
    methods: GET|HEAD
    requirements:
        id: "[0-9]"+
    defaults:
        page:1
```
Mais je pourrais tout aussi bien utiliser les ressources de PHP :
```php
/**
 * @Route("/authors/{page}", name="author_list", methods={"GET","HEAD"})
 */
public function list(int $page = 1) { /* ... */}
```

##### Variables spéciales

Pour terminer, il existe une série de varibles pré-définies, reconnaissables par le préfixe `_`.

* `_format` : la valeur de cette variable influe sur le type de laréponse HTTP qui sera envoyée; elle est donc utile pour faire facilement de la négociation de contenu par exemple. Une route `/authors/{page}/{_format}` istanciée en `/authors/5/json` permettra à Symfony l'envoi d'une réponse dutype `application/json`,par exemple ;
* `_locale` : cette variable permet d'infiquer à Symfony la langue d'affichage de l'application ; nous en reparlerons dans la partie consacrée à la traduction. Typiquement, nous voudrons construire des routes comme : `{_locale}/authors/{page}` -> `/fr/authors/5`.Dans ce cas, la variable `_locale` doit être au format international des langues ;
* `_fragment`: identifie de le “fragment” dans l'URI, c'est-à-dire la partie optionnelle après le caractère `#`.


#### Routes et requêtes

Le routeur apour rôle d'analyser la requête et de produire un objet de la classe `Symfony\Component\HttpFoundation\Request`, une classe du composant `HTTPFoundation`. Cet objet est en fait un “proxy” de la variable super-globale `$_REQUEST` à qui elle offre une API mieux intégrée à la plate-forme.

##### Conversion de paramètres

#### Construire les URI correspondant aux routes

#### Lister les routes

Si vous voulez connaitre les routes actives, il existe une commande pour cela :
```bash
symfony console debug:router
```
Il est également à noter que ces informations se retrouvent dans le profileur.

## Ressources

* [Routage de Symfony](https://symfony.com/doc/current/routing.html)
* [Les contôleurs](https://symfony.com/doc/current/controller.html)
* [Le composant `HTTPFoundation`](https://symfony.com/doc/current/components/http_foundation.html)
* [Les annotations de SensioFrameworkExtrabundle](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html)

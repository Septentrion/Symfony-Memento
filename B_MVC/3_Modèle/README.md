# Les modèles de domaine avec Doctrine — Partie 1

Cette partie couvre la gestion des modèles dans les aplications Symfony, c'est-à-dire tout ce qui concerne les **données** :
* La prise en compte explicite d'un modèle conceptuel de données
* le stockage des données
* la persistance
* les échanges par le biais de requêtes

Pour cela, Symfony a recours à — par défaut — recours à un ORM : **Doctrine**. L'étude decet ORM constitue donc le cœur de cette partie.

## Table des matières

1. Les entités, modèle de données de Symfony
  * Création des entités
  * Les attributs simples
  * Les associations entre entités
  * Les généralisations et les hiérarchies d'entités
  * Création du schéma d'une base de données relationnelle
  * Notion de migration
2. Utiliser de données factices en phase de développement : les “**fixtures**”
  * Créer une classe génératrice de données factices
  * Lancer le processus d'engendrement
  * Des outils pour obtenir des données “crédibles”
3. Les classes de requêtes ou “**repositories**”
  * Les méthodes de recherche natives de Doctrine
  * Ecrire des requêtes dans un formalisme “psudo-PDO”
4. DQL, le langage de requêtes de Doctrine
  * Les clauses similaires à SQL
  * Les clauses spécifiques à DQL
  * Les fonctions incluses nativement dans DQL
  * Le cas des entités hiérarchiques
5. Le “**QueryBuilder**”, le générateur de requêtes
  * Fonctionnement de base du QueryBuilder
  * L'API de haut niveau
  * La classe “Expr”
6. **Native SQL**, comment mixer Doctrine et SQL
7. Les requêtes de modification de l'état de la base
8. Les extensions à Doctrine
  * Ajouter des fonctions manquandes à DQL
  * Ajouter de nouveaux comportements aux entités


## A suivre

D'autres aspects avancés de Doctrine sont traités ultérieurement.

# LINO-PIMO

Qu'est-ce que LINO-PIMO ? <br>

Des outils en ligne de commande :

* Conçus pour *créer*, *générer*, *gérer* et *anonymiser* les jeux de tests.
* Compatible avec une démarche *DevOps*.
* En adéquation avec la *philosophie Unix*.

**LINO** est un ETL spécialisé dans la gestion de jeux de données de tests. <br>
LINO extrait d'une base de données relationnelle un ensemble de données, liées par des contraintes de clés étrangères, sous la forme d'un flux *JSON*. <br>
LINO permet également le chargement d'un flux *JSON* dans une base de données existante.

**PIMO** est un outil d'anonymisation de données. <br>
Un flux de données au format *JSON* passé en entrée, peut être transformé en un autre flux dans lequel les données sensibles ont étés masquées par des valeurs aléatoires. <br>
Son puissant moteur de masquage permet de paramétrer finement quoi anonymiser dans le flux, et surtout comment grâce au fichier `masking.yml`. <br>
L'outil est également capable de générer de la volumétrie à partir d'une seule ligne d'exemple.

Ces outils peuvent s'exécuter sur tous types d'environnements (*Linux*, *Windows*, *MacOs*) et peuvent être manipulées facilement par un opérateur, un script ou un outil de CI/CD.

## Aperçu Rapide

### LINO

![PULL](img/PULL.gif)

![PUSH](img/PUSH.gif)

### PIMO

![PIMO](img/PIMO.gif)

### Cas d'Usage

* **Echantillonnage** : Fabriquer un jeu de données réduit avec des données cohérentes entre elles.
* **Pseudo-Anonymisation** : Remplacer les données personnelles en préservant le format.
* **Comparaison** : Aligner les données pour identifier les différences, trouver les suppressions.
* **Automatisation des tests** : Données calibrées, suppression des données à la fin du test.
* **Reproduction d'un BUG** : Rechercher une donnée, extraire toutes les entités nécessaires et recharger dans une base de test.
* **Synthèse** : Générer des données.
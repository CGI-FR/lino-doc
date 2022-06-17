# LINO-PIMO-SIGO

## Qu'est-ce que LINO-PIMO ?

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

###  Aperçu Rapide

#### LINO

![PULL](img/PULL.gif)

![PUSH](img/PUSH.gif)

#### PIMO

![PIMO](img/PIMO.gif)

### Cas d'Usage

* **Echantillonnage** : Fabriquer un jeu de données réduit avec des données cohérentes entre elles.
* **Pseudo-Anonymisation** : Remplacer les données personnelles en préservant le format.
* **Comparaison** : Aligner les données pour identifier les différences, trouver les suppressions.
* **Automatisation des tests** : Données calibrées, suppression des données à la fin du test.
* **Reproduction d'un BUG** : Rechercher une donnée, extraire toutes les entités nécessaires et recharger dans une base de test.
* **Synthèse** : Générer des données.

## Qu'est-ce que SIGO ?

* Conçus pour *anonymiser* des jeux de données.
* Compatible avec une démarche *DevOps*.
* En adéquation avec la *philosophie Unix*.

**SIGO** est un outil d'anonymisation de données respectant les principes du k-anonymat et de la l-diversité. <br>
Un jeu de données au format *JSON* passé en entrée, peut-être transformé en un autre jeu de données également au format *JSON* dans lequel les données auront été anonymisées en utilisant différentes méthode comme par exemple l'aggrégation ou l'ajout de bruit. <br>
L'objectif de cet outil est d'empêcher la réidentification des individus tout en préservant l'utilité des données.


###  Aperçu Rapide

#### SIGO

![PIMO](img/SIGO.gif)

### Cas d'Usage:

* **Publication**: Publier des données dans l'Open Data contenant des données sensibles. Pour être conforme au RGPD les données doivent être préalablement anonymisées.
* **Utilisation d'IA**: Les Data Scientist peuvent entraîner des modèles de machine learning sur des jeux de données contenant des données sensibles. Pour limiter les risques, il est possible d'anonymiser les données.
* **Analyse**: De même concernant les personnes qui analysent des données contenant des donées sensibles.
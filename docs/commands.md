# Commandes

LINO-PIMO est un ETL (**E**xtract **T**ransform **L**oad) simple pour gérer les données de test. Cette page liste les commandes qui peuvent être utilisées avec `lino` et `pimo`. 

1. [LINO](#lino)
2. [PIMO](#pimo)
3. [SIGO](#pimo)

## LINO <a name = "lino"></a>

`lino` extrait des données d'une base de données relationnelles pour en créer un échantillon et permet également de recharger des données.

### help

**Description :**
    Aide pour les commandes.

> ```console
> $ lino --help
>```

- **Utilisation :**
    `lino [commande]`
<br>
    - *`[commande]` disponible :*
        - `completion`
        - `dataconnector`
        - `http`
        - `id`
        - `pull`
        - `push`
        - `relation`
        - `table`

Utilisez `lino [command] --help` pour plus d'informations sur une commande.

### completion

**Description :**
    Génère le script d'autocomplétion pour le shell spécifié.

- **Utilisation :**
    `lino completion [commande]`
<br>
    - *`[commande]` disponible :*
        - `bash <option>`: génère le script d'autocompletion pour bash.
        - `fish <option>`: génère le script d'autocompletion pour fish.
        - `powershell <option>`: génère le script d'autocompletion pour powershell.
        - `zsh <option>`: génère le script d'autocompletion pour zsh.

> **Exemples :** <br>

Générer le script d'autocompletion pour bash :
```console
$ lino completion bash > /etc/bash_completion.d/lino
```
Génèrer le script d'autocompletion pour powershell en désactivant les descriptions de complétion :
```console
$ lino completion powershell --no-descriptions
```

Utilisez `lino completion [command] --help` pour plus d'informations sur une commande.

### dataconnector

**Description :**
    Gère les alias de la base de données.

- **Utilisation :**
    `lino dataconnector [commande]`
<br>
    - *`[commande]` disponible :*
        - `add <alias_bdd> <url_bdd> <option>`: ajoute un alias à une base de données.
        - `list`: liste les alias de base de données.
        - `ping <alias_bdd>`: connexion à la base de données `<alias_bdd>`.

**Alias :** `dataconnector`, `dc`

> **Exemples :** <br>

Ajouter un alias à une base de données en indiquant que la base de donnée est en lecture seule.
```console
$ lino dataconnector add mydatabase 'postgresql://postgres:sakila@localhost:5432/postgres?sslmode=disable' -r
```
Lister les alias.
```console
$ lino dc list
```

Utilisez `lino dataconnector [command] --help` pour plus d'informations sur une commande.

### http

**Description :**
    Démarre le serveur HTTP.

- **Utilisation :**
    `lino http <option>`   

> **Exemples :** <br>

Associer un port http (par défaut le port est 8000).
```console
$ lino http --port 8080
```
Préciser le nom du fichier contenant l'ingress-descriptor.
```console
$ lino http -i IngressDescriptor.yaml
```

### id

**Description :**
    Gère l'ingress-descriptor (descripteur d'entrée, ie. le chemin par lequel passe `lino` pour récupérer les données).

- **Utilisation :**
    `lino id [commande]`
<br>
    - *`[commande]` disponible :*
        - `create <start_table>`: créé l'ingress-descriptor à partir de la table `<start_table>`.
        - `display-plan`: montre les étapes par lesquelles la récupération des données se fait.
        - `export`: exporte le contenu de l'ingress-descriptor au format *JSON* vers la sortie stdout.
        - `set-child-lookup <relation> [true|false]`: définit le paramètre `lookup`* de la table enfant de la `<relation>` à true ou false dans l'ingress-descriptor.
        - `set-parent-lookup <relation> [true|false]`: définit le paramètre `lookup`* de la table parent de la `<relation>` à true ou false dans l'ingress-descriptor.
        - `set-start-table <start_table>`: définit la table de départ `<start_table>` dans l'ingress-descriptor
        - `show-graph`: montre le graphe de l'ingress-descriptor.

*`lookup`: indique si l'on veut ou non récupérer la donnée de la table concernée par cette `<relation>`.

> **Exemples :** <br>

Afficher le chemin que parcourt `lino` pour récupérer les données.
```console
$ lino id display-plan
```
Changer le paramètre `lookup` de la table enfant de la relation `fk_pets_owner_id`.
```console
$ lino id set-child-lookup fk_pets_owner_id true
```

Utilisez `lino id [command] --help` pour plus d'informations sur une commande.

### pull

**Description :**
    Extrait des données d'une base de données.

- **Utilisation :**
    `lino pull <alias_bdd> <option>`
<br>

> **Exemples :** <br>

Extraire les données uniques de la table `owners`.
```console
$ lino pull mydatabase --distinct --table owners
```
Extraire les grappes de données correspondant à la clause `where SQL`. 
```console
$ lino pull mydatabase -w "id in (12,24,36)"
```

### push

**Description :**
    Charge des données dans une base de données (mode par défault: *insertion*)

- **Utilisation :**
    `lino push [mode] <alias_bdd> <option>`
<br>
    - *`[mode]` disponible :*
        - `truncate`: permet de supprimer toutes les données d’une base sans supprimer la base de données.
        - `insert`: permet d'inserer une ou plusieurs lignes dans la base existante.
        - `update`: permet d’effectuer des modifications sur des lignes existantes.
        - `delete`: permet de supprimer des lignes dans une base.

> **Exemples :** <br>

Inserer dans la table `owners` un fichier contenant les lignes à inserer.
```console
$ lino push mydatabase --table owners < data.jsonl
```
Mettre à jour des données dans la base en supprimant les contraintes durant le chargement.
```console
$ < data.jsonl | lino push update mydatabase --disable-constraints
```

### relation

**Description :**
    Gère les relations de la base de données.

- **Utilisation :**
    `lino relation [commande]`
<br>
    - *`[commande]` disponible :*
        - `extract <alias_bdd>`: extrait les relations de la base de données.

**Alias :** `relation`, `rel`

> **Exemple :** <br>

Extraire les relations de la base de données `mydatabase`.
```console
$ lino relation extract mydatabase
```

Utilisez `lino relation [command] --help` pour plus d'informations sur une commande.

### sequence

**Description :**
    Gère les séquences.

- **Utilisation :**
    `lino sequence [commande]`
<br>
    - *`[commande]` disponible :*
        - `extract <alias_bdd>`: extrait le nom de la séquence de la base de données.
        - `status <alias_bdd>`: retourne le statut de la séquence de la base de données.
        - `update <alias_bdd>`: mets à jour le statut de la séquence.

**Alias :** `sequence`, `seq`

> **Exemples :** <br>

Extraire le nom de la séquence de `mydatabase`.
```console
$ lino sequence extract mydatabase
```
Connaître le status de la séquence de `mydatabase`.
```console
$ lino sequence status mydatabase
```

Utilisez `lino sequence [command] --help` pour plus d'informations sur une commande.

### table

**Description :**
    Gère les tables de la base de données.

- **Utilisation :**
    `lino table [commande]`
<br>
    - *`[commande]` disponible :*
        - `add-column <alias_table> <alias_colonne> <option>`: ajoute une définition de colonne dans le fichier `table.yml`.
        - `count <alias_bdd>`: compte le nombre de lignes présent dans les tables de la base de données.
        - `extract <alias_bdd>`: extrait les métadonnées des tables de la base de données.
        - `remove-column <alias_table> <alias_colonne>`: supprime une définition de colonne dans le fichier `table.yml`.

**Alias :** `table`, `tab`

> **Exemples :** <br>

Ajouter une colonne au fichier `table.yml` en précisant le type de la colonnes.
```console
$ lino table add-column owners first_name -i string
```
Retourner le nombre de lignes présent dans la base de données `mydatabase`.
```console
$ lino table count mydatabase
```

Utilisez `lino table [command] --help` pour plus d'informations sur une commande.

## PIMO <a name = "pimo"></a>

`pimo` masque des données sensibles contenues dans un flux de jsonlines en utilisant un fichier de configuration contenant les masques à appliquer.

### help

**Description :**
    Aide pour les commandes.

> ```console
> $ pimo --help
>```

- **Utilisation :**
    `pimo [commande]`
    `pimo <option>`
<br>
    - *`[commande]` disponible :*
        - `completion`
        - `flow`
        - `jsonschema`
        - `play`

Utilisez `pimo [command] --help` pour plus d'informations sur une commande.

### completion

**Description :**
    Génère le script d'autocomplétion pour `pimo` pour le shell spécifié.

- **Utilisation :**
    `pimo completion [commande]`
<br>
    - *`[commande]` disponible :*
        - `bash <option>`: génère le script d'autocompletion pour bash.
        - `fish <option>`: génère le script d'autocompletion pour fish.
        - `powershell <option>`: génère le script d'autocompletion pour powershell.
        - `zsh <option>`: génère le script d'autocompletion pour zsh.

> **Exemples :** <br>

Générer le script d'autocompletion pour bash :
```console
$ pimo completion bash > /etc/bash_completion.d/pimo
```
Génèrer le script d'autocompletion pour bash (sur macOS) en désactivant les descriptions de complétion :
```console
$ pimo completion bash --no-descriptions > /usr/local/etc/bash_completion.d/pimo
```

Utilisez `pimo completion [command] --help` pour plus d'informations sur une commande.

### flow

**Description :**
    Génère un diagramme Mermaid permettant de visualiser le processus de transformation de la donnée.

- **Utilisation :**
    `pimo flow <option>`

> **Exemple :** <br>

Générer le diagramme du fichier de configuration `masking.yml`
```console
$ pimo flow masking.yml > masking.mmd
```

### jsonchema

**Description :**
    Génère le schéma json du fichier de configuration de `pimo`.  

- **Utilisation :**
    `pimo jsonchema <option>`

> **Exemple :** <br>

Générer le schéma json.
```console
$ pimo jsonchema
```


### play

[A venir]

## SIGO <a name = "sigo"></a>

`sigo` anonymise un jeu de données en repectant le k-anonymat et la l-diversité. Il est possible de configurer l'anonymisation en ligne de commande ou à l'aide d'un fichier de configuration `yml`.

### help

**Description :**
    Aide pour les commandes.

> ```console
> $ sigo --help
>```

- **Utilisation :**
    `sigo <option>`
<br>
    - *`<option>` disponible :*
        - `--k-value,-k <value>`
        - `--l-value,-l <value>`
        - `--quasi-identifier,-q <list>`
        - `--sensitive,-s <list>`
        - `--anonymizer,-a <method>`
        - `--cluster-info,-i <name_of_field>`
        - `--entropy [true|false]`
        - `--configuration, -c <file>`


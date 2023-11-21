## Parsing XML dans PIMO

Depuis la version `1.20.0` de PIMO, il est possible de traiter des fichiers XML.

PIMO conserve l'indentation et les commentaires du fichier, mais applique des modifications aux balises ciblées.

Voici un exemple de fichier XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<taxes>
    <account location="New York">
        <name gender="male" age="23">Doe John</name>
        <account_number>12345</account_number>
    </account>
</taxes>
```

Si la balise ciblée est `account`, PIMO va lire un flux *XML* en entrée jusqu'à ce qu'il trouve la balise `account`. Si son contenue est converti en format *JSON*, voici le résultat attendu en format JSON :

```json
{"@location":"New York", "name":"Doe John","name@gender":"male", "name@age":"23", "account_number":"12345"}
```

La configuration est également réalisée dans un fichier `YAML`. Selon le `jsonpath`, PIMO va trouver la clé adaptée, et masquer sa valeur. 

Pour suivre ce tutoriel, préparez les deux fichiers YAML suivants :

`masking_agence.yml`

```yaml
version: "1"
seed: 42

masking:
  - selector:
        # Sélection de nom de balise à masquer
        jsonpath: "numero_agence"  
    mask:
        # Fonction de masque à appliquer
        # Génération de 4 chiffres aléatoires
        # Dans XML, on n'accepte que des données du type chaîne de caractères. Pour les autres types, il faut passer par un masque 'template' pour le transformer en chaîne de caractères.
        template: '{{MaskRegex "[0-9]{4}$"}}'   
```

`masking_compte.yml`

```yaml
version: "1"
seed: 42

masking:
  - selector:
        # Sélection du nom d'attribut d'élement parent à modifier
        jsonpath: "@type" 
    mask:
        # Fonction de masque à appliquer
        # Sélection d'un valeur aléatoire parmi 3 choix
        randomChoice:
         - "classique"
         - "épargne"
         - "sécurité"
  - selector:
        # Sélection du nom d'attribut et du nom d'élement enfant à modifier
        jsonpath: "nom@age"
    masks:
        # Fonction de masque à appliquer
        # Génération d'un nombre aléatoire compris entre min et max inclus
      - randomInt:
            min: 18
            max: 95
        # Le symbole '@' n'est pas accepté par GO, donc nous devons utiliser un index dans le template pour convertir un entier en chaîne de caractères.
      - template: "{{index . \"nom@age\"}}"
```

Vous pouvez retrouver les masques disponible dans `pimo` [ici](https://github.com/CGI-FR/PIMO#possible-masks).

Pour masquer les valeurs des attributs, suivez les règles pour définir votre choix dans `jsonpath` :

* Pour les attributs de la balise parent, utilisez : `@nomAttribut` dans `jsonpath`.
* Pour les attributs de la balise enfant, utilisez : `nomBaliseEnfant@nomAttribut` dans `jsonpath`.

A notez aussi:

* Dans XML, On accepte que des données du type `String`. Pour les autres types, il faut passer par un masque `template` pour le transformer en chaîne de caractères.
* Ces balises ciblées ne doivent contenir aucune autre balise imbriquée.
* Le symbole `@` n'est pas accepté par GO, donc si vous avez généré une valeur en dehors de type `String`. Pour la convertir, il faut utiliser un index dans le `template`.

Pour suivre ce tutoriel, voici le fichier XML à créer:

`data.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<impôts>
    <agence>
        <nom>Agence NewYork</nom>               <!-- donnée à conserver   -->
        <numero_agence>0032</numero_agence>     <!-- donnée à masquer     -->
    </agence>
    <compte  type="classique">                  <!-- attribute à masquer  -->
        <nom age="25">Doe</nom>                 <!-- attribute à masquer  -->
        <numero_compte>12345</numero_compte>    <!-- donnée à conserver   -->
        <revenu_annuel>50000</revenu_annuel>    <!-- donnée à conserver   -->
    </compte >
    <compte  type="épargne">                    <!-- attribute à masquer  -->
        <nom age="50">Smith</nom>               <!-- attribute à masquer  -->
        <numero_compte>67890</numero_compte>    <!-- donnée à conserver   -->
        <revenu_annuel>60000</revenu_annuel>    <!-- donnée à conserver   -->
    </compte >
</impôts>
```

Dans cet exemple, vous pouvez masquer la valeur de `numero_agence` dans la balise `agence` et les valeurs des attributs `type` et `age` dans la balise `compte` en utilisant la commande suivante :

```bash
  cat data.xml | pimo xml --subscriber agence=masking_agence.yml --subscriber compte=masking_compte.yml > maskedData.xml
```

* `cat data.xml` définit le fichier d'entré.
* `pimo xml` indique qu'il s'agit d'un fichier XML à traiter.
* `--subscriber agence` spécifie la sélection du nom de la balise parent à masquer.
* `=masking_agence.yml` suit le nom de la balise pour sélectionner la configuration de masque appropriée.
* `--subscriber compte=masking_compte.yml` Vous pouvez choisir plusieurs balises parent à masquer avec des configurations différentes dans une seule commande. 
* `> maskedData.xml` définit le nom du fichier de sortie. 

Après avoir exécuté la commande avec la configuration correcte, voici le résultat attendu dans le fichier `maskedData.xml` :

`maskedData.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<impôts>
    <agence>
        <nom>Agence NewYork</nom>
        <numero_agence>2308</numero_agence>
    </agence>
    <compte type="épargne">
        <nom age="33">Doe</nom>
        <numero_compte>12345</numero_compte>
        <revenu_annuel>50000</revenu_annuel>
    </compte>
    <compte type="épargne">
        <nom age="47">Smith</nom>
        <numero_compte>67890</numero_compte>
        <revenu_annuel>60000</revenu_annuel>
    </compte>
</impôts>
```
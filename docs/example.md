# Exemples pratiques sur les outils LINO-PIMO

## Générer le NIR avec pimo

Le NIR plus communément appelé numéro de sécurité sociale est un numéro unique attribué par l'INSEE dès la naissance.

Il est composé de 15 chiffres : 

|  n°1  |         n°2         |         n°3        |         n°4        |              n°5             |        clé       |
|:-----:|:-------------------:|:------------------:|:------------------:|:----------------------------:|:----------------:|
|  Sexe | Année de  naissance | Mois de  naissance | Lieu de  naissance | Numéro d'ordre  de naissance | Clé de  contrôle |
| **X** |        **XX**       |       **XX**       |      **XXXXX**     |            **XXX**           |      **XX**      |

### 1ère méthode : regex

**`masking.yml`**
```yaml
version: "1"
seed: 42
masking:
  - selector:
      jsonpath: "nir"
    masks:
    - add: ""
    - regex: "(1|2)[0-9]{2}(0[1-9]|1[0-2])((0[1-9]|[1-8][0-9]|9[0-5])([0-8][0-9]{2}|90[0-9]))[0-9]{3}([0-9]{2})"
```

```console 
pimo -c masking.yml --empty-input -r 5
```

```json
{"nir":"195019390697033"}
{"nir":"177020490807345"}
{"nir":"264080615900240"}
{"nir":"185078983974899"}
{"nir":"177063490176950"}
```

### 2ème méthode : enchaînement de masking

**`masking.yml`**
```yaml
version: "2"
seed: 42
masking:
#----------------------------------------------------------
    # Partie n°5 du NIR
  - selector:
      jsonpath: "ORDRE_NAISSANCE"
    masks:
      - add-transient: ""
      - regex: "[0-9]{3}"
#----------------------------------------------------------

#----------------------------------------------------------  
    # Partie n°4 + n°5 du NIR
  - selector:
      jsonpath: "ADRESSE_BRUTE_NAISSANCE"
    masks:
      - add-transient: ""
      - randomChoiceInUri: "file://adresse.csv"
  
  - selector:
      jsonpath: "CODE_INSEE_NAISSANCE"
    masks:
      - add-transient: '{{$a := split ";" (toString .ADRESSE_BRUTE_NAISSANCE) }}{{$a._6}}'

  - selector:
      jsonpath: "FIN_NIR"
    masks:
      - add-transient: "{{(toString .CODE_INSEE_NAISSANCE)}}{{(toString .ORDRE_NAISSANCE)}}"
#----------------------------------------------------------

#----------------------------------------------------------
    # Partie n°1 + n°2 + n°3 + n°4 + n°5 du NIR
  - selector :
      jsonpath: "SEXE"
    masks:
      - add-transient: ""
      - randomChoice:
        - "M"
        - "F"

  - selector :
      jsonpath: "DATE_NAISSANCE"
    masks:
      - add-transient: ""
      - randDate:
          dateMin: "1960-01-01T00:00:00Z"
          dateMax: "2002-12-31T00:00:00Z"
      - dateParser:
          outputFormat: "02/01/2006"

  - selector:
      jsonpath: "VALEUR_NIR"
    masks:
      - add-transient: '{{if eq .SEXE "M" }}1{{else}}2{{end}}{{substr 8 10 (toString .DATE_NAISSANCE)}}{{substr 3 5 (toString .DATE_NAISSANCE)}}{{.FIN_NIR}}'
#----------------------------------------------------------

#----------------------------------------------------------
    # Partie Clé du NIR
    # Clé NIR = 97 - ( ( Valeur numérique du NIR ) modulo 97 )
  - selector:
      jsonpath: "CLE_NIR"
    masks:
      - add-transient: '{{ sub 97 (mod (int64 .VALEUR_NIR)  97)}}'
#----------------------------------------------------------

#----------------------------------------------------------
    # Totalité du NIR
  - selector:
      jsonpath: "NIR"
    masks:
      - add: '{{(toString .VALEUR_NIR)}}{{if eq (len .CLE_NIR) 1}}0{{(toString .CLE_NIR)}}{{else}}{{(toString .CLE_NIR)}}{{end}}'
#----------------------------------------------------------
```

```console 
pimo -c masking.yml --empty-input -r 5
```

```json
{"NIR":"190125900138430"}
{"NIR":"294025900107134"}
{"NIR":"100025900163453"}
{"NIR":"186025900137971"}
{"NIR":"172085900143917"}
```

## Générer une adresse mail avec pimo

**`masking.yml`**
```yaml
version: "1"
seed: 0
masking:

# ----------------------- NOM -----------------------
  - selector :
      jsonpath: "NOM"
    masks:
      - add-transient: "M"
      - randomChoiceInUri: "pimo://surnameFR"
      - template: "{{.NOM | NoAccent | lower}}"
  
# ----------------------- PRÉNOM -----------------------
  - selector :
      jsonpath: "PRENOM"
    masks:
      - add-transient: ""
      - randomChoiceInUri: "pimo://nameFR"
      - template: "{{.PRENOM | NoAccent | lower}}"

#---------------------------- EMAIL ----------------------
  - selector:
      jsonpath: "MAIL"
    masks:
      - add: '{{- .NOM | replace " " ""}}.{{- .PRENOM | replace " " ""}}@yopmail.com'
```

```console 
pimo -c masking.yml --empty-input -r 5
```

```json
{"MAIL":"girard.celiane@yopmail.com"}
{"MAIL":"henry.mickael@yopmail.com"}
{"MAIL":"caron.dalhia@yopmail.com"}
{"MAIL":"martin.veronique@yopmail.com"}
{"MAIL":"gauthier.chantal@yopmail.com"}
```

## Masquer des données imbriquées avec pimo

Prenons comme exemple le fichier suivant composé d'une liste de dossier. Chaque dossier ayant un identifiant et une liste de personnes concernées par ce dossier :

**`input.json`**
```json
{
  "dossier": [
    {
      "identifiant": "AZ18-45B12",
      "personnes": [
        {
          "nom": "Martin",
          "prénom": "Veronique",
          "email": "martin.veronique@mail.com"
        },
        {
          "nom": "Robert",
          "prénom": "Joe",
          "email": "rober.joe@mail.com"
        }
      ]
    },
    {
      "identifiant": "JL34-28C79",
      "personnes": [
        {
          "nom": "Alain",
          "prénom": "Mercier",
          "email": "alain.mercier@mail.com"
        },
        {
          "nom": "Florian",
          "prénom": "Roger",
          "email": "florian.roger@mail.com"
        }
      ]
    }
  ]
}
```

Pour masquer les données sensibles de ce fichier nous utilisons le fichier de configuration suivant:

**`masking.yml`**
```yaml
version: "1"
seed: 42
masking:
  - selector:
      jsonpath: "dossier.identifiant"
    mask:
        regex: "[A-Z]{2}[0-9]{2}-[0-9]{2}[A-Z][0-9]{2}"
  - selector:
      jsonpath: "dossier.personnes"
    mask:
      pipe:
        masking:
          - selector:
              jsonpath: "nom"
            mask: 
              randomChoiceInUri: "pimo://surnameFR"
          - selector:
              jsonpath: "prénom"
            mask: 
              randomChoiceInUri: "pimo://nameFR"
          - selector:
              jsonpath: "email"
            mask:
              template: "{{.nom | NoAccent | lower}}.{{.prénom | NoAccent | lower}}@mail.com"
```

```console 
pimo -c masking.yml < input.json | jq
```

```json
{
  "dossier": [
    {
      "identifiant": "RS03-09T37",
      "personnes": [
        {
          "nom": "Fournier",
          "prénom": "Orianne",
          "email": "fournier.orianne@mail.com"
        },
        {
          "nom": "Leclerc",
          "prénom": "Remi",
          "email": "leclerc.remi@mail.com"
        }
      ]
    },
    {
      "identifiant": "FP37-33R33",
      "personnes": [
        {
          "nom": "Vidal",
          "prénom": "Michaël",
          "email": "vidal.michael@mail.com"
        },
        {
          "nom": "Bertrand",
          "prénom": "Line",
          "email": "bertrand.line@mail.com"
        }
      ]
    }
  ]
}
```

## Générer un identifiant avec pimo

Si l'on reprend le fichier `masking.yml` de l'exemple précédent, on peut le modifier légèrement en remplaçant le masque *regex* par le masque *transcode* qui permet de produire une chaîne de caractère aléatoire (en préservant le format) et ainsi masquer l'identifiant.

**`masking.yml`**
```yaml
version: "2"
seed: 42
masking:
  - selector:
      jsonpath: "dossier.identifiant"
    mask:
        transcode: {}
  - selector:
      jsonpath: "dossier.personnes"
    mask:
      pipe:
        masking:
          - selector:
              jsonpath: "nom"
            mask: 
              randomChoiceInUri: "pimo://surnameFR"
          - selector:
              jsonpath: "prénom"
            mask: 
              randomChoiceInUri: "pimo://nameFR"
          - selector:
              jsonpath: "email"
            mask:
              template: "{{.nom | NoAccent | lower}}.{{.prénom | NoAccent | lower}}@mail.com"
```

```console 
pimo -c masking.yml < input.json | jq
```

```json
{
  "dossier": [
    {
      "identifiant": "PH30-37U04",
      "personnes": [
        {
          "nom": "Fournier",
          "prénom": "Orianne",
          "email": "fournier.orianne@mail.com"
        },
        {
          "nom": "Leclerc",
          "prénom": "Remi",
          "email": "leclerc.remi@mail.com"
        }
      ]
    },
    {
      "identifiant": "FX70-68A25",
      "personnes": [
        {
          "nom": "Vidal",
          "prénom": "Michaël",
          "email": "vidal.michael@mail.com"
        },
        {
          "nom": "Bertrand",
          "prénom": "Line",
          "email": "bertrand.line@mail.com"
        }
      ]
    }
  ]
}
```

Un moyen de générer un identifiant est d'ajouter un nouveau champ **id** contenant la valeur d'un identifiant, et ensuite d'utiliser le masque *transcode* qui va reproduire aléatoirement des chaînes de caractères en copiant le format de l'identifiant ajouté précédemment.

**`masking2.yml`**
```yaml
version: "1"
seed: 42
masking:
  - selector:
      jsonpath: "id"
    masks:
      - add: "1040-AZ-52"
      - transcode: {}
```

```console
pimo -c masking2.yml --empty-input -r 10
```

```json
{"id":"3485-KV-59"}
{"id":"6846-OT-19"}
{"id":"5706-GF-02"}
{"id":"5760-XR-28"}
{"id":"6254-EY-45"}
{"id":"5200-LL-28"}
{"id":"7407-ZN-94"}
{"id":"5547-PY-31"}
{"id":"9949-TY-61"}
{"id":"0383-IQ-93"}
```

# Exemples pratiques sur les outils LINO-PIMO

## Générer le NIR avec pimo

Le NIR plus communément appelé numéro de sécurité sociale est un numéro unique attribué par l'INSEE dès la naissance.

Il est composé de 15 chiffres : 

|  n°1  |         n°2         |         n°3        |         n°4        |              n°5             |        clé       |
|:-----:|:-------------------:|:------------------:|:------------------:|:----------------------------:|:----------------:|
|  Sexe | Année de  naissance | Mois de  naissance | Lieu de  naissance | Numéro d'ordre  de naissance | Clé de  contrôle |
| **X** |        **XX**       |       **XX**       |      **XXXXX**     |            **XXX**           |      **XX**      |

### 1ère méthode : regex

La 1ère méthode consiste à générer le numéro de sécurité sociale sans se préoccuper de connaître les informations liées à la personne (sexe, année de naissance, ...).

**`masking.yml`**
```yaml
version: "1"
seed: 42
masking:
  - selector:
      jsonpath: "VALEUR_NIR"
    masks:
    - add-transient: ""
    - regex: "(1|2)[0-9]{2}(0[1-9]|1[0-2])((0[1-9]|[1-8][0-9]|9[0-5])([0-8][0-9]{2}|90[0-9]))[0-9]{3}"

    # Partie Clé du NIR
    # Clé NIR = 97 - ( ( Valeur numérique du NIR ) modulo 97 )
  - selector:
      jsonpath: "CLE_NIR"
    mask:
      add-transient: '{{ sub 97 (mod (int64 .VALEUR_NIR)  97)}}'
  
  - selector:
      jsonpath: "NIR"
    mask:
      add: '{{(toString .VALEUR_NIR)}}{{if eq (len .CLE_NIR) 1}}0{{(toString .CLE_NIR)}}{{else}}{{(toString .CLE_NIR)}}{{end}}'
```

```console 
pimo -c masking.yml --empty-input -r 5
```

```json
{"NIR":"271067590908916"}
{"NIR":"148029053117547"}
{"NIR":"251079456401559"}
{"NIR":"198109213274847"}
{"NIR":"161113883742960"}
```

### 2ème méthode : enchaînement de masking

La 2ème méthode consiste, cette fois-ci, à générer le NIR en gardant la cohérence entre les informations d'une personne et son numéro de sécurité sociale.

**`masking.yml`**
```yaml
version: "2"
seed: 42
masking:
#----------------------------------------------------------
    # Partie n°1 + n°2 + n°3 du NIR
  - selector :
      jsonpath: "SEXE"
    masks:
      - add: ""
      - randomChoice:
        - "M"
        - "F"

  - selector :
      jsonpath: "DATE_NAISSANCE"
    masks:
      - add: ""
      - randDate:
          dateMin: "1960-01-01T00:00:00Z"
          dateMax: "2002-12-31T00:00:00Z"
      - dateParser:
          outputFormat: "02/01/2006"
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
      - add: '{{$a := split ";" (toString .ADRESSE_BRUTE_NAISSANCE) }}{{$a._6}}'
  
  - selector:
      jsonpath: "ORDRE_NAISSANCE"
    masks:
      - add: ""
      - regex: "[0-9]{3}"

  - selector:
      jsonpath: "FIN_NIR"
    masks:
      - add-transient: "{{(toString .CODE_INSEE_NAISSANCE)}}{{(toString .ORDRE_NAISSANCE)}}"
#----------------------------------------------------------

#----------------------------------------------------------
    # Partie Valeur du NIR
  - selector:
      jsonpath: "VALEUR_NIR"
    masks:
      - add-transient: '{{if eq .SEXE "M" }}2{{else}}1{{end}}{{substr 8 10 (toString .DATE_NAISSANCE)}}{{substr 3 5 (toString .DATE_NAISSANCE)}}{{.FIN_NIR}}'
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

Ci-dessous les 5 premières lignes du fichier `adresse.csv`:

| id                 | id_fantoir | numero | rep | nom_voie           | code_postal | code_insee | nom_commune | code_insee_ancienne_commune | nom_ancienne_commune | x         | y          | lon      | lat       | alias | nom_ld | libelle_acheminement | nom_afnor          | source_position | source_nom_voie |
|:------------------:|:----------:|:------:|:---:|:------------------:|:-----------:|:----------:|:-----------:|:---------------------------:|:--------------------:|:---------:|:----------:|:--------:|:---------:|:-----:|:------:|:--------------------:|:------------------:|:---------------:|:---------------:|
| 59001_nhsgdi_00001 |            | 1      |     | Chemin Hem Lenglet | 59268       | 59001      | Abancourt   |                             |                      | 715617.46 | 7015153.4  | 3.218633 | 50.234188 |       |        | ABANCOURT            | CHEMIN HEM LENGLET | arcep           | arcep           |
| 59001_0260_00001   | 59001_0260 | 1      |     | Rue Verte          | 59268       | 59001      | Abancourt   |                             |                      | 715163.83 | 7015088.68 | 3.21228  | 50.233618 |       |        | ABANCOURT            | RUE VERTE          | inconnue        | inconnue        |
| 59001_0260_00003   | 59001_0260 | 3      |     | Rue Verte          | 59268       | 59001      | Abancourt   |                             |                      | 715152.71 | 7015104.02 | 3.212125 | 50.233756 |       |        | ABANCOURT            | RUE VERTE          | inconnue        | inconnue        |
| 59001_0260_00004   | 59001_0260 | 4      |     | Rue Verte          | 59268       | 59001      | Abancourt   |                             |                      | 715121.01 | 7015153.72 | 3.211683 | 50.234203 |       |        | ABANCOURT            | RUE VERTE          | inconnue        | inconnue        |
| 59001_0260_00005   | 59001_0260 | 5      |     | Rue Verte          | 59268       | 59001      | Abancourt   |                             |                      | 715140.52 | 7015123.37 | 3.211955 | 50.23393  |       |        | ABANCOURT            | RUE VERTE          | inconnue        | inconnue        |

```console 
pimo -c masking.yml --empty-input -r 5
```

```json
{"SEXE":"F","DATE_NAISSANCE":"12/12/1990","CODE_INSEE_NAISSANCE":"59001","ORDRE_NAISSANCE":"384","NIR":"190125900138430"}
{"SEXE":"M","DATE_NAISSANCE":"01/02/1994","CODE_INSEE_NAISSANCE":"59001","ORDRE_NAISSANCE":"071","NIR":"294025900107134"}
{"SEXE":"F","DATE_NAISSANCE":"23/02/2000","CODE_INSEE_NAISSANCE":"59001","ORDRE_NAISSANCE":"634","NIR":"100025900163453"}
{"SEXE":"F","DATE_NAISSANCE":"11/02/1986","CODE_INSEE_NAISSANCE":"59001","ORDRE_NAISSANCE":"379","NIR":"186025900137971"}
{"SEXE":"F","DATE_NAISSANCE":"21/08/1972","CODE_INSEE_NAISSANCE":"59001","ORDRE_NAISSANCE":"439","NIR":"172085900143917"}
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
          "nom": "martin",
          "prénom": "veronique",
          "email": "martin.veronique@mail.com"
        },
        {
          "nom": "robert",
          "prénom": "joe",
          "email": "rober.joe@mail.com"
        }
      ]
    },
    {
      "identifiant": "JL34-28C79",
      "personnes": [
        {
          "nom": "alain",
          "prénom": "mercier",
          "email": "alain.mercier@mail.com"
        },
        {
          "nom": "florian",
          "prénom": "roger",
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
              template: "{{.nom}}.{{.prénom}}@mail.com"
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
          "email": "Fournier.Orianne@mail.com"
        },
        {
          "nom": "Leclerc",
          "prénom": "Remi",
          "email": "Leclerc.Remi@mail.com"
        }
      ]
    },
    {
      "identifiant": "FP37-33R33",
      "personnes": [
        {
          "nom": "Vidal",
          "prénom": "Michaël",
          "email": "Vidal.Michaël@mail.com"
        },
        {
          "nom": "Bertrand",
          "prénom": "Line",
          "email": "Bertrand.Line@mail.com"
        }
      ]
    }
  ]
}
```
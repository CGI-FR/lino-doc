# Exemples pratiques sur les outils LINO-PIMO

## Générer le NIR avec PIMO

Le NIR plus communément appelé numéro de sécurité sociale est un numéro unique attribué par l'INSEE dès la naissance.

Il est composé de 15 chiffres : 

|  n°1  |         n°2         |         n°3        |         n°4        |              n°5             |        clé       |
|:-----:|:-------------------:|:------------------:|:------------------:|:----------------------------:|:----------------:|
|  Sexe | Année de  naissance | Mois de  naissance | Lieu de  naissance | Numéro d'ordre  de naissance | Clé de  contrôle |
| **X** |        **XX**       |       **XX**       |      **XXXXX**     |            **XXX**           |      **XX**      |

### 1ère méthode : regex

```yaml
version: "1"
seed: 42
masking:
  - selector:
      jsonpath: "nir"
    mask:
      regex: "(1|2)[0-9]{2}(0[1-9]|1[0-2])((0[1-9]|[1-8][0-9]|9[0-5])([0-8][0-9]{2}|90[0-9]))[0-9]{3}([0-9]{2})"
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

### 2ème méthode : 

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
      - add-transient: ""
      - template: '{{$a := split ";" (toString .ADRESSE_BRUTE_NAISSANCE) }}{{$a._6}}'

  - selector:
      jsonpath: "FIN_NIR"
    masks:
      - add-transient: ""
      - template: "{{(toString .CODE_INSEE_NAISSANCE)}}{{(toString .ORDRE_NAISSANCE)}}"
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
      - add-transient: ""
      - template: '{{if eq .SEXE "M" }}2{{else}}1{{end}}{{substr 8 10 (toString .DATE_NAISSANCE)}}{{substr 3 5 (toString .DATE_NAISSANCE)}}{{.FIN_NIR}}'
#----------------------------------------------------------

#----------------------------------------------------------
    # Partie Clé du NIR
    # Clé NIR = 97 - ( ( Valeur numérique du NIR ) modulo 97 )
  - selector:
      jsonpath: "CLE_NIR"
    masks:
      - add-transient: ""
      - template: '{{ sub 97 (mod (int64 .VALEUR_NIR)  97)}}'
#----------------------------------------------------------

#----------------------------------------------------------
    # Totalité du NIR
  - selector:
      jsonpath: "NIR"
    mask:
      add: ""

  - selector:
      jsonpath: "NIR"
    mask:
      template: '{{(toString .VALEUR_NIR)}}{{if eq (len .CLE_NIR) 1}}0{{(toString .CLE_NIR)}}{{else}}{{(toString .CLE_NIR)}}{{end}}'
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
# Capítol 2. Format `JSON` i LocalStorage
Angular utilitza el format `JSON` i el LocalStorage per crear objectes complexos, sense fer ús d'una classe, i tenir capacitat de persistència de dades.

## Format `JSON`
El format `JSON` és un format molt simple que permet definir objectes mitjançant un conjunt de tuples `clau-valor`. Actualment, a més a més, ha desplaçat el llenguatge de marques `XML` en l'àmbit de la transmissió de dades per internet (serveis web), ja que el `JSON` és molt més lleuger que l'`XML`.

La seva sintaxi està formada, bàsicament pels següents símbols:`{`, `}`, `[`, `]`, `:`, `"`, `'` i `,`, de tal manera que, per exemple, per descriure les dades d'un alumne podem crear el següent objecte:

```json
{
    "first_name": "Ignasi",
    "last_name": "Vila Guerrero",
    "dni": "123456789A",
    "high_school": "Institut Caparrella",
    "studies": "DAW",
    "course": 2,
    "subjects": [
        {
            "code": "MP06",
            "name": "Desenvolupament en entorn client"
        }, 
        {
            "code": "MP07",
            "name": "Desenvolupament en entorn servidor"
        },
        {
            "code": "MP08",
            "name": "Desplegament d'aplicacions web"
        },
        {
            "code": "MP09",
            "name": "Disseny d'interfícies web"
        },
        {
            "code": "MP12",
            "name": "Projecte"
        },
        {
            "code": "MP13",
            "name": "FCT"
        },
    ]
}
```

Les dades en format `JSON` es poden emmagatzemar en un fitxer amb extensió `.json` o es poden utilitzar per crear objectes dins del codi Angular.

## Exemple {.tabset .tabset-fade}

### tab Codi TS
wegwegw

### tab Codi HTML
weigwoeg
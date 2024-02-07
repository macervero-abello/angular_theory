# Capítol 9. Workspace: una aplicació, múltiples GUI
Moltes vegades ens trobarem amb la necessitat d'adaptar una mateixa aplicació a diferents interfícies gràfiques per fer-ne, per exemple, la versió web i la versió mòbil. Si aquesta adaptació la volem fer mitjançant `CSS` personalitzades o, si menys no, utilitzant el mateix *framework* `CSS` per a totes les versions, no tindrem cap altre problema fent-ho com ho hem fet fins ara. Malauradament, si volem utilitzar un *framework* `CSS` per cada versió, els nostres *components* seran totalment diferents i, per tant, haurem de fer, sí o sí, dues aplicacions.

Podem veure, però, que fer dues aplicacions que tenen la mateixa lògica (la mateixa funcionalitat, el mateix nucli - *core*) només perquè la interfície gràfica canvia, no té massa sentit, ja que qualsevol canvi en les funcionalitats s'haurà d'implementar 2 vegades. Per evitar aquesta duplicació de codi podem utilitzar els [*Workspaces*](https://angular.io/guide/file-structure) d'Angular.

Un *workspace* permet gestionar múltiples aplicacions de manera simultània, així com també llibreries. Per tant, si volem crear una mateixa aplicació per a web i per a mòbil, podem crear un *workspace* que contingui
 1. una lliberia Angular amb el *core* de l'aplicació (tot el codi de la lògica, és a dir, els *services* i els *models*),
 2. una aplicació per a la versió web i
 3. una aplicació per a la versió mòbil.

Les aplicacions web i mòbil només hauran d'implementar la part de la interfície gràfica (GUI) i utilitzar la llibreria per tenir la lògica de programa. D'aquesta manera, si canvia alguna de les funcionalitats o se n'afegeixen de noves, només cal que modifiquem el codi de la llibreria una única vegada i totes les aplicacions se'n veuran beneficiades directament.

## Creació d'un *Workspace*
Per crear un *workspace* cal executar la comanda següent
```bash
   ng new worskspace_name --no-create-application
```
Aquesta comanda crearà una estructura idèntica a la d'un projecte Angular a excepció de la carpeta `src`, que no hi és. Aquesta carpeta serà substituïda pel directori `projects`, que és on es definirà el codi de cadascuna de les aplicacions i llibreries del *workspace*.

Un cop creat el *workspace* podem afegir-hi aplicacions amb la comanda
```bash
   ng generate application application_name
```
i llibreries amb la comanda
```bash
   ng generate library lib_name
```

Això crearà les carpetes `application_name` i `lib_name` dins del directori `projects` i hi instal·larà tots els fitxers de configuració necessaris.

Si suposem que hem creat el *workspace* `WSProva` amb l'aplicació `App1` i la llibreria `Lib`, l'estructura de fitxers serà la següent:

![Estructura de fitxers d'un *workspace* amb una lliberia i una aplicació](img/workspace_file_directory.png)

Cal adonar-se que la carpeta `node_modules` i el fitxer `package.json` pertanyen al *workspace* i, per tant, totes les aplicacions s'alimenten de les mateixes dependències. Les llibreries també tenen un fitxer propi `package.json` per tal que puguin afegir dependències que només necessiten elles i no la resta d'aplicacions del *workspace*.

La carpeta `app1`, corresponent a l'aplicació `App1` té el següent contingut:

![Estructura de fitxers d'una aplicació dins del *workspace*](img/workspace_app_file_directory.png)

El nostre codi, com fins ara, anirà dins del directori `src`.

La carpeta `lib`, corresponent a la llibreria `Lib` té el següent contingut:

![Estructura de fitxers d'una llibreria dins del *workspace*](img/workspace_lib_file_directory.png)

El nostre codi anirà dins del directori `src/lib`. A més a més, el fitxer `public-api.ts` ens permetrà indicar els fitxers als quals es podrà accedir des de les aplicacions externes.

> [!IMPORTANT]
>
> Si utilitzeu la versió 17 d'Angular (comprovar-ho al fitxer `package.json`), per evitar problemes amb les novetats d'aquesta versió respecte de l'anterior, cal crear les noves aplicacions del *Workspace* amb la compnda següent:
>
> `ng generate application --standalone false application_name`

## Distribució del codi entre la llibreria i les aplicacions
Cal tenir en compte que tant les llibreries com les aplicacions poden tenir tot tipus de codi: models, *services* i components entre altres. Ara però, si el que desitgem és externalitzar el codi comú de múltiples aplicacions en una única llibreria que puguin utilitzar totes elles, tenint en compte que estem aplicant el patró de disseny de *software* **Model-View-Controller** (MVC), el que cal fer és el següent:
 1. la llibreria contindrà tot el codi del nucli (*core*) de les aplicacions, és a dir, el codi corresponent als *Controllers* (*services*) i als *Models*
 2. les diverses aplicacions definiran les diferents *Views* i, per tant, s'encarregaran de crear els *components*.

A banda d'això, cal tenir en compte que qualsevol *asset* o variable d'entorn definida als fitxers *environment* només es pot incloure dins de les aplicacions, no de les llibreries.

## Visibilitat i reutilització del codi de la llibreria
Per tal que el codi de la llibreria pugui ser accessible des de qualsevol aplicació i, per tant, es pugui utilitzar o consumir, cal definir una API pública amb tot allò que volem que sigui visible des de l'exterior.

Aquesta API es defineix al fitxer `public-api.ts`, que es troba dins de la carpeta `src` de la llibreria i el seu objectiu és exportar (fer visible) tot aquell codi que vulguem que sigui públic.

Imaginem que tenim llibreria `lib_name` que defineix el seu propi mòdul de dependències al fitxer `lib_name.module.ts`, el model `User` al fitxer `models/user.model.ts` i el *service* `UserService` al fitxer `services/user.service.ts`. Sabent que volem accedir a tots tres fitxers des de qualsevol aplicació, l'API pública de la llibreria és la següent:
```typescript
/*
 * Public API Surface of lib
 */

   export * from './lib_name/services/user.service';
   export * from './lib_name/models/user.model';
   export * from './lib_name/lib_name.module';
```

## Compilació de la llibreria i execució de l'aplicació
Un cop creada la llibreria i les aplicacions, per tal de poder-ho executar cal
 1. compliar la llibreria (dins de la carpeta de la llibreria)
 ```bash
   ng build lib_name
 ```
 2. activar el servidor *on the fly* d'Angular (dins de la carpeta de l'aplicació)
 ```bash
   ng serve
 ```
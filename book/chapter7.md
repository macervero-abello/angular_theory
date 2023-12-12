# Capítol 7. Firebase
Firebase és una *suit* de Google que ofereix tot un conjunt d'eines per ajudar als desenvolupadors d'aplicacions mòbils a crear i posar al mercat els seus projectes.

En el nostre cas utilitzarem la seva base de dades NoSQL (*Firestore*) per tal de tenir un petit servidor de dades per a les nostres aplicacions.

## Configuració de la BD Firebase
Per configurar la base de dades de Firebase i, de fet, qualsevol dels seus serveis, cal fer dos grans passos:
 1. Enllaçar la vostra aplicació Angular a la *suit* Firebase
 2. Enllaçar l'SDK de la suit de Firebase a la vostra aplicació Angular

A continuació s'explica com s'han de fer tots dos passos.

### Enllaç de l'aplicació Angular a la *suit* Firebase
Per crear aquest vincle cal que inicieu sessió a la pàgina web de la [suit](https://firebase.google.com) i accediu a la consola (*dashboard*)

![Accés a la consola de Firebase](img/firebase_console.png)

Un cop dins, per cada aplicació que hagi d'utilitzar Firebase cal crear un nou projecte, seguint les instruccions que va indicant el propi procés.

Quan el procés de creació finalitzi es carregarà el *dashboard* del projecte per tal que el pugueu configurar amb les característiques necessàries per a l'aplicació mòbil (o web) que s'estigui desenvolupant.

El primer que cal fer és afegir una nova aplicació web, que pot ser amb codi nadiu (iOS o Android), codi mutiplataforma (HTML, JS, Angular, React, etc.), codi Unity (per a jocs) o codi Flutter. En el nostre cas afegirem una nova aplicació Angular, tal com mostra la següent imatge i seguirem els passos que va indicant el procés de configuració.

![Configuració de la nova aplicació per al projecte Firebase](img/firebase_add_project.png)

Un cop la nova aplicació quedi registrada es mostrarà el codi que es necessita incloure al codi Angular per fer el vincle amb el projecte de Firebase. Tot i això, de moment, aquest codi no s'utiltizarà perquè està pensat per ser aplicat a una aplicació Javascript i, en canvi, nosaltres, el necessitem aplicar dins del *framework* d'Angular

![Codi per vincular l'aplicació Angular projecte Firebase](img/firebase_link_code.png)

Així doncs, en aquest punt cal accedir a la nostra aplicació Angular i seguir els passos següents:
 1. Instal·lar el paquet `firebase-tools` per poder autenticar al desenvolupador de l'aplicació
 ```bash
    npm install firebase-tools
 ```
 2. Autenticar l'usuari desenvolupador
 ```bash
    firebase login
 ```
 3. Instal·lar el paquet `@angular/fire`. Durant aquesta instal·lació
    * no heu de configurar cap servei de Firebase,
    * heu d'escollir el compte Google del desenvolupador,
    * heu d'escollir el projecte Firebase que heu creat recentment per poder fer el vincle i
    * heu de seleccionar un *hosting* pel projecte (el que indica per defecte)
 ```bash
    ng add @angular/fire
 ```
 ![Configuració del paquet `@angular/fire`](img/install_angular_fire.png)

Fet això, tornarem al *dashboard* del nostre projecte de Firebase per crear i configurar la base de dades, servei que s'anomena *Firestore*.
 1. Activar Firestore dins del *dashboard*
 ![Activació de Firestore](img/firestore_activation.png)
 2. Crear la nova base de dades
 ![Creació de la BD Firestore](img/firebase_create_db.png)
    1. Configurar el servidor on s'allotjarà la base de dades (busqueu un que sigui europeu)
    ![Localització de la BD Firestore](img/firestore_db_location.png)
    2. Configuració del mode *producció* o *desenvolupador* (proves). Per defecte, admetrem el mode de *producció*, el qual haurà de ser modificat posteriorment per permetre les operacions de lectura i escriptura a la base de dades
    ![Mode de la BD Firestore](img/firestore_db_mode.png)
 3. Un cop creada la base de dades, l'aplicatiu fa una redirecció a la pàgina inicial del gestor de dades on, el primer que cal fer, és els permisos (regles) d'accés per tal que tots els usuaris de l'aplicació puguin escriure i llegir dades.
 ![Permisos de la BD Firestore](img/firestore_db_rules.png)

Un cop finalitzats tots aquests passos, ja es pot començar a poblar la base de dades. Prèviament, però, encara cal enllaçar l'SDK de Firebase a l'aplicació Angular.

### Enllaç de l'SDK de la *suit* de Firebase a l'aplicació Angular

Per enllaçar l'SDK de Firebase a l'aplicació Angular al crear els arxius d'*environment*. Aquests arxius han passat a ser opcionals a les aplicacions d'Angular (abans eren obligatoris i es creaven al mateix moment en què es creava el projecte) i la seva funció és la de declarar variables globals segons l'entorn de desenvolupament que es desitgi: *producció* o *desenvolupament*.

Per crear aquests arxius cal executar la comanda
```bash
   ng generate environments
```
Fet això apareixen els dos fitxers següents:
 - `src/environments/environment.development.ts`: entorn de *desenvolupament*
 - `src/environments/environment.ts`: entorn de *producció*

En aquest punt cal recuperar la configuració de les variables de la *suit* Firebase entrant al *dashboard* per accedir a la configuració del nou projecte creat, tal com indica la següent imatge:
![Configuració de les constants de la *suit* de Firebase](img/firebase_sdk.png)

De totes aquestes dades només ens interessa la constant `firebaseConfig`, la qual s'ha de traspassar al fitxer `src/environments/environment.ts` de la següent manera:
```typescript
export const environment = {
    production: false,
    firebaseConfig: {
        apiKey: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        authDomain: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        projectId: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        storageBucket: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        messagingSenderId: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        appId: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      }
};
```
Finalment, ja només cal afegir la dependència dels mòduls de Firebase i dels fitxers `environment` al fitxer `app.module.ts`:
```typescript
...
import { environment } from 'src/environments/environment';
import { AngularFireModule } from '@angular/fire/compat';
import { AngularFirestoreModule } from '@angular/fire/compat/firestore';


import { AppComponent } from './app.component';

@NgModule({
  declarations: [ AppComponent ],
  imports: [
   ...
   AngularFireModule.initializeApp(environment.firebaseConfig),
   AngularFirestoreModule

  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Seguint tots aquests passos s'aconsegueix tenir preparat tant la *suit* Firebase com l'aplicació Angular per a poder utilitzar la base de dades *Firestore*.

### Base de dades *Firestore*
Tal com ja s'ha dit al principi del capítol, aquesta base de dades és del tipus NoSQL, per tant, la manera d'emmagatzemar les dades i de fer-hi consultes canvia significativament.

Per una banda, les dades s'emmagatzemen com a objectes JSON dins de col·leccions. Així doncs, per tal que us feu una idea, i sabent que **no són el mateix**, podeu fer les següents associacions:
 * Col·lecció &rarr; Taula SQL
 * Document JSON &rarr; Registre dins d'una taula SQL

Accedint de nou al *dasboard* de *Firestore* podeu crear una nova col·lecció (per exemple `dishes`) i plenar-la de dades per tenir un exemple de base.

 1. Creació de la nova col·lecció
 ![Creació d'una nova col·lecció a *Firestore*](img/firestore_new_collection.png)
 2. Creació d'un nou document dins de la col·lecció
 ![Creació d'un nou document dins de la col·lecció](img/firestore_new_collection_3.png)
 3. Resultat final
 ![Resultat final](img/firestore_new_collection_4.png)

## Aplicació CRUD bàsica

## Autenticació
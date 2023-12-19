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
 3. Instal·lar els paquets `firebase` i `@angular/fire`. Durant aquesta instal·lació
    * no heu de configurar cap servei de Firebase,
    * heu d'escollir el compte Google del desenvolupador,
    * heu d'escollir el projecte Firebase que heu creat recentment per poder fer el vincle i
    * heu de seleccionar un *hosting* pel projecte (el que indica per defecte)
 ```bash
   npm install firebase
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

De totes aquestes dades només ens interessa la constant `firebaseConfig`, la qual s'ha de traspassar als fitxers `src/environments/environment.ts` i `src/environments/environment.development.ts` de la següent manera:
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

> **ATENCIÓ:** les últimes versions d'Angular (15.2.0), Firebase (10.7.1) i Angular-Fire (7.6.1) no acaben de ser comptabiles i, mentre els desenvolupadors no ho arreglin, cal els retocs que es mostren a continuació al fitxer `node-modules/@angular/fire/compat/firestore/interfaces.d.ts`
```typescript
//Versió instal·lada
...
export interface DocumentSnapshotExists<T> extends firebase.firestore.DocumentSnapshot {...}
...
export interface QueryDocumentSnapshot<T> extends firebase.firestore.QueryDocumentSnapshot {...}
export interface QuerySnapshot<T> extends firebase.firestore.QuerySnapshot {...}
export interface DocumentChange<T> extends firebase.firestore.DocumentChange {...}
...
```

```typescript
//Modificacions que cal fer
...
export interface DocumentSnapshotExists<T> extends firebase.firestore.DocumentSnapshot<T> {...}
...
export interface QueryDocumentSnapshot<T> extends firebase.firestore.QueryDocumentSnapshot<T> {...}
export interface QuerySnapshot<T> extends firebase.firestore.QuerySnapshot<T> {...}
export interface DocumentChange<T> extends firebase.firestore.DocumentChange<T> {...}
...
```

Una aplicació CRUD és tot aquell programari que permet treballar amb bases de dades (siguin fitxers, SQL, NoSQL, etc.) per
 * **C**: crear noves dades (*create*),
 * **R**: consultar dades (*read*),
 * **U**: modificar dades (*update*) i
 * **D**: esborrar dades (*delete*).

El servei *Firestore* de Firebase permet treballar amb una base de dades NoSQL per fer-ne una gestió senzilla. A continuació s'explica com integrar l'API de *Firestore* a Angular.

### Primers passos
Suposarem que estem creant una aplicació de plats de cuina i, per tant, la nostra base de dades emmagatzemarà documents que tenen dos atributs: `name` (nom del plat) i `ingredients` (els ingredients que fan falta per cuinar-lo), tal com mostren les imatges de creació de la col·lecció (vegeu apartat anterior).

El primer que cal fer és dissenyar l'aplicació modularment, aplicant el patró MVC. Per tant, s'haurà de crear un *model* (`Dish`) que repliqui el contingut dels documents de la base de dades, és a dir, que tingui els mateixos atributs que els camps del document i un *service* (`DishesService`) que gestioni l'accés a la base de dades i totes les consultes que s'hi poden fer.

Per a definir el model, com que és molt simple, crearem la *interface* `Dish`, la qual tindrà els mateixos 2 atributs que tenen els documents de la base de dades: `name` i `ingredients`:
```typescript
export interface Dish {
    name: string;
    ingredients: string[];
}
```

A continuació caldrà generar el *service* `DishesService` que serà l'encarregat de gestionar totes les instruccions **CRUD** contra la base de dades *Firestore*.

Abans de fer aquest pas, però, cal explicar una mica les funcions bàsiques de l'API de *Firestore*.

#### Mètodes bàsics de l'API de *Firestore*
Per tal de poder connectar amb una col·lecció de la base de dades, l'API de *Firestore* defineix el *service* `AngularFirestore` i la classe `AngularFirestoreCollection<T>`, la qual té tots els mètodes que permeten llegir i modificar els registres (els documents) de la col·lecció.

El codi necessari per crear una instància d'aquesta classe i enllaçar-la a la col·lecció que es desitja consultar és el següent:
```typescript
import { AngularFirestoreCollection, AngularFirestore } from '@angular/fire/compat/firestore';

class ManageFirestore {
   private _collection: AngularFirestoreCollection<MyType>;

   construct(private _firestoreService: AngularFirestore) {
      this._collection = this._firestore.collection<MyType>('collection_name');
   }
}
```
Així doncs, cal demanar al *service* `AngularFirestore` la collecció `collection_name` mitjançant el mètode `collection()`, el qual retorna, justament, una instància de la classe `AngularFirestoreCollection`.

Vegeu que la classe `AngularFirestoreCollection` és genèrica i s'ha de declarar utilitzant el tipus correcte, és a dir, la *interface* o la classe que s'hagi definit per tal de representar els documents de la col·lecció dins de la nostra aplicació.

Un cop obtinguta la instància que representa la col·lecció de la base de dades ja s'hi pot començar a operar.

##### Consulta de dades (*query*)
Per poder consultar els documents d'una col·lecció hi ha diversos mètodes, depenent del tipus de *query* que es desitja realitzar. El més senzill d'aplicar, però, és el mètode `valueChanges()`.

**Mètode `valueChanges()`**
Aquest mètode permet fer consultes equivalents a la sentència `SELECT * FROM` SQL, és a dir, retorna tots els documents d'una col·lecció. A més a més, implementa el patró `Observer` i, per tant, per obtenir les dades caldrà fer la subscripció a la crida:
```typescript
   this._collection.valueChanges().subscribe({
      next: (data: MyType[]) => {
         //Tractament de les dades
      },
      complete: () => {}
      error: (msg: string) => {
         console.log(msg);
      }
   });
```
Mentre es mantingui la subscripció activa, cada cop que hi hagi un canvi a la col·lecció de la base de dades, aquesta funció el captarà i el retornarà per tal que pugui ser tractat, mostrat per pantalla, etc.

Vegeu que el mètode `next` rep, com a paràmetre, un *array* del tipus amb què s'ha declarat l'objecte col·lecció `AngularFirestoreCollection`, és a dir, `MyType`.

En cas que es desitgi recuperar l'identificador (equivalent a la *primary key* SQL) dels documents, el mètode `valueChanges()` ha de rebre un paràmetre de tipus `JSON` on s'especifiqui el nom de l'atribut identificador al tipus `MyType`. Així doncs, si `MyType` té un atribut `id: string`, la crida al mètode `valueChanges()` és la següent:
```typescript
   this._collection.valueChanges({'idField': 'id'}).subscribe({
      next: (data: MyType[]) => {
         //Tractament de les dades
      },
      complete: () => {}
      error: (msg: string) => {
         console.log(msg);
      }
   });
```

Per a consultes més complexes també s'utilitza el mètode `valueChanges()` però, aquest cop, combinada amb funcions lamda anònimes per poder fer les sentències:
* `where`
* `orderBy`
* `limit`
* `startAt`
* `startAfter`
* `endAt`
* `endBefore`

Per exemple, imaginem que tenim un document que representa el conjunt de la població de la província de Lleida i només volem consultar aquelles persones que són menors d'edat. Per fer-ho, la crida seria similar a la següent:
```typescript
   this._firestore.collection<Person>('person', (ref) => {
      return ref.where('age', '<', 18);
   }).valueChanges({'idField': 'id'}).subscribe({
      next: (data: Person[]) => {
         //Tractament de les dades
      },
      complete: () => {},
      error: (msg: string) => {
         console.log(msg);
      }
   });
```
Ara però, les consultes complexes de *Firestore* tenen força limitacions i una d'elles és que no és poden concatenar dues condicions de desigualtat sobre camps diferents. Per tant, la consulta que ens permetria trobar tots els menors d'edat que no són de Lleida no es pot fer directament amb la sentència `WHERE` i l'estratègia per aconseguir els resultats és aplicar un filtre:
```typescript
   this._firestore.collection<Person>('person', (ref) => {
      return ref.where('age', '<', 18);
   }).valueChanges().subscribe(
      next: (data: Person[]) => {
         data.filter(
            (person: Person) => return person.city != "Lleida";
         );
      },
      complete: () => {};
      error: (msg: string) => {
         console.log(msg);
      }
   );
```

Per analitzar totes les possibilitats de les diverses sentències es pot consultar la pàgina [Angular Firebase](https://github.com/angular/angularfire/blob/master/docs/compat/firestore/querying-collections.md)

##### Inserció de dades
Per fer la inserció d'un nou document en una col·lecció s'utilitza el mètode `add`. Així doncs, donada la instància de la col·lecció (`this._collection`) i l'objecte amb les dades que es volen inserir (`my_data`), el codi per fer la inserció és el següent:
```typescript
   this._collection.add(my_data);
```

##### Eliminació de dades
Per fer l'eliminació d'un document d'una col·lecció s'utilitzen els mètodes `doc` i `delete`. El primer mètode permet seleccionar el document a través del seu identificador (equivalent a la `primary key` en el cas d'SQL) i, un cop seleccionat, el mètode `delete` l'esborra. Així doncs, donada la instància de la col·lecció (`this._collection`) i l'identificador de l'objecte que volem esborrar (`my_data.id`), el codi per fer la inserció és el següent:
```typescript
   this._collection.add(my_data.id).delete();
```

##### Actualització de dades
Per fer l'actualització d'un document d'una col·lecció s'utilitzen els mètodes `doc` i `update` o `set`. Així doncs, en aquest cas tenim diverses opcions:
 1. Utilitzar el mètode `set` l'objecte sencer que volen actualitzar (encara que hi hagi camps que no faci falta canviar)
 2. Utilitzar el mètode `set` amb només els camps que es volen actualitzar
 3. Utilitzar el mètode `update`, el qual només treballa amb els camps que es volen actualitzar (equival al `set` del 2n punt).

Suposant que tenim la instància de la col·lecció (`this._collection`), l'identificador de l'objecte que volem actualitzar (`my_data.id`) i que aquest objecte té 2 camps (`title` i `url`), els següents apartats mostren el codi de cadascuna de les opcions indicades a sobre.

**Mètode `set` complet**
```typescript
   let new_data: any = {
      "title": "My title",
      "url": "My URL"
   };

   this._collection.doc(my_data.id).set(new_data);
```
El mètode `set` utilitzat d'aquesta manera actualitza el document, si aquest ja existia a la col·lecció, o el crea de nou, si no existia, és a dir, si el document amb identificador `my_data.id` no existeix, el mètode `set` equival al mètode `add`.

**Mètode `set` que actualitza camps de manera independent**
```typescript
   this._collection.doc(my_data.id).set({"title": "My title"}, {"merge": true});
```

**Mètode `update` (actualitza camps de manera independent)**
```typescript
   this._collection.doc(my_data.id).update({"title": "My title"});
```

## Autenticació
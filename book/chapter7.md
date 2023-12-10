# Capítol 7. Firebase
Firebase és una *suit* de Google que ofereix tot un conjunt d'eines per ajudar als desenvolupadors d'aplicacions mòbils a crear i posar al mercat els seus projectes.

En el nostre cas utilitzarem la seva base de dades NoSQL per tal de tenir un petit servidor de dades per a les nostres aplicacions.

## Configuració de la BD Firebase
Per configurar la base de dades Firebase cal que inicieu sessió a la pàgina web de la [suit](https://firebase.google.com) i accediu a la consola (*dashboard*)

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

## Aplicació CRUD bàsica

## Autenticació
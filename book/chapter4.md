# Capítol 4. *Routing*
La configuració del *routing* a Angular permet navegar a través de les diverses pàgines de l'aplicació web, tenint en compte que es tracta d'una SPA (*Single Page Application*) i que, per tant, només existeix un únic fitxer `HTML` (l'index.html).

Així doncs, aquesta nagevació no és un canvi de fitxer `HTML` real (la `URL` base del navegador sempre serà la mateixa), sinó una càrrega i descàrrega dels `HTML` dels diversos components que constitueixen les diferents pàgines de l'aplicació. És a dir, el *routing* activa tot un mecanisme que, a través de codi `Javascript` permet modificar el `DOM` de la pàgina index.html.

Per fer tot això, es necessita afegir a la `URL` base un conjunt de fragments que, ben configurats, carreguen el component desitjat.

Angular permet dos tipus de *routing*:
* *Routing* clàssic o tradicional
* *Lazy routing*

## *Routing* clàssic o tradicional
El *routing* clàssic o tradicional es caracteritza pel fet que, quan l'usuari accedeix per primer cop a una aplicació web Angular, el mecanisme de *routing* precarrega totes les rutes disponibles.

Això significa que l'arrencada inicial de l'aplicació és més lenta i, en cas que hi hagi moltes rutes a precarregar, pot arribar a ser un petit coll d'ampolla que faci que l'usuari tingui la sensació de què l'aplicació va lenta. Ara però, un cop feta aquesta precàrrega, el canvi entre pàgines és immediat.

Aquest tipus de *routing* és addient en els casos d'aplicacions amb poques rutes configurades, ja que el temps de càrrega inicial es pot considerar inapreciable.

### Configuració bàsica
Per configurar el *routing* clàssic cal seguir els passos següents:
1. Configurar els fragments de les rutes i enllaçar-los als components corresponents
2. Activar el servei de *routing*
3. Crear els enllaços i la navegació als fitxers `HTML` i `TS` dels components

#### Contextualització d'un exemple
Per fer l'explicació, suposarem que tenim una aplicació amb dues pàgines: la pàgina `home`, definida al `HomeComponent`, i la pàgina `about`, definida a l'`AboutComponent`.

#### Configuració dels fragments de les rutes i activació del servei de *routing*
La configuració de les rutes i l'activació del servei de *routing* es realitza al fitxer de configuració de dependències `app.module.ts`.

Primer de tot cal crear un array de rutes (`Routes`), on cada ruta és un objecte `JSON` que, com a mínim, té l'atribut `path` per definir el fragment que es desitja i l'atribut `component` per establir el component que s'ha de carregar.

```typescript
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent }
]
```

L'ordre en què es defineixen les rutes dins d'aquest array és important, ja que l'enrutador d'Angular utilitza la política *first-mach wins*, és a dir, utilitza la primera ruta que coincideix amb el fragment de la `URL`. Per tant, les rutes s'han d'ordenar de més específiques a menys.

Un cop definides les rutes cal importar el mòdul `RouterModule` i configurar-lo amb l'array creat.

```typescript
imports: [
  BrowserModule,
  RouterModule.forRoot(routes)
]
```

El codi complet del fitxer app.module.ts seria el següent:

{% tabs %}
{% tab title="Codi app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { HomeComponent } from './pages/home/home.component';
import { AboutComponent } from './pages/about/about.component';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
];

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    AboutComponent,
    ListComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }


```
{% endtab %}
{% endtabs %}

##### Configuració d'una ruta per defecte i gestió de ruta no trobada
Per configurar la ruta per defecte, el `JSON` ha de tenir tres atributs: el `path`, el `redirectTo` i el `pathMatch`.
1. `path`: serà el fragment buit (l'`URL` només tindrà la part del domini)
2. `redirectTo`: indicarà a quina ruta cal redirigir tot el trànsit per defecte que arribi a l'aplicació
3. `pathMatch`: indica quina política de coincidència s'ha d'aplicar al fragment del `path`7

Així doncs, si s'afegeix una ruta per defecte, l'array de rutes anterior queda de la manera següent:

```typescript
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: '', redirectTo: 'home', pathMatch: 'full'}
]
```

Per gestionar una ruta no trobada (o no accessible) només cal crear un component al qual redirigir l'intent d'accés erroni i configurar la ruta per al `path: '**'`

```typescript
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: '', redirectTo: 'home', pathMatch: 'full'},
  { path: '**', component: PageNotFoundComponent },
]
```

#### Navegació a través d'enllaços i botons
Un cop configurades totes les rutes desitjades només fa falta crear la navegació entre pàgines a través de botos i enllaços.

Com que tota aplicació Angular és una SPA (*Single Page Application*), es necessita un component principal que faci de contenidor de la resta de components/pàgines. Aquest paper se l'endú l'`AppComponent`, el qual, dins del seu codi `HTML`, tindrà l'etiqueta `<router-outlet>`, que és la que fa de contenidor per a la resta de components. Cal veure que l'`AppComponent` pot funcionar com a component *layout*, és a dir, aquell component que defineix els elements comuns al llarg de tota l'aplicació (capçalera, menú, etc.).

Fet això, només cal afegir enllaços i botons per saltar entre components, tenint en compte que l'ús de l'atribut `href` dels enllaços queda totalment prohibit perquè provoca una recàrrega de la pàgina i, per tant, trenca el principi bàsic de l'SPA (reinicia tota l'aplicació perquè la torna a descarregar del servidor). Aquest atribut queda substituït per l'atribut `routerLink` que ofereix Angular i que es pot aplicar a qualsevol etiqueta.

```html
<a [routerLink]="['/home']">Home</a>
```

L'atribut `routerLink` espera rebre un array amb els múltiples fragments de la nova `URL`, els quals poden ser absoluts o relatius. Per exemple:
* Si estem a la pàgina `home` (`HomeComponent`), l'enllaç
```html
<a [routerLink]="['/about']">Home</a>
```
defineix una ruta absoluta i generarà la `URL` `localhost:4200/about`. 
* Si estem a la pàgina `home` (`HomeComponent`), l'enllaç
```html
<a [routerLink]="['about']">Home</a>
```
defineix una ruta relativa i generarà la `URL` `localhost:4200/home/about`.
* L'enllaç 
```html
<a [routerLink]="['/home', gallery]">Home</a>
```
defineix múltiples fragments i crearà la ruta `localhost:4200/home/gallery`.

Pel que fa als botons, la navegació s'haurà de definir a través del seu event `click` i l'ús del *service* `Router` que ofereix Angular; per exemple, el component `AboutComponent` pot definir el botó següent:
```html
<button (click)="navigateToHome()">Home</button>
```
A partir d'aquí, per la navegació per botó haurà d'injectar el *service* `Router` dins del seu constructor, tal com mostra el codi següent:
```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-about',
  templateUrl: './about.component.html',
  styleUrls: ['./about.component.css']
})
export class AboutComponent {

  constructor(private _router: Router) {}
}
```
Com es veurà en els capítols següents, tots els *services*, tant els que ofereix Angular com els que crearem nosaltres segons les necessitats de la nostra aplicació, aconstumen a seguir el patró d'instanciació `Singelton` i, per tant, només exiteix una única instància de cada *service* per a tota l'aplicació. Per poder-ne sol·licitar l'ús, els components o les classes ho han de fer a través del procés d'injecció, el qual defineix un atribut de classe dins de la zona de paràmetres del constructor.

El *service* `Router` té, entre altres, el mètode `navigate`, el qual espera rebre un array de fragments de ruta (exactament com l'atribut `routerLink`). Així doncs, per acabar de configurar la navegació a través del botó de l'exemple, només cal definir la funció `navigateToHome()`, la qual haurà de fer ús del `Router`:

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-about',
  templateUrl: './about.component.html',
  styleUrls: ['./about.component.css']
})
export class AboutComponent {

  constructor(private _router: Router) {}

  navigateToAbout(): void {
    this._router.navigate(['/home']);
  }
}
```

Exemple complet:
{% tabs %}
{% tab title="Codi app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { HomeComponent } from './pages/home/home.component';
import { AboutComponent } from './pages/about/about.component';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
];

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    AboutComponent,
    ListComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
{% endtab %}

{% tab title="Codi app.component.html" %}
```typescrip
<router-outlet></router-outlet>
```
{% endtab %}

{% tab title="Codi home.component.html" %}
```html
<a [routerLink]="['/about']">About</a>
```
{% endtab %}

{% tab title="Codi about.component.html" %}
```html
<button (click)="navigateToHome()">Home</button>
```
{% endtab %}

{% tab title="Codi about.component.ts" %}
```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-about',
  templateUrl: './about.component.html',
  styleUrls: ['./about.component.css']
})
export class AboutComponent {

  constructor(private _router: Router) {}

  navigateToAbout(): void {
    this._router.navigate(['/home']);
  }
}
```
{% endtab %}
{% endtabs %}

### Configuració de subrutes
De subrutes n'hi ha de dos tipus, les estàtiques i les parametritzades i, tot i que la teoria bàsica és la mateixa, les subrutes es tracten a part pel petit increment de complexitat del tractament de les rutes parametritzades.

## *Lazy routing*
El *lazy routing* es caracteritza pel fet que les rutes i, per tant, els components associats, no es carreguen fins que l'usuari hi entra per primer cop (no hi ha cap precàrrega)

Això significa que cada cop que l'usuari entra per primer cop a una pàgina, hi ha un petit lapsus de temps durant el qual el sistema de *routing* fa la càrrega de la nova ruta. Aquest temps d'espera, però, és molt ràpid i, per tant, no és apreciable per l'usuari

Aquest tipus de *routing* és el més addient, especialment en aplicacions web molt grans.

# Capítol 10. *Routing*
La configuració del *routing* a Angular permet navegar a través de les diverses pàgines de l'aplicació web, tenint en compte que es tracta d'una SPA (*Single Page Application*) i que, per tant, només existeix un únic fitxer `HTML` (l'`index.html`).

Així doncs, aquesta nagevació no és un canvi de fitxer `HTML` real (la `URL` base del navegador sempre serà la mateixa), sinó una càrrega i descàrrega dels `HTML` dels diversos *components* que constitueixen les diferents pàgines de l'aplicació. És a dir, el *routing* activa tot un mecanisme que, a través de codi `Javascript` permet modificar el `DOM` de la pàgina `index.html`.

Per fer tot això, es necessita afegir a la `URL` base un conjunt de fragments que, ben configurats, carreguen el *component* desitjat.

Angular permet dos tipus de *routing*:
* *Routing* clàssic o tradicional
* *Lazy routing*

## *Routing* clàssic o tradicional
El *routing* clàssic o tradicional es caracteritza pel fet que, quan l'usuari accedeix per primer cop a una aplicació web Angular, el mecanisme de *routing* precarrega totes les rutes disponibles.

Això significa que l'arrencada inicial de l'aplicació és més lenta i, en cas que hi hagi moltes rutes a precarregar, pot arribar a ser un petit coll d'ampolla que faci que l'usuari tingui la sensació de què l'aplicació va lenta. Ara però, un cop feta aquesta precàrrega, el canvi entre pàgines és immediat.

Aquest tipus de *routing* és addient en els casos d'aplicacions amb poques rutes configurades, ja que el temps de càrrega inicial es pot considerar inapreciable.

### Configuració bàsica
Per configurar el *routing* clàssic cal seguir els passos següents:
1. Definir el contenidor de rutes a partir de l'etiqueta `<router-outlet />`
2. Crear els *components* que representaran les diverses rutes (normalment s'associa una ruta a una pàgina de l'aplicació)
3. Configurar els fragments de les rutes i enllaçar-los als *components* corresponents (creats en l'apartat anterior)
4. Activar el servei de *routing*
5. Crear els enllaços i la navegació als fitxers `HTML` i `TS` dels *components*

#### Contextualització d'un exemple
Per fer l'explicació, suposarem que tenim una aplicació amb dues pàgines: la pàgina `home`, definida al *component* `Home`, i la pàgina `about`, definida al *component* `About`.

A més a més, com és evident, també tindrem el component arrel `App` que, aquest cop, actuarà com a contenidor de rutes i, per tant, només contindrà l'etiqueta `<router-outlet />` en el seu `HTML` i la dependència `RouterOutlet` dins dels `imports` del seu `TS`.

{% tabs %}
{% tab title="Codi app.html" overflow="wrap" lineNumbers="true" %}
  ```html
  <router-outlet />
  ```
{% endtab %}

{% tab title="Codi app.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { Component } from '@angular/core';
  import { RouterOutlet } from '@angular/router';

  @Component({
    selector: 'app-root',
    imports: [RouterOutlet],
    templateUrl: './app.html',
    styleUrl: './app.css'
  })
  export class App {}
  ```
{% endtab %}
{% endtabs %}

#### Configuració dels fragments de les rutes i activació del servei de *routing*
La configuració de les rutes es realitza al fitxer de configuració `app.routes.ts` i el servei de *routing* es configura dins del fitxer `app.config.ts`

Així doncs, primer de tot cal crear un array de rutes (`Routes`) dins d'`app.routes.ts`, on cada ruta és un objecte `JSON` que, com a mínim, té l'atribut `path` per definir el fragment que es desitja i l'atribut `component` per establir el *component* que s'ha de carregar.

{% code title="Codi app.routes.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { Routes } from '@angular/router';

  import { Home } from './view/pages/home/home';
  import { About } from './view/pages/about/about';

  export const routes: Routes = [
    { path: 'home', component: Home },
    { path: 'about', component: About }
  ];
  ```
{% endcode %}

L'ordre en què es defineixen les rutes dins d'aquest array és important, ja que l'enrutador d'Angular utilitza la política *first-mach wins*, és a dir, utilitza la primera ruta que coincideix amb el fragment de la `URL`. Per tant, les rutes s'han d'ordenar de més específiques a menys.

Un cop definides les rutes cal activar el servei de *routing* dins del fitxer de configuració `app.config.ts` i configurar-lo amb l'array creat (aquest pas ja ve fet de manera automàtica en les versions d'Angular més noves).

{% code title="Codi app.config.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZonelessChangeDetection } from '@angular/core';
  import { provideRouter } from '@angular/router';

  import { routes } from './app.routes';

  export const appConfig: ApplicationConfig = {
    providers: [
      provideBrowserGlobalErrorListeners(),
      provideZonelessChangeDetection(),
      provideRouter(routes)                       //Activació del servei de routing
    ]
  };
  ```
{% endcode %}

##### Configuració d'una ruta per defecte i gestió de ruta no trobada
Dins del fitxer `app.routes.ts` també es pot crear una ruta per defecte (la pàgina que s'ha d'obrir en cas que la `URL` no especifiqui cap fragment de ruta) i una ruta per a gestionar els fragments no trobats. Per fer-ho, el `JSON` de l'objecte `Route` que cal definir al fitxer de configuració `app.routes.ts` ha de contenir tres atributs: el `path`, el `redirectTo` i el `pathMatch`.
1. `path`: serà el fragment buit (l'`URL` només tindrà la part del domini)
2. `redirectTo`: indicarà a quina ruta cal redirigir tot el trànsit per defecte que arribi a l'aplicació
3. `pathMatch`: indica quina política de coincidència s'ha d'aplicar al fragment del `path`

Així doncs, si s'afegeix una ruta per defecte, l'array de rutes mostrat en l'exemple de l'apartat anterior queda de la manera següent:

```typescript
export const routes: Routes = [
  { path: 'home', component: Home },
  { path: 'about', component: About },
  { path: '', redirectTo: 'home', pathMatch: 'full' }
];
```

Per gestionar una ruta no trobada (o no accessible) només cal crear un component al qual redirigir l'intent d'accés erroni, per exemple, el *component* `PageNotFound`, i configurar la ruta per al `path: '**'`

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

Com que tota aplicació Angular és una SPA (*Single Page Application*), es necessita un *component* principal que faci de contenidor de la resta de *components*/pàgines. Tal com hem dit a l'inici de l'explicació (apartat [Contextualització d'un exemple](#contextualització-dun-exemple)), aquest paper se l'endú el *component* `App`, el qual, dins del seu codi `HTML`, tindrà l'etiqueta `<router-outlet />`, que és la que fa de contenidor per a la resta de *components*. Cal veure que el *component* `App` pot funcionar com a *component* *layout*, és a dir, aquell *component* que defineix els elements comuns al llarg de tota l'aplicació (capçalera, menú, etc.).

Fet això, només cal afegir enllaços i botons per saltar entre *components*, tenint en compte que **l'ús de l'atribut `href` dels enllaços queda totalment prohibit perquè provoca una recàrrega de la pàgina** i, per tant, trenca el principi bàsic de l'SPA (reinicia tota l'aplicació perquè la torna a descarregar del servidor). Aquest atribut queda substituït per l'atribut `routerLink` que ofereix Angular dins del *module* `RouterModule` i que es pot aplicar a qualsevol etiqueta.

```html
<a [routerLink]="['/home']">Home</a>
```

L'atribut `routerLink` espera rebre un array amb els múltiples fragments de la nova `URL`, els quals poden ser absoluts o relatius. Per exemple:
* Si estem a la pàgina `home` (*component* `Home`), l'enllaç
```html
<a [routerLink]="['/about']">About</a>
```
defineix una ruta absoluta i generarà la `URL` `localhost:4200/about`. 
* Si estem a la pàgina `home` (*component* `Home`), l'enllaç
```html
<a [routerLink]="['about']">About</a>
```
defineix una ruta relativa i generarà la `URL` `localhost:4200/home/about`.
* L'enllaç 
```html
<a [routerLink]="['/home', gallery]">Home</a>
```
defineix múltiples fragments i crearà la ruta `localhost:4200/home/gallery`.

Pel que fa als botons, la navegació s'haurà de definir a través del seu event `click` i l'ús del *service* `Router` que ofereix Angular; per exemple, el *component* `About` pot definir el botó següent:
```html
<button (click)="navigateToHome()">Home</button>
```

A partir d'aquí, per la navegació per botó haurà d'injectar el *service* `Router` (dins del seu constructor o utilitzant el mètode `inject()`), tal com mostra el codi següent:
{% code title="Codi about.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { Component, inject } from '@angular/core';
  import { Router } from '@angular/router';

  @Component({
    selector: 'app-about',
    imports: [],
    templateUrl: './about.html',
    styleUrl: './about.css'
  })
  export class About {
    //private _router: Rounter = inject(Router);    //Injecció amb el mètode inject()
    constructor(private _router: Router) {}       //Injecció dins del constructor
  }
  ```
{% endcode %}

Com es veurà en els capítols següents, tots els *services*, tant els que ofereix Angular com els que crearem nosaltres segons les necessitats de la nostra aplicació, aconstumen a seguir el patró d'instanciació `Singelton` i, per tant, només exiteix una única instància de cada *service* per a tota l'aplicació. Per poder-ne sol·licitar l'ús, els *components* o les classes ho han de fer a través del procés d'injecció, el qual
* o defineix un atribut de classe dins de la zona de paràmetres del constructor;
* o defineix un atribut de classe a la zona d'atributs i utilitza el mètode `inject()` per a invocar el *service* desitjat

El *service* `Router` té, entre altres, el mètode `navigate()`, el qual espera rebre un array de fragments de ruta (exactament com l'atribut `routerLink`). Així doncs, per acabar de configurar la navegació a través del botó de l'exemple, només cal definir la funció `onNavigateToHome()`, la qual haurà de fer ús del `Router`:

{% code title="Codi about.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { Component, inject } from '@angular/core';
  import { Router } from '@angular/router';

  @Component({
    selector: 'app-about',
    imports: [],
    templateUrl: './about.html',
    styleUrl: './about.css'
  })
  export class About {
    //private _router: Rounter = inject(Router);    //Injecció amb el mètode inject()
    constructor(private _router: Router) {}       //Injecció dins del constructor

    public onNavigateToHome() {
      this._router.navigate(["/home"]);
    }
  }
  ```
{% endcode %}

Exemple complet:
{% tabs %}
{% tab title="Codi app.config.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZonelessChangeDetection } from '@angular/core';
  import { provideRouter } from '@angular/router';

  import { routes } from './app.routes';

  export const appConfig: ApplicationConfig = {
    providers: [
      provideBrowserGlobalErrorListeners(),
      provideZonelessChangeDetection(),
      provideRouter(routes)
    ]
  };
  ```
{% endtab %}

{% tab title="Codi app.routes.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
  import { Routes } from '@angular/router';

  import { Home } from './view/pages/home/home';
  import { About } from './view/pages/about/about';
  import { PageNotFound } from './view/pages/page-not-found/page-not-found';

  export const routes: Routes = [
      { path: 'home', component: Home },
      { path: 'about', component: About },
      { path: '', redirectTo: 'home', pathMatch: 'full' },
      { path: '**', component: PageNotFound }
  ];
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

A part d'això, la càrrega d'una subruta es pot tractar com una pàgina addicional o com un element nou dins de la pàgina que ja s'estava visitant.

#### Subrutes estàtiques
Són aquelles rutes que tenen una `URL` constant (els segments estan predefinits) i, per tant, sempre carreguen el mateix contingut.

Per exemplificar-ho, suposem que a la pàgina d'inici hi posem un nou enllaç que, en preme'l, mostri un formulari de contacte. La `URL` inicial serà `localhost:4200/home` i la que mostrarà el formulari serà `localhost:4200/home/contact.` Aquest nou formulari es pot mostrar com un nou element dins de la pàgina inicial o com una pàgina individual.

##### Subrutes estàtiques que formen part d'una altra pàgina
Seguint l'exemple anterior, la configuració de les rutes al fitxer `app.module.ts` seria el següent:

```typescript
...
const routes: Routes = [
  { path: 'home', component: HomeComponent, children: [
      {path: 'contact', component: ContactComponent}
    ]
  },
  { path: 'about', component: AboutComponent },
  ...
];

@NgModule({
  ...
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  ...
})
export class AppModule { }
```

Aquest codi mostra una ruta principal definida pel `path: 'home'` que té una subruta especificada amb la propietat `children`. Quan l'usuari entri a l'`URL` `localhost:4200/home` es carregarà el component `HomeComponent` i, en canvi, quan entri a l'`URL` `localhost:4200/home/contact` es carregaran els components `HomeComponent` i, també, `ContactComponent`.

Per tal d'assolir aquest objectiu, el codi `HTML` del component `HomeComponent` ha de ser el següent (navegant amb enllaços, és clar):

```html
<div class="home_content">
  <a [routerLink]="['/about']">About</a><br/>
  <a [routerLink]="['contact']">Contact</a>

  <router-outlet></router-outlet>
</div>
```

Quan l'usuari premi l'enllaç *Contact*, la subruta `/home/contact` es carregarà en el contenidor de routes (etiqueta *router-outlet*) del `HomeComponent`. D'aquesta manera aconseguim visualitzar, a la vegada, el `HomeComponent` (carregat a través del *router-outlet* de l'`AppComponent`) i el `ContactComponent` (carregat a través del *router-outlet* del `HomeComponent`).

Si el contingut del component `ContentComponent` és el que es mostra a continuació:

```html
<div class="contact_content">
  <label>eMail</label>
  <input type="email" placeholder="User's email"/> <br/>
  
  <label>Message</label>
  <textarea placeholder="User's message"></textarea>
</div>
```

El resultat final en pantalla mostra el següent:

![Visualització del resultat al navegador](img/subrouting_1.png)

##### Subrutes estàtiques que són pàgines diferents
Seguint amb el mateix exemple, la configuració de les rutes al fitxer `app.module.ts` és lleugerament diferent a l'anterior versió:

```typescript
...
const routes: Routes = [
  { path: 'home', children: [
      {path: '', component: HomeComponent},
      {path: 'contact', component: ContactComponent}
    ]
  },
  { path: 'about', component: AboutComponent },
  ...
];

@NgModule({
  ...
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  ...
})
export class AppModule { }
```

En aquest cas, la ruta `home` té dues subrutes:
 - una ruta per defecte que és l'encarregada de carregar el `HomeComponent` i
 - una ruta amb el nou segment `contact` que s'encarrega de carregar el `ContactComponent`.

A més a més, per assolir l'objectiu, el document `HTML` de `HomeComponent` ja no ha de tenir l'etiqueta *router-outlet*

```html
<div class="home_content">
  <a [routerLink]="['/about']">About</a><br/>
  <a [routerLink]="['contact']">Contact</a>
</div>
```

Fent aquests petits canvis, totes les pàgines es carreguen al contenidor *router-outlet* que conté `AppComponent` i, per tant, el resultat en pantalla és el següent:

![Visualització del resultat al navegador](img/subrouting_2.png)

### Subrutes parametritzades
Són aquelles rutes que tenen una `URL` variable, és a dir, els segments estan parametritzats i, per tant, són capaces de canviar el contingut mostrat depenent d'aquests paràmetres.

Per exemplificar-ho, suposem que afegim una pàgina que mostra un llistat d'elements. Cada cop que l'usuari premi un d'aquests elements es mostrarà una nova ruta amb totes les seves dades detallades.La `URL` de la llista serà `localhost:4200/list` i la que mostrarà els detalls de cadascun dels elements `localhost:4200/home/0.`, on `0` és el paràmetre i serà un identificador o un valor que estigui enllaçat a l'element que volem visualitzar.

Addicionalment, i tal com passa amb les subrutes estàtiques, la pàgina amb la informació detallada dels elements es pot mostrar dins del llistat o com una pàgina individual.

##### Subrutes parametritzades que formen part d'una altra pàgina
Seguint l'exemple anterior, la configuració de les rutes al fitxer `app.module.ts` seria el següent:

```typescript
...
const routes: Routes = [
  { path: 'home', component: HomeComponent, children: [
      {path: 'contact', component: ContactComponent}
    ]
  },
  { path: 'about', component: AboutComponent },
  { path: 'list', component: ListComponent, children: [
      {path: ':id', component: DetailsComponent}
    ]
  }
  ...
];

@NgModule({
  ...
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  ...
})
export class AppModule { }
```

Aquest codi mostra una ruta principal definida pel `path: 'list'` que té una subruta parametritzada especificada amb la propietat `children`. Quan un `path` de ruta començar pel símbol `:` significa que el valor és un paràmetre que rep el nom especificat a continuació. En el cas de l'exemple, el paràmetre rep el nom d'`id`.

Quan l'usuari entri a l'`URL` `localhost:4200/list` es carregarà el component `ListComponent` i, en canvi, quan entri a l'`URL` `localhost:4200/list/3` es carregaran els components `ListComponent` i, també, `DetailsComponent`.

Per tal d'assolir aquest objectiu, el codi `HTML` del component `ListComponent` ha de ser el següent (navegant amb enllaços):

{% tabs %}
{% tab title="Codi list.component.html" %}
```html
<div class="list_content">
  <ul>
    <li *ngFor="let elem of elems; let idx=index" [routerLink]="[idx]">
      {{ elem }}
    </li>
  </ul>

  <router-outlet></router-outlet>
</div>
```
{% endtab %}

{% tab title="Codi list.component.ts" %}
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {

  public elems: string[] = ['Element 1', 'Element 2', 'Element 3'];

}
```
{% endtab %}
{% endtabs %}

Quan l'usuari premi l'enllaç associat a qualsevol dels elements, la subruta `/list/:id` es carregarà en el contenidor de routes (etiqueta *router-outlet*) del `ListComponent`. D'aquesta manera aconseguim visualitzar, a la vegada, el `ListComponent` (carregat a través del *router-outlet* de l'`AppComponent`) i el `DetailsComponent` (carregat a través del *router-outlet* del `ListComponent`).

Ara cal veure com tracta el paràmetre el component `DetailsComponent` sabent que, el seu contingut `HTML` és el següent:

```html
<div class="details_content">
  Element {{ idx }}
</div>
```

Per tal d'obtenir l'identificador correcte, el codi *typescript* del component `DetailsComponent` ha de fer ús d'un nou *service* que ofereix Angular, l'`ActivatedRoute`, que s'encarrega de tractar totes i cadascuna de les rutes en el moment en què s'activen (l'usuari les escriu al navegador).

```typescript
import { Component } from '@angular/core';
import { ActivatedRoute, Params } from '@angular/router';

@Component({
  selector: 'app-details',
  templateUrl: './details.component.html',
  styleUrls: ['./details.component.css']
})
export class DetailsComponent {

  public idx: number = 0;

  constructor(private _activatedRoute: ActivatedRoute) {
    this._activatedRoute.params.subscribe({
      next: (params: Params) => {
        this.idx = params['id'] as number;
        this.idx++;
      },
      complete: () => {
        console.log("Parameterized route processed");
      },
      error: (msg: string) => {
        console.log("Error: " + msg);
      }
    });
  }
}
```

Aquest codi segueix els passos següents:
 1. Injecta el servei `Activated route` dins del constructor
 2. Un dels atributs que té aquest servei és el `params`, que és un *map* (un *diccionari* o un *array associatiu*) que compleix el patró `Observer`, és a dir, que es manté *observant* la ruta i, cada cop que els paràmetres canvien, es modifiquen el valors d'aquest atribut.
 3. Com que estem treballant amb el patró `Observer`, caldrà *subscriure'ns* (subscribe) a l'atribut `params` per tal que poder-nos assabentar de tots els seus canvis.

El resultat final en pantalla mostra el següent:

![Visualització del resultat al navegador](img/subrouting_3.png)

##### Subrutes parametritzades que són pàgines diferents
Tal com hem vist abans, per fer subrutes parametritzades que siguin pàgines diferents només cal canviar la configuració de l'`app.module.ts` i el fitxer `HTML` del component pare, en aquest cas, el `ListComponent`.

Per tant, el fitxer `app.module.ts` queda de la manera següent:

```typescript
...
const routes: Routes = [
  { path: 'home', component: HomeComponent, children: [
      {path: 'contact', component: ContactComponent}
    ]
  },
  { path: 'about', component: AboutComponent },
  { path: 'list', children: [
      {path: '', component: ListComponent},
      {path: ':id', component: DetailsComponent}
    ]
  }
  ...
];

@NgModule({
  ...
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
  ...
})
export class AppModule { }
```

La ruta `list` té dues subrutes:
 - una ruta per defecte que és l'encarregada de carregar el `ListComponent` i
 - una ruta amb el nou segment parametritzat `id` que s'encarrega de carregar el `DetailComponent`.

Finalment, el codi del component `ListComponent` ha de ser el següent, tenint en compte que la navegació es fa amb enllaços i que el fitxer `TS` del component no canvia:

{% tabs %}
{% tab title="Codi list.component.html" %}
```html
<div class="list_content">
  <ul>
    <li *ngFor="let elem of elems; let idx=index" [routerLink]="[idx]">
      {{ elem }}
    </li>
  </ul>
</div>
```
{% endtab %}

{% tab title="Codi list.component.ts" %}
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {

  public elems: string[] = ['Element 1', 'Element 2', 'Element 3'];

}
```
{% endtab %}
{% endtabs %}

Amb aquests petits canvis, el resultat final en pantalla mostra el següent:

![Visualització del resultat al navegador](img/subrouting_4.png)

## *Lazy routing*
El *lazy routing* es caracteritza pel fet que les rutes i, per tant, els components associats, no es carreguen fins que l'usuari hi entra per primer cop (no hi ha cap precàrrega)

Això significa que cada cop que l'usuari entra per primer cop a una pàgina, hi ha un petit lapsus de temps durant el qual el sistema de *routing* fa la càrrega de la nova ruta. Aquest temps d'espera, però, és molt ràpid i, per tant, no és apreciable per l'usuari

Aquest tipus de *routing* és el més addient, especialment en aplicacions web molt grans.

### Passos inicials per poder configurar el *lazy routing*
Per tal de poder configurar el *lazy routing*, cada pàgina de la nostra aplicació, és a dir, cada component que pugui ser enrutat, ha de tenir associats dos fitxers:
1. `component_name-routing.module.ts` i 
2. `component_name.module.ts`.

En el cas dep component inicial (`AppComponent`), aquests dos fitxers són l'`app-routing.module.ts` i l'`app.module.ts`.

Si en el moment de crear el nou projecte Angular ja indiquem que volem gestionar el routing mitjançant la comanda
```bash
  ng new project_name --routing true
```
ja es creen els dos fitxers. En canvi, si volem afegir *lazy routing* a un projecte Angular ja existent, el primer que caldrà fer és crear el fitxer `app-routing.module.ts`. Per fer-ho, només cal executar la següent comanda:
```bash
  ng generate module app-routing --module app --flat
```
Això és crearà el fitxer que falta i actualitzarà els `imports` de l'`app.module.ts` correctament:
```typescript:highlight={5,13}:title="Actualització app.module.ts"
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { AppRoutingModule } from './app-routing.module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Addicionalment, però, cal modificar el mòdul `app-routing.module.ts` per tal de configurar-lo per acceptar les rutes, essent el seu codi el següent:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

#### Contextualització d'un exemple
Per fer l'explicació seguirem el mateix exemple que s'ha utilitzat en l'explicació del *routing* tradicional, dues pàgines: la pàgina `home`, definida al `HomeComponent`, i la pàgina `about`, definida a l'`AboutComponent`.

#### Configuració dels fragments de les rutes i activació del servei de routing
La ruta `home` i la ruta `about` són rutes arrel o principals, per tant, han de quedar configurades dins del fitxer `app-routing.module.ts`.

Aquesta configuració es pot fer a mà o de manera automàtica mitjançant el terminal, per tant, en comptes de generar els components amb la comanda
```bash
ng generate component path/component_name --skip-tests
```
ho farem amb la comanda
```bash
ng generate module path/component_name  --route route_name --module app.module
```
Aquesta instrucció genera el següent:
1. El component `component_name`, és a dir, els seus fitxers `HTML`, `TS` i `CSS`
2. El mòdul `component_name.module.ts`
3. El mòdul d'enrutament `component_name-routing.module.ts`
A més a més, configura els fitxers `app-routing.module.ts`, `component_name.module.ts` i `component_name-routing.module.ts` de la manera correcta per treballar amb *lazy routing*.

En el cas que s'exemplifica, les comandes seran les següents:
```bash
ng generate module view/pages/home  --route home --module app.module
ng generate module view/pages/about  --route about --module app.module
```
Les quals, a part dels dos components, actualitzen els fitxers de la manera següent:
{% tabs %}
{% tab title="Codi app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { AppRoutingModule } from './app-routing.module';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
{% endtab %}

{% tab title="Codi app-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', loadChildren: () => import('./view/pages/home/home.module').then(m => m.HomeModule) },
  { path: 'about', loadChildren: () => import('./view/pages/about/about.module').then(m => m.AboutModule) }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
{% endtab %}

{% tab title="Codi home.module.ts (equivalent en el cas del component about)" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { HomeRoutingModule } from './home-routing.module';
import { HomeComponent } from './home.component';


@NgModule({
  declarations: [
    HomeComponent
  ],
  imports: [
    CommonModule,
    HomeRoutingModule
  ]
})
export class HomeModule { }
```
{% endtab %}

{% tab title="Codi home-routing.module.ts (equivalent en el cas del component about)"%}
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';

const routes: Routes = [
  { path: '', component: HomeComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomeRoutingModule { }
```
{% endtab %}
{% endtabs %}

Interpretació de tot aquest codi:
1. A diferència del *routing* tradicional, els components ja no s'importen a l'`app.module.ts`, sinó que s'importen en el seu propi mòdul (`home.module.ts` i `about.module.ts` respectivament).
2. L'`app.module.ts` importa l'`app-routing.module.ts` i, per tant, permet realitar la càrrega de les diverses rutes existents.
3. L'`app-routing.module.ts` no defineix les rutes directament amb la propietat `component` (això és el que fa el *routing* tradicional), sinó que defineix la propietat `loadChildren` per indicar quin mòdul cal carregar en el moment d'accedir a aquella ruta.
4. La ruta definitiva, la que defineix el `component` que cal activar, es troba al fitxer de *routing* específic (`home.module.ts` i `about.module.ts` respectivament), definida sobre el `path` buit (perquè el fragment de ruta ja ha quedat definit a l'`app-routing.module.ts`).

Així doncs, en el moment en que l'usuari demana una ruta per primer cop, Angular la carrega seguint el següent patró:
1. A través de l'`app.module.ts` sap que les rutes estan definides a l'`app-routing.module.ts`
2. Va a buscar la ruta a l'`app-routing.module.ts` i mira quin mòdul ha de carregar (posem, per exemple que s'ha activat la ruta `home` i, per tant, cal activar el `home.module.ts`).
3. Accedeix al mòdul específic (`home.module.ts`) i importa el component (`HomeComponent`) i descobreix el fitxer de *routing* (`home-routing.module.ts`) que cal carregar a continuació.
4. Va a buscar la ruta al fitxer de *routing* (`home-routing.module.ts`) i, finalment, mostra per pantalla el `component` que s'hi indica (`HomeComponent`), el qual ja ha estat importat en el pas anterior.

##### Configuració d'una ruta per defecte i gestió de ruta no trobada
La ruta per defecte s'ha de configurar a mà dins del fitxer `app-routing.module.ts`, seguint les mateixes normes que en el cas del *routing* tradicional. En canvi, per gestionar una ruta no trobada, s'ha de fer mitjançant la comanda indicada anteriorment:
```bash
ng generate module view/pages/page-not-found  --route '**' --module app.module
```

El fitxer `app-routing.module.ts` quedarà de la manera següent:
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', loadChildren: () => import('./view/pages/home/home.module').then(m => m.HomeModule) },
  { path: 'about', loadChildren: () => import('./view/pages/about/about.module').then(m => m.AboutModule) },
  { path: '', redirectTo: 'home', pathMatch: 'full'},
  { path: '**', loadChildren: () => import('./view/pages/page-not-found/page-not-found.module').then(m => m.PageNotFoundModule) }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

#### Navegació a través d'enllaços i botons
La navegació mitjançant enllaços i botons en el *lazy routing* es fa exactament igual que en el cas del *routing* tradicional:
1. Cal posar l'etiqueta `<router-outlet></router-outlet>` a l'`app.component.html` per definir el contenidor de rutes
2. Per navegar mitjançant enllaços s'ha d'utilitzar la propietat `routerLink` per definir la ruta (mai utilitzar `href`!)
3. Per navegar mitjançant botons, es pot utilitzar la propietat `routerLink` o el *service* `Router` a través de l'*Event Binding* de l'esdeveniment *clic*

### Configuració de subrutes
En el *lazy routing* tenim, exactament, la mateixa casuística de tipus de subrutes que en el cas del *routing* tradicional:
* Subrutes estàtiques
 - Subrutes estàtiques que formen part d'una altra pàgina (dependents)
 - Subrutes estàtiques que són pàgines diferents (independents)
* Subrutes parametritzades
 - Subrutes parametritzades que formen part d'una altra pàgina (dependents)
 - Subrutes parametritzades que són pàgines diferents (independents)

L'únic que canvia respecte del *routing* tradicional és la manera com configurem els `routing.module` corresponents; la resta (etiquetes `<router-outlet>`, navegació, captació dels paràmetres amb el *service* `ActivatedRoute`) funciona exactament igual.

#### Subrutes estàtiques
Per fer l'explicació seguirem el mateix exemple que en el cas del *routing* tradicional i crearem una pàgina `contact` que pengi de la pàgina `home`.

##### Subrutes estàtiques que formen part d'una altra pàgina (dependents)
Si volem crear una pàgina `contact` subruta de la pàgina `home` (`contact` formarà part de la pàgina `home`) caldrà seguir els passos següents:
1. El fitxer `home.component.html` haurà d'incloure l'etiqueta `<router-outlet>`
2. El nou component es crearà a partir de la mateixa comanda que en el cas anterior
```bash
ng generate module view/pages/home/contact  --route contact --module view/pages/home/home.module
```
Fixeu-vos que aquesta comanda indica que el component i la ruta `contact` depenen del mòdul `home.module.ts`, per tant, ja no és una ruta principal (les principals són les que apareixen a `app-routing.module.ts`), sinó una ruta niada (`/home/contact`).
3. Cal modificar el fitxer `home-routing.module.ts` per tal que les rutes siguin dependents l'una de l'altra mitjançant la propietat `children` i, a més a més, aconseguir que tots dos components es mostrin en pantalla en cas que l'usuari activi la ruta `/home/contact`:
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';

const routes: Routes = [
  { path: '', component: HomeComponent, children:[
    { path: 'contact', loadChildren: () => import('./contact/contact.module').then(m => m.ContactModule) }
  ]},
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomeRoutingModule { }
```

##### Subrutes estàtiques que són pàgines diferents (independents)
Seguint el mateix exemple, si volem crear una pàgina `contact` subruta de la pàgina `home`, però essent pàgines independents, caldrà crear el nou component a partir de la següent comanda
```bash
ng generate module view/pages/home/contact  --route contact --module view/pages/home/home.module
```
Fet això, el mòdul `home-routing.module.ts` queda de la manera següent:
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'contact', loadChildren: () => import('./contact/contact.module').then(m => m.ContactModule) }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomeRoutingModule { }
```
Aquest codi activa la ruta `/home/contact`, essent `home` i `contact` dues pàgines independents visualment.

Una altra manera vàlida d'activar la mateixa ruta i que depèn de l'estil del programador (a uns els agradarà més d'una manera i a altres d'una altra) s'aconsegueix modificant el `home-routing.module.ts` de la manera següent:
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';

const routes: Routes = [
  { path: '', children: [
    { path: '', component: HomeComponent }, 
    { path: 'contact', loadChildren: () => import('./contact/contact.module').then(m => m.ContactModule) }
  ]},
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomeRoutingModule { }
```
Totes dues opcions funcionen correctament; la segona, però, s'assembla més a la versió del *routing* tradicional.

### Subrutes parametritzades
Per seguir l'explicació d'aquest apartat, també seguirem el mateix exemple que en el cas de *routing* tradicional i, per tant, suposarem que tenim una pàgina que mostra un llistat d'elements. Cada cop que l'usuari premi un d'aquests elements es mostrarà una nova ruta amb totes les seves dades detallades. La URL de la llista serà `localhost:4200/list` i la que mostrarà els detalls de cadascun dels elements `localhost:4200/home/0`, on 0 és el paràmetre i serà un identificador o un valor que estigui enllaçat a l'element que volem visualitzar.

Per crear la ruta `list` haurem executat la comanda:
```bash
ng generate module view/pages/list  --route list --module app.module
```
la qual haurà generat el component `ListComponent` amb el seus mòduls (`list.module.ts` i `list-routing.module.ts`) i també haurà modificat el mòdul `app-routing.module.ts` de la manera següent:
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', loadChildren: () => import('./view/pages/home/home.module').then(m => m.HomeModule) },
  { path: 'about', loadChildren: () => import('./view/pages/about/about.module').then(m => m.AboutModule) },
  { path: 'list', loadChildren: () => import('./view/pages/list/list.module').then(m => m.ListModule) },
  { path: '', redirectTo: 'home', pathMatch: 'full'},
  { path: '**', loadChildren: () => import('./view/pages/page-not-found/page-not-found.module').then(m => m.PageNotFoundModule) }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

##### Subrutes parametritzades que formen part d'una altra pàgina
Per configurar la ruta s'han de seguir els mateixos passos que en el cas de les rutes estàtiques que formen part d'una altra pàgina. Ara però, en aquest cas, la comanda a executar serà la següent:
```bash
ng generate module view/pages/list/details  --route ':id' --module view/pages/list/list.module
```

##### Subrutes parametritzades que són pàgines diferents
Per configurar la ruta s'han de seguir els mateixos passos que en el cas de les rutes estàtiques que són pàgines diferents. Ara però, en aquest cas, la comanda a executar serà la següent:
```bash
ng generate module view/pages/list/details  --route ':id' --module view/pages/list/list.module
```

## Remarcar l'enllaç del menú corresponent a la ruta activa
En els menús de les nostres pàgines i aplicacions web és molt útil deixar remarcat el botó o l'enllaç corresponent a la ruta que hi ha activa en cada moment. Per fer-ho podem utilitzar l'atribut `routerLinkActive`, proporcionat per Angular. Aplicant un *property binding* a aquest atribut es pot definir l'estil que cal aplicar quan es detecti que la `URL` conté la ruta a la qual fa referència.

Seguint amb l'exemple d'aquest capítol, afegim el `HeaderComponent` per fer un menú comú a totes les pàgines (quedarà incrustat a l'`HTML` de l'`AppComponent`), de tal manera que en definim el seu codi `HTML` i `CSS` de la manera següent:

{% tabs %}
{% tab title="Codi header.component.html" %}
```html
<header>
    <h3>Capçalera</h3>

    <a [routerLink]="['/home']" [routerLinkActive]="['activatedLink']">Home</a>
    <a [routerLink]="['/about']" [routerLinkActive]="['activatedLink']">About</a>
    <a [routerLink]="['/list']" [routerLinkActive]="['activatedLink']">List</a>
</header>
```
{% endtab %}

{% tab title="Codi header.component.css" %}
```css
header {
    border: #AAAAAA 5px solid;
    margin-bottom: 32px;
}

h3 {
    text-align: center;
}

a{
    margin-right: 100px;
}

.activatedLink {
    color: #AAAAAA;
}
```
{% endtab %}
{% endtabs %}

A la següent imatge podem veure com, cada cop que es canvia de pàgina, l'enllaç del menú ressaltat en gris correspon a la ruta activada.

![Visualització del resultat al navegador](img/activeLink.png)
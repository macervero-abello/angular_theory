# Capítol 14. Components de navegació amb Ionic
Ionic ofereix una gran quantitat de components d'interfície gràfica i una [documentació](https://ionicframework.com/docs/components) molt similar a altres frameworks, com, per exemple, Bootstrap, que en facilita molt el seu ús. Tot i això, el primer que cal aprendre és a gestionar-ne el sistema de navegació.

En tota aplicació mòbil (i web) existeixen tres mètodes de navegació bàsics:
1. Mitjançant botons (i enllaços)
2. Mitjançant un menú
3. Mitjançant pestanyes

A continuació s'expliquen les diverses metodologies

## *Routing*
Tal com passa amb qualsevol aplicació Angular, el component `AppComponent` fa de contenidor principal de les diverses pàgines que es mostren a través del sistema d'enrutament. Ara però, aquest cop, l'etiqueta encarregada de generar aquest contenidor no és la `<router-outlet></router-outlet>`, sinó que l'estructura `HTML` és la següent:
```html
    <ion-app>
        <ion-router-outlet></ion-router-outlet>
    </ion-app>
```

L'etiqueta `<ion-app>` s'encarrega d'iniciarlitzar la part gràfica de l'aplicació i l'etiqueta `<ion-router-outlet>` actua de contenidor de rutes.

A part d'aquesta diferència, també cal tenir en compte dos aspectes addicionals que ja s'han comentat en el capítol anterior:
1. l'enrutament per defecte que defineix Ionic és el *Lazy Routing* i
2. per generar una nova pàgina no crearem un `component`, sinó una `page`. Al cap i a la fi, una `page` no és res més que un `component`, però Ionic diferencia el concepte pàgina (`page`), com tot l'`HTML` que es mostra per una ruta en concret, i element d'una pàgina (`component`), com l'`HTML` corresponent a una part de la pàgina (una llista, la capçalera, el peu de pàgina, etc.). La comanda que cal utilitzar és la de `ionic generate page path/page_name`

## Navegació amb botons (i enllaços)
La navegació mitjançant botons i enllaços es fa exactament igual que en Angular, és a dir, mitjançant
* la propietat `routerLink` o
* programaticament utilitzant el *service* `Router`.

Així doncs, suposem que tenim dues pàgines `HomePage` i `AboutPage` amb les rutes `/home` i `/about` definides respectivament i de manera automàtica per cadascuna d'elles en el moment d'executar les comenades següents:
```bash
ionic generate page view/pages/home
ionic generage page view/pages/about
```
Podem navegar d'una pàgina a l'altra de la manera que mostra el codi següent:

{% tabs %}
{% tab title="Codi home.page.html" %}
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>Home</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
    <ion-button [routerLink]="['/about']">About</ion-button>
</ion-content>
```
{% endtab %}

{% tab title="Codi about.page.html" %}
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>About</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-button (click)="goToHome()">Home</ion-button>
</ion-content>
```
{% endtab %}

{% tab title="Codi about.page.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-about',
  templateUrl: './about.page.html',
  styleUrls: ['./about.page.scss'],
})
export class AboutPage implements OnInit {

  constructor(private _router: Router) { }

  ngOnInit() { }

  goToHome(): void {
    this._router.navigate(['/home']);
  }

}
```
{% endtab %}
{% endtabs %}

## Navegació amb menú
Moltes aplicacions mòbils, per no dir la majoria, estan gestionades per un menú principal que permet navegar a través de totes les pàgines. Per tant, com que aquest menú ha d'estar present (i ha de ser visible) des de qualsevol punt de l'aplicació, el component encarregat de gestionar-lo és l'`AppComponent`.

Suposem que tenim tres pàgines, la `HomePage`, la `ListPage` i l'`AboutPage`, amb les rutes `/home`, `/list` i `/about` definides respectivament i de manera automàtica per cadascuna d'elles en el moment d'executar les comenades següents:
```bash
ionic generate page view/pages/home
ionic generate page view/pages/list
ionic generage page view/pages/about
```
El fitxer `app.component.html`té l'aspecte següent:

```html
<ion-app>

  <ion-menu contentId="main-content">
    <ion-header>
      <ion-toolbar>
        <ion-title>Menú</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding">
      <ion-list>
        <ion-menu-toggle>
          <ion-item [routerLink]="['/home']" [routerLinkActive]="['activatedLink']">
            <ion-label><ion-icon name="home"></ion-icon> Home</ion-label>
          </ion-item>
        </ion-menu-toggle>
        <ion-menu-toggle>
          <ion-item [routerLink]="['/list']" [routerLinkActive]="['activatedLink']">
            <ion-label>List</ion-label>
          </ion-item>
        </ion-menu-toggle>
        <ion-menu-toggle>
          <ion-item [routerLink]="['/about']" [routerLinkActive]="['activatedLink']">
            <ion-label>About</ion-label>
          </ion-item>
        </ion-menu-toggle>
      </ion-list>
    </ion-content>
  </ion-menu>
  
  <ion-router-outlet id="main-content"></ion-router-outlet>
</ion-app>
```

Dins de l'etiqueta `<ion-app>` es defineix tant el menú `<ion-menu>` com el contenidor de rutes `<ion-router-outlet>`, de tal manera que cal entendre els següents aspectes:
1. L'etiqueta `<ion-menu>` sempre ha de definir la propietat `contentId`, el valor de la qual ha de ser igual a la propietat `id` de la l'etiqueta `<div>` o del contenidor de rutes `<ion-router-outlet>` on es vol carregar la pàgina que l'usuari vol navegar.
2. Un menú no és res més que una "mini pàgina" i, per tant, està format per les etiquetes `<ion-header>` i `<ion-content>`.
3. Normalment, dins de l'`<ion-content>` es crea un llistat amb totes les pàgines on es vol poder navegar. Aquest llistat està definit per l'etiqueta `<ion-list>`, la qual conté un conjunt d'`<ion-menu-toggle>` amb un `<ion-item>` en el seu interior. El contingut dels `<ion-item>` pot ser més o menys complex (només text, només icones, text i icones, etc.). L'etiqueta `<ion-menu-toggle>` és la que permet tancar el menú automàticament tan aviat com l'usuari a premut una de les opcions per canviar de pàgina.
4. Sempre cal tenir present que tota la navegació es farà a través de la propietat `routerLink` i mai mitjançant un `href`. A més a més, per augmentar la usabilitat i l'accessibilitat de l'aplicació es pot utilitzar la propietat `routerLinkActive` per ressaltar la pàgina on es troba, actualment, l'usuari. En el cas de l'exemple, la classe `activatedLink` està definida al fitxer `app.component.scss` de la manera següent:
```scss
.activatedLink {
    color: var(--ion-color-primary)
}
```
on `--ion-colo-primary` és una variable CSS que ofereix Ionic per definir el seu color primary (blau) i per tal de poder-ne agafar el seu valor cal utiltizar el mètode `var()`.

Aquest exemple crea el menú que es mostra en la següent imatge

![Visualització del menú creat en l'exemple d'aquest apartat](img/ionic_menu_example.png)

Un cop fet el menú, cal modificar les capçaleres de les diverses pàgines per tal d'afegir-hi el botó que permeti obrir-lo i tancar-lo segons la necessitat de l'usuari. Així doncs, el codi `HTML` de `HomePage`, per exemple, queda de la següent manera:
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-buttons slot="start">
      <ion-menu-button></ion-menu-button>
    </ion-buttons>
    <ion-title>Home</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-button [routerLink]="['/about']">About</ion-button>
</ion-content>
```
Es pot veure que s'afegeix una botonera a la part esquerra de la capçalera (`slot="start"`) amb la típica icona del menú (les tres barres). Visualment queda de la manera següent:

![Visualització del botó de menú](img/ionic_menu_icon.png)

### Múltiples menús i menús secundaris
Ionic preveu que una aplicació pugui tenir múltiples menús o que, fins i tot, en pugui tenir un de secundari, és a dir, que no gestioni la navegació principal, sinó només un petita part o un apartat de tota l'aplicació.

En cas que es vulgui crear múltiples menús per una mateixa aplicació, tot i que no és massa aconsellable, caldrà jugar amb diversos `contentId` i diversos `<ion-router-outlet>` per poder carregar les diferents vistes.

En cas que es vulgui crear un menú secundari, aquest menú, s'haurà de definir dins de l'`HTML` de la pàgina que s'encarregarà de fer-ne la gestió (ja no estarà definit a l'`app.component.html`).

## Navegació amb pestanyes
La navegació per pestanyes és una mica més complexa que la resta perquè necessita la creació d'una pàgina que gestioni les pestanyes (aquesta funció no la fa l'`AppComponent`) i la modificació de la definició de les rutes per defecte que es generen per tal que tot funcioni correctament.

Així doncs, en cas que la navegació principal de la nostra aplicació estigui guiada per pestanyes, el fitxer `app.component.html` només contindrà les etiquetes `<ion-app>` i `<ion-router-outlet>`, com en el cas de navegació per botons:
```typescript
<ion-app>  
  <ion-router-outlet></ion-router-outlet>
</ion-app>
```
Suposant que volen tenir 3 pestanyes, una per al *Home*, una per a la pàgina *List* i una per a l'*About*, necessitarem crear la infraestructura següent:

![Infraestructura per a la gestió de pestanyes](img/ionic_tabs_structure.png)

Dins de la carpeta `view` hi crearem la pàgina `TabsPage`, la gestora de les pestanyes, i, com a filles de `TabsPage`, crearem les pàgies `HomePage`, `ListPage` i `AboutPage`.  De totes elles, la que cal modificar per crear tota l'estructura de pestanyes en el codi `HTML` és `TabsPage`, per tant, el fitxer `tabs.page.html` queda de la manera següent:
```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="home">
      <ion-icon name="home"></ion-icon>
      Home
    </ion-tab-button>
    <ion-tab-button tab="list">
      <ion-icon name="list-outline"></ion-icon>
      List
    </ion-tab-button>
    <ion-tab-button tab="about">
      <ion-icon name="information-outline"></ion-icon>
      About
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```
Aquest codi crea una botonera a la part inferior de la pàgina (`<ion-tab-bar slot="bottom">`) amb tres pestanyes (`<ion-tab-button>`), les quals estan formades per una icona i un títol. L'atribut `tab` de l'etiqueta `<ion-tab-button>` és la que indica la ruta de la pàgina que volem obrir (en aquest cas no naveguem amb el `routerLink` sinó amb el `tab`). La següent imatge en mostra el resultat:

![Visualització de les pestanyes](img/ionic_tabs.png)

Fet tot això, caldrà modificar les rutes creades per defecte per adaptar-les a les nostres necessitats, tal com mostren els fragments de codi següents:
{% tabs %}
{% tab title="Codi app-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', loadChildren: () => import('./view/tabs/tabs.module').then( m => m.TabsPageModule) },
  { path: '**', loadChildren: () => import('./view/pages/page-not-found/page-not-found.module').then( m => m.PageNotFoundPageModule) }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
{% endtab %}

{% tab title="Codi tabs-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { TabsPage } from './tabs.page';

const routes: Routes = [
  { path: '', component: TabsPage, children: [
    { path: 'home', loadChildren: () => import('./home/home.module').then( m => m.HomePageModule) },
    { path: 'list', loadChildren: () => import('./list/list.module').then( m => m.ListPageModule)  },
    { path: 'about', loadChildren: () => import('./about/about.module').then( m => m.AboutPageModule) },
    { path: '', redirectTo: '', pathMatch: 'full' }
  ]}
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class TabsPageRoutingModule {}
```
{% endtab %}
{% endtabs %}
Si analitzem aquest codi es pot veure que el flux d'enrutament és el següent:
1. L'`app-routing.module.ts` té una ruta per defecte (`path: ''`) que carrega el mòdul `TabsPageModule` i, per tant, acaba carregant el fitxer `tabs-routing.module.ts`.
2. A `tabs-routing.module.ts` es defineix una ruta per defecte que carrega la pàgina `TabsPage` (la que conté les pestanyes) i que, a més a més, té 3 filles:
  1. `HomePage`, a la qual s'hi accedeix a través de la ruta `home`,
  2. `ListPage`, a la qual s'hi accedeix a través de la ruta `list` i
  3. `AboutPage`, a la qual s'hi accedeix a través de la ruta `about`.
3. Finalment, a `tabs-routing.module.ts` també es defineix el control de *ruta per defecte* perquè si l'usuari no introdueix cap fragment de ruta la navegació es redirigeixi a `home`.

### Pestanyes secundàries
En cas que no es vulgui gestionar la navegació principal de l'aplicació mitjançant pestanyes, sinó que aquesta metodologia només es vulgui aplicar en un apartat concret de l'aplicació, l'estructura que caldrà crear és la mateixa però, en comptes de fer-la penjar directament de l'`AppComponent`, penjarà de la pàgina principal de l'apartat que es vol gestionar amb pestanyes.
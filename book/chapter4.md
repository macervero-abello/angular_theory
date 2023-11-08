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

### Configuració
Per configurar el *routing* clàssic cal seguir els passos següents:
1. Configurar els fragments de les rutes i enllaçar-los als components corresponents
2. Activar el servei de *routing*
3. Crear els enllaços i la navegació als fitxers `HTML` i `TS` dels components

#### Contextualització d'un exemple
Per fer l'explicació, suposarem que tenim una aplicació amb dues pàgines: la pàgina `home`, definida al `HomeComponent`, i la pàgina `about`, definida a l'`AboutComponent`.

#### Configuració dels fragments de les rutes i activació del servei de *routing*
La configuració de les rutes i l'activació del servei de *routing* es realitza al fitxer de configuració de dependències `app.module.ts`.

Primer de tot cal crear un array de rutes (`Routes`), on cada ruta és un objecte `JSON` que, com a mínim, té l'atribut `path` per definir el fragment que es desitja i l'atribut `component` per establir el component que s'ha de carregar.

L'ordre en què es defineixen les rutes dins d'aquest array és important, ja que l'enrutador d'Angular utilitza la política *first-mach wins*, és a dir, utilitza la primera ruta que coincideix amb el fragment de la `URL`. Per tant, les rutes s'han d'ordenar de més específiques a menys.

Un cop definides les rutes cal importar el mòdul `RouterModule` i configurar-lo amb l'array creat.

{% tabs %}
{% tab title="Codi app.module.ts" %}
```typescript:hightlight{2, 4}
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { HomeComponent } from './pages/home/home.component';
import { AboutComponent } from './pages/about/about.component';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
]

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



## *Lazy routing*
El *lazy routing* es caracteritza pel fet que les rutes i, per tant, els components associats, no es carreguen fins que l'usuari hi entra per primer cop (no hi ha cap precàrrega)

Això significa que cada cop que l'usuari entra per primer cop a una pàgina, hi ha un petit lapsus de temps durant el qual el sistema de *routing* fa la càrrega de la nova ruta. Aquest temps d'espera, però, és molt ràpid i, per tant, no és apreciable per l'usuari

Aquest tipus de *routing* és el més addient, especialment en aplicacions web molt grans.

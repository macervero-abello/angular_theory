# Capítol 15. Internacionalització i Localització
Un *locale* identifica una regió en concret amb una llengua específica i consisteix a adaptar
* unitats de mesura,
* data i hora,
* xifres,
* monedes i
* textos.

La **internacionalització** (i18n) consisteix a dissenyar i preparar un projecte per tal que es pugui utilitzar amb els diferents *locales* i, en canvi, la **localització** (l10n) consisteix a crear diverses versions del projecte adaptades a cada *locale* on es vulgui posar en producció.

Per poder internacionalitzar i localitzar una aplicació Angular s'han de seguir els passos següents:
1. Instal·lació de la llibreria de localització
2. Preparar els components i els textos per tal que puguin ser traduïts
3. Extracció de tots els elements que han de ser traduïts
4. Crear els fitxers de traducció per als diversos idiomes

L'explicació teòrica es realitzarà sobre el projecte que es mostra a continuació, el qual només té el *component* principal `App`

{% tabs %}
{% tab title="Codi app.html" overflow="wrap" lineNumbers="true" %}
```html
    <h1>Tiquet de la compra internacionalitzat/localitzat</h1>
    <p>{{ date }}, {{ time }}</p>

    <p>S'han comprat {{ products.length }} productes</p>

    <li>
    @for(prod of prodcts: track prod.id) {
        <ul>{{ prod.name }} - {{prod.price}}€</ul>
    }
    </li>
```
{% endtab %}

{% tab title="Codi app.ts" overflow="wrap" lineNumbers="true" %}
```typescript
    import { Component } from '@angular/core';

    @Component({
    selector: 'app-root',
    imports: [],
    templateUrl: './app.html',
    styleUrl: './app.css'
    })
    export class App {
        public products: any[] = [
            {name: 'Llet', price: 1.58},
            {name: 'Pernil dolç', price: 2.15}
        ];
    }
```
{% endtab %}
{% endtabs %}

## Instal·lació de la llibreria de localització
Al mercat es poden trobar múltiples llibreries que permeten internacionalitzar i localitzar una aplicació Angular. No obstant això, la llibreria oficial es troba en el *package* `@angluar/localize`. Per poder-la instal·lar cal executar la comanda següent dins del projecte que es vol internacionalitzar:

```bash
$ ng add @angluar/localize
```

Un cop la llibreria està instal·lada, el projecte ja està preparat per tal de poder ser internacionalitzat i localitzat.

## Preparació dels components i dels textos que han de ser traduïts
La llibreria `@angular/localize` ofereix tres elements que permeten marcar tots aquells elements i textos que s'han d'adaptar per fer una bona internacionalització i localització de l'aplicació:

1. L'atribut `i18n` indica que el text d'una etiqueta `HTML` (o d'un *component*) s'ha de traduir (per exemple, el text d'un `<h1>`)
2. L'atribut `i18n-{attribute-name}` indica que el text associat a un atribut d'una etiqueta `HTML` (o d'un *component*) s'ha de traduir (per exemple, el text de l'atribut `alt` de l'etiqueta `<img/>`)
3. La marca `$localize` indica que el contingut d'una propietat definida dins del codi `TS` del *component* s'ha de traduir

Així doncs, podem deduir que els atributs `i18n` i `i18n-{attribute-name}` s'utilitzen dins del codi `HTML` i, en canvi, la marca `$localize` s'utilitza dins del codi `TS`

### Atribut `i18n`
Per indicar que el text d'una determinada etiqueta `HTML` o d'un *component* s'ha de traduir només cal afegir l'atribut `i18n` dins de l'etiqueta concreta. Per exemple, per marcar que el text del títol principal `<h1>` del codi bàsic presentat al principi del capítol s'ha de traduir, només fa falta fer el següent:

```html
<h1 i18n>Tiquet de la compra internacionalitzat/localitzat</h1>
<p>{{ date() }}</p>

<p>S'han comprat {{ products.length }} productes</p>

<li>
  @for(prod of products(); track prod.id) {
    <ul>{{ prod.name }} - {{prod.price}}€</ul>
  }
</li>

<img src="img/shopping_cart.png" alt="Carro de la compra" width="128px"/>
```

Això ens permetrà extreure el text `Tiquet de la compra internacionalitzat/localitzat` a un fitxer de traducció per tal que pugui ser adaptat a l'idioma que es desitgi. Ara però, aquesta acció és la més simple i la que dóna menys informació a l'equip de traducció de l'aplicació, ja que, tal com es veurà en l'apartat [Extracció de tots els elements que han de ser traduïts](#extracció-de-tots-els-elements-que-han-de-ser-traduïts), el fitxer de traducció només recull el text que cal traduir, sense contextualitzar-lo (en quina pàgina es troba, quin tipus d'etiqueta és, etc.). Per posar en context el text que s'ha de traduir, l'atribut `i18n` pot tenir assignat un valor optatiu que consta de 3 parts:

* el significat (*meaning*),
* una explicació (*description*) i
* l'identificador (*custom id*).

Aquestes porcions de contextualització poden aparéixer totes, només algunes d'elles o cap (tal com hem vist en l'exemple anterior), essent la més útil i important de totes elles l'idenficador. Ara però, es decideixi el que es decideixi, és important l'ordre en que és posen:

```html
    i18n="{meaning}|{description}@@{custom_id}"
    i18n="Site header|Complete title of the site@@pageTitle"
```

Tal com es pot veure, el primer que apareix és el significat (*meaning*), seguit del símbol `|`; a continuació hi apareix l'explicació (*description*) i, finalment, l'identificador decorat amb els símbols `@@`.

Així doncs, algunes de les opcions que podem aplicar al codi d'exemple són les següents:

```html
<!--Sense contextualització-->
<h1 i18n>Tiquet de la compra internacionalitzat/localitzat</h1>

<!--Amb l'identificador: aquesta contextualització és bàsica i la que s'acostuma a posar sempre, com a mínim-->
<h1 i18n="@@pageTitle">Tiquet de la compra internacionalitzat/localitzat</h1>

<!--Amb l'explicació-->
<h1 i18n="Complete title of the site">Tiquet de la compra internacionalitzat/localitzat</h1>

<!--Amb l'explicació i l'identificador: l'opció més comuna-->
<h1 i18n="Complete title of the site@@pageTitle">Tiquet de la compra internacionalitzat/localitzat</h1>

<!--Contextualització completa-->
<h1 i18n="Site header|Complete title of the site@@pageTitle">Tiquet de la compra internacionalitzat/localitzat</h1>
```

### Atribut `i18n-{attribute-name}`
L'atribut `i18n-{attribute-name}` s'utilitza indicar que el text d'un determinat atribut `HTML` o d'un *component*. Per utilitzar-lo només cal afegir l'atribut `i18n-{attribute-name}` dins de l'etiqueta concreta. Per exemple, per marcar que el text de l'atribut `alt` de l'etiqueta `<img/>` del codi bàsic presentat al principi del capítol s'ha de traduir, només fa falta fer el següent:

```html
<h1 i18n>Tiquet de la compra internacionalitzat/localitzat</h1>
<p>{{ date() }}</p>

<p>S'han comprat {{ products.length }} productes</p>

<li>
  @for(prod of products(); track prod.id) {
    <ul>{{ prod.name }} - {{prod.price}}€</ul>
  }
</li>

<img src="img/shopping_cart.png" i18n-alt alt="Carro de la compra" width="128px"/>
```

Tal com passa amb l'atribut `i18n`, l'atribut `i18n-{attribute-name} també es pot contextualitzar per facilitar la feina a l'equip traductor, de tal manera, que les opcions són les mateixes: significat (*meaning*), explicació (*description*) i identificador (*custom id*).

```html
    i18n-{attribute-name}="{meaning}|{description}@@{custom_id}"
    i18n-{attribute-name}="Alt image|Shopping cart image@@imageAlternativeText"
```

Així doncs, algunes de les opcions que podem aplicar al codi d'exemple són les següents:

```html
<!--Sense contextualització-->
<img src="img/shopping_cart.png" i18n-alt alt="Carro de la compra" width="128px"/>

<!--Amb l'identificador: aquesta contextualització és bàsica i la que s'acostuma a posar sempre, com a mínim-->
<img src="img/shopping_cart.png" i18n-alt="@@imageAlternativeText" alt="Carro de la compra" width="128px"/>

<!--Amb l'explicació-->
<img src="img/shopping_cart.png" i18n-alt="Shopping cart image" alt="Carro de la compra" width="128px"/>

<!--Amb l'explicació i l'identificador: l'opció més comuna-->
<img src="img/shopping_cart.png" i18n-alt="Shopping cart image@@imageAlternativeText" alt="Carro de la compra" width="128px"/>

<!--Contextualització completa-->
<img src="img/shopping_cart.png" i18n-alt="Alt image|Shopping cart image@@imageAlternativeText" alt="Carro de la compra" width="128px"/>
```


## Extracció de tots els elements que han de ser traduïts

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
2. Preparar els textos per tal que puguin ser traduïts
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
    import { Component, signal } from '@angular/core';

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
Al mercat es poden trobar múltiples llibreries que permeten internacionalitzar i localitzar una aplicació Angular. No obstant això, l'oficial és la llibreria `@angluar/localize`. Per poder-la instal·lar cal executar la comanda següent dins del projecte que es vol internacionalitzar:

```bash
$ ng add @angluar/localize
```

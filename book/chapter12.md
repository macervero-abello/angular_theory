# Capítol 12. Accés a dades externes: serveis web (API REST) i fitxers JSON
Moltes aplicacions web obtenen les seves dades i les seves funcionalitats a partir de la gestió d'un conjunt de serveis web (APIs REST). Aquests serveis poden ser completament lliures o, per contra, necessitar algun sistema d'autenticació, dels quals, els més habituals són:
- Autenticació mitjançant una *API Key*
- Autenticació mitjançant el protocol OAuth v2

Una altra característica d'aquests serveis web és que molts d'ells poden retornar les dades en múltiples formats, com poden ser `XML`, `CSV` `JSON`, etc. Tot i això, actualment, el format més estès, o popular, per transmetre dades a través de la xarxa és el `JSON`.

Per tal de poder accedir a tot aquest conjunt de dades externes, l'aplicació web ha de ser capaç d'emetre peticions `http` configurades correctament, per tant, Angular ofereix el *service* `HttpClient`, que és l'encarregat de fer-ne tota la gestió.

L'[enllaç](https://github.com/public-apis/public-apis) següent conté un recull de serveis web que permeten fer proves i, fins i tot, aplicacions que es poden posar en producció. Addicionalment, fent diverses cerques a internet, es poden trobar múltiples serveis web que cobreixen moltes necessitats.

## *Service* `HttpClient`
Com ja s'ha dit, el *service* `HttpClient` permet fer crides `http`, tot gestionant-ne tota la configuració necessària. És per aquesta raó, doncs, que té les funcions que equivalen als diversos verbs del [protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP):
 * `get`
 * `post`
 * `put`
 * `delete`
 * `options`
entre altres.

Ara però, per poder utilitzar aquest *service* dins de l'aplicació Angular cal, prèviament, configurar-ne un `provider` dins del fitxer `app.config.ts`, el qual queda estructurat de la manera següent:

{% code title="Codi app.config.ts" overflow="wrap" lineNumbers="true" %}
```typescript
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZonelessChangeDetection } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';     //Importació del package on es troba el provider necessari
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZonelessChangeDetection(),
    provideHttpClient(),                                      //Càrrega del provider del service HttpClient
    provideRouter(routes)
  ]
};
```
{% endcode %}

Un cop fet això, el *service* ja es pot injectar allí on es necessiti, sigui un *component* o un altre *service* (el més habitual), utilitzant la injecció a través del constructor de l'element corresponent o mitjançant el mètode `inject()`.

### Configuració dels mètodes del *service* `HttpClient`
Qualsevol mètode del *service* `HttpClient` s'executa en segon pla i, d'aquesta manera, s'evita bloquejar l'aplicació durant l'estona que triga en arribar la resposta (cal pensar que les crides `http` acostumen a ser crides a servidors externs i, per tant, poden tenir associat un cert retard). Així doncs, aquests *service* i els seus mètodes implementen al patró `Observer`, de tal manera que, quan obtenen el resultat de la crida llencen un avís a tots els objectes que n'estaven pendents (l'estaven observant). Per poder gestionar aquesta arribada de la informació i, per tant, el seu tractament, cal aplicar el mètode `subscribe`, el qual defineix 3 accions:
 1. `next`: acció a realitzar si tot ha anat bé i la informació ha arribat correctament
 2. `complete`: acció a realitzar quan s'ha acabat el tractament de la informació
 3. `error`: acció a realitzar si s'ha produït algun error i la informació no ha arribat correctament.

Aquest mètode, a més a més, retorna l'objecte `Observable` per tal de gestionar el tancament de la subscripció en cas que calgui. 

Resumint, i utilitzant de base el mètode equivalent al verb `get`, el tractament de la crida `http` té aquesta estructura bàsica

```typescript
const subscription = this._http.get<generic_type>("path_to_file or url").subscribe({
    next: (data_param) => {code},
    complete: () => {code},
    error: (error_param) => {code}
});
```

on `next`, `complete` i `error` es defineixen com a 3 funcions *lambda* (anònimes).

{% hint style="info" %}
**Nota:** l'explicació teòrica d'aquest capítol es realitzarà utilitzant el mètode `get`, per qüestions de simplificació i perquè és el mètode que ofereixen la major part dels serveis web gratuïts.
{% endhint %}

Ens els següents apartats s'exemplificarà com utilitzar el *service* `HttpClient` i el seu mètode `get` per tal d'assolir l'objectiu de la lectura de dades externes.

## Accés a un servei web públic (sense autenticació)
En aquest apartat s'explica, a través de codi, com accedir a les dades que ofereix un servei web públic, en concret, el servei web [chucknorris.io](https://api.chucknorris.io/), el qual permet obtenir acudits fets sobre el personatge de Chick Norris.

Quan ja tenim seleccionat el servei web que volem consultar, i abans de començar a crear el codi de la nostra aplicació, cal que analizem la documentació de l'API per tal de saber
1. quines funcionalitats (mètodes) ofereix;
2. quines dades retorna cadascuna de les funcionalitats i
3. quin verb cal utilitzar per accedir a cadascuna d'aquestes funcionalitats (normalment, `GET` o `POST`).

El servei web proposat té diversos mètodes als quals es pot accedir a través del verb `HTTP GET`; en concret, en té un que genera múltiples acudits a partir d'una paraula passada per paràmetre

```url
https://api.chucknorris.io/jokes/search?query={query}
```

on `query` és la paraula amb la qual es vol parametritzar els acudits. Per exemple `https://api.chucknorris.io/jokes/search?query=titanic`.

Segons la documentació, i si tot ha anat bé, aquest recurs retorna un codi 200 i un `JSON` amb la següent estructura:

```json
{
  "total": 14,
  "result": [
    {
      "categories": [],
      "created_at": "2020-01-05 13:42:19.324003",
      "icon_url": "https://api.chucknorris.io/img/avatar/chuck-norris.png",
      "id": "r4ac4izrrte97luheuytmg",
      "updated_at": "2020-01-05 13:42:19.324003",
      "url": "https://api.chucknorris.io/jokes/r4ac4izrrte97luheuytmg",
      "value":"Contrary to popular belief, the Titanic didn't hit an iceberg. The ship was off course and ran into Chuck Norris while he was doing the backstroke across the Atlantic."
    },
    ...
  ]
}
```

En canvi, si s'ha produït un error, retorna l'estructura JSON següent:

```json
{
  "timestamp":	"2025-07-08T19:11:13.624Z",
  "status":	400,
  "error":  "Bad Request",
  "message":	"search.query: size must be between 3 and 120",
  "violations": {
    "search.query": "search.query: size must be between 3 and 120"
  }	
}
```

Analitzades totes aquestes dades, ja podem començar a implementar l'aplicació, tot seguint els passos següents:
1. Crear un *model*, sigui `interface` sigui `class`, que pugui encapsular els atributs dels objectes definits dins del `JSON`
2. Crear un *service* que s'encarregui de gestionar les dades i, per tant, d'accedir al servei web
3. Injectar el *service* `HttpClient` al nostre servei (cal recordar que també caldrà configurar el *provider* corresponent dins del fitxer `app.config.ts`)
4. Fer la petició `http` mitjançant el mètode `get`, implementant correctament el tractament del patró `Observer` a través de la funció `subscription`.

### Creació del *model*
Com que l'aplicació que ha de consumir el servei web utilitza el patró MVC, el primer que cal fer és implementar un model que pugui contenir les dades que es volen consultar. Així doncs, crearem el model `Joke` mitjançant la comanda `ng generate class model/jokes --skip-tests`, la qual tindrà dos atributs: `query` i `jokes`.

{% code title="Codi jokes.ts" overflow="wrap" lineNumbers="true" %}
```typescript
export class Jokes {
  private _query: string;
  private _jokes: string[];

  constructor() {
    this._query = "";
    this._jokes = [];
  }

  get query(): string { return this._query; }
  get jokes(): string[] { return this._jokes; }

  set query(query: string) { this._query = query; }
  set jokes(jokes: string[]) { this._jokes = jokes; }
}
```
{% endcode %}

### Creació del *service*
Quan el *model* ja està definit, el següent que cal fer és crear el *service* que s'encarregui de fer la consulta al servei web. Aquest servei es crearà utilitzant la comanda `ng generate service service/joke-service --skip-tests` i haurà d'injectar el *service* `HttpClient` (recordar que l'apartat [*Service* `HttpClient`](#service-httpclient) mostra la configuració del fitxer `app.config.ts`).

{% code title="Codi jokes-service.ts" overflow="wrap" lineNumbers="true" %}
```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable, Signal, signal, WritableSignal } from '@angular/core';
import { Jokes } from '../model/jokes';

@Injectable({
  providedIn: 'root'
})
export class JokeService {
  private readonly BASE_URL = "https://api.chucknorris.io/jokes/search?query=";
  private _jokes: WritableSignal<Jokes>;
  private _error: WritableSignal<boolean>;

  public jokes: Signal<Jokes>;
  public error: Signal<boolean>;

  constructor(private _http: HttpClient) {
    this._jokes = signal(new Jokes());
    this._error = signal(false);

    this.jokes = this._jokes.asReadonly();
    this.error = this._error.asReadonly();
  }

  retrieveJokes(query: string): void {
    const url = this.BASE_URL + query;
    this._http.get<Jokes>(url).subscribe({
      next: (value: any) => {
        let jokes = new Jokes();
        jokes.query = query;
        for(let i=0; i<value.total; i++) {
          jokes.jokes.push(value.result[i].value);
        }
        this._jokes.set(jokes);
        this._error.set(false);
      },
      error: (error: any) => {
        let jokes = new Jokes();
        this._jokes.set(jokes);
        this._error.set(true);
      },
      complete: () => {}
    });
  }
}
```
{% endcode %}

### Creació de la *view*
Finalment, només queda completar la *view* a les necessitats visuals de l'aplicació. Com que en aquest cas l'aplicació és molt senzilla, tota la vista es crearà al *component* `App` (per fer-la més agradable s'hi afegeixen els estils Bootstrap, els quals s'expliquen en el [Capítol 9](chapter09.md)).

{% tabs %}
{% tab title="Codi HTML: app.html" %}
```html
<div class="m-3">
  <input type="text" class="form-control" placeholder="Paraula a cercar" aria-label="Paraula a cercar" [(ngModel)]="query" />
</div>

<div class="clearfix">
  <button type="button" class="float-end m-3 btn btn-success" (click)="retrieveJokes()">Consultar</button>
  <button type="button" class="float-end m-3 btn btn-danger" (click)="clearForm()">Cancel·lar</button>
</div>

@if(error()) {
  <div class="m-3 p-3 text-bg-danger" >
    Error, no s'han trobat acudits a partir de la paraula {{ query() }}
  </div>
} @else if(jokes()) {
  <div class="m-3 p-3 border border-success">
    <h3>{{ query() }}</h3>

    <br/>

    @for(joke of jokes().jokes; track joke) {
      <p>{{ joke }}</p>
    }
  </div>
}

<router-outlet />
```
{% endtab %}

{% tab title="Codi TS: app.ts" %}
```typescript
import { Component, Signal, signal, WritableSignal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { Jokes } from './model/jokes';
import { JokeService } from './service/joke-service';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, FormsModule],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  public query: WritableSignal<string>;
  public jokes: Signal<Jokes>;
  public error: Signal<boolean>;

  constructor(private _jokesService: JokeService) {
    this.query = signal("");

    this.jokes = this._jokesService.jokes;
    this.error = this._jokesService.error;
  }

  retrieveJokes() {
    this._jokesService.retrieveJokes(this.query());
  }

  clearForm(): void {
    this.query.set("");
  }
}
```
{% endtab %}
{% endtabs %}

## Accés a un servei web autenticat amb *API Key*
Per accedir a un servei web autenticat amb *API Key* cal seguir els mateixos passos que en el cas anterior, posant molta atenció a la documentació per esbrinar com obtenir i utilitzar l'*API Key* necessària per autenticar les crides als recursos que ofereix.

L'exemple de codi que es mostra a continuació utilitza el servei web [Pixabay](https://pixabay.com/service/about/api/), el qual permet, entre altres coses, cercar imatges seguint els criteris marcats per l'usuari.

El recurs web que permet fer la cerca és un `HTTP GET` amb la url parametritzada següent:

```url
https://pixabay.com/api/?key={API_KEY}&q={QUERY}&image_type=photo
```

on `API_KEY` és el *token* d'autenticació i `QUERY` els criteris de cerca que es volen utilitzar per trobar imatges d'un cert tipus.

Segons la documentació, i si tot ha anat bé, aquest recurs retorna un codi 200 i un JSON amb la següent estructura:

```json
{
	"total": 4692,
	"totalHits": 500,
	"hits": [
	    {
	        "id": 195893,
	        "pageURL": "https://pixabay.com/en/blossom-bloom-flower-195893/",
	        "type": "photo",
	        "tags": "blossom, bloom, flower",
	        "previewURL": "https://cdn.pixabay.com/photo/2013/10/15/09/12/flower-195893_150.jpg"
	        "previewWidth": 150,
	        "previewHeight": 84,
	        "webformatURL": "https://pixabay.com/get/35bbf209e13e39d2_640.jpg",
	        "webformatWidth": 640,
	        "webformatHeight": 360,
	        "largeImageURL": "https://pixabay.com/get/ed6a99fd0a76647_1280.jpg",
	        "fullHDURL": "https://pixabay.com/get/ed6a9369fd0a76647_1920.jpg",
	        "imageURL": "https://pixabay.com/get/ed6a9364a9fd0a76647.jpg",
	        "imageWidth": 4000,
	        "imageHeight": 2250,
	        "imageSize": 4731420,
	        "views": 7671,
	        "downloads": 6439,
	        "likes": 5,
	        "comments": 2,
	        "user_id": 48777,
	        "user": "Josch13",
	        "userImageURL": "https://cdn.pixabay.com/user/2013/11/05/02-10-23-764_250x250.jpg",
	    },
	    {
	        "id": 73424,
	        ...
	    },
	    ...
	]
}
```

on la propietat `hints` és l'*array* que conté les imatges la descripció de les quals, d'una manera o altra, encaixa amb la cerca realitzada.

En canvi, si s'ha produït un error, retorna el codi d'error més apropiat segons el problema que ha esdevingut conjuntament amb un text descriptiu.

Per tal de poder obtenir la votra `API_KEY` cal que inicieu sessió a Pixabay. D'aquesta manera, la pàgina de la documentació de l'API us mostrarà el vostre *token* directament.

Un cop obtinguda l'`API_KEY` ja es pot començar a crear l'aplicació seguint el patró MVC i creant el model `Image` i el service `PixabayService`.

### Creació del *model*
Per crear la classe `Image` amb els atributs `id`, `tags`, `url`, `width` i `height`, s'utilitzarà la comanda `ng generate classe model/image --skip-tests`. El codi resultant serà el següent:

{% code title="Codi image.ts" overflow="wrap" lineNumbers="true" %}
```typescript
export class Image {
  private _id: number;
  private _tags: string[];
  private _url: string;
  private _width: number;
  private _height: number;

  constructor() {
    this._id = -1;
    this._tags = [];
    this._url = "";
    this._width = -1;
    this._height = -1;
  }

  get id(): number { return this._id; }
  get tags(): string[] { return this._tags; }
  get url(): string { return this._url; }
  get width(): number { return this._width }
  get height(): number { return this._height; }

  set id(id: number) { this._id = id; }
  set tags(tags: string[]) { this._tags = tags; }
  set url(url: string) { this._url = url; }
  set width(width: number) { this._width = width; }
  set height(height: number) { this._height = height; }
}
```
{% endcode %}

Cal veure que el model no ha de reproduir al 100% el contingut del JSON que retorna el servei web, sinó que només ha de tenir els camps que interessen al desenvolupador.

### Creació del *service*
Fet el *model*, ja es pot passar a implementar el *service*, que es crearà utilitzant la comanda `ng generate service service/pixabay-service --skip-tests` i gestionarà l'accés al recurs mitjançant el codi següen:

{% code title="Codi pixabay-service.ts" overflow="wrap" lineNumbers="true" %}
```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable, Signal, signal, WritableSignal } from '@angular/core';

import { Image } from '../model/image';

@Injectable({
  providedIn: 'root'
})
export class PixabayService {
  private readonly API_KEY: string = "...";
  private _images: WritableSignal<Image[]>;

  public images: Signal<Image[]>;

  constructor(private _http: HttpClient) {
    this._images = signal([]);
    this.images = this._images.asReadonly();
  }

  retrieveImageSearch(searchTags: string[]) {
    let searchParam: string = "";
    for(let i=0; i<searchTags.length; i++) {
      searchParam += searchTags[i];
      if(i < searchParam.length - 1) searchParam += "+";
    }

    const url = "https://pixabay.com/api/?key=" + this.API_KEY + "&q=" + searchParam + "&image_type=photo";

    this._http.get(url).subscribe({
      next: (value: any) => {
        let images: Image[] = [];
        value.hits.forEach((element: any) => {
          let image: Image = new Image();
          image.id = element.id;
          image.tags = element.tags.split(", ");
          image.url = element.largeImageURL;
          image.width = element.imageWidth;
          image.height = element.imageHeight;

          images.push(image);

          this._images.set(images);
        });
      },
      error: (error: any) => {},
      complete: () => {}
    });
  }
}
```
{% endcode %}

### Creació de la *view*
Finalment, el *component* `App` s'adapta a les necessitats de l'aplicació tal com mostra el tros de codi següent.

{% tabs %}
{% tab title="Codi HTML: app.html" %}
```html
<div class="m-3">
  <input type="text" class="form-control" placeholder="Paràmetres de cerca: 'dog, puppy, water dog'" aria-label="Paràmetres de cerca, separats per comes" [(ngModel)]="searchParams">
</div>
<div class="clearfix">
  <button type="button" class="btn btn-success float-end m-3" (click)="retrieveSearch()">Consultar</button>
  <button type="button" class="btn btn-danger float-end mt-3" (click)="clearForm()">Cancel·lar</button>
</div>

<div class="row row-cols-1 row-cols-md-4">
  @for(image of images(); track image.id) {
    <div class="col">
      <div class="card mx-auto" style="width: 18rem;">
        <img [src]="image.url" class="card-img-top" >
        <div class="card-body">
          <h5 class="card-title">Resultat {{ $index + 1 }} - {{ image.id }}</h5>
          <p class="card-text">
            Amplada: {{ image.width }}
            <br/>
            Alçada: {{ image.height }}
          </p>
          <a [href]=image.url target="_blank">Enllaç a la imatge</a>
        </div>
      </div>
    </div>
  }
</div>

<router-outlet />
```
{% endtab %}

{% tab title="Codi TS: app.ts" %}
```typescript
import { Component, Signal, signal, WritableSignal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { PixabayService } from './service/pixabay-service';
import { Image } from './model/image';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, FormsModule],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  public searchParams: WritableSignal<string>;
  public images: Signal<Image[]>;

  constructor(private _pixabayService: PixabayService) {
    this.searchParams = signal("");
    this.images = this._pixabayService.images;
  }

  retrieveSearch() {
    let params: string[] = this.searchParams().replace(", ", ",").split(",");
    this._pixabayService.retrieveImageSearch(params);
  }

  clearForm() {
    this.searchParams.set("");
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**Nota important**\
En aquest exemple es mostra un servei web autenticat per *API Key* a través dels paràmetres de la URL del recurs. Ara però, sempre que es vulgui utilitzar un servei web amb *API Key* cal llegir atentament la seva documentació per esbrinar, exactament, com fer l'autenticació, ja que hi ha vegades que l'*API Key* pot anar dins dels *Headers* de la crida `HTTP`.
{% endhint %}


## Webgrafia del capítol
* Google (2025). [Angular](https://angular.dev/). Consultat el 7 de juliol de 2025.
* Udemy (2025). [Curs *Angular - The Complete Guide (2025 Edition)*](https://www.udemy.com/course/the-complete-guide-to-angular-2/). Consultat el 7 de juliol de 2025.

<!--https://developer.mozilla.org/en-US/docs/Web/HTTP-->
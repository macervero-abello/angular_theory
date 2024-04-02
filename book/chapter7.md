# Capítol 7. Accés a un servei web (API REST)
Moltes aplicacions web obtenen les seves dades i les seves funcionalitats a partir de la gestió d'un conjunt de serveis web (APIs REST). Aquests serveis poden ser completament lliures o, per contra, necessitar algun sistema d'autenticació, dels quals, els més habituals són:
- Autenticació mitjançant una *API Key*
- Autenticació mitjançant el protocol OAuth v2

L'[enllaç](https://github.com/public-apis/public-apis) següent conté un recull de serveis web que permeten fer proves i, fins i tot, aplicacions que es poden posar en producció. Addicionalment, fent diverses cerques a internet, es poden trobar múltiples serveis web que cobreixen moltes necessitats.

## Accés a un servei web públic (sense autenticació)
L'accés a un servei web públic es fa utiltizant la llibreria `HttpClient`, la qual permet llegir a recurs en línia.

El codi és molt similar al que s'utilitza per fer la lectura d'un fitxer JSON ((Capítol 6)[./chapter6.md]) ja que, en realitat, un servei web que implementi una API REST treballa sobre els verbs `HTTP` i té l'opció de retornar les dades en format JSON.

Els passos que cal fer per consultar un servei web públic són els següents:
1. Cercar el servei web que es vol utilitzar.
2. Analitzar-ne la documentació per saber quines funcionalitats ofereix i si aquestes són `GET`, `POST`, etc.
3. Crear l'aplicació per fer la consulta

Per mostrar un exemple a través de codi, utilitzen el Servei Web [lyrics.ovh](https://lyricsovh.docs.apiary.io/), el qual permet obtenir la lletra d'una cançó a partir del seu títol i del nom del seu artista.

Per assolir aquest objectiu ofereix el següent recurs web (URL del servei) a través del verb `HTTP GET`:

```url
https://api.lyrics.ovh/v1/artist/title
```

on `artist` ha de ser el nom de l'artista de la cançó i `title` n'ha de ser el títol. Per exemple `https://api.lyrics.ovh/v1/adele/skyfall`.

Segons la documentació, i si tot ha anat bé, aquest recurs retorna un codi 200 i un JSON amb la següent estructura:

```json
{
    "lyrics": "..."
}
```

En canvi, si s'ha produït un error, retorna el codi 404 i l'estructura JSON següent:

```json
{
    "error": "No lyrics found"
}
```

Com que l'aplicació que ha de consumir aquest servei web ha d'estar feta seguint el patró MVC, el primer que cal fer és implementar un model que pugui contenir les dades que es volen consultar. Així doncs, crearem el model `Song` mitjançant la comanda `ng generate class model/song --skip-tests`, la qual tindrà tres atributs: `artistName`, `title` i `lyrics`.

```typescript
export class Song {
    private _artistName: string;
    private _title: string;
    private _lyrics: string[];

    constructor() {
        this._artistName = "";
        this._title = "";
        this._lyrics = [];
    }

    get artistName(): string { return this._artistName }
    get title(): string { return this._title; }
    get lyrics(): string[] { return this._lyrics }

    set artistName(artistName: string) { this._artistName = artistName; }
    set title(title: string) { this._title = title; }
    set lyrics(lyrics: string[]) { this._lyrics = lyrics; }
}
```

Fet això, és necessita un *service* que s'encarregui de fer la consulta al servei web, el qual es crearà utilitzant la comanda `ng generate service service/song --skip-tests`. Tal com s'ha dit abans, aquest *service* utilitzarà el *service* `HttpClient` propi d'Angular, per tal, dins de l'`app.module.ts` caldrà importar l'`HttpClientModule`.

```typescript
import { Injectable } from '@angular/core';
import { Song } from '../model/song.model';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class SongService {
  
  private _song: Song | null = null;
  private _error: boolean = false;

  constructor(private _http: HttpClient) {}

  retrieveSong(artistName: string, title: string): void {
    this._error = false;

    const url = "https://api.lyrics.ovh/v1/" + artistName + "/" + title;
    this._http.get(url).subscribe({
      next: (value: any) => {
        this._song = new Song();
        this._song.artistName = artistName;
        this._song.title = title;
        this._song.lyrics = value.lyrics.split("\n");
      },
      error: (error: any) => {
        this._song = null;
        this._error = true;
      },
      complete: () => {}
    })
  }

  get song(): Song | null { return this._song; }
  get hasError(): boolean { return this._error; }
}
```

Finalment, només queda completar la vista i, per tant, adaptar l'`AppComponent` a les necessitats visuals de l'aplicació.

{% tabs %}
{% tab title="Codi HTML: app.component.html" %}
```html
<div class="m-3">
  <input type="text" class="form-control" placeholder="Nom de l'artista" aria-label="Nom de l'artista" [(ngModel)]="artistName">
  <br/>
  <input type="text" class="form-control" placeholder="Títol de la cançó" aria-label="Títol de la cançó" [(ngModel)]="title">
</div>

<div class="clearfix">
  <button type="button" class="float-end m-3 btn btn-success" (click)="retrieveLyrics()">Consultar</button>
  <button type="button" class="float-end m-3 btn btn-danger" (click)="clearForm()">Cancel·lar</button>
</div>

<div class="m-3 p-3 border border-success" *ngIf="song">
  <h3>{{ song.title }}</h3>
  <h5>{{ song.artistName }}</h5>

  <br/>

  <p *ngFor="let paragraph of song.lyrics">{{ paragraph }}</p>
</div>

<div class="m-3 p-3 text-bg-danger" *ngIf="hasError">
  Error, no s'ha trobat la cançó
</div>
```
{% endtab %}

{% tab title="Codi TS: app.component.ts" %}
```typescript
import { Component } from '@angular/core';
import { SongService } from './service/song.service';
import { Song } from './model/song.model';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  public artistName: string = "";
  public title: string = "";

  constructor(private _songService: SongService) {}

  retrieveLyrics() { this._songService.retrieveSong(this.artistName, this.title); }

  clearForm(): void {
    this.artistName = "";
    this.title = "";
  }
  
  get song(): Song | null { return this._songService.song; }
  get hasError(): boolean { return this._songService.hasError; }
}

```
{% endtab %}
{% endtabs %}

## Accés a un servei web autenticat amb *API Key*
Per accedir a un servei web autenticat amb *API Key* cal seguir els mateixos passos que en el cas anterior. Cal posar molta atenció, però, a la documentació, per esbrinar com obtenir i utilitzar l'*API Key* necessària per autenticar les crides als recursos que ofereix.

L'exemple de codi que es mostra a continuació utilitza el Servei Web [Pixabay](https://pixabay.com/service/about/api/), el qual permet, entre altres coses, cercar imatges seguint els criteris marcats per l'usuari.

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

Com sempre, per crear el model, s'utilitzarà la comanda `ng generate classe model/image --skip-tests` i tindrà l'aspecte següent:

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

Cal veure que el model no ha de reproduir al 100% el contingut del JSON que retorna el servei web, sinó que només ha de tenir els camps que interessen al desenvolupador.

Fet el model, ja es pot passar a implementar el *service*, que es crearà utilitzant la comanda `ng generate service service/pixabay --skip-tests`.

```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Image } from '../model/image.model';


@Injectable({
  providedIn: 'root'
})
export class PixabayService {
  
  private API_KEY: string = " ... ";
  private _images: Image[];

  constructor(private _http: HttpClient) {
    this._images = [];
  }

  retrieveImageSearch(searchTags: string[]) {
    let searchParam: string = "";
    for(let i=0; i<searchTags.length; i++) {
      searchParam += searchTags[i];
      if(i < searchParam.length - 1) searchParam += "+";
    }

    const url = "https://pixabay.com/api/?key=" + this.API_KEY + "&q=" + searchParam + "&image_type=photo";

    this._images = [];
    this._http.get(url).subscribe({
      next: (value: any) => {
        value.hits.forEach((element: any) => {
          let image: Image = new Image();
          image.id = element.id;
          image.tags = element.tags.split(", ");
          image.url = element.largeImageURL;
          image.width = element.imageWidth;
          image.height = element.imageHeight;

          this._images.push(image);
        });
      },
      error: (error: any) => {},
      complete: () => {}
    });
  }

  get images(): Image[] { return this._images; }
}
```

Finalment, l'`AppComponent` s'adapta a les necessitats de l'aplicació tal com mostra el tros de codi següent.

{% tabs %}
{% tab title="Codi HTML: app.component.html" %}
```html
<div class="m-3">
  <input type="text" class="form-control" placeholder="Paràmetres de cerca: 'dog, puppy, water dog'" aria-label="Paràmetres de cerca, separats per comes" [(ngModel)]="searchParams">
</div>
<div class="clearfix">
  <button type="button" class="btn btn-success float-end m-3" (click)="retrieveSearch()">Consultar</button>
  <button type="button" class="btn btn-danger float-end mt-3" (click)="clearForm()">Cancel·lar</button>
</div>

<div class="row row-cols-1 row-cols-md-4">
  <div class="col" *ngFor="let image of images; let idx = index">
    <div class="card mx-auto" style="width: 18rem;">
      <img [src]="image.url" class="card-img-top" >
      <div class="card-body">
        <h5 class="card-title">Resultat {{ idx + 1 }} - {{ image.id }}</h5>
        <p class="card-text">
          Amplada: {{ image.width }}
          <br/>
          Alçada: {{ image.height }}
        </p>
        <a [href]=image.url target="_blank">Enllaç a la imatge</a>
      </div>
    </div>
  </div>
</div>
```
{% endtab %}

{% tab title="Codi TS: app.component.ts" %}
```typescript
import { Component } from '@angular/core';
import { Image } from './model/image.model';
import { PixabayService } from './service/pixabay.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {

  public searchParams: string;

  constructor(private _pixabayService: PixabayService) { this.searchParams = ""; }

  retrieveSearch() {
    let params: string[] = this.searchParams.replace(", ", ",").split(",");
    this._pixabayService.retrieveImageSearch(params);
  }
  clearForm() { this.searchParams = ""; }
  get images(): Image[] { return this._pixabayService.images; }
}
```
{% endtab %}
{% endtabs %}

**Nota important**\
En aquest exemple es mostra un servei web autenticat per *API Key* a través dels paràmetres de la URL del recurs. Ara però, sempre que es vulgui utilitzar un servei web amb *API Key* cal llegir atentament la seva documentació per esbrinar, exactament, com fer l'autenticació, ja que hi ha vegades que l'*API Key* pot anar dins dels *Headers* de la crida `HTTP`.
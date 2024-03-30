# Capítol 7. Accés a un servei web (API REST)
Moltes aplicacions web obtenen les seves dades i les seves funcionalitats a partir de la gestió d'un conjunt de serveis web (APIs REST). Aquests serveis poden ser completament lliures o, per contra, necessitar algun sistema d'autenticació, dels quals, els més habituals són:
- Autenticació mitjançant una *API Key*
- Autenticació mitjançant el protocol OAuth v2

L'(enllaç)[https://github.com/public-apis/public-apis] següent conté un recull de serveis web que permeten fer proves i, fins i tot, aplicacions que es poden posar en producció. Addicionalment, fent diverses cerques a internet, es poden trobar múltiples serveis web que cobreixen moltes necessitats.

## Accés a un servei web públic (sense autenticació)
L'accés a un servei web públic es fa utiltizant la llibreria `HttpClient`, la qual permet llegir a recurs en línia.

El codi és molt similar al que s'utilitza per fer la lectura d'un fitxer JSON ((Capítol 6)[./chapter6.md]) ja que, en realitat, un servei web que implementi una API REST treballa sobre els verbs `HTTP` i té l'opció de retornar les dades en format JSON.

Els passos que cal fer per consultar un servei web públic són els següents:
1. Cercar el servei web que es vol utilitzar.
2. Analitzar-ne la documentació per saber quines funcionalitats ofereix i si aquestes són `GET`, `POST`, etc.
3. Crear l'aplicació per fer la consulta

Per mostrar un exemple a través de codi, utilitzen el Servei Web (lyrics.ovh)[https://lyricsovh.docs.apiary.io/], el qual permet obtenir la lletra d'una cançó a partir del seu títol i del nom del seu artista.

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

https://pixabay.com/api/docs
L'exemple de codi que es mostra a continuació utilitza el Servei Web (Pixabay)[https://pixabay.com/api/docs], el qual permet, entre altres coses, cercar imatges seguint els criteris marcats per l'usuari.

El recurs web que permet fer la cerca és un `HTTP GET` amb la url parametritzada següent:

```url
https://pixabay.com/api/?key={API_KEY}&q={QUERY}&image_type=photo
```


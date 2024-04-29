# Capítol 15. *Infinite Scroll*
Ionic té un element específic que permet fer una càrrega *lazy*, és a dir, sota demanda, de les dades que cal presentar per pantalla. Aquest element és el que utilitzen la majoria d'aplicacions actuals (*Twitter*, *Whatsapp*, *Telegram*, etc.) per fer la càrrega d'un gran volum d'informació sense que això impliqui bloquejar l'aplicació i que l'usuari hagi d'esperar molta estona a veure resultats.

Normalment, aquesta càrrega *lazy* va associada al consum d'un servei web, el qual permet que el desenvolupador que l'utilitza decideixi si vol rebre totes les dades de cop o les vol rebre de manera parcial.

Per exemple, el servei web [*NewsAPI*](https://newsapi.org/) té la crida [*Top headlines*](https://newsapi.org/docs/endpoints/top-headlines) la qual permet rebre titulars de notícies. En particular, té dos paràmetres, `pageSize` i `page`, que permeten definir la mida de la resposta que es vol rebre. Si aquests dos paràmetres no s'especifíquen, el servei web retornarà tots els titulars disponibles; en canvi, si s'especifiquen, el paràmetre `pageSize` defineix quantes notícies es volen rebre de cop i el paràmetre `page` defineix quina pàgina (quin bloc) de titulars es vol rebre. Si `pageSize` és 50 i `page` és 1, és rebran les 50 primeres notícies; quan `page` passi a ser 2, es rebran les notícies que van de la 51 a la 100; quan `page`sigui 3 es rebran les notícies que van de la 101 a la 150 i així successivament.

A continuació es presenta com crear el *service* per tractar aquest tipus de serveis web i s'explica com crear una vista que permeti la càrrega *lazy* a través de l'*infinite scroll*.

## Creació d'un *service* per fer crides parcials a l'API *NewsAPI*
Aquest service és molt similar al mostrat al [Capítol 7](chapter7.md) i es crearà amb la comanda:

```bash
    ionic generate service service/news --skip-tests
```

El codi serà el següent:

```typescript
import { Injectable } from '@angular/core';
import { News } from '../model/news';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class NewsService {
  
  private _page: number = 0;
  private _pageSize: number = 50;
  private _news: News[];
  private _baseURL = "https://newsapi.org/v2/top-headlines";

  constructor(private _http: HttpClient) {
    this._news = [];
  }

  retrieveNews(): void {
    const url = this._baseURL + "?page=" + this._page + "&pageSize=" + this._pageSize + "&apiKey=...&language=en";
    this._http.get(url).subscribe({
      next: (news: any) => {
        for(let i=0; i<news.articles.length; i++) {
          let pieceNews: News = new News();
          pieceNews.source = news.articles[i].source.name;
          pieceNews.author = news.articles[i].author;
          pieceNews.title = news.articles[i].title;
          pieceNews.description = news.articles[i].description;

          this._news.push(pieceNews);
        }
      },
      error: () => {},## Creació d'un *service* per fer crides parcials a l'API *NewsAPI*
      complete: () => {}
    });

    this._page++;
  }

  get news(): News[] {
    return this._news;
  }
}
```

Es pot veure que, cada cop que s'executa el mètode `retrieveNews()`, l'atribut `this._page` s'incrementa en una unitat, per tant, la pròxima crida que se'n faci retornarà el següent bloc de notícies.

Es pot veure al codi que aquest *service* utilitza objectes de la classe *News*, la qual es pot generar mitjançant la comanda:

```bash
    ionic generate class model/news --skip-tests
```

i té el codi següent:

```typescript
export class News {
    private _source: string;
    private _author: string;
    private _title: string;
    private _description: string;

    constructor() {
        this._source = "";
        this._author = "";
        this._title = "";
        this._description = "";
    }

    get source(): string {return this._source;}
    set source(source: string) {this._source = source;}
    get author(): string {return this._author;}
    set author(author: string) {this._author = author;}
    get title(): string {return this._title;}
    set title(title: string) {this._title = title;}
    get description(): string {return this._description;}
    set description(description: string) {this._description = description;}
}
```

## Creació de la pàgina amb *Infinite Scroll*
Aquesta pàgina ha de tenir un llistat que mostri els titulars de les notícies, per exemple, i les vagi carregant poc a poc amb l'[*Infinite Scroll*](https://ionicframework.com/docs/api/infinite-scroll). Així doncs, el seu codi és el següent:

{% tabs %}
{% tab title="Codi list.page.html" %}
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>Notícies</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-list>
    <ion-item *ngFor="let pieceNews of news; let index">
      <ion-label>{{ pieceNews.title }}</ion-label>
    </ion-item>
  </ion-list>
  <ion-infinite-scroll (ionInfinite)="retrieveNextData($event)">
    <ion-infinite-scroll-content></ion-infinite-scroll-content>
  </ion-infinite-scroll>
</ion-content>

```
{% endtab %}
{% tab title="Codi list.page.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { InfiniteScrollCustomEvent } from '@ionic/angular';
import { News } from 'src/app/model/news';
import { NewsService } from 'src/app/service/news.service';

@Component({
  selector: 'app-news',
  templateUrl: './news.page.html',
  styleUrls: ['./news.page.scss'],
})
export class NewsPage implements OnInit {

  constructor(private _newsService: NewsService) {
    this._newsService.retrieveNews();
  }

  ngOnInit() {}

  get news(): News[] {
    return this._newsService.news;
  }

  retrieveNextData(event: any): void {
    this._newsService.retrieveNews();
    setTimeout(() => {
      (event as InfiniteScrollCustomEvent).target.complete();
    }, 500);
  }

}

```
{% endtab %}
{% endtabs %}

Es pot comprovar que el codi `HTML` conté un llistat i, tot just a sota, l'element `<ion-infinite-scroll>`. Aquest element ha de definir l'*event binding* de l'event `ionInfinite`, el qual es llença cada cop que l'usuari ha fet *scroll* fins al final de tot de la pàgina. El seu objectiu, per tant, és cridar a una funció que demani i carregui el següent bloc de dades. En el cas de l'exemple mostrat, aquesta funció és `retrieveNextData()` del codi `TS`, la qual executa el mètode `retrieveNews()` del service `NewsService`.
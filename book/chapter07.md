# Capítol 7. Detecció de canvis
El *framework* Angular està, constantment, analitzant els canvis d'estat que s'esdevenen de la interacció amb l'usuari (per exemple, events i *data-bindings*) per tal de poder actualitzar les pàgines i les dades que es mostren correctament. Fins a la versió 17 la seva estratègia es basava en l'ús de la llibreria [Zone.js](https://www.npmjs.com/package/zone.js), la qual fa un resseguiment de qualsevol operació asíncrona dins del codi per poder rastrejar els canvis i actualitzar la interfície d'usuari correctament. Ara però, tot i que l'ús de la llibreria Zone.js és molt còmoda de cara al desenvolupador, des de la versió 18, Angular està evolucionant cap al que es coneix com a estratègia *zoneless* per, bàsicament, motius d'eficiència.

En aquest capítol s'explicarà com funcionen totes dues estratègies i com transformar una aplicació que utilitza Zone.js en una aplicació *zoneless*. Per fer-ho, utilitzarem l'esquema d'aplicació que mostra la Figura 7.1

<figure>
    <img src="img/ch07/app_schema.png" alt="Esquema dels components de l'aplicació d'exemple">
    <figcaption>Figura 7.1: esquema dels components de l'aplicació d'exemple</figcaption>
</figure>

A continuació es mostra el codi inicial, basat en l'ús de la llibreria Zone.js.
{% tabs %}
{% tab title="Codi TS App" %}
```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { Header } from './layout/header/header';
import { Messages } from './elements/messages/messages';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, Header, Messages],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  protected title = 'change-detection-project';

  public onCheckUpdates() {
    let str: string = "App component";
    console.log("[" + str + "] checked!");
    return str;
  }
}
```
{% endtab %}

{% tab title="Codi HTML App" %}
```html
<div>
    <app-header></app-header>
    <app-messages></app-messages>
    
    <p>{{ onCheckUpdates() }}</p>
    <router-outlet></router-outlet>
</div>
```
{% endtab %}

{% tab title="Codi TS Header" %}
```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-header',
  imports: [FormsModule],
  templateUrl: './header.html',
  styleUrl: './header.css'
})
export class Header {
  private _username: string;
  private _hasSession: boolean;

  constructor() {
    this._username = "Invitat";
    this._hasSession = false;
  }

  get username(): string {
    return this._username;
  }

  set username(username: string) {
    this._username = username;
  }

  get hasSession(): boolean {
    return this._hasSession;
  }

  set hasSession(hasSession: boolean) {
    this._hasSession = hasSession;
  }

  public onIniSession() {
    if(this._username.length >= 1 && this._username != "Invitat") this._hasSession = true;
  }

  public onCloseSession() {
    this._hasSession = false;
    this._username = "Invitat";
  }

  public onCheckUpdates() {
    let str: string = "Header component";
    console.log("[" + str + "] checked!");
    return str;
  }
}
```
{% endtab %}

{% tab title="Codi HTML Header" %}
```html
<header>
    <div class="w3-flex">
        <div class="flex-welcome">
            <h1>Benvingut a ChangeDetector, {{ username }}</h1>
        </div>
    
        <div class="flex-session w3-right-align">
            @if(!hasSession) {
                <input type="text" placeholder="Username..." [(ngModel)]="username" class="w3-margin-right w3-input w3-border"/>
                <button (click)="onIniSession();" class="w3-button w3-light-gray">Iniciar sessió</button>
            } @else {
                <button (click)="onCloseSession();" class="w3-button w3-light-gray">Tancar sessió</button>
            }
        </div>
    </div>

    <p>{{ onCheckUpdates() }}</p>
</header>
```
{% endtab %}

{% tab title="Codi TS Messages" %}
```typescript
import { Component } from '@angular/core';
import { ListMessages } from './list-messages/list-messages';

@Component({
  selector: 'app-messages',
  imports: [ListMessages],
  templateUrl: './messages.html',
  styleUrl: './messages.css'
})
export class Messages {
  public onCheckUpdates() {
    let str: string = "Messages component";
    console.log("[" + str + "] checked!");
    return str;
  }
}
```
{% endtab %}

{% tab title="Codi HTML Messages" %}
```html
<div class="messages">
    <app-list-messages></app-list-messages>

    <p>{{ onCheckUpdates() }}</p>
</div>
```
{% endtab %}

{% tab title="Codi TS ListMessage" %}
```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-list-messages',
  imports: [FormsModule],
  templateUrl: './list-messages.html',
  styleUrl: './list-messages.css'
})
export class ListMessages {
  private _newMsg: string;
  private _messages: string[];

  constructor() {
    this._newMsg = "";
    this._messages = [];
  }

  get newMsg(): string {
    return this._newMsg;
  }

  set newMsg(msg: string) {
    this._newMsg = msg;
  }

  get messages(): string[] {
    return this._messages;
  }

  public onAddNewMessage() {
    this._messages.push(this._newMsg);
  }

  public onCheckUpdates() {
    let str: string = "ListMessages component";
    console.log("[" + str + "] checked!");
    return str;
  }
}
```
{% endtab %}

{% tab title="Codi HTML ListMessage" %}
```html
<div class="list-messages">
    <input type="text" [(ngModel)]="newMsg" class="w3-input w3-border" />
    <button (click)="onAddNewMessage()" class="w3-button w3-light-gray">Afegir missatge</button>

    <ul>
        @for(msg of messages; track msg) {
            <li>{{ msg }}</li>
        }
    </ul>

    <p>{{ onCheckUpdates() }}</p>
</div>
```
{% endtab %}
{% endtabs %}

Com es pot veure, l'aplicació està formada per una capçalera, on es simula un inici de sessió, i un contenidor de missatges per poder gestionar una llista de notes. A cadascun dels *components* s'hi ha afegit el mètode `onCheckUpdates()` el qual mostra un missatge a la consola de les eines de desenvolupador del navegador on s'executa. Aquest mètode ajudarà a visualitzar les comprovacions que es realitzen utilitzant l'estratègia Zone.js o l'estratègia *zoneless*.

## Detecció de canvis clàssica: Zone.js
Quan s'utilita la llibreria Zone.js, qualsevol canvi en algun dels *components* activa la comprovació d'absolutament tots els *components* de l'arbre jeràrquic mostrat a la Figura 7.1.

Així doncs, si l'usuari intenta iniciar sessió i, per tant, modifica l'atribut `username` del *component* `Header`, s'activa la comprovació de tot l'arbre, començant pel *component* arrel `App`, tal com mostra la Figura 7.2

<figure>
    <img src="img/ch07/change_detection1.png" alt="Comprovacions que es disparen al llarg de tot l'arbre jeràrquic quan es produeix un canvi i s'utilitza Zone.js">
    <figcaption>Figura 7.2: comprovacions que es disparen al llarg de tot l'arbre jeràrquic quan es produeix un canvi i s'utilitza Zone.js</figcaption>
</figure>

Totes aquestes comprovacions provoquen l'execució del mètode `onCheckUpdates()` utilitzat en l'*String Interpolation* que es fa al final de cada component, per tant, la consola de les eines de desenvolupament mostra les comprovacions en l'ordre en què es fan, tal com mostra la Figura 7.3

<figure>
    <img src="img/ch07/change_detection2_1.png" alt="Situació tot just iniciar l'aplicació">
    <img src="img/ch07/change_detection2_2.png" alt="Situació un cop modificat el nom d'usuari">
    <figcaption>Figura 7.3: la primera imatge mostra la situació inicial de l'aplicació i, en canvi, la segona imatge mostra les comprovacions que ha provocat la modificació del nom d'usuari dins del component Header</figcaption>
</figure>

## Primera millora d'eficiència: *Angular Signals*
L'*Angular Signals* és un sistema que permet el resseguiment precís (granular) dels canvis d'estat de l'aplicació per fer que les actualitzacions del *renderitzat* siguin el més òptimes possible.

{% hint style="info" %}
**Informació:** s'entèn com a estat el valor que reben les dades emmagatzemades dins de l'aplicació en un moment determinat.
{% endhint %}

Així doncs, un *signal* és un embolcall sobre les dades que permet notificar tots els canvis que pateix aquest valor a tots els consumidors que s'hi han subscrit. Si parlem des del punt de vista d'arquitectura de software, els *signals* implementen un patró *Observer* on l'*Observable* són les dades que conformen l'estat de l'aplicació i que són susceptibles d'anar canviant al llarg del temps; i els *Observers* són tots aquells elements (ara per ara *components*) que volen detectar qualsevol canvi d'estat per poder actualitzar correctament la seva part `HTML`.

Tal com passa amb les dades que embolcallen, els *signals* poden ser de lectura/escriptura (*writebles*) o només de lectura (*read-only*)

### Treballar amb *signals*
A continuació s'expliquen els mètodes que permeten crear i manipular *signals* per, a posteriori, aplicar les transformacions necessàries al codi inicial del capítol per permetre el seu funcionament mitjançant aquesta metodologia.

#### Creació de *signals*
Per poder crear un *signal* només necessitem crear un atribut de tipus `WritableSignal` (pels de lectura/escriptura) o `Signal` (pels només de lectura) i utilitzar el constructor `signal()`:
```typescript
  //Creació un WritableSignal per emmagatzemar un valor numèric aleatori
  rndvalue: WritableSignal<number> = signal(Math.floor(Math.random() * 100));

  //Creació d'un Signal per emmagatzemar un valor numèric constant (vegeu l'ús del mètode asReadonly())
  readonly myguess: Signal<number> = signal(53).asReadonly();
```


## Segona millora d'eficiència: estratègia OnPush

## Aplicacions *Zoneless*

<!--
Webgrafia
1. https://www.npmjs.com/package/zone.js 23 de juny
2. https://medium.com/@sehban.alam/what-is-zone-js-in-angular-e0029c21c32f 23 de juny
-->
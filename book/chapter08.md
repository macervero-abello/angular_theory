# Capítol 8. Comunicació entre components Angular
En el  [Capítol 3. Components Angular](./chapter03.md) s'ha explicat els conceptes bàsics dels *components* Angular. Ara però, una aplicació mínimament complexa acaba tenint múltiples *components*, molts niats els uns dins dels altres, que, a més a més, interactúen entre ells sigui traspassant-se dades, sigui mitjançant events.

En aquest capítol, doncs, aprendrem a ampliar la implementació bàsica dels *components* per tal de definir-los un conjunt de propietats (o atributs) que els permetin rebre dades de l'exterior i un conjunt d'events als quals puguin reaccionar.

## Creació de propietats i events: comunicació entre components niats
Quan nosaltres creem un *component*, en certa manera, estem ampliant el ventall d'etiquetes d'`HTML`. Per tant, també hem de ser capaços de definir nous atributs i nous events per aquestes etiquetes.

### Creació d'atributs: *signal* `input()`
Per crear un nou atribut d'etiqueta és necessari utilitzar el mètode `input()`, el qual retorna un *signal*.

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {
  @Input() nelems: number = 0;
}
```

Aquest decorador permet exposar qualsevol dels propietat del nostre component, de tal manera que es pot fer servir com a atribut a la seva etiqueta i, per tant, aplicar-hi un *property binding*.

```html
<h1>AppComponent</h1>
<app-list [nelems]="10"></app-list>
```

Si modifiquem l'exemple de l'apartat anterior, des de l'`AppComponent` podem crear la llista parametritzada que mostri tants elements com defineixi el nou atribut de `ListComponent`

{% tabs %}
{% tab title="Codi TS ListComponent" %}
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {
  @Input() nelems: number = 0;

  public generateArrayElems(): number[] {
    let vals: number[] = [];
    for(let i=0; i<this.nelems; i++) {
      vals.push(i);
    }
    return vals;
  }
}
```
{% endtab %}

{% tab title="Codi HTML AppComponent" %}
```html
<h1>AppComponent</h1>
<app-list [nelems]="10"></app-list>
```
{% endtab %}

{% tab title="Codi HTML ListComponent" %}
```html
<div style="border: solid black 2px;">
    <h3>ListComponent</h3>
    <ol>
        <li *ngFor="let elem in generateArrayElems()">Element {{ elem+1 }}</li>
    </ol>
</div>
```
{% endtab %}

{% tab title="Visualització al navegador" %}
![Visualització del resultat al navegador](img/parameterized_nested_list.png)
{% endtab %}
{% endtabs %}

*Vídeos 067 i 068 del capítol "05 Components & Databinding Deep Dive" del curs d'Udemy*

### Creació d'events
La creació d'events és molt similar a la creació d'atributs. No obstant això, en aquest cas l'objectiu és afegir funcionalitat al nou component: a través de la detecció dels events que creem podrem donar una funcionalitat o una alra, segons el tractament que programem.
.
Per crear un nou event d'etiqueta és necessari utilitzar 
1. el decorador `@Output`, el qual també s'ha d'importar des de la llibreria `@angular/core`
2. la classe EventEmitter, per definir l'event i
3. el llençament del nou event creat (`emit`).

```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {
  @Input() nelems: number = 0;
  @Output() onrandom: EventEmitter<void> = new EventEmitter();
  
  public generateArrayElems(): number[] {
    let vals: number[] = [];
    for(let i=0; i<this.nelems; i++) {
      vals.push(i);
    }
    return vals;
  }

  public emitRandomEvent(): void {
    this.onrandom.emit();
  }
}
```

El decorador `@Output` permet exposar qualsevol de les propietat del nostre component, de tal manera que es pot fer servir com a event a la seva etiqueta i, per tant, aplicar-hi un *event binding*.

```html
<h1>AppComponent</h1>
<app-list [nelems]="rndnum" (onrandom)="generateRandomNumber()"></app-list>
```

Si tornem a modificar l'exemple de l'apartat anterior, des de l'`AppComponent` podem crear la llista parametritzada aleatòriament a `ListComponent`

{% tabs %}
{% tab title="Codi TS ListComponent" %}
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent {
  @Input() nelems: number = 0;
  @Output() onrandom: EventEmitter<void> = new EventEmitter();
  
  public generateArrayElems(): number[] {
    let vals: number[] = [];
    for(let i=0; i<this.nelems; i++) {
      vals.push(i);
    }
    return vals;
  }

  public emitRandomEvent(): void {
    this.onrandom.emit();
  }
}
```
{% endtab %}

{% tab title="Codi HTML ListComponent" %}
```html
<div style="border: solid black 2px;">
    <h3>ListComponent</h3>
    <ol>
        <li *ngFor="let elem of generateArrayElems()">Element {{ elem+1 }}</li>
    </ol>
    <button (click)="emitRandomEvent()">Generació aleactòria</button>
</div>
```
{% endtab %}

{% tab title="Codi TS AppComponent" %}
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  public rndnum: number = 0;

  public generateRandomNumber(): void {
    this.rndnum = Math.floor(Math.random() * 100);
  }
}
```
{% endtab %}

{% tab title="Codi HTML AppComponent" %}
```html
<h1>AppComponent</h1>
<app-list [nelems]="rndnum" (onrandom)="generateRandomNumber()"></app-list>
```
{% endtab %}


{% tab title="Visualització al navegador" %}
![Visualització del resultat al navegador](img/random_nested_list.png)
{% endtab %}
{% endtabs %}

*Vídeo 069 capítol "05 Components & Databinding Deep Dive" del curs d'Udemy*

# Capítol 5. Modificació del DOM
El *framework* Angular ofereix eines que permeten modificar el DOM (*Document Object Model*) d'HTML de manera automàtica. Aquesta modificació es pot fer de manera
* iterativa, creant múltiples elements iguals, o
* condicional, creant elements depenent de si una condició s'avalua certa o no.

## Control de flux iteratiu `@for`
El bloc de codi iteratiu `@for` permet crear i *renderitzar* codi `HTML` per cadascun dels elements d'una estructura de dades iterable. A més a més, quan detecta qualsevol modificació de l'estructura de dades (eliminació, actualització o inserció d'un element) adapta els elements *renderitzats* de manera automàtica.

Per fer aquesta adaptació de la manera més eficient possible necessita utilitzar l'expressió `track`, la qual permet fer un seguiment dels elements de la col·lecció de dades a través dels seus identificadors, que han de ser únics. Això permet que, donada qualsevol alteració de les dades, el *framework* Angular no hagi de repintar (*renderitzar*) de nou tots els elements `HTML`, sinó que pot aplicar canvis quirúrgics i concrets per fer el mínim d'operacions sobre el DOM possibles.

### Ús del control de flux iteratiu `@for`
Per poder utiltizar la instrucció `@for` cal que el codi del *component* compleixi un conjunt de requeriments:
1. El codi `TS` ha de definir una estructura de dades
```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  private _fruitsList: string[] = ['pomes', 'peres', 'plàtans'];

  get fruitsList(): string[] {
    return this._fruitsList;
  }
}
```
2. wegweg
3. 

### Codi *legacy* amb la directiva `*ngFor`


## Control de flux condicional `@if`

### Codi *legacy* amb la directiva `*ngIf`
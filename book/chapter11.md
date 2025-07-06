# Capítol 11. Patró *MVC*
Angular és un *framework* pensat per treballar amb el patró *Model View Controller* (MVC), el qual estableix que el codi s'ha de modularitzar i encapsular segons les funcionalitats necessàries, separant, de manera clara, la part la vista (la *view*, que permet interactuar amb l'usuari) de la lògica de l'aplicació (el *model* i el *controller*, que gestonen les dades i les funcionalitats).

Ara però, cal tenir en compte que el patró MVC és lleugerament diferent al que s'implementa en altres llenguatges de programació, especialment en aplicacions d'escriptori. Així doncs, el patró clàssic segueix l'esquema representat en la Figura 11.1, on es pot veure que és el *model* l'encarregat d'accedir a les fonts de dades (bases de dades, fitxers, etc.), aplicant altres dissenys de *software* com, per exemple, el *Data Access Object* (DAO) o altres propisdels *frameworks* dels diversos llenguatges.

<figure>
    <img src="img/ch11/classic_mvc.png" alt="Implementació clàssica del patró MVC">
    <figcaption>Figura 11.1: implementació clàssica del patró MVC</figcaption>
</figure>

En canvi, el patró MVC a Angular s'implementa tal com mostra la Figura 11.2, on és el *controller* (en Angular s'anomena *service*) qui s'encarrega d'accedir a les fonts de dades.

<figure>
    <img src="img/ch11/angular_mvc.png" alt="Implementació del patró MVC a Angular">
    <figcaption>Figura 11.2: implementació del patró MVC a Angular</figcaption>
</figure>

## *Model*
Els *models* emmagatzemen les dades i l'estat de l'aplicació en un moment donat, per tant, són tot el conjunt de classes i objectes que es necessiten per representar la informació gestionada.

Angular ofereix dos tipus de *models*, les *interfaces* i les *classes*.

### *Interfaces*
Les *interfaces* d'Angular, a diferència de les *interfaces* d'altres llenguatges com, per exemple, Java, no defineixen un conjunt de mètodes per tal que siguin implementades per subclasses. Ans al contrari, defineixen les propietats d'un objecte.

Per tal de poder-ne tenir una imatge mental més clara, les *interfaces* d'Angular són la representació, en codi, d'un objecte JSON definit en un fitxer. Així doncs, si, per exemple, tenim el següent document JSON.

{% code title="Fitxer students.json" overflow="wrap" lineNumbers="true" %}
  ```json
    [
        {
            "id": "123456789A",
            "firstName": "Zebulen",
            "lastName1": "Adlam",
            "lastName2": "Wareham",
            "age": 19,
            "email": "zwareham0@exblog.jp",
            "course": "DAM2",
        },
        {
            "id": "987654321Z",
            "firstName": "Alana",
            "lastName1": "Diament",
            "lastName2": "Mitchelmore",
            "age": 21,
            "email": "amitchelmore1@canalblog.com",
            "course": "DAM2"
        },
        ...
    ]
  ```
{% endcode %}

La *interface* `Student` que en pot representar els objectes podria ser la següent:

{% code title="Interface student.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
    export interface Student {
        id: string,
        firstName: string,
        lastName1: string,
        lastName2: string,
        age: number,
        email: string,
        course: string,
        assistance: boolean
    }
  ```
{% endcode %}

Per tal de poder crear les *interfaces* cal executar la comanda següent:
```bash
$ ng generate interface [path/interface_name]
```

Per tant, si s'executa
```bash
$ ng generate interface model/student
```
es crearà la *interface* `Student` dins de la ruta `src/app/model` del projecte.

Un cop creada l'*interface*, per crear-ne una instància s'ha d'implementar el codi següent, on es pot veure que, en el moment de la declaració ja s'han de definir els valors de totes les seves propietats (siguin valors reals, siguin per defecte):

{% code title="Codi per declarar un objecte de tipus Student" overflow="wrap" lineNumbers="true" %}
```typescript
let defaultStudent: Student = {
    'id': "",
    'firstName': "",
    'lastName1': "",
    'lastName2': "",
    'age': 0,
    'email': "",
    'course': ""
};
let realStudent: Student = {
    'id': "123456789A",
    'firstName': "Zebulen",
    'lastName1': "Adlam",
    'lastName2': "Wareham",
    'age': 19,
    'email': "zwareham0@exblog.jp",
    'course': "DAM2"
}
```
{% endcode %}

### Classes
Les classes d'Angular, tal com passa amb Javascript i la resta de llenguatges *orientats a objectes*, defineixen un objecte de la vida real mitjançant un conjunt de propietats (atributs) i mètodes.

Una classe senzilla només tindrà els mètodes constructor i *accessors* (*getters* i *setters* de Typescript), però classes més elaborades poden tenir mètodes molt més complexos; tot depèn de les necessitats de l'aplicació.

Per exemple, si creem una classe que representi un professor, el codi podria ser el següent:

{% code title="Classe teacher.ts" overflow="wrap" lineNumbers="true" %}
```typescript
export class Teacher {
    private _firstName: string;
    private _lastName1: string;
    private _lastName2: string;
    private _courses: string[];

    constructor() {
        this._firstName = "";
        this._lastName1 = "";
        this._lastName2 = "";
        this._courses = [];
    }

    get firstName(): string {
        return this._firstName;
    }

    set firstName(firstName: string) {
        this._firstName = firstName;
    }

    get lastName1(): string {
        return this._lastName1;
    }

    set lastName1(lastName1: string) {
        this._lastName1 = lastName1;
    }

    get lastName2(): string {
        return this._lastName2;
    }

    set lastName2(lastName2: string) {
        this._lastName2 = lastName2;
    }

    get courses(): string[] {
        return [...this._courses];
    }

    public addCourse(course: string): void {
        this._courses.push(course);
    }
}
```
{% endcode %}

Per poder-ne instanciar un objecte, el codi necessari és el següent:

{% code title="Codi per a instanciar un objecte de tipus Teacher" overflow="wrap" lineNumbers="true" %}
```typescript
let teacher: Teacher = new Teacher();
```
{% endcode %}

La comanda que permet crear una classe és la següent:
```bash
$ ng generate class [path/class_name]
$ ng generate class [path/class_name] --skip-tests      #Per evitar la creació del fitxer de proves .spec.ts
```

Per tant, si s'executa
```bash
$ ng generate class model/teacher --skip-tests
```
es crearà la classe `Teacher` dins de la ruta `src/app/model` del projecte.


## *View*
Les diferents vistes de l'aplicació (pàgines i elements visuals) s'implementen, tal com hem vist fins ara, amb els *components*.

## *Services* (*Controller*)
Els *services* són el nucli (*core*) de tota aplicació Angular ja que:
1. Recullen les peticions de l'usuari a través de la interacció amb la *view*
2. Gestionen les dades (accedeixen als fitxers, al *LocalStorage*, a les bases de dades, a les diferents APIs, etc.) i creen els models necessaris
3. Defineixen les funcionalitats de l'aplicació a través dels seus mètodes

Per crear un *service* cal executar la comanda següent:
```bash
$ ng generate service [path/service_name]
$ ng generate service [path/service_name] --skip-tests      #Per evitar la creació del fitxer de proves .spec.ts
```

Per tant, si s'executa
```bash
$ ng generate service service/attendance-list-manager --skip-tests
```
es crearà el *service* `AttendanceListManager` dins de la ruta `src/app/service` del projecte.

Així doncs, tota la lògica de programa que, fins ara, hem implementat dins del fitxer `TS` dels diversos *components* s'ha de passar dins dels *services*, de tal manera que els *components* accedirant als diversos *services* per tal de poder executar les funcionalitats desitjades. A més a més, cal tenir en compte un punt molt important: cada cop que s'accedeix per primer cop a una aplicació web feta en Angular, el *framework* analitza tots els *services* que es necessitaran al llarg de la seva execució i en crea una única instància, seguint el patró *Singleton*. Quan un dels elements de l'aplicació (sobretot un *component* o un altre *service*) necessiten un *service* determinat, en demanen la instància a través del procés que s'anomena *injection*. Aquesta *injection* es pot dur a terme declarant el service necessari directament a la zona de paràmetres del constructor de l'element que el necessita o mitjançant el mètode `injection()`, tal com es veurà en el codi d'exemple més avall.

Per poder fer un exemple complet, imaginem una aplicació per passar llista on, al LocalStorage emmagatzemem les dades tots els estudiants de l'assignatura i del professor. Els models que s'utilitzaran seran la *interface* `Student` i la classe `Teacher`. El *service* serà el següent:

{% code title="Codi del service AttendanceListManager" overflow="wrap" lineNumbers="true" %}
```typescript
import { Injectable, Signal, signal, WritableSignal } from '@angular/core';

import { Student } from '../model/student';
import { Teacher } from '../model/teacher';

@Injectable({
  providedIn: 'root'
})
export class AttendanceListManager {
  private _students: WritableSignal<Student[]>;
  private _teacher: WritableSignal<Teacher>;
  private _missing: WritableSignal<string[]>;


  public students: Signal<Student[]>;
  public teacher: Signal<Teacher>;

  constructor() {
    let students = localStorage.getItem("STUDENTS");
    let teacher = localStorage.getItem("TEACHER");
    let missing = localStorage.getItem("MISSING");

    if(students != null) this._students = signal(JSON.parse(students));
    else {
      this._students = signal([
        {
          "id": "123456789A",
          "firstName": "Zebulen",
          "lastName1": "Adlam",
          "lastName2": "Wareham",
          "age": 19,
          "email": "zwareham0@exblog.jp",
          "course": "DAM2"
        },
        {
          "id": "987654321Z",
          "firstName": "Alana",
          "lastName1": "Diament",
          "lastName2": "Mitchelmore",
          "age": 21,
          "email": "amitchelmore1@canalblog.com",
          "course": "DAM2"
        }
      ]);
    }

    if(teacher != null) this._teacher = signal(JSON.parse(teacher));
    else {
      let teacher: Teacher = new Teacher();
      teacher.firstName = "M.Àngels";
      teacher.lastName1 = "Cerveró";
      teacher.lastName2 = "Abelló";
      teacher.addCourse("DAM2");
      this._teacher = signal(teacher);
    }

    if(missing != null) this._missing = signal(JSON.parse(missing));
    else this._missing = signal([]);

    this.students = this._students.asReadonly();
    this.teacher = this._teacher.asReadonly();
  }

  public addMissingRegister(id: string, attend: boolean) {
    if(!this._missing().includes(id) && !attend) {
      this._missing.set([...this._missing(), id]);
    } else if(this._missing().includes(id) && attend) {
      this._missing.update((curVal: string[]) => {
        let idx = curVal.findIndex((elem: string) => {
          return elem == id;
        });
        curVal.splice(idx, 1);
        return curVal;
      });
    }

    localStorage.setItem("MISSING", JSON.stringify(this._missing()));
  }

  public isAttending(id: string): boolean {
    return this._missing().includes(id);
  }
}
```
{% endcode %}

Les principals característiques d'aquest *service* són les següents:
1. Decorador `@Injectable`: indica que, justament, la classe `AttendanceListManager` és un *service* i que, per tant, se n'ha de crear una instància per tal que pugui ser injectat dins dels *components* o altres *services* que necessitin utilitzar les funcionalitats que ofereix. En concret, aquest *service* estarà disponible per a tota l'aplicació, ja que el seu àmbit d'actuació s'ha configurat com a `root`. Aquest àmbit també es pot definir com a `platform`, amb la qual cosa el *service* estarà disponible per a totes les aplicacions que col·laboren per a crear una mateixa aplicació web.
2. Tots aquells atributs del *service* que han de respondre a la interacció amb l'usuari o a modificacions de les dades seran *signals*
3. El constructor, en aquest cas, llegeix els valors que l'aplicació ha guardat al *LocalStorage* i, en cas que aquest emmagatzematge estigui buit, crea valors per defecte.
4. El mètode `addMissingRegister()` crea un registre de falta d'assistència a l'alumne amb identificador `id` i el guarda al *LocalStorage*
5. El mètode `isAttending()` comprova si l'alumne amb identificador `id` assiteix o no a classe

Finalment, el codi del *component* `App` és el següent:

{% tabs %}
{% tab title="Codi app.html" overflow="wrap" lineNumbers="true" %}
  ```html
  <div class="w3-container w3-teal">
      <h1>Professor: {{ teacher().firstName }}</h1>
  </div>

  <ul class="w3-ul">
      @for(student of students(); track student.id) {
          <li class="w3-row">
              <div class="w3-col l9">
                  {{ student.lastName1 }} {{ student.lastName2 }}, {{ student.firstName }}
              </div>
              <div class="w3-col l3">
                  <button (click)="onAssistance(student.id)" class="mybtn w3-btn w3-green w3-margin-right" [class.w3-disabled]="!isAttending(student.id)">Assisteix</button>
                  <button (click)="onMissing(student.id)" class="mybtn w3-btn w3-red" [class.w3-disabled]="isAttending(student.id)">Falta</button>
              </div>
          </li>
      }
  </ul>

  <router-outlet />
  ```
{% endtab %}

{% tab title="Codi app.css" overflow="wrap" lineNumbers="true" %}
```css
.mybtn {
    width: 128px;
}
```
{% endtab %}

{% tab title="Codi app.ts" overflow="wrap" lineNumbers="true" %}
  ```typescript
    import { Component, Signal } from '@angular/core';
    import { RouterOutlet } from '@angular/router';

    import { AttendanceListManager } from './service/attendance-list-manager';
    import { Student } from './model/student';
    import { Teacher } from './model/teacher';

    @Component({
    selector: 'app-root',
    imports: [RouterOutlet],
    templateUrl: './app.html',
    styleUrl: './app.css'
    })
    export class App {
    public students: Signal<Student[]>;
    public teacher: Signal<Teacher>;

    constructor(private _attendanceListManager: AttendanceListManager) {
        this.students = this._attendanceListManager.students;
        this.teacher = this._attendanceListManager.teacher
    }

    public onAssistance(id: string): void {
        this._attendanceListManager.addMissingRegister(id, true);
    }

    public onMissing(id: string): void {
        this._attendanceListManager.addMissingRegister(id, false);
    }

    public isAttending(id: string): boolean {
        return this._attendanceListManager.isAttending(id);
    }
    }
  ```
{% endtab %}
{% endtabs %}

Com es pot veure en el codi `TS` del *component* `App`, el *service* `AttendanceListManager` s'ha injectat creant un atribut `private` dins de la zona de paràmetres del constructor. Cal recordar que també es pot injectar amb el mètode `inject()`, tal com mostra el codi següent:

{% code title="Codi app.ts" overflow="wrap" lineNumbers="true" %}
```typescript
  import { Component, inject, Signal } from '@angular/core';
  import { RouterOutlet } from '@angular/router';

  import { AttendanceListManager } from './service/attendance-list-manager';
  import { Student } from './model/student';
  import { Teacher } from './model/teacher';

  @Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css'
  })
  export class App {
    public students: Signal<Student[]>;
    public teacher: Signal<Teacher>;

    private _attendanceListManager: AttendanceListManager = inject(AttendanceListManager);

    constructor() {
        this.students = this._attendanceListManager.students;
        this.teacher = this._attendanceListManager.teacher
    }

    public onAssistance(id: string): void {
        this._attendanceListManager.addMissingRegister(id, true);
    }

    public onMissing(id: string): void {
        this._attendanceListManager.addMissingRegister(id, false);
    }

    public isAttending(id: string): boolean {
        return this._attendanceListManager.isAttending(id);
    }
  }
```
{% encode %}
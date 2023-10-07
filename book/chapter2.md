# Components Angular
La creació de nou contingut en Angular (pàgines, una llista, una capçalera, etc.) es basa en els *components*. Així doncs, un component pot ser tota una nova pàgina
![La pàgina està formada per un únic component: AppComponent](img/wireframe_component1_1.png)
o podem crear una pàgina utilitzant múltiples components que podran ser reutilitzables
![L'AppComponent estructura tota la pàgina i, en el seu interior, conté el HeaderComponent i el CarouselComponent](img/wireframe_component1_2.png)

## Creació d'un Component Angular
Per crear un component d'Angular cal fer-ho utilitzant la comanda següent dins de la carpeta del projecte angular que, en aquest cas, suposarem que s'anomena `AngularProject`
```console
macervero@debian:~AngularProject$ ng generate component [ruta/nom_component]
```

Així doncs, si executem
```console
macervero@debian:~AngularProject$ ng generate component pages/home
```
es crearà el component `HomeComponent` dins de la ruta `src/app/pages` del nostre projecte.

Aquesta comanda crea els quatre fitxers que estan associats a un component:
- home.component.html: 
- home.component.ts
- home.component.css
- home.component.ts.spec
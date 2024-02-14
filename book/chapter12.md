# Capítol 12. Creació de pàgines (*pages*) i configuració del *Lazy Routing* amb Ionic
Ionic distingeix clarament el concepte *component* del concepte *page*, de tal manera que
* una *page* és la pàgina sencera, tot el que es mostrarà per pantalla, i
* un *component* és un element d'una pàgina que pot ser reutilitzable a diferents llocs de l'aplicació (per exemple, una llista, el *header*, el *footer*, etc.).

Per crear un component, la comanda que cal utilitzar és la mateixa que s'utilitza en Angular
```bash
ionic generate component path/component_name
```
Cal tenir en compte, però, que en aquest cas, l'opció `--skip-test` no funciona.

Per crear una pàgina, la comanda és la següent:
```bash
ionic generate page path/page_name
```
> En cas que el projecte inicial no sigui Ionic, sinó Angular amb la instal·lació d'Ionic, la comanda és la següent:
>```bash
>ng generate page path/page_name --skip-tests
>```

## Característiques d'una pàgina Ionic
Suposem, com a exemple, la creació de la pàgina inicial `home` mitjançant la comanda
```bash
ionic generate page view/pages/home
```
El procés implica el següent:
1. Creació del component `HomeComponent`
2. Configuració del *lazy routing*:
    1. Configuració de la ruta principal `home` a `app-routing.module.ts`
    2. Configuració del mòdul `home.module.ts`
    3. Configuració del mòdul d'enrutament `home-routing.module.ts`
3. Creació del codi `HTML` bàsic d'una pàgina Ionic
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>home</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
</ion-content>
```

Com podeu veure, tota pàgina Ionic està formada per dos grans components:
* la capçalera `<ion-header>` i
* el contingut de la pàgina en si mateixa `<ion-content>`

La capçalera, a més a més, té la definició d'una barra `<ion-toolbar>` on, a priori, només hi ha el títol `<ion-title>` però on, posteriorment, hi afegirem diverses icones i botons.
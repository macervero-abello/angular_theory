# Capítol 2. Creació de pàgines (*pages*) amb Ionic
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

{% hint style="info" %}
En cas que el projecte inicial no sigui Ionic, sinó Angular amb la instal·lació d'Ionic, la comanda és la següent:
```bash
ng generate page path/page_name --skip-tests
```
{% endhint %}

## Característiques d'una pàgina Ionic
Suposem, com a exemple, la creació de la pàgina inicial `home` mitjançant la comanda
```bash
ionic generate page view/home
```
El procés implica el següent:
1. Creació la pàgina `HomePage`
2. Configuració del *lazy routing*:
    1. Configuració de la ruta principal `home` a `app.routes.ts`
3. Creació del codi `HTML` bàsic d'una pàgina Ionic

{% code title="Codi home.page.html" overflow="wrap" lineNumbers="true" %}
  ```html
    <ion-header [translucent]="true">
      <ion-toolbar>
        <ion-title>home</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content [fullscreen]="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">home</ion-title>
        </ion-toolbar>
      </ion-header>
    </ion-content>
  ```
{% endcode %}

Com es pot veure, tota pàgina Ionic està formada per dos grans components:
* la capçalera `<ion-header>` i
* el contingut de la pàgina en si mateixa `<ion-content>`

La capçalera, a més a més, té la definició d'una barra `<ion-toolbar>` on, a priori, només hi ha el títol `<ion-title>` però on, posteriorment, s'hi poden afegir diverses icones i botons.
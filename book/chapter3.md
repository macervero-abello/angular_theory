# Capítol 3. Estils externs
Angular permet utilitzar les diverses llibreries CSS que existeixen al mercat per poder definir l'estil de les aplicacions. Sí que és cert, però, que no es recomana barrejar múltiples llibreries en un mateix projecte per les possibles incompatibilitats que puguin sorgir.

Per poder utilitzar una llibreria CSS dins del nostre projecte cal configurar-la, pas que es pot fer de múltiples maneres.

## Llibreria CSS W3.CSS
Per utilitzar aquesta llibreria en un projecte Angular cal seguir 3 passos:
1. Descarregar el fitxer w3.css i guardar-lo localment
2. Configurar el fitxer w3.css dins del projecte Angular

### Pas 1. Descàrrega del fitxer W3.CSS
Trobareu el fitxer w3.css a la pàgina [w3Schools](https://www.w3schools.com/), sota l'apartat [W3.CSS](https://www.w3schools.com/w3css/default.asp), dins de qualsevol requadre "*Try it yourself*".
[Enllaç directe al document](https://www.w3schools.com/w3css/4/w3.css)

Un cop descarregat el fitxer, cal guardar-lo dins del projecte, en concret, dins de la carpeta `assets/css`.

### Pas 2. Configuració del fitxer W3.CSS dins del projecte
La configuració del fitxer W3.CSS dins del projecte es pot fer de 3 maneres diferents
1. Modificant el fitxer `index.html` per tal d'afegir l'enllaç al fitxer dins de la seva capçalera (`<head>`)
2. Modificant el fitxer `styles.css` per incloure el fitxer W3.CSS
3. Modificant el fitxer de configuració `angular.json` per afegir la nova dependència

#### Modificació del fitxer `index.html`
És la manera més fàcil d'afegir els estils W3.CSS al projecte Angular, però també la més poc ortodoxa de totes.

Per fer-ho, només cal afegit l'etiqueta `<link>` corresponent dins de la capçalera del fitxer `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>FirstAngularProject</title>
    <base href="/">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" type="image/x-icon" href="favicon.ico">
    
    <!-- Enllaç al fitxer de la llibreria W3.CSS -->
    <link rel="stylesheet" href="assets/css/w3.css">
  </head>
  <body>
    <app-root></app-root>
  </body>
</html>
```

#### Modificació del fitxer `styles.css`
També és una manera molt senzilla d'afegir la llibreria W3.CSS al projecte Angular, tot i que tampoc és del tot ortodoxa.

Per fer-ho, s'utilitza la comanda CSS `@import url`

```css
/* You can add global styles to this file, and also import other style files */
@import url("assets/css/w3.css");

```

#### Modificació del fitxer `angular.json`
Aquesta metodologia és la més genuïna i autènticament Angular. Consisteix en modificar el fitxer de configuració `angular.json` per afegir la ruta al fitxer `w3.css` fins de l'apartat `architect` $\rightarrow$ `build` $\rightarrow$ `options` $\rightarrow$ `styles`

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "FirstAngularProject": {
      //...
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            //...
            "styles": [
              "src/styles.css",
              "src/assets/css/w3.css"
            ],
            "scripts": []
          },
          //...
        },
        //...
      }
    }
  }
}
```

{% hint style="warning" %}
**Atenció:** en cas que el servidor Angular `ng serve` estigués actiu, caldrà aturar-lo i tornar-lo a iniciar cada cop que es modifiqui el fitxer `angular.json`
{% endhint %}

## Llibreria CSS Bootstrap (*framework*)
El *framework* [Bootstrap](https://getbootstrap.com/) es pot instal·lar amb el gestor de paquets `npm` de `NodeJS`. Així doncs, per aplicar Bootstrap a un projecte Angular només fa falta fer 3 passos:
 1. Obrir una consola dins de la carpeta del projecte
 2. Instal·lar el *framework*
  ```bash
  npm install bootstrap@latest
  ```
 3. Configurar les rutes als fitxers CSS i JS dins de l'`angular.json`,
  ```json
  {
    //...
    "projects": {
      "Project": {
        //...
        "architect": {
          "build": {
            //...
            "options": {
              //...
              "styles": [
                "src/styles.css",
                "node_modules/bootstrap/dist/css/bootstrap.min.css"
              ],
              "scripts": [
                "node_modules/bootstrap/dist/js/bootstrap.min.js"
              ]
            },
            //...
          },
          //...
        }
      }
    }
  }
  ```

## Llibreria FontAwesome (icones)
La llibreria [FontAwesome](https://fontawesome.com/) conté un conjunt d'icones, algunes de pagament però moltes de lliures.

Per poder-la utilitzar en qualsevol projecte Angular només fa falta instal·lar la llibreria corresponent seguint les comandes següents:

```
npm install @fortawesome/fontawesome-svg-core
npm install @fortawesome/free-solid-svg-icons
npm install @fortawesome/angular-fontawesome@0.14
```

Vegeu la compatibilitat de les versions Angular i FontAwesome en aquesta [pàgina](https://www.npmjs.com/package/@fortawesome/angular-fontawesome)
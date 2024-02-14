# Capítol 13. Components de navegació amb Ionic
Ionic ofereix una gran quantitat de components d'interfície gràfica i una [documentació](https://ionicframework.com/docs/components) molt similar a altres frameworks, com, per exemple, Bootstrap, que en facilita molt el seu ús. Tot i això, el primer que cal aprendre és a gestionar-ne el sistema de navegació.

En tota aplicació mòbil (i web) existeixen tres mètodes de navegació bàsics:
1. Mitjançant botons (i enllaços)
2. Mitjançant un menú
3. Mitjançant pestanyes

A continuació s'expliquen les diverses metodologies

## *Routing*
Tal com passa amb qualsevol aplicació Angular, el component `AppComponent` fa de contenidor principal de les diverses pàgines que es mostren a través del sistema d'enrutament. Ara però, aquest cop, l'etiqueta encarregada de generar aquest contenidor no és la `<router-outlet></router-outlet>`, sinó que l'estructura `HTML` és la següent:
```bash
    <ion-app>
        <ion-router-outlet></ion-router-outlet>
    </ion-app>
```

L'etiqueta `<ion-app>` s'encarrega d'iniciarlitzar la part gràfica de l'aplicació i l'etiqueta `<ion-router-outlet>` actua de contenidor de rutes.

A part d'aquesta diferència, també cal tenir en compte dos aspectes addicionals:
1. l'enrutament per defecte que defineix Ionic és el *Lazy Routing* i
2. per generar una nova pàgina no crearem un `component`, sinó una `page`. Al cap i a la fi, una `page` no és res més que un `component`, però Ionic diferencia el concepte pàgina (`page`), com tot l'`HTML` que es mostra per una ruta en concret, i element d'una pàgina (`component`), com l'`HTML` corresponent a una part de la pàgina (una llista, la capçalera, el peu de pàgina, etc.). La comanda que cal utilitzar és la de `ionic generate page path/page_name`

## Navegació amb botons (i enllaços)

TODO

## Navegació amb menú

TODO

## Navegació amb pestanyes

TODO
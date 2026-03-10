# Capítol 14. Guardes de ruta
Angular ofereix un sistema de protecció de rutes per evitar que l'usuari pugui navegar lliurement per l'aplicació web (o mòbil) sense tenir en compte autenticació i permisos. Aquesta protecció es fa a traves de les *guardes de ruta*.

{% hint style="danger" %}
**Atenció!** La seguretat d'una aplicació no es pot basar només amb el control d'accés des de l'entorn client. Cal tenir en compte que qualsevol usuari mínimament viu pot arribar a modificar el codi Javascript (recordeu que TypeScript i, per tant, Angular, es compilen a Javascript) i acabar trencant les regles de protecció.

Aquesta seguretat sempre ha d'estar reforçada des de l'aplicació servidor mitjançant autenticació i gestió de permisos.
{% endhint %}

De *guardes de ruta* n'hi ha de 4 tipus, tal com es pot veure en aquest [enllaç](https://angular.dev/guide/routing/route-guards#types-of-route-guards).
* `CanActivate`: indica si un usuari pot accedir a una ruta o no (autenticació i permisos)
* `CanActivateChild`: donada una ruta principal, indica si un usuari pot accedir a les subrutes que en pengen (protecció de seccions completes que pengen d'una subruta)
* `CanDeactivate`: indica si un usuari pot sortir d'una ruta (evitar sortir de formularis amb dades que no s'han guardat)
* `CanMatch`: indica si a una ruta se li pot aplicar el procés de *path matching*

Podem veure, doncs, que per gestionar aquest tipus de guardes, prèviament, s'ha de crear un sistema de *routing* ([Captíol 10](chapter10.md)) i un sistema d'autenticació i de gestió de la sessió, per exemple, amb Firebase ([Capítol 13](chapter13.md)).

Ara però, en general, la guarda més útil per aplicacions mínimament senzilles és la `CanActivate`, ja que és la que aporta la seguretat associada a la gestió de la sessió i dels permisos en el moment de decidir si es pot accedir o no a una `URL` en concret.

## Contextualització de l'exemple pràctic
Suposem que tenim una aplicació amb les següents rutes
```typescript
   const routes: Routes = [
      { path: 'home', component: HomeComponent },
      { path: 'login', component: LoginComponent },
      { path: 'logout', component: LogoutComponent },
      { path: 'dashboard', component: DashboardComponent },
      { path: '', redirectTo: 'home', pathMatch: 'full' }
   ];
```
de tal manera que:
 * la ruta `/home` és pública i, per tant, hi pot accedir tothom;
 * la ruta `/dashboard` és privada i només s'hi pot accedir si s'ha iniciat sessió;
 * la ruta `/login` només serà accessible en cas que la sessió no estigui iniciada i
 * la ruta `/logout` només s'activarà si hi ha sessió oberta.

A més a més, aquesta aplicació també té el servei `AuthSessionService` tal com s'ha explicat en el [Captíol 13](chapter13.md).

A partir de tota aquesta arquitectura ja es poden afegir les guardes desitjades.

## Creació d'una guarda de ruta
Per crear una nova guarda de ruta cal executar la comanda
```bash
   ng generate guard path/guard_name
```
i escollir el tipus de guarda desitjada, en aquest cas, `CanActivate`. Cal recordar que es pot afegir l'opció `--skip-test` a la comanda per tal d'evitar la generació del fitxer de proves unitàries `spec.ts`.

Per exemple, si executem `ng generate guard guards/auth --skip-tests` per crear una guarda del tipus `CanActivate`, tal com mostra la figura,

![Creació d'una guarda de ruta de tipus `CanActivate`](img/guard_creation.png)

es generarà el fitxer `guards/auth-guard.ts`.
```typescript
   import { CanActivateFn } from '@angular/router';

   export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
      //Your code here
      return true;
   };
```

Com es pot comprovar, una ruta del tipus `CanActivate` no és res més que una funció que s'executa en el moment en què l'usuari intenta accedir a les rutes protegides. Segons el codi que s'hi implementi, la guarda permetra l'accés o, per contra, el denegarà i redigirà l'usuari a una altra pàgina.

## Implementació de la lògica de la guarda
Un cop creada la ruta, ja s'hi pot definir el codi per discernir si l'usuari pot accedir o no a les diverses pàgines, segons les normes establertes a l'apartat [Contextualització de l'exemple pràctic](#contextualització-de-lexemple-pràctic). En l'exemple que es mostra a continuació només es té en compte si l'usuari ha iniciat sessió o no, però, evidentment, guardes més complexes també poden tenir en compte el seu rol i, per tant, els permisos que té associats.

Per saber si l'usuari ha iniciat sessió o no caldrà utilitzar el mètode `isSessionActive()` del servei `AuthSessionService` (vegeu el [Capítol 13](chapter13.md)). Així doncs, cal injectar aquest servei a la funció de guarda però, com que és una funció, no es pot executar la injecció a través del constructor i s'ha de fer utilitzant el mètode `inject` que ofereix Angular:
```typescript
   import { inject } from '@angular/core';
   import { ActivatedRouteSnapshot, CanActivateFn, RouterStateSnapshot } from '@angular/router';
   import { AuthSessionService } from '../services/auth-session.service';

   export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
      const authSessionService: AuthSessionService = inject(AuthSessionService);
      return true;
   };
```

El paràmetre `route` que rep la funció de guarda té la propietat `url`, la qual és un `array` amb tots els fragments que formen l'`URL` a la qual s'intenta accedir. Per tant, aquest paràmetre permet discernir a quina pàgina s'està intentant entrar. Com que les rutes que cal protegir són `/login`, `/dashboard` i `/logout` (`/home` és pública), només cal gestionar aquestes:
```typescript
   import { inject } from '@angular/core';
   import { ActivatedRouteSnapshot, CanActivateFn, RouterStateSnapshot } from '@angular/router';
   import { AuthSessionService } from '../services/auth-session.service';

   export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
      const authSessionService: AuthSessionService = inject(AuthSessionService);

      if(route.url[0].path == "login") {
         //Gestió de l'accés a la pàgina login
      } else {
         //Gestió de l'accés a la resta de pàgines (dasboard i logout)
      }
      return true;
   };
```

S'ha definit que a la ruta `/login` només s'hi podrà accedir en cas que l'usuari no hagi iniciat sessió. En canvi, a les altres dues, només hi podrà entrar si ja ha estat autenticat, per tant:
```typescript
   import { inject } from '@angular/core';
   import { ActivatedRouteSnapshot, CanActivateFn, RouterStateSnapshot } from '@angular/router';
   import { AuthSessionService } from '../services/auth-session.service';

   export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
      const authSessionService: AuthSessionService = inject(AuthSessionService);

      if(route.url[0].path == "login") {
         if(authSessionService.isSessionActive()) {
            //S'intenta accedir a "/login" un cop iniciada la sessió => Redirecció a "/home"
            router.navigate(["/home"]);
            return false;
         } else {
            //S'intenta accedir a "/login" sense cap sessió iniciada => Accés permès
            return true;
         }
      } else {
         if(authSessionService.isSessionActive()) {
            //S'intenta accedir a "/dashboard" o a "/logout" amb la sessió iniciada => Accés permès
            return true;
         } else {
            //S'intenta accedir a "/dashboard" o a "/logout" sense sessió iniciada => Redirecció a "/login"
            router.navigate(["/login"]);
            return false;
         }
      }
   };
```

## Protecció de les rutes
Un cop definida la guarda cal aplicar-la a les rutes i, per tant, modificar el fitxer `app.routes.ts` de la manera següent:
```typescript
   const routes: Routes = [
      { path: 'home', component: HomeComponent },
      { path: 'login', component: LoginComponent, canActivate: [authGuard] },
      { path: 'logout', component: LogoutComponent, canActivate: [authGuard] },
      { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] },
      { path: '', redirectTo: 'home', pathMatch: 'full' }
   ];
```

Totes aquelles rutes que han de quedar protegides per la guarda necessiten definir la propietat `canActivate`, el valor de la qual és un `array` que conté totes aquelles guardes que es volen aplicar, en el cas de l'exemple, només `authGuard`.
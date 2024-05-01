# Capítol 19. Lector de codis de barres
<!--https://ionic.io/blog/how-to-build-an-ionic-barcode-scanner-with-capacitor

https://github.com/robingenz/ionic-capacitor-barcode-scanner-->

Tal com s'ha dit en el [Capítol 17](chapter17.md), Capacitor és una eina de l'ecosistema Ionic que no només permet compilar codi Ionic i crear l'executable segons la plataforma per la qual s'estigui implementant, sinó que també ofereix un conjunt de [*pluguins*](https://capacitorjs.com/docs/plugins) que permeten l'accés directe al *hardware* del dispositiu. Aquests *pluguins* poden ser oficials (fets per l'equip desenvolupador de Capacitor) o de la comunitat (fets per desenvolupadors externs i validats per l'equip de l'ecosistema d'Ionic).

En concret, dins dels *pluguins* de comunitat en trobem un que permet utilitzar la càmara del mòbil per crear un lector de codi de barres, sigui del tipus que sigui (EAN13, QR, ISBN, etc.): [Capacitor ML Kit Barcode Scanning Plugin](https://capawesome.io/plugins/mlkit/barcode-scanning/).

Els passos a seguir per aconseguir implementar el lector són els següents:
1. Instal·lació del plugin
2. Creació del codi de l'aplicació
3. Instal·lació de les plataformes de compilació desitjades (Android o IOS)
4. Configuració dels fitxers descriptius d'aquestes plataformes
5. Compliació, creació i execució de l'aplicació final

## Instal·lació del *pluguin* Capacitor ML Kit Barcode Scanning Plugin
Per poder instal·lar aquest *pluguin* cal comprovar que el projecte Ionic té instal·lat Capacitor. Si aquest no és el cas, caldrà instal·lar-lo seguint els passos indicats al [primer apartat del Capítol 17](chapter17.md#installar-capacitor).

Fet això, ja es pot procedir a instal·lar el *pluguin* mitjançant la comanda següent:
```bash
    npm install @capacitor-mlkit/barcode-scanning
```

## Creació del codi de l'aplicació
Un cop instal·lat el *pluguin* necessari, és important analitzar-ne la [documentació](https://capawesome.io/plugins/mlkit/barcode-scanning/) i, més especifícament, la seva [API](https://capawesome.io/plugins/mlkit/barcode-scanning/#api), és a dir, totes les funcions que ofereix per tal de poder fer un lector de codi de barres.

D'entre tots aquests mètodes, els més importants són els següents:
1. [`isSupported()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#issupported): indica si el dispositiu on es vol executar el lector suporta aquesta funcionalitat
2. [`requestPermission()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#requestpermissions): demana permís per a utilitzar la càmera del dispositiu per poder fer la captura del codi de barres
3. [`scan()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#scan): realitza la lectura del codi de barres tenint en compte un conjunt d'opcions de configuració ([`ScanOptions`](https://capawesome.io/plugins/mlkit/barcode-scanning/#scanoptions)) que especifiquen, bàsicament, els [formats de codi](https://capawesome.io/plugins/mlkit/barcode-scanning/#barcodeformat) que es poden llegir.

El primer que caldrà fer és implementar el `BarcodeScannerService` per tal que ofereixi les funcions bàsiques per inicialitzar el lector (la càmera del sistema) i fer la captura del codi de barres:

```bash
ionic generate service service/barcode-scanner --skip-tests
```

Un cop implementat el *service*, només faltarà implementar la part de la *view* en una de les pàgines de l'aplicació.

### Comprovació del suport que ofereix el dispositiu per fer la lectura
La primera funció necessària és la que comprova si el dispositiu suporta la funcionalitat:

```typescript
import { Injectable } from '@angular/core';
import { BarcodeScanner, IsSupportedResult } from '@capacitor-mlkit/barcode-scanning';

@Injectable({
  providedIn: 'root'
})
export class BarcodeScannerService {

  private _supported: boolean;

  constructor() {
    this._supported = false;
    this.isSupported();
  }

  isSupported(): void {
    BarcodeScanner.isSupported().then(
      (result: IsSupportedResult) => {
        this._supported = result.supported;
      }
    ).catch(
      (error: any) => {
        this._supported = false;
      }
    ).finally(() => {});
  }

  get supported(): boolean {
    return this._supported
  }
}
```

Així doncs, el *service* `BarcodeScannerService` tindrà el mètode `isSupported()` que delegarà la comprovació al mètode `isSupported()` de la classe `BarcodeScanner` que ofereix el *pluguin Capacitor ML Kit Barcode Scanning Plugin*. Aquest mètode retorna una `Promise<IsSupportedResult>` que cal tractar per tal d'inicialitzar l'atribut private `_supported`.

Un cop feta aquesta implementació només fa falta
1. cridar el mètode `isSupported()` des del constructor, d'aquesta manera, la comprovació es fa de manera immediata en el moment en què s'inicialitza l'aplicació, i
2. crear un mètode *getter* per tal de poder consultar el valor de l'atribut privat `_supported`.


### Comprovació de la configuració de permisos
Un cop l'aplicació s'ha assegurat de què el dispositiu té capacitat per executar el lector de codi de barres, cal comprovar els permisos del sistema per tal de poder accedir a la càmera. Per tant, s'ha d'afegir el mètode `requestPermissions()` al `BarcodeScannerService`.

```typescript
import { Injectable } from '@angular/core';
import { BarcodeScanner, IsSupportedResult, PermissionStatus } from '@capacitor-mlkit/barcode-scanning';

@Injectable({
  providedIn: 'root'
})
export class BarcodeScannerService {

  private _supported: boolean;

  constructor() {...}
  isSupported(): void {...}
  get supported(): boolean {...}

  async requestPermissions(): Promise<boolean> {
    const permissions: PermissionStatus = await BarcodeScanner.requestPermissions();
    return permissions.camera === 'granted' || permissions.camera === 'limited';
  }
}
```

Aquest mètode accedeix a la funcionalitat `requestPermission()` del *pluguin Capacitor ML Kit Barcode Scanning Plugin*, la qual retorna una [`Promise<PermissionStatus>`](https://capawesome.io/plugins/mlkit/barcode-scanning/#permissionstatus). L'objecte `PermissionStatus` té un únic atribut `camera` que pot tenir 4 estats diferents:
* `'prompt'`
* `'prompt-with-rationale'`
* `'granted'`
* `'denied'`
* `'limited'`
Només els estats `'granted'` i '`limited`' asseguren que l'aplicació pugui utilitzar la càmera, per tant, el mètode del *service* `BarcodeScannerService` només retornarà `true` en aquests casos i `false` en la resta.

Cal tenir en compte que, com que no es pot fer anar el lector sense tenir els permisos necessaris, la `Promise<PermissionStatus>` que retorna el mètode `requestPermission()`  es tractarà de manera seqüèncial mitjançant un `async-await`.

### Lectura del codi
Finalment, un cop l'aplicació s'ha assegurat que té el suport i els permisos necessaris per poder executar el lector, ja pot implementar el mètode `scan()` per fer la captura del codi de barres.

```typescript
import { Injectable } from '@angular/core';
import { Barcode, BarcodeFormat, BarcodeScanner, IsSupportedResult, PermissionStatus, ScanOptions, ScanResult } from '@capacitor-mlkit/barcode-scanning';

@Injectable({
  providedIn: 'root'
})
export class BarcodeScannerService {

  private _supported: boolean;
  private _barcodes: Barcode[];

  constructor() {
    this._supported = false;
    this._barcodes = [];
    this.isSupported();
  }

  isSupported(): void {...}
  get supported(): boolean {...}
  async requestPermissions(): Promise<boolean> {...}

  async scan(): Promise<boolean> {
    const granted = await this.requestPermissions();
    if(granted) { 
      const options: ScanOptions = {
        formats: [BarcodeFormat.Ean13, BarcodeFormat.QrCode]
      }

      const result: ScanResult = await BarcodeScanner.scan(options);
      this._barcodes = result.barcodes;

      return true;
    }
    this._barcodes = [];
    return false;
  }

  get barcodes(): Barcode[] {
    return this._barcodes;
  }
}
```

Tal com passa en els casos anteriors, el mètode `scan()` del *service* `BarcodeScannerService` crida a la funció `scan()` del *pluguin Capacitor ML Kit Barcode Scanning Plugin*. Prèviament, però, s'ha assegurat que l'aplicació té els permisos necessaris invocant al mètode `requestPermissions()`.

Pel que fa al mètode `scan()` del *plugin*, cal tenir en compte diversos aspectes:
1. Pot rebre un paràmetre de tipus `ScanOptions`. Si el rep, aquest paràmetre especificarà quins formats de codi de barres serà capaç de llegir el lector fent que l'aplicació tinqui capacitats més limitades però, en canvi, sigui més eficient; si no el rep, el lector podrà llegir tots els [formats disponibles](https://capawesome.io/plugins/mlkit/barcode-scanning/#barcodeformat), la qual cosa farà que l'aplicació sigui molt adaptable però, en canvi, més ineficient.
2. Retorna un objecte de tipus `Promise<ScanResult>`, el qual conté un *array* de codis de barres (les dades llegides).

Per acabar d'arrodonir el codi, el *service* necessita tenir un atribut privat per guardar els `Barcode` que retorna l'`ScanResult` del mètode `scan()` i el mètode *getter* corresponent.

### Implementació de la vista
El lector de codi de barres es pot inserir en qualsevol de les pàgines de l'aplicació. En aquest exemple, però, es crearà una pàgina, conjuntament amb la ruta corresponent, específica mitjançant la comanda:

```bash
ionic generate page view/barcode-scanner --skip-tests
```

Un cop fet això, només necessitem crear una pàgina que tingui un botó que activi la càmera per poder llegir el codi de barres desitjat i un llistat que mostri el contingut del codi llegit. Aquest botó només estarà actiu si el dispositiu suporta la funcionalitat.

{% tabs %}
{% tab title="Codi barcode-scanner.page.html" %}
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>
      Lector de codis de barres
    </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-list>
    <ion-item *ngFor="let barcode of barcodes">
      <ion-label position="stacked">{{ barcode.format }}</ion-label>
      <ion-input type="text" [value]="barcode.rawValue"></ion-input>
    </ion-item>
  </ion-list>
  <ion-fab slot="fixed" vertical="bottom" horizontal="end">
    <ion-fab-button (click)="scan()" [disabled]="!isSupported">
      <ion-icon name="scan"></ion-icon>
    </ion-fab-button>
  </ion-fab>
</ion-content>
```
{% endtab %}
{% tab title="Codi barcode-scanner.page.ts" %}
```typescript
import { Component } from '@angular/core';
import { Barcode } from '@capacitor-mlkit/barcode-scanning';
import { BarcodeScannerService } from '../service/barcode-scanner.service';

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
})
export class BarcodeScannerPage {

  constructor(private _barcodeScanner: BarcodeScannerService) {}

  async scan(): Promise<boolean> {
    let done: boolean = await this._barcodeScanner.scan();
    return done;
  }

  get isSupported(): boolean {
    return this._barcodeScanner.supported;
  }

  get barcodes(): Barcode[] {
    return this._barcodeScanner.barcodes;
  }
}
```
{% endtab %}
{% endtabs %}

Com es pot comprovar, el codi d'aquesta pàgina injecta el service que s'ha implementat prèviament (`BarcodeScannerService`) i només té 3 mètodes:
1. `isSupported()`: per comprovar si el dispositiu té capacitat per llegir el codi de barres
2. `scan()`: activa la càmera i fa la lectora
3. `barcodes()`: el *getter* que permet retornar els codis llegits (per accedir al contingut descodificat, s'ha de cridar a l'atribut rawValue de la interfície [`Barcode`](https://capawesome.io/plugins/mlkit/barcode-scanning/#barcode), tal com mostra el codi HTML).

### Resolució de problemes
En segons quines versions d'Android, sigui per la versió del sistema operatiu o pel software instal·lat pel model de mòbil en concret, quan s'intenta utilitzar el lector de codis de barres apareix aquest error en els *logs* de l'aplicació:

```bash
Error: The Google Barcode Scanner Module is not available. You must install it first
```

Si aquest és el vostre cas, cal modificar una mica el codi del `BarcodeScannerService` per gestionar la instal·lació del mòdul de l'escàner de Google. Així doncs, el nou *service* quedarà de la manera següent:

```typescript
import { Injectable } from '@angular/core';
import { Barcode, BarcodeFormat, BarcodeScanner, IsSupportedResult, PermissionStatus, ScanOptions, ScanResult } from '@capacitor-mlkit/barcode-scanning';

@Injectable({
  providedIn: 'root'
})
export class BarcodeScannerService {

  private _supported: boolean;
  private _moduleAvailable: boolean;
  private _barcodes: Barcode[];

  constructor() {
    this._supported = false;
    this._moduleAvailable = false;
    this._barcodes = [];
    this.isSupported();
  }

  isSupported(): void {...}
  get supported(): boolean {...}
  async requestPermissions(): Promise<boolean> {...}

  async checkGoogleGarcodeScannerModule(): Promise<void> {
    BarcodeScanner.isGoogleBarcodeScannerModuleAvailable().then(
      (avaliableModule: IsGoogleBarcodeScannerModuleAvailableResult) => {
        this._moduleAvailable = avaliableModule.available;
        if(!this._moduleAvailable) BarcodeScanner.installGoogleBarcodeScannerModule();
      }
    );
  }

  get moduleAvailable() {
    if(!this._moduleAvailable) {
      BarcodeScanner.isGoogleBarcodeScannerModuleAvailable().then(
        (avaliableModule: IsGoogleBarcodeScannerModuleAvailableResult) => {
          this._moduleAvailable = avaliableModule.available;
        }
      );
    }

    return this._moduleAvailable;
  }

  async scan(): Promise<boolean> {...}
  get barcodes(): Barcode[] {...}
}
```

S'afegeixen dos nous mètodes `checkGoogleGarcodeScannerModule()` i `get moduleAvailable()`. El primer s'encarrega de comprovar si el mòdul està instal·lat al sistema i, en cas que no ho estigui, l'instal·la en 2n pla. El segon mètode és un getter per poder consultar si el mòdul ja ha estat instal·lat i, per tant, ja està disponible per a poder-se utilitzar.

Un cop modificat el *service* només cal utilitzar les noves funcions dins de les vistes. Així doncs, tan bon punt s'inicialitzi l'aplicació s'executarà el mètode `checkGoogleGarcodeScannerModule()` a l'`AppComponent` per assegurar que el mòdul queda instal·lat al més aviat possible:

```typescript
import { Component } from '@angular/core';
import { BarcodeScannerService } from './service/barcode-scanner.service';

@Component({
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.scss'],
})
export class AppComponent {
  constructor(private _barcodeScanner: BarcodeScannerService) {
    this._barcodeScanner.checkGoogleGarcodeScannerModule();
  }
}
```

Fet això, també es modificarà la vista des d'on s'executa el lector (`barcode-scanner.page.ts`) per comprovar si el mòdul ja està disponible o no:
```typescript
import { Component } from '@angular/core';
import { Barcode } from '@capacitor-mlkit/barcode-scanning';
import { BarcodeScannerService } from '../service/barcode-scanner.service';

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
})
export class BarcodeScannerPage {

  constructor(private _barcodeScanner: BarcodeScannerService) {}

  async scan(): Promise<boolean> {...}

  get isSupported(): boolean {
    return this._barcodeScanner.supported && this._barcodeScanner.moduleAvailable;
  }

  get barcodes(): Barcode[] {...}
}
```

## Instal·lació de les plataformes de compilació desitjades (Android o IOS)
Un cop el codi de l'aplicació ja està implementat, cal preparar tot el sistema per tal que pugui ser compilat i executat en les plataformes desitjades. 

### Plataforma Android
Si volem crear l'aplicació per a Android, caldrà executar la comanda:

```bash
  ionic capacitor add android
  ionic capacitor sync
```

La primera instrucció crearà la carpeta `android`, preparada per a ser oberta des d'*AndroidStudio*, amb tots els fitxers necessaris per poder configurar i compilar l'aplicació per a dispositius Android. La segona actualitzarà els fitxers de la carpeta amb la última versió del nostre codi *Ionic/Anguar*.

### Plataforma IOS
Si es vol treballar sobre IOS, la comanda serà la següent:

```bash
  ionic capacitor add ios
  ionic capacitor sync
```

La primera instrucció crearà la carpeta `ios`, preparada per a ser oberta des d'*XCode*, amb tots els fitxers necessaris per poder configurar i compilar l'aplicació per a dispositius IOS. La segona actualitzarà els fitxers de la carpeta amb la última versió del nostre codi *Ionic/Anguar*.

## Configuració dels fitxers descriptius d'aquestes plataformes
Després d'instal·lar la plataforma (o plataformes) desitjades, cal modificar-ne els fitxers de configuració per tal que les aplicacions finals tinguin els permisos necessaris per poder-se executar i fer anar la càmera del dispositiu.

### Configuració de la plataforma Android
Per configurar l'aplicació Android cal modificar el fitxer *AndroidManifest.xml*, el qual es troba dins de la carpeta `android/app/src/main`. Cal tenir en compte que qualsevol modificació d'aquest fitxer s'ha de fer amb el servidor *on-the-fly* tancat i que, per provar-ne els resultats, cal, prèviament, desinstal·lar l'aplicació del mòbil, en cas que ja hi estigués instal·lada.

Les modificaions a fer-hi són les següents:
1. Afegir meta dades dins de l'etiqueta `<application>`

```html
  <meta-data android:name="com.google.mlkit.vision.DEPENDENCIES" android:value="barcode_ui"/>
```

2. Afegir els permisos per a l'ús de la càmera i del flash

```html
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-permission android:name="android.permission.FLASHLIGHT"/>
```

Per tant, el fitxer final queda de la manera següent:

```html
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        ...

        <meta-data android:name="com.google.mlkit.vision.DEPENDENCIES" android:value="barcode_ui"/>
    </application>

    <!-- Permissions -->

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.FLASHLIGHT"/>
</manifest>
```

### Configuració de la plataforma IOS
Per configurar l'aplicació IOS cal modificar el fitxer *Info.plist*, el qual es troba dins de la carpeta `ios/App/App`. Cal tenir en compte que qualsevol modificació d'aquest fitxer també s'ha de fer amb el servidor *on-the-fly* tancat i que, per provar-ne els resultats, cal, prèviament, desinstal·lar l'aplicació del mòbil, en cas que ja hi estigués instal·lada.

En aquest cas, només fa falta afegir el permís d'ús de la càmera:

```html
  <key>NSCameraUsageDescription</key>
  <string>The app enables the scanning of various barcodes.</string>
```

## Compliació, creació i execució de l'aplicació final
Si ja s'han fet tots els passos anteriors, l'aplicació ja pot ser compilada, executada i provada. Per fer-ho, es poden seguir els passos del [Capítol 17](./chapter17.md) i, d'aquesta manera poder utilitzar la funcionalitat *Live Reload* sobre el mòbil o, en canvi, crear i instal·lar l'APK directament, tal com s'explica al [Capítol 18](./chapter18.md).
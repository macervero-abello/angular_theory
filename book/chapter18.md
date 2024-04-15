# Capítol 18. Lector de codis de barres
<!--https://ionic.io/blog/how-to-build-an-ionic-barcode-scanner-with-capacitor

https://github.com/robingenz/ionic-capacitor-barcode-scanner-->

[comment]: # (blablabla)

Tal com s'ha dit en el [Capítol 15](chapter15.md), Capacitor és una eina de l'ecosistema Ionic que no només permet compilar codi Ionic i crear l'executable segons la plataforma per la qual s'estigui implementant, sinó que també ofereix un conjunt de [*pluguins*](https://capacitorjs.com/docs/plugins) que permeten l'accés directe al *hardware* del dispositiu. Aquests *pluguins* poden ser oficials (fets per l'equip desenvolupador de Capacitor) o de la comunitat (fets per desenvolupadors externs i validats per l'equip de l'ecosistema d'Ionic).

En concret, dins dels *pluguins* de comunitat en trobem un que permet utilitzar la càmara del mòbil per crear un lector de codi de barres, sigui del tipus que sigui (EAN13, QR, ISBN, etc.): [Capacitor ML Kit Barcode Scanning Plugin](https://capawesome.io/plugins/mlkit/barcode-scanning/).

Els passos a seguir per aconseguir implementar el lector són els següents:
1. Instal·lació del plugin
2. Creació del codi de l'aplicació
3. Instal·lació de les plataformes de compilació desitjades (Android o IOS)
4. Configuració dels fitxers descriptius d'aquestes plataformes
5. Compliació, creació i execució de l'aplicació final

## Instal·lació del *pluguin* Capacitor ML Kit Barcode Scanning Plugin
Per poder instal·lar aquest *pluguin* cal comprovar que el projecte Ionic té instal·lat Capacitor. Si aquest no és el cas, caldrà instal·lar-lo seguint els passos indicats al [primer apartat del Capítol 16](chapter16.md#installar-capacitor).

Fet això, ja es pot procedir a instal·lar el *pluguin* mitjançant la comanda següent:
```bash
    npm install @capacitor-mlkit/barcode-scanning
```

## Creació del codi de l'aplicació
Un cop instal·lat el *pluguin* necessari, és important analitzar-ne la [documentació](https://capawesome.io/plugins/mlkit/barcode-scanning/) i, més especifícament, la seva [API] (https://capawesome.io/plugins/mlkit/barcode-scanning/#api), és a dir, totes les funcions que ofereix per tal de poder fer un lector de codi de barres.

D'entre tots aquests mètodes, els més importants són els següents:
1. [`isSuported()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#issupported): indica si el dispositiu on es vol executar el lector suporta aquesta funcionalitat
2. [`requestPermission()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#requestpermissions): demana permís per a utilitzar la càmera del dispositiu per poder fer la captura del codi de barres
3. [`scan()`](https://capawesome.io/plugins/mlkit/barcode-scanning/#scan): realitza la lectura del codi de barres tenint en compte un conjunt d'opcions de configuració ([`ScanOptions`](https://capawesome.io/plugins/mlkit/barcode-scanning/#scanoptions)) que especifiquen, bàsicament, els [formats de codi](https://capawesome.io/plugins/mlkit/barcode-scanning/#barcodeformat) que es poden llegir.

El primer que caldrà fer és implementar el *BarcodeScannerService* per tal que ofereixi les funcions bàsiques per inicialitzar el lector (la càmera del sistema) i fer la captura del codi de barres:

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

  private _suported: boolean;

  constructor() {
    this._suported = false;
    this.isSuported();
  }

  isSuported(): void {
    BarcodeScanner.isSupported().then(
      (result: IsSupportedResult) => {
        this._suported = result.supported;
      }
    ).catch(
      (error: any) => {
        this._suported = false;
      }
    ).finally(() => {});
  }

  get suported(): boolean {
    return this._suported
  }
}
```

Així doncs, el *service* `BarcodeScannerService` tindrà el mètode `isSuported()` que delegarà la comprovació al mètode `isSuported()` de la classe `BarcodeScanner` que ofereix el *pluguin Capacitor ML Kit Barcode Scanning Plugin*. Aquest mètode retorna una `Promise<IsSupportedResult>` que cal tractar per tal d'inicialitzar l'atribut private `_suported`.

Un cop feta aquesta implementació només fa falta
1. cridar el mètode `isSuported()` des del constructor, d'aquesta manera, la comprovació es fa de manera immediata en el moment en què s'inicialitza l'aplicació, i
2. crear un mètode *getter* per tal de poder consultar el valor de l'atribut privat `_suported`.


### Comprovació de la configuració de permisos
Un cop l'aplicació s'ha assegurat de què el dispositiu té capacitat per executar el lector de codi de barres, cal comprovar els permisos del sistema per tal de poder accedir a la càmera. Per tant, s'ha d'afegir el mètode `requestPermissions()` al `BarcodeScannerService`.

```typescript
import { Injectable } from '@angular/core';
import { BarcodeScanner, IsSupportedResult, PermissionStatus } from '@capacitor-mlkit/barcode-scanning';

@Injectable({
  providedIn: 'root'
})
export class BarcodeScannerService {

  private _suported: boolean;

  constructor() {...}
  isSuported(): void {...}
  get suported(): boolean {...}

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

  private _suported: boolean;
  private _barcodes: Barcode[];

  constructor() {
    this._suported = false;
    this._barcodes = [];
    this.isSuported();
  }

  isSuported(): void {...}
  get suported(): boolean {...}
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
ionic generate page view/barcode-scanner
```
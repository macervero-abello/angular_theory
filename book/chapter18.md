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
    this._supported = false;
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
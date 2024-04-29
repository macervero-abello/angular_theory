# Capítol 20. Sistema de fitxers
<!-- 
    https://ionicframework.com/docs/native/filesystem
    https://capacitorjs.com/docs/apis/filesystem
-->

Un altre dels *plugins* que ofereix Capacitor, en aquest cas oficial o nadiu, és el que permet accedir al sistema de fitxers (*filesystem*) del dispositiu amb el que s'estigui treballant.

Els passos a seguir per aconseguir implementar una bona gestió dels fitxers generats per l'aplicació són els següents:
1. Instal·lació del plugin
2. Creació del codi de l'aplicació
3. Instal·lació de les plataformes de compilació desitjades (Android o IOS)
4. Configuració dels fitxers descriptius d'aquestes plataformes
5. Compliació, creació i execució de l'aplicació final

## Instal·lació del *pluguin Filesystem*
Per poder instal·lar aquest *pluguin* cal comprovar que el projecte Ionic té instal·lat Capacitor. Si aquest no és el cas, caldrà instal·lar-lo seguint els passos indicats al [primer apartat del Capítol 17](chapter17.md#installar-capacitor).

Fet això, ja es pot procedir a instal·lar el *pluguin* mitjançant la comanda següent:
```bash
    npm install @capacitor/filesystem
```

## Creació del codi de l'aplicació
Un cop instal·lat el *pluguin* necessari, és important analitzar-ne la [documentació](https://capawesome.io/plugins/mlkit/barcode-scanning/) i la seva [API](https://capacitorjs.com/docs/apis/filesystem#api) per saber totes les funcionalitats.

Els mètodes més importants són els següents:
1. *CRUD* d'un fitxer:
  * [`writeFile()`](https://capacitorjs.com/docs/apis/filesystem#writefile): escriu les dades a fitxer; en cas que el fitxer no existeixi dins del sistema, el crea.
  * [`appendFile()`](https://capacitorjs.com/docs/apis/filesystem#appendfile): afegeix dades al final del fitxer
  * [`readFile()`](https://capacitorjs.com/docs/apis/filesystem#readfile): llegeis les dades de fitxer
  * [`deleteFile()`](https://capacitorjs.com/docs/apis/filesystem#deletefile): esborra el fitxer del sistema
  * [`downloadFile()`](https://capacitorjs.com/docs/apis/filesystem#downloadfile): descarrega un fitxer des d'un servidor i l'emmagatzema dins del sistema
2. *CRUD* d'un directori:
  * [`mkdir()`](https://capacitorjs.com/docs/apis/filesystem#mkdir): crea un nou directori dins del sistema
  * [`rmdir()`](https://capacitorjs.com/docs/apis/filesystem#rmdir): esborra el directori del sistema
  * [`readdir()`](https://capacitorjs.com/docs/apis/filesystem#readdir): llista el contingut d'un directori
3. Gestió general de fitxers i directoris:
  * [`getUri()`](https://capacitorjs.com/docs/apis/filesystem#geturi): retorna l'adreça absoluta del fitxer o directori
  * [`stat()`](https://capacitorjs.com/docs/apis/filesystem#stat): retorna les propietats del fitxer o directori
  * [`rename()`](https://capacitorjs.com/docs/apis/filesystem#rename): canvia el nom (i la ubicació, si cal) del fitxer o directori
  * [`copy()`](https://capacitorjs.com/docs/apis/filesystem#copy): copia el fitxer o directori a una altra ubicació
4. Gestió de permisos:
  * [`checkPermissions()`](https://capacitorjs.com/docs/apis/filesystem#checkpermissions): comprova si l'aplicació té els permisos necessaris per poder gestionar el sistema de fitxers (aquest mètode només s'ha d'invocar des d'Android si es vol accedir al directori `Documents` o a l'emmagatzematge extern - `ExternalStorage`)
  * [`requestPermissions()`](hhttps://capacitorjs.com/docs/apis/filesystem#requestpermissions): delana els permisos necessaris per poder gestionar el sistema de fitxers (aquest mètode només s'ha d'invocar des d'Android si es vol accedir al directori `Documents` o a l'emmagatzematge extern - `ExternalStorage`)

El primer que caldrà fer és implementar el `FilesystemService` per tal que ofereixi les funcions bàsiques per gestionar el sistema de fitxers:

```bash
ionic generate service service/filesystem --skip-tests
```

Un cop ja s'hagi implementat el *service*, només faltarà implementar la part de la *view* en una de les pàgines de l'aplicació.


### Directoris que ofereix el *plugin Filesystem*
El *plugin Filesystem* ofereix un conjunt de [directoris](https://capacitorjs.com/docs/apis/filesystem#directory) per tal que les aplicacions instal·lades al dispositiu puguin gestionar els seus propis fitxers.

1. `Documents`
    * En el sistema IOS correspon al directori *documents* de la pròpia aplicació
    * En el sistema Android correspon a la carpeta pública *Documents* i l'aplicació només té accés als fitxers i carpetes que ha creat ella mateixa (per accedir-hi cal gestionar la configuració de permisos)
2. `Data`
    * En el sistema IOS equival al directori `Documents`
    * En el sistema Android és el directori que conté els fitxers de l'aplicació (s'esborra quan l'aplicació es desinstal·la)
3. `Library`
    * En el sistema IOS correspon a la carpeta *library*
    * En el sistema Android equival al directori `Data`
4. `Cache`: correspon a la cache del sistema
5. `External`
    * En el sistema IOS equival al directori `Documents`
    * En el sistema Android correspon al dispositiu (disc dur) d'emmagatzematge primari, sigui compartit o extern (els fitxers que s'hi creen s'esborren quan l'aplicació es desinstal·la)
6. `ExternalStorage`
    * En el sistema IOS equival al directori `Documents`
    * En el sistema Android correspon al directori d'emmagatzematge primari, sigui compartit o extern (per accedir-hi des d'un Android 10 cal gestionar la configuració de permisos; Androids més nous ja no hi tenen accés)


### Escriptura de dades a un fitxer
La primera funció que es pot implementar és la d'escriptura a fitxer:

```typescript
import { Injectable } from '@angular/core';
import { Directory, Encoding, Filesystem, WriteFileResult } from '@capacitor/filesystem';

@Injectable({
  providedIn: 'root'
})
export class FilesystemService {

  private _path: string = "buy_list.txt"
  private _idata: any = {
    'greengrocer': ['enciam', 'tomàquets'],
    'supermarket': ['pizza congelada', 'iogurts']
  };

  constructor() {}

  async writeToFile(): Promise<boolean> {
    let result: WriteFileResult = await Filesystem.writeFile({
        path: this._path,
        data: JSON.stringify(this._idata),
        directory: Directory.Documents,
        encoding: Encoding.UTF8
    });
    
    if(result.uri && result.uri != '') return true;
    else {
        console.log("Error d'escriptura");
        return false;
    }
  }
}
```

El *service* `FilesystemService` tindrà el mètode `writeToFile()` que delegarà l'escriptura al mètode `writeFile()` de la classe `Filesystem` que ofereix el *pluguin Filesystem*. Aquest mètode retorna una `Promise<WriteFileResult>` que cal tractar per tal d'esbrinar si l'escriptura s'ha pogut dur a terme o no.

En aquesta implementació cal tenir en compte diverses coses:
1. La ruta del fitxer (`this._path`) i/o les dades a escriure (`this._idata`) poden venir determinades per l'usuari, per tant, en comptes de ser atributs s'hauran de passar per paràmetre al mètode `writeToFile()`.
2. Les dades que s'han d'escriure a fitxer han d'estar en format `string`, per tant, si cal escriure tot un objecte a fitxer, prèviament caldrà aplicar-li un `toString()` o un `JSON.stringify()`.
3. Els directoris on es poden emmagatzemar les dades són els llistats en l'[apartat anterior](#directoris-que-ofereix-el-plugin-filesystem)
4. La [codificació](https://capacitorjs.com/docs/apis/filesystem#encoding) del fitxer pot estar en `ASCII`, `UTF-8` o `UTF16`.

### Lectura de dades d'un fitxer
Un cop l'aplicació ja té implementada la funcionalitat per escriure dades a fitxer, també cal que les sàpiga llegir per poder-les recuperar:

```typescript
import { Injectable } from '@angular/core';
import { Directory, Encoding, Filesystem, ReadFileResult, WriteFileResult } from '@capacitor/filesystem';

@Injectable({
  providedIn: 'root'
})
export class FilesystemService {

  private _path: string = "buy_list.txt"
  private _idata: any = {
    'greengrocer': ['enciam', 'tomàquets'],
    'supermarket': ['pizza congelada', 'iogurts']
  };
  private _odata: any;


  constructor() {}

  async writeToFile(): Promise<boolean> {...}

  async readFromFile(): Promise<boolean> {
    let result: ReadFileResult = await Filesystem.readFile({
      path: this._path,
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    });

    if(result.data) {
      this._odata = result.data;
      if(!(this._odata instanceof Blob)) this._odata = JSON.parse(this._odata);
      return true;
    } else {
      return false;
    }
  }

  get data(): any {
    return this._odata;
  }
}
```

En aquest cas, el *service* `FilesystemService` tindrà el mètode `readFromFile()` que delegarà l'escriptura al mètode `readFile()` de la classe `Filesystem` que ofereix el *pluguin Filesystem*. Aquest mètode retorna una `Promise<ReadFileResult>` que cal tractar per tal d'esbrinar si la lectura s'ha pogut dur a terme o no.

En aquesta implementació cal tenir en compte diverses coses:
1. La ruta del fitxer (`this._path`) pet venir determinada per l'usuari, per tant, en comptes de ser un atribut s'haurà de passar per paràmetre al mètode `writeToFile()`.
2. Les dades que que es llegeixen del fitxer poden estar en format `string` o en format `Blob` (només el en cas de plataforma web), per tant, un cop llegides caldrà aplicar-los-hi un `JSON.parse()`.
3. Els directoris on es poden emmagatzemar les dades són els llistats en l'[apartat anterior](#directoris-que-ofereix-el-plugin-filesystem)
4. La [codificació](https://capacitorjs.com/docs/apis/filesystem#encoding) del fitxer pot estar en `ASCII`, `UTF-8` o `UTF16`.

A més a més de la implementació del mètode `readFromFile()` també és necessari crear un mètode *getter* per tal de poder consultar el valor de l'atribut privat `this._odata`.


### Implementació de la vista
La gestió del sistema de fitxers es pot fer des de qualsevol pàgina de l'aplicació. En aquest exemple, però, es crearà una pàgina, conjuntament amb la ruta corresponent, específica mitjançant la comanda:

```bash
ionic generate page view/list
```

Un cop fet això, només necessitem crear una pàgina que tingui un botó que activi l'escriptura de fitxer i un altre botó per a la lectura.

{% tabs %}
{% tab title="Codi list.page.html" %}
```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>list</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-button (click)="saveFile()">Save File data</ion-button>
  <ion-button (click)="readFile()">Show File data</ion-button>

  <div *ngIf="error_msg != ''">{{ error_msg }}</div>

  <ul *ngIf="data">
    <li *ngFor="let elem of data.greengrocer">
      {{ elem }}
    </li>
  </ul>
  <ul *ngIf="data">
    <li *ngFor="let elem of data.supermarket">
      {{ elem }}
    </li>
  </ul>
</ion-content>
```
{% endtab %}
{% tab title="Codi list.page.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { FilesystemService } from 'src/app/service/filesystem.service';

@Component({
  selector: 'app-list',
  templateUrl: './list.page.html',
  styleUrls: ['./list.page.scss'],
})
export class ListPage implements OnInit {

  public error_msg: string = "";

  constructor(private _filesystemService: FilesystemService) {}

  ngOnInit() {}

  async saveFile() {
    this.error_msg = "";

    if(!await this._filesystemService.writeToFile()) {
      this.error_msg = "No s'ha pogut fer l'escriptura a fitxer";
    }
  }

  async readFile() {
    this.error_msg = "";
    if(!await this._filesystemService.readFromFile()) {
      this.error_msg = "No s'ha pogut llegir el fitxer"
    }
  }

  get data(): any {
    return this._filesystemService.data;
  }
}
```
{% endtab %}
{% endtabs %}

Com es pot comprovar, el codi d'aquesta pàgina injecta el service que s'ha implementat prèviament (`FilesystemService`) i només té 3 mètodes:
1. `saveFile()`: per guardar les dades a un fitxer
2. `readFile()`: per llegir les dades de fitxer
3. `data()`: el *getter* que permet retornar les dades llegides de fitxer


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
Per configurar l'aplicació Android, sempre que es vulguin utilitzar els directoris `Documents` i `ExternalStorage`, cal modificar el fitxer *AndroidManifest.xml*, el qual es troba dins de la carpeta `android/app/src/main`. Cal tenir en compte que qualsevol modificació d'aquest fitxer s'ha de fer amb el servidor *on-the-fly* tancat i que, per provar-ne els resultats, cal, prèviament, desinstal·lar l'aplicació del mòbil, en cas que ja hi estigués instal·lada.

La modificació a fer-hi només implica afegir permisos per a l'accès a l'emmagatzematge extern:

```html
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
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
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
</manifest>
```

En cas que faci falta treballar amb fitxers molt grans, també farà falta afegir l'atribut `android:largeHeap="true"` dins de l'etiqueta `application`:

```html
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:largeHeap="true">

        ...

        <meta-data android:name="com.google.mlkit.vision.DEPENDENCIES" android:value="barcode_ui"/>
    </application>

    <!-- Permissions -->

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
</manifest>
```

### Configuració de la plataforma IOS
Per configurar l'aplicació IOS cal fer diversos passos:
1. Crear el fitxer `PrivacyInfo.xcprivacy` dins de la carpeta `ios/App`
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
    <!-- Add this dict entry to the array if the PrivacyInfo file already exists -->
    <dict>
        <key>NSPrivacyAccessedAPIType</key>
        <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
        <key>NSPrivacyAccessedAPITypeReasons</key>
        <array>
        <string>C617.1</string>
        </array>
    </dict>
    </array>
</dict>
</plist>
```
2. Configurar els permisos dins del fitxer `Info.plist`, que es troba dins de la carpeta `ios/App/App`:
    * Clau `UIFileSharingEnabled` (permís per compartir fitxers a través d'*iTunes*) amb valor `Yes`
    * Clau `LSSupportsOpeningDocumentsInPlace` (permís per gestió de fitxers locals) amb valor `Yes`

```html
  <key>UIFileSharingEnabled</key>
  <true/>
  <key>LSSupportsOpeningDocumentsInPlace</key>
  <true/>
```


## Compliació, creació i execució de l'aplicació final
Si ja s'han fet tots els passos anteriors, l'aplicació ja pot ser compilada, executada i provada. Per fer-ho, es poden seguir els passos del [Capítol 17](./chapter17.md) i, d'aquesta manera poder utilitzar la funcionalitat *Live Reload* sobre el mòbil o, en canvi, crear i instal·lar l'APK directament, tal com s'explica al [Capítol 18](./chapter18.md).
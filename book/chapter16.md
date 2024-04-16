# Capítol 16. *Live Reload* de l'aplicació mòbil
La nova versió d'Ionic ofereix la possibilitat d'executar un servidor *on-the-fly* directament a un dispositiu mòbil, sigui físic o un emulador. D'aquesta manera, qualsevol canvi que es faci al codi es podrà provar, directament, sobre el hardware on ha de funcionar sense la necessitat de crear un executable (APK) i instal·lar-lo al dispositiu.

Per poder preparar el sistema per treballar amb aquesta mena de servidor *on-the-fly* cal, però, seguir uns quants passos:
1. Instal·lar [Capacitor](https://capacitorjs.com/) dins del projecte Ionic (en les noves versions és possible que ja estigui instal·lat per defecte)
2. Afegir la plataforma Android (o IOS, segons es desitgi)
3. Instal·lar i configurar [Android Studio](https://developer.android.com/studio) (o [Xcode](https://apps.apple.com/es/app/xcode/id497799835?mt=12) en el cas d'IOS)
4. Configurar totes les variables d'entorn relacionades amb Android Studio
5. Instal·lar la [JDK 17 de JAVA](https://www.oracle.com/es/java/technologies/downloads/) i configurar la variable d'entorn `JAVA_HOME`
6. Actiar les opcions de desenvolupador al mòbil de prova (en cas que no es tiguin mòbil, crear un simulador a Android Studio)
7. Connectar el mòbil a l'ordinador amb el projecte a través del cable USB i, a més a més, connectar tots dos dispositius a la mateixa xarxa

Si tots aquests passos, que només s'han de fer 1 cop al principi de tot, s'han dut a terme exitosament, ja només caldrà activar el servidor *on-the-fly*.

## Instal·lar Capacitor
Capacitor és una eina de l'ecosistema Ionic que permet compilar codi Ionic, amb la base que sigui (Angular, JS, React, Vue, etc), i crear l'executable segons la plataforma per la qual s'estigui implementant (Android o IOS). A part d'això, ofereix un conjunt de [*pluguins*](https://capacitorjs.com/docs/plugins) que permeten l'accés directe al *hardware* del dispositiu i, per tant, permete utilitzar la càmera de fotos, el GPS, el sistema de fitxers, etc.


Per saber si un projecte Ionic té instal·lades les dependències de Capacitor només cal comprovar si estan llistades dins del fitxer `package.json`:

```json
{
  "name": "FirstIonicProject",
  "version": "0.0.1",
  "author": "Ionic Framework",
  "homepage": "https://ionicframework.com/",
  "scripts": {
    ...
  },
  "private": true,
  "dependencies": {
    ...
    "@capacitor/android": "5.6.0",
    "@capacitor/app": "5.0.7",
    "@capacitor/core": "5.6.0",
    "@capacitor/haptics": "5.0.7",
    "@capacitor/keyboard": "5.0.8",
    "@capacitor/status-bar": "5.0.7",
    "@ionic/angular": "^7.0.0",
    "ionicons": "^7.0.0",
    ...
  },
  "devDependencies": {
    ...
    "@capacitor/cli": "^5.6.0",
    "@ionic/angular-toolkit": "^9.0.0",
    ...
  },
  "description": "An Ionic project"
}
```

En cas que no hi siguin, les comandes per poder-les instal·lar localment dins de la carpeta del projecte són les següents:

```bash
    npm i @capacitor/core
    npm i -D @capacitor/cli
```

## Afegir la plataforma Android (o IOS, segons es desitgi)
Un cop instal·lat Capacitor, cal instal·lar les eines necessàries segons la plataforma per a la qual s'estiguin desenvolupant i crear el projecte. Així doncs, la comanda per instal·lar les eines d'Android i poder-ne crear el projecte i sincronitzar-ne el codi són les següents:

```bash
    npm i @capacitor/android        # Només en cas que la dependència no estigui al package.json
    ionic capacitor add android
    ionic capacitor sync
```

En el cas d'IOS, les comandes són les següents:

```bash
    npm i @capacitor/ios            # Només en cas que la dependència no estigui al package.json
    ionic capacitor add ios
    ionic capacitor sync
```

## Instal·lar i configurar Android Studio
Per instal·lar Android Studio només cal descarregar l'instal·lador de la seva [pàgina web](https://developer.android.com/studio). Si el sistema operatiu que s'utilitza és Windows, cal executar l'instal·lador i seguir els passos que s'hi indiquen. En canvi, si el sistema operatiu que s'utilitza és Linux, prèviament a poder executar l'instal·lador cal seguir els passos següents:
1. Decarregar la versió del codi comprimida en TAR.GS
2. Descomprimir la carpeta i moure-la a /opt (com a `root`):
```bash
    mv android-studio-2023.2.1.24-linux /opt/
```
3. Modificació del `PATH` (s'ha de fer com a `root`)
```bash
    nano /etc/bash.bashrc

    # Al final del fitxer /etc/bash.bashrc afegir-hi les línies següents:
    export ANDROID_STUDIO_HOME=/opt/android-studio-2023.2.1.24-linux/android-studio/
    export PATH=$PATH:/usr/sbin:$NODEJS_HOME/bin:$ANDROID_STUDIO_HOME/bin
```
4. Reiniciar el terminal per tal que els canvis de configuració siguin efectius
5. Executar l'instal·lador d'Android Studio mitjançant la comanda (com a usuari normal):
```bash
    studio.sh
```

La instal·lació, tant en Windows com en Linux, és directa (sense canvi d'opcions) i, un cop finalitzada, dins del vostre directori d'usuari, hi trobareu la carpeta `Android/Sdk` amb les eines de desenvolupament d'Android que permeten crear i compilar l'APK. En Linux la trobareu a `/home/user_name`

**Usuaris de Windows**\
Cal que comproveu que la variable d'entorn `ANDROID_STUDIO_HOME` s'ha afegit i configurat automàticament dins del vostre sistema. En cas que no sigui així, afegiu-la i modifiqueu el `PATH` per tal que la inclogui.

## Configurar les variables d'entorn relacionades amb Android Studio
Un cop finalitzat el pas anterior i localitzada la carpeta `Android/Sdk` cal configurar la variable d'entorn `ANDROID_SDK_ROOT`.

Els usuaris de Linux han de seguir aquestes instruccions (com a `root`):
1. Modificació del fitxer `/etc/bash.bashrc`
```bash
    nano /etc/bash.bashrc

    # A sobre de les línies afegides en el pas anterior, afegir-hi la següent comanda:
    export ANDROID_SDK_ROOT=/home/milana/Android/Sdk
```
2. Reiniciar el terminal per tal que els canvis de configuració siguin efectius
3. Comprovar que la variable ha estat ben configurada:
```bash
    echo $ANDROID_SDK_ROOT
```

## Instal·lar la JDK 17 de JAVA i configurar la variable d'entorn `JAVA_HOME`
Trobareu l'instal·lador de la JDK en aquest [enllaç](https://www.oracle.com/es/java/technologies/downloads/). En el cas de Windows, és una instal·lació directa, tot assegurant que la variable d'entorn `JAVA_HOME` s'actualitza per apuntar a la JDK 17 (en cas que no sigui així, s'ha d'actualitzar manualment).

En el cas de Linux, cal descarregar l'arxiu comprimit TAR.GZ i seguir els passos següents:
1. Descomprimir la carpeta i moure-la a /opt (com a `root`):
```bash
    mv jdk-17_linux-x64_bin /opt/
```
2. Modificació del `PATH` (s'ha de fer com a `root`)
```bash
    nano /etc/bash.bashrc

    # A sobre de les línies afegides en els passos anteriors, posar-hi la comanda següent:
    export JAVA_HOME=/opt/jdk-17_linux-x64_bin/jdk-17.0.10
```
4. Reiniciar el terminal per tal que els canvis de configuració siguin efectius
5. Comprovar que la comanda `java` apunta a la JDK 17
```bash
    echo $JAVA_HOME
    java --version
```

## Actiar les opcions de desenvolupador al mòbil de prova
Aquest pas depèn de cadascun dels vostres mòbils, però *grosso modo* cal anar a la configuració del dispositiu i buscar-ne el seu *número de compliació* (acostuma a estar dins de l'apartat *Informació del telèfon*). Un cop trobada aquesta informació cal fer diversos tocs a sobre fins que surt l'avís indicant que les eines de desenvolupador s'han activat.

Un cop fet això, s'han de configurar les eines de desenvolupador (normalment es troben dins de l'apartat *Sistema*) per tal que permetin la depuració per USB i la depuració sense fil (wifi).

## Connectar el mòbil a l'ordinador amb el projecte a través del cable USB i, a més a més, connectar tots dos dispositius a la mateixa xarxa
Finalment, connecteu el mòbil a l'ordinador a través del cable USB i comproveu que els dos dispositius ja es poden comunicar mitjançant la comanda

```bash
    ionic capacitor run android --list
```

Aquesta comanda hauria de mostrar per pantalla dos dispositius: un de físic amb el nom del vostre mòbil Android i un simulador *Pixel 3* que genera, de manera automàtica, Android Studio.

Si la comanda anterior no llista cap dispositiu, cal comprovar si la següent comanda sí funciona correctament (com a root):

```bash
    $ANDROID_SDK_ROOT/platform-tools/adb devices
```

En cas que aquesta 2a comanda sí llisti el vostre dispositiu físic però digui que el seu estat és *unauthorized*, obriu el mòbil i doneu permisos a la depuració USB (us haurà sortit una petita notificació per fer aquest pas). Fet això, ja us haurien de funcionar totes dues comandes correctament.

També us heu d'assegurar que tant l'ordinador com el mòbil estan dins de la mateixa xarxa d'internet. Una possible solució és que el mòbil doni xarxa wifi a l'ordinador.

## Activació del servidor *on-the-fly*
Per poder comprovar el funcionament de la vostra aplicació directament sobre el mòbil només cal executar la comanda següent:

```bash
    ionic capacitor run android -l --external
```

Si tot ha anat bé, aquesta instrucció activa el procés de compilació del codi i demana el dispositiu i la interfície de xarxa sobre el qual es vol activar el servidor. A partir d'aquí, qualsevol canvi que es faci al codi mentre el servidor estigui activat, serà directament llençat contra el dispositiu per poder ser provat i *debbugat* sense necessitat de crear l'instal·lador APK cada vegada.

Addicionalment, si el mòbil està connectat per USB a l'ordinador, es pot fer la *debbugació* de l'aplicació a través dels navegadors *Firefox* i *Chrome* accedint a les adreces
* `about:debugging` a *Firefox*
* `chrome://inspect/#devices` a *Chrome*

<!-- https://stackoverflow.com/questions/68346778/ionic-capacitor-how-to-see-console-when-running-on-android-emulator -->
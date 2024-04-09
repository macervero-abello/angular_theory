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
# Instal·lació i preparació de l'entorn de desenvolupament

Per poder traballar amb el framework [Angular](https://angular.dev/) cal instal·lar l'entorn de desenvolupament i tenir un bon editor que faciliti la tasca de desenvolupament.

En aquest curs es proposa utilitzar l'editor de codi [Visual Studio Code](https://code.visualstudio.com/), l'ús del qual està molt estès dins del món de desenvolupament web i que, a més a més, disposa d'un molt bon catàleg de *plugins* per adaptar l'experiència de desenvolupament de software al gust del programador.

## Instal·lació de l'entorn de desenvolupament
Per poder instal·lar el *framwork* Angular i poder començar a treballar-hi cal, primer, instal·lar [NodeJS](https://nodejs.org/), el qual ens permetrà gestionar tots els pàquets (mòduls) que s'hauran d'anar instal·lant mitjançant la seva eina `npm` (*node package manager*).

Un cop fet això, ja podem instal·lar Angular la nostre sistema.

### Instal·lació en Windows
En cas que es tingui el sistema operatiu Windows, la instal·lació de NodeJS segueix el procediment clàssic de qualsevol instal·lador.

Per comprovar que NodeJS ha estat instal·lat correctament es pot executar la comanda
```bash
$ npm -v
```
en qualsevol terminal acabat d'obrir. Si tot ha anat bé, aquesta comanda mostrarà per pantalla la versió del *node package manager* instal·lada, de tal manera que, si s'ha instal·lat NodeJS v22.15.0, la versió del *node package manager* serà la 10.9.2, com a mínim. En cas que hi hagi algun problema, s'haurà d'actualitzar la variable d'entorn `PATH` per tal d'afegir-hi la ruta al directori on s'hagi instal·lat NodeJS.

Quan NodeJS estigui ben instal·lat al sistema es podrà instal·lar el *framework* Angular de forma global (opció `-g`) mitjançant la comanda següent (des d'un CMD):
```bash
$ npm install -g @angular/cli
```
Aquest paquet instal·la les eines de *command line interface*, les quals ens permetran gestionar tots els projectes Angular de manera senzilla des de línia de comanda.

Per comprovar que Angular ha estat ben instal·lat es pot executar la comanda
```bash
$ ng --version
```
la qual mostra la versió d'Angular del sistema (actualment, la 19.2.11).

### Instal·lació en Linux
En el cas de Linux, per instal·lar NodeJS cal 
1. descarregar-se el fitxer comprimit `tar.xz`;
2. descomprimir-lo;
3. ubicar-lo segons les preferències del desenvolupador (per exemple, al directori `/opt`) i, finalment,
4. modificar la variable d'entorn `$PATH` per tal que inclogui els executables.

Per fer l'últim pas (a nivell de sistema), s'han d'afegir les següents instruccions al final del fitxer `/etc/bash.bashrc`:
```bash
NODEJS_HOME=/opt/node-v22.15.0-linux-x64
export PATH=$PATH:$NODEJS_HOME/bin
```
Un cop fet això i per comprovar que l'eina `npm` ja és accessible, es pot executar la comanda
```bash
$ npm -v
```
en qualsevol terminal acabat d'obrir. Si tot ha anat bé, aquesta comanda mostrarà per pantalla la versió del *node package manager* instal·lada, de tal manera que, si s'ha instal·lat NodeJS v22.15.0, la versió del *node package manager* serà la 10.9.2, com a mínim.

Quan NodeJS estigui ben instal·lat al sistema es podrà instal·lar el *framework* Angular de forma global (opció `-g`) mitjançant la comanda següent:
```bash
$ npm install -g @angular/cli
```
Aquest paquet instal·la les eines de *command line interface*, les quals ens permetran gestionar tots els projectes Angular de manera senzilla des de línia de comanda.

Per comprovar que Angular ha estat ben instal·lat es pot executar la comanda
```bash
$ ng --version
```
la qual mostra la versió d'Angular del sistema (actualment en el moment de preparar aquesta documentació, la 19.2.11).

### Actualització d'una instal·lació antiga
En cas que algú ja tingui una versió d'Angular antiga al seu sistema i la vulgui actualitzar a una versió més nova o, fins i tot, a l'última, pot executar una de les següents comandes:
```bash
$ npm install -g @angular/cli@XX        #Instal·larà la versió XX
$ npm install -g @angular/cli@latest    #Instal·larà la versió més nova
```

## Creació del primer projecte Angular

1. Estructura de directoris i fitxers
2. Compilació/activació del propi server

## Webgrafia del capítol
* Google (2025). [Angular](https://angular.dev/). Consultat el 12 de maig de 2025.
* Angular (2025). [GitHub Angular](https://github.com/angular/angular). Consultat de 12 de maig de 2025.
* Microsoft (2025). [Visual Studio Code](https://code.visualstudio.com/). Consultat el 3 de juny de 2025
* OpenJS Foundation (2025). [NodeJS](https://nodejs.org/). Consultat el 12 de maig de 2025.
* Udemy (2025). [Curs *Angular - The Complete Guide (2025 Edition)*](https://www.udemy.com/course/the-complete-guide-to-angular-2/). Consultat el 12 de maig de 2025.
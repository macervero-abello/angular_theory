# Capítol 14. Creació d'estils amb Ionic
Per crear els estils d'una aplicació Ionic cal tenir en compte 4 factors:
1. La creació d'un *layout*
2. La distribució del contingut dins de les pàgines
3. Els estils que ofereix el propi *framework* Ionic
4. La creació d'un tema propi (la guia d'estil o marca)

## Creació d'un *layout*
Ionic ofereix 4 tipus de *layouts* diferents, els quals podeu trobar explicats en aquest [enllaç](https://ionicframework.com/docs/layout/structure)
* *Single page*: formada per un  *header*, el contingut i un *footer*, tot i que el *header* i el *footer* són opcionals (aquest *layout* bàsic és el que s'ha presentat en el [Captíol 12](chapter12.md))
* *Tabs*: la navegació està gestionada per un conjunt de pestanyes (analitzat en el [capítol anterior](chapter13.md))
* Menú: la navegació està gestionada per un menú (analitzat en el [capítol anterior](chapter13.md))
* *Split pane*: s'utilitza per crear pàgines web que tenen el menú fixat en una columna a l'esquerra (no estudiarem en profunditat aquest *layout* perquè no s'utilitza en aplicacions mòbils)

## Distribució del contingut
Un cop escollit el layout, cal definir una bona distribució dels elements en pantalla per tal que l'aplicació sigui *responsive* (penseu que l'*APK* - executable de l'aplicació en la seva versió Android - s'ha de veure igual de bé en tablet i en mòbil). Per fer-ho, Ionic ofereix un sistema de [grid](https://ionicframework.com/docs/api/grid) molt similar al de Bootstrap i altres frameworks de CSS.

Aquesta grid està basada en *flexbox* i, per crear-la, s'utilitzen tres etiquetes diferents:

```html
<ion-grid>
	<ion-row>
		<ion-col> Contingut </ion-col>
		<ion-col> Contingut </ion-col>
		<ion-col> Contingut </ion-col>
	</ion-row>
</ion-grid>
```

Tal com passa amb Boostrap, quan s'utilitza el sistema de grid, la pantalla queda dividida en 12 columnes. En cas que no especifiquem altra cosa, les columnes que definim dins d'una fila s'expandiran tant com puguin fins a omplir completament la pantalla (divint l'espai equitativament). Ara però, nosaltres podem modificar la seva mida a voluntat.

### Especificació de les diferents mides
**Grid**

Per defecte, la grid intentarà ocupar tota l'amplada disponible. Ara però, el desenvolupador pot decidir donar-li una mida fixa (segons el tipus de dispositiu on es trobi) utilitzant l'atribut "fixed": `<ion-grid [fixed]="true">`

Vegeu les mides possibles al següent [enllaç](https://ionicframework.com/docs/api/grid#fixed-grid).

A través de variables CSS proporcionades per Ionic es pot personalitzar totes les característiques de la grid: nombre de columnes, padding, width i padding de columna (vegeu l'enllaços [1](https://ionicframework.com/docs/layout/grid#customizing-the-grid) i [2](https://ionicframework.com/docs/theming/advanced#grid-variables))

**Columnes**

Si no s'indica el contrari, les columnes s'estiren per intentar utilitzar tota l'amplada disponible. Ara però, podem fixar la mida d'una columna a voluntat a traves de l'atribut `size-{breakpoint}`. Exemples:
* `<ion-col size="4">`: aquesta columna sempre ocuparà 4 columnes de les 12 que té la grid (sigui quina sigui la mida del dispositiu)
* `<ion-col size-xs="4">`: ocuparà 4 columnes en pantalles de mida molt petita en amunt
* `<ion-col size-sm="4">`: ocuparà 4 columnes en pantalles de mida petita en amunt (en pantalles molt petites, la columna s'expandirà tant com pugui)
* `<ion-col size-md="4">`: ocuparà 4 columnes en pantalles mitjanes en amunt (en pantalles petites i molt petites, s'expandirà tant com pugui).
* `<ion-col size-lg="4">`: ocuparà 4 columnes en pantalles grans en amunt (per la resta, la columna s'expandirà tant com pugui)
* `<ion-col size-xl="4">`: només ocuparà 4 columnes en pantalles molt grans (per la resta, s'expandirà tant com pugui)

Per defecte, les columnes tenen una mica de padding que les separa dels marges de la grid. En cas que es vulgui eliminar, s'ha d'aplicar la classe ion-no-padding a la grid:
```html
<ion-grid [ngClass]="['ion-no-padding']">
```
Si només es vol eliminar d'una columna en concret, s'aplicarà l'estil sobre l'etiqueta `<ion-col>` corresponent.

A part de la mida, a les columnes també se'ls pot especificar un *offset*, un *pull* i un *push*
* *Offset*: desplaça la columna cap a la dreta 
    * `<ion-col offset="4">`: la desplaça 4 columnes, independentment de la mida de pantalla
    * `<ion-col offset-xs="4">`: la desplaça 4 columnes en pantalles molt petites en amunt
    * `<ion-col offset-sm="4">`: la desplaça 4 columnes en pantalles petites en amunt
    * `<ion-col offset-md="4">`: la desplaça 4 columnes en pantalles mitjanes en amunt
    * `<ion-col offset-lg="4">`: la desplaça 4 columnes en pantalles grans en amunt
    * `<ion-col offset-xl="4">`: la desplaça 4 columnes en pantalles molt grans
* *Pull*: empeny la columna X columnes cap a la dreta (pot anar sol o acompanyat de les mides adients)
* *Push*: estira la columna X columnes cap a l'esquerra (pot anar sol o acompanyat de les mides adients)

Per alinear les columnes dins de la cel·la es pot utilitzar diferents classes proporcionades per Ionic.
* Alineació vertical afectant a tota la fila
```html
<ion-row class="ion-align-items-start">
<ion-row class="ion-align-items-center">
<ion-row class="ion-align-items-end">
```
* Alineació vertical afectant a una columna en concret
```html
<ion-col class="ion-align-self-start">
<ion-col class="ion-align-self-center">
<ion-col class="ion-align-self-end">
```
* Alineació horitzontal
```html
<ion-row class="ion-justify-content-start">
<ion-row class="ion-justify-content-center">
<ion-row class="ion-justify-content-end">
<ion-row class="ion-justify-content-around">
<ion-row class="ion-justify-content-between">
```

En aquest [enllaç](https://ionicframework.com/docs/layout/css-utilities) trobareu altres estils que es poden aplicar a la grid i a les columnes.


## Estils
Ionic ofereix un conjunt de [variables css](https://ionicframework.com/docs/layout/css-utilities) i classes per tal de poder modificar els aspectes següents:

**Text**
 * [Alineació del text](https://ionicframework.com/docs/layout/css-utilities#text-alignment)
 * [Transformació del text (majúscules, minúscules, camel case)](https://ionicframework.com/docs/layout/css-utilities#text-transformation)
	
Totes aquestes classes tenen la versió general i la versió *breakpoint* (xs, sm, md, lg i xl)

**Distribució dels elements**
 * [Aplicació d'elements flotants](https://ionicframework.com/docs/layout/css-utilities#float-elements), en la seva versió general i també *breakpoint*

 **Ocultació d'elements**
  * [Ocultació d'elements](https://ionicframework.com/docs/layout/css-utilities#element-display), en la seva versió general i també *breakpoint*

**Espaïat** 
 * [Padding](https://ionicframework.com/docs/layout/css-utilities#element-padding)
 * [Margin](https://ionicframework.com/docs/layout/css-utilities#element-margin)
 * [Propietats flex (alineació elements)](https://ionicframework.com/docs/layout/css-utilities#element-margin)
	
***BORDER***
 * [*Border*](https://ionicframework.com/docs/layout/css-utilities#border-display)

Addicionalment a tots els estils que ofereix pròpiament Ionic, cada desenvolupador pot definir estils propis, sigui al fitxer `SCSS` de cada pàgina, sigui de manera general al fitxer `global.scss`.

## Creació d'un tema (guia d'estil o marca)
Ionic facilita la personalització completa de l'estil de l'aplicació mitjançant la [creació d'un tema](https://ionicframework.com/docs/theming/basics) al fitxer `theme/variables.scss`.

**Colors**

L'estil per defecte d'ionic defineix una [gamma bàsica de 9 colors](https://ionicframework.com/docs/theming/basics#colors), descrits en les variables `CSS` corresponents dins del fitxer `theme/variables.scss`. Tots ells, però, són completament personalitzables i ampliables segons les necessitats de cadascuna de les aplicacions desenvolupades, tal com s'explica en pel següent [enllaç](https://ionicframework.com/docs/theming/colors)

**Plataformes**

A part dels colors, Ionic defineix dos estils diferents, depenent de la plataforma sobre la qual s'hagi d'executar l'aplicació
 * La classe `CSS` `ios` defineix els estils que cal aplicar a la plataforma d'Apple
 * La classe `CSS` `md` defineix els estils que cal aplicar a la plataforma Android (o qualsevol altra)

Per especificar l'ús d'una plataforma o altra, la classe s'especifica a l'etiqueta `html`:
```html
<html class="md"></html>
```


===============================> M'HE QUEDAT AQUÍ!!!

Per modificar un estil en una de les dues plataformes caldrà crear un fitxer `SCSS` dins d'`assets/css`. Allí hi podrem posar tot allò que nosaltres creguem necessari:
	.ios ion-title {
		//code
	}
	.md ion-title {
		//code
	}
	
	//Sobreescriptura de variables predefinides (això també es pot fer a theme/variables.scss)
	.ios {
  		--ion-background-color: #222;
	}

Variables que ofereix Ionic
	COMPONENT VARIABLES
	Per cada element (tag), Ionic ofereix un conjunt de variables (propietats CSS) que es poden personalitzar a gust. Per exemple, en el cas de l'ion-button podem personalitzar totes les variables que es mostren en aquest enllaç: https://ionicframework.com/docs/api/button#css-custom-properties.

	GLOBAL VARIABLES
	A part de les variables de component, hi ha tot un conjunt de variables relacionades amb la creació de la "marca" (tema, guia d'estil) de l'aplicació. Aquestes variables es troben dins del fitxer theme/variables.scss, el qual queda organitzat en diversos blocs. Els més importants:
	* Bloc "root": dins d'aquest selector s'especifiquen les variables que s'aplicaran a tots els modes (ios i md).
	* Bloc ".ios": dins d'aquesta classe s'especifiquen les variables per al mode ios
	* Bloc ".md": dins d'aquesta classe s'especifiquen les variables per al mode md.
	

Per utilitzar aquestes variables en les nostres CSS:
	h1 {
		color: var(--ion-color-primary);
	}

Per canviar alguna variable de les predefinides en els elements d'Ionic:
	ion-button {
		--background: #FF0000;
		--background: var(--ion-color-primary);
	}
	
	-- COLORS --
	https://ionicframework.com/docs/theming/colors
	Ionic presenta una gamma bàsica de 9 colors, amb diferents capes, definits a variables.scss
	Aquest esquema de colors es pot ampliar i canviar a voluntat
	
	Per crear un color nou: https://ionicframework.com/docs/theming/colors#new-color-creator
	Per crear tot un tema nou: https://ionicframework.com/docs/theming/color-generator

Per canviar al mode "dark" a voluntat:
	1. Amb la media quary que hi ha a variables.scss s'aconsegueix que, si l'usuari té el sistema en dark, Ionic apliqui les CSS dark
	2. Per fer-ho manualment, cal duplicar les variables "dark" fora del CSS, tot afegint-hi un atribut "[color-theme="dark"]" (també es pot fer amb una classe)
	Fet això, des del menú, per exemple, caldrà fer un "toggle" que indiqui quin tipus de mode es desitja
	"document.body.setAttribute('color-theme', 'dark');
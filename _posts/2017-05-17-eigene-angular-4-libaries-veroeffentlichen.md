---
title: "Erstellen und Publizieren einer Library ."
author: "Johannes Millan"
slug: "angular-4-libs-veroeffentlichen"
published_at: 2017-04-03 08:00:01.000000Z
categories: "angular angular4"
---

## Open Source: Eine wertvolle Erfahrung für jeden Entwickler
An der Verwendung von Third-party-modulen über npm und yarn führt für den modernen Webentwickler eigentlich kein Weg vorbei. Dennoch ist es unter den allermeisten meiner häufig wechselnden Kollegen eher die Ausnahme, dass jemand auch eigene Softwareprojekte auf GitHub veröffentlicht.
 
 Teil des Problems ist sicher, dass es nur in wenigen Agenturen und Unternehmen eine Kultur gibt, die Open Source fördert. Schade eigentlich, denn von der Erfahrung kann jeder Entwickler eigentlich nur profitieren. Das Veröffentlichen des eigenen Codes für ein (potentiell) großes Publikum, kann sich nicht nur sehr gut anfühlen, sondern es erweitert auch die Perspektive darauf, mit wem man alles seine Profession teilt und am wichtigsten vielleicht: Auf den eigenen Code selbst, denn dieser sollte im Idealfall auch immer für Andere verständlich bleiben und möglichst auch einen einfachen Einstieg ermöglichen. Dies ist nicht nur nützlich, wenn man im Team arbeitet, sondern auch dein zukünftiges Ich wird es dir danken, wenn du einmal einen Fehler in einer Jahre alten Applikation fixen musst.  
 
 Es kann sicher auch etwas beängstigend sein, seinen Code mit einem breiten Publikum zu teilen, denn Hand aufs Herz: Wir alle pfuschen ein bisschen hin und wieder und bei gar nicht mal so wenigen Zeilen Code, die wir schreiben, ist uns zunächst nicht wirklich klar, warum sie funktioniert oder warum nicht. Aber keine Angst: Das gilt gleichermaßen für wohl so ziemlich jeden auch noch so gestandenen Programmierer und nicht wenige der 1000+ Sternchen Module enthalten zum Teil wirklich schaurigen Code und seltsam anmutende Designentscheidungen!

## Das Erstellen und Veröffentlichen deiner eigenen Angular 4 Libary ist (leider nicht) ganz einfach
Mittlerweile haben wohl die meisten von uns ihre ersten Erfahrungen mit Angular2+ gesammelt. Die vielen Neuerungen machen den Einstieg sicher nicht unbedingt einfach. Dies gilt mehr noch für das komplexe Ökosystem um das Framework herum. [Angular-cli](https://github.com/angular/angular-cli) erleichtert den Prozess des Erstellens einer eigenen App ganz erheblich, hilft einem für die Erstellung eigener Libraries für sich genommen, aber leider nicht wirklich weiter.

Deswegen möchte ich im Folgenden beschreiben, welche Schritte erforderlich sind, um eure Library via npm oder yarn installierbar zu machen. 

Wenn euch das alles zu lange dauert, schaut euch gerne mal meinen Yeoman Generator an: [Ihr findet das fertige Resultat hier](https://github.com/johannesjo/generator-angular2-lib).

## Wo wollen wir hin
Nehmen wir an, wir haben folgende Komponente geschrieben:

```
.
├── package.json
└── src
    ├── index.ts
    ├── package.json
    ├── banana.module.ts
    └── banana.directive.ts
```
index.ts:
```typescript
// exportiert alle Komponenten, die ihr importierbar machen wollt:
export {BananaModule} from './banana.module';
export {BananaDirective} from './banana.directive'
```
banana.module.ts:
```typescript
import {NgModule} from '@angular/core';
import {BananaDirective} from './banana.directive';

@NgModule({
  declarations: [
    BananaDirective,
  ],
  imports: [],
  exports: [
    BananaDirective,
  ],
  providers: []
})
export class BananaModule {
}
```
banana.directive.ts:
```typescript
import {ElementRef} from '@angular/core';

@Directive({
  selector: '[banana]'
})

export class BananaDirective {
  constructor(el: ElementRef) {
        el.nativeElement.insertAdjacentHTML('beforeend', 'BANANA');
  }
}
```




Dann müsste der Output, um korrekt via yarn and npm von einer anderen Anuglar 4 Applicaktion verarbeitet zu werden, in etwa so aussehen:
## 
```
package.json
dist
├── index.js                   
├── index.d.ts                 
├── (index.d.js.map)                 
├── index.metadata.json                 
├── banana.module.js     
├── banana.module.d.ts     
├── (banana.module.d.js.map)     
├── banana.module.metadata.json     
├── banana.directive.js     
├── banana.directive.d.ts     
├── (banana.directive.d.js.map)
└── banana.directive.metadata.json
```

Die `.js` Dateien sind die (zu commonjs) kompilierten JavaScript Modul-Dateien. 

Die `.d.ts` Dateien sind TypeScript Definitions Dateien und werden unter anderem dazu genutzt um Code Completion in der IDE zu unterstützen.
  
Die `.js.map` Dateien sind sourcemap Dateien, die eigentlich nur benötigt werden, wenn man im Stacktrace wissen möchte in welcher Zeile in den Ausgangsdateien ein Fehler auftrat.

Die `.metadata.json` Dateien werden benötigt, wenn Ahead-of-time-Compiling unterstützt werden soll (und es wäre ja schade, wenn Applikationen mit AoT eure Library nicht nutzen könnten).

## Warum wir den Output nicht bundlen?
Die meisten Applikationen, die eure Library nutzen werden, werden in der Regel über ein eigens Bundling System in Form von Gulp, SystemJs oder Webpack verfügen. Aus diesem Grund ist es besser, wenn der Nutzer der Lib selbst entscheiden kann, wie und welche eurer Dateien er laden möchte.


## Vom Sourcecode zum Modul
Um das gewünschte Resultat zu erreichen brauchen wir eigentlich nur zwei Dinge: Den ngc compiler und eine tsconfig.json.

Den compiler fügt ihr am besten zu euren devDependencies hinzu:
```
npm install -D @angular/compiler @angular/compiler-cli
```
Anschließend könnt ihr euch in eurer package.json ein script zur Ausführung des Kompilers
```
  "scripts": {
    "build": "ngc -p ./tsconfig.json",
  },
```
Das erlaubt euch mit `npm run build` den build eurer Library zu startn.


Typescript file
```
{
  "compilerOptions": {
    // wir wollen simples pre Ecma6 JavaScript
    "target": "es5",
    // führt dazu, dass die Dateien als sperate commonjs module kompiliert werden
    "module": "commonjs",
    // sorgt dafür, dass import Pfade im Browser genauso funktionieren, wie sie dies innerhalb eines Node Kontextes tun würden
    "moduleResolution": "node",
    // erstellung der .js.map Dateien
    "sourceMap": true,
    // erlaubt die Nutzung von annotations, wie etwa @Directive
    "emitDecoratorMetadata": true,
    // ob die typing Dateien (*.d.ts) erstellet werden sollen
    "declaration": true,
    // pfad zum output der kompilierung
    "outDir": "./dist",
    // libraries die während der kompilierung verfügbar sein sollen
    "lib": [
      "dom",
      "es6"
    ]
  },
  // alle dateien, die berücksichtigt werden sollen
  "include": [
    "src/**/*"
  ],
  // ausnahmen
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ],
  // entry files
  "files": [
    "./src/index.ts"
  ]
}
```

## html & css
Am einfachsten Inline.

## Ein Modul bei npm veröffentlichen
Falls noch nicht geschehen, könnt ihr über `npm adduser` einen Account bei npm erstellen. Anschließend lässt sich das Modul über `npm publish` veröffentlichen. Hilfreich ist auch der `npm version [major|minor|patch]` Befehl, der automatisch die package.json mit einer neuen Versionsnummer updated.  

## Was sonst noch wichtig ist
Idealer Weise sollte eurer Modul über eine kleine Demo  verfügen, so dass sich andere schnell ein Bild machen können, was eure Library leistet. In jedem Fall vorhanden sein sollte eine Readme in der ihr mindstens erklärt, was das Modul macht und wie man es installiert.

Für die Demo bietet es sich zumeist an [GitHub Pages](https://pages.github.com/) zu verwenden. Eine fertige Vorkonfiguration findet ihr ebenfalls [in dem bereits erwähnten generator](https://github.com/johannesjo/generator-angular2-lib). 

## Dein Modul bekannt machen
Neben vielen anderen Möglichkeiten bieten sich hier das [Angular2+ reddit](https://www.reddit.com/r/Angular2/), sowie auch der [angularjs.de slack chat](https://angularjs-de.slack.com) an.

## Zusammenfassung



## Nützliche Ressourcen

* [How to create a ready to publish to npm angular 2 library (stackoverflow)](http://stackoverflow.com/questions/40261492/how-to-create-a-ready-to-publish-to-npm-angular-2-library)
* [How to create an Angular component library, and how to consume it using SystemJs or Webpack](http://blog.angular-university.io/how-to-create-an-angular-2-library-and-how-to-consume-it-jspm-vs-webpack/)
* [“What the #@$% was I thinking!?” How to write understandable code](https://hackernoon.com/what-the-was-i-thinking-how-to-write-understandable-code-6162b363e735)
* [Publishing an Angular 2 Component NPM Package](http://bobonmedicaldevicesoftware.com/blog/2016/12/09/publishing-an-angular-2-component-npm-package/)

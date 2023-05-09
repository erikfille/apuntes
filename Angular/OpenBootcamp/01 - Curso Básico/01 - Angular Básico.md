# 01 - Introducción a Angular (v.13)

## 1. Introduccion

Angular es un framework de Javascript para Front End que requiere de conocimientos previos, como son HTML, JavaScript y Typescript, ya que hace uso de los distintos tipos de datos de estos ultimos y los renderiza con etiquetas del primero.

Angular no es el framework mas utilizado actualmente en el mercado, pero si es uno de los mas robustos, por eso su utilizacion va creciendo dia a dia.

## 2. Dependencias necesarias para levantar un proyecto

Commando de instalacion:

Angular CLI

```
npm i -g @angular/cli
```

> Es necesario instalar el CLI de Angular de manera global para poder utilizar los comandos `ng`

Para crear un proyecto Angular, tenemos que hacerlo con un comando `ng`.

Entre los comandos `ng`estan:

- add: ofrece añadir dependencias a un proyecto e incluso autoconfigurar algunos ajustes para que la herramienta se ajuste automaticamente al proyecto sin necesidad de estar agregando lineas en cada archivo.

- analytics: permite sacar metricas del proyecto

- build (b): permite compilar para deployar

- config: permite ajustar los valores de configuracion en el angular.json, el archivo básico de configuración para el espacio de trabajo del proyecto.

- doc (d): permite abrir la documentación oficial de Angular.

- e2e (e): compila y sirve una aplicación de Angular, mostrando el trabajo realizado en el navegador y permitiendo correr tests.

- extract-i18n (i18n-extract, xi18n): permite extraer todas las lineas identificadas con la etiqueta i18n, para poder internacionalizar la app, cambiando manualmente el idioma de los mensajes mostrados, y modificando algunas configuraciones del angular.json.

- generate (g): genera y/o modifica archivos basados en un esquema. Es uno de los comandos mas habituales

- help

- lint (l): sirve para ejecutar las herramientas de linting que se hayan configurado.

- new (n): sirve para generar un workspace con una aplicacion inicial de Angular.

- run: ejecuta el target de la aplicacion.

- serve (s): compila y sirve la aplicacion, recompilandose a medida que cambian los archivos.

- test (t): corre tests unitarios en el proyecto.

- update: actualiza la aplicacion y sus dependencias.

- version (v): indica la version utilizada de Angular

## 3. Creando el Proyecto

Al momento de crear un proyecto nuevo con el comando `ng new`, podemos pasar opciones por linea de comando para elegir como levantar nuestro proyecto.

- `--create-application`: si este comando se setea en false, se creará el workspace, pero no se creará ninguna aplicación, dandonos la opcion de crearla luego manualmente.

- `--prefix` (-p): cambia el prefijo por defecto para los selectores generados para el proyecto inicial

- `--routing`: añade o no añade el modulo de ruteo al proyecto inicial

- `--strict`: crea un workspace con un chequeo de tipos mas estricto

- `--style`: la extensión de archivos o preprocesador para utilizar para los estilos

- `--verbose` añade mas salidas por consola

Ejemplo para crear proyecto:

```
ng new HolaMundo
```

Esto nos crea varios archivos, entre ellos:

- editorconfig: configura la indentacion, los espaciados, etc. para Visual Code
- angular.json: configura todo lo que es el proyecto de Angular
- tsconfig.json: indica la configuracion general para toda la aplicacion
- browserlistrc: indica todos los navegadores compatibles con el proyecto
- karma.conf.js: archivo de configuracion para la ejecucion de tests
- tsconfig.aap.json y tsconfig.spec.json: expanden la configuracion del tsconfig para la app y para los tests
- Carpeta .vscode: configura la depuracion de la app
- Carpeta src con el proyecto:
  - main.ts: punto de entrada del proyecto para montarse en el html
  - polyfills.ts: archivo de configuracion para compatibilidad con todos los navegadores
  - styles.scss: hoja de estilos inicial para todo el proyecto
  - test.ts: archivo de configuracion de las pruebas unitarias y de integracion
- Carpeta assets:
  - .gitkeep: incluye imagenes y demas del proyecto
  - carpeta de environments para:
    - environment.prod.ts: archivo de configuracion de entorno de producción
    - environment.ts: archivo de configuracion de entorno de desarrollo
- Carpeta de app:
  - app.component.ts: archivo controlador principal de la app
  - app.component.html: archivo de vista principal de la app
  - app.component.scss: archivo de estilos principal de la app
  - app.component.spec.ts: archivo de testeo principal de la app

La instalación tambien genera un archivo de git

## 4. Compilar y Servir Proyecto

Para mostrar nuestro proyecto, podemos usar el comando

```
ng serve --open
```

`ng serve` compilará el proyecto y quedará atento a cambios para re-renderizarlo en el navegador, mientras que el modificador `--open` se encarga de abrirlo en el navegador por defecto.

Por defecto, el comando `ng serve` despliega el proyecto en el localhost, en el puerto 4200 (http://localhost:4200/)

> La aplicación por defecto que se levanta al crear un proyecto de Angular nuevo contiene mucha información de interes para quien recien inicia con Angular, entre ellas, esta la documentación de Angular, y un link a Angular Material, que consta de varios componentes predeterminados; también cuenta con una seccion de "Siguientes pasos", que nos indican los comandos para crear nuevos componentes, instalar las dependencias de Angular Material, agregar soporte para crear una PWA (Progressive Web Application), agregar una nueva dependencia, correr test y compilar para produccion.

## 5. Archivos Básicos

### a. Angular.json

En el archivo angular.json esta la configuración básica del proyecto, el espacio de trabajo y los comandos ng que podemos utilizar.

Archivo angular.json:

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    // Indica los proyectos actuales del espacio de trabajo
    "HolaMundo": {
      // Proyecto base
      "projectType": "application", // Tipo de proyecto: aplicacion, libreria, etc
      "schematics": {
        // Esquemas aplicados a este proyecto
        "@schematics/angular:component": {
          "style": "scss"
        },
        "@schematics/angular:application": {
          "strict": true // Configura el modo estricto por defecto
        }
      },
      "root": "", // Raiz del proyecto
      "sourceRoot": "src", // Donde se encuentra el codigo source en la raiz del proyecto
      "prefix": "app", // Prefijo que se genera por defecto
      "architect": {
        // Comandos ng que se pueden ejecutar
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/hola-mundo",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.scss"],
            "scripts": []
          },
          "configurations": {
            "production": {
              // configuraciones del comando para el entorno de produccion
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "outputHashing": "all"
            },
            "development": {
              // configuraciones del comando para el entorno de desarrollo
              "buildOptimizer": false,
              "optimization": false,
              "vendorChunk": true,
              "extractLicenses": false,
              "sourceMap": true,
              "namedChunks": true
            }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": {
              "browserTarget": "HolaMundo:build:production"
            },
            "development": {
              "browserTarget": "HolaMundo:build:development"
            }
          },
          "defaultConfiguration": "development"
        },
        "extract-i18n": {
          "builder": "@angular-devkit/build-angular:extract-i18n",
          "options": {
            "browserTarget": "HolaMundo:build"
          }
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "polyfills": ["zone.js", "zone.js/testing"],
            "tsConfig": "tsconfig.spec.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.scss"],
            "scripts": []
          }
        }
      }
    }
  },
  "cli": {
    "analytics": "0a0cf7b8-8f47-433f-8daa-cb037993a675"
  }
}
```

### b.main.ts

El archivo `main.ts` es el punto de entrada del proyecto. Es donde se monta toda la aplicación que luego se incrustará en el archivo html de base.

main.ts:

```typescript
import { platformBrowserDynamic } from "@angular/platform-browser-dynamic";

import { AppModule } from "./app/app.module";

platformBrowserDynamic()
  .bootstrapModule(AppModule) // modulo principal de la aplicacion. Es el modulo que inicia el proyecto. AppModule indica es el componente principal de la aplicacion, que en este caso se encuentra definido en src/app/app.module
  .catch((err) => console.error(err));
```

### c. app.module.ts

Este archivo es un modulo de Angular (NgModule), que básicamente son decoradores. Los decoradores son anotaciones que se pueden añadir a clases, funciones, metodos, parametros de funciones, etc. cuyo fin es añadir informacion y metadatos.

app.module.ts

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";

import { AppComponent } from "./app.component";

@NgModule({
  declarations: [
    // Son los componentes utilizables y visibles en el modo. En este caso se declara la importación de AppComponent para indicarlo luego como el bootstrap del modulo (componente que inicia el modulo en cuestion). Los componentes se deben declarar para poder renderizarlos en el componente actual
    AppComponent,
  ],
  imports: [
    // Los componentes importados tambien se pueden utilizar, pero se necesita que esten exportados en su origen.
    BrowserModule, // Router
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}

/*

Este modulo implica que AppModule incorporará a AppComponent, a BrowserModule y tomará a AppComponent como componente principal (al declararlo como bootstrap: [AppComponent])

Define el origen de la aplicacion y lo que debe cargar y mostrar

*/
```

### d. app.component.ts

El componente de app importa el decorador experimental de Typescript llamado `Component` de @angular/core.

```typescript
import { Component } from "@angular/core";

@Component({
  // Inyecta metadatos
  selector: "app-root", // Nombre de la etiqueta. En este caso, es la que vemos en el index.html como <app-root>. Al utilizarse el selector en index.html, se le indica a este archivo que en ese espacio debe renderizar el contenido de app.component.html
  templateUrl: "./app.component.html", // Indica la ruta donde se encuentra el template html del componente
  styleUrls: ["./app.component.scss"], // Indica la ruta donde se encuentra la hoja de estilos del componente
})
export class AppComponent {
  // Aqui puedo enviar las variables que quiero que se rendericen en el HTML a modo de props
  title = "HolaMundo"; // title es la prop que se pasa al componente HTML para el renderizado
}
```

### e. app.component.html

En `app.component.html`es donde se indicara la estructura HTML y se aplicarán las clases necesarias para el renderizado del componente en cuestión.

Estructura del archivo HTML del componente de la app por defecto:
```html
<style>
 /* Aqui van los estilos del html del componente */
</style>

<!-- Toolbar -->
<div class="toolbar" role="banner"></div>

<div class="content" role="main">
    <!-- Highlight Card -->
    <!-- Aqui se renderiza la prop que se pasa desde el componente -->
    <span>{{ title }} app is running!</span>
</div>

  <!-- Resources -->
  <h2>Resources</h2>
  <p>Here are some links to help you get started:</p>

  <div class="card-container"></div>

  <!-- Next Steps -->
  <h2>Next Steps</h2>
  <p>What do you want to do next with your app?</p>

  <input type="hidden" #selection />

  <div class="card-container"></div>

  <!-- Links -->
  <div class="card-container"></div>

  <!-- Footer -->
  <footer></footer>

</div>
```

## 6. Extensiones de Visual Code utiles para trabajar con Angular

- Angular Language Service (angular.ng-template)
- Amgular Snippets (johnpapa.angular2)
- Auto Complete Tag (formulahendry.auto-complete-tag)
- Better Comments (aaron-bond.better-comments)
- Bracket Pair Colorizer 2 (Nativo en VSCode)
- Color Highlight (naumovs.color-highlight)
- ESLint (dbaeumer.vscode-eslint)
- Image preview (kisstkondoros.vscode-gutter-preview)
- Intellisense for CSS class names in HTML (zignd.html-css-class-completion)
- Jasmine Test Explorer (hbenl.vscode-jasmine-test-adapter)
- Material Icon Theme (pkief.material-icon-theme)
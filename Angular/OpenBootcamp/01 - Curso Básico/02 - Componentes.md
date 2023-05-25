# 02 - Componentes

## 0. Contenido

- Modelo Vista / Controlador
- ¿Que es un componente?
- LifeCycle del Componente
- Anidar Componentes y pasar Props

## 1. Introduccion

Un componente es una porcion de la pantalla que controla un contenido dentro de la pantalla del usuario.

Básicamente es lo que se puede llamar "Vista" y cualquier vista tendrá su controlador.

En el arbol de carpetas de nuestro proyecto, el componente principal es App y generalmente se encuentra en la carpeta raíz del proyecto, aunque puede encontrarse tambien dentro de otra carpeta, para una organizacion mas clara.

Este componente App consta de:

- app.component.ts: que cuenta con los metadatos del componente.
- app.component.html: que renderiza lo que se va a mostrar en pantalla.
- app.component.scss: es la parte del componente que contiene los estilos.

Existen distintos tipos de componentes, según el nivel de complejidad que tienen.

- Componentes de orden superior, que contienen información y lógica.
- Componentes de orden inferior, que se encargan de mostrar la información de los componentes de orden superior.

Esta organización de los componentes en distintos ordenes nos permiten tener una mayor facilidad al momento de depurar la app.

Un componente puede ser:

- Componente de pagina (renderiza toda una pagina)
- Componente contenedor (de otros componentes)
- Componente sencillo (que solo renderiza lo que se le diga)

Los componentes se basan en la filosofia de programación DRY (Don't Repeat Yourself) que nos prevee de tener que estar escribiendo codigo repetido constantemente.

La idea básica de los componente es que van a estar compuestos por controlador (ej: app.component.ts) y vista (ej: app.component.html + app.component.scss), y la estrategia de cambio va a ser a traves de la modificacion de alguna variable que se esta renderizando en la vista, y que será actualizada a traves del motor de renderizado de Angular.

## 2. Metadatos de componentes

Hasta ahora, al levantar una app con el `ng new`, el componente App no trae metadatos por defecto, como son:

```javascript
import { Component } from "@angular/core";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.scss"],
})
export class AppComponent {
  nombre = "Erik";
  title = "HolaMundo";
}
```

Pero estos no son los unicos metadatos que se pueden incorporar en un componente, sino que hay mas, como pueden ser:

- encapsulation
- exportAs (para exportar el componente con algun otro nombre)
- inputs
- outputs
- preservateWhiteSpaces (para preservar espacios en blanco)
- providers (servicios o instancias singleton que queremos inyectar dentro de las clases)
- changeDetection (para controlar cuando el componente se actualiza, lo cual es importante para la performance)
- etc.

Existen muchos otros metadatos, entre los que podemos encontrar algunos no tan recomendables como son Template y Styles, que permiten desarrollar el template html y los estilos en la misma linea, es decir, en el mismo decorador, lo cual se debe evitar a toda costa.

## 3. Crear un nuevo componente

Para crear un nuevo componente, debemos utilizar un comando en el CLI, pasandole el comando `ng`, seguido de la directiva `generate` (o el alias `g`), la opcion `component` (o el alias `c`) y el path donde debe crearse ese componente

```
ng generate component components/Saludo

ng g c components/Saludo
```

Al correr el comando, Angular nos creará una nueva carpeta especifica para nuestro componente, y generará los modulos necesarios de la vista y el modulo de controlador, ademas del modulo de tests, y no solo esto, sino que ademas registrará nuestro nuevo componente en el modulo principal (`app.module.ts`), en la sección de `declarations`.

Generalmente, un componente creado de esta manera constará de:

- En su controlador:

  - Decorador Component, con los metadatos básicos
  - Modificador OnInit: este hook se implementa al exportar la clase del componente, y se llama como un metodo dentro de la clase. Esto nos permite ejecutar acciones cuando el componente se ha creado y esta listo para ser renderizado.

    ```javascript
    import { Component, OnInit } from "@angular/core";

    @Component({
      selector: "app-saludo",
      templateUrl: "./saludo.component.html",
      styleUrls: ["./saludo.component.scss"],
    })
    export class SaludoComponent implements OnInit {
      // En el constructor se podrán inyectar dependencias y providers
      constructor() {}

      ngOnInit(): void {
        // instrucciones previas al renderizado del componente
      }
    }
    ```

- En la vista:
  - Genera una etiqueta `<p>` con un texto que indica que el componente funciona.

Al momento de crear el componente ya podemos empezar a enviarle información desde el controlador hacia la vista simplemente declarando variables en la clase del controlador y renderizandolas en la vista a través de la sintaxis de double-mustaches.

### 3.a Pasar Información de un Componente Padre a un Componente Hijo

Para pasar información de un componente padre a un componente hijo, necesitamos hacer uso del decorador Input en el controlador del componente hijo, que permitirá esta acción:

```javascript
import { Component, OnInit, Input } from "@angular/core";

@Component({
  selector: "app-saludo",
  templateUrl: "./saludo.component.html",
  styleUrls: ["./saludo.component.scss"],
})
export class SaludoComponent implements OnInit {
  // El decorador Input permite pasar información de un componente de orden superior a un componente de orden inferior
  // Si no se declara una variable en el componente padre, este es el valor que se renderizará por defecto
  @Input() nombre: string = "Anonimo";

  // En el constructor se podrán inyectar dependencias y providers
  constructor() {}

  ngOnInit(): void {
    // instrucciones previas al renderizado del componente
  }
}
```

Y en el componente padre, tenemos que pasar esta variable declarada con el decorador `@Input` como una directiva en la etiqueta del componente, en la sección de la vista:

```html
<div class="content" role="main">
  <!-- <h1>¡Hola, {{ nombre }}!</h1> -->
  <app-saludo nombre="Erik"></app-saludo>
</div>
```

De esta manera, en el componente hijo estamos "haciendo lugar" a esa variable que vamos a recibir desde el componente padre e incluso le estamos asignando un valor por defecto, por si esta variable no se envía de manera correcta. Mientras tanto, en la etiqueta de componente en la vista del componente padre estamos pasando efectivamente esta información que el componente hijo tomará a través de su decorador.
Es importante que esta variable se declare con el mismo nombre en el decorador del componente hijo y en la directiva de la etiqueta en el componente padre.

Algo a tener en cuenta es que esta forma de pasar variables, solo pasa variables estaticas; es decir, es como si estuvieramos creando la variable en la misma etiqueta del componente padre y no podemos cambiarla, a menos que lo hagamos manualmente.

Pero **_¿Cómo puedo hacer si quiero que el componente padre pase una variable que se encuentra declarada en su controlador, para darle reactividad a la app?_**

En caso de querer hacer esto, podemos hacer unos ligeros cambios.

Para empezar, vamos a necesitar declarar la variable en el controlador de nuestro componente padre:

```javascript
import { Component } from "@angular/core";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.scss"],
})
export class AppComponent {
  nombre = "Erik";
  title = "HolaMundo";
}
```

Luego, haremos un cambio en como declaramos esta propiedad en la etiqueta en la vista del mismo componente, agregando corchetes a la directiva y cambiando el valor:

```html
<div class="toolbar" role="banner">
  <img
    width="40"
    alt="Angular Logo"
    src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTAgMjUwIj4KICAgIDxwYXRoIGZpbGw9IiNERDAwMzEiIGQ9Ik0xMjUgMzBMMzEuOSA2My4ybDE0LjIgMTIzLjFMMTI1IDIzMGw3OC45LTQzLjcgMTQuMi0xMjMuMXoiIC8+CiAgICA8cGF0aCBmaWxsPSIjQzMwMDJGIiBkPSJNMTI1IDMwdjIyLjItLjFWMjMwbDc4LjktNDMuNyAxNC4yLTEyMy4xTDEyNSAzMHoiIC8+CiAgICA8cGF0aCAgZmlsbD0iI0ZGRkZGRiIgZD0iTTEyNSA1Mi4xTDY2LjggMTgyLjZoMjEuN2wxMS43LTI5LjJoNDkuNGwxMS43IDI5LjJIMTgzTDEyNSA1Mi4xem0xNyA4My4zaC0zNGwxNy00MC45IDE3IDQwLjl6IiAvPgogIDwvc3ZnPg=="
  />
  <span>Mi primera aplicacion de Angular: {{ title }}</span>
</div>

<div class="content" role="main">
  <!-- Aqui le indicamos que busque la variable "nombre" dentro del controlador, mediante el uso de corchetes. Ahora, en lugar de pasar un valor estatico, pasaremos el nombre de la variable a buscar -->
  <app-saludo [nombre]="nombre"></app-saludo>
</div>
```

- Los corchetes indican propiedades y atributos que se pueden encontrar en el controlador del componente,
- Los parentesis indican eventos

### 3.b Pasar Información de un Componente Hijo a un Componente Padre y viceversa (Double Binding)

Para poder pasar información en sentido inverso, existe el evento ngModel.

Este evento bindea en dos direcciónes:

- Esta atento al valor en el componente padre para utilizarlo y/o renderizarlo en el componente hijo, pero tambien
- Esta atento a los cambios que se puedan generar en el componente hijo para modificar el valor en el componente padre.

El `ngModel` es una directiva que asocia un evento de cambio con una propiedad concreta. Es importante saber que si vamos a utilizar la directiva `ngModel` para hacer un double binding, necesitamos importar en el modulo general de nuestra app el `FormsModule` de `@angular/forms`, para que se reconozca como una directiva de HTML.

```html
<div class="toolbar" role="banner">
  <img
    width="40"
    alt="Angular Logo"
    src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTAgMjUwIj4KICAgIDxwYXRoIGZpbGw9IiNERDAwMzEiIGQ9Ik0xMjUgMzBMMzEuOSA2My4ybDE0LjIgMTIzLjFMMTI1IDIzMGw3OC45LTQzLjcgMTQuMi0xMjMuMXoiIC8+CiAgICA8cGF0aCBmaWxsPSIjQzMwMDJGIiBkPSJNMTI1IDMwdjIyLjItLjFWMjMwbDc4LjktNDMuNyAxNC4yLTEyMy4xTDEyNSAzMHoiIC8+CiAgICA8cGF0aCAgZmlsbD0iI0ZGRkZGRiIgZD0iTTEyNSA1Mi4xTDY2LjggMTgyLjZoMjEuN2wxMS43LTI5LjJoNDkuNGwxMS43IDI5LjJIMTgzTDEyNSA1Mi4xem0xNyA4My4zaC0zNGwxNy00MC45IDE3IDQwLjl6IiAvPgogIDwvc3ZnPg=="
  />
  <span>Mi primera aplicacion de Angular: {{ title }}</span>
</div>

<div class="content" role="main">
  <!-- <h1>¡Hola, {{ nombre }}!</h1> -->

  <!-- Input para trabajar con ngModel -->
  <!-- En algunos casos, en uso del ngModel puede traer problemas de Typescript al indicar que no es una directiva declarada en HTML, con lo cual es necesario importar el modulo de Forms de Angular en el app.module.ts -->
  <!-- ngModel se declara con corchetes y con parentesis para indicar que es un bindeo doble, es decir, que esta tanto enviando información como recibiendola -->
  <input type="text" placeholder="Nombre del usuario" [(ngModel)]="nombre" />

  <!-- Aqui hay un bindeo simple, ya que la información solo se envía al componente hijo -->
  <app-saludo [nombre]="nombre"></app-saludo>
</div>
```

### 3.c Eventos

En la vista se pueden dar distintos eventos que se pueden controlar con metodos declarados en el controlador.

Ej:

En el controlador:

```javascript
import { Component, OnInit, Input } from "@angular/core";

@Component({
  selector: "app-saludo",
  templateUrl: "./saludo.component.html",
  styleUrls: ["./saludo.component.scss"],
})
export class SaludoComponent implements OnInit {
  @Input() nombre: string = "Anonimo";

  constructor() {}

  ngOnInit(): void {
    console.log("ngOnInit del componente Saludo");
  }

  // Ejemplo de gestion de un evento Click en el DOM

  // Creamos un método que se ejecutará al dispararse un evento en la vista
  alertaSaludo(): void {
    alert(`Hola, ${this.nombre}. Alerta despachada desde un click de botón`);
  }
}
```

En la vista:

```html
<h1>¡Hola, {{ nombre }}!</h1>

<!-- Creamos un botón que gestiona un evento -->
<!-- El evento se declara con parentesis y se vincula a un metodo del controlador, en este caso, el alertaSaludo() -->
<button id="emit-alerta" (click)="alertaSaludo()">Mostrar Alerta</button>
```

## 4. Uso de Directivas - InnerText

Las directivas que se pueden vincular a valores definidos en el controlador son varias, como por ejemplo:

- Utilizar la directiva `[innerText]` para declarar el texto interno de una etiqueta sin necesidad de utilizar la sintaxis de double mustaches.
  Ej:
  ```html
  <h3 id="usuario" [innerText]="nombre">
    <!-- No indicamos nada en las etiquetas, sino que el valor viene definido por la propiedad -->
  </h3>
  ```

> InnerText en particular, quizas puede ser util para inyectar HTML dentro de la etiqueta

## 5. Uso de Outputs

Mientras los Inputs son decoradores que se utilizan para pasar información de "arriba a abajo" (componente padre -> componente hijo), los Outputs se encargan de gestionar eventos, es decir, de "abajo hacia arriba" (componente hijo -> componente padre).

Los Outputs son eventos que ocurren en el hijo y que ejecutan algo en el padre.

Para utilizar el Output vamos a declarar un event emitter que enviará la información al componente padre

Ejemplo:

- En el controlador del componente hijo declaramos el decorador `@Output` con el eventEmitter, y creamos el método que hace uso de ese eventEmitter en cuestión:

  ```javascript
  import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

  @Component({
    selector: 'app-saludo',
    templateUrl: './saludo.component.html',
    styleUrls: ['./saludo.component.scss'],
  })
  export class SaludoComponent implements OnInit {
    // El decorador Input permite pasar información de un componente de orden superior a un componente de orden inferior
    // Si no se declara una variable en el componente padre, este es el valor que se renderizará por defecto
    @Input() nombre: string = 'Anonimo';

    @Output() mensajeEmitter: EventEmitter<string> = new EventEmitter<string>()

    // En el constructor se podrán inyectar dependencias y providers
    constructor() {}

    ngOnInit(): void {
      // instrucciones previas al renderizado del componente
      console.log('ngOnInit del componente Saludo');
    }

    //Ejemplo de gestion de un evento Click en el DOM y enviar un texto al componente Padre
    enviarMensajeAlPadre(): void {
      this.mensajeEmitter.emit(`Hola, ${this.nombre}. Alerta despachada desde un click de botón`)
    }
  }

  ```

- En la vista del componente hijo declaramos el evento en la etiqueta correspondiente, haciendo uso del método declarado en el controlador:

  ```html
  <h1>¡Hola, {{ nombre }}!</h1>

  <h3 id="usuario" [innerText]="nombre">
    <!-- No indicamos nada en las etiquetas, sino que el valor viene definido por la propiedad -->
  </h3>

  <!-- Creamos un botón que gestiona un evento -->
  <!-- El evento se declara con parentesis y se vincula a un metodo del controlador, en este caso, el alertaSaludo() -->
  <button id="emit-alerta" (click)="enviarMensajeAlPadre()">
    Enviar Mensaje de Saludo al Componente Padre</button
  >
  ```

- En el controlador del componente padre creamos un método que recibe el mensaje del componente hijo y realiza una operacion con el:

  ```javascript
  import { Component } from "@angular/core";

  @Component({
    selector: "app-root",
    templateUrl: "./app.component.html",
    styleUrls: ["./app.component.scss"],
  })
  export class AppComponent {
    usuario = "Erik";
    title = "HolaMundo";

    // Esta funcion se ejecuta cuando en el hijo se pulse un boton
    recibirMensajeDelHijo(evento: string) {
      alert(evento);
    }
  }
  ```

- En la vista del componente padre declaramos el evento con el nombre que declaramos en el decorador `@Output` del componente hijo, y le pasamos el método declarado en el componente padre para operar con la información del evento:

  ```html
  <div class="toolbar" role="banner">
    <img
      width="40"
      alt="Angular Logo"
      src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTAgMjUwIj4KICAgIDxwYXRoIGZpbGw9IiNERDAwMzEiIGQ9Ik0xMjUgMzBMMzEuOSA2My4ybDE0LjIgMTIzLjFMMTI1IDIzMGw3OC45LTQzLjcgMTQuMi0xMjMuMXoiIC8+CiAgICA8cGF0aCBmaWxsPSIjQzMwMDJGIiBkPSJNMTI1IDMwdjIyLjItLjFWMjMwbDc4LjktNDMuNyAxNC4yLTEyMy4xTDEyNSAzMHoiIC8+CiAgICA8cGF0aCAgZmlsbD0iI0ZGRkZGRiIgZD0iTTEyNSA1Mi4xTDY2LjggMTgyLjZoMjEuN2wxMS43LTI5LjJoNDkuNGwxMS43IDI5LjJIMTgzTDEyNSA1Mi4xem0xNyA4My4zaC0zNGwxNy00MC45IDE3IDQwLjl6IiAvPgogIDwvc3ZnPg=="
    />
    <span>Mi primera aplicacion de Angular: {{ title }}</span>
  </div>

  <div class="content" role="main">
    <!-- <h1>¡Hola, {{ nombre }}!</h1> -->

    <!-- Input para trabajar con ngModel -->
    <!-- En algunos casos, en uso del ngModel puede traer problemas de Typescript al indicar que no es una directiva declarada en HTML, con lo cual es necesario importar el modulo de Forms de Angular en el app.module.ts -->

    <input type="text" placeholder="Nombre del usuario" [(ngModel)]="usuario" />

    <!-- Pasamos el evento mensajeEmitter, que ejecuta la funcion recibirMensajeDelHijo, con el evento como argumento -->

    <app-saludo
      [nombre]="usuario"
      (mensajeEmitter)="recibirMensajeDelHijo($event)"
    ></app-saludo>
  </div>
  ```

## 6. LifeCycle

El LifeCycle de los componentes es siempre el mismo que en otros frameworks como React o Vue:

- Created (`ngOnInit`): se ejecuta el método cuando se crea el componente.
- Before Mounted
- Mounted
- Before Updated
- Updated (`onChanges`): se ejecuta el método cuando hay cambios en el estado del componente.
- Before Unmount (`onDestroy`): se ejecuta el método cuando el componente se desmonta.

### 6.a Utilización de los hooks de Ciclo de Vida

```javascript
import { Component, OnInit, Input, Output, EventEmitter, OnChanges, OnDestroy, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-saludo',
  templateUrl: './saludo.component.html',
  styleUrls: ['./saludo.component.scss'],
})
export class SaludoComponent implements OnInit, OnDestroy, OnChanges {

  // Decoradores de Inputs y Outputs
  @Input() nombre: string = 'Anonimo';
  @Output() mensajeEmitter: EventEmitter<string> = new EventEmitter<string>()

  // Constructor
  constructor() {}

  // Hooks de Ciclos de Vida
  ngOnInit(): void {
    // instrucciones previas al renderizado del componente
    console.log('ngOnInit del componente Saludo');
  }

  // Al declarar los SimpleChanges como parametro del ngOnChanges, podemos ver los cambios individuales (es decir, cada uno de los cambios que se ejecutarán como un batch luego) que se generaron al modificar el estado, en forma de un objeto que nos indica las variables que cambiaron, y el valor previo y el valor actual
  ngOnChanges(changes: SimpleChanges): void {
    // instrucciones al actualizar el componente
    console.log('Valor Previo', changes['nombre'].previousValue);
    console.log('Valor Actual', changes['nombre'].currentValue);
  }

  ngOnDestroy(): void {
    // instrucciones previas a la destrucción del componente
    console.log('ngOnDestroy el componente va a desaparecer');
  }

  // Métodos, etc.

}
```

### 6.b Orden de ejecución del Ciclo de Vida

1. `ngOnChanges` (Importante)
2. `ngOnInit` (Importante)
3. `ngAfterContentInit`
4. `ngAfterContentChecked`
5. `ngAfterViewInit`
6. `ngAfterViewChecked`
7. `ngAfterContentChecked`
8. `ngAfterViewChecked`
9. `ngOnDestroy` (Importante)

## 7. Estilos de los componentes

En lo que respecta a los estilos, estos funcionan como cualquier estilo CSS respecto a la sintaxis (dependiendo del preprocesador que utilicemos tambien), pero lo importante es que la ruta al archivo de estilo este declarada correctamente en el controlador del componente, y que las clases se correspondan con las clases utilizadas en las etiquetas en la vista.

Pero esto no es todo. Angular nos provee de la directiva `ngClass`, que se puede vincular a una variable en el controlador, lo que nos permite establecer clases dinamicas, es decir, con condicionalidades.

Por ejemplo:

```html
<h1>¡Hola, {{ nombre }}!</h1>

<!-- ngClass nos permite vincular la clase al estado de la variable "nombre" en el controlador -->
<h3
  id="usuario"
  [ngClass]="nombre == 'Anonimo' ? 'anonimo' : 'saludo'"
  [innerText]="nombre"
></h3>

<button id="emit-alerta" (click)="enviarMensajeAlPadre()">
  Enviar Mensaje de Saludo al Componente Padre
</button>
```

Tambien podemos hacer uso de la directiva `ngStyle`, que nos permite definir un objeto con parametros al estilo CSS en el controlador que se utilizaran como estilos en la vista (similar a lo que nos permite hacer Styled Components en React).

Ejemplo:

- En el controlador: 

```javascript
import {
  Component,
  OnInit,
  Input,
  Output,
  EventEmitter,
  OnChanges,
  OnDestroy,
  SimpleChanges,
} from '@angular/core';

@Component({
  selector: 'app-saludo',
  templateUrl: './saludo.component.html',
  styleUrls: ['./saludo.component.scss'],
})
export class SaludoComponent implements OnInit, OnDestroy, OnChanges {
  // Inputs y Outputs
  @Input() nombre: string = 'Anonimo';
  @Output() mensajeEmitter: EventEmitter<string> = new EventEmitter<string>();

  // Estilos para ngStyle
  myStyle: object = {
    color: 'blue',
    fontSize: '20px', // En los objetos de estilos, se utiliza camelCase
    fontWeight: 'bold',
  };

  // Constructor
  constructor() {}

  //Hooks de Lifecycle
  //...

  // Metodos
  //...
}
```

- En la vista: 

```html
<h1>¡Hola, {{ nombre }}!</h1>

<h3 id="usuario" [ngClass]="nombre == 'Anonimo' ? 'anonimo' : 'saludo'" [innerText]="nombre">
    <!-- No indicamos nada en las etiquetas, sino que el valor viene definido por la propiedad -->
</h3>

<p [ngStyle]="{color: 'orange'}">Hola, ¿Que tal?</p>
<p [ngStyle]="myStyle">Adios, muy buenas</p>


<button id="emit-alerta" (click)="enviarMensajeAlPadre()">Enviar Mensaje de Saludo al Componente Padre</button>
```

Recomendacion de utilizacion de estas directivas:

- Si trabajamos con preprocesadores podemos usar el `ngClass` sin problemas
- Si trabajamos con CSS puro, trabajar con `ngStyle` es practico ya que podemos crear un archivo de estilos que podemos modificar a traves de cambiar unas pocas variables
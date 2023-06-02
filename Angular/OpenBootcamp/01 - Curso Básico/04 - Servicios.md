# 04 - Servicios

## 0. Contenido

- ¿Que son los servicios y para que sirven?
- ¿Como funciona la inyeccion de las clases especificas de servicios?
- ¿Cual es su utilidad y filosofia?
- ¿Como se crea un servicio? ¿Como accedemos a sus metodos?
- ¿Como gestionamos los métodos asíncronos de un servicio?
- HttpClientModule

## 1. Objetivo de los Servicios

La idea de los servicios es desgranar la logica para sacarla de los componentes, mediante la practica de tomar toda la logica, metodos y funcionalidades que seran compartidos por varios componentes, para ser gestionados desde otras clases (separadas del contexto de los componentes), y asi tener una única instancia de los mismos, que se inyectarán en el constructor del componente que los requiera, para hacer uso de sus datos y sus metodos.

De esta manera evitamos:

- Complejidad en los componentes
- Duplicar código
- Pasar por depuraciones complejas

En resumen, un servicio sirve para hacer funcionalidades especificas que suelen ser compartidas para toda la aplicacion.

Ejemplo:

- Proceso de Login
- Solicitudes HTTP

### 1.a ¿Que es un servicio?

Un servicio es una clase, con determinados métodos, y dentro del componente tendremos un decorador especifico para poder inyectarlo y utilizarlo.

Generalmente, un servicio se inyecta en un módulo, a nivel raíz, aunque tambien puede inyectarse en diferentes niveles del proyecto.

A través de los servicios podemos mantener el estado de la aplicación de manera sencilla, ademas de poder mantener los procesos asíncronos con la misma facilidad.

Generalmente, los servicios devolveran una promesa

### 1.b Crear un servicio

El proceso para crear un servicio por el CLI de Angular es tan simple como crear un componente nuevo.

Simplemente se utiliza el comando `ng generate`, aclarando que lo que se creará es una instancia de servicio, e indicandole el path:

```
ng generate service services/nombreDelServicio

ng g s services/nombreDelServicio
```

### 1.c ¿Como se ve un servicio?

Por dentro, un servicio recién creado se ve de la siguiente manera:

```javascript
import { Injectable } from "@angular/core"; // Utiliza el decorador "Injectable", para poder inyectarse en la app

@Injectable({
  // Este decorador es el que nos permite inyectar la información
  providedIn: "root", // El servicio se inyecta a nivel de raiz del módulo
})
export class ContactoService {
  constructor() {}
}
```

Si dentro del servicio no tenemos el `providedIn: 'root'`, siempre podemos inyectarlo, agregandolo en la opcion `providers` del modulo.

### 1.d Inyectar el servicio en el componente

Para inyectar el servicio en el componente, necesitamos importarlo y declararlo en el constructor de la clase del componente.

Ademas, si es un servicio que provee información que se va a utilizar inmediatamente, este se debe declarar en el `ngOnInit`, asignandolo a una variable local del componente.

Ej:

```javascript
import { Component, OnInit } from '@angular/core';
import { IContacto } from 'src/app/models/Contacto.interface';
import { ContactoService } from 'src/app/services/contacto.service';

@Component({
  selector: 'app-lista-contactos',
  templateUrl: './lista-contactos.component.html',
  styleUrls: ['./lista-contactos.component.scss'],
})
export class ListaContactosComponent implements OnInit {
  // Inyectamos el servicio en el constructor
  constructor(private contactoService: ContactoService) {}

  // Creamos una lista de contactos
  listaContactos: IContacto[] = [];

  ngOnInit(): void {
    // Obtener la lista de contactos que nos brinda el servicio
    this.listaContactos = this.contactoService.obtenerContactos();
  }
}
```

## 2. Observables

Al utilizar servicios, nos veremos mas comunmente enfrentados a la necesidad de resolver funciones asíncronas, con lo cual deberemos gestionar promesas.

Dentro de Angular contamos con `Observables`, unos objetos similares a las promesas, pero que emitiran un nuevo valor ante cada ejecucion.

Los `Observables` nos permiten suscribir un componente a un metodo, que irá emitiendo nuevos valores, que el componente puede escuchar.

Los `Observables` nos darán informacion futura a través de los `Pipes`, y nos permitirán mantener una escucha activa sobre un valor, en lugar de llamarlo nuevamente a través de una promesa.

Ejemplo:

a. Servicio con Promesa:

```typescript
import { Injectable } from "@angular/core";

// importamos la lista de contactos MOCK
import { CONTACTOS } from "../mocks/contactos.mock";
import { IContacto } from "../models/Contacto.interface";

// importamos Observables de rxjs
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class ContactoService {
  constructor() {}

  obtenerContactos(): Promise<IContacto[]> {
    return Promise.resolve(CONTACTOS);
  }

  obtenerContactoPorId(id: number): Promise<IContacto> | undefined {
    // Buscamos el contacto por ID dentro de la lista de contactos mockeados
    const contacto = CONTACTOS.find(
      (contacto: IContacto) => contacto.id === id
    );

    if (contacto) {
      return Promise.resolve(contacto);
    } else {
      return;
    }
  }
}
```

b. Componente que recibe el resultado de la promesa:

```typescript
import { Component, OnInit } from "@angular/core";
import { IContacto } from "src/app/models/Contacto.interface";
import { ContactoService } from "src/app/services/contacto.service";

@Component({
  selector: "app-lista-contactos",
  templateUrl: "./lista-contactos.component.html",
  styleUrls: ["./lista-contactos.component.scss"],
})
export class ListaContactosComponent implements OnInit {
  // Creamos una lista de contactos
  listaContactos: IContacto[] = [];
  contactoSeleccionado: IContacto | undefined;

  // Inyectamos el servicio en el constructor
  constructor(private contactoService: ContactoService) {}

  ngOnInit(): void {
    // Obtener la lista de contactos que nos brinda el servicio
    this.contactoService
      .obtenerContactos()
      .then((lista: IContacto[]) => {
        this.listaContactos = lista;
      })
      .catch((error) =>
        console.error(
          "Ha habido un error al recuperar la lista de contactos: " + error
        )
      )
      .finally(() => console.log("Petición de lista de contactos terminada"));
  }

  obtenerContacto(id: number) {
    this.contactoService
      .obtenerContactoPorId(id)
      ?.then((contacto: IContacto) => (this.contactoSeleccionado = contacto))
      .catch((error) =>
        console.error("Ha habido un error al recuperar el contacto: " + error)
      )
      .finally(() => console.log("Petición del contacto por ID terminado"));
  }
}
```

c. Servicio con Observable

```typescript
import { Injectable } from "@angular/core";

// importamos la lista de contactos MOCK
import { CONTACTOS } from "../mocks/contactos.mock";
import { IContacto } from "../models/Contacto.interface";

// importamos Observables de rxjs
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class ContactoService {
  constructor() {}

  obtenerContactos(): Promise<IContacto[]> {
    return Promise.resolve(CONTACTOS);
  }

  obtenerContactoPorId(id: number): Promise<IContacto> | undefined {
    // Buscamos el contacto por ID dentro de la lista de contactos mockeados
    const contacto = CONTACTOS.find(
      (contacto: IContacto) => contacto.id === id
    );

    if (contacto) {
      return Promise.resolve(contacto);
    } else {
      return;
    }
  }
}
```

d. Componente que se suscribe al Observable

```typescript
import { Component, OnDestroy, OnInit } from "@angular/core";
import { Subscription } from "rxjs";
import { IContacto } from "src/app/models/Contacto.interface";
import { ContactoService } from "src/app/services/contacto.service";

@Component({
  selector: "app-lista-contactos",
  templateUrl: "./lista-contactos.component.html",
  styleUrls: ["./lista-contactos.component.scss"],
})
export class ListaContactosComponent implements OnInit, OnDestroy {
  // Creamos una lista de contactos
  listaContactos: IContacto[] = [];
  contactoSeleccionado: IContacto | undefined;

  // Subscripcion de Servicio
  subscription: Subscription | undefined;

  // Inyectamos el servicio en el constructor
  constructor(private contactoService: ContactoService) {}

  obtenerContacto(id: number) {
    this.subscription = this.contactoService
      .obtenerContactoPorId(id)
      ?.subscribe(
        (contacto: IContacto) => (this.contactoSeleccionado = contacto)
      );
  }

  ngOnInit(): void {
    // Obtener la lista de contactos que nos brinda el servicio
    this.contactoService
      .obtenerContactos()
      .then((lista: IContacto[]) => {
        this.listaContactos = lista;
      })
      .catch((error) =>
        console.error(
          "Ha habido un error al recuperar la lista de contactos: " + error
        )
      )
      .finally(() => console.log("Petición de lista de contactos terminada"));
  }

  ngOnDestroy(): void {
    this.subscription?.unsubscribe(); // Nos permite desuscribirnos al momento de desmontar el componente
  }
}
```

> Es importante que cuando el componente se sucribe a un observable, tambien debe desuscribirse cuando el componente se destruye, para no dejar ejecutando en segundo plano una logica innecesaria.

## 3. Instalar el modulo de peticiones HTTP

Para poder trabajar con asincronia, lo que vamos a necesitar es el modulo de peticiones HTTP de Angular, el cual deberemos importar en el modulo de la aplicación, para poder utilizarla a lo largo de todos los componentes y servicios de la misma.

Para importar este modulo, debemos declararlo en los imports del modulo general, importandolo desde `@angular/commons/http`

Ej:

```javascript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { FormsModule } from "@angular/forms";

import { HttpClientModule } from "@angular/common/http";

import { AppComponent } from "./app.component";
import { SaludoComponent } from "./components/saludo/saludo.component";

// Módulo personalizado que exporta componentes de tipo Lista
import { ListsModule } from "./modules/lists/lists.module";
import { ListaContactosComponent } from "./components/lista-contactos/lista-contactos.component";

@NgModule({
  declarations: [AppComponent, SaludoComponent, ListaContactosComponent],
  imports: [
    BrowserModule, // para ruteo
    FormsModule, // Para poder utilizar el ngModels
    // Importamos el módulo personalizado
    ListsModule,
    // Importamos el modulo HTTPClientModule para hacer peticiones HTTP
    HttpClientModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

> Link de utilidad: https://reqres.in
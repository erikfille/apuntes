# 04 - Routing

## 0. Contenido

- Caracteristicas principales del enrutado
- Navegación a través de rutas
- Acceso a parámetros (Query, Params y paso de contenido entre rutas)
- Uso de Guards para protección de rutas y subrutas

## 1. Incorporación del enrutador

Tal como se vió hasta ahora, al momento de crear un nuevo proyecto, el CLI de Angular nos consulta si queremos incluir el modulo de enrutado, y luego nos consulta sobre que formato de hoja de estilo queremos utilizar.

Al darle "Si" al modulo de enrutado, lo va a incorporar a través de un archivo en la carpeta `src/app`, con el nombre `app-routing.module.ts`. Esto no implica que este modulo sea el unico que vamos a poder utilizar, sino que también podemos declarar otros modulos de enrutado, si asi lo deseamos.

Modulo de enrutado por defecto:

```typescript
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";

const routes: Routes = []; // Lista de rutas a las cuales podemos navegar desde nuestra app

@NgModule({
  // Importación de RouterModule, que viene de '@angular/router'
  imports: [RouterModule.forRoot(routes)],
  // Exportación del modulo para el modulo principal
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

> Este modulo estará declarado en el modulo principal de la app (`app.module.ts`)

Si no incorporamos por defecto el sistema de enrutado, siempre podemos declararlo, creándolo como un módulo mas e importándolo en el módulo principal de la aplicación.

El sistema de enrutado se encarga de sustituir un contenido por otro en una SPA.

Dentro del archivo `app.component.html`, al final de todo del contenido por defecto podemos encontrar la etiqueta `<router-outlet></router-outlet>`, que se encarga de pintar todo el HTML de los componentes.

Este sistema de enrutado no es distinto del sistema de enrutado de otros frameworks como React o Vue.

## 2. Declaración de las rutas

Las rutas nos permiten crear un sistema de carga dinamica bajo demanda desde una SPA (es decir, desde un único HTML), lo que nos evita el desarrollo de paginas estáticas (como era antes) y también que nos permitiría entre otras cosas, hacer un lazy loading

Una vez creados los componentes de Pagina que vamos a renderizar en cada una de las rutas, tenemos que declarar, en el `app-routing-module.ts` las rutas donde se renderizarán estos componentes.

Ejemplo de declaración de rutas:

```javascript
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";

import { HomePageComponent } from "./pages/home-page/home-page.component";
import { LoginPageComponent } from "./pages/login-page/login-page.component";
import { ContactsPageComponent } from "./pages/contacts-page/contacts-page.component";
import { ContactDetailPageComponent } from "./pages/contact-detail-page/contact-detail-page.component";
import { NotFoundError } from "rxjs";
import { NotFoundPageComponent } from "./pages/not-found-page/not-found-page.component";

const routes: Routes = [
  {
    // Declara la redirección al ingresar a la ruta raíz del proyecto. Si se ingresa a 'midominio.com/', se redireccionará a 'midominio.com/home'
    path: "",
    // Se indica que debe matchear la ruta completamente
    pathMatch: "full",
    redirectTo: "home",
  },
  {
    path: "login",
    component: LoginPageComponent,
  },
  {
    path: "home",
    component: HomePageComponent,
  },
  {
    path: "contacts",
    component: ContactsPageComponent,
  },
  {
    // :id indica que el contenido que viene luego de los ':' es información variable que se va a utilizar en el componente a renderizar.
    path: "contacts/:id",
    component: ContactDetailPageComponent,
  },
  {
    // Los '**' indican que si no se ha encontrado el componente a renderizar en la ruta, entonces debe renderizarse el componente indicado aqui
    path: "**",
    component: NotFoundPageComponent,
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

> El orden en que se declaran las rutas es importante. Las rutas con parametros deben declararse siempre debajo de la principal

## 2.a Creación de Subrutas

Dentro de cada declaración de rutas, podemos definir una opcion `children`, que conste de un array de objetos de ruta, donde cada elemento de ese array se renderice en una URL con el formato `'padre/hijo'`.

Ejemplo de declaración de subrutas:

```javascript
{
    path: 'home',
    component: HomePageComponent,
    // Declaración de subrutas (rutas hijas)
    children: [
      {
        // Indica el path anidado y el componente que debe renderizarse. En este caso 'home/hijo', y se renderiza el componente HijoComponent.
        // Aquí tambien podemos declarar un path vacio (path: '') para indicar que debe renderizarse en el path padre, y luego definir los hijos.
        path: 'hijo',
        component: HijoComponent
      }
    ]
  },
```

Algo importante para agregar es que si en la ruta padre definimos el `pathMatch` como `prefix`, la ruta padre se convertira en un prefijo que abarcara a todas las rutas hijas, permitiendo la navegación de una a la siguiente.

## 3. Navegación entre componentes

Para navegar en nuestra SPA, podemos hacerlo a través de distintas formas:

- A traves de un elemento HTML con la propiedad routerLink (que puede estar vinculada a una variable del componente o puede ser literal):
  Ej:

```HTML
<!-- Navegamos a /login desde nuestro elemento a -->
<!-- A través de una declaración literal -->
<a routerLink="/login">Iniciar Sesión</a>

<!-- A través de un valor del componente -->
<a [routerLink]="link">Iniciar Sesión</a>
```

- A través de una función en el controlador (disparada desde un botón, por ejemplo): Al disparar una acción desde un botón, podemos tener importado el Router en el componente y podemos utilizarlo para navegar a traves del método `.navigate`, dentro de la acción que se dispara.

## 4. Gestión de parametros entre rutas.

Tenemos que ser capaces de acceder al contenido de cada una de estas rutas, para obtener determinados parametros que deberán utilizarse en el componente que se renderizara en la ruta de destino.

Una forma de realizar esto son los Params, que, como se vio en el caso de `'contacts/:id`, se declaran con los `:` en el path.

De esta manera, teniendo la declaracion de rutas:

```javascript
const routes: Routes = [
  {
    // Declara la redirección al ingresar a la ruta raíz del proyecto. Si se ingresa a 'midominio.com/', se redireccionará a 'midominio.com/home'
    path: "",
    // Se indica que debe matchear la ruta completamente
    pathMatch: "full",
    redirectTo: "home",
  },
  {
    path: "login",
    component: LoginPageComponent,
  },
  {
    path: "home",
    component: HomePageComponent,
  },
  {
    path: "contacts",
    component: ContactsPageComponent,
  },
  {
    // :id indica que el contenido que viene luego de los ':' es información variable que se va a utilizar en el componente a renderizar.
    path: "contacts/:id",
    component: ContactDetailPageComponent,
  },
  {
    // Los '**' indican que si no se ha encontrado el componente a renderizar en la ruta, entonces debe renderizarse el componente indicado aqui
    path: "**",
    component: NotFoundPageComponent,
  },
];
```

Teniendo el componente Contacts:

```typescript
import { Component } from "@angular/core";

// Modelo de Contacto importado:
import { IContacto } from "src/app/models/contact.interface";

@Component({
  selector: "app-contacts-page",
  templateUrl: "./contacts-page.component.html",
  styleUrls: ["./contacts-page.component.scss"],
})
export class ContactsPageComponent {
  listaContactos: IContacto[] = [
    {
      id: 0,
      nombre: "Martin",
      apellidos: "San Jose",
      email: "martin@imaginagroup.com",
    },
    {
      id: 1,
      nombre: "Andrés",
      apellidos: "García",
      email: "andres@imaginagroup.com",
    },
    {
      id: 2,
      nombre: "Ana",
      apellidos: "Hernandez",
      email: "ana@imaginagroup.com",
    },
  ];

  constructor() {}
}
```

Y teniendo el HTML del componente Contacts:

```html
<h1>Tus contactos:</h1>

<!-- Listar los contactos y poder navegar al detalle del contacto al clickear uno de ellos -->
<div *ngFor="let contacto of listaContactos; let i = index">
  <!-- En este caso, routerLink añade el parametro pasado como subruta, ya que la ruta /contacts ya existe y esta declarada por encima de la ruta contacts/:id -->
  <div routerLink="{{ contacto.id }}">
    <h6>Nombre: {{ contacto.nombre }}</h6>
  </div>
</div>
```

Con los componentes estructurados de esta manera, al seleccionar un contacto en la vista de ContactList, podemos acceder a la vista que contiene el componente de Contact con el detalle del usuario, permitiendonos recuperar la info del usuario a través del id pasado por parametro en el path.

Entre todos los parametros a los que podemos acceder, estan:

- `params`: devuelve un observable con los parametros que vienen en la ruta
- `queryParams`: Son aquellos parametros que llegan por consulta de query
- `snapshot`: devuelve las rutas que tenemos en ese momento en concreto

Ejemplo de acceso a Params:

```typescript
import { Component, OnInit } from "@angular/core";
import { ActivatedRoute } from "@angular/router";

@Component({
  selector: "app-contact-detail-page",
  templateUrl: "./contact-detail-page.component.html",
  styleUrls: ["./contact-detail-page.component.scss"],
})
export class ContactDetailPageComponent implements OnInit {
  id: any | undefined;

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    // Leemos los parametros, suscribiendonos a los params
    this.route.params.subscribe((params: any) => {
      // Si params.id existe
      if (params.id) {
        // Guardamos esta info en nuestra variable local
        this.id = params.id;
      }
    });
  }
}
```

Entre otras opciones que nos brinda ActivatedRoute, esta la capacidad de acceder a parametros de la ruta padre (parent) o de la ruta hija (child), lo cual puede ser fundamental para acceder a información en cascada de rutas.

## 5. Guards

Los Guards nos permiten tener un mayor control sobre el acceso a determinadas rutas.

Esto es especialmente util para definir el acceso a rutas que requieren de una autorizacion previa a traves de un login o register.

### 5.a Generar un Guard

Los guards tienen su propio metodo de generacion a traves de la CLI de Angular

```
ng g guard guards/nombreDelGuard

ng g g guards/nombreDelGuard
```

Al momento de generar un nuevo Guard, nos preguntara que interfaces queremos implementar para ese guard. Entre las opciones estan:

- CanActivate: Indica que se puede activar la ruta eventualmente para navegarla
- CanActivateChild: Indica que se pueden activar los hijos de esa ruta.
- CanDeactivate: Indica que se puede desactivar esa ruta.
- CanLoad: Indica que se puede cargar esa ruta, en caso de que sea un Lazy Loading.

Ejemplo de Guard predeterminado con CanActivate:

```typescript
import { CanActivateFn } from "@angular/router";

// Implementa la funcion CanActivate, que requiere por defecto de un RouteSnapshot (es decir, la ruta en la que estamos) y un estado, que puede ser un observable o una promesa booleanos o un booleano en general.
export const authGuard: CanActivateFn = (route, state) => {
  // Implementamos una condicion que debe cumplirse
  //...

  // La funcion debe retornar un booleano que indica si se carga la ruta (true) o no (false)
  return true;
};
```

En el enrutador del modulo de enrutado, los guards se aplican indicando la funcion que se va a ejecutar del mismo:

```javascript
const routes: Routes = [
  {
    // Declara la redirección al ingresar a la ruta raíz del proyecto. Si se ingresa a 'midominio.com/', se redireccionará a 'midominio.com/home'
    path: "",
    pathMatch: "full",
    redirectTo: "home",
  },
  {
    path: "login",
    component: LoginPageComponent,
  },
  {
    path: "home",
    component: HomePageComponent,
    // Declaración de subrutas (rutas hijas)
    children: [
      {
        // ...
      },
    ],
  },
  {
    path: "contacts",
    component: ContactsPageComponent,
    // La ruta tiene un guard que verifica si esa ruta se puede mostrar o no
    canActivate: [authGuard],
  },
  {
    path: "contacts/:id",
    component: ContactDetailPageComponent,
    canActivate: [authGuard],
  },
  {
    path: "**",
    component: NotFoundPageComponent,
  },
];
```

En caso de aplicar un guard a una ruta, esa ruta en cuestion debería tener un control de redirección en caso de que el guard impida la carga de la pagina, asi el usuario no se queda atrapado en una pagina en blanco.

Por ejemplo, en el Guard de mas arriba indicamos que se traiga un token del sessionStorage, que se ha guardado al hacer un login, y lo hacemos de la siguiente manera, indicando que sucede en caso de que ese token se encuentre y que sucede en caso de que no (en este caso, redireccionaría al login)

```typescript
import { CanActivateFn, Router } from "@angular/router";
import { inject } from "@angular/core";

export const authGuard: CanActivateFn = (route, state) => {
  // Como este es un componente funcional, usamos inject para inyectar el servicio Router tal como si lo hicieramos a traves de un constructor
  const router = inject(Router);

  // Simulamos la verificacion de un token guardado en sessionStorage

  let token = sessionStorage.getItem("token");

  // La funcion debe retornar un booleano que indica si se carga la ruta (true) o no (false)
  if (token) {
    return true;
  } else {
    router.navigate(["login"]);
    return false;
  }
};
```

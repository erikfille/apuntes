# Vue Router 3 (para Vue 2)

## 1. Comenzando

1. Instalar las dependencias al proyecto:

```
npm install vue-router
```

2. Importar el router al proyecto e indicar el uso del plugin.

```javascript
import Vue from "vue";
import VueRouter from "vue-router";

Vue.use(VueRouter);
```

## 2. Definición de las rutas basicas.

Ahora que tenemos todo listo para trabajar con las rutas, es necesario indicar donde se renderizará un componente y que componentes deberán matchear con que rutas para saber si deben renderizarse o no.

Supongamos que tenemos el siguiente codigo, donde en el HTML tenemos dos links (a Foo y a Bar), y en el JS indicamos la logica para renderizar esos componentes, a la par que se importan como dependencias:

HTML:

```html
<script src="https://unpkg.com/vue@2/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router@3/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- Utilizacion del componente <router-link> para la navegacion. -->
    <!-- Se especifica el link a traves de la prop 'to'. -->
    <!-- '<router-link>' se renderizara como una etiqueta '<a>' por default -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- Salida de la ruta -->
  <!-- El componente que matchee con la ruta se renderizara aqui -->
  <router-view></router-view>
</div>
```

Javascript:

```javascript
// 0. Importar Vue y VueRouter
// y llamar a `Vue.use(VueRouter)`.

Vue.use(router);

// 1. Definir los componentes ruteados.
import Foo from "./Foo.vue";
import Bar from "./Bar.vue";

// 2. Definir algunas rutas
// Cada ruta debe mapearse a un componente.
// El componente puede ser tanto un componente constructor creado via Vue.extend()
// o puede ser solo un componente de objeto de opciones
// Luego veremos rutas anidadas.
const routes = [
  { path: "/foo", component: Foo },
  { path: "/bar", component: Bar },
];

// 3. Crear una instancia de rutas y pasar la opcion `routes`.
// tambien se pueden pasar opciones adicionales, pero vamos a mantenerlo simple por ahora.
const router = new VueRouter({
  routes, // abreviatura para `routes: routes`
});

// 4. Crear y montar la instancia raiz.
// Asegurarse de injectar el router con la opcion de router para hacer que toda la app este al tanto de su utilizacion.
const app = new Vue({
  router,
}).$mount("#app");

// Ya se ha iniciado la App!
```

### `this.$router` y `this.$route`

A traves de injectar el Router, podemos acceder a este a traves de `this.$router`, asi como podemos acceder a la ruta actual a traves de `this.$route` dentro de cualquier componente:

Ej:

```javascript
export default {
  computed: {
    username() {
      // Ejemplo con uso de Params
      return this.$route.params.username;
    },
  },
  methods: {
    goBack() {
      window.history.length > 1 ? this.$router.go(-1) : this.$router.push("/");
    },
  },
};
```

Utilizar la instancia `this.$router` es lo mismo que utilizar `router`, pero no utilizamos esta ultima forma ya que no queremos importar `router` en cada componente en el que se deba manipular el ruteo.

## 2. Rutas Dinamicas

Las rutas dinamicas permiten que se renderice el mismo componente en distintas rutas con el mismo patron.
Un caso practico es cuando queremos renderizar la informacion de los usuarios de nuestra app. Como todos los usuarios tendran el mismo formato de informacion, podemos renderizar un unico componente para ellos, e indicar a traves de una ruta dinamica que alli se renderizará el componente de Usuarios, indicando a traves de la ruta un identificador unico que ayudará al componente a identificar que usuario tiene que traer para renderizar.

Ej:

```javascript
const User = {
  template: "<div>User</div>",
};

const router = new VueRouter({
  routes: [
    // Las rutas dinamicas inicial con dos puntos
    { path: "/user/:id", component: User },
  ],
});
```

Ahora las rutas como `/user/foo` y `user/bar` se mapearan a la misma ruta

El segmento dinamico se denota con los dos puntos (`:`) y cuando una ruta matchee, el valor del segmento dinamico se expondrá a través de `this.$route.params` en cada componente.
De esta manera, podemos renderizar el ID actual del usuario, mediante la actualizacion de la plantilla de `User` de esta manera:

```javascript
const User = {
  template: "<div>User {{ $route.params.id }} </div>",
};
```

Se pueden tener multiples segmentos dinámicos en la misma ruta, y cada uno de ellos mapeara los campos correspondientes en `$route.params`

Ejemplos:

| Patron                        | Rutas Matcheadas    | $route.params                        |
| :---------------------------- | :------------------ | :----------------------------------- |
| /user/:username               | /user/evan          | `{username: 'evan'}`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{username: 'evan', post_id: '123'}` |

El objeto `$route` no solo expone params, sino que tambien puede exponer otra informacion util como `$route.query` (si es que hay una query en la URL), `$route.hash`, etc.

> Info sobre el Objeto Route: https://v3.router.vuejs.org/api/#the-route-object

### Reaccionando a cambios en los Params

Algo notable es que cuando el usuario navega entre distintos renders del mismo componente a traves de rutas con params, la misma instancia del componente es reutilizada.
Como las rutas renderizan el mismo componente, esto es mas eficiente que destruir la vieja instancia y crear una nueva. Sin embargo, esto significa que los hooks del ciclo de vida del componente no se llamaran.

Para reaccionar a los cambios en los params en el mismo componente, se pueden hacer dos cosas:

Ejemplo:

```javascript
// 1) Utilizar un watcher en el componente, que escuche al objeto `$route`
const User = {
  template: "...",
  watch: {
    $route(to, from) {
      // reaccionar a los cambios de ruta
    },
  },
};

// 2) Utilizar el Navigation Guard beforeRouteUpdate()
const User = {
  template: "...",
  beforeRouteUpdate(to, from, next) {
    // reaccionar a los cambios de ruta
    // sin olvidar de llamar a next()
  },
};
```

Los Navigation Guards son provistos por `vue-router` y se encargan de vigilar los cambios en el ruteo y facilitan la navegación entre rutas

> Mas info sobre los Navigation Guards: https://v3.router.vuejs.org/guide/advanced/navigation-guards.html

### Atrapar todas / 404 Not Found

Mientras que los params regulares solo matchean los fragmentos de URL separados por una barra `/`, si queremos matchear cualquier cosa, podemos utilizar un asterisco:

```javascript
{
  // matcheara cualquier ruta
  path: "*";
}
{
  // matcheara cualquier ruta que comience con `/user-`
  path: "/user-*";
}
```

Al utilizar este tipo de rutas, es necesario que las rutas esten declaradas en el orden correcto y que las rutas con asteriscos esten al final. La ruta `{ path: '*' }` se utiliza generalmente para renderizar el error 404 del lado del cliente.

Cuando se utiliza un asterisco, se agrega un param llamado `pathMatch` a `$route.params`. Este contiene el resto de la URL matcheada por el asterisco:

```javascript
// Dada la ruta { path: '/user-*' }
this.$router.push("/user-admin");
this.$route.params.pathMatch; // 'admin'

// Dada la ruta { path: '*' }
this.$router.push("/non-existing");
this.$route.params.pathMatch; // '/non-existing'
```

> Como `vue-router` utilza `path-to-regexp`, permite patrones avanzados de reconocimiento. Para ver estos patrones, se puede visitar https://github.com/pillarjs/path-to-regexp/tree/v1.7.0#parameters

### Prioridad de Matcheo

Como uan misma URL puede matchear multiples rutas, la prioridad de matcheo se define por el orden de declaracion de las rutas, siendo las primeras declaradas las de prioridad mas alta, y las ultimas las de prioridad mas baja.

## 3. Rutas Anidadas

En las aplicaciones grandes, es habitual ver segmentos de URL que se correspondan a cierta estructura de componentes anidados.

Con `vue-router` es muy simple expresar esta relacion usando configuraciones de rutas anidadas.

Dado el ejemplo previo:

```html
<div id="app">
  `
  <router-view></router-view>
</div>
```

```javascript
const User = {
  template: "<div>User {{ $route.params.id }}</div>",
};

const router = new VueRouter({
  routes: [{ path: "/user/:id", component: User }],
});
```

Aqui `router-view` es una salida de alto nivel, ya que renderiza toda la aplicacion mediante el matcheo de una ruta de alto nivel. De manera similar, un componente renderizado puede contener su propio `<router-view>` anidado.

Por ejemplo, podemos agregar uno dentro del componente de `User`

```javascript
const User = {
  template: `
   <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `,
};
```

Para renderizar in esta salida anidada, necesitamos usar la opcion `children` en la configuracion del constructor `VueRouter`:

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/user/:id",
      component: User,
      children: [
        {
          // UserProfile se renderizara dentro del <router-view> de User
          // cuando matchee con /user/:id/profile
          path: "profile",
          component: UserProfile,
        },
        {
          // UserPosts se renderizara dentro del <router-view> de User
          // cuando matchee con /user/:id/posts
          path: "posts",
          component: UserPosts,
        },
      ],
    },
  ],
});
```

Notese que las direcciones anidadas que comienzan con `/` seran tratadas como la direccion raiz. Esto permite aprovechar el anidamiento de componentes sin tener que usar una URL anidada.

La opcion `children` es solo otro array de los objetos de configuracion de rutas, como las `routes` en si mismas. De esta manera se pueden anidar tantas vistas como se necesite.

En caso de que se quiera renderizar algo en un lugar donde no se matchea ninguna subruta, siempre se puede indicar que se renderice un componente por defecto, indicando un string vacio en el path:

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/user/:id",
      component: User,
      children: [
        // UserHome will be rendered inside User's <router-view>
        // when /user/:id is matched
        { path: "", component: UserHome },

        // ...other sub routes
      ],
    },
  ],
});
```

## 4. Navegacion Programatica

Ademas de utilizar `<router-link>` oara crear etiquetas `<a>` para la navegacion declarativa, tambien se pueden utilizar metodos de la instancia de router.

### `router.push(location, onComplete?, onAbort?)`

> Esto tambien se puede llamar `this.$router.push` dentro de una instancia de Vue.

Para navegar a una URL diferente, podemos usar `router.push`. Este metodo pushea una nueva entrada en la pila del historial de navegacion (history stack), asi que cuando el usuario clickea el boton de volver en el navegador, sera llevado a la url previa.

Este es el metodo que se llama internamente cuando clickeas un `<router-link>`, asi que clickear en un `router-link :to""...">` es equivalente a llamar a `router.push(...)`

El argumento puede ser un string de direccion o un objeto descriptor de locacion.

Ej:

```javascript
// String de direccion literal
router.push("home");

// Objeto
router.push({ path: "home" });

// Ruta nombrada
router.push({ name: "user", params: { userId: "123" } });

// Con un query, resultando en /register?plan=private
router.push({ path: "register", query: { plan: "private" } });
```

Los `params` se ignoran si un path se provee, el cual no es el caso para `query`, tal como se ve en el ejemplo de arriba. En su lugar, necesitas proveer el nombre de la ruta o especificar manualmente todo el path con cualquier parametro:

```javascript
const userId = "123";
router.push({ name: "user", params: { userId } }); // -> /user/123
router.push({ path: `/user/${userId}` }); // -> /user/123
// This will NOT work
router.push({ path: "/user", params: { userId } }); // -> /user
```

Esta misma regla aplica para la propiedad `to` del componente `router-link`.

En las versiones superiores a 2.2.0, se puede proveer de manera opcional las callbacks `onComplete` y `onAbort` para `router.push` o `router.replace` como segundo y tercer argumento. Estas callbacks seran llamadas cuando la navegacion se complete satisfactoriamente (luego de que todos los hooks asincronos se resuelvan), o sea abortada (navegando a la misma ruta, o a una ruta diferente antes de que la navegacion actual haya finalizado), respectivamente.
En las versiones superiores a la 3.1.0, se pueden omitir el segundo y tercer parametro y `router.push` / `router.replace` retornaran una promesa en su lugar, si las promesas son compatibles.

> Si el destino es el mismo a la ruta actual, y solo los params estan cambiando, se necesitará utilizar un `beforeRouteUpdate` para reaccionar a los cambios (como es el traer la informacion del usuario).

### `router.replace(location, onComplete?, onAbort?)`

Funciona como `router.push`, con la diferencia de que la navegacion no se pushea al historial de navegacion, sino que reemplaza la entrada actual.

### `router.go(n)`

Este metodo toma un unico numero entero como parametro, que indica cuantos pasos ir hacia adelante o hacia atras en el historial de navegacion, similar a lo que hace `windows.history.go(n)`

Ej:

```javascript
// Hacia adelante un registro, al igual que history.forward()
router.go(1);

// Hacia atras un registro, al igual que history.back()
router.go(-1);

// Hacia atras, 3 registros
router.go(3);

// Falla silenciosamente si no hay tantos registros.
router.go(-100);
router.go(100);
```

## 5. Rutas nombradas

A veces es mas conveniente identificar una ruta con un nombre, especialmente cuando se linea a una ruta o se realizan navegaciones.
Se puede nombrar una ruta en la opcion `routes` cuando se crea la instancia de Router:

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/user/:userId",
      name: "user",
      component: User,
    },
  ],
});
```

Para linkear a una ruta nombrada, se puede pasar como un objeto a la propiedad `to` del componente `router-link`, o se puede pasar de manera programatica a traves de `router.push()`:

```javascript
// Como propiedad to del componente router-link

<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

// Como parametro de router.push()

router.push({ name: 'user', params: { userId: 123 } })
```

En ambos casos, el router navegara hacia el path indicado.

## 6. Vistas nombradas

A veces se necesita mostrar multiples vistas al mismo tiempo, en lugar de anidarlas; por ejemplo, creando un layout con una vista de `sidebar` y una vista `main`. Aqui es cuando las vistas nombradas son utiles.
En lugar de tener una unica salida en tu vista, puedes tener multiples y asignar un nombre a cada una de ellas.
A un `router-view` sin un nombre se le dara el nombre `default`

```javascript
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

Una vista se renderiza utilizando un componente, por lo cual multiples vistas requieren de multiples componentes para la misma ruta. Para esto es necesario utilizar la opcion de `components`:

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/",
      components: {
        default: Foo,
        a: Bar,
        b: Baz,
      },
    },
  ],
});
```

### Vistas Nombradas Anidadas

Es posible crear layouts complejos utilizando vistas nombradas con vistas anidadas.
Al hacer esto, tambien se necesitara nombrar los componentes `router-view` anidados.

Un buen ejemplo sería un panel de Settings:

```
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
```

- `Nav` es un componente regular
- `UserSettings` es el componente de vista
- `UserEmailsSubscriptions`, `UserProfile`, `UserProfilePreview` son componentes de vistas anidadas

La seccion del componente `<template>` del componente `UserSettings` en el layout de arriba ser vería similar a esto:

```html
<!-- UserSettings.vue` -->
<div>
  <h1>User Settings</h1>
  <NavBar />
  <router-view />
  <router-view name="helper" />
</div>
```

> Los componentes de vista anidada se omiten aqui

Entonces puedes lograr el layout de arriba con esta configuracion:

```javascript
{
  path: './settings',
  // tambien se pueden utilizar las vistas nombradas en la parte superior
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

Ejemplo completo:

```javascript
const UserSettingsNav = {
  template: `
<div class="us__nav">
  <router-link to="/settings/emails">emails</router-link>
  <br>
  <router-link to="/settings/profile">profile</router-link>
</div>
`,
};
const UserSettings = {
  template: `
<div class="us">
  <h2>User Settings</h2>
  <UserSettingsNav/>
  <router-view class ="us__content"/>
  <router-view name="helper" class="us__content us__content--helper"/>
</div>
  `,
  components: { UserSettingsNav },
};

const UserEmailsSubscriptions = {
  template: `
<div>
	<h3>Email Subscriptions</h3>
</div>
  `,
};

const UserProfile = {
  template: `
<div>
	<h3>Edit your profile</h3>
</div>
  `,
};

const UserProfilePreview = {
  template: `
<div>
	<h3>Preview of your profile</h3>
</div>
  `,
};

const router = new VueRouter({
  mode: "history",
  routes: [
    {
      path: "/settings",
      // You could also have named views at tho top
      component: UserSettings,
      children: [
        {
          path: "emails",
          component: UserEmailsSubscriptions,
        },
        {
          path: "profile",
          components: {
            default: UserProfile,
            helper: UserProfilePreview,
          },
        },
      ],
    },
  ],
});

router.push("/settings/emails");

new Vue({
  router,
  el: "#app",
});
```

## 7. Redireccion y Alias

### Redireccion

La redireccion tambien se realiza en la configuracion de `routes`.

Para redireccionar de `/a` a `/b`:

```javascript
const router = new VueRouter({
  routes: [{ path: "/a", redirect: "/b" }],
});
```

La redireccion tambien puede realizarse a una ruta nombrada, o incluso utilizar una funcion para una redireccion dinamica:

```javascript
// Redireccion a Ruta Nombrada
const router = new VueRouter({
  routes: [{ path: "/a", redirect: { name: "foo" } }],
});

// Redireccion dinamica a traves de una funcion
const router = new VueRouter({
  routes: [
    {
      path: "/a",
      redirect: (to) => {
        // the function receives the target route as the argument
        // return redirect path/location here.
      },
    },
  ],
});
```

Debe notarse que los Navigation Guards no aplican en la ruta que redirecciona, sino que se aplican solo en la ruta a la cual se redireccion.

### Alias

Una redireccion significa que cuando el usuario visita `/a`, la URL sera reemplazada por `/b` y entonces se matcheara como `/b`. ¿Pero que es un alias?

Un alias de `/a` como `/b` significa que cuando el usuario visita `/b`, la URL se mantiene `/b`, pero se matcheara como si el usuario estuviera visitando `/a`

Lo dicho arriba puede expresarse en la configuacion de ruta como:

```javascript
const router = new VueRouter({
  routes: [{ path: "/a", component: A, alias: "b" }],
});
```

Un alias da la libertad de mapear una estrctura UI a una URL arbitraria, en lugar de estar restringido por la estructura anidada de la configuracion.

## 8. Pasar Props a los componentes de Ruta

Usar `$route` en tu componente crea un acoplamiento ajustado con la ruta, que limita la flexibilidad del componente, ya que solo podra ser utilizado con ciertas URLs

Para desacoplar el componente del router, se utiliza la opcion `props`:

En lugar de acoplar a `$route`:

```javascript
const User = {
  template: "<div>User {{ $route.params.id }}</div>",
};

const router = new VueRouter({
  routes: [{ path: "/user/:id", component: User }],
});
```

Se puede desacoplar utilizando `props`:

```javascript
const User = {
  props: ["id"],
  template: "<div>User {{ id }}",
};

const router = new VueRouter({
  routes: [
    { path: "user/:id", component: User, props: true },

    // para rutas con vistas nombradas, se debe definir la opcion 'props'
    {
      path: "/user/:id",
      components: {
        default: User,
        sidebar: Sidebar,
      },
      props: {
        default: true,
        // modo funcion, mas detalles abajo
        sidebar: (route) => ({ search: route.query.q }),
      },
    },
  ],
});
```

Esto permite utilizar el componente en cualquier lado, lo que hace que el componente sea mas sencillo de reutilizar y testear.

### Modo Booleano

Cuando `props` se configura en `true`, `route.params` sera seteado como las props del componente.

### Modo Objeto

Cuando `props` es un objeto, estas se configuraran como las props del componente per se. Esto es util cuando las props son estaticas:

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/promotion/from-newsletter",
      component: Promotion,
      props: { newsletterPopup: false },
    },
  ],
});
```

### Modo Funcion

Se puede crear una funcion que retorne las `props`. Esto permite lanzar parametros en otros tipos, combinar valores estaticos con valores basados en rutas, etc.

```javascript
const router = new VueRouter({
  routes: [
    {
      path: "/search",
      component: SearchUser,
      props: (route) => ({ query: route.query.q }),
    },
  ],
});
```

La URL `/search?q=vue` pasara `{ query: vue }` como props al componente `SearchUser`

Intenta mantener la funcion `props` sin estado, ya que solo se evalúan en los cambios de rutas.
Utiliza un componente wrapper si necesitas un estado para definir las props, asi Vue puede reaccionar a los cambios.

## 9. Modo History de HTML5

El modo por defecto para `vue-router` es _hash mode_. Se utiliza el URL hash para simular una URL completa , de manera que la pagina no será recargada cuando la URL cambie.

Para liberarse del hash, se puede utilizar el _history mode_ de router, lo cual facilita a la API `history.pushState` a alcanzar la navegacion de URL, sin recargar la pagina:

```javascript
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

Cuando se utiliza el modo history, la URL se vera "normal" (ejemplo: `http://nuestroSitio.com/user/id`)

Pero desde que nuestra app es una SPA del lado del cliente, sin una configuracion apropiada del servidor, los usuarios obtendran un error 404 si no acceden a `http://nuestroSitio.com/user/id` directamente en el browser.

Para solucionar este problema, lo unico que se necesita hacer es agregar una simple ruta fallback que atrape cualquier caso de error. Si la URL no matchea con ningun elemento estatico, debería devolver la misma pagina `index.html` en la que vive la aplicacion.

### Ejemplos de configuracion de Servidor

> Los ejemplos siguientes asumen que se esta mostrando tu app desde la carpeta raiz

Para Node.js con Express es recomendable utilizar el middleware connect-history-api-fallback

> Documentacion de connect-history-api-fallback: https://github.com/bripkens/connect-history-api-fallback

Primero es necesario instalar las dependencias:

```
npm install --save connect-history-api-fallback
```

```javascript
// 1) Importar la libreria
var history = require("connect-history-api-fallback");

// 2) Agregar el middleware a la aplicacion
var express = require("express");

var app = express();
app.use(history());
```

Existe una advertencia en este caso, y es que el servidor no reportara mas errores 404, ya que para todas las direcciones no encontradas se mostrara el archivo `index.html`.
Para evitar este problema, se debería implementar una ruta catch-all dentro de la app de Vue para mostrar una pagina 404:

```javascript
const router = new VueRouter({
  mode: "history",
  routes: [
    {
      path: "/:catchAll(.*)",
      component: NotFoundComponent,
      name: "NotFound",
    },
  ],
});
```

Alternativamente, si se esta utilizando un servidor de Node.js, se puede implementar la fallback mediante el uso del router del lado del servidor para matchear la URL y responder con un 404 si no se matchea ninguna ruta.

> Documentacion sobre Server Side Rendering (SSR) con Vue: https://vuejs.org/guide/scaling-up/ssr.html

## 10. Documentacion Avanzada

- Navigation Guards: https://v3.router.vuejs.org/guide/advanced/navigation-guards.html#global-before-guards
- Campos Meta de Ruta: https://v3.router.vuejs.org/guide/advanced/meta.html
- Transiciones: https://v3.router.vuejs.org/guide/advanced/transitions.html
- Data Fetching: https://v3.router.vuejs.org/guide/advanced/data-fetching.html
- Comportamiento del Scroll: https://v3.router.vuejs.org/guide/advanced/scroll-behavior.html
- Lazy Loading de Rutas: https://v3.router.vuejs.org/guide/advanced/lazy-loading.html
- Fallas de Navegacion: https://v3.router.vuejs.org/guide/advanced/navigation-failures.html 
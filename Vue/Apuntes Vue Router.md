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
      path: '/user/:id',
      component: User,
      children: [
        // UserHome will be rendered inside User's <router-view>
        // when /user/:id is matched
        { path: '', component: UserHome }

        // ...other sub routes
      ]
    }
  ]
})
```

## 4. Navegacion Programatica


# Vuex

## 1. ¿Que es Vuex?

Vuex es un patron y libreria de gestion de estado para aplicaciones desarrolladas con Vue.js. Funciona como un store centralizado para todos los componentes de la aplicacion, con reglas que aseguran que el estado solo puede ser mutado de manera predecible.

### ¿Que es un "Patron de Gestion de Estado"?

Ejemplo simple:

```javascript
const Counter = {
  // Estado
  data() {
    return {
      count: 0,
    };
  },
  // Vista
  template: `
    <div>{{ count }}</div>
  `,
  // Acciones
  methods: {
    increment() {
      this.count++;
    },
  },
};

createApp(Counter).mount("#app");
```

Esta es una app auto-contenida con las siguientes partes:

- El Estado, la fuente de verdad que guia a nuestra app
- La Vista, un mapeo declarativo del Estado
- Las Acciones, las formas posibles de cambiar el estado en relacion a los inputs del usuario a traves de la Vista

La siguiente es una simple representacion del concepto de "Flujo de Datos de Unico Sentido":

<img src="./assets/flow.png">

Sin embargo, la simpleza se rompe facilmente cuando varios componentes comparten un unico estado:

- Multiples vistas podrían depender de una misma pieza de estado
- Las acciones de diferentes vistas podrian necesitar mutar la misma pieza de estado.

Para el primer problema, pasar propiedades podria ser tedioso para los componentes aniadados profundamente, y simplemente no funciona para componentes hermanos.

Para el problema 2, generalmente nos encontramos apoyandonos en soluciones como buscar referencias de instancias padre/hijo o tratando de mutar y sincronizar multiples copias del estado via eventos.

Ambos patrones son fragiles y llevan rapidamente a un codigo inmantenible.

Mediante la creacion de un estado compartido por fuera de los componentes, cada uno de ellos podrá acceder al estado o disparar acciones, sin importar en que posicion esten en el arbol de componentes.

Mediante la definicion y separacion de los conceptos envueltos en la gestion de estado y reforzando las reglas que mantienen la independencia entre vistas y estados, podemos darte al codigo mayor estructura y mantencion.

Esta es la idea básica detras de Vuex, inspirada por Flux, Redux y la Arquitectura Elm. A diferencia de otros patrones, Vuex tambien es una libreria de implementacion creada especificamente para Vue.js, para tomar ventaja de su sistema de reactividad granular para actualizaciones eficientes.

> Curso gratuito para aprender Vuex: https://scrimba.com/learn/vuex

<img src="./assets/vuex.png">

### ¿Cuando se debería utilizar Vuex?

Vuex ayuda a lidiar con problemas de gestion de estado, con el costo de la necesidad de mayores conceptos y boilerplate. Es un intercambio entre productividad a corto y largo plazo.

Si necesitas desarrollar una app simple, estaría bien desarrollarla sin Vuex, pero si necesitas desarrollar una app de gran escala, lo mas probable es que Vuex te sea no solo util, sino necesario.

## 2. Instalacion

```
npm install vuex@next --save
```

## 3. Comenzando con Vuex

En el medio de toda aplicacion de Vuex se encuentra el Store.
Un Store basicamente es un contenedor que mantiene el estado de tu aplicacion.

Hay dos cosas que hacen que un Store de Vuex sea distinto a un simple objeto global:

1. Los Store de Vuex son reactivos. Cuando un componente de Vue toma su estado de el, van a actualizarse de manera reactiva y eficiente si el estado del Store cambia.

2. No se puede mutar directamente el estado del Store. La unica manera de cambiar el estado del Store es mediante la ejecucion explicita de mutaciones. Esto asegura que cada cambio de estado deja un registro rastreable y habilita herramientas que nos ayudaran a entender mejor nuestras aplicaciones.

### El Store mas simple

Luego de instalar Vuex, hay que crear un Store.

Para esto necesitamos proveer un objeto de estado inicial y algunas mutaciones:

```javascript
import { createApp } from "Vue";
import { createStore } from "Vuex";

// Crear una nueva instancia de Store

const store = createStore({
  state() {
    return {
      count: 0,
    };
  },
  mutations: {
    increment(state) {
      state.count++;
    },
  },
});

const app = createApp({
  /*el componente raiz*/
});

// Instalacion de la instancia del Store como un plugin
app.use(store);
```

Ahora se puede acceder al objeto de estado como `store.state` y se puede disparar un cambio de estado con el metodo `store.commit`:

```javascript
store.commit("increment");

console.log(store.state.count); // -> 1
```

En un componente Vue, se puede acceder al store a traves de `this.$store`.

Ahora podemos ejecutar una mutacion, utilizando el metodo del componente:

```javascript
methods: {
  increment() {
    this.$store.commit('increment')
  }
}
```

De nuevo, la razon por la que lanzamos (commit) una mutacion en lugar de cambiar `store.state.count` directamente esta en que queremos poder rastrearla de manera explicita.
Esta simple convencion hace tus intenciones mas explicitas, de manera que puedas razonar mejor sobre los cambios en el estado de tu aplicacion al leer el codigo. Ademas, nos da la oportunidad de implementar herramientas que pueden loguear cada mutacion, tomar capturas del estado o incluso realizar un debugging "a traves del tiempo".

Usar el estado del Store en un componente simplemente involucra retornar el estado dentro de una propiedad computada, ya que el estado del store es reactivo. Disparar cambios simplemente significa lanzar mutaciones en metodos del componente.

## 4. Estado

### Arbol de Unico Estado

Vuex utiliza un arbol de unico estado (Single State Tree), es decir, un unico objeto que contiene todos los niveles de estado y funciona como la "unica fuente de verdad". Esto tambien significa que usualmente se tendra un unico Store para cada aplicacion. Un arbol de unico estado hace que la localizacion de una pieza especifica del estado sea mas directa, y permite que hacer capturas del estado actual de la aplicacion con fines de debugging de manera mas sencilla.

### Llevar el estado de Vuex a los componentes de Vue

Ya que los Stores de Vuex son reactivos, la manera mas sencilla de "traer" el estado de Vuex es simplemente retornando algun estado del store desde dentro de una propiedad computada:

```javascript
// Creamos un componente Counter:
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count() {
      return store.state.count;
    },
  },
};
```

Cada vez que `store.state.count` cambie, causara que la propiedad computada se reevalue y dispare los las actualizaciones del DOM asociadas.

De todas formas, este patron causa que los componentes se apoyen en un unico estado global.
Cuando se utiliza un sistema de modulos, se requiere importar el Store en cada componente que utilice el estado del Store, y tambien requiere maquetarlo cuando se testea el componente.

Vuex "inyecta" el Store en cada componente hijo desde el componente raiz a traves del sistema de plugins de Vue, y estara disponible a traves de `this.$store`.
Actualizando la implementacion en `counter` nos quedaria:

```javascript
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count() {
      return this.$store.state.count;
    },
  },
};
```

### El `mapState`helper

Cuando un componente necesita hacer uso de multiples propiedades o getters del estado del Store, declarar todas estas propiedades computadas puede volverse repetitivo. Para lidiar con esto, podemos hacer uso del helper `mapState`, que genera funciones getter computadas por nosotros:

```javascript
// En builds completas, los helpers se muestran como Vuex.mapState

import { mapState } from "vuex";

export default {
  // ...
  computed: mapState({
    // Arrow functions pueden ayudar tener un codigo sucinto
    count: (state) => state.count,

    // Pasar el string 'count' es lo mismo que 'state => state.count'
    countAlias: "count",

    // para acceder al estado local con 'this' se debe utilizar una funcion normal
    countPlusLocalState(state) {
      return state.count + this.localCount;
    },
  }),
};
```

Tambien podemos pasar un array de strings a `mapState` cuando el nombre de la propiedad computada mapeada es el mismo que el nombre de la rama del estado.

```javascript
computed: mapState([
  // mapea this.count a store.state.count
  "count",
]);
```

### Spread Operator de Objeto

Notese que `mapState` retorna un objeto, pero a veces es necesario utilizar ese objeto junto con otras propiedades computadas locales.
Para poder hacer esto sencillo, podemos usar un spread operator de objeto:

```javascript
computed: {
  localComputed () { /* ... */ },
  // Mezcla esto con el objeto externo que incluye las propiedades computadas locales
  ...mapState({
    // ...
  })
}
```

### Los componentes aun pueden tener un Estado Local

Usar Vuex no significa que se deban poner todos los estados en Vuex. Aunque poner mas estados en Vuex puede hacer que las mutaciones sean mas explicitas y debuggeables, a veces tambien pueden hacer el codigo mas indirecto. Si una pieza de estado pertenece estrictamente a un unico componente, estaría bien dejarlo como un estado local.
Es necesario poner en la balanza los intercambios y tomar de decisiones que encajen con el desarrollo que necesita la app.

## 5. Getters

A veces podemos necesitar computar un estado derivado, basado en el estado del Store, como por ejemplo, filtrar a traves de una lista de elementos y contarlos:

```javascript
computed: {
  doneTodosCount(){
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

Si mas de un componente necesita usar esto, tenemos que o duplicar la funcion, o extraerla en un "ayudante" (helper) compartido e importarla en multiples lugares. Ambas opciones son menos que ideales.

Vuex permite definir "getters" en el store. Estos getters podrian pensarse como propiedades computadas para stores.

Los getters recibiran el estado como primer argumento:

```javascript
const store = createStore({
  state: {
    todos: [
      { id: 1, text: "...", done: true },
      { id: 2, text: "...", done: false },
    ],
  },
  getters: {
    doneTodos(state) {
      return state.todos.filter((todo) => todo.done);
    },
  },
});
```

### Estilo de Acceso de Propiedad

Los getters seran expuestos en el objeto `store.getters`, por lo que se puede acceder como propiedad

```javascript
store.getters.doneTodos; // [{ id: 1, text: '...', done: true }]
```

Los Getters tambien recibiran otros getters como segundo argumento:

```javascript
getters: {
  //...
  doneTodosCount(state, getters) {
    return getters.doneTodos.length
  }
}
```

```javascript
store.getters.doneTodosCount; // 1
```

Ahora se pueden utilizar facilmente dentro de cualquier componente:

```javascript
computed: {
  doneTodosCount() {
    return this.$store.getters.doneTodosCount
  }
}
```

> Los getters a los que se acceden como propiedades se cachean como parte del sistema de reactividad de Vue

### Estilo de acceso de Metodo

Se puede pasar argumento a los getters para retornar una funcion.
Esto es particularmente util cuando queremos operar sobre un array en el store:

```javascript
getters: {
  //...
  getTodoById: (state) => (id) => {
    return state.todos.find((todo) => todo.id === id);
  };
}
```

```javascript
store.getters.getTodoById(2); // -> { id: 2, text: '...', done: false }
```

> Los getters accedidos como metodos correran cada vez que se llamen y el resultado no es cacheado

### El helper `mapGetters`

El helper `mapGetters` simplemente mapea los getters del store a las propiedades computadas:

```javascript
import { mapGetters } from "vuex";

export default {
  //...
  computed: {
    // Se agregan los getters en 'computed' con el spread operator de objetos
    ...mapGetters([
      "doneTodosCount",
      "anotherGetter",
      //...
    ]),
  },
};
```

Si se quiere mapear un getter con un nombre diferente, se debe usar un objeto:

```javascript
...mapGetters({
  // mapea 'this.doneCount' a 'this.$store.getters.doneTodosCount'
  doneCount: 'doneTodosCount'
})
```

## 6. Mutaciones

La unica manera de efectivamente cambiar el estado en el store de Vuex es mediante el lanzamiento (commit) de una mutacion.
Las mutaciones de Vuez son muy similares a los eventos: cada mutacion tiene un tipo de String y un handler.

La funcion handler es donde se realizan las verdaderas modificaciones del estado y van a recibir el estado como primer argumento.

```javascript
const store = createStore({
  state: {
    count: 1,
  },
  mutations: {
    increment(state) {
      // muta el estado
      state.count++;
    },
  },
});
```

No se puede llamar directamente a un handler de mutacion. Piensa mas como un registro de evento: "Cuando la mutacion con tipo `increment` se dispare, llama a este handler". Para invocar un handler de mutacion, necesitamos llamar a `store.commit` con este tipo:

```javascript
store.commit("increment");
```

### Commit con un payload

Se pueden pasar argumentos adicionales a `store.commit`, lo que se llama la "payload" (carga) para la mutacion:

```javascript
//...
mutations: {
  increment(state, n) {
    state.count += n
  }
}
```

```javascript
store.commit("increment", 10);
```

En la mayoría de los casos, el payload debería ser un objeto, asi puede contener multiples campos, y la mutacion grabada puede ser mas descriptiva

```javascript
// ...
mutations: {
  increment(state, payload) {
    state.count += payload.amount
  }
}
```

```javascript
store.commit("increment", {
  amount: 10,
});
```

### Commit de Estilo de Objeto

Una forma alternativa de lanzar una mutacion es mediante el uso directo de un objeto que tiene la propiedad `type`:

```javascript
store.commit({
  type: "increment",
  amount: 10,
});
```

Cuando se utiliza commit con estilo de objeto, todo el objeto sera pasado como payload para los hanlders de mutacion, asi que el handler permanece igual:

```javascript
mutations: {
  increment(state, payload) {
    state.count += payload.amount
  }
}
```

### Usando constantes para los tipos de mutacion

En varias implementaciones de Flux, es un patron comunmente visto el de usar constantes para los tipos de mutacion.aprovechetome ventaja de herramientas como linters y poner todas las constantes en un unico archivo permite que los colaboradores tengan una vista rapida de que mutaciones son posibles en toda la aplicacion.

```javascript
// mutation-types.js
export const SOME_MUTATION = "SOME_MUTATION";
```

```javascript
// store.js
import { createStore } from 'vuex';
import { SOME_MUTATION } from './mutation-types';

const store = createStore({
  state: {...},
  mutations: {
    // podemos utilizar la caracteristica de ES2015 de propiedad computada de nombre para usar una constante como nombre de la funcion
    [SOME_MUTATIONS] (state) {
      // mutacion del estado
    }
  }
})
```

> Mientras que utilizar constantes es mas que nada una preferencia, esto puede ser util en proyectos grandes con muchos desarrolladores, pero es totalmente opcional si no deseas utilizarlas

### Las mutaciones deben ser sincronicas

Una regla importante para recordar es que los handlers de mutacion deben ser sincronicos.

Consideren el siguiente ejemplo:

```javascript
mutations: {
  someMutation(state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

Ahora imaginen que estamos debugueando nuestra app y estamos mirando el log de mutaciones del devtool.
Por cada mutacion logueada, la devtool necesitara capturar un "antes" y un "despues" del estado.
Sin embargo, la callback asincrona en la mutacion del ejemplo hace que esto sea imposible: la callback no se llama todavia cuando la mutacion se lanza, y no hay forma de que el devtool sepa cuando la callback sera llamara realmente. Cada mutacion realizada en esta callback es esencialmente no rastreable.

### Lanzar mutaciones en los componentes

Se pueden lanzar mutaciones en los componentes con `this.$store.commit('someMutation')`, o se puede utilizar el helper `mapMutations`, que mapeara metodos del componente a las llamadas de `store.commit` (esto requiere la inyeccion de la raiz de `store`):

```javascript
import { mapMutations } from "vuex";

export default {
  // ...
  methods: {
    ...mapMutations([
      "increment", // mapea 'this.increment()' a 'this.$store.commit('increment')'

      // 'mapMutations' tambien soporta payloads:
      "incrementBy", // mapea 'this.incrementBy(amount)' a 'this.$store.commit('incrementBy', amount)'
    ]),
    ...mapMutations({
      add: "increment", // mapea 'this.add' a 'this.$store.commit('increment')'
    }),
  },
};
```

### A las acciones

El asincronismo combinado con la mutacion del estado pueden hacer que tu programa sea muy dificil de entender.
Por ejemplo, cuando se llaman dos metodos, ambos con llamadas asincronas que mutan el estado... ¿como sabemos entonces cuando son llamados y que callback se llamo primero?

Esto es exactamente el por que queremos separar dos conceptos. En Vuex las mutaciones son transacciones sincronicas

```javascript
store.commit("increment");
// Cualquier cambio que la mutacion 'increment' realice deberia ser hecha en el mismo momento
```

Para gestionar las operaciones asincronicas es que existen las acciones

## 7. Acciones

Las acciones son similares a las mutaciones, pero se diferencian en que

- En lugar de mutar el estado, las acciones lanzan mutaciones.
- Las acciones pueden contener operaciones asincronas arbitrarias

Registremos una simple accion:

```javascript
const store = createStore({
  state: {
    count: 0,
  },
  mutations: {
    increment(state) {
      state.count++;
    },
  },
  actions: {
    increment(context) {
      context.commit("increment");
    },
  },
});
```

Los handlers de acciones reciben un objeto de contexto que expone el mismo set de metodos/propiedades en la instancia del store, por lo que se puede llamar `context.commit` para lanzar una mutacion o acceder al estado y los getter a traves de `context.state` y `context.getters`. Incluso podemos llamar otras acciones con `context.dispatch`.
Veremos como este objeto de contexto no es la instancia del store en si cuando veamos Modulos mas adelante.

En la practica, utilizamos seguido el desestructurador de argumento de ES2015 para simplificar el codigo un poco (sobre todo cuando neceistamos llamar a `commit` multiples veces):

```javascript
actions: {
  increment({ commit }) {
    commit('increment')
  }
}
```

### Despachando acciones

Las acciones se disparan con el metodo `store.dispatch`

```javascript
store.dispatch("increment");
```

Esto puede verse tonto a primera vista: si queremos incrementar la cuenta, ¿por que no llamamos directamente a `store.commit('increment')`?

Es necesario recordar que las mutaciones deben ser sincronicas. Las acciones no.

Podemos realizar operaciones asincronas dentro de la accion:

```javascript
actions: {
  incrementAsync({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

El dispatch de las acciones soportan el mismo formato de payload y estilo de objeto:

```javascript
// dispatch con un payload
store.dispatch("incrementAsync", {
  amount: 10,
});

// dispatch con un objeto
store.dispatch({
  type: "incrementAsync",
  amount: 10,
});
```

En ejemplo mas practico de las acciones en el "mundo real" podria ser el de la accion para hacer el checkout de un carrito de compras, que incluye el llamado a una API asincrona y lanzar multiples mutaciones:

```javascript
actions: {
  checkout({ commit, state }, products) {
    // guarda los elementos actualmente en el carrito
    const savedCartItems = [... state.cart.added]

    // envia el request de checkout y limpia el carrito de manera optimista
    commit(types.CHECKOUT_REQUEST)

    // La API del negocio acepta una callback de exito y una de fallo
    shop.buyProducts(
      products,
      // manejador de exito
      () => commit(types.CHECKOUT_SUCCESS),

      // manejador de error
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

> Notese que se esta realizando un flujo de operaciones asincronas, y guardando los efectos secundarios (mutaciones de estado) de cada accion que los lanza.

### Despachando acciones en los componentes

Se pueden despachar acciones en los componentes con `this.$store.dispatch('action')` o utilizar el helper de `mapActions` que mapea los metodos del componente a las llamadas de `store.dispatch` (esto requiere la inyeccion de la raiz de `store`):

```javascript
import { mapActions } from "vuex";

export default {
  // ...
  methods: {
    ...mapActions([
      "increment", // mapea `this.increment()` a `this.$store.dispatch('increment')`

      // `mapActions` tambien soporta payloads:
      "incrementBy", // mapea `this.incrementBy(amount)` a `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: "increment", // mapea `this.add()` a `this.$store.dispatch('increment')`
    }),
  },
};
```

### Componiendo acciones

Las acciones generalmente son asincronas, asi que ¿como sabemos cuando una accion se completo?.

Y mas importante, ¿Como podemos componer multimples acciones juntas para manejar flujos asincronos mas complejos?

Lo primero que tenemos que conocer es que `store.dispatch` puede manejar promesas retornadas por el handler de accion disparado y tambien retorna una promesa:

```javascript
actions: {
  actionA({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

Ahora se puede hacer:

```javascript
store.dispatch("actionA").then(() => {
  // ...
});
```

Y tambien, en otra accion:

```javascript
actions: {
  //...
  actionB({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

Finalmente, si usamos async/await, podemos componer nuestras acciones de la siguiente manera:

```javascript
// asumiendo que 'getData' y getOtherdata' retornan promesas

actions: {
  async actionA({ commit }) {
    commit('gotData', await getData())
  },
  async actionB({ dispatch, commit }) {
    await dispatch('actionA') // espera a que termine 'actionA'
    commit('gotOtherData', await getOtherData())
  }
}
```

> Es posible para un `store.dispatch` disparar multiples handlers de acciones en diferentes modulos. En estos casos, el valor retornado sera una promesa que resuelve cuando todos los handlers disparados se resuelvan.

## 8. Modulos

Debido al uso de un arbol de unico estado, todos los estados de nuestra aplicacion son contenidos dentro de un unico gran objeto. Sin embargo, mientras nuestra aplicacion crece en escala, el store se puede ver bastante cargado.

Para ayudar con esto, Vuex nos permite dividir nuestro estado en modulos. Cada modulo puede contener su propio estado, mutaciones, acciones, getters y hasta modulos anidados. Es fractal de arriba a abajo:

```javascript
const moduleA = {
  state: () => ({...}),
  mutations: {...},
  actions: {...},
  getters: {...}
}

const moduleB = {
  state: () => ({...}),
  mutations: {...},
  actions: {...},
}

const store = createdStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // Estado del 'moduleA'
store.state.b // Estado del 'moduleB'
```

### Modulo de Estado Local

Dentro de las mutaciones y los getters de un modulo, el primer argumento recibido sera el del estado local del modulo.

```javascript
const moduleA = {
  state: () => ({
    count: 0,
  }),
  mutations: {
    increment(state) {
      // 'state' es el estado del modulo local
      state.count++;
    },
  },
  getters: {
    doubleCount(state) {
      return state.count * 2;
    },
  },
};
```

De manera similar, dentro de las acciones de los modulos, `context.state` expondra al estado local, y el estado raiz sera expuesto como `context.rootState`:

```javascript
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit("increment");
      }
    },
  },
};
```

Tambien, dentro de los getters del modulo, el estado raiz sera expuesto como tercer argumento:

```javascript
const moduleA = {
  // ...
  getters: {
    sumWithRootCount(state, getters, rootState) {
      return state.count + rootState.count;
    },
  },
};
```

### Espacio de Nombres

Por defecto, las acciones y las mutaciones se siguen registrando bajo el espacio de nombres global. Esto permite que multiples modulos reaccionen al mismo tipo de mutacion/accion.

Los getters tambien se registran en el espacio de nombres global por defecto. Sin embargo, esto actualmente no tiene un fin funcional (ya que solo se utiliza para evitar cambios que puedan romper la aplicacion). Hay que ser cuidadoso con no definir dos getters con el mismo nombre en diferentes modulos sin espacio de nombre, resultando esto en un error.

Si deseas que tus modulos sean mas aislados y reutilizables, puedes marcarlos como nombre-espaciado con `namespaced: true`.
Cuando el modulo se registra, todos sus getters, acciones y mutaciones seran llamadas automaticamente basadas en la direccion donde el modulo esta registrado. Por ejemplo:

```javascript
const store = createStore({
  modules: {
    account: {
      namespaced: true,

      // module assets
      state: () => ({ ... }), // El estado del modulo ya esta anidado y no es afectado por la opcion 'namespaced'
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // Modulos Anidados
      modules: {
        // Hereda el namespace del modulo padre
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // anida aun mas el namespace
        posts: {
          namespaced: true,

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

Los getters y las acciones con namespacing recibiran `getters`, `dispatch` y `commit` localizados.

En otras palabras, puedes utilizar los activos del modulo sin la necesidad de escribir el prefijo en el mismo modulo. Cambiar entre la opcion de namespaced o no-namespaced no afecta el codigo dentro del modulo.

#### Accediendo a activos globales en modulos con namespace

Si se desea utilizar estados globales y getters, `rootState`y `rootGetters` se pasan como tercer y cuarto argumentos en las funciones de getters, y tambien se exponen como propiedades en el objeto `context` pasado a las funciones de accion.

Para despachar acciones o lanzar mutaciones en el namespace global, se puede pasar `{ root: true }` como el tercer argumento a `dispatch` y `commit`.

```javascript
modules: {
  foo: {
    namespaced: true,

    getters: {
      // 'getters' esta localizado en los getters de este modulo
      // Se puede utilizar rootGetters como cuarto argumento de los getters
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
        rootGetters['bar/someOtherGetter'] // -> 'bar/someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 'dispatch' y 'commit' tambien estan localizados para este modulo, y aceptaran la opcion 'root' para los dispatch/commit de la raiz
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'
        rootGetters['bar/someGetter'] // -> 'bar/someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```

#### Registrar acciones globales en modulos con namespace

Si se desea registrar acciones globales en modulos con namespace, se puede marcar este como `root: true` y ubicar la definicion de la accion a un handler de funcion.

Por ejemplo:

```javascript
{
  actions: {
    someOtherAction({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,
      actions: {
        someAction: {
          root: true,
          handler(namespacedContext, payload) {...} // 'someAction'
        }
      }
    }
  }
}
```

#### Vinculando helpers con el Namespace

Cuando vinculamos un modulo con namespace a componentes con los helpers `mapState`, `mapGetters`, `mapActions`y `mapMutations`, esto puede volverse un poco verborragico:

```javascript
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  }),
  ...mapGetters([
    'some/nested/module/someGetter', // -> this['some/nested/module/someGetter']
    'some/nested/module/someOtherGetter', // -> this['some/nested/module/someOtherGetter']
  ])
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}
```

En estos casos, se puede pasar el namespace del modulo como un string como primer argumento, para ayudar a los helpers para que todas las vinculaciones se realicen utilizando ese modulo como contexto.

Lo de arriba puede simplificarse de la siguiente manera:

```javascript
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  }),
  ...mapGetters('some/nested/module', [
    'someGetter', // -> this.someGetter
    'someOtherGetter', // -> this.someOtherGetter
  ])
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```

Aun mas, se pueden crear helpers con namespacing utilizando `createNamespacedHelpers`. Esto retorna un objeto con helpers vinculados al nuevo componente que estan vinculados con el valor del namespace dado:

```javascript
import { createNamespaceHelpers } from "vuex";

const { mapState, mapActions } = createNamespacedHelpers("some/nested/module");

export default {
  computed: {
    // busca en 'some/nested/module'
    ...mapState({
      a: (state) => state.a,
      b: (state) => state.b,
    }),
  },
  methods: {
    // busca en 'some/nested/module'
    ...mapActions(["foo", "bar"]),
  },
};
```

#### Advertencia para desarrolladores de Plugins

Si desarrollas plugins, quizas te pueda preocupar la impredecibilidad del namespacing de tus modulos cuando creas un plugin que provee modulos y deja a los usuarios agregarlos a su store de Vuex.
Tus modulos tambien seran namespaced si el usuario del plugin agrega tus modulos bajo un modulo con namespacing.
Para adaptar esta situacion, necesitaras recibir un valor de namespace a traves de la opcion de tu plugin:

```javascript
// Trae el valor del namespace a traves de la opcion del plugin y retorna una funcion de plugin de Vuex
export function createPlugin(options = {}) {
  return function (store) {
    // Agrega el namespace a los tipos del modulo del plugin
    const namespace = options.namespace || "";
    store.dispatch(namespace + "pluginAction");
  };
}
```

### Registro dinamico del modulo

Se puede registrar el modulo luego de que el store haya sido creado con el metodo `store.registerModule`:

```javascript
import { createStore } from "vuex";

const store = createStore({
  /* Opciones */
});

// registra el modulo a 'myModule'
store.registerModule("myModule", {
  //...
});

// registra un modulo anidado 'nested/myModule'
store.registerModule(["nested", "myModule"]),
  {
    //...
  };
```

El estado del modulo se expondra como `store.state.myModule`y `store.state.nested.MyModule`.

El registro dinamico de modulos hace posible que otros plugins de Vue puedan tomar ventaja para la gestion de estado de Vuex, mediante vincular el modulo al store de la aplicacion.

Por ejemplo, la libreria `vuex-router-sync` integra `vue-router` con Vuex para manejar el estado de rutas de la aplicacion en un modulo vinculado dinamicamente.

Tambien se puede remover de manera dinamica el modulo registrado con `store.unregisterModule(moduleName)`.
Notese que se puede chequear si el modulo ya esta registrado al store o no, a traves del metodo `store.hasModule(moduleName)`. Una cosa para tener en mente es que los modulos anidados se deben pasar como arrays, tanto para `registerModule` como para `hasModule` y no como un string con el path al modulo.

#### Preservando el estado

Puede ser posible que se necesite preservar el estado previo cuando se registra un nuevo modulo, como puede ser preservar el estado de una app renderizada del lado del servidor.

Esto se puede hacer con la opcion `preserveState`:

```javascript
store.registerModule("a", module, { preserveState: true });
```

cuando se configura `preserveState: true`, el modulo se registra, las acciones, mutaciones y los getters se agregan al store, pero no el estado.

Se asume que tu estado del store ya contiene el estado para ese modulo y no se necesita sobreescribirlo.

### Reutilizacion del modulo

A veces se necesita crear multiples instancias de un modulo, como por ejemplo:

- Crear multiples stores que utilizan el mismo modulo (Por ejemplo, para evitar componentes solitarios con estado en el SSR cuando la opcion `runInNewContext` es `false` o `unica`)
- Registrar el mismo modulo multiples veces en el mismo store

Si usamos un objeto simple para declarar el estado del modulo, entonces el objeto de estado sera compartido por referencia y causara contaminacion cruzada en el estado del modulo y del store cuando mute.

Este es el mismo problema con `data` dentro de los componentes de Vue, asi que la solucion es la misma: usar una funcion para declarar el estado del modulo (soportado en versiones superiores a 2.3.0):

```javascript
const MyReusableModule = {
  state: () => ({
    foo: "bar",
  }),
  // mutaciones, acciones, getters
};
```

## Informacion Avanzada

- Estructura de la Aplicacion: https://vuex.vuejs.org/guide/structure

- Composition API: https://vuex.vuejs.org/guide/composition-api

- Plugins: https://vuex.vuejs.org/guide/plugins

- Modo Estricto: https://vuex.vuejs.org/guide/strict

- Manejo de Formularios: https://vuex.vuejs.org/guide/forms

- Testing: https://vuex.vuejs.org/guide/testing

- Recarga Caliente (Hot Reloading): https://vuex.vuejs.org/guide/hot-reload

- Soporte de TypeScript: https://vuex.vuejs.org/guide/typescript-support

- Guia de Migracion a 4.0 desde 3.x: https://vuex.vuejs.org/guide/migrating-to-4-0-from-3-x
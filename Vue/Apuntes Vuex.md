# Vue Router 3 (para Vue 2)

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

### Sread Operator de Objeto

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
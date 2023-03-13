# Vue 3: Documentacion Tutorial

## 1. Renderizado Declarativo

La funcionalidad principal de Vue es el Renderizado Declarativo.

A partir de una sintaxis que extiende HTML, se describe como deberia verse el HTML, basado en un estado de Javascript.

Cuando el estado cambia, el HTML se actualiza automaticamente.

Los estados que pueden disparar actualizaciones cuando estos se modifican se consideran Reactivos. En Vue, el estado reactivo se encuentra en los componentes.

Podemos declarar un estado reactivo usando la opcion `data`del componente, que deberia ser una funcion que retorna un objeto:

```javascript
export defatul {
    data() {
        return {
            message: "Hello World"
        }
    }
}
```

La propiedad `message` estará disponible en la plantilla.

Esta es la forma en la que podemos renderizar texto dinamico basado en el valor de `message`, utilizando la sintaxis mustache ({{}})

```html
<h1>{{ message }}</h1>
```

El contenido dentro de los mustaches no esta limitado solo a identificadores o direcciones (paths), sino que tambien podemos usar cualquier expresion valida de Javascript:

```javascript
<h1>{{ message.split('').reverse().join('') }}</h1>
```

## 2. Vinculaciones de atributos

En Vue, los mustaches solo se utilizan para la interpolacion de texto. Para vincular un atributo a un valor dinamico, usamos la directiva `v-bind`:

```html
<div v-bind:id="dynamicId"></div>
```

> Una directiva es un atributo especial que comienza con el prefijo `v-`. Son parte de la sintaxis de plantilla de Vue

De manera similar a las interpolaciones de texto, los valores de las directivas son expresiones de Javascript que tienen acceso al estado del componente.

> Detalles completos de `v-bind` y la sintaxis de directivas se puede encontrar en https://vuejs.org/guide/essentials/template-syntax.html

La parte luego de los dos puntos (`:id`) es el "argumento" de la directiva. Aqui, el atributo `id` del elemento se sincronizará con la propiedad `dynamicId` del estado del componente.

Dado que `v-bind` se utiliza tan frecuentemente, tiene una sintaxis acotada:

```html
<div :id="dynamicId"></div>
```

## 3. Event Listeners y Methods

Podemos escuchar al DOM utilizando la directiva `v-on`

```html
<button v-on:click="increment">{{ count }}</button>
```

Debido al uso frecuente de `v-on`, tambien tiene una sintaxis acotada:

```html
<button @click="increment">{{ count }}</button>
```

Aquí `increment`hace referencia a una funcion declarada utilizando la opcion `methods`:

```javascript
export default {
  data() {
    return {
      count: 0,
    };
  },
  methods: {
    increment() {
      this.count++;
    },
  },
};
```

Dentro de un metodo podemos acceder a la instancia del componente utilizando `this`. La instancia del componente expone las propiedades data declaradas por `data`.

Podemos actualizar el estado del componente mutando estas propiedades.

Los Event handlers tambien pueden utilizar expresiones inline, y pueden simplificar tareas comunes con modificadores.

> Mas información sobre Event Handlers en https://vuejs.org/guide/essentials/event-handling.html

## 4. Vinculacion de Formularios

Utilizando `v-bind`y `v-on` juntos, podemos crear una vinculacion en dos sentidos en elementos de input de formularios:

```html
<input value="text" @input="onInput" placeholder="Type Here" />
```

```javascript
export default {
  data() {
    return {
      text: "",
    };
  },
  methods: {
    onInput(e) {
      this.text = e.target.value;
    },
  },
};
```

Para simplificar las vinculaciones en dos direcciones, Vue provee una directiva `v-model` que es esencialmente azucar sintactica para lo que vimos arriba:

```html
<input v-model="text" />
```

`v-model` sincroniza automaticamente el valor de `<input>` con el estado vinculado, asi no necesitamos utilizar un event handler para eso.

`v-model` no solo funciona para inputs de texto, sino tambien para otros tipos de inputs, como checkboxes, botones radiales y dropdowns de seleccion.

> Para mas detalles sobre el funcionamiendo del `v-model` o de las vinculaciones de Formulario, ver https://vuejs.org/guide/essentials/forms.html

## 5. Renderizado Condicional

Se puede utilizar la directiva `v-if`para hacer un renderizado condicional de un elemento:

```html
<h1 v-if="awesome">Vue is awesome!</h1>
```

Este `<h1>` se rendizará solo si el valor `awesome` es verdadero. Si `awesome` cambia a falso, se removerá del DOM.

Tambien podemos usar directivas `v-else` y `v-else-if` para denotar otras ramas de la condicion.

```html
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh, no 😢</h1>
```

> Mas informacion sobre el renderizado condicional en https://vuejs.org/guide/essentials/conditional.html

```javascript
<script>
export default {
  data() {
    return {
      awesome: true
    }
  },
  methods: {
    toggle() {
      this.awesome ? this.awesome = false : this.awesome = true
    }
  }
}
</script>

<template>
  <button @click="toggle">toggle</button>
  <h1 v-if="awesome">Vue is awesome!</h1>
  <h1 v-else>Oh no 😢</h1>
</template>
```

## 6. Renderizado de Listas

Podemos utilizar la directiva `v-for` para renderizar una lista de elementos basada en un array:

```html
<ul>
  <li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
</ul>
```

Aqui `todo` es una variable local que representa el elemento del array que esta siendo iterado actualmente. Solo es accesible en o dentro del elemento `v-for`, similar a un scope de funcion.

Tambien se le esta dando a cada objeto `todo` un unico `id`, y se esta vinculando como el atributo especial `key` para cada `<li>`.
El atributo `key` permite que Vue pueda mover de manera precisa cada `<li>` para que se relacione con la posicion de su objeto correspondiente en el array.

Hay dos formas de actualizar la lista:

1. Llamando metodos mutables en el array de origen:

```javascript
this.todos.push(newTodo);
```

2. Reemplazando el array con uno nuevo:

```javascript
this.todos = this.todos.filter(/*logica del filter*/);
```

> Los metodos mutables son aquellos que generan un cambio en el array de origen, en lugar de generar una copia del mismo.

> Para mas informacion sobre el atributo especial `key`: https://vuejs.org/api/built-in-special-attributes.html#key

> Para mas información sobre la directiva `v-for` y el renderizado de listas: https://vuejs.org/guide/essentials/list.html

## 7. Propiedad Computada

Las propiedades computadas son propiedades que se computan de manera reactiva a partir de otras propiedades, utilizando la opcion de `computed`

```javascript
export default {
  //...
  computed: {
    filteredTodos() {
      // retorna los todos filtrados en base a `this.hideCompleted`
    },
  },
};
```

Una propiedad computada escucha otro estado reactivo utilizado en su computacion como dependencias. Cachea el resultado y lo actualiza automaticamente cuando su dependencia cambia. (Esto es similar a cuando en React se utiliza un useEffect y se asigna una dependencia para escuchar y volver a dispararse llegado el caso de que esta dependencia sufra cambios)

> Mas información sobre propiedades computadas en: https://vuejs.org/guide/essentials/computed.html

Ej: Lista Todo donde los elementos completados se pueden ocultar

```javascript
let id = 0

export default {
  data() {
    return {
      newTodo: '',
      hideCompleted: false,
      todos: [
        { id: id++, text: 'Learn HTML', done: true },
        { id: id++, text: 'Learn JavaScript', done: true },
        { id: id++, text: 'Learn Vue', done: false }
      ]
    }
  },
  computed: {
    filteredTodos() {
      return this.hideCompleted
        ? this.todos.filter((t) => !t.done)
        : this.todos
    }
  },
  methods: {
    addTodo() {
      this.todos.push({ id: id++, text: this.newTodo, done: false })
      this.newTodo = ''
    },
    removeTodo(todo) {
      this.todos = this.todos.filter((t) => t !== todo)
    }
  }
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style>
.done {
  text-decoration: line-through;
}
</style>
```

## 8. Ciclo de Vida y Plantillas de Ref

Hasta ahora, Vue ha estado manejando todas las actualizaciones del DOM por nosotros, gracias a la reactividad y el renderizado declarativo.
Sin embargo, inevitablemente habran casos donde necesitemos trabajar manualmente con el DOM.

Podemos solicitar una plantilla Ref (por ejemplo, una referencia a un elemento en la plantilla), utilizando el atributo especial `ref`:

```html
<p ref="p">Hello</p>
```

El elemento sera expuesto en `this.$refs` como `this.$refs.p`.

> Para mas informacion sobre el atributo especial `ref`, visita https://vuejs.org/api/built-in-special-attributes.html#ref

De todas maneras, solo se podrá acceder a el una vez que el componente este montado.

Para correr codigo luego del montaje, podemos utilizar la opcion `mounted`

```javascript
export default {
  mounted() {
    // component is now mounted
  },
};
```

Este es un hook de Estado de Vida, que nos permite registrar una callback para ser llamada en una etapa determinada del ciclo de vida del componente. Hay otros hooks como `created`o `updated`

> Para mas detalles, puedes ver este diagrama: https://vuejs.org/guide/essentials/lifecycle.html#lifecycle-diagram

Ej:

```javascript
<script>
export default {
  mounted() {
    this.$refs.p.textContent = "Hola"
  }
}
</script>

<template>
  <p ref="p">hello</p>
</template>
```

## 9. Watchers

A veces necesitamos realizar "efectos secundarios" de manera reactiva, como por ejemplo, loguear un numero en la consola cuando este cambia.
Para lograr esto podemos utilizar la opcion watchers:

```javascript
export default {
  data() {
    return {
      count: 0,
    };
  },
  watch: {
    count(newCount) {
      console.log(`new count is: ${newCount}`);
    },
  },
};
```

Aqui estamos usando la opcion `watch` para que vigile los cambios en la propiedad `count`. El callback de `watch` se llama cuando `count` cambia, y recibe el nuevo valor como argumento.

Los watchers pueden recibir dos argumentos, siendo el primero de ellos el nuevo valor de la propiedad observada, y el segundo, el valor previo (util para comparaciones)

Ej:

```javascript
watch: {
  count(newCount, oldCount) {
    // Logica que se quiera realizar
  }
}
```

> Para mas informacion sobre watchers, ver https://vuejs.org/guide/essentials/watchers.html

Ejemplo del uso de Watchers para iterar datos:

```javascript
<script>
export default {
  data() {
    return {
      todoId: 1,
      todoData: null
    }
  },
  methods: {
    async fetchData() {
      this.todoData = null
      const res = await fetch(
        `https://jsonplaceholder.typicode.com/todos/${this.todoId}`
      )
      this.todoData = await res.json()
    }
  },
  mounted() {
    this.fetchData()
  },
  watch: {
    todoId(id) {
      this.fetchData()
    }
  }
}
</script>

<template>
  <p>Todo id: {{ todoId }}</p>
  <button @click="todoId++">Fetch next todo</button>
  <p v-if="!todoData">Loading...</p>
  <pre v-else>{{ todoData }}</pre>
</template>
```

## 10. Componentes

Hasta ahora estuvimos viendo situaciones de un unico componente, pero las aplicaciones reales de Vue tipicamente se crean con componentes anidados.

Un componente padre puede renderizar otro componente en su template, como un componente hijo. Para utilizar un componente hijo, primero necesitamos importarlo:

```javascript
import ChildComp from "./ChildComp.vue";

export default {
  components: {
    ChildComp,
  },
};
```

Ademas de importarlo, necesitamos registrar el componente usando la opcion `components`. Aqui estamos usando la abreviacion de la propiedad del objeto para registrar el componente `ChildComp` bajo la key `ChildComp`.

Luego podemos usar el componente en la plantilla como:

```html
<ChildComp />
```

> Es importante hacer la importacion dentro de la etiqueta de `<script>`

Ejemplo de importacion:

```javascript
<script>
  import ChildComp from "./ChildComp.vue"

export default {
  components: {
    ChildComp
  }
}
</script>

<template>
  <ChildComp />
</template>
```

## 11. Props

Un componente hijo puede aceptar inputs desde los componentes padres via `props`. Primero necesitamos declarar las `props` que este aceptará:

```javascript
// En el componente hijo
export default {
  props: {
    msg: String,
  },
};
```

Una vez declarado, la prop `msg` queda expuesta al `this` y puede ser utilizada en la template del componente hijo.

El componente padre puede pasar la prop al hijo como atributos. Para pasar un valor dinamico, tambien podemos utilizar la sintaxis de `v-bind`:

```javascript
<ChildComp :msg="greeting" />
```

Ejemplo completo:

```javascript
// Componente Padre
<script>
import ChildComp from './ChildComp.vue'

export default {
  components: {
    ChildComp
  },
  data() {
    return {
      greeting: 'Hello from parent'
    }
  }
}
</script>

<template>
  <ChildComp :msg="greeting" />
</template>

// Componente hijo
<script>
export default {
  props: {
    msg: String
  }
}
</script>

<template>
  <h2>{{ msg || 'No props passed yet' }}</h2>
</template>
```

## 12. Emits

En adicion a recibir props, un componente tambien puede emitir eventos al padre:

```javascript
export default {
  // Se declaran los eventos a emitir
  emits: ["response"],
  created() {
    // emite con un argumento
    this.$emit("response", "hello from child");
  },
};
```

El primer argumento de `this.$emit()` es el nombre del evento. Cualquier argumento adicional se pasa al event listener.

El componente padre puede escuchar los eventos emitidos por el componente hijo a traves del uso de `v-on`. Aqui el handler recibe el argumento extra de la llamada a la emision del hijo y lo asigna al estado local:

```javascript
<ChildComp @response="(msg) => childMsg=msg" />
```

Ejemplo completo:

```javascript
// Componente padre
<script>
import ChildComp from './ChildComp.vue'

export default {
  components: {
    ChildComp
  },
  data() {
    return {
      childMsg: 'No child msg yet'
    }
  }
}
</script>

<template>
  <ChildComp @response="(msg) => childMsg = msg" />
  <p>{{ childMsg }}</p>
</template>

// Componente hijo

<script>
export default {
  emits: ['response'],
  created() {
    this.$emit('response', 'hello from child')
  }
}
</script>

<template>
  <h2>Child component</h2>
</template>
```

## 13. Slots

En adicion a pasar informacion via props, los componentes padres pueden pasar tambien fragmentos de template a los hijos mediante `slots`:

```javascript
<ChildComp>Esto es algo de contenido de slot!</ChildComp>
```

En el componente hijo, se puede renderizar el contenido del slot del padre utilizando el elemento <slot> como salida:

```javascript
// En la template del componente hijo
<slot />
```

El contenido dentro de la salida de `<slot>` sera tratado como contenido "alternativo" y se mostrara solo si el componente padre no pasa ningun contenido en el slot:

```html
<slot>Contenido Alternativo< /slot></slot>
```

Ejemplo completo:

```javascript
// Componente Padre
<script>
import ChildComp from './ChildComp.vue'

export default {
  components: {
    ChildComp
  },
  data() {
    return {
      msg: 'from parent'
    }
  }
}
</script>

<template>
  <ChildComp>{{msg}}</ChildComp>
</template>

// Componente Hijo

<template>
  <slot>Contenido Alternativo</slot>
</template>

```

## **Mas informacion**

Crear un proyecto de Vue:
https://vuejs.org/guide/quick-start.html

Revisar todos los temas tratados en mayor detalle:
https://vuejs.org/guide/essentials/application.html

Ejemplos Practicos:
https://vuejs.org/examples/

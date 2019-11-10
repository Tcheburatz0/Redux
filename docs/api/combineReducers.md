# `combineReducers(reducers)`

Как только ваше приложение становится все более сложным, вы захотите разделить ваш [функцию редюсер](../Glossary.md#reducer) на отдельные функции, которые управляют независимыми частями [состояния](../Glossary.md#state).

Вспомогательная функция `combineReducers` преобразует объект, значениями которого являются различные функции редюсеры, в одну функцию редюсер, которую можно передать в метод [`createStore`](createStore.md).

Результирующий редюсер вызывает вложенные редюсеры и собирает их результаты в единый объект состояния. 
**Состояние, созданное именами `combineReducers()`, сохраняет состояние каждого редуктора под их ключами, переданные в `combineReducers()`**

Пример:
```
rootReducer = combineReducers({potato: potatoReducer, tomato: tomatoReducer})
// Это создаст следующий объект состояния
{
  potato: {
    // ... potatoes и другое состояние управляемое potatoReducer ... 
  },
  tomato: {
    // ... tomatoes и другое состояние управляемое tomatoReducer, возможно, какой-нибудь хороший соус? ...
  }
}
```

Вы можете управлять именами ключей состояний, используя разные ключи для редюсеров в переданном объекте. Например, вы можете называть `combineReducers({ todos: myTodosReducer, counter: myCounterReducer })` для формы состояния как `{todos, counter}`.

Популярное соглашение - назвать редюсеры после разделения состояний, которыми они управляют, поэтому вы можете использовать сокращенную обозначение свойства ES6: `combReducers ({counter, todos})`. Это эквивалентно написанию `combReducers ({counter: counter, todos: todos})`.


> ##### Примечания для пользователей Flux
> 
> Эта функция поможет вам организовать ваши редюсеры для управления их собственными частями состояния, подобно тому, как вы бы имели различные Flux хранилища для управления разными состояниями. С Redux у вас есть только одно хранилище, но `combineReducers` помогает вам сохранять такое же логическое разделение между редюсерами.

#### Параметры

  1. `reducers` (*Object*): объект, значения которого соответствуют различным функциям редюсерам, которые должны быть объединены в один. Ниже идут примечания для некоторых правил, которым должен следовать каждый переданный редюсер.

> Ранее документация предлагала использовать ES6-синтаксис `import * as reducers` для получения объекта редюсеров. Это было источником многочисленной путаницы, поэтому сейчас рекомендуется экспортировать один редюсер, полученный с помощью `combineReducers()` из `reducers/index.js` вместо этого. Ниже приведен пример.

#### Возвращает

(* Function *): редюсер, который вызывает каждый редюсер внутри объекта `reducers` и создает объект состояния с той же формой.

#### Примечания

Эта функция слегка самоуверенная и искажена, чтобы помочь новичкам избежать общих ошибок. Именно поэтому она пытается применять некоторые правила, которым не нужно следовать, если вы пишете корневой редюсер вручную.

Любой редюсер передаваемый `combineReducers` должен соответствовать этим правилам:

  * Для любых действий, которые не определены, он должен возвращать `state`, переданный ему в качестве первого аргумента.

  * Он никогда не должен возвращать ` undefined `. Это очень легко сделать по ошибке через предыдущие `return`, поэтому `combineReducers` создает исключение, если вы сделали это, предотвращая появление ошибки где-нибудь еще.

  * Если `state` переданный ему ` не определен (undefined)`, то он должен возвратить начальное состояние (state) для этого конкретного редюсера. Согласно предыдущему правилу, начальное состояние (state) не должно быть равно `undefined`. Это удобно указывать с ES6-синтаксисом опциональных аргументов, но вы можете также явно проверить первый аргумент на `undefined`.

В то время как `combineReducers` пытается проверить, что ваши редюсеры соответствуют некоторым из этих правил, вам следует помнить о них и сделать все возможное, чтобы следовать им. `combineReducers` проверит ваши редюсеры, передав им` undefined`; это делается, даже если вы указываете начальное состояние на `Redux.createStore(combineReducers(...), initialState)`. Поэтому вы **должны** гарантировать, что ваши редюсеры будут работать должным образом при получении `undefined` в качестве state, даже если вы никогда не намереваетесь фактически получить` undefined` в своем собственном коде.

#### Example

#### `reducers/todos.js`

#### `reducers/todos.js`

```js
export default function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}
```

#### `reducers/counter.js`

```js
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

#### `reducers/index.js`

```js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
```

#### `App.js`

```js
import { createStore } from 'redux'
import reducer from './reducers/index'

const store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
```

#### Советы

  * Этот помощник – это всего лишь удобство! Вы можете написать свой собственный `combineReducers` который [работает иначе](https://github.com/acdlite/reduce-reducers) или даже вручную собрать объект состояния из вложенного редюсера и написать родительскую функцию-редюсер явно, как можно было бы написать любую другую функцию.

  * Вы можете вызвать `combineReducers` на любом уровне иерархии редюсера. Это не обязательно должно произойти наверху. На самом деле, вы можете использовать его снова, чтобы разделить "детские" редюсеры, которые получаются слишком сложными, на независимых "внуков" и так далее.
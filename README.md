# Ducks: Redux Reducer Bundles

<img src="duck.jpg" align="right"/>

При написании redux приложений я замечаю, что работая над отдельными частями своего функционала, мне каждый раз приходится добавлять `{actionTypes, actions, reducer}`. Я храню их в разных файлах и даже в разных директориях, однако, в 95% процентах случаев требуется лишь одна пара reducer/actions для их связанных действий.

Как по мне, логичнее объединять эти части в самодостаточные изолированные модули, которые с легкостью можно даже упаковать в библиотеку.

## Предложение

### Пример

Смотрите также: [Общий JS Пример](CommonJs.md).

```javascript
// widgets.js

// Actions
const LOAD   = 'my-app/widgets/LOAD';
const CREATE = 'my-app/widgets/CREATE';
const UPDATE = 'my-app/widgets/UPDATE';
const REMOVE = 'my-app/widgets/REMOVE';

// Reducer
export default function reducer(state = {}, action = {}) {
  switch (action.type) {
    // do reducer stuff
    default: return state;
  }
}

// Action Creators
export function loadWidgets() {
  return { type: LOAD };
}

export function createWidget(widget) {
  return { type: CREATE, widget };
}

export function updateWidget(widget) {
  return { type: UPDATE, widget };
}

export function removeWidget(widget) {
  return { type: REMOVE, widget };
}

// сайд-эффекты, только подходящие
// например, thunks, epics, и т.д.
export function getWidget () {
  return dispatch => get('/widget').then(widget => dispatch(updateWidget(widget)))
}

```
### Правила

Модуль...

1. ДОЛЖЕН делать `export default` функции `reducer()`
2. ДОЛЖЕН делать `export` action creators как функций
3. ДОЛЖЕН иметь action types в виде `npm-module-or-app/reducer/ACTION_TYPE`
3. МОЖЕТ экспортировать action types как `UPPER_SNAKE_CASE`, если внешнему reducer нужно следить за событиями этих типов или если это опубликованная переиспользуемая библиотека.

Те же правила рекомендуемы для `{actionType, action, reducer}` пакетов, что используются как переиспользуемые Redux библиотеки.

### Название

У Java есть jars и beans. У Ruby есть gems. Я предлагаю называть эти reducer пакеты "ducks" (от англ. "утки"), как последний слог в слове "redux".

### Использование

Вы все еще можете:

```javascript
import { combineReducers } from 'redux';
import * as reducers from './ducks/index';

const rootReducer = combineReducers(reducers);
export default rootReducer;
```

Вы все еще можете:

```javascript
import * as widgetActions from './ducks/widgets';
```
...и это будет делать import только функций action creators, чтобы быть переданными функции `bindActionCreators()`.

> Вообще говоря, это выражение также сделает import `default`, результатом которого станет функция reducer. Это добавит функцию action creator, названную `default`, которая не будет работать. Если это проблема, вам следует перечислять каждую функцию action creator во время импортирования.

Будет время, когда вам захочется экспортировать что-то помимо функций action creators. Это также нормально. Правила не говорят, что вы можете делать *исключительно*  `export` функций action creators. Когда вам понадобится делать `export` функций action creators, вам всего лишь нужно перечислить те из них, что вам нужны. Ничего серьезного.

```javascript
import {loadWidgets, createWidget, updateWidget, removeWidget} from './ducks/widgets';
// ...
bindActionCreators({loadWidgets, createWidget, updateWidget, removeWidget}, dispatch);
```

### Пример

[React Redux Универсальный Горячий Пример,](https://github.com/erikras/react-redux-universal-hot-example) использующий ducks. Смотрите [`/src/redux/modules`](https://github.com/erikras/react-redux-universal-hot-example/tree/master/src/redux/modules).

[Todomvc, использующий ducks.](https://github.com/goopscoop/ga-react-tutorial/tree/6-reduxActionsAndReducers)

### BattleCry генераторы

Существуют конфигурируемые [BattleCry](https://github.com/pedsmoreira/battlecry) генераторы, готовые к скачиванию и оказанию помощи для написания ducks:

```sh
npm install -g battlecry
cry download generator erikras/ducks-modular-redux
cry init duck
```

Запустите `cry --help`, чтобы получить больше информации о возможных генераторах;

### Реализация

Миграция на такую структура кода была [безболезненной](https://github.com/erikras/react-redux-universal-hot-example/commit/3fdf194683abb7c40f3cb7969fd1f8aa6a4f9c57), и я предвижу, что в будущем это уменьшит проблемы при разработке.

Хотя реализовать ducks возможно без какой-либо внешней библиотеки, существует несколько инструментов, что могут вам помочь:

 * [extensible-duck](https://github.com/investtools/extensible-duck) - Реализация Ducks. С этой библиотекой вы можете создать переиспользуемые и расширяемые ducks.
 * [saga-duck](https://github.com/cyrilluce/saga-duck) - Реализация Ducks в Typescript с [sagas](https://github.com/redux-saga/redux-saga). Результаты в переиспользуемых и расширяемых ducks.
 * [redux-duck](https://github.com/PlatziDev/redux-duck) - Функция-помощник для создания Redux-модулей, использующих ducks-modular-redux принцип.
 * [modular-redux-thunk](https://github.com/benbeadle/modular-redux-thunk) - Вдохновленный принципом ducks пакет для помощи в организации actions, функций reducers, и селекторов вместе - со встроенной redux-thunk поддержкой для асинхронных actions.
 * [molecular-js](https://www.npmjs.com/package/molecular-js) - Набор утилит, чтобы облегчить разработку шаблонов модульного управления состоянием (modular state management) с библиотекой Redux (также изв. как ducks).
 * [ducks-reducer](https://github.com/drpicox/ducks-reducer) - Функция для объединения _ducks object_ функций-reducers в одну функцию-reducer (эквивалентной функции [combineReducers](https://redux.js.org/docs/api/combineReducers.html)), и функции [ducks-middleware](https://github.com/drpicox/ducks-middleware) для объединения _ducks object_ middleware в одну middleware, совместимую с [applyMiddleware](https://redux.js.org/docs/api/applyMiddleware.html).
 * [simple-duck](https://github.com/xander27/simple-duck) - Реализация модульной системы, основанная на классах, вдохновленная ducks-modular-redux. Включает такие преимущества ООП, как наследование и композиция. Поддерживает объединение классов duck-модулей и обычных функций-reducer используя `combineModules` функцию.

Пожалуйста, оставьте обратную связь через issue или твит на [@erikras](https://twitter.com/erikras). Буду крайне признателен.

Счастливого кодинга!

-- Erik Rasmussen


### Перевод

[한국어](https://github.com/JisuPark/ducks-modular-redux)
[中文](https://github.com/deadivan/ducks-modular-redux)
[Türkçe](https://github.com/mfyz/ducks-modular-redux-tr)
[Русский](https://github.com/DnEgorWeb/ducks-modular-redux)

---

![Вперед! Мигрируем все наши reducers!](migrate.jpg)
> Фото [Airwolfhound](https://www.flickr.com/photos/24874528@N04/3453886876/).

---

[![Beerpay](https://beerpay.io/erikras/ducks-modular-redux/badge.svg?style=beer-square)](https://beerpay.io/erikras/ducks-modular-redux)  [![Beerpay](https://beerpay.io/erikras/ducks-modular-redux/make-wish.svg?style=flat-square)](https://beerpay.io/erikras/ducks-modular-redux?focus=wish)

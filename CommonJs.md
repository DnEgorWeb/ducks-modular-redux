## Common JS Example

```javascript
// widgets.js

const LOAD   = 'my-app/widgets/LOAD';
const CREATE = 'my-app/widgets/CREATE';
const UPDATE = 'my-app/widgets/UPDATE';
const REMOVE = 'my-app/widgets/REMOVE';

function reducer(state = {}, action = {}) {
  switch (action.type) {
    // do reducer stuff
    default: return state;
  }
}

reducer.loadWidgets = function() {
  return { type: LOAD };
}

reducer.createWidget = function(widget) {
  return { type: CREATE, widget };
}

reducer.updateWidget = function(widget) {
  return { type: UPDATE, widget };
}

reducer.removeWidget = function(widget) {
  return { type: REMOVE, widget };
}

module.exports = reducer;
```

Есть одна особенность: вы не можете использовать функцию Redux `bindActionCreators()` прямо с модулем duck, так как это предполагает, что функция должна быть одиночной функцией action creator, поэтому вам нужно делать что-то вроде:

```javascript
var actionCreators = require('./ducks/widgets');
bindActionCreators({ ...actionCreators });
```

Другая особенность заключается в том, что если вы также экспортируете константы типов, вам необходимо также присоединить их к reducer функциям, поэтому вы не можете так просто распаковать только функции action creators в другой объект во время импорта (никакого синтаксиса `as`), поэтому трюк выше не сработает.

Вы можете избежать и того, и другого, развернув вашу собственную dispatch binding функцию - это то, что я использую для создания функции для передачи как `mapDispatchToProps` аргумента функции `connect()`:

```javascript
/**
 * Создает функцию, которая создает аналогично названные action dispatchers из объекта,
 * чьи свойства функции - это action creators. Любые не-функции в actionCreators игнорируются.
 */
var createActionDispatchers = actionCreators => dispatch =>
  Object.keys(actionCreators).reduce((actionDispatchers, name) => {
    var actionCreator = actionCreators[name];
    if (typeof actionCreator == 'function') {
      actionDispatchers[name] = (...args) => dispatch(actionCreator(...args));
    }
    return actionDispatchers;
  }, {})

var actionCreators = require('./ducks/widgets');
var mapStateToProps = state => state.widgets;
var mapDispatchToProps = createActionDispatchers(actionCreators);

var MyComponent = React.createClass({ /* ... */ });

module.exports = connect(mapStateToProps , mapDispatchToProps)(MyComponent);
```

---
Этот документ скопирован практичесли дословно из issue [@insin](https://github.com/insin) [#2](https://github.com/erikras/ducks-modular-redux/issues/2).

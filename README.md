# Redux
![image](https://user-images.githubusercontent.com/44378669/78541042-b2e4ed00-77fd-11ea-91df-f21f4d72aa2f.png)
# source-code-analysis-redux-legabog
[Redux github repository](https://github.com/reduxjs/redux)
-----------
[Redux JS code in 100 lines](https://gist.github.com/gaearon/ffd88b0e4f00b22c3159)
-----------
`JS source code in 100 lines`
-----------
```js
function mapValues(obj, fn) { // функция mapValues принимает в качестве аргументов объект и функцию
  return Object.keys(obj).reduce((result, key) => { 
  // возвращает массив свойств объекта и с помощью reduce применяет callback функцию  
  // к кажому свойству, начальное значение пустой объект {}, в callback фунцию в  
  // качестве аргументов передаётся result и key, result от ключа равен функции fn(obj[key], key).  
    result[key] = fn(obj[key], key);
    return result;
  }, {});
}

function pick(obj, fn) { // функция pick принимает в качестве аргументов объект и функцию
  return Object.keys(obj).reduce((result, key) => {
  // возвращает массив свойств объекта и с помощью reduce применяет callback функцию к  
  // кажому свойству, начальное значение пустой объект {}, в callback фунцию в качестве  
  // аргументов передаётся result и key
    if (fn(obj[key])) {
 // если fn(obj[key]) === true выполняется:
      result[key] = obj[key];
    }
    return result;
  }, {});
}

function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args));
}

export function bindActionCreators(actionCreators, dispatch) { // фунция bindActionCreators получает в качестве аргументов actionCreators, dispatch
  return typeof actionCreators === 'function' ?
  // если actionCreators фунция то выполняется bindActionCreator
    bindActionCreator(actionCreators, dispatch) :
    mapValues(actionCreators, actionCreator =>
    // иначе применяется фунция mapValues 
      bindActionCreator(actionCreator, dispatch)
    // применяется bindAtionCreator к каждому actionCreator
    );
}

//-----Функциональное композиция-----
// Последование выполнение функциональной композиции с последовательной передачей результатов выполнения следующей функции:
 `(...args) => f(g(h(...args)))`
 // выполнится h(...args) -> g(с результатом выполнения h(...args) -> на последнем этапе выполнится f(с реультатом g()) 
 // обычно принято называть такую цепочку функциональной композицией (функцией compose):
 // выполняется с права налево compose(f <- g <- h), обратное выполнение слева направо  
 // называется конвеер pipeline, pipe compose(f -> g -> h)
 `compose(f, g, h)`
 
export function compose(...funcs) {
  return arg => funcs.reduceRight((composed, f) => f(composed), arg);
}



//-----Middleware-----
// Расширение функциональности redux, дополнительная прослойка
// applyMiddleware() - связывает store с middleware 
// основной принцип конвеер, первый middleare самостоятельно отрабатывает и запускает следующий middleware(1 -> 2 -> 3 -> ...)
// применяется концепция каррирования

export function applyMiddleware(...middlewares) {
  return (next) => (reducer, initialState) => {
    var store = next(reducer, initialState);
    var dispatch = store.dispatch;
    var chain = [];

    chain = middlewares.map(middleware => middleware({
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }));
    dispatch = compose(...chain)(store.dispatch);

    return { ...store, dispatch };
  };
}

// -----combineReducers-----
// функция объединяющая множество reducers, в отдельную абстрагированную часть

export function combineReducers(reducers) { // в конструкции данной функции используется ранее объявленная функция pick и mapValues
  var finalReducers = pick(reducers, (val) => typeof val === 'function');
  return (state = {}, action) => mapValues(finalReducers,
    (reducer, key) => reducer(state[key], action)
    // возвращает callback фунцию, прогоняет все значения reducers с помощью map, на выходе возвращается reducer
  );
}

-----createStore-----
// создание store и объявление внутри функций относящихся к store: getState(), subscribe(), dispatch(), replaceReducers()

export function createStore(reducer, initialState) {
  var currentReducer = reducer;
  var currentState = initialState;
  var listeners = [];
  var isDispatching = false;

  function getState() { // getState - возвращает текущий State
    return currentState;
  }

  function subscribe(listener) { subscribe - слушатель событий, подписываемся на изменения state
    listeners.push(listener);

    return function unsubscribe() { // в конце отписываемся unsubscribe()
      var index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  }

  function dispatch(action) { // dispatch - функция диспетчер
    if (isDispatching) { // если isDispatching === true 
      throw new Error('Reducers may not dispatch actions.'); // выкидываем ошибку 'Reducers may not dispatch actions.'
    }

    try {
      isDispatching = true; // в процессе dispatch выставляем свойство isDispatch в true
      currentState = currentReducer(currentState, action); // изменение state за счёт currentReducer
    } finally {
      isDispatching = false; // всегда в конечном итоге в конечной части конструкции выставляем в свойство isDispatching = false 
    }

    listeners.slice().forEach(listener => listener()); // делаем копию массива listeners для каждого элемента выполняем функцию слушателя
    return action;
  }

  function replaceReducer(nextReducer) { // функция перехода к следующему Reducer
    currentReducer = nextReducer;
    dispatch({ type: '@@redux/INIT' });
  }

  dispatch({ type: '@@redux/INIT' });

  return { dispatch, subscribe, getState, replaceReducer };
}
```
[разбор файловой иерархии формата TypeScript в redux/src](https://github.com/Legabog/source-code-analysis-redux-legabog/blob/redux/src/README.md)




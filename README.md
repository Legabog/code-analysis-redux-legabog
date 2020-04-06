# Redux
![image](https://user-images.githubusercontent.com/44378669/78541042-b2e4ed00-77fd-11ea-91df-f21f4d72aa2f.png)
# source-code-analysis-redux-legabog
[Redux github repository](https://github.com/reduxjs/redux)
-----------
[Redux JS code in 100 lines](https://gist.github.com/gaearon/ffd88b0e4f00b22c3159)
-----------
`JS source code in 100 lines`
-----------
```
function mapValues(obj, fn) { // функция mapValues принимает в качестве аргументов объект и функцию
  return Object.keys(obj).reduce((result, key) => { 
  // возвращает массив свойств объекта и с помощью reduce применяет callback функцию к кажому свойству, начальное значение пустой объект {}, в callback фунцию в качестве аргументов передаётся result и key, result от ключа равен функции fn(obj[key], key).  
    result[key] = fn(obj[key], key);
    return result;
  }, {});
}

function pick(obj, fn) { // функция pick принимает в качестве аргументов объект и функцию
  return Object.keys(obj).reduce((result, key) => {
  // возвращает массив свойств объекта и с помощью reduce применяет callback функцию к кажому свойству, начальное значение пустой объект {}, в callback фунцию в качестве аргументов передаётся result и key
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
 // выполняется с права налево compose(f <- g <- h), обратное выполнение слева направо называется конвеер pipeline, pipe compose(f -> g -> h)
 `compose(f, g, h)`
 
export function compose(...funcs) {
  return arg => funcs.reduceRight((composed, f) => f(composed), arg);
}



//-----Middleware-----
// Расширение функциональности redux, дополнительная прослойка
//applyMiddleware() - связывает store с middleware 
//основной принцип конвеер, первый middleare самостоятельно отрабатывает и запускает следующий middleware(1 -> 2 -> 3 -> ...)
//применяется концепция каррирования

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

//-----combineReducers-----
//функция объединяющая множество reducers, в отдельную абстрагированную часть

export function combineReducers(reducers) { // В конструкции данной функции используется ранее объявленная функция pick и mapValues
  var finalReducers = pick(reducers, (val) => typeof val === 'function');
  return (state = {}, action) => mapValues(finalReducers,
    (reducer, key) => reducer(state[key], action)
    // Возвращает callback фунцию, прогоняет все значения reducers с помощью map, на выходе возвращается reducer
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
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.');
    }

    try {
      isDispatching = true;
      currentState = currentReducer(currentState, action);
    } finally {
      isDispatching = false;
    }

    listeners.slice().forEach(listener => listener());
    return action;
  }

  function replaceReducer(nextReducer) {
    currentReducer = nextReducer;
    dispatch({ type: '@@redux/INIT' });
  }

  dispatch({ type: '@@redux/INIT' });

  return { dispatch, subscribe, getState, replaceReducer };
}
```
[разбор файловой иерархии формата TypeScript в redux/src](https://github.com/reduxjs/redux/tree/master/src)





redux/src/compose.ts
Функциональное композиция
// Последование выполнение функциональной композиции с последовательной передачей результатов выполнения следующей функции:
 `(...args) => f(g(h(...args)))`
 // выполнится h(...args) -> g(с результатом выполнения h(...args) -> на последнем этапе выполнится f(с реультатом g()) 
 // обычно принято называть такую цепочку функциональной композицией (функцией compose):
 // выполняется с права налево compose(f <- g <- h), обратное выполнение слева направо называется конвеер pipeline, pipe compose(f -> g -> h)
 `compose(f, g, h)`
 
 `Код основной фунции compose на TS`
 //структура функции реализована по правилу 0, 1, бесконечность ... 
 export default function compose(...funcs: Function[]) {  //функция compose принимает в качестве аргументов массив из функций
  if (funcs.length === 0) { // если передано 0 фунций возвращается фунция с arg
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args: any) => a(b(...args))) //выполняется фунциональная композиция  
}
 
  `Код основной фунции compose на JS`
 export default function compose(...funcs) {
  if (funcs.length === 0) {
        return React.createElement(T, null, "(arg: T) => arg }");
    }
  if (funcs.length === 1) {
        return funcs[0];
    }
  return funcs.reduce((a, b) => (...args) => a(b(...args)));  
}

//----------------------------------------------

redux/src/applyMiddleware.ts

//Middleware - расширение функциональности redux, дополнительная прослойка

//applyMiddleware() - связывает store с middleware 
//основной принцип конвеер, первый middleare самостоятельно отрабатывает и запускает следующий middleware(1 -> 2 -> 3 -> ...)
//применяется концепция каррирования

`Код на TS`
export default function applyMiddleware(): StoreEnhancer
export default function applyMiddleware<Ext1, S>(
  middleware1: Middleware<Ext1, S, any>
): StoreEnhancer<{ dispatch: Ext1 }>
export default function applyMiddleware<Ext1, Ext2, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 }>
export default function applyMiddleware<Ext1, Ext2, Ext3, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 }>
export default function applyMiddleware<Ext1, Ext2, Ext3, Ext4, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>,
  middleware4: Middleware<Ext4, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 & Ext4 }>
export default function applyMiddleware<Ext1, Ext2, Ext3, Ext4, Ext5, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>,
  middleware4: Middleware<Ext4, S, any>,
  middleware5: Middleware<Ext5, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 & Ext4 & Ext5 }>
export default function applyMiddleware<Ext, S = any>(
  ...middlewares: Middleware<any, S, any>[]
): StoreEnhancer<{ dispatch: Ext }>
export default function applyMiddleware(
  ...middlewares: Middleware[]
): StoreEnhancer<any> {
  return (createStore: StoreCreator) => <S, A extends AnyAction>(
    reducer: Reducer<S, A>,
    ...args: any[]
  ) => {
    const store = createStore(reducer, ...args)
    let dispatch: Dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI: MiddlewareAPI = {
      getState: store.getState,
      dispatch: (action, ...args) => dispatch(action, ...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose<typeof dispatch>(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}

`Код на JS`

export default function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, ...args) => {
        const store = createStore(reducer, ...args);
        let dispatch = () => {
            throw new Error('Dispatching while constructing your middleware is not allowed. ' +
                'Other middleware would not be applied to this dispatch.');
        };
        const middlewareAPI = {
            getState: store.getState,
            dispatch: (action, ...args) => dispatch(action, ...args)
        };
        const chain = middlewares.map(middleware => middleware(middlewareAPI));
        dispatch = compose(...chain)(store.dispatch);
        return Object.assign(Object.assign({}, store), { dispatch });
    };
}

redux/src/bindActionCreators.ts 

`Код на TS`

export default function bindActionCreators<A, C extends ActionCreator<A>>(
  actionCreator: C,
  dispatch: Dispatch
): C

export default function bindActionCreators<
  A extends ActionCreator<any>,
  B extends ActionCreator<any>
>(actionCreator: A, dispatch: Dispatch): B

export default function bindActionCreators<
  A,
  M extends ActionCreatorsMapObject<A>
>(actionCreators: M, dispatch: Dispatch): M
export default function bindActionCreators<
  M extends ActionCreatorsMapObject<any>,
  N extends ActionCreatorsMapObject<any>
>(actionCreators: M, dispatch: Dispatch): N

export default function bindActionCreators(
  actionCreators: ActionCreator<any> | ActionCreatorsMapObject,
  dispatch: Dispatch
) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  const boundActionCreators: ActionCreatorsMapObject = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

`Код на JS`

export default function bindActionCreators(actionCreators, dispatch) { 
    if (typeof actionCreators === 'function') {
        return bindActionCreator(actionCreators, dispatch);
    }
    if (typeof actionCreators !== 'object' || actionCreators === null) {
        throw new Error(`bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
            `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`);
    }
    const boundActionCreators = {};
    for (const key in actionCreators) {
        const actionCreator = actionCreators[key];
        if (typeof actionCreator === 'function') {
            boundActionCreators[key] = bindActionCreator(actionCreator, dispatch);
        }
    }
    return boundActionCreators;
}




# redux/src
[ссылка на репозиторий](https://github.com/reduxjs/redux/tree/master/src)
-----
`branches`
-----
[applyMidddleware.ts](https://github.com/Legabog/source-code-analysis-redux-legabog/blob/applyMiddleware.ts/README.md)
-----
[bindActionCreators.ts](https://github.com/Legabog/source-code-analysis-redux-legabog/blob/bindActionCreators.ts/README.md)
-----
[combineReducers.ts](https://github.com/Legabog/source-code-analysis-redux-legabog/tree/combineReducers.ts)
-----
[compose.ts](https://github.com/Legabog/source-code-analysis-redux-legabog/tree/compose.ts)
-----
[createStore.ts](https://github.com/reduxjs/redux/tree/master/src)
-----
[index.ts](https://github.com/reduxjs/redux/tree/master/src)
-----

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





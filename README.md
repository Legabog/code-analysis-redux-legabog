# redux/src/bindActionCreators.ts
// все проверки и манипулирования с actionCreators
`Код на TS`  
  
```  
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
```
  
`Код на JS`
```
// объявление функции bindActionCreators с аргументами actionCreators, dispatch
export default function bindActionCreators(actionCreators, dispatch) { 
// если тип actionCreators равен функции, то возвращается bindActionCreator(actionCreators, dispatch)
    if (typeof actionCreators === 'function') {
        return bindActionCreator(actionCreators, dispatch);
    }
// если тип actionCreators не равен объекту или actionCreators равна null, то будет ошибка    
    if (typeof actionCreators !== 'object' || actionCreators === null) {
        throw new Error(`bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
            `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`);
    }
// объявление boundActionCreators = пустой объект  
    const boundActionCreators = {};
// циклом проходим по actionCreators     
    for (const key in actionCreators) {
// объявляем переменную actionCreator и присваиваем значение actionCreators по ключу    
        const actionCreator = actionCreators[key];
// если тип actionCreator равен функции        
        if (typeof actionCreator === 'function') {
// помещаем в объект boundActionCreators результат выполнения bindActionCreator с actionCreator и  dispatch       
            boundActionCreators[key] = bindActionCreator(actionCreator, dispatch);
        }
    }
// возвращаем получившийся объект    
    return boundActionCreators;
}
```





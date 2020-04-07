# redux/src/combineReducers.ts

`Код на TS`  
  
```  
export default function combineReducers<S>(
  reducers: ReducersMapObject<S, any>
): Reducer<CombinedState<S>>
export default function combineReducers<S, A extends Action = AnyAction>(
  reducers: ReducersMapObject<S, A>
): Reducer<CombinedState<S>, A>
export default function combineReducers<M extends ReducersMapObject<any, any>>(
  reducers: M
): Reducer<
  CombinedState<StateFromReducersMapObject<M>>,
  ActionFromReducersMapObject<M>
>
export default function combineReducers(reducers: ReducersMapObject) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers: ReducersMapObject = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  // This is used to make sure we don't warn about the same
  // keys multiple times.
  let unexpectedKeyCache: { [key: string]: true }
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  let shapeAssertionError: Error
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
  }
}
```
  
`Код на JS`
```
// объявление функции combineReducers, в качестве аргументов выступают редьюсеры reducers
export default function combineReducers(reducers) {
// объявляем переменную reducerKeys и присваиваем массив свойств reducers
    const reducerKeys = Object.keys(reducers);
// объявляем переменную finalReducers и присваиваем пустой объект
    const finalReducers = {};
// объявляем цикл от 0 до длины массива reducerKeys    
    for (let i = 0; i < reducerKeys.length; i++) {
// объявляем переменную key и присваиваем ей значение массива reducerKeys    
        const key = reducerKeys[i];
// если в пользовательском окружении NODE_ENV не равен production        
        if (process.env.NODE_ENV !== 'production') {
// если тип редьюсера равен undefined вызывается warning        
            if (typeof reducers[key] === 'undefined') {
                warning(`No reducer provided for key "${key}"`);
            }
        }
// если тип редьюсера равен функции, то редьюсер добавляется в объект finalReducers      
        if (typeof reducers[key] === 'function') {
            finalReducers[key] = reducers[key];
        }
    }
// объявляется переменная finalReducersKeys, которая будет массивом свойств объекта finalReducers    
    const finalReducerKeys = Object.keys(finalReducers);
// объявляется переменная unexpectedKeyCashe
    let unexpectedKeyCache;
// если в пользовательском окружении NODE_ENV не равен production, unexpectedKeyCashe равен пустому объекту     
    if (process.env.NODE_ENV !== 'production') {
        unexpectedKeyCache = {};
    }
// объявляется переменная shapeAssertionError для отлова ошибки, как можно увидеть дальше в коде
    let shapeAssertionError;
    try {
        assertReducerShape(finalReducers);
    }
    catch (e) {
        shapeAssertionError = e;
    }
 // возвращается функция combination принимающая state (стандартное значение пустой объект) и action
    return function combination(state = {}, action) {
 // если shapeAssertionError содержит ошибку, то она показывается   
        if (shapeAssertionError) {
            throw shapeAssertionError;
        }
 // если в пользовательском окружении NODE_ENV не равен production, объявляется переменная warningMessage получающая значение  getUnexpectedStateShapeWarningMessage      
        if (process.env.NODE_ENV !== 'production') {
            const warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache);
 // если warningMessage === true, вызывается warning со значением warningMessage           
            if (warningMessage) {
                warning(warningMessage);
            }
        }
// объявляем переменную hasChanged значение false        
        let hasChanged = false;
// объявляем переменную nextState, присваиваем пустой объект          
        const nextState = {};
// объявляем цикл от 0 до длины массива finalReducerKeys          
        for (let i = 0; i < finalReducerKeys.length; i++) {
            const key = finalReducerKeys[i];
            const reducer = finalReducers[key];
            const previousStateForKey = state[key];
            const nextStateForKey = reducer(previousStateForKey, action);
            if (typeof nextStateForKey === 'undefined') {
                const errorMessage = getUndefinedStateErrorMessage(key, action);
                throw new Error(errorMessage);
            }
            nextState[key] = nextStateForKey;
            hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
        }
        hasChanged =
            hasChanged || finalReducerKeys.length !== Object.keys(state).length;
        return hasChanged ? nextState : state;
    };
}
```





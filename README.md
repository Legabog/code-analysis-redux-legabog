# redux/src/applyMiddleware.ts

> Middleware - расширение функциональности redux, дополнительная прослойка  
> applyMiddleware() - связывает store с middleware   
> Основной принцип конвеер, первый middleare самостоятельно отрабатывает и запускает следующий middleware(1 -> 2 -> 3 -> ...)  
> Применяется концепция каррирования  
  
`Код на TS`  
```ts
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
```
`Код на JS`
```js
// функция applyMiddlewares получает на входе массив из всех переданных middlewares с помощью ...(rest оператора) оператора расширения: [middlware[0], middlware[1], middlware[2], middlware[3] ...]
export default function applyMiddleware(...middlewares) { 
// возвращает стрелочную функцию
    return (createStore) => (reducer, ...args) => {
// объявляем store, присваиваем значение createStore, первый аргумент reducer, второй  массив из args(на этом месте должен быть initialState)   
        const store = createStore(reducer, ...args);
// объявляем dispatch, стрелочная функция         
        let dispatch = () => {
            throw new Error('Dispatching while constructing your middleware is not allowed. ' +
                'Other middleware would not be applied to this dispatch.');
        };
// объявляем объект middlewareAPI
        const middlewareAPI = {
// свойства getState и dispatch        
            getState: store.getState,
            dispatch: (action, ...args) => dispatch(action, ...args)
        };
// объявляем chain, цепочка выполнения middllewares, мапим все значения        
        const chain = middlewares.map(middleware => middleware(middlewareAPI))
// dispatch присваиваем функцию compose, которая представялет собой функциональную композицию, ()() каррирование, параметризация
        dispatch = compose(...chain)(store.dispatch);
// возвращаем на выходе копию копии store, с дизструктуризацией dispatch        
        return Object.assign(Object.assign({}, store), { dispatch });
    };
}
```



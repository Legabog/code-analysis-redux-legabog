# redux/src/compose.ts
Функциональное композиция  
// Последование выполнение функциональной композиции с последовательной передачей результатов выполнения следующей функции:  
 `(...args) => f(g(h(...args)))`  
 // выполнится h(...args) -> g(с результатом выполнения h(...args) -> на последнем этапе выполнится f(с реультатом g())   
 // обычно принято называть такую цепочку функциональной композицией (функцией compose):  
 // выполняется с права налево compose(f <- g <- h), обратное выполнение слева направо называется конвеер pipeline, pipe compose(f -> g -> h)  
 `compose(f, g, h)`  
 
 `Код на TS`
 ```
 export default function compose(...funcs: Function[]) {  
  if (funcs.length === 0) { 
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args: any) => a(b(...args))) 
}
```
 `Код на JS`
 ```
  //структура функции реализована по правилу 0, 1, бесконечность ...
 export default function compose(...funcs) { //функция compose принимает в качестве аргументов массив из функций
  if (funcs.length === 0) { // если передано 0 фунций возвращается фунция с arg
        return React.createElement(T, null, "(arg: T) => arg }");
    }
  if (funcs.length === 1) {
        return funcs[0];
    }
  return funcs.reduce((a, b) => (...args) => a(b(...args)));  //выполняется фунциональная композиция 
}
```




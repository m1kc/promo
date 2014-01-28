# Асинхронный JavaScript должен стать лучше

Категорически сабж.

## Проблемы текущей реализации

```javascript
setTimeout(function(){
  _get("/something.ajax?greeting", function(err, greeting) {
    if (err) { console.log(err); throw err; }
    _get("/else.ajax?who&greeting="+greeting, function(err, who) {
      if (err) { console.log(err); throw err; }
      console.log(greeting+" "+who);
    });
  });
}, 1000);
```

1. Лапша.
2. Результат выполнения функции нельзя использовать в том же контексте.
3. Ошибки надо обрабатывать вручную.

```javascript
var fibonacci = function(n, callback) {
    var inner = function(n1, n2, i) {
        if (i > n) {
            callback(null, n2);
            return;
        }
        var func = (i % 100) ? inner : inner_tick;
        func(n2, n1 + n2, i + 1);
    }
    var inner_tick = function(n1, n2, i) {
        process.nextTick(function() { inner(n1, n2, i); });
    }
    if (n == 1 || n == 2) {
        callback(null, 1);
    } else {
        inner(1, 1, 3);
    }
}
```

1. Эффективная реализация асинхронной функции часто бывает сильно запутанной.
2. Невозможно продолжить выполнение в том же контексте, в котором оно началось.

## Существующие решения

### async.series

```javascript
async.series([
  function(callback){ callback(null, 42); },
  function(callback){ callback(null, 24); },
],
function(err, results){
  if (!!err)
  {
    console.error(err);
  }
  else
  {
    console.log(results[0]); // 42
    console.log(results[1]); // 24
  }
});
```

1. Самое главное - функция не может знать результат выполнения предыдущей.
2. Ошибки всё равно приходится обрабатывать, хотя уже и без рутины.
3. Работа через массив results не слишком интуитивна.
4. Не даёт никакого удобства в написании самих функций, только в работе с ними.

### async.waterfall

```javascript
async.waterfall([
  function(callback){
    callback(null, 'one', 'two');
  },
  function(arg1, arg2, callback){
    callback(null, 'three');
  },
  function(arg1, callback){
    // arg1 now equals 'three'
    callback(null, 'done');
  }
], function (err, result) {
  if (!!err)
  {
    console.error(err);
  }
  else
  {
    // result now equals 'done'
  }  
});
```

1. Аргументы - дикий пиздец. Нельзя написать функцию, не посчитав число результатов предыдущей. Даже если все функции возвращают только один результат, у первой и второй число аргументов будет разным.
2. Ошибки всё равно надо обрабатывать.
3. Если результат работы первой функции требуется в последней, его приходится таскать за собой. Если требуются все результаты - нужно таскать все результаты через все функции. От этого можно сойти с ума.
4. Не даёт никакого удобства в написании самих функций, только в работе с ними.

### sync

```javascript
var result1 = func1.sync(null, 4, 2);
var result2 = func2.sync(null, 2, 4);
console.log(result1+", "+result2);
```

Почти идеал. Ошибки обрабатываются сами (если error != null, библиотека сама бросит исключение). Но тоже есть проблемы:

1. Приходится писать `null` первым аргументом при каждом вызове.
2. Изменяет прототип Function, что не очень хорошо.
3. Каждый раз писать `sync` тоже слегка напрягает.

```javascript
sum = function(x, y) {
  return x+y;
}.async();
```

Асинхронная функция выглядит как синхронная. Кайф. Единственная проблема:

1. Писать `async()` каждый раз надоедает, особенно в CoffeeScript.


## Предлагаемое решение №1

Давайте просто поправим косяки sync и сделаем это частью стандарта языка.

Асинхронные функции предлагается вызывать через ключевое слово `async`. Ключевое слово `async` добавляет к параметрам функции невидимый колбэк и получает результат через него. Если в колбэк пришла ошибка - вылетает исключение.

```javascript
var result1 = async func1(4, 2);
var result2 = async func2(2, 4);
console.log(result1+", "+result2);
```

Если функция принимает колбэк не с двумя аргументами, к ключевому слову `async` может добавляться их описание.

```javascript
var x = async(result) sum(2, 2); // sum возвращает только результат
var x = async(error, result) sum(2, 2); // sum может вернуть и ошибку, и результат
var x = async(error) sum(2, 2); // sum может вернуть либо ошибку, либо ничего
var x = async sum(2, 2); // аналогично async(error, result)
```

При написании асинхронных функций используется это же слово, но уже безо всяких описаний.

```javascript
sum = async function(x, y) {
  return x+y;
};
```

В данном случае `return` передаст результат вторым аргументом, а `throw` передаст ошибку первым.

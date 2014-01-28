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

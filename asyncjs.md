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

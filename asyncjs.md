# Предлагаемая спецификация асинхронного JavaScript

Нас заебали ваши колбэки. Мы хотим нормальный линейный код. Мы хотим, чтобы асинхронное I/O не мешало нам писать программы.

## Принципы

1. Асинхронный код должен выглядеть как синхронный.
2. Вызванная функция должна возвращать значение одним и тем же способом вне зависимости от того, асинхронная она или нет.
3. Исключения должны обрабатываться одним и тем же способом вне зависимости от того, асинхронная функция вызвана или нет.

## Как это должно выглядеть

### Использование

```javascript
isInFight = function(dbConnection, userid) {
  return dbConnection.query("SELECT `fight_mode` FROM `uniusers` WHERE `id` = ?", [userid]);
  // query - асинхронная функция, но нас это не должно волновать
};
```

### Написание

```javascript
asyncFac = function(n) {
  if (n == 0) return 1;
  if (n == 1) return 1;
  var result = n;
  for (i = n-1; i >= 1; i--)
  {
    result = result*i;
    if (i % 100 == 0) yield; // раз в сто итераций освобождать поток
  }
  return result;
}
```

```javascript
readFromSlowStream = function(stream, bytes) {
  var output = [];
  var left = bytes;
  while(true)
  {
    // читаем из потока все байты, какие в нём есть (но не больше, чем нам надо),
    // и помещаем в output
    var toRead = Math.min(stream.available(), left);
    if (toRead > 0)
    {
      output.append(stream.readBytes(toRead));
      left -= toRead;
    }
    // если байты кончились, а нам надо ещё - отдаём управление
    // иначе - возвращаем результат
    if (left == 0)
    {
      return output;
    }
    else
    {
      yield;
    }
  }
}
```

```javascript
dbQuery = function(sql) {
  dbSocket.send(sql);
  while(dbSocket.noAnswer())
  {
    yield;
  }
  return dbSocket.readAnswer();
}
```

## Совместимость со старым кодом

Если код умеет возвращать значение только через колбэк, используется ключевое слово `callback`, добавляющее последний неявный аргумент.

```javascript
// legacy code
sum = function(x, y, callback) {
  callback(x+y);
}

var result = callback sum(2, 2);
```

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
    if (i % 100 == 0) yield;
  }
  return result;
}
```

```javascript
readFromSlowStream = function(stream, bytes) {
  var output = [];
  for (i = 0; i < bytes; i++)
  {
    if(stream.available > 0)
    {
      output.push(stream.readByte());
    }
    else
    {
      yield;
    }
  }
  return output;
}
```

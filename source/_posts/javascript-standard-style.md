title: JavaScript Standard Style
date: 2016-05-09 10:50:28
tags: [javascript]
---

# Javascript Standard Style

这是一个标准的 JavaScript 书写规范。

## 规范

### 使用 2 个空格进行缩进

```javascript
  function hello (name) {
    console.log('hi', name)
  }
```

### 使用单引号包裹字符串以防止转义

```javascript
  console.log('hello there')
  $("<div class='box'>")
```

### 不定义未使用的变量

```javascript
  function myFunction () {
    var result = something()    // avoid
  }
```

### 在关键字后应追加一个空格

```javascript
  if (condition) { ... }    // ok
  if(condition) { ... }     // avoid
```

### 在函数声明的括号前追加一个空格

```javascript
  function name (arg) { ... }     // ok
  function name(arg) { ... }      // avoid

  run(function () { ... })        // ok
  run(function() { ... })         // avoid
```

### 应始终使用 `===` 来代替 `==`

例外： `obj == null` 允许使用 `==` 来判断 `null || undefined`

```javascript
  if (name === 'John')      // ok
  if (name == 'John')       // avoid

  if (name !== 'John')      // ok
  if (name != 'John')       // avoid
```

### 操作符前后应有空格

```javascript
  // ok
  var x = 2
  var message = 'hello, ' + name + '!'

  // avoid
  var x=2
  var message = 'hello, '+name+'!'
```

### 逗号后应该追加一个空格

```javascript
  // ok
  var list = [1, 2, 3, 4]
  function greet (name, options) { ... }

  // avoid
  var list = [1,2,3,4]
  function greet (name,options) { ... }
```

### 使 else 声明和其大括号保持在同一行

```javascript
  // ok
  if (condition) {
    // ...
  } else {
    // ...
  }

  // avoid
  if (condition) {
    // ...
  }
  else {
    // ...
  }
```

### 对于多行的 if 语句，应使用大括号

```javascript
  // ok
  if (options.quiet !== true) console.log('done')

  // ok
  if (options.quiet !== true) {
    console.log('done')
  }

  // avoid
  if (option.quiet !== true)
    console.log('done')
```

### 总是处理函数中的 err 参数

```javascript
  // ok
  run(function (err) {
    if (err) throw err
    window.alert('done')
  })

  // avoid
  run(function (err) {
    window.alert('done')
  })
```

### 对于浏览器全局变量应始终使用 `window.` 前缀

例外： `document`，`console` 和 `navigator`

```javascript
  window.alert('hi')
```

### 不允许存在多行空行

```javascript
  // ok
  var value = 'hello world'
  console.log(value)

  // avoid
  var value = 'hello world'


  console.log(value)
```

### 对于多行的三元运算，应将 `?` 和 `:` 独立一行

```javascript
  // ok
  var location = env.development ? 'localhost' : 'www.api.com'

  // ok
  var location = env.development
    ? 'localhost'
    : 'www.api.com'

  // avoid
  var location = env.development ?
    'localhost' :
    'www.api.com'
```

### 对于 var 声明，每个变量应该进行独立声明

```javascript
  // ok
  var silent = true
  var verbose = true

  // avoid
  var silent = true, verbose = true

  // avoid
  var silent = true,
      verbose = true
```

### 在条件判断的表达式中，应使用括号包裹赋值语句

这样明确的表示是故意使用 `=` 赋值，而非 `===`

```javascript
  // ok
  while ((m = text.match(expr))) {
    // ...
  }

  // avoid
  while (m = text.match(expr)) {
    // ...
  }
```

## 分号

### 不允许使用分号

```javascript
  window.alert('hi')      // ok
  window.alert('hi');     // avoid
```

### 永远不使用 **(**、**[**、**`** 作为一行的开头

```javascript
  // ok
  ;(function () {
    window.alert('ok')
  }())
  
  // avoid
  (function () {
    window.alert('ok')
  }())


  // ok
  ;[1, 2, 3].forEach(bar)
  
  // avoid
  [1, 2, 3].forEach(bar)

  // ok
  ;`hello`.indexOf('o')

  // avoid
  `hello`.indexOf('o')
```

对于这种表达式：

```javascript
  ;[1, 2, 3].forEach(bar)
```

可以这么替代:

```javascript
  var nums = [1, 2, 3]
  nums.forEach(bar)
```

参考： [JavaScript Standrad Style](https://github.com/feross/standard/blob/master/RULES.md)
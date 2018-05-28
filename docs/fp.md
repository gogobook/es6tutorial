# 函數式編程

JavaScript 語言從一誕生，就具有函數式編程的烙印。它將函數作為一種獨立的數據類型，與其他數據類型處於完全平等的地位。在 JavaScript 語言中，你可以採用面向物件編程，也可以採用函數式編程。有人甚至說，JavaScript 是有史以來第一種被大規模採用的函數式編程語言。

ES6 的種種新增功能，使得函數式編程變得更方便、更強大。本章介紹 ES6 如何進行函數式編程。

## 柯裡化

柯裡化（currying）指的是將一個多參數的函數拆分成一系列函數，每個拆分後的函數都只接受一個參數（unary）。

```javascript
function add (a, b) {
  return a + b;
}

add(1, 1) // 2
```

上面代碼中，函數`add`接受兩個參數`a`和`b`。

柯裡化就是將上面的函數拆分成兩個函數，每個函數都只接受一個參數。

```javascript
function add (a) {
  return function (b) {
    return a + b;
  }
}
// 或者採用箭頭函數寫法
const add = x => y => x + y;

const f = add(1);
f(1) // 2
```

上面代碼中，函數`add`只接受一個參數`a`，返回一個函數`f`。函數`f`也只接受一個參數`b`。

## 函數合成

函數合成（function composition）指的是，將多個函數合成一個函數。

```javascript
const compose = f => g => x => f(g(x));

const f = compose (x => x * 4) (x => x + 3);
f(2) // 20
```

上面代碼中，`compose`就是一個函數合成器，用於將兩個函數合成一個函數。

可以發現，柯裡化與函數合成有著密切的聯繫。前者用於將一個函數拆成多個函數，後者用於將多個函數合併成一個函數。

## 參數倒置

參數倒置（flip）指的是改變函數前兩個參數的順序。

```javascript
var divide = (a, b) => a / b;
var flip = f.flip(divide);

flip(10, 5) // 0.5
flip(1, 10) // 10

var three = (a, b, c) => [a, b, c];
var flip = f.flip(three);
flip(1, 2, 3); // => [2, 1, 3]
```

上面代碼中，如果按照正常的參數順序，10 除以 5 等於 2。但是，參數倒置以後得到的新函數，結果就是 5 除以 10，結果得到 0.5。如果原函數有 3 個參數，則只顛倒前兩個參數的位置。

參數倒置的代碼非常簡單。

```javascript
let f = {};
f.flip =
  fn =>
    (a, b, ...args) => fn(b, a, ...args.reverse());
```

## 執行邊界

執行邊界（until）指的是函數執行到滿足條件為止。

```javascript
let condition = x => x > 100;
let inc = x => x + 1;
let until = f.until(condition, inc);

until(0) // 101

condition = x => x === 5;
until = f.until(condition, inc);

until(3) // 5
```

上面代碼中，第一段的條件是執行到`x`大於 100 為止，所以`x`初值為 0 時，會一直執行到 101。第二段的條件是執行到等於 5 為止，所以`x`最後的值是 5。

執行邊界的實現如下。

```javascript
let f = {};
f.until = (condition, f) =>
  (...args) => {
    var r = f.apply(null, args);
    return condition(r) ? r : f.until(condition, f)(r);
  };
```

上面代碼的關鍵就是，如果滿足條件就返回結果，否則不斷遞歸執行。

## 隊列操作

隊列（list）操作包括以下幾種。

- `head`： 取出隊列的第一個非空成員。
- `last`： 取出有限隊列的最後一個非空成員。
- `tail`： 取出除了“隊列頭”以外的其他非空成員。
- `init`： 取出除了“隊列尾”以外的其他非空成員。

下面是例子。

```javascript
f.head(5, 27, 3, 1) // 5
f.last(5, 27, 3, 1) // 1
f.tail(5, 27, 3, 1) // [27, 3, 1]
f.init(5, 27, 3, 1) // [5, 27, 3]
```

這些方法的實現如下。

```javascript
let f = {};
f.head = (...xs) => xs[0];
f.last = (...xs) => xs.slice(-1);
f.tail = (...xs) => Array.prototype.slice.call(xs, 1);
f.init = (...xs) => xs.slice(0, -1);
```

## 合併操作

合併操作分為`concat`和`concatMap`兩種。前者就是將多個陣列合成一個，後者則是先處理一下參數，然後再將處理結果合成一個陣列。

```javascript
f.concat([5], [27], [3]) // [5, 27, 3]
f.concatMap(x => 'hi ' + x, 1, [[2]], 3) // ['hi 1', 'hi 2', 'hi 3']
```

這兩種方法的實現代碼如下。

```javascript
let f = {};
f.concat =
  (...xs) => xs.reduce((a, b) => a.concat(b));
f.concatMap =
  (f, ...xs) => f.concat(xs.map(f));
```

## 配對操作

配對操作分為`zip`和`zipWith`兩種方法。`zip`操作將兩個隊列的成員，一一配對，合成一個新的隊列。如果兩個隊列不等長，較長的那個隊列多出來的成員，會被忽略。`zipWith`操作的第一個參數是一個函數，然後會將後面的隊列成員一一配對，輸入該函數，返回值就組成一個新的隊列。

下面是例子。

```javascript
let a = [0, 1, 2];
let b = [3, 4, 5];
let c = [6, 7, 8];

f.zip(a, b) // [[0, 3], [1, 4], [2, 5]]
f.zipWith((a, b) => a + b, a, b, c) // [9, 12, 15]
```

上面代碼中，`zipWith`方法的第一個參數是一個求和函數，它將後面三個隊列的成員，一一配對進行相加。

這兩個方法的實現如下。

```javascript
let f = {};

f.zip = (...xs) => {
  let r = [];
  let nple = [];
  let length = Math.min.apply(null, xs.map(x => x.length));

  for (var i = 0; i < length; i++) {
    xs.forEach(
      x => nple.push(x[i])
    );

    r.push(nple);
    nple = [];
  }

  return r;
};

f.zipWith = (op, ...xs) =>
  f.zip.apply(null, xs).map(
    (x) => x.reduce(op)
  );
```

## 參考鏈接

- Mateo Gianolio, [Haskell in ES6: Part 1](http://casualjavascript.com/?1)
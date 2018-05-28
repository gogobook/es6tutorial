# Generator 函數的語法

## 簡介

### 基本概念

Generator 函數是 ES6 提供的一種異步編程解決方案，語法行為與傳統函數完全不同。本章詳細介紹 Generator 函數的語法和 API，它的異步編程應用請看《Generator 函數的異步應用》一章。

Generator 函數有多種理解角度。語法上，首先可以把它理解成，Generator 函數是一個狀態機，封裝了多個內部狀態。

執行 Generator 函數會返回一個遍歷器物件，也就是說，Generator 函數除了狀態機，還是一個遍歷器物件生成函數。返回的遍歷器物件，可以依次遍歷 Generator 函數內部的每一個狀態。

形式上，Generator 函數是一個普通函數，但是有兩個特徵。一是，`function`關鍵字與函數名之間有一個星號；二是，函數體內部使用`yield`表達式，定義不同的內部狀態（`yield`在英語裡的意思就是“產出”）。

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
```

上面代碼定義了一個 Generator 函數`helloWorldGenerator`，它內部有兩個`yield`表達式（`hello`和`world`），即該函數有三個狀態：hello，world 和 return 語句（結束執行）。

然後，Generator 函數的調用方法與普通函數一樣，也是在函數名後面加上一對圓括號。不同的是，調用 Generator 函數後，該函數並不執行，返回的也不是函數運行結果，而是一個指向內部狀態的指針物件，也就是上一章介紹的遍歷器物件（Iterator Object）。

下一步，必須調用遍歷器物件的`next`方法，使得指針移向下一個狀態。也就是說，每次調用`next`方法，內部指針就從函數頭部或上一次停下來的地方開始執行，直到遇到下一個`yield`表達式（或`return`語句）為止。換言之，Generator 函數是分段執行的，`yield`表達式是暫停執行的標記，而`next`方法可以恢復執行。

```javascript
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

上面代碼一共調用了四次`next`方法。

第一次調用，Generator 函數開始執行，直到遇到第一個`yield`表達式為止。`next`方法返回一個物件，它的`value`屬性就是當前`yield`表達式的值`hello`，`done`屬性的值`false`，表示遍歷還沒有結束。

第二次調用，Generator 函數從上次`yield`表達式停下的地方，一直執行到下一個`yield`表達式。`next`方法返回的物件的`value`屬性就是當前`yield`表達式的值`world`，`done`屬性的值`false`，表示遍歷還沒有結束。

第三次調用，Generator 函數從上次`yield`表達式停下的地方，一直執行到`return`語句（如果沒有`return`語句，就執行到函數結束）。`next`方法返回的物件的`value`屬性，就是緊跟在`return`語句後面的表達式的值（如果沒有`return`語句，則`value`屬性的值為`undefined`），`done`屬性的值`true`，表示遍歷已經結束。

第四次調用，此時 Generator 函數已經運行完畢，`next`方法返回物件的`value`屬性為`undefined`，`done`屬性為`true`。以後再調用`next`方法，返回的都是這個值。

總結一下，調用 Generator 函數，返回一個遍歷器物件，代表 Generator 函數的內部指針。以後，每次調用遍歷器物件的`next`方法，就會返回一個有著`value`和`done`兩個屬性的物件。`value`屬性表示當前的內部狀態的值，是`yield`表達式後面那個表達式的值；`done`屬性是一個布爾值，表示是否遍歷結束。

ES6 沒有規定，`function`關鍵字與函數名之間的星號，寫在哪個位置。這導致下面的寫法都能通過。

```javascript
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
```

由於 Generator 函數仍然是普通函數，所以一般的寫法是上面的第三種，即星號緊跟在`function`關鍵字後面。本書也採用這種寫法。

### yield 表達式

由於 Generator 函數返回的遍歷器物件，只有調用`next`方法才會遍歷下一個內部狀態，所以其實提供了一種可以暫停執行的函數。`yield`表達式就是暫停標誌。

遍歷器物件的`next`方法的運行邏輯如下。

（1）遇到`yield`表達式，就暫停執行後面的操作，並將緊跟在`yield`後面的那個表達式的值，作為返回的物件的`value`屬性值。

（2）下一次調用`next`方法時，再繼續往下執行，直到遇到下一個`yield`表達式。

（3）如果沒有再遇到新的`yield`表達式，就一直運行到函數結束，直到`return`語句為止，並將`return`語句後面的表達式的值，作為返回的物件的`value`屬性值。

（4）如果該函數沒有`return`語句，則返回的物件的`value`屬性值為`undefined`。

需要注意的是，`yield`表達式後面的表達式，只有當調用`next`方法、內部指針指向該語句時才會執行，因此等於為 JavaScript 提供了手動的“惰性求值”（Lazy Evaluation）的語法功能。

```javascript
function* gen() {
  yield  123 + 456;
}
```

上面代碼中，`yield`後面的表達式`123 + 456`，不會立即求值，只會在`next`方法將指針移到這一句時，才會求值。

`yield`表達式與`return`語句既有相似之處，也有區別。相似之處在於，都能返回緊跟在語句後面的那個表達式的值。區別在於每次遇到`yield`，函數暫停執行，下一次再從該位置繼續向後執行，而`return`語句不具備位置記憶的功能。一個函數裡面，只能執行一次（或者說一個）`return`語句，但是可以執行多次（或者說多個）`yield`表達式。正常函數只能返回一個值，因為只能執行一次`return`；Generator 函數可以返回一系列的值，因為可以有任意多個`yield`。從另一個角度看，也可以說 Generator 生成了一系列的值，這也就是它的名稱的來歷（英語中，generator 這個詞是“生成器”的意思）。

Generator 函數可以不用`yield`表達式，這時就變成了一個單純的暫緩執行函數。

```javascript
function* f() {
  console.log('執行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
```

上面代碼中，函數`f`如果是普通函數，在為變數`generator`賦值時就會執行。但是，函數`f`是一個 Generator 函數，就變成只有調用`next`方法時，函數`f`才會執行。

另外需要注意，`yield`表達式只能用在 Generator 函數裡面，用在其他地方都會報錯。

```javascript
(function (){
  yield 1;
})()
// SyntaxError: Unexpected number
```

上面代碼在一個普通函數中使用`yield`表達式，結果產生一個句法錯誤。

下面是另外一個例子。

```javascript
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a) {
  a.forEach(function (item) {
    if (typeof item !== 'number') {
      yield* flat(item);
    } else {
      yield item;
    }
  });
};

for (var f of flat(arr)){
  console.log(f);
}
```

上面代碼也會產生句法錯誤，因為`forEach`方法的參數是一個普通函數，但是在裡面使用了`yield`表達式（這個函數裡面還使用了`yield*`表達式，詳細介紹見後文）。一種修改方法是改用`for`循環。

```javascript
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a) {
  var length = a.length;
  for (var i = 0; i < length; i++) {
    var item = a[i];
    if (typeof item !== 'number') {
      yield* flat(item);
    } else {
      yield item;
    }
  }
};

for (var f of flat(arr)) {
  console.log(f);
}
// 1, 2, 3, 4, 5, 6
```

另外，`yield`表達式如果用在另一個表達式之中，必須放在圓括號裡面。

```javascript
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
```

`yield`表達式用作函數參數或放在賦值表達式的右邊，可以不加括號。

```javascript
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```

### 與 Iterator 接口的關係

上一章說過，任意一個物件的`Symbol.iterator`方法，等於該物件的遍歷器生成函數，調用該函數會返回該物件的一個遍歷器物件。

由於 Generator 函數就是遍歷器生成函數，因此可以把 Generator 賦值給物件的`Symbol.iterator`屬性，從而使得該物件具有 Iterator 接口。

```javascript
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

上面代碼中，Generator 函數賦值給`Symbol.iterator`屬性，從而使得`myIterable`物件具有了 Iterator 接口，可以被`...`運算符遍歷了。

Generator 函數執行後，返回一個遍歷器物件。該物件本身也具有`Symbol.iterator`屬性，執行後返回自身。

```javascript
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true
```

上面代碼中，`gen`是一個 Generator 函數，調用它會生成一個遍歷器物件`g`。它的`Symbol.iterator`屬性，也是一個遍歷器物件生成函數，執行後返回它自己。

## next 方法的參數

`yield`表達式本身沒有返回值，或者說總是返回`undefined`。`next`方法可以帶一個參數，該參數就會被當作上一個`yield`表達式的返回值。

```javascript
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

上面代碼先定義了一個可以無限運行的 Generator 函數`f`，如果`next`方法沒有參數，每次運行到`yield`表達式，變數`reset`的值總是`undefined`。當`next`方法帶一個參數`true`時，變數`reset`就被重置為這個參數（即`true`），因此`i`會等於`-1`，下一輪循環就會從`-1`開始遞增。

這個功能有很重要的語法意義。Generator 函數從暫停狀態到恢復運行，它的上下文狀態（context）是不變的。通過`next`方法的參數，就有辦法在 Generator 函數開始運行之後，繼續向函數體內部注入值。也就是說，可以在 Generator 函數運行的不同階段，從外部向內部注入不同的值，從而調整函數行為。

再看一個例子。

```javascript
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

上面代碼中，第二次運行`next`方法的時候不帶參數，導致 y 的值等於`2 * undefined`（即`NaN`），除以 3 以後還是`NaN`，因此返回物件的`value`屬性也等於`NaN`。第三次運行`Next`方法的時候不帶參數，所以`z`等於`undefined`，返回物件的`value`屬性等於`5 + NaN + undefined`，即`NaN`。

如果向`next`方法提供參數，返回結果就完全不一樣了。上面代碼第一次調用`b`的`next`方法時，返回`x+1`的值`6`；第二次調用`next`方法，將上一次`yield`表達式的值設為`12`，因此`y`等於`24`，返回`y / 3`的值`8`；第三次調用`next`方法，將上一次`yield`表達式的值設為`13`，因此`z`等於`13`，這時`x`等於`5`，`y`等於`24`，所以`return`語句的值等於`42`。

注意，由於`next`方法的參數表示上一個`yield`表達式的返回值，所以在第一次使用`next`方法時，傳遞參數是無效的。V8 引擎直接忽略第一次使用`next`方法時的參數，只有從第二次使用`next`方法開始，參數才是有效的。從語義上講，第一個`next`方法用來啟動遍歷器物件，所以不用帶有參數。

再看一個通過`next`方法的參數，向 Generator 函數內部輸入值的例子。

```javascript
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}

let genObj = dataConsumer();
genObj.next();
// Started
genObj.next('a')
// 1. a
genObj.next('b')
// 2. b
```

上面代碼是一個很直觀的例子，每次通過`next`方法向 Generator 函數輸入值，然後打印出來。

如果想要第一次調用`next`方法時，就能夠輸入值，可以在 Generator 函數外面再包一層。

```javascript
function wrapper(generatorFunction) {
  return function (...args) {
    let generatorObject = generatorFunction(...args);
    generatorObject.next();
    return generatorObject;
  };
}

const wrapped = wrapper(function* () {
  console.log(`First input: ${yield}`);
  return 'DONE';
});

wrapped().next('hello!')
// First input: hello!
```

上面代碼中，Generator 函數如果不用`wrapper`先包一層，是無法第一次調用`next`方法，就輸入參數的。

## for...of 循環

`for...of`循環可以自動遍歷 Generator 函數時生成的`Iterator`物件，且此時不再需要調用`next`方法。

```javascript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

上面代碼使用`for...of`循環，依次顯示 5 個`yield`表達式的值。這裡需要注意，一旦`next`方法的返回物件的`done`屬性為`true`，`for...of`循環就會中止，且不包含該返回物件，所以上面代碼的`return`語句返回的`6`，不包括在`for...of`循環之中。

下面是一個利用 Generator 函數和`for...of`循環，實現斐波那契數列的例子。

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

從上面代碼可見，使用`for...of`語句時不需要使用`next`方法。

利用`for...of`循環，可以寫出遍歷任意物件（object）的方法。原生的 JavaScript 物件沒有遍歷接口，無法使用`for...of`循環，通過 Generator 函數為它加上這個接口，就可以用了。

```javascript
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

上面代碼中，物件`jane`原生不具備 Iterator 接口，無法用`for...of`遍歷。這時，我們通過 Generator 函數`objectEntries`為它加上遍歷器接口，就可以用`for...of`遍歷了。加上遍歷器接口的另一種寫法是，將 Generator 函數加到物件的`Symbol.iterator`屬性上面。

```javascript
function* objectEntries() {
  let propKeys = Object.keys(this);

  for (let propKey of propKeys) {
    yield [propKey, this[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

jane[Symbol.iterator] = objectEntries;

for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

除了`for...of`循環以外，擴展運算符（`...`）、解構賦值和`Array.from`方法內部調用的，都是遍歷器接口。這意味著，它們都可以將 Generator 函數返回的 Iterator 物件，作為參數。

```javascript
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}

// 擴展運算符
[...numbers()] // [1, 2]

// Array.from 方法
Array.from(numbers()) // [1, 2]

// 解構賦值
let [x, y] = numbers();
x // 1
y // 2

// for...of 循環
for (let n of numbers()) {
  console.log(n)
}
// 1
// 2
```

## Generator.prototype.throw()

Generator 函數返回的遍歷器物件，都有一個`throw`方法，可以在函數體外拋出錯誤，然後在 Generator 函數體內捕獲。

```javascript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('內部捕獲', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕獲', e);
}
// 內部捕獲 a
// 外部捕獲 b
```

上面代碼中，遍歷器物件`i`連續拋出兩個錯誤。第一個錯誤被 Generator 函數體內的`catch`語句捕獲。`i`第二次拋出錯誤，由於 Generator 函數內部的`catch`語句已經執行過了，不會再捕捉到這個錯誤了，所以這個錯誤就被拋出了 Generator 函數體，被函數體外的`catch`語句捕獲。

`throw`方法可以接受一個參數，該參數會被`catch`語句接收，建議拋出`Error`物件的實例。

```javascript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log(e);
  }
};

var i = g();
i.next();
i.throw(new Error('出錯了！'));
// Error: 出錯了！(…)
```

注意，不要混淆遍歷器物件的`throw`方法和全局的`throw`命令。上面代碼的錯誤，是用遍歷器物件的`throw`方法拋出的，而不是用`throw`命令拋出的。後者只能被函數體外的`catch`語句捕獲。

```javascript
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('內部捕獲', e);
    }
  }
};

var i = g();
i.next();

try {
  throw new Error('a');
  throw new Error('b');
} catch (e) {
  console.log('外部捕獲', e);
}
// 外部捕獲 [Error: a]
```

上面代碼之所以只捕獲了`a`，是因為函數體外的`catch`語句塊，捕獲了拋出的`a`錯誤以後，就不會再繼續`try`代碼塊裡面剩餘的語句了。

如果 Generator 函數內部沒有部署`try...catch`代碼塊，那麼`throw`方法拋出的錯誤，將被外部`try...catch`代碼塊捕獲。

```javascript
var g = function* () {
  while (true) {
    yield;
    console.log('內部捕獲', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕獲', e);
}
// 外部捕獲 a
```

上面代碼中，Generator 函數`g`內部沒有部署`try...catch`代碼塊，所以拋出的錯誤直接被外部`catch`代碼塊捕獲。

如果 Generator 函數內部和外部，都沒有部署`try...catch`代碼塊，那麼程序將報錯，直接中斷執行。

```javascript
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();
g.throw();
// hello
// Uncaught undefined
```

上面代碼中，`g.throw`拋出錯誤以後，沒有任何`try...catch`代碼塊可以捕獲這個錯誤，導致程序報錯，中斷執行。

`throw`方法拋出的錯誤要被內部捕獲，前提是必須至少執行過一次`next`方法。

```javascript
function* gen() {
  try {
    yield 1;
  } catch (e) {
    console.log('內部捕獲');
  }
}

var g = gen();
g.throw(1);
// Uncaught 1
```

上面代碼中，`g.throw(1)`執行時，`next`方法一次都沒有執行過。這時，拋出的錯誤不會被內部捕獲，而是直接在外部拋出，導致程序出錯。這種行為其實很好理解，因為第一次執行`next`方法，等同於啟動執行 Generator 函數的內部代碼，否則 Generator 函數還沒有開始執行，這時`throw`方法拋錯只可能拋出在函數外部。

`throw`方法被捕獲以後，會附帶執行下一條`yield`表達式。也就是說，會附帶執行一次`next`方法。

```javascript
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```

上面代碼中，`g.throw`方法被捕獲以後，自動執行了一次`next`方法，所以會打印`b`。另外，也可以看到，只要 Generator 函數內部部署了`try...catch`代碼塊，那麼遍歷器的`throw`方法拋出的錯誤，不影響下一次遍歷。

另外，`throw`命令與`g.throw`方法是無關的，兩者互不影響。

```javascript
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();

try {
  throw new Error();
} catch (e) {
  g.next();
}
// hello
// world
```

上面代碼中，`throw`命令拋出的錯誤不會影響到遍歷器的狀態，所以兩次執行`next`方法，都進行了正確的操作。

這種函數體內捕獲錯誤的機制，大大方便了對錯誤的處理。多個`yield`表達式，可以只用一個`try...catch`代碼塊來捕獲錯誤。如果使用回調函數的寫法，想要捕獲多個錯誤，就不得不為每個函數內部寫一個錯誤處理語句，現在只在 Generator 函數內部寫一次`catch`語句就可以了。

Generator 函數體外拋出的錯誤，可以在函數體內捕獲；反過來，Generator 函數體內拋出的錯誤，也可以被函數體外的`catch`捕獲。

```javascript
function* foo() {
  var x = yield 3;
  var y = x.toUpperCase();
  yield y;
}

var it = foo();

it.next(); // { value:3, done:false }

try {
  it.next(42);
} catch (err) {
  console.log(err);
}
```

上面代碼中，第二個`next`方法向函數體內傳入一個參數 42，數值是沒有`toUpperCase`方法的，所以會拋出一個 TypeError 錯誤，被函數體外的`catch`捕獲。

一旦 Generator 執行過程中拋出錯誤，且沒有被內部捕獲，就不會再執行下去了。如果此後還調用`next`方法，將返回一個`value`屬性等於`undefined`、`done`屬性等於`true`的物件，即 JavaScript 引擎認為這個 Generator 已經運行結束了。

```javascript
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}

function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次運行next方法', v);
  } catch (err) {
    console.log('捕捉錯誤', v);
  }
  try {
    v = generator.next();
    console.log('第二次運行next方法', v);
  } catch (err) {
    console.log('捕捉錯誤', v);
  }
  try {
    v = generator.next();
    console.log('第三次運行next方法', v);
  } catch (err) {
    console.log('捕捉錯誤', v);
  }
  console.log('caller done');
}

log(g());
// starting generator
// 第一次運行next方法 { value: 1, done: false }
// throwing an exception
// 捕捉錯誤 { value: 1, done: false }
// 第三次運行next方法 { value: undefined, done: true }
// caller done
```

上面代碼一共三次運行`next`方法，第二次運行的時候會拋出錯誤，然後第三次運行的時候，Generator 函數就已經結束了，不再執行下去了。

## Generator.prototype.return()

Generator 函數返回的遍歷器物件，還有一個`return`方法，可以返回給定的值，並且終結遍歷 Generator 函數。

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

上面代碼中，遍歷器物件`g`調用`return`方法後，返回值的`value`屬性就是`return`方法的參數`foo`。並且，Generator 函數的遍歷就終止了，返回值的`done`屬性為`true`，以後再調用`next`方法，`done`屬性總是返回`true`。

如果`return`方法調用時，不提供參數，則返回值的`value`屬性為`undefined`。

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return() // { value: undefined, done: true }
```

如果 Generator 函數內部有`try...finally`代碼塊，那麼`return`方法會推遲到`finally`代碼塊執行完再執行。

```javascript
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```

上面代碼中，調用`return`方法後，就開始執行`finally`代碼塊，然後等到`finally`代碼塊執行完，再執行`return`方法。

## next()、throw()、return() 的共同點

`next()`、`throw()`、`return()`這三個方法本質上是同一件事，可以放在一起理解。它們的作用都是讓 Generator 函數恢復執行，並且使用不同的語句替換`yield`表達式。

`next()`是將`yield`表達式替換成一個值。

```javascript
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

const gen = g(1, 2);
gen.next(); // Object {value: 3, done: false}

gen.next(1); // Object {value: 1, done: true}
// 相當於將 let result = yield x + y
// 替換成 let result = 1;
```

上面代碼中，第二個`next(1)`方法就相當於將`yield`表達式替換成一個值`1`。如果`next`方法沒有參數，就相當於替換成`undefined`。

`throw()`是將`yield`表達式替換成一個`throw`語句。

```javascript
gen.throw(new Error('出錯了')); // Uncaught Error: 出錯了
// 相當於將 let result = yield x + y
// 替換成 let result = throw(new Error('出錯了'));
```

`return()`是將`yield`表達式替換成一個`return`語句。

```javascript
gen.return(2); // Object {value: 2, done: true}
// 相當於將 let result = yield x + y
// 替換成 let result = return 2;
```

## yield\* 表達式

如果在 Generator 函數內部，調用另一個 Generator 函數，默認情況下是沒有效果的。

```javascript
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'x';
  foo();
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "y"
```

上面代碼中，`foo`和`bar`都是 Generator 函數，在`bar`裡面調用`foo`，是不會有效果的。

這個就需要用到`yield*`表達式，用來在一個 Generator 函數裡面執行另一個 Generator 函數。

```javascript
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同於
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同於
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```

再來看一個對比的例子。

```javascript
function* inner() {
  yield 'hello!';
}

function* outer1() {
  yield 'open';
  yield inner();
  yield 'close';
}

var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一個遍歷器物件
gen.next().value // "close"

function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}

var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
```

上面例子中，`outer2`使用了`yield*`，`outer1`沒使用。結果就是，`outer1`返回一個遍歷器物件，`outer2`返回該遍歷器物件的內部值。

從語法角度看，如果`yield`表達式後面跟的是一個遍歷器物件，需要在`yield`表達式後面加上星號，表明它返回的是一個遍歷器物件。這被稱為`yield*`表達式。

```javascript
let delegatedIterator = (function* () {
  yield 'Hello!';
  yield 'Bye!';
}());

let delegatingIterator = (function* () {
  yield 'Greetings!';
  yield* delegatedIterator;
  yield 'Ok, bye.';
}());

for(let value of delegatingIterator) {
  console.log(value);
}
// "Greetings!
// "Hello!"
// "Bye!"
// "Ok, bye."
```

上面代碼中，`delegatingIterator`是代理者，`delegatedIterator`是被代理者。由於`yield* delegatedIterator`語句得到的值，是一個遍歷器，所以要用星號表示。運行結果就是使用一個遍歷器，遍歷了多個 Generator 函數，有遞歸的效果。

`yield*`後面的 Generator 函數（沒有`return`語句時），等同於在 Generator 函數內部，部署一個`for...of`循環。

```javascript
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}

// 等同於

function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}
```

上面代碼說明，`yield*`後面的 Generator 函數（沒有`return`語句時），不過是`for...of`的一種簡寫形式，完全可以用後者替代前者。反之，在有`return`語句時，則需要用`var value = yield* iterator`的形式獲取`return`語句的值。

如果`yield*`後面跟著一個陣列，由於陣列原生支持遍歷器，因此就會遍曆陣列成員。

```javascript
function* gen(){
  yield* ["a", "b", "c"];
}

gen().next() // { value:"a", done:false }
```

上面代碼中，`yield`命令後面如果不加星號，返回的是整個陣列，加了星號就表示返回的是陣列的遍歷器物件。

實際上，任何數據結構只要有 Iterator 接口，就可以被`yield*`遍歷。

```javascript
let read = (function* () {
  yield 'hello';
  yield* 'hello';
})();

read.next().value // "hello"
read.next().value // "h"
```

上面代碼中，`yield`表達式返回整個字符串，`yield*`語句返回單個字符。因為字符串具有 Iterator 接口，所以被`yield*`遍歷。

如果被代理的 Generator 函數有`return`語句，那麼就可以向代理它的 Generator 函數返回數據。

```javascript
function* foo() {
  yield 2;
  yield 3;
  return "foo";
}

function* bar() {
  yield 1;
  var v = yield* foo();
  console.log("v: " + v);
  yield 4;
}

var it = bar();

it.next()
// {value: 1, done: false}
it.next()
// {value: 2, done: false}
it.next()
// {value: 3, done: false}
it.next();
// "v: foo"
// {value: 4, done: false}
it.next()
// {value: undefined, done: true}
```

上面代碼在第四次調用`next`方法的時候，屏幕上會有輸出，這是因為函數`foo`的`return`語句，向函數`bar`提供了返回值。

再看一個例子。

```javascript
function* genFuncWithReturn() {
  yield 'a';
  yield 'b';
  return 'The result';
}
function* logReturned(genObj) {
  let result = yield* genObj;
  console.log(result);
}

[...logReturned(genFuncWithReturn())]
// The result
// 值為 [ 'a', 'b' ]
```

上面代碼中，存在兩次遍歷。第一次是擴展運算符遍歷函數`logReturned`返回的遍歷器物件，第二次是`yield*`語句遍歷函數`genFuncWithReturn`返回的遍歷器物件。這兩次遍歷的效果是疊加的，最終表現為擴展運算符遍歷函數`genFuncWithReturn`返回的遍歷器物件。所以，最後的數據表達式得到的值等於`[ 'a', 'b' ]`。但是，函數`genFuncWithReturn`的`return`語句的返回值`The result`，會返回給函數`logReturned`內部的`result`變數，因此會有終端輸出。

`yield*`命令可以很方便地取出嵌套陣列的所有成員。

```javascript
function* iterTree(tree) {
  if (Array.isArray(tree)) {
    for(let i=0; i < tree.length; i++) {
      yield* iterTree(tree[i]);
    }
  } else {
    yield tree;
  }
}

const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];

for(let x of iterTree(tree)) {
  console.log(x);
}
// a
// b
// c
// d
// e
```

下面是一個稍微複雜的例子，使用`yield*`語句遍歷完全二叉樹。

```javascript
// 下面是二叉樹的構造函數，
// 三個參數分別是左樹、當前節點和右樹
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}

// 下面是中序（inorder）遍歷函數。
// 由於返回的是一個遍歷器，所以要用generator函數。
// 函數體內採用遞歸算法，所以左樹和右樹要用yield*遍歷
function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}

// 下面生成二叉樹
function make(array) {
  // 判斷是否為葉節點
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

// 遍歷二叉樹
var result = [];
for (let node of inorder(tree)) {
  result.push(node);
}

result
// ['a', 'b', 'c', 'd', 'e', 'f', 'g']
```

## 作為物件屬性的 Generator 函數

如果一個物件的屬性是 Generator 函數，可以簡寫成下面的形式。

```javascript
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```

上面代碼中，`myGeneratorMethod`屬性前面有一個星號，表示這個屬性是一個 Generator 函數。

它的完整形式如下，與上面的寫法是等價的。

```javascript
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```

## Generator 函數的`this`

Generator 函數總是返回一個遍歷器，ES6 規定這個遍歷器是 Generator 函數的實例，也繼承了 Generator 函數的`prototype`物件上的方法。

```javascript
function* g() {}

g.prototype.hello = function () {
  return 'hi!';
};

let obj = g();

obj instanceof g // true
obj.hello() // 'hi!'
```

上面代碼表明，Generator 函數`g`返回的遍歷器`obj`，是`g`的實例，而且繼承了`g.prototype`。但是，如果把`g`當作普通的構造函數，並不會生效，因為`g`返回的總是遍歷器物件，而不是`this`物件。

```javascript
function* g() {
  this.a = 11;
}

let obj = g();
obj.next();
obj.a // undefined
```

上面代碼中，Generator 函數`g`在`this`物件上面添加了一個屬性`a`，但是`obj`物件拿不到這個屬性。

Generator 函數也不能跟`new`命令一起用，會報錯。

```javascript
function* F() {
  yield this.x = 2;
  yield this.y = 3;
}

new F()
// TypeError: F is not a constructor
```

上面代碼中，`new`命令跟構造函數`F`一起使用，結果報錯，因為`F`不是構造函數。

那麼，有沒有辦法讓 Generator 函數返回一個正常的物件實例，既可以用`next`方法，又可以獲得正常的`this`？

下面是一個變通方法。首先，生成一個空物件，使用`call`方法綁定 Generator 函數內部的`this`。這樣，構造函數調用以後，這個空物件就是 Generator 函數的實例物件了。

```javascript
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var obj = {};
var f = F.call(obj);

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

obj.a // 1
obj.b // 2
obj.c // 3
```

上面代碼中，首先是`F`內部的`this`物件綁定`obj`物件，然後調用它，返回一個 Iterator 物件。這個物件執行三次`next`方法（因為`F`內部有兩個`yield`表達式），完成 F 內部所有代碼的運行。這時，所有內部屬性都綁定在`obj`物件上了，因此`obj`物件也就成了`F`的實例。

上面代碼中，執行的是遍歷器物件`f`，但是生成的物件實例是`obj`，有沒有辦法將這兩個物件統一呢？

一個辦法就是將`obj`換成`F.prototype`。

```javascript
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var f = F.call(F.prototype);

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```

再將`F`改成構造函數，就可以對它執行`new`命令了。

```javascript
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```

## 含義

### Generator 與狀態機

Generator 是實現狀態機的最佳結構。比如，下面的`clock`函數就是一個狀態機。

```javascript
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}
```

上面代碼的`clock`函數一共有兩種狀態（`Tick`和`Tock`），每運行一次，就改變一次狀態。這個函數如果用 Generator 實現，就是下面這樣。

```javascript
var clock = function* () {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```

上面的 Generator 實現與 ES5 實現對比，可以看到少了用來保存狀態的外部變數`ticking`，這樣就更簡潔，更安全（狀態不會被非法篡改）、更符合函數式編程的思想，在寫法上也更優雅。Generator 之所以可以不用外部變數保存狀態，是因為它本身就包含了一個狀態信息，即目前是否處於暫停態。

### Generator 與協程

協程（coroutine）是一種程序運行的方式，可以理解成“協作的線程”或“協作的函數”。協程既可以用單線程實現，也可以用多線程實現。前者是一種特殊的子例程，後者是一種特殊的線程。

**（1）協程與子例程的差異**

傳統的“子例程”（subroutine）採用堆棧式“後進先出”的執行方式，只有當調用的子函數完全執行完畢，才會結束執行父函數。協程與其不同，多個線程（單線程情況下，即多個函數）可以並行執行，但是只有一個線程（或函數）處於正在運行的狀態，其他線程（或函數）都處於暫停態（suspended），線程（或函數）之間可以交換執行權。也就是說，一個線程（或函數）執行到一半，可以暫停執行，將執行權交給另一個線程（或函數），等到稍後收回執行權的時候，再恢復執行。這種可以並行執行、交換執行權的線程（或函數），就稱為協程。

從實現上看，在內存中，子例程只使用一個棧（stack），而協程是同時存在多個棧，但只有一個棧是在運行狀態，也就是說，協程是以多佔用內存為代價，實現多任務的並行。

**（2）協程與普通線程的差異**

不難看出，協程適合用於多任務運行的環境。在這個意義上，它與普通的線程很相似，都有自己的執行上下文、可以分享全局變數。它們的不同之處在於，同一時間可以有多個線程處於運行狀態，但是運行的協程只能有一個，其他協程都處於暫停狀態。此外，普通的線程是搶先式的，到底哪個線程優先得到資源，必須由運行環境決定，但是協程是合作式的，執行權由協程自己分配。

由於 JavaScript 是單線程語言，只能保持一個調用棧。引入協程以後，每個任務可以保持自己的調用棧。這樣做的最大好處，就是拋出錯誤的時候，可以找到原始的調用棧。不至於像異步操作的回調函數那樣，一旦出錯，原始的調用棧早就結束。

Generator 函數是 ES6 對協程的實現，但屬於不完全實現。Generator 函數被稱為“半協程”（semi-coroutine），意思是只有 Generator 函數的調用者，才能將程序的執行權還給 Generator 函數。如果是完全執行的協程，任何函數都可以讓暫停的協程繼續執行。

如果將 Generator 函數當作協程，完全可以將多個需要互相協作的任務寫成 Generator 函數，它們之間使用`yield`表達式交換控制權。

### Generator 與上下文

JavaScript 代碼運行時，會產生一個全局的上下文環境（context，又稱運行環境），包含了當前所有的變數和物件。然後，執行函數（或塊級代碼）的時候，又會在當前上下文環境的上層，產生一個函數運行的上下文，變成當前（active）的上下文，由此形成一個上下文環境的堆棧（context stack）。

這個堆棧是“後進先出”的數據結構，最後產生的上下文環境首先執行完成，退出堆棧，然後再執行完成它下層的上下文，直至所有代碼執行完成，堆棧清空。

Generator 函數不是這樣，它執行產生的上下文環境，一旦遇到`yield`命令，就會暫時退出堆棧，但是並不消失，裡面的所有變數和物件會凍結在當前狀態。等到對它執行`next`命令時，這個上下文環境又會重新加入調用棧，凍結的變數和物件恢復執行。

```javascript
function* gen() {
  yield 1;
  return 2;
}

let g = gen();

console.log(
  g.next().value,
  g.next().value,
);
```

上面代碼中，第一次執行`g.next()`時，Generator 函數`gen`的上下文會加入堆棧，即開始運行`gen`內部的代碼。等遇到`yield 1`時，`gen`上下文退出堆棧，內部狀態凍結。第二次執行`g.next()`時，`gen`上下文重新加入堆棧，變成當前的上下文，重新恢復執行。

## 應用

Generator 可以暫停函數執行，返回任意表達式的值。這種特點使得 Generator 有多種應用場景。

### （1）異步操作的同步化表達

Generator 函數的暫停執行的效果，意味著可以把異步操作寫在`yield`表達式裡面，等到調用`next`方法時再往後執行。這實際上等同於不需要寫回調函數了，因為異步操作的後續操作可以放在`yield`表達式下面，反正要等到調用`next`方法時再執行。所以，Generator 函數的一個重要實際意義就是用來處理異步操作，改寫回調函數。

```javascript
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
var loader = loadUI();
// 加載UI
loader.next()

// 卸載UI
loader.next()
```

上面代碼中，第一次調用`loadUI`函數時，該函數不會執行，僅返回一個遍歷器。下一次對該遍歷器調用`next`方法，則會顯示`Loading`界面（`showLoadingScreen`），並且異步加載數據（`loadUIDataAsynchronously`）。等到數據加載完成，再一次使用`next`方法，則會隱藏`Loading`界面。可以看到，這種寫法的好處是所有`Loading`界面的邏輯，都被封裝在一個函數，按部就班非常清晰。

Ajax 是典型的異步操作，通過 Generator 函數部署 Ajax 操作，可以用同步的方式表達。

```javascript
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}

var it = main();
it.next();
```

上面代碼的`main`函數，就是通過 Ajax 操作獲取數據。可以看到，除了多了一個`yield`，它幾乎與同步操作的寫法完全一樣。注意，`makeAjaxCall`函數中的`next`方法，必須加上`response`參數，因為`yield`表達式，本身是沒有值的，總是等於`undefined`。

下面是另一個例子，通過 Generator 函數逐行讀取文本文件。

```javascript
function* numbers() {
  let file = new FileReader("numbers.txt");
  try {
    while(!file.eof) {
      yield parseInt(file.readLine(), 10);
    }
  } finally {
    file.close();
  }
}
```

上面代碼打開文本文件，使用`yield`表達式可以手動逐行讀取文件。

### （2）控制流管理

如果有一個多步操作非常耗時，採用回調函數，可能會寫成下面這樣。

```javascript
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // Do something with value4
      });
    });
  });
});
```

採用 Promise 改寫上面的代碼。

```javascript
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(function (value4) {
    // Do something with value4
  }, function (error) {
    // Handle any error from step1 through step4
  })
  .done();
```

上面代碼已經把回調函數，改成了直線執行的形式，但是加入了大量 Promise 的語法。Generator 函數可以進一步改善代碼運行流程。

```javascript
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}
```

然後，使用一個函數，按次序自動執行所有步驟。

```javascript
scheduler(longRunningTask(initialValue));

function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函數未結束，就繼續調用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
```

注意，上面這種做法，只適合同步操作，即所有的`task`都必須是同步的，不能有異步操作。因為這裡的代碼一得到返回值，就繼續往下執行，沒有判斷異步操作何時完成。如果要控制異步的操作流程，詳見後面的《異步操作》一章。

下面，利用`for...of`循環會自動依次執行`yield`命令的特性，提供一種更一般的控制流管理的方法。

```javascript
let steps = [step1Func, step2Func, step3Func];

function* iterateSteps(steps){
  for (var i=0; i< steps.length; i++){
    var step = steps[i];
    yield step();
  }
}
```

上面代碼中，陣列`steps`封裝了一個任務的多個步驟，Generator 函數`iterateSteps`則是依次為這些步驟加上`yield`命令。

將任務分解成步驟之後，還可以將專案分解成多個依次執行的任務。

```javascript
let jobs = [job1, job2, job3];

function* iterateJobs(jobs){
  for (var i=0; i< jobs.length; i++){
    var job = jobs[i];
    yield* iterateSteps(job.steps);
  }
}
```

上面代碼中，陣列`jobs`封裝了一個專案的多個任務，Generator 函數`iterateJobs`則是依次為這些任務加上`yield*`命令。

最後，就可以用`for...of`循環一次性依次執行所有任務的所有步驟。

```javascript
for (var step of iterateJobs(jobs)){
  console.log(step.id);
}
```

再次提醒，上面的做法只能用於所有步驟都是同步操作的情況，不能有異步操作的步驟。如果想要依次執行異步的步驟，必須使用後面的《異步操作》一章介紹的方法。

`for...of`的本質是一個`while`循環，所以上面的代碼實質上執行的是下面的邏輯。

```javascript
var it = iterateJobs(jobs);
var res = it.next();

while (!res.done){
  var result = res.value;
  // ...
  res = it.next();
}
```

### （3）部署 Iterator 接口

利用 Generator 函數，可以在任意物件上部署 Iterator 接口。

```javascript
function* iterEntries(obj) {
  let keys = Object.keys(obj);
  for (let i=0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}

let myObj = { foo: 3, bar: 7 };

for (let [key, value] of iterEntries(myObj)) {
  console.log(key, value);
}

// foo 3
// bar 7
```

上述代碼中，`myObj`是一個普通物件，通過`iterEntries`函數，就有了 Iterator 接口。也就是說，可以在任意物件上部署`next`方法。

下面是一個對陣列部署 Iterator 接口的例子，儘管陣列原生具有這個接口。

```javascript
function* makeSimpleGenerator(array){
  var nextIndex = 0;

  while(nextIndex < array.length){
    yield array[nextIndex++];
  }
}

var gen = makeSimpleGenerator(['yo', 'ya']);

gen.next().value // 'yo'
gen.next().value // 'ya'
gen.next().done  // true
```

### （4）作為數據結構

Generator 可以看作是數據結構，更確切地說，可以看作是一個陣列結構，因為 Generator 函數可以返回一系列的值，這意味著它可以對任意表達式，提供類似陣列的接口。

```javascript
function* doStuff() {
  yield fs.readFile.bind(null, 'hello.txt');
  yield fs.readFile.bind(null, 'world.txt');
  yield fs.readFile.bind(null, 'and-such.txt');
}
```

上面代碼就是依次返回三個函數，但是由於使用了 Generator 函數，導致可以像處理陣列那樣，處理這三個返回的函數。

```javascript
for (task of doStuff()) {
  // task是一個函數，可以像回調函數那樣使用它
}
```

實際上，如果用 ES5 表達，完全可以用陣列模擬 Generator 的這種用法。

```javascript
function doStuff() {
  return [
    fs.readFile.bind(null, 'hello.txt'),
    fs.readFile.bind(null, 'world.txt'),
    fs.readFile.bind(null, 'and-such.txt')
  ];
}
```

上面的函數，可以用一模一樣的`for...of`循環處理！兩相一比較，就不難看出 Generator 使得數據或者操作，具備了類似陣列的接口。
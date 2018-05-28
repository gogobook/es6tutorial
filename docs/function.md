# 函數的擴展

## 函數參數的默認值

### 基本用法

ES6 之前，不能直接為函數的參數指定默認值，只能採用變通的方法。

```javascript
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World
```

上面代碼檢查函數`log`的參數`y`有沒有賦值，如果沒有，則指定默認值為`World`。這種寫法的缺點在於，如果參數`y`賦值了，但是對應的布爾值為`false`，則該賦值不起作用。就像上面代碼的最後一行，參數`y`等於空字符，結果被改為默認值。

為了避免這個問題，通常需要先判斷一下參數`y`是否被賦值，如果沒有，再等於默認值。

```javascript
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 允許為函數的參數設置默認值，即直接寫在參數定義的後面。

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

可以看到，ES6 的寫法比 ES5 簡潔許多，而且非常自然。下面是另一個例子。

```javascript
function Point(x = 0, y = 0) {
  this.x = x;
  this.y = y;
}

const p = new Point();
p // { x: 0, y: 0 }
```

除了簡潔，ES6 的寫法還有兩個好處：首先，閱讀代碼的人，可以立刻意識到哪些參數是可以省略的，不用查看函數體或文檔；其次，有利於將來的代碼優化，即使未來的版本在對外接口中，徹底拿掉這個參數，也不會導致以前的代碼無法運行。

參數變數是默認聲明的，所以不能用`let`或`const`再次聲明。

```javascript
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
```

上面代碼中，參數變數`x`是默認聲明的，在函數體中，不能用`let`或`const`再次聲明，否則會報錯。

使用參數默認值時，函數不能有同名參數。

```javascript
// 不報錯
function foo(x, x, y) {
  // ...
}

// 報錯
function foo(x, x, y = 1) {
  // ...
}
// SyntaxError: Duplicate parameter name not allowed in this context
```

另外，一個容易忽略的地方是，參數默認值不是傳值的，而是每次都重新計算默認值表達式的值。也就是說，參數默認值是惰性求值的。

```javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```

上面代碼中，參數`p`的默認值是`x + 1`。這時，每次調用函數`foo`，都會重新計算`x + 1`，而不是默認`p`等於 100。

### 與解構賦值默認值結合使用

參數默認值可以與解構賦值的默認值，結合起來使用。

```javascript
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
```

上面代碼只使用了物件的解構賦值默認值，沒有使用函數參數的默認值。只有當函數`foo`的參數是一個物件時，變數`x`和`y`才會通過解構賦值生成。如果函數`foo`調用時沒提供參數，變數`x`和`y`就不會生成，從而報錯。通過提供函數參數的默認值，就可以避免這種情況。

```javascript
function foo({x, y = 5} = {}) {
  console.log(x, y);
}

foo() // undefined 5
```

上面代碼指定，如果沒有提供參數，函數`foo`的參數默認為一個空物件。

下面是另一個解構賦值默認值的例子。

```javascript
function fetch(url, { body = '', method = 'GET', headers = {} }) {
  console.log(method);
}

fetch('http://example.com', {})
// "GET"

fetch('http://example.com')
// 報錯
```

上面代碼中，如果函數`fetch`的第二個參數是一個物件，就可以為它的三個屬性設置默認值。這種寫法不能省略第二個參數，如果結合函數參數的默認值，就可以省略第二個參數。這時，就出現了雙重默認值。

```javascript
function fetch(url, { body = '', method = 'GET', headers = {} } = {}) {
  console.log(method);
}

fetch('http://example.com')
// "GET"
```

上面代碼中，函數`fetch`沒有第二個參數時，函數參數的默認值就會生效，然後才是解構賦值的默認值生效，變數`method`才會取到默認值`GET`。

作為練習，請問下面兩種寫法有什麼差別？

```javascript
// 寫法一
function m1({x = 0, y = 0} = {}) {
  return [x, y];
}

// 寫法二
function m2({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
```

上面兩種寫法都對函數的參數設定了默認值，區別是寫法一函數參數的默認值是空物件，但是設置了物件解構賦值的默認值；寫法二函數參數的默認值是一個有具體屬性的物件，但是沒有設置物件解構賦值的默認值。

```javascript
// 函數沒有參數的情況
m1() // [0, 0]
m2() // [0, 0]

// x 和 y 都有值的情況
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

// x 有值，y 無值的情況
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

// x 和 y 都無值的情況
m1({}) // [0, 0];
m2({}) // [undefined, undefined]

m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]
```

### 參數默認值的位置

通常情況下，定義了默認值的參數，應該是函數的尾參數。因為這樣比較容易看出來，到底省略了哪些參數。如果非尾部的參數設置默認值，實際上這個參數是沒法省略的。

```javascript
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 報錯
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 報錯
f(1, undefined, 2) // [1, 5, 2]
```

上面代碼中，有默認值的參數都不是尾參數。這時，無法只省略該參數，而不省略它後面的參數，除非顯式輸入`undefined`。

如果傳入`undefined`，將觸發該參數等於默認值，`null`則沒有這個效果。

```javascript
function foo(x = 5, y = 6) {
  console.log(x, y);
}

foo(undefined, null)
// 5 null
```

上面代碼中，`x`參數對應`undefined`，結果觸發了默認值，`y`參數等於`null`，就沒有觸發默認值。

### 函數的 length 屬性

指定了默認值以後，函數的`length`屬性，將返回沒有指定默認值的參數個數。也就是說，指定了默認值後，`length`屬性將失真。

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```

上面代碼中，`length`屬性的返回值，等於函數的參數個數減去指定了默認值的參數個數。比如，上面最後一個函數，定義了 3 個參數，其中有一個參數`c`指定了默認值，因此`length`屬性等於`3`減去`1`，最後得到`2`。

這是因為`length`屬性的含義是，該函數預期傳入的參數個數。某個參數指定默認值以後，預期傳入的參數個數就不包括這個參數了。同理，後文的 rest 參數也不會計入`length`屬性。

```javascript
(function(...args) {}).length // 0
```

如果設置了默認值的參數不是尾參數，那麼`length`屬性也不再計入後面的參數了。

```javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

### 作用域

一旦設置了參數的默認值，函數進行聲明初始化時，參數會形成一個單獨的作用域（context）。等到初始化結束，這個作用域就會消失。這種語法行為，在不設置參數默認值時，是不會出現的。

```javascript
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2
```

上面代碼中，參數`y`的默認值等於變數`x`。調用函數`f`時，參數形成一個單獨的作用域。在這個作用域裡面，默認值變數`x`指向第一個參數`x`，而不是全局變數`x`，所以輸出是`2`。

再看下面的例子。

```javascript
let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1
```

上面代碼中，函數`f`調用時，參數`y = x`形成一個單獨的作用域。這個作用域裡面，變數`x`本身沒有定義，所以指向外層的全局變數`x`。函數調用時，函數體內部的局部變數`x`影響不到默認值變數`x`。

如果此時，全局變數`x`不存在，就會報錯。

```javascript
function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // ReferenceError: x is not defined
```

下面這樣寫，也會報錯。

```javascript
var x = 1;

function foo(x = x) {
  // ...
}

foo() // ReferenceError: x is not defined
```

上面代碼中，參數`x = x`形成一個單獨作用域。實際執行的是`let x = x`，由於暫時性死區的原因，這行代碼會報錯”x 未定義“。

如果參數的默認值是一個函數，該函數的作用域也遵守這個規則。請看下面的例子。

```javascript
let foo = 'outer';

function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar(); // outer
```

上面代碼中，函數`bar`的參數`func`的默認值是一個匿名函數，返回值為變數`foo`。函數參數形成的單獨作用域裡面，並沒有定義變數`foo`，所以`foo`指向外層的全局變數`foo`，因此輸出`outer`。

如果寫成下面這樣，就會報錯。

```javascript
function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar() // ReferenceError: foo is not defined
```

上面代碼中，匿名函數裡面的`foo`指向函數外層，但是函數外層並沒有聲明變數`foo`，所以就報錯了。

下面是一個更複雜的例子。

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y();
  console.log(x);
}

foo() // 3
x // 1
```

上面代碼中，函數`foo`的參數形成一個單獨作用域。這個作用域裡面，首先聲明了變數`x`，然後聲明了變數`y`，`y`的默認值是一個匿名函數。這個匿名函數內部的變數`x`，指向同一個作用域的第一個參數`x`。函數`foo`內部又聲明了一個內部變數`x`，該變數與第一個參數`x`由於不是同一個作用域，所以不是同一個變數，因此執行`y`後，內部變數`x`和外部全局變數`x`的值都沒變。

如果將`var x = 3`的`var`去除，函數`foo`的內部變數`x`就指向第一個參數`x`，與匿名函數內部的`x`是一致的，所以最後輸出的就是`2`，而外層的全局變數`x`依然不受影響。

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}

foo() // 2
x // 1
```

### 應用

利用參數默認值，可以指定某一個參數不得省略，如果省略就拋出一個錯誤。

```javascript
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
// Error: Missing parameter
```

上面代碼的`foo`函數，如果調用的時候沒有參數，就會調用默認值`throwIfMissing`函數，從而拋出一個錯誤。

從上面代碼還可以看到，參數`mustBeProvided`的默認值等於`throwIfMissing`函數的運行結果（注意涵數名`throwIfMissing`之後有一對圓括號），這表明參數的默認值不是在定義時執行，而是在運行時執行。如果參數已經賦值，默認值中的函數就不會運行。

另外，可以將參數默認值設為`undefined`，表明這個參數是可以省略的。

```javascript
function foo(optional = undefined) { ··· }
```

## rest 參數

ES6 引入 rest 參數（形式為`...變數名`），用於獲取函數的多餘參數，這樣就不需要使用`arguments`物件了。rest 參數搭配的變數是一個陣列，該變數將多餘的參數放入陣列中。

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```

上面代碼的`add`函數是一個求和函數，利用 rest 參數，可以向該函數傳入任意數目的參數。

下面是一個 rest 參數代替`arguments`變數的例子。

```javascript
// arguments變數的寫法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest參數的寫法
const sortNumbers = (...numbers) => numbers.sort();
```

上面代碼的兩種寫法，比較後可以發現，rest 參數的寫法更自然也更簡潔。

`arguments`物件不是陣列，而是一個類似陣列的物件。所以為了使用陣列的方法，必須使用`Array.prototype.slice.call`先將其轉為陣列。rest 參數就不存在這個問題，它就是一個真正的陣列，陣列特有的方法都可以使用。下面是一個利用 rest 參數改寫陣列`push`方法的例子。

```javascript
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)
```

注意，rest 參數之後不能再有其他參數（即只能是最後一個參數），否則會報錯。

```javascript
// 報錯
function f(a, ...b, c) {
  // ...
}
```

函數的`length`屬性，不包括 rest 參數。

```javascript
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

## 嚴格模式

從 ES5 開始，函數內部可以設定為嚴格模式。

```javascript
function doSomething(a, b) {
  'use strict';
  // code
}
```

ES2016 做了一點修改，規定只要函數參數使用了默認值、解構賦值、或者擴展運算符，那麼函數內部就不能顯式設定為嚴格模式，否則會報錯。

```javascript
// 報錯
function doSomething(a, b = a) {
  'use strict';
  // code
}

// 報錯
const doSomething = function ({a, b}) {
  'use strict';
  // code
};

// 報錯
const doSomething = (...a) => {
  'use strict';
  // code
};

const obj = {
  // 報錯
  doSomething({a, b}) {
    'use strict';
    // code
  }
};
```

這樣規定的原因是，函數內部的嚴格模式，同時適用於函數體和函數參數。但是，函數執行的時候，先執行函數參數，然後再執行函數體。這樣就有一個不合理的地方，只有從函數體之中，才能知道參數是否應該以嚴格模式執行，但是參數卻應該先於函數體執行。

```javascript
// 報錯
function doSomething(value = 070) {
  'use strict';
  return value;
}
```

上面代碼中，參數`value`的默認值是八進制數`070`，但是嚴格模式下不能用前綴`0`表示八進制，所以應該報錯。但是實際上，JavaScript 引擎會先成功執行`value = 070`，然後進入函數體內部，發現需要用嚴格模式執行，這時才會報錯。

雖然可以先解析函數體代碼，再執行參數代碼，但是這樣無疑就增加了複雜性。因此，標準索性禁止了這種用法，只要參數使用了默認值、解構賦值、或者擴展運算符，就不能顯式指定嚴格模式。

兩種方法可以規避這種限制。第一種是設定全局性的嚴格模式，這是合法的。

```javascript
'use strict';

function doSomething(a, b = a) {
  // code
}
```

第二種是把函數包在一個無參數的立即執行函數裡面。

```javascript
const doSomething = (function () {
  'use strict';
  return function(value = 42) {
    return value;
  };
}());
```

## name 屬性

函數的`name`屬性，返回該函數的函數名。

```javascript
function foo() {}
foo.name // "foo"
```

這個屬性早就被瀏覽器廣泛支持，但是直到 ES6，才將其寫入了標準。

需要注意的是，ES6 對這個屬性的行為做出了一些修改。如果將一個匿名函數賦值給一個變數，ES5 的`name`屬性，會返回空字符串，而 ES6 的`name`屬性會返回實際的函數名。

```javascript
var f = function () {};

// ES5
f.name // ""

// ES6
f.name // "f"
```

上面代碼中，變數`f`等於一個匿名函數，ES5 和 ES6 的`name`屬性返回的值不一樣。

如果將一個具名函數賦值給一個變數，則 ES5 和 ES6 的`name`屬性都返回這個具名函數原本的名字。

```javascript
const bar = function baz() {};

// ES5
bar.name // "baz"

// ES6
bar.name // "baz"
```

`Function`構造函數返回的函數實例，`name`屬性的值為`anonymous`。

```javascript
(new Function).name // "anonymous"
```

`bind`返回的函數，`name`屬性值會加上`bound`前綴。

```javascript
function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

## 箭頭函數

### 基本用法

ES6 允許使用“箭頭”（`=>`）定義函數。

```javascript
var f = v => v;

// 等同於
var f = function (v) {
  return v;
};
```

如果箭頭函數不需要參數或需要多個參數，就使用一個圓括號代表參數部分。

```javascript
var f = () => 5;
// 等同於
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同於
var sum = function(num1, num2) {
  return num1 + num2;
};
```

如果箭頭函數的代碼塊部分多於一條語句，就要使用大括號將它們括起來，並且使用`return`語句返回。

```javascript
var sum = (num1, num2) => { return num1 + num2; }
```

由於大括號被解釋為代碼塊，所以如果箭頭函數直接返回一個物件，必須在物件外面加上括號，否則會報錯。

```javascript
// 報錯
let getTempItem = id => { id: id, name: "Temp" };

// 不報錯
let getTempItem = id => ({ id: id, name: "Temp" });
```

下面是一種特殊情況，雖然可以運行，但會得到錯誤的結果。

```javascript
let foo = () => { a: 1 };
foo() // undefined
```

上面代碼中，原始意圖是返回一個物件`{ a: 1 }`，但是由於引擎認為大括號是代碼塊，所以執行了一行語句`a: 1`。這時，`a`可以被解釋為語句的標籤，因此實際執行的語句是`1;`，然後函數就結束了，沒有返回值。

如果箭頭函數只有一行語句，且不需要返回值，可以採用下面的寫法，就不用寫大括號了。

```javascript
let fn = () => void doesNotReturn();
```

箭頭函數可以與變數解構結合使用。

```javascript
const full = ({ first, last }) => first + ' ' + last;

// 等同於
function full(person) {
  return person.first + ' ' + person.last;
}
```

箭頭函數使得表達更加簡潔。

```javascript
const isEven = n => n % 2 == 0;
const square = n => n * n;
```

上面代碼只用了兩行，就定義了兩個簡單的工具函數。如果不用箭頭函數，可能就要佔用多行，而且還不如現在這樣寫醒目。

箭頭函數的一個用處是簡化回調函數。

```javascript
// 正常函數寫法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭頭函數寫法
[1,2,3].map(x => x * x);
```

另一個例子是

```javascript
// 正常函數寫法
var result = values.sort(function (a, b) {
  return a - b;
});

// 箭頭函數寫法
var result = values.sort((a, b) => a - b);
```

下面是 rest 參數與箭頭函數結合的例子。

```javascript
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

### 使用注意點

箭頭函數有幾個使用注意點。

（1）函數體內的`this`物件，就是定義時所在的物件，而不是使用時所在的物件。

（2）不可以當作構造函數，也就是說，不可以使用`new`命令，否則會拋出一個錯誤。

（3）不可以使用`arguments`物件，該物件在函數體內不存在。如果要用，可以用 rest 參數代替。

（4）不可以使用`yield`命令，因此箭頭函數不能用作 Generator 函數。

上面四點中，第一點尤其值得注意。`this`物件的指向是可變的，但是在箭頭函數中，它是固定的。

```javascript
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```

上面代碼中，`setTimeout`的參數是一個箭頭函數，這個箭頭函數的定義生效是在`foo`函數生成時，而它的真正執行要等到 100 毫秒後。如果是普通函數，執行時`this`應該指向全局物件`window`，這時應該輸出`21`。但是，箭頭函數導致`this`總是指向函數定義生效時所在的物件（本例是`{id: 42}`），所以輸出的是`42`。

箭頭函數可以讓`setTimeout`裡面的`this`，綁定定義時所在的作用域，而不是指向運行時所在的作用域。下面是另一個例子。

```javascript
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭頭函數
  setInterval(() => this.s1++, 1000);
  // 普通函數
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
```

上面代碼中，`Timer`函數內部設置了兩個定時器，分別使用了箭頭函數和普通函數。前者的`this`綁定定義時所在的作用域（即`Timer`函數），後者的`this`指向運行時所在的作用域（即全局物件）。所以，3100 毫秒之後，`timer.s1`被更新了 3 次，而`timer.s2`一次都沒更新。

箭頭函數可以讓`this`指向固定化，這種特性很有利於封裝回調函數。下面是一個例子，DOM 事件的回調函數封裝在一個物件裡面。

```javascript
var handler = {
  id: '123456',

  init: function() {
    document.addEventListener('click',
      event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
};
```

上面代碼的`init`方法中，使用了箭頭函數，這導致這個箭頭函數裡面的`this`，總是指向`handler`物件。否則，回調函數運行時，`this.doSomething`這一行會報錯，因為此時`this`指向`document`物件。

`this`指向的固定化，並不是因為箭頭函數內部有綁定`this`的機制，實際原因是箭頭函數根本沒有自己的`this`，導致內部的`this`就是外層代碼塊的`this`。正是因為它沒有`this`，所以也就不能用作構造函數。

所以，箭頭函數轉成 ES5 的代碼如下。

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

上面代碼中，轉換後的 ES5 版本清楚地說明了，箭頭函數裡面根本沒有自己的`this`，而是引用外層的`this`。

請問下面的代碼之中有幾個`this`？

```javascript
function foo() {
  return () => {
    return () => {
      return () => {
        console.log('id:', this.id);
      };
    };
  };
}

var f = foo.call({id: 1});

var t1 = f.call({id: 2})()(); // id: 1
var t2 = f().call({id: 3})(); // id: 1
var t3 = f()().call({id: 4}); // id: 1
```

上面代碼之中，只有一個`this`，就是函數`foo`的`this`，所以`t1`、`t2`、`t3`都輸出同樣的結果。因為所有的內層函數都是箭頭函數，都沒有自己的`this`，它們的`this`其實都是最外層`foo`函數的`this`。

除了`this`，以下三個變數在箭頭函數之中也是不存在的，指向外層函數的對應變數：`arguments`、`super`、`new.target`。

```javascript
function foo() {
  setTimeout(() => {
    console.log('args:', arguments);
  }, 100);
}

foo(2, 4, 6, 8)
// args: [2, 4, 6, 8]
```

上面代碼中，箭頭函數內部的變數`arguments`，其實是函數`foo`的`arguments`變數。

另外，由於箭頭函數沒有自己的`this`，所以當然也就不能用`call()`、`apply()`、`bind()`這些方法去改變`this`的指向。

```javascript
(function() {
  return [
    (() => this.x).bind({ x: 'inner' })()
  ];
}).call({ x: 'outer' });
// ['outer']
```

上面代碼中，箭頭函數沒有自己的`this`，所以`bind`方法無效，內部的`this`指向外部的`this`。

長期以來，JavaScript 語言的`this`物件一直是一個令人頭痛的問題，在物件方法中使用`this`，必須非常小心。箭頭函數”綁定”`this`，很大程度上解決了這個困擾。

### 嵌套的箭頭函數

箭頭函數內部，還可以再使用箭頭函數。下面是一個 ES5 語法的多重嵌套函數。

```javascript
function insert(value) {
  return {into: function (array) {
    return {after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }};
  }};
}

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```

上面這個函數，可以使用箭頭函數改寫。

```javascript
let insert = (value) => ({into: (array) => ({after: (afterValue) => {
  array.splice(array.indexOf(afterValue) + 1, 0, value);
  return array;
}})});

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```

下面是一個部署管道機制（pipeline）的例子，即前一個函數的輸出是後一個函數的輸入。

```javascript
const pipeline = (...funcs) =>
  val => funcs.reduce((a, b) => b(a), val);

const plus1 = a => a + 1;
const mult2 = a => a * 2;
const addThenMult = pipeline(plus1, mult2);

addThenMult(5)
// 12
```

如果覺得上面的寫法可讀性比較差，也可以採用下面的寫法。

```javascript
const plus1 = a => a + 1;
const mult2 = a => a * 2;

mult2(plus1(5))
// 12
```

箭頭函數還有一個功能，就是可以很方便地改寫 λ 演算。

```javascript
// λ演算的寫法
fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))

// ES6的寫法
var fix = f => (x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));
```

上面兩種寫法，幾乎是一一對應的。由於 λ 演算對於計算機科學非常重要，這使得我們可以用 ES6 作為替代工具，探索計算機科學。

## 雙冒號運算符

箭頭函數可以綁定`this`物件，大大減少了顯式綁定`this`物件的寫法（`call`、`apply`、`bind`）。但是，箭頭函數並不適用於所有場合，所以現在有一個[提案](https://github.com/zenparsing/es-function-bind)，提出了“函數綁定”（function bind）運算符，用來取代`call`、`apply`、`bind`調用。

函數綁定運算符是並排的兩個冒號（`::`），雙冒號左邊是一個物件，右邊是一個函數。該運算符會自動將左邊的物件，作為上下文環境（即`this`物件），綁定到右邊的函數上面。

```javascript
foo::bar;
// 等同於
bar.bind(foo);

foo::bar(...arguments);
// 等同於
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}
```

如果雙冒號左邊為空，右邊是一個物件的方法，則等於將該方法綁定在該物件上面。

```javascript
var method = obj::obj.foo;
// 等同於
var method = ::obj.foo;

let log = ::console.log;
// 等同於
var log = console.log.bind(console);
```

如果雙冒號運算符的運算結果，還是一個物件，就可以採用鏈式寫法。

```javascript
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));
```

## 尾調用優化

### 什麼是尾調用？

尾調用（Tail Call）是函數式編程的一個重要概念，本身非常簡單，一句話就能說清楚，就是指某個函數的最後一步是調用另一個函數。

```javascript
function f(x){
  return g(x);
}
```

上面代碼中，函數`f`的最後一步是調用函數`g`，這就叫尾調用。

以下三種情況，都不屬於尾調用。

```javascript
// 情況一
function f(x){
  let y = g(x);
  return y;
}

// 情況二
function f(x){
  return g(x) + 1;
}

// 情況三
function f(x){
  g(x);
}
```

上面代碼中，情況一是調用函數`g`之後，還有賦值操作，所以不屬於尾調用，即使語義完全一樣。情況二也屬於調用後還有操作，即使寫在一行內。情況三等同於下面的代碼。

```javascript
function f(x){
  g(x);
  return undefined;
}
```

尾調用不一定出現在函數尾部，只要是最後一步操作即可。

```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

上面代碼中，函數`m`和`n`都屬於尾調用，因為它們都是函數`f`的最後一步操作。

### 尾調用優化

尾調用之所以與其他調用不同，就在於它的特殊的調用位置。

我們知道，函數調用會在內存形成一個“調用記錄”，又稱“調用幀”（call frame），保存調用位置和內部變數等信息。如果在函數`A`的內部調用函數`B`，那麼在`A`的調用幀上方，還會形成一個`B`的調用幀。等到`B`運行結束，將結果返回到`A`，`B`的調用幀才會消失。如果函數`B`內部還調用函數`C`，那就還有一個`C`的調用幀，以此類推。所有的調用幀，就形成一個“調用棧”（call stack）。

尾調用由於是函數的最後一步操作，所以不需要保留外層函數的調用幀，因為調用位置、內部變數等信息都不會再用到了，只要直接用內層函數的調用幀，取代外層函數的調用幀就可以了。

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同於
function f() {
  return g(3);
}
f();

// 等同於
g(3);
```

上面代碼中，如果函數`g`不是尾調用，函數`f`就需要保存內部變數`m`和`n`的值、`g`的調用位置等信息。但由於調用`g`之後，函數`f`就結束了，所以執行到最後一步，完全可以刪除`f(x)`的調用幀，只保留`g(3)`的調用幀。

這就叫做“尾調用優化”（Tail call optimization），即只保留內層函數的調用幀。如果所有函數都是尾調用，那麼完全可以做到每次執行時，調用幀只有一項，這將大大節省內存。這就是“尾調用優化”的意義。

注意，只有不再用到外層函數的內部變數，內層函數的調用幀才會取代外層函數的調用幀，否則就無法進行“尾調用優化”。

```javascript
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  return inner(a);
}
```

上面的函數不會進行尾調用優化，因為內層函數`inner`用到了外層函數`addOne`的內部變數`one`。

### 尾遞歸

函數調用自身，稱為遞歸。如果尾調用自身，就稱為尾遞歸。

遞歸非常耗費內存，因為需要同時保存成千上百個調用幀，很容易發生“棧溢出”錯誤（stack overflow）。但對於尾遞歸來說，由於只存在一個調用幀，所以永遠不會發生“棧溢出”錯誤。

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
```

上面代碼是一個階乘函數，計算`n`的階乘，最多需要保存`n`個調用記錄，複雜度 O(n) 。

如果改寫成尾遞歸，只保留一個調用記錄，複雜度 O(1) 。

```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

還有一個比較著名的例子，就是計算 Fibonacci 數列，也能充分說明尾遞歸優化的重要性。

非尾遞歸的 Fibonacci 數列實現如下。

```javascript
function Fibonacci (n) {
  if ( n <= 1 ) {return 1};

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}

Fibonacci(10) // 89
Fibonacci(100) // 堆棧溢出
Fibonacci(500) // 堆棧溢出
```

尾遞歸優化過的 Fibonacci 數列實現如下。

```javascript
function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
  if( n <= 1 ) {return ac2};

  return Fibonacci2 (n - 1, ac2, ac1 + ac2);
}

Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
Fibonacci2(10000) // Infinity
```

由此可見，“尾調用優化”對遞歸操作意義重大，所以一些函數式編程語言將其寫入了語言規格。ES6 是如此，第一次明確規定，所有 ECMAScript 的實現，都必須部署“尾調用優化”。這就是說，ES6 中只要使用尾遞歸，就不會發生棧溢出，相對節省內存。

### 遞歸函數的改寫

尾遞歸的實現，往往需要改寫遞歸函數，確保最後一步只調用自身。做到這一點的方法，就是把所有用到的內部變數改寫成函數的參數。比如上面的例子，階乘函數 factorial 需要用到一個中間變數`total`，那就把這個中間變數改寫成函數的參數。這樣做的缺點就是不太直觀，第一眼很難看出來，為什麼計算`5`的階乘，需要傳入兩個參數`5`和`1`？

兩個方法可以解決這個問題。方法一是在尾遞歸函數之外，再提供一個正常形式的函數。

```javascript
function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

function factorial(n) {
  return tailFactorial(n, 1);
}

factorial(5) // 120
```

上面代碼通過一個正常形式的階乘函數`factorial`，調用尾遞歸函數`tailFactorial`，看起來就正常多了。

函數式編程有一個概念，叫做柯裡化（currying），意思是將多參數的函數轉換成單參數的形式。這裡也可以使用柯裡化。

```javascript
function currying(fn, n) {
  return function (m) {
    return fn.call(this, m, n);
  };
}

function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

const factorial = currying(tailFactorial, 1);

factorial(5) // 120
```

上面代碼通過柯裡化，將尾遞歸函數`tailFactorial`變為只接受一個參數的`factorial`。

第二種方法就簡單多了，就是採用 ES6 的函數默認值。

```javascript
function factorial(n, total = 1) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5) // 120
```

上面代碼中，參數`total`有默認值`1`，所以調用時不用提供這個值。

總結一下，遞歸本質上是一種循環操作。純粹的函數式編程語言沒有循環操作命令，所有的循環都用遞歸實現，這就是為什麼尾遞歸對這些語言極其重要。對於其他支持“尾調用優化”的語言（比如 Lua，ES6），只需要知道循環可以用遞歸代替，而一旦使用遞歸，就最好使用尾遞歸。

### 嚴格模式

ES6 的尾調用優化只在嚴格模式下開啟，正常模式是無效的。

這是因為在正常模式下，函數內部有兩個變數，可以跟蹤函數的調用棧。

- `func.arguments`：返回調用時函數的參數。
- `func.caller`：返回調用當前函數的那個函數。

尾調用優化發生時，函數的調用棧會改寫，因此上面兩個變數就會失真。嚴格模式禁用這兩個變數，所以尾調用模式僅在嚴格模式下生效。

```javascript
function restricted() {
  'use strict';
  restricted.caller;    // 報錯
  restricted.arguments; // 報錯
}
restricted();
```

### 尾遞歸優化的實現

尾遞歸優化只在嚴格模式下生效，那麼正常模式下，或者那些不支持該功能的環境中，有沒有辦法也使用尾遞歸優化呢？回答是可以的，就是自己實現尾遞歸優化。

它的原理非常簡單。尾遞歸之所以需要優化，原因是調用棧太多，造成溢出，那麼只要減少調用棧，就不會溢出。怎麼做可以減少調用棧呢？就是採用“循環”換掉“遞歸”。

下面是一個正常的遞歸函數。

```javascript
function sum(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1);
  } else {
    return x;
  }
}

sum(1, 100000)
// Uncaught RangeError: Maximum call stack size exceeded(…)
```

上面代碼中，`sum`是一個遞歸函數，參數`x`是需要累加的值，參數`y`控制遞歸次數。一旦指定`sum`遞歸 100000 次，就會報錯，提示超出調用棧的最大次數。

蹦床函數（trampoline）可以將遞歸執行轉為循環執行。

```javascript
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}
```

上面就是蹦床函數的一個實現，它接受一個函數`f`作為參數。只要`f`執行後返回一個函數，就繼續執行。注意，這裡是返回一個函數，然後執行該函數，而不是函數裡面調用函數，這樣就避免了遞歸執行，從而就消除了調用棧過大的問題。

然後，要做的就是將原來的遞歸函數，改寫為每一步返回另一個函數。

```javascript
function sum(x, y) {
  if (y > 0) {
    return sum.bind(null, x + 1, y - 1);
  } else {
    return x;
  }
}
```

上面代碼中，`sum`函數的每次執行，都會返回自身的另一個版本。

現在，使用蹦床函數執行`sum`，就不會發生調用棧溢出。

```javascript
trampoline(sum(1, 100000))
// 100001
```

蹦床函數並不是真正的尾遞歸優化，下面的實現才是。

```javascript
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}

var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
```

上面代碼中，`tco`函數是尾遞歸優化的實現，它的奧妙就在於狀態變數`active`。默認情況下，這個變數是不激活的。一旦進入尾遞歸優化的過程，這個變數就激活了。然後，每一輪遞歸`sum`返回的都是`undefined`，所以就避免了遞歸執行；而`accumulated`陣列存放每一輪`sum`執行的參數，總是有值的，這就保證了`accumulator`函數內部的`while`循環總是會執行。這樣就很巧妙地將“遞歸”改成了“循環”，而後一輪的參數會取代前一輪的參數，保證了調用棧只有一層。

## 函數參數的尾逗號

ES2017 [允許](https://github.com/jeffmo/es-trailing-function-commas)函數的最後一個參數有尾逗號（trailing comma）。

此前，函數定義和調用時，都不允許最後一個參數後面出現逗號。

```javascript
function clownsEverywhere(
  param1,
  param2
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar'
);
```

上面代碼中，如果在`param2`或`bar`後面加一個逗號，就會報錯。

如果像上面這樣，將參數寫成多行（即每個參數佔據一行），以後修改代碼的時候，想為函數`clownsEverywhere`添加第三個參數，或者調整參數的次序，就勢必要在原來最後一個參數後面添加一個逗號。這對於版本管理系統來說，就會顯示添加逗號的那一行也發生了變動。這看上去有點冗餘，因此新的語法允許定義和調用時，尾部直接有一個逗號。

```javascript
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```

這樣的規定也使得，函數參數與陣列和物件的尾逗號規則，保持一致了。

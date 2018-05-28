# Iterator 和 for...of 循環

## Iterator（遍歷器）的概念

JavaScript 原有的表示“集合”的數據結構，主要是陣列（`Array`）和物件（`Object`），ES6 又添加了`Map`和`Set`。這樣就有了四種數據集合，用戶還可以組合使用它們，定義自己的數據結構，比如陣列的成員是`Map`，`Map`的成員是物件。這樣就需要一種統一的接口機制，來處理所有不同的數據結構。

遍歷器（Iterator）就是這樣一種機制。它是一種接口，為各種不同的數據結構提供統一的訪問機制。任何數據結構只要部署 Iterator 接口，就可以完成遍歷操作（即依次處理該數據結構的所有成員）。

Iterator 的作用有三個：一是為各種數據結構，提供一個統一的、簡便的訪問接口；二是使得數據結構的成員能夠按某種次序排列；三是 ES6 創造了一種新的遍歷命令`for...of`循環，Iterator 接口主要供`for...of`消費。

Iterator 的遍歷過程是這樣的。

（1）創建一個指針物件，指向當前數據結構的起始位置。也就是說，遍歷器物件本質上，就是一個指針物件。

（2）第一次調用指針物件的`next`方法，可以將指針指向數據結構的第一個成員。

（3）第二次調用指針物件的`next`方法，指針就指向數據結構的第二個成員。

（4）不斷調用指針物件的`next`方法，直到它指向數據結構的結束位置。

每一次調用`next`方法，都會返回數據結構的當前成員的信息。具體來說，就是返回一個包含`value`和`done`兩個屬性的物件。其中，`value`屬性是當前成員的值，`done`屬性是一個布爾值，表示遍歷是否結束。

下面是一個模擬`next`方法返回值的例子。

```javascript
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}
```

上面代碼定義了一個`makeIterator`函數，它是一個遍歷器生成函數，作用就是返回一個遍歷器物件。對陣列`['a', 'b']`執行這個函數，就會返回該陣列的遍歷器物件（即指針物件）`it`。

指針物件的`next`方法，用來移動指針。開始時，指針指向陣列的開始位置。然後，每次調用`next`方法，指針就會指向陣列的下一個成員。第一次調用，指向`a`；第二次調用，指向`b`。

`next`方法返回一個物件，表示當前數據成員的信息。這個物件具有`value`和`done`兩個屬性，`value`屬性返回當前位置的成員，`done`屬性是一個布爾值，表示遍歷是否結束，即是否還有必要再一次調用`next`方法。

總之，調用指針物件的`next`方法，就可以遍歷事先給定的數據結構。

對於遍歷器物件來說，`done: false`和`value: undefined`屬性都是可以省略的，因此上面的`makeIterator`函數可以簡寫成下面的形式。

```javascript
function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++]} :
        {done: true};
    }
  };
}
```

由於 Iterator 只是把接口規格加到數據結構之上，所以，遍歷器與它所遍歷的那個數據結構，實際上是分開的，完全可以寫出沒有對應數據結構的遍歷器物件，或者說用遍歷器物件模擬出數據結構。下面是一個無限運行的遍歷器物件的例子。

```javascript
var it = idMaker();

it.next().value // 0
it.next().value // 1
it.next().value // 2
// ...

function idMaker() {
  var index = 0;

  return {
    next: function() {
      return {value: index++, done: false};
    }
  };
}
```

上面的例子中，遍歷器生成函數`idMaker`，返回一個遍歷器物件（即指針物件）。但是並沒有對應的數據結構，或者說，遍歷器物件自己描述了一個數據結構出來。

如果使用 TypeScript 的寫法，遍歷器接口（Iterable）、指針物件（Iterator）和`next`方法返回值的規格可以描述如下。

```javascript
interface Iterable {
  [Symbol.iterator]() : Iterator,
}

interface Iterator {
  next(value?: any) : IterationResult,
}

interface IterationResult {
  value: any,
  done: boolean,
}
```

## 默認 Iterator 接口

Iterator 接口的目的，就是為所有數據結構，提供了一種統一的訪問機制，即`for...of`循環（詳見下文）。當使用`for...of`循環遍歷某種數據結構時，該循環會自動去尋找 Iterator 接口。

一種數據結構只要部署了 Iterator 接口，我們就稱這種數據結構是“可遍歷的”（iterable）。

ES6 規定，默認的 Iterator 接口部署在數據結構的`Symbol.iterator`屬性，或者說，一個數據結構只要具有`Symbol.iterator`屬性，就可以認為是“可遍歷的”（iterable）。`Symbol.iterator`屬性本身是一個函數，就是當前數據結構默認的遍歷器生成函數。執行這個函數，就會返回一個遍歷器。至於屬性名`Symbol.iterator`，它是一個表達式，返回`Symbol`物件的`iterator`屬性，這是一個預定義好的、類型為 Symbol 的特殊值，所以要放在方括號內（參見《Symbol》一章）。

```javascript
const obj = {
  [Symbol.iterator] : function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
```

上面代碼中，物件`obj`是可遍歷的（iterable），因為具有`Symbol.iterator`屬性。執行這個屬性，會返回一個遍歷器物件。該物件的根本特徵就是具有`next`方法。每次調用`next`方法，都會返回一個代表當前成員的信息物件，具有`value`和`done`兩個屬性。

ES6 的有些數據結構原生具備 Iterator 接口（比如陣列），即不用任何處理，就可以被`for...of`循環遍歷。原因在於，這些數據結構原生部署了`Symbol.iterator`屬性（詳見下文），另外一些數據結構沒有（比如物件）。凡是部署了`Symbol.iterator`屬性的數據結構，就稱為部署了遍歷器接口。調用這個接口，就會返回一個遍歷器物件。

原生具備 Iterator 接口的數據結構如下。

- Array
- Map
- Set
- String
- TypedArray
- 函數的 arguments 物件
- NodeList 物件

下面的例子是陣列的`Symbol.iterator`屬性。

```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

上面代碼中，變數`arr`是一個陣列，原生就具有遍歷器接口，部署在`arr`的`Symbol.iterator`屬性上面。所以，調用這個屬性，就得到遍歷器物件。

對於原生部署 Iterator 接口的數據結構，不用自己寫遍歷器生成函數，`for...of`循環會自動遍歷它們。除此之外，其他數據結構（主要是物件）的 Iterator 接口，都需要自己在`Symbol.iterator`屬性上面部署，這樣才會被`for...of`循環遍歷。

物件（Object）之所以沒有默認部署 Iterator 接口，是因為物件的哪個屬性先遍歷，哪個屬性後遍歷是不確定的，需要開發者手動指定。本質上，遍歷器是一種線性處理，對於任何非線性的數據結構，部署遍歷器接口，就等於部署一種線性轉換。不過，嚴格地說，物件部署遍歷器接口並不是很必要，因為這時物件實際上被當作 Map 結構使用，ES5 沒有 Map 結構，而 ES6 原生提供了。

一個物件如果要具備可被`for...of`循環調用的 Iterator 接口，就必須在`Symbol.iterator`的屬性上部署遍歷器生成方法（原型鏈上的物件具有該方法也可）。

```javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value); // 0, 1, 2
}
```

上面代碼是一個類部署 Iterator 接口的寫法。`Symbol.iterator`屬性對應一個函數，執行後返回當前物件的遍歷器物件。

下面是通過遍歷器實現指針結構的例子。

```javascript
function Obj(value) {
  this.value = value;
  this.next = null;
}

Obj.prototype[Symbol.iterator] = function() {
  var iterator = { next: next };

  var current = this;

  function next() {
    if (current) {
      var value = current.value;
      current = current.next;
      return { done: false, value: value };
    } else {
      return { done: true };
    }
  }
  return iterator;
}

var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for (var i of one){
  console.log(i); // 1, 2, 3
}
```

上面代碼首先在構造函數的原型鏈上部署`Symbol.iterator`方法，調用該方法會返回遍歷器物件`iterator`，調用該物件的`next`方法，在返回一個值的同時，自動將內部指針移到下一個實例。

下面是另一個為物件添加 Iterator 接口的例子。

```javascript
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
```

對於類似陣列的物件（存在數值鍵名和`length`屬性），部署 Iterator 接口，有一個簡便方法，就是`Symbol.iterator`方法直接引用陣列的 Iterator 接口。

```javascript
NodeList.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
// 或者
NodeList.prototype[Symbol.iterator] = [][Symbol.iterator];

[...document.querySelectorAll('div')] // 可以執行了
```

NodeList 物件是類似陣列的物件，本來就具有遍歷接口，可以直接遍歷。上面代碼中，我們將它的遍歷接口改成陣列的`Symbol.iterator`屬性，可以看到沒有任何影響。

下面是另一個類似陣列的物件調用陣列的`Symbol.iterator`方法的例子。

```javascript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // 'a', 'b', 'c'
}
```

注意，普通物件部署陣列的`Symbol.iterator`方法，並無效果。

```javascript
let iterable = {
  a: 'a',
  b: 'b',
  c: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // undefined, undefined, undefined
}
```

如果`Symbol.iterator`方法對應的不是遍歷器生成函數（即會返回一個遍歷器物件），解釋引擎將會報錯。

```javascript
var obj = {};

obj[Symbol.iterator] = () => 1;

[...obj] // TypeError: [] is not a function
```

上面代碼中，變數`obj`的`Symbol.iterator`方法對應的不是遍歷器生成函數，因此報錯。

有了遍歷器接口，數據結構就可以用`for...of`循環遍歷（詳見下文），也可以使用`while`循環遍歷。

```javascript
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  var x = $result.value;
  // ...
  $result = $iterator.next();
}
```

上面代碼中，`ITERABLE`代表某種可遍歷的數據結構，`$iterator`是它的遍歷器物件。遍歷器物件每次移動指針（`next`方法），都檢查一下返回值的`done`屬性，如果遍歷還沒結束，就移動遍歷器物件的指針到下一步（`next`方法），不斷循環。

## 調用 Iterator 接口的場合

有一些場合會默認調用 Iterator 接口（即`Symbol.iterator`方法），除了下文會介紹的`for...of`循環，還有幾個別的場合。

**（1）解構賦值**

對陣列和 Set 結構進行解構賦值時，會默認調用`Symbol.iterator`方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

**（2）擴展運算符**

擴展運算符（...）也會調用默認的 Iterator 接口。

```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

上面代碼的擴展運算符內部就調用 Iterator 接口。

實際上，這提供了一種簡便機制，可以將任何部署了 Iterator 接口的數據結構，轉為陣列。也就是說，只要某個數據結構部署了 Iterator 接口，就可以對它使用擴展運算符，將其轉為陣列。

```javascript
let arr = [...iterable];
```

**（3）yield\***

`yield*`後面跟的是一個可遍歷的結構，它會調用該結構的遍歷器接口。

```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

**（4）其他場合**

由於陣列的遍歷會調用遍歷器接口，所以任何接受陣列作為參數的場合，其實都調用了遍歷器接口。下面是一些例子。

- for...of
- Array.from()
- Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
- Promise.all()
- Promise.race()

## 字符串的 Iterator 接口

字符串是一個類似陣列的物件，也原生具有 Iterator 接口。

```javascript
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

上面代碼中，調用`Symbol.iterator`方法返回一個遍歷器物件，在這個遍歷器上可以調用 next 方法，實現對於字符串的遍歷。

可以覆蓋原生的`Symbol.iterator`方法，達到修改遍歷器行為的目的。

```javascript
var str = new String("hi");

[...str] // ["h", "i"]

str[Symbol.iterator] = function() {
  return {
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};

[...str] // ["bye"]
str // "hi"
```

上面代碼中，字符串 str 的`Symbol.iterator`方法被修改了，所以擴展運算符（`...`）返回的值變成了`bye`，而字符串本身還是`hi`。

## Iterator 接口與 Generator 函數

`Symbol.iterator`方法的最簡單實現，還是使用下一章要介紹的 Generator 函數。

```javascript
let myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  }
}
[...myIterable] // [1, 2, 3]

// 或者採用下面的簡潔寫法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// "hello"
// "world"
```

上面代碼中，`Symbol.iterator`方法幾乎不用部署任何代碼，只要用 yield 命令給出每一步的返回值即可。

## 遍歷器物件的 return()，throw()

遍歷器物件除了具有`next`方法，還可以具有`return`方法和`throw`方法。如果你自己寫遍歷器物件生成函數，那麼`next`方法是必須部署的，`return`方法和`throw`方法是否部署是可選的。

`return`方法的使用場合是，如果`for...of`循環提前退出（通常是因為出錯，或者有`break`語句或`continue`語句），就會調用`return`方法。如果一個物件在完成遍歷前，需要清理或釋放資源，就可以部署`return`方法。

```javascript
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}
```

上面代碼中，函數`readLinesSync`接受一個文件物件作為參數，返回一個遍歷器物件，其中除了`next`方法，還部署了`return`方法。下面的三種情況，都會觸發執行`return`方法。

```javascript
// 情況一
for (let line of readLinesSync(fileName)) {
  console.log(line);
  break;
}

// 情況二
for (let line of readLinesSync(fileName)) {
  console.log(line);
  continue;
}

// 情況三
for (let line of readLinesSync(fileName)) {
  console.log(line);
  throw new Error();
}
```

上面代碼中，情況一輸出文件的第一行以後，就會執行`return`方法，關閉這個文件；情況二輸出所有行以後，執行`return`方法，關閉該文件；情況三會在執行`return`方法關閉文件之後，再拋出錯誤。

注意，`return`方法必須返回一個物件，這是 Generator 規格決定的。

`throw`方法主要是配合 Generator 函數使用，一般的遍歷器物件用不到這個方法。請參閱《Generator 函數》一章。

## for...of 循環

ES6 借鑑 C++、Java、C# 和 Python 語言，引入了`for...of`循環，作為遍歷所有數據結構的統一的方法。

一個數據結構只要部署了`Symbol.iterator`屬性，就被視為具有 iterator 接口，就可以用`for...of`循環遍歷它的成員。也就是說，`for...of`循環內部調用的是數據結構的`Symbol.iterator`方法。

`for...of`循環可以使用的範圍包括陣列、Set 和 Map 結構、某些類似陣列的物件（比如`arguments`物件、DOM NodeList 物件）、後文的 Generator 物件，以及字符串。

### 陣列

陣列原生具備`iterator`接口（即默認部署了`Symbol.iterator`屬性），`for...of`循環本質上就是調用這個接口產生的遍歷器，可以用下面的代碼證明。

```javascript
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}

const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for(let v of obj) {
  console.log(v); // red green blue
}
```

上面代碼中，空物件`obj`部署了陣列`arr`的`Symbol.iterator`屬性，結果`obj`的`for...of`循環，產生了與`arr`完全一樣的結果。

`for...of`循環可以代替陣列實例的`forEach`方法。

```javascript
const arr = ['red', 'green', 'blue'];

arr.forEach(function (element, index) {
  console.log(element); // red green blue
  console.log(index);   // 0 1 2
});
```

JavaScript 原有的`for...in`循環，只能獲得物件的鍵名，不能直接獲取鍵值。ES6 提供`for...of`循環，允許遍歷獲得鍵值。

```javascript
var arr = ['a', 'b', 'c', 'd'];

for (let a in arr) {
  console.log(a); // 0 1 2 3
}

for (let a of arr) {
  console.log(a); // a b c d
}
```

上面代碼表明，`for...in`循環讀取鍵名，`for...of`循環讀取鍵值。如果要通過`for...of`循環，獲取陣列的索引，可以借助陣列實例的`entries`方法和`keys`方法（參見《陣列的擴展》一章）。

`for...of`循環調用遍歷器接口，陣列的遍歷器接口只返回具有數字索引的屬性。這一點跟`for...in`循環也不一樣。

```javascript
let arr = [3, 5, 7];
arr.foo = 'hello';

for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}

for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```

上面代碼中，`for...of`循環不會返回陣列`arr`的`foo`屬性。

### Set 和 Map 結構

Set 和 Map 結構也原生具有 Iterator 接口，可以直接使用`for...of`循環。

```javascript
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

上面代碼演示了如何遍歷 Set 結構和 Map 結構。值得注意的地方有兩個，首先，遍歷的順序是按照各個成員被添加進數據結構的順序。其次，Set 結構遍歷時，返回的是一個值，而 Map 結構遍歷時，返回的是一個陣列，該陣列的兩個成員分別為當前 Map 成員的鍵名和鍵值。

```javascript
let map = new Map().set('a', 1).set('b', 2);
for (let pair of map) {
  console.log(pair);
}
// ['a', 1]
// ['b', 2]

for (let [key, value] of map) {
  console.log(key + ' : ' + value);
}
// a : 1
// b : 2
```

### 計算生成的數據結構

有些數據結構是在現有數據結構的基礎上，計算生成的。比如，ES6 的陣列、Set、Map 都部署了以下三個方法，調用後都返回遍歷器物件。

- `entries()` 返回一個遍歷器物件，用來遍歷`[鍵名, 鍵值]`組成的陣列。對於陣列，鍵名就是索引值；對於 Set，鍵名與鍵值相同。Map 結構的 Iterator 接口，默認就是調用`entries`方法。
- `keys()` 返回一個遍歷器物件，用來遍歷所有的鍵名。
- `values()` 返回一個遍歷器物件，用來遍歷所有的鍵值。

這三個方法調用後生成的遍歷器物件，所遍歷的都是計算生成的數據結構。

```javascript
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
```

### 類似陣列的物件

類似陣列的物件包括好幾類。下面是`for...of`循環用於字符串、DOM NodeList 物件、`arguments`物件的例子。

```javascript
// 字符串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList物件
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments物件
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

對於字符串來說，`for...of`循環還有一個特點，就是會正確識別 32 位 UTF-16 字符。

```javascript
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
```

並不是所有類似陣列的物件都具有 Iterator 接口，一個簡便的解決方法，就是使用`Array.from`方法將其轉為陣列。

```javascript
let arrayLike = { length: 2, 0: 'a', 1: 'b' };

// 報錯
for (let x of arrayLike) {
  console.log(x);
}

// 正確
for (let x of Array.from(arrayLike)) {
  console.log(x);
}
```

### 物件

對於普通的物件，`for...of`結構不能直接使用，會報錯，必須部署了 Iterator 接口後才能使用。但是，這樣情況下，`for...in`循環依然可以用來遍歷鍵名。

```javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (let e of es6) {
  console.log(e);
}
// TypeError: es6[Symbol.iterator] is not a function
```

上面代碼表示，對於普通的物件，`for...in`循環可以遍歷鍵名，`for...of`循環會報錯。

一種解決方法是，使用`Object.keys`方法將物件的鍵名生成一個陣列，然後遍歷這個陣列。

```javascript
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
```

另一個方法是使用 Generator 函數將物件重新包裝一下。

```javascript
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```

### 與其他遍歷語法的比較

以陣列為例，JavaScript 提供多種遍歷語法。最原始的寫法就是`for`循環。

```javascript
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```

這種寫法比較麻煩，因此陣列提供內置的`forEach`方法。

```javascript
myArray.forEach(function (value) {
  console.log(value);
});
```

這種寫法的問題在於，無法中途跳出`forEach`循環，`break`命令或`return`命令都不能奏效。

`for...in`循環可以遍曆陣列的鍵名。

```javascript
for (var index in myArray) {
  console.log(myArray[index]);
}
```

`for...in`循環有幾個缺點。

- 陣列的鍵名是數字，但是`for...in`循環是以字符串作為鍵名“0”、“1”、“2”等等。
- `for...in`循環不僅遍曆數字鍵名，還會遍歷手動添加的其他鍵，甚至包括原型鏈上的鍵。
- 某些情況下，`for...in`循環會以任意順序遍歷鍵名。

總之，`for...in`循環主要是為遍歷物件而設計的，不適用於遍曆陣列。

`for...of`循環相比上面幾種做法，有一些顯著的優點。

```javascript
for (let value of myArray) {
  console.log(value);
}
```

- 有著同`for...in`一樣的簡潔語法，但是沒有`for...in`那些缺點。
- 不同於`forEach`方法，它可以與`break`、`continue`和`return`配合使用。
- 提供了遍歷所有數據結構的統一操作接口。

下面是一個使用 break 語句，跳出`for...of`循環的例子。

```javascript
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```

上面的例子，會輸出斐波納契數列小於等於 1000 的項。如果當前項大於 1000，就會使用`break`語句跳出`for...of`循環。
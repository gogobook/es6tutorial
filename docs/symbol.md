# Symbol

## 概述

ES5 的物件屬性名都是字符串，這容易造成屬性名的衝突。比如，你使用了一個他人提供的物件，但又想為這個物件添加新的方法（mixin 模式），新方法的名字就有可能與現有方法產生衝突。如果有一種機制，保證每個屬性的名字都是獨一無二的就好了，這樣就從根本上防止屬性名的衝突。這就是 ES6 引入`Symbol`的原因。

ES6 引入了一種新的原始數據類型`Symbol`，表示獨一無二的值。它是 JavaScript 語言的第七種數據類型，前六種是：`undefined`、`null`、布爾值（Boolean）、字符串（String）、數值（Number）、物件（Object）。

Symbol 值通過`Symbol`函數生成。這就是說，物件的屬性名現在可以有兩種類型，一種是原來就有的字符串，另一種就是新增的 Symbol 類型。凡是屬性名屬於 Symbol 類型，就都是獨一無二的，可以保證不會與其他屬性名產生衝突。

```javascript
let s = Symbol();

typeof s
// "symbol"
```

上面代碼中，變數`s`就是一個獨一無二的值。`typeof`運算符的結果，表明變數`s`是 Symbol 數據類型，而不是字符串之類的其他類型。

注意，`Symbol`函數前不能使用`new`命令，否則會報錯。這是因為生成的 Symbol 是一個原始類型的值，不是物件。也就是說，由於 Symbol 值不是物件，所以不能添加屬性。基本上，它是一種類似於字符串的數據類型。

`Symbol`函數可以接受一個字符串作為參數，表示對 Symbol 實例的描述，主要是為了在控制台顯示，或者轉為字符串時，比較容易區分。

```javascript
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

上面代碼中，`s1`和`s2`是兩個 Symbol 值。如果不加參數，它們在控制台的輸出都是`Symbol()`，不利於區分。有了參數以後，就等於為它們加上了描述，輸出的時候就能夠分清，到底是哪一個值。

如果 Symbol 的參數是一個物件，就會調用該物件的`toString`方法，將其轉為字符串，然後才生成一個 Symbol 值。

```javascript
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
sym // Symbol(abc)
```

注意，`Symbol`函數的參數隻是表示對當前 Symbol 值的描述，因此相同參數的`Symbol`函數的返回值是不相等的。

```javascript
// 沒有參數的情況
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有參數的情況
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

上面代碼中，`s1`和`s2`都是`Symbol`函數的返回值，而且參數相同，但是它們是不相等的。

Symbol 值不能與其他類型的值進行運算，會報錯。

```javascript
let sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string
```

但是，Symbol 值可以顯式轉為字符串。

```javascript
let sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```

另外，Symbol 值也可以轉為布爾值，但是不能轉為數值。

```javascript
let sym = Symbol();
Boolean(sym) // true
!sym  // false

if (sym) {
  // ...
}

Number(sym) // TypeError
sym + 2 // TypeError
```

## 作為屬性名的 Symbol

由於每一個 Symbol 值都是不相等的，這意味著 Symbol 值可以作為標識符，用於物件的屬性名，就能保證不會出現同名的屬性。這對於一個物件由多個模塊構成的情況非常有用，能防止某一個鍵被不小心改寫或覆蓋。

```javascript
let mySymbol = Symbol();

// 第一種寫法
let a = {};
a[mySymbol] = 'Hello!';

// 第二種寫法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三種寫法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上寫法都得到同樣結果
a[mySymbol] // "Hello!"
```

上面代碼通過方括號結構和`Object.defineProperty`，將物件的屬性名指定為一個 Symbol 值。

注意，Symbol 值作為物件屬性名時，不能用點運算符。

```javascript
const mySymbol = Symbol();
const a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```

上面代碼中，因為點運算符後面總是字符串，所以不會讀取`mySymbol`作為標識名所指代的那個值，導致`a`的屬性名實際上是一個字符串，而不是一個 Symbol 值。

同理，在物件的內部，使用 Symbol 值定義屬性時，Symbol 值必須放在方括號之中。

```javascript
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};

obj[s](123);
```

上面代碼中，如果`s`不放在方括號中，該屬性的鍵名就是字符串`s`，而不是`s`所代表的那個 Symbol 值。

採用增強的物件寫法，上面代碼的`obj`物件可以寫得更簡潔一些。

```javascript
let obj = {
  [s](arg) { ... }
};
```

Symbol 類型還可以用於定義一組常量，保證這組常量的值都是不相等的。

```javascript
log.levels = {
  DEBUG: Symbol('debug'),
  INFO: Symbol('info'),
  WARN: Symbol('warn')
};
log(log.levels.DEBUG, 'debug message');
log(log.levels.INFO, 'info message');
```

下面是另外一個例子。

```javascript
const COLOR_RED    = Symbol();
const COLOR_GREEN  = Symbol();

function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default:
      throw new Error('Undefined color');
    }
}
```

常量使用 Symbol 值最大的好處，就是其他任何值都不可能有相同的值了，因此可以保證上面的`switch`語句會按設計的方式工作。

還有一點需要注意，Symbol 值作為屬性名時，該屬性還是公開屬性，不是私有屬性。

## 實例：消除魔術字符串

魔術字符串指的是，在代碼之中多次出現、與代碼形成強耦合的某一個具體的字符串或者數值。風格良好的代碼，應該儘量消除魔術字符串，改由含義清晰的變數代替。

```javascript
function getArea(shape, options) {
  let area = 0;

  switch (shape) {
    case 'Triangle': // 魔術字符串
      area = .5 * options.width * options.height;
      break;
    /* ... more code ... */
  }

  return area;
}

getArea('Triangle', { width: 100, height: 100 }); // 魔術字符串
```

上面代碼中，字符串`Triangle`就是一個魔術字符串。它多次出現，與代碼形成“強耦合”，不利於將來的修改和維護。

常用的消除魔術字符串的方法，就是把它寫成一個變數。

```javascript
const shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```

上面代碼中，我們把`Triangle`寫成`shapeType`物件的`triangle`屬性，這樣就消除了強耦合。

如果仔細分析，可以發現`shapeType.triangle`等於哪個值並不重要，只要確保不會跟其他`shapeType`屬性的值衝突即可。因此，這裡就很適合改用 Symbol 值。

```javascript
const shapeType = {
  triangle: Symbol()
};
```

上面代碼中，除了將`shapeType.triangle`的值設為一個 Symbol，其他地方都不用修改。

## 屬性名的遍歷

Symbol 作為屬性名，該屬性不會出現在`for...in`、`for...of`循環中，也不會被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。但是，它也不是私有屬性，有一個`Object.getOwnPropertySymbols`方法，可以獲取指定物件的所有 Symbol 屬性名。

`Object.getOwnPropertySymbols`方法返回一個陣列，成員是當前物件的所有用作屬性名的 Symbol 值。

```javascript
const obj = {};
let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```

下面是另一個例子，`Object.getOwnPropertySymbols`方法與`for...in`循環、`Object.getOwnPropertyNames`方法進行對比的例子。

```javascript
const obj = {};

let foo = Symbol("foo");

Object.defineProperty(obj, foo, {
  value: "foobar",
});

for (let i in obj) {
  console.log(i); // 無輸出
}

Object.getOwnPropertyNames(obj)
// []

Object.getOwnPropertySymbols(obj)
// [Symbol(foo)]
```

上面代碼中，使用`Object.getOwnPropertyNames`方法得不到`Symbol`屬性名，需要使用`Object.getOwnPropertySymbols`方法。

另一個新的 API，`Reflect.ownKeys`方法可以返回所有類型的鍵名，包括常規鍵名和 Symbol 鍵名。

```javascript
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```

由於以 Symbol 值作為名稱的屬性，不會被常規方法遍歷得到。我們可以利用這個特性，為物件定義一些非私有的、但又希望只用於內部的方法。

```javascript
let size = Symbol('size');

class Collection {
  constructor() {
    this[size] = 0;
  }

  add(item) {
    this[this[size]] = item;
    this[size]++;
  }

  static sizeOf(instance) {
    return instance[size];
  }
}

let x = new Collection();
Collection.sizeOf(x) // 0

x.add('foo');
Collection.sizeOf(x) // 1

Object.keys(x) // ['0']
Object.getOwnPropertyNames(x) // ['0']
Object.getOwnPropertySymbols(x) // [Symbol(size)]
```

上面代碼中，物件`x`的`size`屬性是一個 Symbol 值，所以`Object.keys(x)`、`Object.getOwnPropertyNames(x)`都無法獲取它。這就造成了一種非私有的內部方法的效果。

## Symbol.for()，Symbol.keyFor()

有時，我們希望重新使用同一個 Symbol 值，`Symbol.for`方法可以做到這一點。它接受一個字符串作為參數，然後搜索有沒有以該參數作為名稱的 Symbol 值。如果有，就返回這個 Symbol 值，否則就新建並返回一個以該字符串為名稱的 Symbol 值。

```javascript
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true
```

上面代碼中，`s1`和`s2`都是 Symbol 值，但是它們都是同樣參數的`Symbol.for`方法生成的，所以實際上是同一個值。

`Symbol.for()`與`Symbol()`這兩種寫法，都會生成新的 Symbol。它們的區別是，前者會被登記在全局環境中供搜索，後者不會。`Symbol.for()`不會每次調用就返回一個新的 Symbol 類型的值，而是會先檢查給定的`key`是否已經存在，如果不存在才會新建一個值。比如，如果你調用`Symbol.for("cat")`30 次，每次都會返回同一個 Symbol 值，但是調用`Symbol("cat")`30 次，會返回 30 個不同的 Symbol 值。

```javascript
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```

上面代碼中，由於`Symbol()`寫法沒有登記機制，所以每次調用都會返回一個不同的值。

`Symbol.keyFor`方法返回一個已登記的 Symbol 類型值的`key`。

```javascript
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```

上面代碼中，變數`s2`屬於未登記的 Symbol 值，所以返回`undefined`。

需要注意的是，`Symbol.for`為 Symbol 值登記的名字，是全局環境的，可以在不同的 iframe 或 service worker 中取到同一個值。

```javascript
iframe = document.createElement('iframe');
iframe.src = String(window.location);
document.body.appendChild(iframe);

iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo')
// true
```

上面代碼中，iframe 窗口生成的 Symbol 值，可以在主頁面得到。

## 實例：模塊的 Singleton 模式

Singleton 模式指的是調用一個類，任何時候返回的都是同一個實例。

對於 Node 來說，模塊文件可以看成是一個類。怎麼保證每次執行這個模塊文件，返回的都是同一個實例呢？

很容易想到，可以把實例放到頂層物件`global`。

```javascript
// mod.js
function A() {
  this.foo = 'hello';
}

if (!global._foo) {
  global._foo = new A();
}

module.exports = global._foo;
```

然後，加載上面的`mod.js`。

```javascript
const a = require('./mod.js');
console.log(a.foo);
```

上面代碼中，變數`a`任何時候加載的都是`A`的同一個實例。

但是，這裡有一個問題，全局變數`global._foo`是可寫的，任何文件都可以修改。

```javascript
global._foo = { foo: 'world' };

const a = require('./mod.js');
console.log(a.foo);
```

上面的代碼，會使得加載`mod.js`的腳本都失真。

為了防止這種情況出現，我們就可以使用 Symbol。

```javascript
// mod.js
const FOO_KEY = Symbol.for('foo');

function A() {
  this.foo = 'hello';
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```

上面代碼中，可以保證`global[FOO_KEY]`不會被無意間覆蓋，但還是可以被改寫。

```javascript
global[Symbol.for('foo')] = { foo: 'world' };

const a = require('./mod.js');
```

如果鍵名使用`Symbol`方法生成，那麼外部將無法引用這個值，當然也就無法改寫。

```javascript
// mod.js
const FOO_KEY = Symbol('foo');

// 後面代碼相同 ……
```

上面代碼將導致其他腳本都無法引用`FOO_KEY`。但這樣也有一個問題，就是如果多次執行這個腳本，每次得到的`FOO_KEY`都是不一樣的。雖然 Node 會將腳本的執行結果緩存，一般情況下，不會多次執行同一個腳本，但是用戶可以手動清除緩存，所以也不是絕對可靠。

## 內置的 Symbol 值

除了定義自己使用的 Symbol 值以外，ES6 還提供了 11 個內置的 Symbol 值，指向語言內部使用的方法。

### Symbol.hasInstance

物件的`Symbol.hasInstance`屬性，指向一個內部方法。當其他物件使用`instanceof`運算符，判斷是否為該物件的實例時，會調用這個方法。比如，`foo instanceof Foo`在語言內部，實際調用的是`Foo[Symbol.hasInstance](foo)`。

```javascript
class MyClass {
  [Symbol.hasInstance](foo) {
    return foo instanceof Array;
  }
}

[1, 2, 3] instanceof new MyClass() // true
```

上面代碼中，`MyClass`是一個類，`new MyClass()`會返回一個實例。該實例的`Symbol.hasInstance`方法，會在進行`instanceof`運算時自動調用，判斷左側的運算子是否為`Array`的實例。

下面是另一個例子。

```javascript
class Even {
  static [Symbol.hasInstance](obj) {
    return Number(obj) % 2 === 0;
  }
}

// 等同於
const Even = {
  [Symbol.hasInstance](obj) {
    return Number(obj) % 2 === 0;
  }
};

1 instanceof Even // false
2 instanceof Even // true
12345 instanceof Even // false
```

### Symbol.isConcatSpreadable

物件的`Symbol.isConcatSpreadable`屬性等於一個布爾值，表示該物件用於`Array.prototype.concat()`時，是否可以展開。

```javascript
let arr1 = ['c', 'd'];
['a', 'b'].concat(arr1, 'e') // ['a', 'b', 'c', 'd', 'e']
arr1[Symbol.isConcatSpreadable] // undefined

let arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
['a', 'b'].concat(arr2, 'e') // ['a', 'b', ['c','d'], 'e']
```

上面代碼說明，陣列的默認行為是可以展開，`Symbol.isConcatSpreadable`默認等於`undefined`。該屬性等於`true`時，也有展開的效果。

類似陣列的物件正好相反，默認不展開。它的`Symbol.isConcatSpreadable`屬性設為`true`，才可以展開。

```javascript
let obj = {length: 2, 0: 'c', 1: 'd'};
['a', 'b'].concat(obj, 'e') // ['a', 'b', obj, 'e']

obj[Symbol.isConcatSpreadable] = true;
['a', 'b'].concat(obj, 'e') // ['a', 'b', 'c', 'd', 'e']
```

`Symbol.isConcatSpreadable`屬性也可以定義在類裡面。

```javascript
class A1 extends Array {
  constructor(args) {
    super(args);
    this[Symbol.isConcatSpreadable] = true;
  }
}
class A2 extends Array {
  constructor(args) {
    super(args);
  }
  get [Symbol.isConcatSpreadable] () {
    return false;
  }
}
let a1 = new A1();
a1[0] = 3;
a1[1] = 4;
let a2 = new A2();
a2[0] = 5;
a2[1] = 6;
[1, 2].concat(a1).concat(a2)
// [1, 2, 3, 4, [5, 6]]
```

上面代碼中，類`A1`是可展開的，類`A2`是不可展開的，所以使用`concat`時有不一樣的結果。

注意，`Symbol.isConcatSpreadable`的位置差異，`A1`是定義在實例上，`A2`是定義在類本身，效果相同。

### Symbol.species

物件的`Symbol.species`屬性，指向一個構造函數。創建衍生物件時，會使用該屬性。

```javascript
class MyArray extends Array {
}

const a = new MyArray(1, 2, 3);
const b = a.map(x => x);
const c = a.filter(x => x > 1);

b instanceof MyArray // true
c instanceof MyArray // true
```

上面代碼中，子類`MyArray`繼承了父類`Array`，`a`是`MyArray`的實例，`b`和`c`是`a`的衍生物件。你可能會認為，`b`和`c`都是調用陣列方法生成的，所以應該是陣列（`Array`的實例），但實際上它們也是`MyArray`的實例。

`Symbol.species`屬性就是為瞭解決這個問題而提供的。現在，我們可以為`MyArray`設置`Symbol.species`屬性。

```javascript
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
}
```

上面代碼中，由於定義了`Symbol.species`屬性，創建衍生物件時就會使用這個屬性返回的函數，作為構造函數。這個例子也說明，定義`Symbol.species`屬性要採用`get`取值器。默認的`Symbol.species`屬性等同於下面的寫法。

```javascript
static get [Symbol.species]() {
  return this;
}
```

現在，再來看前面的例子。

```javascript
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
}

const a = new MyArray();
const b = a.map(x => x);

b instanceof MyArray // false
b instanceof Array // true
```

上面代碼中，`a.map(x => x)`生成的衍生物件，就不是`MyArray`的實例，而直接就是`Array`的實例。

再看一個例子。

```javascript
class T1 extends Promise {
}

class T2 extends Promise {
  static get [Symbol.species]() {
    return Promise;
  }
}

new T1(r => r()).then(v => v) instanceof T1 // true
new T2(r => r()).then(v => v) instanceof T2 // false
```

上面代碼中，`T2`定義了`Symbol.species`屬性，`T1`沒有。結果就導致了創建衍生物件時（`then`方法），`T1`調用的是自身的構造方法，而`T2`調用的是`Promise`的構造方法。

總之，`Symbol.species`的作用在於，實例物件在運行過程中，需要再次調用自身的構造函數時，會調用該屬性指定的構造函數。它主要的用途是，有些類庫是在基類的基礎上修改的，那麼子類使用繼承的方法時，作者可能希望返回基類的實例，而不是子類的實例。

### Symbol.match

物件的`Symbol.match`屬性，指向一個函數。當執行`str.match(myObject)`時，如果該屬性存在，會調用它，返回該方法的返回值。

```javascript
String.prototype.match(regexp)
// 等同於
regexp[Symbol.match](this)

class MyMatcher {
  [Symbol.match](string) {
    return 'hello world'.indexOf(string);
  }
}

'e'.match(new MyMatcher()) // 1
```

### Symbol.replace

物件的`Symbol.replace`屬性，指向一個方法，當該物件被`String.prototype.replace`方法調用時，會返回該方法的返回值。

```javascript
String.prototype.replace(searchValue, replaceValue)
// 等同於
searchValue[Symbol.replace](this, replaceValue)
```

下面是一個例子。

```javascript
const x = {};
x[Symbol.replace] = (...s) => console.log(s);

'Hello'.replace(x, 'World') // ["Hello", "World"]
```

`Symbol.replace`方法會收到兩個參數，第一個參數是`replace`方法正在作用的物件，上面例子是`Hello`，第二個參數是替換後的值，上面例子是`World`。

### Symbol.search

物件的`Symbol.search`屬性，指向一個方法，當該物件被`String.prototype.search`方法調用時，會返回該方法的返回值。

```javascript
String.prototype.search(regexp)
// 等同於
regexp[Symbol.search](this)

class MySearch {
  constructor(value) {
    this.value = value;
  }
  [Symbol.search](string) {
    return string.indexOf(this.value);
  }
}
'foobar'.search(new MySearch('foo')) // 0
```

### Symbol.split

物件的`Symbol.split`屬性，指向一個方法，當該物件被`String.prototype.split`方法調用時，會返回該方法的返回值。

```javascript
String.prototype.split(separator, limit)
// 等同於
separator[Symbol.split](this, limit)
```

下面是一個例子。

```javascript
class MySplitter {
  constructor(value) {
    this.value = value;
  }
  [Symbol.split](string) {
    let index = string.indexOf(this.value);
    if (index === -1) {
      return string;
    }
    return [
      string.substr(0, index),
      string.substr(index + this.value.length)
    ];
  }
}

'foobar'.split(new MySplitter('foo'))
// ['', 'bar']

'foobar'.split(new MySplitter('bar'))
// ['foo', '']

'foobar'.split(new MySplitter('baz'))
// 'foobar'
```

上面方法使用`Symbol.split`方法，重新定義了字符串物件的`split`方法的行為，

### Symbol.iterator

物件的`Symbol.iterator`屬性，指向該物件的默認遍歷器方法。

```javascript
const myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

物件進行`for...of`循環時，會調用`Symbol.iterator`方法，返回該物件的默認遍歷器，詳細介紹參見《Iterator 和 for...of 循環》一章。

```javascript
class Collection {
  *[Symbol.iterator]() {
    let i = 0;
    while(this[i] !== undefined) {
      yield this[i];
      ++i;
    }
  }
}

let myCollection = new Collection();
myCollection[0] = 1;
myCollection[1] = 2;

for(let value of myCollection) {
  console.log(value);
}
// 1
// 2
```

### Symbol.toPrimitive

物件的`Symbol.toPrimitive`屬性，指向一個方法。該物件被轉為原始類型的值時，會調用這個方法，返回該物件對應的原始類型值。

`Symbol.toPrimitive`被調用時，會接受一個字符串參數，表示當前運算的模式，一共有三種模式。

- Number：該場合需要轉成數值
- String：該場合需要轉成字符串
- Default：該場合可以轉成數值，也可以轉成字符串

```javascript
let obj = {
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return 123;
      case 'string':
        return 'str';
      case 'default':
        return 'default';
      default:
        throw new Error();
     }
   }
};

2 * obj // 246
3 + obj // '3default'
obj == 'default' // true
String(obj) // 'str'
```

### Symbol.toStringTag

物件的`Symbol.toStringTag`屬性，指向一個方法。在該物件上面調用`Object.prototype.toString`方法時，如果這個屬性存在，它的返回值會出現在`toString`方法返回的字符串之中，表示物件的類型。也就是說，這個屬性可以用來定製`[object Object]`或`[object Array]`中`object`後面的那個字符串。

```javascript
// 例一
({[Symbol.toStringTag]: 'Foo'}.toString())
// "[object Foo]"

// 例二
class Collection {
  get [Symbol.toStringTag]() {
    return 'xxx';
  }
}
let x = new Collection();
Object.prototype.toString.call(x) // "[object xxx]"
```

ES6 新增內置物件的`Symbol.toStringTag`屬性值如下。

- `JSON[Symbol.toStringTag]`：'JSON'
- `Math[Symbol.toStringTag]`：'Math'
- Module 物件`M[Symbol.toStringTag]`：'Module'
- `ArrayBuffer.prototype[Symbol.toStringTag]`：'ArrayBuffer'
- `DataView.prototype[Symbol.toStringTag]`：'DataView'
- `Map.prototype[Symbol.toStringTag]`：'Map'
- `Promise.prototype[Symbol.toStringTag]`：'Promise'
- `Set.prototype[Symbol.toStringTag]`：'Set'
- `%TypedArray%.prototype[Symbol.toStringTag]`：'Uint8Array'等
- `WeakMap.prototype[Symbol.toStringTag]`：'WeakMap'
- `WeakSet.prototype[Symbol.toStringTag]`：'WeakSet'
- `%MapIteratorPrototype%[Symbol.toStringTag]`：'Map Iterator'
- `%SetIteratorPrototype%[Symbol.toStringTag]`：'Set Iterator'
- `%StringIteratorPrototype%[Symbol.toStringTag]`：'String Iterator'
- `Symbol.prototype[Symbol.toStringTag]`：'Symbol'
- `Generator.prototype[Symbol.toStringTag]`：'Generator'
- `GeneratorFunction.prototype[Symbol.toStringTag]`：'GeneratorFunction'

### Symbol.unscopables

物件的`Symbol.unscopables`屬性，指向一個物件。該物件指定了使用`with`關鍵字時，哪些屬性會被`with`環境排除。

```javascript
Array.prototype[Symbol.unscopables]
// {
//   copyWithin: true,
//   entries: true,
//   fill: true,
//   find: true,
//   findIndex: true,
//   includes: true,
//   keys: true
// }

Object.keys(Array.prototype[Symbol.unscopables])
// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'includes', 'keys']
```

上面代碼說明，陣列有 7 個屬性，會被`with`命令排除。

```javascript
// 沒有 unscopables 時
class MyClass {
  foo() { return 1; }
}

var foo = function () { return 2; };

with (MyClass.prototype) {
  foo(); // 1
}

// 有 unscopables 時
class MyClass {
  foo() { return 1; }
  get [Symbol.unscopables]() {
    return { foo: true };
  }
}

var foo = function () { return 2; };

with (MyClass.prototype) {
  foo(); // 2
}
```

上面代碼通過指定`Symbol.unscopables`屬性，使得`with`語法塊不會在當前作用域尋找`foo`屬性，即`foo`將指向外層作用域的變數。
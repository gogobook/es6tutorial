# Class 的基本語法

## 簡介

JavaScript 語言中，生成實例物件的傳統方法是通過構造函數。下面是一個例子。

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
```

上面這種寫法跟傳統的面向物件語言（比如 C++ 和 Java）差異很大，很容易讓新學習這門語言的程序員感到困惑。

ES6 提供了更接近傳統語言的寫法，引入了 Class（類）這個概念，作為物件的模板。通過`class`關鍵字，可以定義類。

基本上，ES6 的`class`可以看作只是一個語法糖，它的絕大部分功能，ES5 都可以做到，新的`class`寫法只是讓物件原型的寫法更加清晰、更像面向物件編程的語法而已。上面的代碼用 ES6 的`class`改寫，就是下面這樣。

```javascript
//定義類
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

上面代碼定義了一個“類”，可以看到裡面有一個`constructor`方法，這就是構造方法，而`this`關鍵字則代表實例物件。也就是說，ES5 的構造函數`Point`，對應 ES6 的`Point`類的構造方法。

`Point`類除了構造方法，還定義了一個`toString`方法。注意，定義“類”的方法的時候，前面不需要加上`function`這個關鍵字，直接把函數定義放進去了就可以了。另外，方法之間不需要逗號分隔，加了會報錯。

ES6 的類，完全可以看作構造函數的另一種寫法。

```javascript
class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```

上面代碼表明，類的數據類型就是函數，類本身就指向構造函數。

使用的時候，也是直接對類使用`new`命令，跟構造函數的用法完全一致。

```javascript
class Bar {
  doStuff() {
    console.log('stuff');
  }
}

var b = new Bar();
b.doStuff() // "stuff"
```

構造函數的`prototype`屬性，在 ES6 的“類”上面繼續存在。事實上，類的所有方法都定義在類的`prototype`屬性上面。

```javascript
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同於

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```

在類的實例上面調用方法，其實就是調用原型上的方法。

```javascript
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
```

上面代碼中，`b`是`B`類的實例，它的`constructor`方法就是`B`類原型的`constructor`方法。

由於類的方法都定義在`prototype`物件上面，所以類的新方法可以添加在`prototype`物件上面。`Object.assign`方法可以很方便地一次向類添加多個方法。

```javascript
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```

`prototype`物件的`constructor`屬性，直接指向“類”的本身，這與 ES5 的行為是一致的。

```javascript
Point.prototype.constructor === Point // true
```

另外，類的內部所有定義的方法，都是不可枚舉的（non-enumerable）。

```javascript
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代碼中，`toString`方法是`Point`類內部定義的方法，它是不可枚舉的。這一點與 ES5 的行為不一致。

```javascript
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代碼採用 ES5 的寫法，`toString`方法就是可枚舉的。

類的屬性名，可以採用表達式。

```javascript
let methodName = 'getArea';

class Square {
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

上面代碼中，`Square`類的方法名`getArea`，是從表達式得到的。

## 嚴格模式

類和模塊的內部，默認就是嚴格模式，所以不需要使用`use strict`指定運行模式。只要你的代碼寫在類或模塊之中，就只有嚴格模式可用。

考慮到未來所有的代碼，其實都是運行在模塊之中，所以 ES6 實際上把整個語言升級到了嚴格模式。

## constructor 方法

`constructor`方法是類的默認方法，通過`new`命令生成物件實例時，自動調用該方法。一個類必須有`constructor`方法，如果沒有顯式定義，一個空的`constructor`方法會被默認添加。

```javascript
class Point {
}

// 等同於
class Point {
  constructor() {}
}
```

上面代碼中，定義了一個空的類`Point`，JavaScript 引擎會自動為它添加一個空的`constructor`方法。

`constructor`方法默認返回實例物件（即`this`），完全可以指定返回另外一個物件。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

上面代碼中，`constructor`函數返回一個全新的物件，結果導致實例物件不是`Foo`類的實例。

類必須使用`new`調用，否則會報錯。這是它跟普通構造函數的一個主要區別，後者不用`new`也可以執行。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

Foo()
// TypeError: Class constructor Foo cannot be invoked without 'new'
```

## 類的實例物件

生成類的實例物件的寫法，與 ES5 完全一樣，也是使用`new`命令。前面說過，如果忘記加上`new`，像函數那樣調用`Class`，將會報錯。

```javascript
class Point {
  // ...
}

// 報錯
var point = Point(2, 3);

// 正確
var point = new Point(2, 3);
```

與 ES5 一樣，實例的屬性除非顯式定義在其本身（即定義在`this`物件上），否則都是定義在原型上（即定義在`class`上）。

```javascript
//定義類
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

上面代碼中，`x`和`y`都是實例物件`point`自身的屬性（因為定義在`this`變數上），所以`hasOwnProperty`方法返回`true`，而`toString`是原型物件的屬性（因為定義在`Point`類上），所以`hasOwnProperty`方法返回`false`。這些都與 ES5 的行為保持一致。

與 ES5 一樣，類的所有實例共享一個原型物件。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true
```

上面代碼中，`p1`和`p2`都是`Point`的實例，它們的原型都是`Point.prototype`，所以`__proto__`屬性是相等的。

這也意味著，可以通過實例的`__proto__`屬性為“類”添加方法。

> `__proto__` 並不是語言本身的特性，這是各大廠商具體實現時添加的私有屬性，雖然目前很多現代瀏覽器的 JS 引擎中都提供了這個私有屬性，但依舊不建議在生產中使用該屬性，避免對環境產生依賴。生產環境中，我們可以使用 `Object.getPrototypeOf` 方法來獲取實例物件的原型，然後再來為原型添加方法/屬性。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

上面代碼在`p1`的原型上添加了一個`printName`方法，由於`p1`的原型就是`p2`的原型，因此`p2`也可以調用這個方法。而且，此後新建的實例`p3`也可以調用這個方法。這意味著，使用實例的`__proto__`屬性改寫原型，必須相當謹慎，不推薦使用，因為這會改變“類”的原始定義，影響到所有實例。

## Class 表達式

與函數一樣，類也可以使用表達式的形式定義。

```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```

上面代碼使用表達式定義了一個類。需要注意的是，這個類的名字是`MyClass`而不是`Me`，`Me`只在 Class 的內部代碼可用，指代當前類。

```javascript
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

上面代碼表示，`Me`只在 Class 內部有定義。

如果類的內部沒用到的話，可以省略`Me`，也就是可以寫成下面的形式。

```javascript
const MyClass = class { /* ... */ };
```

採用 Class 表達式，可以寫出立即執行的 Class。

```javascript
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('張三');

person.sayName(); // "張三"
```

上面代碼中，`person`是一個立即執行的類的實例。

## 不存在變數提升

類不存在變數提升（hoist），這一點與 ES5 完全不同。

```javascript
new Foo(); // ReferenceError
class Foo {}
```

上面代碼中，`Foo`類使用在前，定義在後，這樣會報錯，因為 ES6 不會把類的聲明提升到代碼頭部。這種規定的原因與下文要提到的繼承有關，必須保證子類在父類之後定義。

```javascript
{
  let Foo = class {};
  class Bar extends Foo {
  }
}
```

上面的代碼不會報錯，因為`Bar`繼承`Foo`的時候，`Foo`已經有定義了。但是，如果存在`class`的提升，上面代碼就會報錯，因為`class`會被提升到代碼頭部，而`let`命令是不提升的，所以導致`Bar`繼承`Foo`的時候，`Foo`還沒有定義。

## 私有方法和私有屬性

### 現有的方法

私有方法是常見需求，但 ES6 不提供，只能通過變通方法模擬實現。

一種做法是在命名上加以區別。

```javascript
class Widget {

  // 公有方法
  foo (baz) {
    this._bar(baz);
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz;
  }

  // ...
}
```

上面代碼中，`_bar`方法前面的下劃線，表示這是一個只限於內部使用的私有方法。但是，這種命名是不保險的，在類的外部，還是可以調用到這個方法。

另一種方法就是索性將私有方法移出模塊，因為模塊內部的所有方法都是對外可見的。

```javascript
class Widget {
  foo (baz) {
    bar.call(this, baz);
  }

  // ...
}

function bar(baz) {
  return this.snaf = baz;
}
```

上面代碼中，`foo`是公有方法，內部調用了`bar.call(this, baz)`。這使得`bar`實際上成為了當前模塊的私有方法。

還有一種方法是利用`Symbol`值的唯一性，將私有方法的名字命名為一個`Symbol`值。

```javascript
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

上面代碼中，`bar`和`snaf`都是`Symbol`值，導致第三方無法獲取到它們，因此達到了私有方法和私有屬性的效果。

### 私有屬性的提案

與私有方法一樣，ES6 不支持私有屬性。目前，有一個[提案](https://github.com/tc39/proposal-private-methods)，為`class`加了私有屬性。方法是在屬性名之前，使用`#`表示。

```javascript
class Point {
  #x;

  constructor(x = 0) {
    #x = +x; // 寫成 this.#x 亦可
  }

  get x() { return #x }
  set x(value) { #x = +value }
}
```

上面代碼中，`#x`就是私有屬性，在`Point`類之外是讀取不到這個屬性的。由於井號`#`是屬性名的一部分，使用時必須帶有`#`一起使用，所以`#x`和`x`是兩個不同的屬性。

私有屬性可以指定初始值，在構造函數執行時進行初始化。

```javascript
class Point {
  #x = 0;
  constructor() {
    #x; // 0
  }
}
```

之所以要引入一個新的前綴`#`表示私有屬性，而沒有採用`private`關鍵字，是因為 JavaScript 是一門動態語言，使用獨立的符號似乎是唯一的可靠方法，能夠準確地區分一種屬性是否為私有屬性。另外，Ruby 語言使用`@`表示私有屬性，ES6 沒有用這個符號而使用`#`，是因為`@`已經被留給了 Decorator。

這種寫法不僅可以寫私有屬性，還可以用來寫私有方法。

```javascript
class Foo {
  #a;
  #b;
  #sum() { return #a + #b; }
  printSum() { console.log(#sum()); }
  constructor(a, b) { #a = a; #b = b; }
}
```

上面代碼中，`#sum()`就是一個私有方法。

另外，私有屬性也可以設置 getter 和 setter 方法。

```javascript
class Counter {
  #xValue = 0;

  get #x() { return #xValue; }
  set #x(value) {
    this.#xValue = value;
  }

  constructor() {
    super();
    // ...
  }
}
```

上面代碼中，`#x`是一個私有屬性，它的讀寫都通過`get #x()`和`set #x()`來完成。

## this 的指向

類的方法內部如果含有`this`，它默認指向類的實例。但是，必須非常小心，一旦單獨使用該方法，很可能報錯。

```javascript
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```

上面代碼中，`printName`方法中的`this`，默認指向`Logger`類的實例。但是，如果將這個方法提取出來單獨使用，`this`會指向該方法運行時所在的環境，因為找不到`print`方法而導致報錯。

一個比較簡單的解決方法是，在構造方法中綁定`this`，這樣就不會找不到`print`方法了。

```javascript
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}
```

另一種解決方法是使用箭頭函數。

```javascript
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }

  // ...
}
```

還有一種解決方法是使用`Proxy`，獲取方法的時候，自動綁定`this`。

```javascript
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    get (target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}

const logger = selfish(new Logger());
```

## name 屬性

由於本質上，ES6 的類只是 ES5 的構造函數的一層包裝，所以函數的許多特性都被`Class`繼承，包括`name`屬性。

```javascript
class Point {}
Point.name // "Point"
```

`name`屬性總是返回緊跟在`class`關鍵字後面的類名。

## Class 的取值函數（getter）和存值函數（setter）

與 ES5 一樣，在“類”的內部可以使用`get`和`set`關鍵字，對某個屬性設置存值函數和取值函數，攔截該屬性的存取行為。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

上面代碼中，`prop`屬性有對應的存值函數和取值函數，因此賦值和讀取行為都被自定義了。

存值函數和取值函數是設置在屬性的 Descriptor 物件上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```

上面代碼中，存值函數和取值函數是定義在`html`屬性的描述物件上面，這與 ES5 完全一致。

## Class 的 Generator 方法

如果某個方法之前加上星號（`*`），就表示該方法是一個 Generator 函數。

```javascript
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```

上面代碼中，`Foo`類的`Symbol.iterator`方法前有一個星號，表示該方法是一個 Generator 函數。`Symbol.iterator`方法返回一個`Foo`類的默認遍歷器，`for...of`循環會自動調用這個遍歷器。

## Class 的靜態方法

類相當於實例的原型，所有在類中定義的方法，都會被實例繼承。如果在一個方法前，加上`static`關鍵字，就表示該方法不會被實例繼承，而是直接通過類來調用，這就稱為“靜態方法”。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

上面代碼中，`Foo`類的`classMethod`方法前有`static`關鍵字，表明該方法是一個靜態方法，可以直接在`Foo`類上調用（`Foo.classMethod()`），而不是在`Foo`類的實例上調用。如果在實例上調用靜態方法，會拋出一個錯誤，表示不存在該方法。

注意，如果靜態方法包含`this`關鍵字，這個`this`指的是類，而不是實例。

```javascript
class Foo {
  static bar () {
    this.baz();
  }
  static baz () {
    console.log('hello');
  }
  baz () {
    console.log('world');
  }
}

Foo.bar() // hello
```

上面代碼中，靜態方法`bar`調用了`this.baz`，這裡的`this`指的是`Foo`類，而不是`Foo`的實例，等同於調用`Foo.baz`。另外，從這個例子還可以看出，靜態方法可以與非靜態方法重名。

父類的靜態方法，可以被子類繼承。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```

上面代碼中，父類`Foo`有一個靜態方法，子類`Bar`可以調用這個方法。

靜態方法也是可以從`super`物件上調用的。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod() // "hello, too"
```

## Class 的靜態屬性和實例屬性

靜態屬性指的是 Class 本身的屬性，即`Class.propName`，而不是定義在實例物件（`this`）上的屬性。

```javascript
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```

上面的寫法為`Foo`類定義了一個靜態屬性`prop`。

目前，只有這種寫法可行，因為 ES6 明確規定，Class 內部只有靜態方法，沒有靜態屬性。

```javascript
// 以下兩種寫法都無效
class Foo {
  // 寫法一
  prop: 2

  // 寫法二
  static prop: 2
}

Foo.prop // undefined
```

目前有一個靜態屬性的[提案](https://github.com/tc39/proposal-class-fields)，對實例屬性和靜態屬性都規定了新的寫法。

（1）類的實例屬性

類的實例屬性可以用等式，寫入類的定義之中。

```javascript
class MyClass {
  myProp = 42;

  constructor() {
    console.log(this.myProp); // 42
  }
}
```

上面代碼中，`myProp`就是`MyClass`的實例屬性。在`MyClass`的實例上，可以讀取這個屬性。

以前，我們定義實例屬性，只能寫在類的`constructor`方法裡面。

```javascript
class ReactCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
```

上面代碼中，構造方法`constructor`裡面，定義了`this.state`屬性。

有了新的寫法以後，可以不在`constructor`方法裡面定義。

```javascript
class ReactCounter extends React.Component {
  state = {
    count: 0
  };
}
```

這種寫法比以前更清晰。

為了可讀性的目的，對於那些在`constructor`裡面已經定義的實例屬性，新寫法允許直接列出。

```javascript
class ReactCounter extends React.Component {
  state;
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
```

（2）類的靜態屬性

類的靜態屬性只要在上面的實例屬性寫法前面，加上`static`關鍵字就可以了。

```javascript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

同樣的，這個新寫法大大方便了靜態屬性的表達。

```javascript
// 老寫法
class Foo {
  // ...
}
Foo.prop = 1;

// 新寫法
class Foo {
  static prop = 1;
}
```

上面代碼中，老寫法的靜態屬性定義在類的外部。整個類生成以後，再生成靜態屬性。這樣讓人很容易忽略這個靜態屬性，也不符合相關代碼應該放在一起的代碼組織原則。另外，新寫法是顯式聲明（declarative），而不是賦值處理，語義更好。

## new.target 屬性

`new`是從構造函數生成實例物件的命令。ES6 為`new`命令引入了一個`new.target`屬性，該屬性一般用在構造函數之中，返回`new`命令作用於的那個構造函數。如果構造函數不是通過`new`命令調用的，`new.target`會返回`undefined`，因此這個屬性可以用來確定構造函數是怎麼調用的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必須使用 new 命令生成實例');
  }
}

// 另一種寫法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必須使用 new 命令生成實例');
  }
}

var person = new Person('張三'); // 正確
var notAPerson = Person.call(person, '張三');  // 報錯
```

上面代碼確保構造函數只能通過`new`命令調用。

Class 內部調用`new.target`，返回當前 Class。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 輸出 true
```

需要注意的是，子類繼承父類時，`new.target`會返回子類。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 輸出 false
```

上面代碼中，`new.target`會返回子類。

利用這個特點，可以寫出不能獨立使用、必須繼承後才能使用的類。

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本類不能實例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape();  // 報錯
var y = new Rectangle(3, 4);  // 正確
```

上面代碼中，`Shape`類不能被實例化，只能用於繼承。

注意，在函數外部，使用`new.target`會報錯。
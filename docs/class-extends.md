# Class 的繼承

## 簡介

Class 可以通過`extends`關鍵字實現繼承，這比 ES5 的通過修改原型鏈實現繼承，要清晰和方便很多。

```javascript
class Point {
}

class ColorPoint extends Point {
}
```

上面代碼定義了一個`ColorPoint`類，該類通過`extends`關鍵字，繼承了`Point`類的所有屬性和方法。但是由於沒有部署任何代碼，所以這兩個類完全一樣，等於複製了一個`Point`類。下面，我們在`ColorPoint`內部加上代碼。

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 調用父類的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 調用父類的toString()
  }
}
```

上面代碼中，`constructor`方法和`toString`方法之中，都出現了`super`關鍵字，它在這裡表示父類的構造函數，用來新建父類的`this`物件。

子類必須在`constructor`方法中調用`super`方法，否則新建實例時會報錯。這是因為子類自己的`this`物件，必須先通過父類的構造函數完成塑造，得到與父類同樣的實例屬性和方法，然後再對其進行加工，加上子類自己的實例屬性和方法。如果不調用`super`方法，子類就得不到`this`物件。

```javascript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

上面代碼中，`ColorPoint`繼承了父類`Point`，但是它的構造函數沒有調用`super`方法，導致新建實例時報錯。

ES5 的繼承，實質是先創造子類的實例物件`this`，然後再將父類的方法添加到`this`上面（`Parent.apply(this)`）。ES6 的繼承機制完全不同，實質是先創造父類的實例物件`this`（所以必須先調用`super`方法），然後再用子類的構造函數修改`this`。

如果子類沒有定義`constructor`方法，這個方法會被默認添加，代碼如下。也就是說，不管有沒有顯式定義，任何一個子類都有`constructor`方法。

```javascript
class ColorPoint extends Point {
}

// 等同於
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```

另一個需要注意的地方是，在子類的構造函數中，只有調用`super`之後，才可以使用`this`關鍵字，否則會報錯。這是因為子類實例的構建，是基於對父類實例加工，只有`super`方法才能返回父類實例。

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正確
  }
}
```

上面代碼中，子類的`constructor`方法沒有調用`super`之前，就使用`this`關鍵字，結果報錯，而放在`super`方法之後就是正確的。

下面是生成子類實例的代碼。

```javascript
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

上面代碼中，實例物件`cp`同時是`ColorPoint`和`Point`兩個類的實例，這與 ES5 的行為完全一致。

最後，父類的靜態方法，也會被子類繼承。

```javascript
class A {
  static hello() {
    console.log('hello world');
  }
}

class B extends A {
}

B.hello()  // hello world
```

上面代碼中，`hello()`是`A`類的靜態方法，`B`繼承`A`，也繼承了`A`的靜態方法。

## Object.getPrototypeOf()

`Object.getPrototypeOf`方法可以用來從子類上獲取父類。

```javascript
Object.getPrototypeOf(ColorPoint) === Point
// true
```

因此，可以使用這個方法判斷，一個類是否繼承了另一個類。

## super 關鍵字

`super`這個關鍵字，既可以當作函數使用，也可以當作物件使用。在這兩種情況下，它的用法完全不同。

第一種情況，`super`作為函數調用時，代表父類的構造函數。ES6 要求，子類的構造函數必須執行一次`super`函數。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

上面代碼中，子類`B`的構造函數之中的`super()`，代表調用父類的構造函數。這是必須的，否則 JavaScript 引擎會報錯。

注意，`super`雖然代表了父類`A`的構造函數，但是返回的是子類`B`的實例，即`super`內部的`this`指的是`B`，因此`super()`在這裡相當於`A.prototype.constructor.call(this)`。

```javascript
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

上面代碼中，`new.target`指向當前正在執行的函數。可以看到，在`super()`執行時，它指向的是子類`B`的構造函數，而不是父類`A`的構造函數。也就是說，`super()`內部的`this`指向的是`B`。

作為函數時，`super()`只能用在子類的構造函數之中，用在其他地方就會報錯。

```javascript
class A {}

class B extends A {
  m() {
    super(); // 報錯
  }
}
```

上面代碼中，`super()`用在`B`類的`m`方法之中，就會造成句法錯誤。

第二種情況，`super`作為物件時，在普通方法中，指向父類的原型物件；在靜態方法中，指向父類。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

上面代碼中，子類`B`當中的`super.p()`，就是將`super`當作一個物件使用。這時，`super`在普通方法之中，指向`A.prototype`，所以`super.p()`就相當於`A.prototype.p()`。

這裡需要注意，由於`super`指向父類的原型物件，所以定義在父類實例上的方法或屬性，是無法通過`super`調用的。

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

上面代碼中，`p`是父類`A`實例的屬性，`super.p`就引用不到它。

如果屬性定義在父類的原型物件上，`super`就可以取到。

```javascript
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

上面代碼中，屬性`x`是定義在`A.prototype`上面的，所以`super.x`可以取到它的值。

ES6 規定，在子類普通方法中通過`super`調用父類的方法時，方法內部的`this`指向當前的子類實例。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
  }
}

let b = new B();
b.m() // 2
```

上面代碼中，`super.print()`雖然調用的是`A.prototype.print()`，但是`A.prototype.print()`內部的`this`指向子類`B`的實例，導致輸出的是`2`，而不是`1`。也就是說，實際上執行的是`super.print.call(this)`。

由於`this`指向子類實例，所以如果通過`super`對某個屬性賦值，這時`super`就是`this`，賦值的屬性會變成子類實例的屬性。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

上面代碼中，`super.x`賦值為`3`，這時等同於對`this.x`賦值為`3`。而當讀取`super.x`的時候，讀的是`A.prototype.x`，所以返回`undefined`。

如果`super`作為物件，用在靜態方法之中，這時`super`將指向父類，而不是父類的原型物件。

```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

上面代碼中，`super`在靜態方法之中指向父類，在普通方法之中指向父類的原型物件。

另外，在子類的靜態方法中通過`super`調用父類的方法時，方法內部的`this`指向當前的子類，而不是子類的實例。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```

上面代碼中，靜態方法`B.m`裡面，`super.print`指向父類的靜態方法。這個方法裡面的`this`指向的是`B`，而不是`B`的實例。

注意，使用`super`的時候，必須顯式指定是作為函數、還是作為物件使用，否則會報錯。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 報錯
  }
}
```

上面代碼中，`console.log(super)`當中的`super`，無法看出是作為函數使用，還是作為物件使用，所以 JavaScript 引擎解析代碼的時候就會報錯。這時，如果能清晰地表明`super`的數據類型，就不會報錯。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super.valueOf() instanceof B); // true
  }
}

let b = new B();
```

上面代碼中，`super.valueOf()`表明`super`是一個物件，因此就不會報錯。同時，由於`super`使得`this`指向`B`的實例，所以`super.valueOf()`返回的是一個`B`的實例。

最後，由於物件總是繼承其他物件的，所以可以在任意一個物件中，使用`super`關鍵字。

```javascript
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); // MyObject: [object Object]
```

## 類的 prototype 屬性和\_\_proto\_\_屬性

大多數瀏覽器的 ES5 實現之中，每一個物件都有`__proto__`屬性，指向對應的構造函數的`prototype`屬性。Class 作為構造函數的語法糖，同時有`prototype`屬性和`__proto__`屬性，因此同時存在兩條繼承鏈。

（1）子類的`__proto__`屬性，表示構造函數的繼承，總是指向父類。

（2）子類`prototype`屬性的`__proto__`屬性，表示方法的繼承，總是指向父類的`prototype`屬性。

```javascript
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

上面代碼中，子類`B`的`__proto__`屬性指向父類`A`，子類`B`的`prototype`屬性的`__proto__`屬性指向父類`A`的`prototype`屬性。

這樣的結果是因為，類的繼承是按照下面的模式實現的。

```javascript
class A {
}

class B {
}

// B 的實例繼承 A 的實例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 繼承 A 的靜態屬性
Object.setPrototypeOf(B, A);

const b = new B();
```

《物件的擴展》一章給出過`Object.setPrototypeOf`方法的實現。

```javascript
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

因此，就得到了上面的結果。

```javascript
Object.setPrototypeOf(B.prototype, A.prototype);
// 等同於
B.prototype.__proto__ = A.prototype;

Object.setPrototypeOf(B, A);
// 等同於
B.__proto__ = A;
```

這兩條繼承鏈，可以這樣理解：作為一個物件，子類（`B`）的原型（`__proto__`屬性）是父類（`A`）；作為一個構造函數，子類（`B`）的原型物件（`prototype`屬性）是父類的原型物件（`prototype`屬性）的實例。

```javascript
Object.create(A.prototype);
// 等同於
B.prototype.__proto__ = A.prototype;
```

### extends 的繼承目標

`extends`關鍵字後面可以跟多種類型的值。

```javascript
class B extends A {
}
```

上面代碼的`A`，只要是一個有`prototype`屬性的函數，就能被`B`繼承。由於函數都有`prototype`屬性（除了`Function.prototype`函數），因此`A`可以是任意涵數。

下面，討論三種特殊情況。

第一種特殊情況，子類繼承`Object`類。

```javascript
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

這種情況下，`A`其實就是構造函數`Object`的複製，`A`的實例就是`Object`的實例。

第二種特殊情況，不存在任何繼承。

```javascript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

這種情況下，`A`作為一個基類（即不存在任何繼承），就是一個普通函數，所以直接繼承`Function.prototype`。但是，`A`調用後返回一個空物件（即`Object`實例），所以`A.prototype.__proto__`指向構造函數（`Object`）的`prototype`屬性。

第三種特殊情況，子類繼承`null`。

```javascript
class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === undefined // true
```

這種情況與第二種情況非常像。`A`也是一個普通函數，所以直接繼承`Function.prototype`。但是，`A`調用後返回的物件不繼承任何方法，所以它的`__proto__`指向`undefined`，即實質上執行了下面的代碼。

```javascript
class C extends null {
  constructor() { return Object.create(null); }
}
```

### 實例的 \_\_proto\_\_ 屬性

子類實例的`__proto__`屬性的`__proto__`屬性，指向父類實例的`__proto__`屬性。也就是說，子類的原型的原型，是父類的原型。

```javascript
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto__ // false
p2.__proto__.__proto__ === p1.__proto__ // true
```

上面代碼中，`ColorPoint`繼承了`Point`，導致前者原型的原型是後者的原型。

因此，通過子類實例的`__proto__.__proto__`屬性，可以修改父類實例的行為。

```javascript
p2.__proto__.__proto__.printName = function () {
  console.log('Ha');
};

p1.printName() // "Ha"
```

上面代碼在`ColorPoint`的實例`p2`上向`Point`類添加方法，結果影響到了`Point`的實例`p1`。

## 原生構造函數的繼承

原生構造函數是指語言內置的構造函數，通常用來生成數據結構。ECMAScript 的原生構造函數大致有下面這些。

- Boolean()
- Number()
- String()
- Array()
- Date()
- Function()
- RegExp()
- Error()
- Object()

以前，這些原生構造函數是無法繼承的，比如，不能自己定義一個`Array`的子類。

```javascript
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});
```

上面代碼定義了一個繼承 Array 的`MyArray`類。但是，這個類的行為與`Array`完全不一致。

```javascript
var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```

之所以會發生這種情況，是因為子類無法獲得原生構造函數的內部屬性，通過`Array.apply()`或者分配給原型物件都不行。原生構造函數會忽略`apply`方法傳入的`this`，也就是說，原生構造函數的`this`無法綁定，導致拿不到內部屬性。

ES5 是先新建子類的實例物件`this`，再將父類的屬性添加到子類上，由於父類的內部屬性無法獲取，導致無法繼承原生的構造函數。比如，`Array`構造函數有一個內部屬性`[[DefineOwnProperty]]`，用來定義新屬性時，更新`length`屬性，這個內部屬性無法在子類獲取，導致子類的`length`屬性行為不正常。

下面的例子中，我們想讓一個普通物件繼承`Error`物件。

```javascript
var e = {};

Object.getOwnPropertyNames(Error.call(e))
// [ 'stack' ]

Object.getOwnPropertyNames(e)
// []
```

上面代碼中，我們想通過`Error.call(e)`這種寫法，讓普通物件`e`具有`Error`物件的實例屬性。但是，`Error.call()`完全忽略傳入的第一個參數，而是返回一個新物件，`e`本身沒有任何變化。這證明了`Error.call(e)`這種寫法，無法繼承原生構造函數。

ES6 允許繼承原生構造函數定義子類，因為 ES6 是先新建父類的實例物件`this`，然後再用子類的構造函數修飾`this`，使得父類的所有行為都可以繼承。下面是一個繼承`Array`的例子。

```javascript
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined
```

上面代碼定義了一個`MyArray`類，繼承了`Array`構造函數，因此就可以從`MyArray`生成陣列的實例。這意味著，ES6 可以自定義原生數據結構（比如`Array`、`String`等）的子類，這是 ES5 無法做到的。

上面這個例子也說明，`extends`關鍵字不僅可以用來繼承類，還可以用來繼承原生的構造函數。因此可以在原生數據結構的基礎上，定義自己的數據結構。下面就是定義了一個帶版本功能的陣列。

```javascript
class VersionedArray extends Array {
  constructor() {
    super();
    this.history = [[]];
  }
  commit() {
    this.history.push(this.slice());
  }
  revert() {
    this.splice(0, this.length, ...this.history[this.history.length - 1]);
  }
}

var x = new VersionedArray();

x.push(1);
x.push(2);
x // [1, 2]
x.history // [[]]

x.commit();
x.history // [[], [1, 2]]

x.push(3);
x // [1, 2, 3]
x.history // [[], [1, 2]]

x.revert();
x // [1, 2]
```

上面代碼中，`VersionedArray`會通過`commit`方法，將自己的當前狀態生成一個版本快照，存入`history`屬性。`revert`方法用來將陣列重置為最新一次保存的版本。除此之外，`VersionedArray`依然是一個普通陣列，所有原生的陣列方法都可以在它上面調用。

下面是一個自定義`Error`子類的例子，可以用來定製報錯時的行為。

```javascript
class ExtendableError extends Error {
  constructor(message) {
    super();
    this.message = message;
    this.stack = (new Error()).stack;
    this.name = this.constructor.name;
  }
}

class MyError extends ExtendableError {
  constructor(m) {
    super(m);
  }
}

var myerror = new MyError('ll');
myerror.message // "ll"
myerror instanceof Error // true
myerror.name // "MyError"
myerror.stack
// Error
//     at MyError.ExtendableError
//     ...
```

注意，繼承`Object`的子類，有一個[行為差異](http://stackoverflow.com/questions/36203614/super-does-not-pass-arguments-when-instantiating-a-class-extended-from-object)。

```javascript
class NewObj extends Object{
  constructor(){
    super(...arguments);
  }
}
var o = new NewObj({attr: true});
o.attr === true  // false
```

上面代碼中，`NewObj`繼承了`Object`，但是無法通過`super`方法向父類`Object`傳參。這是因為 ES6 改變了`Object`構造函數的行為，一旦發現`Object`方法不是通過`new Object()`這種形式調用，ES6 規定`Object`構造函數會忽略參數。

## Mixin 模式的實現

Mixin 指的是多個物件合成一個新的物件，新物件具有各個組成成員的接口。它的最簡單實現如下。

```javascript
const a = {
  a: 'a'
};
const b = {
  b: 'b'
};
const c = {...a, ...b}; // {a: 'a', b: 'b'}
```

上面代碼中，`c`物件是`a`物件和`b`物件的合成，具有兩者的接口。

下面是一個更完備的實現，將多個類的接口“混入”（mix in）另一個類。

```javascript
function mix(...mixins) {
  class Mix {}

  for (let mixin of mixins) {
    copyProperties(Mix, mixin); // 拷貝實例屬性
    copyProperties(Mix.prototype, mixin.prototype); // 拷貝原型屬性
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor"
      && key !== "prototype"
      && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```

上面代碼的`mix`函數，可以將多個物件合成為一個類。使用的時候，只要繼承這個類即可。

```javascript
class DistributedEdit extends mix(Loggable, Serializable) {
  // ...
}
```
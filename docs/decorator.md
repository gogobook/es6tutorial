# 修飾器

## 類的修飾

許多面向物件的語言都有修飾器（Decorator）函數，用來修改類的行為。目前，有一個[提案](https://github.com/tc39/proposal-decorators)將這項功能，引入了 ECMAScript。

```javascript
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

MyTestableClass.isTestable // true
```

上面代碼中，`@testable`就是一個修飾器。它修改了`MyTestableClass`這個類的行為，為它加上了靜態屬性`isTestable`。`testable`函數的參數`target`是`MyTestableClass`類本身。

基本上，修飾器的行為就是下面這樣。

```javascript
@decorator
class A {}

// 等同於

class A {}
A = decorator(A) || A;
```

也就是說，修飾器是一個對類進行處理的函數。修飾器函數的第一個參數，就是所要修飾的目標類。

```javascript
function testable(target) {
  // ...
}
```

上面代碼中，`testable`函數的參數`target`，就是會被修飾的類。

如果覺得一個參數不夠用，可以在修飾器外面再封裝一層函數。

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```

上面代碼中，修飾器`testable`可以接受參數，這就等於可以修改修飾器的行為。

注意，修飾器對類的行為的改變，是代碼編譯時發生的，而不是在運行時。這意味著，修飾器能在編譯階段運行代碼。也就是說，修飾器本質就是編譯時執行的函數。

前面的例子是為類添加一個靜態屬性，如果想添加實例屬性，可以通過目標類的`prototype`物件操作。

```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```

上面代碼中，修飾器函數`testable`是在目標類的`prototype`物件上添加屬性，因此就可以在實例上調用。

下面是另外一個例子。

```javascript
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

上面代碼通過修飾器`mixins`，把`Foo`物件的方法添加到了`MyClass`的實例上面。可以用`Object.assign()`模擬這個功能。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

實際開發中，React 與 Redux 庫結合使用時，常常需要寫成下面這樣。

```javascript
class MyReactComponent extends React.Component {}

export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
```

有了裝飾器，就可以改寫上面的代碼。

```javascript
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```

相對來說，後一種寫法看上去更容易理解。

## 方法的修飾

修飾器不僅可以修飾類，還可以修飾類的屬性。

```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
```

上面代碼中，修飾器`readonly`用來修飾“類”的`name`方法。

修飾器函數`readonly`一共可以接受三個參數。

```javascript
function readonly(target, name, descriptor){
  // descriptor物件原來的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 類似於
Object.defineProperty(Person.prototype, 'name', descriptor);
```

修飾器第一個參數是類的原型物件，上例是`Person.prototype`，修飾器的本意是要“修飾”類的實例，但是這個時候實例還沒生成，所以只能去修飾原型（這不同於類的修飾，那種情況時`target`參數指的是類本身）；第二個參數是所要修飾的屬性名，第三個參數是該屬性的描述物件。

另外，上面代碼說明，修飾器（readonly）會修改屬性的描述物件（descriptor），然後被修改的描述物件再用來定義屬性。

下面是另一個例子，修改屬性描述物件的`enumerable`屬性，使得該屬性不可遍歷。

```javascript
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```

下面的`@log`修飾器，可以起到輸出日誌的作用。

```javascript
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling ${name} with`, arguments);
    return oldValue.apply(this, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```

上面代碼中，`@log`修飾器的作用就是在執行原始的操作之前，執行一次`console.log`，從而達到輸出日誌的目的。

修飾器有註釋的作用。

```javascript
@testable
class Person {
  @readonly
  @nonenumerable
  name() { return `${this.first} ${this.last}` }
}
```

從上面代碼中，我們一眼就能看出，`Person`類是可測試的，而`name`方法是只讀和不可枚舉的。

下面是使用 Decorator 寫法的[組件](https://github.com/ionic-team/stencil)，看上去一目瞭然。

```javascript
@Component({
  tag: 'my-component',
  styleUrl: 'my-component.scss'
})
export class MyComponent {
  @Prop() first: string;
  @Prop() last: string;
  @State() isVisible: boolean = true;

  render() {
    return (
      <p>Hello, my name is {this.first} {this.last}</p>
    );
  }
}
```

如果同一個方法有多個修飾器，會像剝洋蔥一樣，先從外到內進入，然後由內向外執行。

```javascript
function dec(id){
  console.log('evaluated', id);
  return (target, property, descriptor) => console.log('executed', id);
}

class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```

上面代碼中，外層修飾器`@dec(1)`先進入，但是內層修飾器`@dec(2)`先執行。

除了註釋，修飾器還能用來類型檢查。所以，對於類來說，這項功能相當有用。從長期來看，它將是 JavaScript 代碼靜態分析的重要工具。

## 為什麼修飾器不能用於函數？

修飾器只能用於類和類的方法，不能用於函數，因為存在函數提升。

```javascript
var counter = 0;

var add = function () {
  counter++;
};

@add
function foo() {
}
```

上面的代碼，意圖是執行後`counter`等於 1，但是實際上結果是`counter`等於 0。因為函數提升，使得實際執行的代碼是下面這樣。

```javascript
@add
function foo() {
}

var counter;
var add;

counter = 0;

add = function () {
  counter++;
};
```

下面是另一個例子。

```javascript
var readOnly = require("some-decorator");

@readOnly
function foo() {
}
```

上面代碼也有問題，因為實際執行是下面這樣。

```javascript
var readOnly;

@readOnly
function foo() {
}

readOnly = require("some-decorator");
```

總之，由於存在函數提升，使得修飾器不能用於函數。類是不會提升的，所以就沒有這方面的問題。

另一方面，如果一定要修飾函數，可以採用高階函數的形式直接執行。

```javascript
function doSomething(name) {
  console.log('Hello, ' + name);
}

function loggingDecorator(wrapped) {
  return function() {
    console.log('Starting');
    const result = wrapped.apply(this, arguments);
    console.log('Finished');
    return result;
  }
}

const wrapped = loggingDecorator(doSomething);
```

## core-decorators.js

[core-decorators.js](https://github.com/jayphelps/core-decorators.js)是一個第三方模塊，提供了幾個常見的修飾器，通過它可以更好地理解修飾器。

**（1）@autobind**

`autobind`修飾器使得方法中的`this`物件，綁定原始物件。

```javascript
import { autobind } from 'core-decorators';

class Person {
  @autobind
  getPerson() {
    return this;
  }
}

let person = new Person();
let getPerson = person.getPerson;

getPerson() === person;
// true
```

**（2）@readonly**

`readonly`修飾器使得屬性或方法不可寫。

```javascript
import { readonly } from 'core-decorators';

class Meal {
  @readonly
  entree = 'steak';
}

var dinner = new Meal();
dinner.entree = 'salmon';
// Cannot assign to read only property 'entree' of [object Object]
```

**（3）@override**

`override`修飾器檢查子類的方法，是否正確覆蓋了父類的同名方法，如果不正確會報錯。

```javascript
import { override } from 'core-decorators';

class Parent {
  speak(first, second) {}
}

class Child extends Parent {
  @override
  speak() {}
  // SyntaxError: Child#speak() does not properly override Parent#speak(first, second)
}

// or

class Child extends Parent {
  @override
  speaks() {}
  // SyntaxError: No descriptor matching Child#speaks() was found on the prototype chain.
  //
  //   Did you mean "speak"?
}
```

**（4）@deprecate (別名@deprecated)**

`deprecate`或`deprecated`修飾器在控制台顯示一條警告，表示該方法將廢除。

```javascript
import { deprecate } from 'core-decorators';

class Person {
  @deprecate
  facepalm() {}

  @deprecate('We stopped facepalming')
  facepalmHard() {}

  @deprecate('We stopped facepalming', { url: 'http://knowyourmeme.com/memes/facepalm' })
  facepalmHarder() {}
}

let person = new Person();

person.facepalm();
// DEPRECATION Person#facepalm: This function will be removed in future versions.

person.facepalmHard();
// DEPRECATION Person#facepalmHard: We stopped facepalming

person.facepalmHarder();
// DEPRECATION Person#facepalmHarder: We stopped facepalming
//
//     See http://knowyourmeme.com/memes/facepalm for more details.
//
```

**（5）@suppressWarnings**

`suppressWarnings`修飾器抑制`deprecated`修飾器導致的`console.warn()`調用。但是，異步代碼發出的調用除外。

```javascript
import { suppressWarnings } from 'core-decorators';

class Person {
  @deprecated
  facepalm() {}

  @suppressWarnings
  facepalmWithoutWarning() {
    this.facepalm();
  }
}

let person = new Person();

person.facepalmWithoutWarning();
// no warning is logged
```

## 使用修飾器實現自動發佈事件

我們可以使用修飾器，使得物件的方法被調用時，自動發出一個事件。

```javascript
const postal = require("postal/lib/postal.lodash");

export default function publish(topic, channel) {
  const channelName = channel || '/';
  const msgChannel = postal.channel(channelName);
  msgChannel.subscribe(topic, v => {
    console.log('頻道: ', channelName);
    console.log('事件: ', topic);
    console.log('數據: ', v);
  });

  return function(target, name, descriptor) {
    const fn = descriptor.value;

    descriptor.value = function() {
      let value = fn.apply(this, arguments);
      msgChannel.publish(topic, value);
    };
  };
}
```

上面代碼定義了一個名為`publish`的修飾器，它通過改寫`descriptor.value`，使得原方法被調用時，會自動發出一個事件。它使用的事件“發佈/訂閱”庫是[Postal.js](https://github.com/postaljs/postal.js)。

它的用法如下。

```javascript
// index.js
import publish from './publish';

class FooComponent {
  @publish('foo.some.message', 'component')
  someMethod() {
    return { my: 'data' };
  }
  @publish('foo.some.other')
  anotherMethod() {
    // ...
  }
}

let foo = new FooComponent();

foo.someMethod();
foo.anotherMethod();
```

以後，只要調用`someMethod`或者`anotherMethod`，就會自動發出一個事件。

```bash
$ bash-node index.js
頻道:  component
事件:  foo.some.message
數據:  { my: 'data' }

頻道:  /
事件:  foo.some.other
數據:  undefined
```

## Mixin

在修飾器的基礎上，可以實現`Mixin`模式。所謂`Mixin`模式，就是物件繼承的一種替代方案，中文譯為“混入”（mix in），意為在一個物件之中混入另外一個物件的方法。

請看下面的例子。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

上面代碼之中，物件`Foo`有一個`foo`方法，通過`Object.assign`方法，可以將`foo`方法“混入”`MyClass`類，導致`MyClass`的實例`obj`物件都具有`foo`方法。這就是“混入”模式的一個簡單實現。

下面，我們部署一個通用腳本`mixins.js`，將 Mixin 寫成一個修飾器。

```javascript
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list);
  };
}
```

然後，就可以使用上面這個修飾器，為類“混入”各種方法。

```javascript
import { mixins } from './mixins';

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // "foo"
```

通過`mixins`這個修飾器，實現了在`MyClass`類上面“混入”`Foo`物件的`foo`方法。

不過，上面的方法會改寫`MyClass`類的`prototype`物件，如果不喜歡這一點，也可以通過類的繼承實現 Mixin。

```javascript
class MyClass extends MyBaseClass {
  /* ... */
}
```

上面代碼中，`MyClass`繼承了`MyBaseClass`。如果我們想在`MyClass`裡面“混入”一個`foo`方法，一個辦法是在`MyClass`和`MyBaseClass`之間插入一個混入類，這個類具有`foo`方法，並且繼承了`MyBaseClass`的所有方法，然後`MyClass`再繼承這個類。

```javascript
let MyMixin = (superclass) => class extends superclass {
  foo() {
    console.log('foo from MyMixin');
  }
};
```

上面代碼中，`MyMixin`是一個混入類生成器，接受`superclass`作為參數，然後返回一個繼承`superclass`的子類，該子類包含一個`foo`方法。

接著，目標類再去繼承這個混入類，就達到了“混入”`foo`方法的目的。

```javascript
class MyClass extends MyMixin(MyBaseClass) {
  /* ... */
}

let c = new MyClass();
c.foo(); // "foo from MyMixin"
```

如果需要“混入”多個方法，就生成多個混入類。

```javascript
class MyClass extends Mixin1(Mixin2(MyBaseClass)) {
  /* ... */
}
```

這種寫法的一個好處，是可以調用`super`，因此可以避免在“混入”過程中覆蓋父類的同名方法。

```javascript
let Mixin1 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin1');
    if (super.foo) super.foo();
  }
};

let Mixin2 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin2');
    if (super.foo) super.foo();
  }
};

class S {
  foo() {
    console.log('foo from S');
  }
}

class C extends Mixin1(Mixin2(S)) {
  foo() {
    console.log('foo from C');
    super.foo();
  }
}
```

上面代碼中，每一次`混入`發生時，都調用了父類的`super.foo`方法，導致父類的同名方法沒有被覆蓋，行為被保留了下來。

```javascript
new C().foo()
// foo from C
// foo from Mixin1
// foo from Mixin2
// foo from S
```

## Trait

Trait 也是一種修飾器，效果與 Mixin 類似，但是提供更多功能，比如防止同名方法的衝突、排除混入某些方法、為混入的方法起別名等等。

下面採用[traits-decorator](https://github.com/CocktailJS/traits-decorator)這個第三方模塊作為例子。這個模塊提供的`traits`修飾器，不僅可以接受物件，還可以接受 ES6 類作為參數。

```javascript
import { traits } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') }
};

@traits(TFoo, TBar)
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
```

上面代碼中，通過`traits`修飾器，在`MyClass`類上面“混入”了`TFoo`類的`foo`方法和`TBar`物件的`bar`方法。

Trait 不允許“混入”同名方法。

```javascript
import { traits } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar)
class MyClass { }
// 報錯
// throw new Error('Method named: ' + methodName + ' is defined twice.');
//        ^
// Error: Method named: foo is defined twice.
```

上面代碼中，`TFoo`和`TBar`都有`foo`方法，結果`traits`修飾器報錯。

一種解決方法是排除`TBar`的`foo`方法。

```javascript
import { traits, excludes } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar::excludes('foo'))
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
```

上面代碼使用綁定運算符（::）在`TBar`上排除`foo`方法，混入時就不會報錯了。

另一種方法是為`TBar`的`foo`方法起一個別名。

```javascript
import { traits, alias } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar::alias({foo: 'aliasFoo'}))
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.aliasFoo() // foo
obj.bar() // bar
```

上面代碼為`TBar`的`foo`方法起了別名`aliasFoo`，於是`MyClass`也可以混入`TBar`的`foo`方法了。

`alias`和`excludes`方法，可以結合起來使用。

```javascript
@traits(TExample::excludes('foo','bar')::alias({baz:'exampleBaz'}))
class MyClass {}
```

上面代碼排除`了TExample`的`foo`方法和`bar`方法，為`baz`方法起了別名`exampleBaz`。

`as`方法則為上面的代碼提供了另一種寫法。

```javascript
@traits(TExample::as({excludes:['foo', 'bar'], alias: {baz: 'exampleBaz'}}))
class MyClass {}
```

## Babel 轉碼器的支持

目前，Babel 轉碼器已經支持 Decorator。

首先，安裝`babel-core`和`babel-plugin-transform-decorators`。由於後者包括在`babel-preset-stage-0`之中，所以改為安裝`babel-preset-stage-0`亦可。

```bash
$ npm install babel-core babel-plugin-transform-decorators
```

然後，設置配置文件`.babelrc`。

```javascript
{
  "plugins": ["transform-decorators"]
}
```

這時，Babel 就可以對 Decorator 轉碼了。

腳本中打開的命令如下。

```javascript
babel.transform("code", {plugins: ["transform-decorators"]})
```

Babel 的官方網站提供一個[在線轉碼器](https://babeljs.io/repl/)，只要勾選 Experimental，就能支持 Decorator 的在線轉碼。
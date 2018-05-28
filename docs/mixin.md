# Mixin

JavaScript 語言的設計是單一繼承，即子類只能繼承一個父類，不允許繼承多個父類。這種設計保證了物件繼承的層次結構是樹狀的，而不是複雜的[網狀結構](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)。

但是，這大大降低了編程的靈活性。因為實際開發中，有時不可避免，子類需要繼承多個父類。舉例來說，“貓”可以繼承“哺乳類動物”，也可以繼承“寵物”。

各種單一繼承的編程語言，有不同的多重繼承解決方案。比如，Java 語言也是子類只能繼承一個父類，但是還允許繼承多個界面（interface），這樣就間接實現了多重繼承。Interface 與父類一樣，也是一個類，只不過它只定義接口（method signature），不定義實現，因此又被稱為“抽象類”。凡是繼承於 Interface 的方法，都必須自己定義實現，否則就會報錯。這樣就避免了多重繼承的最大問題：多個父類的同名方法的碰撞（naming collision）。

JavaScript 語言沒有採用 Interface 的方案，而是通過代理（delegation）實現了從其他類引入方法。

```javascript
var Enumerable_first = function () {
  this.first = function () {
    return this[0];
  };
};

var list = ["foo", "bar", "baz"];
Enumerable_first.call(list); // explicit delegation
list.first() // "foo"
```

上面代碼中，`list`是一個陣列，本身並沒有`first`方法。通過`call`方法，可以把`Enumerable_first`裡面的方法，綁定到`list`，從而`list`就具有`first`方法。這就叫做“代理”（delegation），`list`物件代理了`Enumerable_first`的`first`方法。

## 含義

Mixin 這個名字來自於冰淇淋，在基本口味的冰淇淋上面混入其他口味，這就叫做 Mix-in。

它允許向一個類裡面注入一些代碼，使得一個類的功能能夠“混入”另一個類。實質上是多重繼承的一種解決方案，但是避免了多重繼承的複雜性，而且有利於代碼復用。

Mixin 就是一個正常的類，不僅定義了接口，還定義了接口的實現。

子類通過在`this`物件上面綁定方法，達到多重繼承的目的。

很多庫提供了 Mixin 功能。下面以 Lodash 為例。

```javascript
function vowels(string) {
  return /[aeiou]/i.test(this.value);
}

var obj = { value: 'hello' };
_.mixin(obj, {vowels: vowels})
obj.vowels() // true
```

上面代碼通過 Lodash 庫的`_.mixin`方法，讓`obj`物件繼承了`vowels`方法。

Underscore 的類似方法是`_.extend`。

```javascript
var Person = function (fName, lName) {
  this.firstName = fName;
  this.lastName = lName;
}

var sam = new Person('Sam', 'Lowry');

var NameMixin = {
  fullName: function () {
    return this.firstName + ' ' + this.lastName;
  },
  rename: function(first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};
_.extend(Person.prototype, NameMixin);
sam.rename('Samwise', 'Gamgee');
sam.fullName() // "Samwise Gamgee"
```

上面代碼通過`_.extend`方法，在`sam`物件上面（準確說是它的原型物件`Person.prototype`上面），混入了`NameMixin`類。

`extend`方法的實現非常簡單。

```javascript
function extend(destination, source) {
  for (var k in source) {
    if (source.hasOwnProperty(k)) {
      destination[k] = source[k];
    }
  }
  return destination;
}
```

上面代碼將`source`物件的所有方法，添加到`destination`物件。

## Trait

Trait 是另外一種多重繼承的解決方案。它與 Mixin 很相似，但是有一些細微的差別。

- Mixin 可以包含狀態（state），Trait 不包含，即 Trait 裡面的方法都是互不相干，可以線性包含的。比如，`Trait1`包含方法`A`和`B`，`Trait2`繼承了`Trait1`，同時還包含一個自己的方法`C`，實際上就等同於直接包含方法`A`、`B`、`C`。
- 對於同名方法的碰撞，Mixin 包含瞭解決規則，Trait 則是報錯。
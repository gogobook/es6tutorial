# Set 和 Map 數據結構

## Set

### 基本用法

ES6 提供了新的數據結構 Set。它類似於陣列，但是成員的值都是唯一的，沒有重複的值。

Set 本身是一個構造函數，用來生成 Set 數據結構。

```javascript
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4
```

上面代碼通過`add`方法向 Set 結構加入成員，結果表明 Set 結構不會添加重複的值。

Set 函數可以接受一個陣列（或者具有 iterable 接口的其他數據結構）作為參數，用來初始化。

```javascript
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5

// 例三
const set = new Set(document.querySelectorAll('div'));
set.size // 56

// 類似於
const set = new Set();
document
 .querySelectorAll('div')
 .forEach(div => set.add(div));
set.size // 56
```

上面代碼中，例一和例二都是`Set`函數接受陣列作為參數，例三是接受類似陣列的物件作為參數。

上面代碼也展示了一種去除陣列重複成員的方法。

```javascript
// 去除陣列的重複成員
[...new Set(array)]
```

向 Set 加入值的時候，不會發生類型轉換，所以`5`和`"5"`是兩個不同的值。Set 內部判斷兩個值是否不同，使用的算法叫做“Same-value-zero equality”，它類似於精確相等運算符（`===`），主要的區別是`NaN`等於自身，而精確相等運算符認為`NaN`不等於自身。

```javascript
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}
```

上面代碼向 Set 實例添加了兩個`NaN`，但是只能加入一個。這表明，在 Set 內部，兩個`NaN`是相等。

另外，兩個物件總是不相等的。

```javascript
let set = new Set();

set.add({});
set.size // 1

set.add({});
set.size // 2
```

上面代碼表示，由於兩個空物件不相等，所以它們被視為兩個值。

### Set 實例的屬性和方法

Set 結構的實例有以下屬性。

- `Set.prototype.constructor`：構造函數，默認就是`Set`函數。
- `Set.prototype.size`：返回`Set`實例的成員總數。

Set 實例的方法分為兩大類：操作方法（用於操作數據）和遍歷方法（用於遍歷成員）。下面先介紹四個操作方法。

- `add(value)`：添加某個值，返回 Set 結構本身。
- `delete(value)`：刪除某個值，返回一個布爾值，表示刪除是否成功。
- `has(value)`：返回一個布爾值，表示該值是否為`Set`的成員。
- `clear()`：清除所有成員，沒有返回值。

上面這些屬性和方法的實例如下。

```javascript
s.add(1).add(2).add(2);
// 注意2被加入了兩次

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false
```

下面是一個對比，看看在判斷是否包括一個鍵上面，`Object`結構和`Set`結構的寫法不同。

```javascript
// 物件的寫法
const properties = {
  'width': 1,
  'height': 1
};

if (properties[someName]) {
  // do something
}

// Set的寫法
const properties = new Set();

properties.add('width');
properties.add('height');

if (properties.has(someName)) {
  // do something
}
```

`Array.from`方法可以將 Set 結構轉為陣列。

```javascript
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);
```

這就提供了去除陣列重複成員的另一種方法。

```javascript
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

### 遍歷操作

Set 結構的實例有四個遍歷方法，可以用於遍歷成員。

- `keys()`：返回鍵名的遍歷器
- `values()`：返回鍵值的遍歷器
- `entries()`：返回鍵值對的遍歷器
- `forEach()`：使用回調函數遍歷每個成員

需要特別指出的是，`Set`的遍歷順序就是插入順序。這個特性有時非常有用，比如使用 Set 保存一個回調函數列表，調用時就能保證按照添加順序調用。

**（1）`keys()`，`values()`，`entries()`**

`keys`方法、`values`方法、`entries`方法返回的都是遍歷器物件（詳見《Iterator 物件》一章）。由於 Set 結構沒有鍵名，只有鍵值（或者說鍵名和鍵值是同一個值），所以`keys`方法和`values`方法的行為完全一致。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

上面代碼中，`entries`方法返回的遍歷器，同時包括鍵名和鍵值，所以每次輸出一個陣列，它的兩個成員完全相等。

Set 結構的實例默認可遍歷，它的默認遍歷器生成函數就是它的`values`方法。

```javascript
Set.prototype[Symbol.iterator] === Set.prototype.values
// true
```

這意味著，可以省略`values`方法，直接用`for...of`循環遍歷 Set。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let x of set) {
  console.log(x);
}
// red
// green
// blue
```

**（2）`forEach()`**

Set 結構的實例與陣列一樣，也擁有`forEach`方法，用於對每個成員執行某種操作，沒有返回值。

```javascript
set = new Set([1, 4, 9]);
set.forEach((value, key) => console.log(key + ' : ' + value))
// 1 : 1
// 4 : 4
// 9 : 9
```

上面代碼說明，`forEach`方法的參數就是一個處理函數。該函數的參數與陣列的`forEach`一致，依次為鍵值、鍵名、集合本身（上例省略了該參數）。這裡需要注意，Set 結構的鍵名就是鍵值（兩者是同一個值），因此第一個參數與第二個參數的值永遠都是一樣的。

另外，`forEach`方法還可以有第二個參數，表示綁定處理函數內部的`this`物件。

**（3）遍歷的應用**

擴展運算符（`...`）內部使用`for...of`循環，所以也可以用於 Set 結構。

```javascript
let set = new Set(['red', 'green', 'blue']);
let arr = [...set];
// ['red', 'green', 'blue']
```

擴展運算符和 Set 結構相結合，就可以去除陣列的重複成員。

```javascript
let arr = [3, 5, 2, 2, 5, 5];
let unique = [...new Set(arr)];
// [3, 5, 2]
```

而且，陣列的`map`和`filter`方法也可以間接用於 Set 了。

```javascript
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// 返回Set結構：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// 返回Set結構：{2, 4}
```

因此使用 Set 可以很容易地實現並集（Union）、交集（Intersect）和差集（Difference）。

```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 並集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

如果想在遍歷操作中，同步改變原來的 Set 結構，目前沒有直接的方法，但有兩種變通方法。一種是利用原 Set 結構映射出一個新的結構，然後賦值給原來的 Set 結構；另一種是利用`Array.from`方法。

```javascript
// 方法一
let set = new Set([1, 2, 3]);
set = new Set([...set].map(val => val * 2));
// set的值是2, 4, 6

// 方法二
let set = new Set([1, 2, 3]);
set = new Set(Array.from(set, val => val * 2));
// set的值是2, 4, 6
```

上面代碼提供了兩種方法，直接在遍歷操作中改變原來的 Set 結構。

## WeakSet

### 含義

WeakSet 結構與 Set 類似，也是不重複的值的集合。但是，它與 Set 有兩個區別。

首先，WeakSet 的成員只能是物件，而不能是其他類型的值。

```javascript
const ws = new WeakSet();
ws.add(1)
// TypeError: Invalid value used in weak set
ws.add(Symbol())
// TypeError: invalid value used in weak set
```

上面代碼試圖向 WeakSet 添加一個數值和`Symbol`值，結果報錯，因為 WeakSet 只能放置物件。

其次，WeakSet 中的物件都是弱引用，即垃圾回收機制不考慮 WeakSet 對該物件的引用，也就是說，如果其他物件都不再引用該物件，那麼垃圾回收機制會自動回收該物件所佔用的內存，不考慮該物件還存在於 WeakSet 之中。

這是因為垃圾回收機制依賴引用計數，如果一個值的引用次數不為`0`，垃圾回收機制就不會釋放這塊內存。結束使用該值之後，有時會忘記取消引用，導致內存無法釋放，進而可能會引發內存洩漏。WeakSet 裡面的引用，都不計入垃圾回收機制，所以就不存在這個問題。因此，WeakSet 適合臨時存放一組物件，以及存放跟物件綁定的信息。只要這些物件在外部消失，它在 WeakSet 裡面的引用就會自動消失。

由於上面這個特點，WeakSet 的成員是不適合引用的，因為它會隨時消失。另外，由於 WeakSet 內部有多少個成員，取決於垃圾回收機制有沒有運行，運行前後很可能成員個數是不一樣的，而垃圾回收機制何時運行是不可預測的，因此 ES6 規定 WeakSet 不可遍歷。

這些特點同樣適用於本章後面要介紹的 WeakMap 結構。

### 語法

WeakSet 是一個構造函數，可以使用`new`命令，創建 WeakSet 數據結構。

```javascript
const ws = new WeakSet();
```

作為構造函數，WeakSet 可以接受一個陣列或類似陣列的物件作為參數。（實際上，任何具有 Iterable 接口的物件，都可以作為 WeakSet 的參數。）該陣列的所有成員，都會自動成為 WeakSet 實例物件的成員。

```javascript
const a = [[1, 2], [3, 4]];
const ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}
```

上面代碼中，`a`是一個陣列，它有兩個成員，也都是陣列。將`a`作為 WeakSet 構造函數的參數，`a`的成員會自動成為 WeakSet 的成員。

注意，是`a`陣列的成員成為 WeakSet 的成員，而不是`a`陣列本身。這意味著，陣列的成員只能是物件。

```javascript
const b = [3, 4];
const ws = new WeakSet(b);
// Uncaught TypeError: Invalid value used in weak set(…)
```

上面代碼中，陣列`b`的成員不是物件，加入 WeaKSet 就會報錯。

WeakSet 結構有以下三個方法。

- **WeakSet.prototype.add(value)**：向 WeakSet 實例添加一個新成員。
- **WeakSet.prototype.delete(value)**：清除 WeakSet 實例的指定成員。
- **WeakSet.prototype.has(value)**：返回一個布爾值，表示某個值是否在 WeakSet 實例之中。

下面是一個例子。

```javascript
const ws = new WeakSet();
const obj = {};
const foo = {};

ws.add(window);
ws.add(obj);

ws.has(window); // true
ws.has(foo);    // false

ws.delete(window);
ws.has(window);    // false
```

WeakSet 沒有`size`屬性，沒有辦法遍歷它的成員。

```javascript
ws.size // undefined
ws.forEach // undefined

ws.forEach(function(item){ console.log('WeakSet has ' + item)})
// TypeError: undefined is not a function
```

上面代碼試圖獲取`size`和`forEach`屬性，結果都不能成功。

WeakSet 不能遍歷，是因為成員都是弱引用，隨時可能消失，遍歷機制無法保證成員的存在，很可能剛剛遍歷結束，成員就取不到了。WeakSet 的一個用處，是儲存 DOM 節點，而不用擔心這些節點從文檔移除時，會引發內存洩漏。

下面是 WeakSet 的另一個例子。

```javascript
const foos = new WeakSet()
class Foo {
  constructor() {
    foos.add(this)
  }
  method () {
    if (!foos.has(this)) {
      throw new TypeError('Foo.prototype.method 只能在Foo的實例上調用！');
    }
  }
}
```

上面代碼保證了`Foo`的實例方法，只能在`Foo`的實例上調用。這裡使用 WeakSet 的好處是，`foos`對實例的引用，不會被計入內存回收機制，所以刪除實例的時候，不用考慮`foos`，也不會出現內存洩漏。

## Map

### 含義和基本用法

JavaScript 的物件（Object），本質上是鍵值對的集合（Hash 結構），但是傳統上只能用字符串當作鍵。這給它的使用帶來了很大的限制。

```javascript
const data = {};
const element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

上面代碼原意是將一個 DOM 節點作為物件`data`的鍵，但是由於物件只接受字符串作為鍵名，所以`element`被自動轉為字符串`[object HTMLDivElement]`。

為瞭解決這個問題，ES6 提供了 Map 數據結構。它類似於物件，也是鍵值對的集合，但是“鍵”的範圍不限於字符串，各種類型的值（包括物件）都可以當作鍵。也就是說，Object 結構提供了“字符串—值”的對應，Map 結構提供了“值—值”的對應，是一種更完善的 Hash 結構實現。如果你需要“鍵值對”的數據結構，Map 比 Object 更合適。

```javascript
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

上面代碼使用 Map 結構的`set`方法，將物件`o`當作`m`的一個鍵，然後又使用`get`方法讀取這個鍵，接著使用`delete`方法刪除了這個鍵。

上面的例子展示了如何向 Map 添加成員。作為構造函數，Map 也可以接受一個陣列作為參數。該陣列的成員是一個個表示鍵值對的陣列。

```javascript
const map = new Map([
  ['name', '張三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "張三"
map.has('title') // true
map.get('title') // "Author"
```

上面代碼在新建 Map 實例時，就指定了兩個鍵`name`和`title`。

`Map`構造函數接受陣列作為參數，實際上執行的是下面的算法。

```javascript
const items = [
  ['name', '張三'],
  ['title', 'Author']
];

const map = new Map();

items.forEach(
  ([key, value]) => map.set(key, value)
);
```

事實上，不僅僅是陣列，任何具有 Iterator 接口、且每個成員都是一個雙元素的陣列的數據結構（詳見《Iterator》一章）都可以當作`Map`構造函數的參數。這就是說，`Set`和`Map`都可以用來生成新的 Map。

```javascript
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```

上面代碼中，我們分別使用 Set 物件和 Map 物件，當作`Map`構造函數的參數，結果都生成了新的 Map 物件。

如果對同一個鍵多次賦值，後面的值將覆蓋前面的值。

```javascript
const map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"
```

上面代碼對鍵`1`連續賦值兩次，後一次的值覆蓋前一次的值。

如果讀取一個未知的鍵，則返回`undefined`。

```javascript
new Map().get('asfddfsasadf')
// undefined
```

注意，只有對同一個物件的引用，Map 結構才將其視為同一個鍵。這一點要非常小心。

```javascript
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```

上面代碼的`set`和`get`方法，表面是針對同一個鍵，但實際上這是兩個值，內存地址是不一樣的，因此`get`方法無法讀取該鍵，返回`undefined`。

同理，同樣的值的兩個實例，在 Map 結構中被視為兩個鍵。

```javascript
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```

上面代碼中，變數`k1`和`k2`的值是一樣的，但是它們在 Map 結構中被視為兩個鍵。

由上可知，Map 的鍵實際上是跟內存地址綁定的，只要內存地址不一樣，就視為兩個鍵。這就解決了同名屬性碰撞（clash）的問題，我們擴展別人的庫的時候，如果使用物件作為鍵名，就不用擔心自己的屬性與原作者的屬性同名。

如果 Map 的鍵是一個簡單類型的值（數字、字符串、布爾值），則只要兩個值嚴格相等，Map 將其視為一個鍵，比如`0`和`-0`就是一個鍵，布爾值`true`和字符串`true`則是兩個不同的鍵。另外，`undefined`和`null`也是兩個不同的鍵。雖然`NaN`不嚴格相等於自身，但 Map 將其視為同一個鍵。

```javascript
let map = new Map();

map.set(-0, 123);
map.get(+0) // 123

map.set(true, 1);
map.set('true', 2);
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123
```

### 實例的屬性和操作方法

Map 結構的實例有以下屬性和操作方法。

**（1）size 屬性**

`size`屬性返回 Map 結構的成員總數。

```javascript
const map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
```

**（2）set(key, value)**

`set`方法設置鍵名`key`對應的鍵值為`value`，然後返回整個 Map 結構。如果`key`已經有值，則鍵值會被更新，否則就新生成該鍵。

```javascript
const m = new Map();

m.set('edition', 6)        // 鍵是字符串
m.set(262, 'standard')     // 鍵是數值
m.set(undefined, 'nah')    // 鍵是 undefined
```

`set`方法返回的是當前的`Map`物件，因此可以採用鏈式寫法。

```javascript
let map = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');
```

**（3）get(key)**

`get`方法讀取`key`對應的鍵值，如果找不到`key`，返回`undefined`。

```javascript
const m = new Map();

const hello = function() {console.log('hello');};
m.set(hello, 'Hello ES6!') // 鍵是函數

m.get(hello)  // Hello ES6!
```

**（4）has(key)**

`has`方法返回一個布爾值，表示某個鍵是否在當前 Map 物件之中。

```javascript
const m = new Map();

m.set('edition', 6);
m.set(262, 'standard');
m.set(undefined, 'nah');

m.has('edition')     // true
m.has('years')       // false
m.has(262)           // true
m.has(undefined)     // true
```

**（5）delete(key)**

`delete`方法刪除某個鍵，返回`true`。如果刪除失敗，返回`false`。

```javascript
const m = new Map();
m.set(undefined, 'nah');
m.has(undefined)     // true

m.delete(undefined)
m.has(undefined)       // false
```

**（6）clear()**

`clear`方法清除所有成員，沒有返回值。

```javascript
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
map.clear()
map.size // 0
```

### 遍歷方法

Map 結構原生提供三個遍歷器生成函數和一個遍歷方法。

- `keys()`：返回鍵名的遍歷器。
- `values()`：返回鍵值的遍歷器。
- `entries()`：返回所有成員的遍歷器。
- `forEach()`：遍歷 Map 的所有成員。

需要特別注意的是，Map 的遍歷順序就是插入順序。

```javascript
const map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"

// 等同於使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"
```

上面代碼最後的那個例子，表示 Map 結構的默認遍歷器接口（`Symbol.iterator`屬性），就是`entries`方法。

```javascript
map[Symbol.iterator] === map.entries
// true
```

Map 結構轉為陣列結構，比較快速的方法是使用擴展運算符（`...`）。

```javascript
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```

結合陣列的`map`方法、`filter`方法，可以實現 Map 的遍歷和過濾（Map 本身沒有`map`和`filter`方法）。

```javascript
const map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

const map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 產生 Map 結構 {1 => 'a', 2 => 'b'}

const map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 產生 Map 結構 {2 => '_a', 4 => '_b', 6 => '_c'}
```

此外，Map 還有一個`forEach`方法，與陣列的`forEach`方法類似，也可以實現遍歷。

```javascript
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```

`forEach`方法還可以接受第二個參數，用來綁定`this`。

```javascript
const reporter = {
  report: function(key, value) {
    console.log("Key: %s, Value: %s", key, value);
  }
};

map.forEach(function(value, key, map) {
  this.report(key, value);
}, reporter);
```

上面代碼中，`forEach`方法的回調函數的`this`，就指向`reporter`。

### 與其他數據結構的互相轉換

**（1）Map 轉為陣列**

前面已經提過，Map 轉為陣列最方便的方法，就是使用擴展運算符（`...`）。

```javascript
const myMap = new Map()
  .set(true, 7)
  .set({foo: 3}, ['abc']);
[...myMap]
// [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

**（2）陣列 轉為 Map**

將陣列傳入 Map 構造函數，就可以轉為 Map。

```javascript
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

**（3）Map 轉為物件**

如果所有 Map 的鍵都是字符串，它可以無損地轉為物件。

```javascript
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```

如果有非字符串的鍵名，那麼這個鍵名會被轉成字符串，再作為物件的鍵名。

**（4）物件轉為 Map**

```javascript
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({yes: true, no: false})
// Map {"yes" => true, "no" => false}
```

**（5）Map 轉為 JSON**

Map 轉為 JSON 要區分兩種情況。一種情況是，Map 的鍵名都是字符串，這時可以選擇轉為物件 JSON。

```javascript
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)
// '{"yes":true,"no":false}'
```

另一種情況是，Map 的鍵名有非字符串，這時可以選擇轉為陣列 JSON。

```javascript
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap)
// '[[true,7],[{"foo":3},["abc"]]]'
```

**（6）JSON 轉為 Map**

JSON 轉為 Map，正常情況下，所有鍵名都是字符串。

```javascript
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes": true, "no": false}')
// Map {'yes' => true, 'no' => false}
```

但是，有一種特殊情況，整個 JSON 就是一個陣列，且每個陣列成員本身，又是一個有兩個成員的陣列。這時，它可以一一對應地轉為 Map。這往往是 Map 轉為陣列 JSON 的逆操作。

```javascript
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
// Map {true => 7, Object {foo: 3} => ['abc']}
```

## WeakMap

### 含義

`WeakMap`結構與`Map`結構類似，也是用於生成鍵值對的集合。

```javascript
// WeakMap 可以使用 set 方法添加成員
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2

// WeakMap 也可以接受一個陣列，
// 作為構造函數的參數
const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
wm2.get(k2) // "bar"
```

`WeakMap`與`Map`的區別有兩點。

首先，`WeakMap`只接受物件作為鍵名（`null`除外），不接受其他類型的值作為鍵名。

```javascript
const map = new WeakMap();
map.set(1, 2)
// TypeError: 1 is not an object!
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
map.set(null, 2)
// TypeError: Invalid value used as weak map key
```

上面代碼中，如果將數值`1`和`Symbol`值作為 WeakMap 的鍵名，都會報錯。

其次，`WeakMap`的鍵名所指向的物件，不計入垃圾回收機制。

`WeakMap`的設計目的在於，有時我們想在某個物件上面存放一些數據，但是這會形成對於這個物件的引用。請看下面的例子。

```javascript
const e1 = document.getElementById('foo');
const e2 = document.getElementById('bar');
const arr = [
  [e1, 'foo 元素'],
  [e2, 'bar 元素'],
];
```

上面代碼中，`e1`和`e2`是兩個物件，我們通過`arr`陣列對這兩個物件添加一些文字說明。這就形成了`arr`對`e1`和`e2`的引用。

一旦不再需要這兩個物件，我們就必須手動刪除這個引用，否則垃圾回收機制就不會釋放`e1`和`e2`佔用的內存。

```javascript
// 不需要 e1 和 e2 的時候
// 必須手動刪除引用
arr [0] = null;
arr [1] = null;
```

上面這樣的寫法顯然很不方便。一旦忘了寫，就會造成內存洩露。

WeakMap 就是為瞭解決這個問題而誕生的，它的鍵名所引用的物件都是弱引用，即垃圾回收機制不將該引用考慮在內。因此，只要所引用的物件的其他引用都被清除，垃圾回收機制就會釋放該物件所佔用的內存。也就是說，一旦不再需要，WeakMap 裡面的鍵名物件和所對應的鍵值對會自動消失，不用手動刪除引用。

基本上，如果你要往物件上添加數據，又不想幹擾垃圾回收機制，就可以使用 WeakMap。一個典型應用場景是，在網頁的 DOM 元素上添加數據，就可以使用`WeakMap`結構。當該 DOM 元素被清除，其所對應的`WeakMap`記錄就會自動被移除。

```javascript
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```

上面代碼中，先新建一個 Weakmap 實例。然後，將一個 DOM 節點作為鍵名存入該實例，並將一些附加信息作為鍵值，一起存放在 WeakMap 裡面。這時，WeakMap 裡面對`element`的引用就是弱引用，不會被計入垃圾回收機制。

也就是說，上面的 DOM 節點物件的引用計數是`1`，而不是`2`。這時，一旦消除對該節點的引用，它佔用的內存就會被垃圾回收機制釋放。Weakmap 保存的這個鍵值對，也會自動消失。

總之，`WeakMap`的專用場合就是，它的鍵所對應的物件，可能會在將來消失。`WeakMap`結構有助於防止內存洩漏。

注意，WeakMap 弱引用的只是鍵名，而不是鍵值。鍵值依然是正常引用。

```javascript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```

上面代碼中，鍵值`obj`是正常引用。所以，即使在 WeakMap 外部消除了`obj`的引用，WeakMap 內部的引用依然存在。

### WeakMap 的語法

WeakMap 與 Map 在 API 上的區別主要是兩個，一是沒有遍歷操作（即沒有`keys()`、`values()`和`entries()`方法），也沒有`size`屬性。因為沒有辦法列出所有鍵名，某個鍵名是否存在完全不可預測，跟垃圾回收機制是否運行相關。這一刻可以取到鍵名，下一刻垃圾回收機制突然運行了，這個鍵名就沒了，為了防止出現不確定性，就統一規定不能取到鍵名。二是無法清空，即不支持`clear`方法。因此，`WeakMap`只有四個方法可用：`get()`、`set()`、`has()`、`delete()`。

```javascript
const wm = new WeakMap();

// size、forEach、clear 方法都不存在
wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
```

### WeakMap 的示例

WeakMap 的例子很難演示，因為無法觀察它裡面的引用會自動消失。此時，其他引用都解除了，已經沒有引用指向 WeakMap 的鍵名了，導致無法證實那個鍵名是不是存在。

賀師俊老師[提示](https://github.com/ruanyf/es6tutorial/issues/362#issuecomment-292109104)，如果引用所指向的值佔用特別多的內存，就可以通過 Node 的`process.memoryUsage`方法看出來。根據這個思路，網友[vtxf](https://github.com/ruanyf/es6tutorial/issues/362#issuecomment-292451925)補充了下面的例子。

首先，打開 Node 命令行。

```bash
$ node --expose-gc
```

上面代碼中，`--expose-gc`參數表示允許手動執行垃圾回收機制。

然後，執行下面的代碼。

```javascript
// 手動執行一次垃圾回收，保證獲取的內存使用狀態準確
> global.gc();
undefined

// 查看內存佔用的初始狀態，heapUsed 為 4M 左右
> process.memoryUsage();
{ rss: 21106688,
  heapTotal: 7376896,
  heapUsed: 4153936,
  external: 9059 }

> let wm = new WeakMap();
undefined

// 新建一個變數 key，指向一個 5*1024*1024 的陣列
> let key = new Array(5 * 1024 * 1024);
undefined

// 設置 WeakMap 實例的鍵名，也指向 key 陣列
// 這時，key 陣列實際被引用了兩次，
// 變數 key 引用一次，WeakMap 的鍵名引用了第二次
// 但是，WeakMap 是弱引用，對於引擎來說，引用計數還是1
> wm.set(key, 1);
WeakMap {}

> global.gc();
undefined

// 這時內存佔用 heapUsed 增加到 45M 了
> process.memoryUsage();
{ rss: 67538944,
  heapTotal: 7376896,
  heapUsed: 45782816,
  external: 8945 }

// 清除變數 key 對陣列的引用，
// 但沒有手動清除 WeakMap 實例的鍵名對陣列的引用
> key = null;
null

// 再次執行垃圾回收
> global.gc();
undefined

// 內存佔用 heapUsed 變回 4M 左右，
// 可以看到 WeakMap 的鍵名引用沒有阻止 gc 對內存的回收
> process.memoryUsage();
{ rss: 20639744,
  heapTotal: 8425472,
  heapUsed: 3979792,
  external: 8956 }
```

上面代碼中，只要外部的引用消失，WeakMap 內部的引用，就會自動被垃圾回收清除。由此可見，有了 WeakMap 的幫助，解決內存洩漏就會簡單很多。

### WeakMap 的用途

前文說過，WeakMap 應用的典型場合就是 DOM 節點作為鍵名。下面是一個例子。

```javascript
let myElement = document.getElementById('logo');
let myWeakmap = new WeakMap();

myWeakmap.set(myElement, {timesClicked: 0});

myElement.addEventListener('click', function() {
  let logoData = myWeakmap.get(myElement);
  logoData.timesClicked++;
}, false);
```

上面代碼中，`myElement`是一個 DOM 節點，每當發生`click`事件，就更新一下狀態。我們將這個狀態作為鍵值放在 WeakMap 裡，對應的鍵名就是`myElement`。一旦這個 DOM 節點刪除，該狀態就會自動消失，不存在內存洩漏風險。

WeakMap 的另一個用處是部署私有屬性。

```javascript
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```

上面代碼中，`Countdown`類的兩個內部屬性`_counter`和`_action`，是實例的弱引用，所以如果刪除實例，它們也就隨之消失，不會造成內存洩漏。
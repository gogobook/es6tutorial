# 最新提案

本章介紹一些尚未進入標準、但很有希望的最新提案。

## do 表達式

本質上，塊級作用域是一個語句，將多個操作封裝在一起，沒有返回值。

```javascript
{
  let t = f();
  t = t * t + 1;
}
```

上面代碼中，塊級作用域將兩個語句封裝在一起。但是，在塊級作用域以外，沒有辦法得到`t`的值，因為塊級作用域不返回值，除非`t`是全局變數。

現在有一個[提案](https://github.com/tc39/proposal-do-expressions)，使得塊級作用域可以變為表達式，也就是說可以返回值，辦法就是在塊級作用域之前加上`do`，使它變為`do`表達式，然後就會返回內部最後執行的表達式的值。

```javascript
let x = do {
  let t = f();
  t * t + 1;
};
```

上面代碼中，變數`x`會得到整個塊級作用域的返回值（`t * t + 1`）。

`do`表達式的邏輯非常簡單：封裝的是什麼，就會返回什麼。

```javascript
// 等同於 <表達式>
do { <表達式>; }

// 等同於 <語句>
do { <語句> }
```

`do`表達式的好處是可以封裝多個語句，讓程序更加模塊化，就像樂高積木那樣一塊塊拼裝起來。

```javascript
let x = do {
  if (foo()) { f() }
  else if (bar()) { g() }
  else { h() }
};
```

上面代碼的本質，就是根據函數`foo`的執行結果，調用不同的函數，將返回結果賦給變數`x`。使用`do`表達式，就將這個操作的意圖表達得非常簡潔清晰。而且，`do`塊級作用域提供了單獨的作用域，內部操作可以與全局作用域隔絕。

值得一提的是，`do`表達式在 JSX 語法中非常好用。

```javascript
return (
  <nav>
    <Home />
    {
      do {
        if (loggedIn) {
          <LogoutButton />
        } else {
          <LoginButton />
        }
      }
    }
  </nav>
)
```

上面代碼中，如果不用`do`表達式，就只能用三元判斷運算符（`?:`）。那樣的話，一旦判斷邏輯複雜，代碼就會變得很不易讀。

## throw 表達式

JavaScript 語法規定`throw`是一個命令，用來拋出錯誤，不能用於表達式之中。

```javascript
// 報錯
console.log(throw new Error());
```

上面代碼中，`console.log`的參數必須是一個表達式，如果是一個`throw`語句就會報錯。

現在有一個[提案](https://github.com/tc39/proposal-throw-expressions)，允許`throw`用於表達式。

```javascript
// 參數的默認值
function save(filename = throw new TypeError("Argument required")) {
}

// 箭頭函數的返回值
lint(ast, {
  with: () => throw new Error("avoid using 'with' statements.")
});

// 條件表達式
function getEncoder(encoding) {
  const encoder = encoding === "utf8" ?
    new UTF8Encoder() :
    encoding === "utf16le" ?
      new UTF16Encoder(false) :
      encoding === "utf16be" ?
        new UTF16Encoder(true) :
        throw new Error("Unsupported encoding");
}

// 邏輯表達式
class Product {
  get id() {
    return this._id;
  }
  set id(value) {
    this._id = value || throw new Error("Invalid value");
  }
}
```

上面代碼中，`throw`都出現在表達式裡面。

語法上，`throw`表達式裡面的`throw`不再是一個命令，而是一個運算符。為了避免與`throw`命令混淆，規定`throw`出現在行首，一律解釋為`throw`語句，而不是`throw`表達式。

## 鏈判斷運算符

編程實務中，如果讀取物件內部的某個屬性，往往需要判斷一下該物件是否存在。比如，要讀取`message.body.user.firstName`，安全的寫法是寫成下面這樣。

```javascript
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';
```

這樣的層層判斷非常麻煩，因此現在有一個[提案](https://github.com/tc39/proposal-optional-chaining)，引入了“鏈判斷運算符”（optional chaining operator）`?.`，簡化上面的寫法。

```javascript
const firstName = message?.body?.user?.firstName || 'default';
```

上面代碼有三個`?.`運算符，直接在鏈式調用的時候判斷，左側的物件是否為`null`或`undefined`。如果是的，就不再往下運算，而是返回`undefined`。

鏈判斷運算符號有三種用法。

- `obj?.prop` // 讀取物件屬性
- `obj?.[expr]` // 同上
- `func?.(...args)` // 函數或物件方法的調用

下面是判斷函數是否存在的例子。

```javascript
iterator.return?.()
```

上面代碼中，`iterator.return`如果有定義，就會調用該方法，否則直接返回`undefined`。

下面是更多的例子。

```javascript
a?.b
// 等同於
a == null ? undefined : a.b

a?.[x]
// 等同於
a == null ? undefined : a[x]

a?.b()
// 等同於
a == null ? undefined : a.b()

a?.()
// 等同於
a == null ? undefined : a()
```

使用這個運算符，有幾個注意點。

（1）短路機制

```javascript
a?.[++x]
// 等同於
a == null ? undefined : a[++x]
```

上面代碼中，如果`a`是`undefined`或`null`，那麼`x`不會進行遞增運算。也就是說，鏈判斷運算符一旦為真，右側的表達式就不再求值。

（2）delete 運算符

```javascript
delete a?.b
// 等同於
a == null ? undefined : delete a.b
```

上面代碼中，如果`a`是`undefined`或`null`，會直接返回`undefined`，而不會進行`delete`運算。

（3）報錯場合

以下寫法是禁止，會報錯。

```javascript
// 構造函數判斷
new a?.()

// 運算符右側是模板字符串
a?.`{b}`

// 鏈判斷運算符前後有構造函數或模板字符串
new a?.b()
a?.b`{c}`

// 鏈運算符用於賦值運算符左側
a?.b = c
```

（4）右側不得為十進制數值

為了保證兼容以前的代碼，允許`foo?.3:0`被解析成`foo ? .3 : 0`，因此規定如果`?.`後面緊跟一個十進制數字，那麼`?.`不再被看成是一個完整的運算符，而會按照三元運算符進行處理，也就是說，那個小數點會歸屬於後面的十進制數字，形成一個小數。

## 直接輸入 U+2028 和 U+2029

JavaScript 字符串允許直接輸入字符，以及輸入字符的轉義形式。舉例來說，“中”的 Unicode 碼點是 U+4e2d，你可以直接在字符串裡面輸入這個漢字，也可以輸入它的轉義形式`\u4e2d`，兩者是等價的。

```javascript
'中' === '\u4e2d' // true
```

但是，JavaScript 規定有5個字符，不能在字符串裡面直接使用，只能使用轉義形式。

- U+005C：反斜槓（reverse solidus)
- U+000D：回車（carriage return）
- U+2028：行分隔符（line separator）
- U+2029：段分隔符（paragraph separator）
- U+000A：換行符（line feed）

舉例來說，字符串裡面不能直接包含反斜槓，一定要轉義寫成`\\`或者`\u005c`。

這個規定本身沒有問題，麻煩在於 JSON 格式允許字符串裡面直接使用 U+2028（行分隔符）和 U+2029（段分隔符）。這樣一來，服務器輸出的 JSON 被`JSON.parse`解析，就有可能直接報錯。

JSON 格式已經凍結（RFC 7159），沒法修改了。為了消除這個報錯，現在有一個[提案](https://github.com/tc39/proposal-json-superset)，允許 JavaScript 字符串直接輸入 U+2028（行分隔符）和 U+2029（段分隔符）。

```javascript
const PS = eval("'\u2029'");
```

根據這個提案，上面的代碼不會報錯。

注意，模板字符串現在就允許直接輸入這兩個字符。另外，正則表達式依然不允許直接輸入這兩個字符，這是沒有問題的，因為 JSON 本來就不允許直接包含正則表達式。

## 函數的部分執行

### 語法

多參數的函數有時需要綁定其中的一個或多個參數，然後返回一個新函數。

```javascript
function add(x, y) { return x + y; }
function add7(x) { return x + 7; }
```

上面代碼中，`add7`函數其實是`add`函數的一個特殊版本，通過將一個參數綁定為`7`，就可以從`add`得到`add7`。

```javascript
// bind 方法
const add7 = add.bind(null, 7);

// 箭頭函數
const add7 = x => add(x, 7);
```

上面兩種寫法都有些冗餘。其中，`bind`方法的侷限更加明顯，它必須提供`this`，並且只能從前到後一個個綁定參數，無法只綁定非頭部的參數。

現在有一個[提案](https://github.com/tc39/proposal-partial-application)，使得綁定參數並返回一個新函數更加容易。這叫做函數的部分執行（partial application）。

```javascript
const add = (x, y) => x + y;
const addOne = add(1, ?);

const maxGreaterThanZero = Math.max(0, ...);
```

根據新提案，`?`是單個參數的佔位符，`...`是多個參數的佔位符。以下的形式都屬於函數的部分執行。

```javascript
f(x, ?)
f(x, ...)
f(?, x)
f(..., x)
f(?, x, ?)
f(..., x, ...)
```

`?`和`...`只能出現在函數的調用之中，並且會返回一個新函數。

```javascript
const g = f(?, 1, ...);
// 等同於
const g = (x, ...y) => f(x, 1, ...y);
```

函數的部分執行，也可以用於物件的方法。

```javascript
let obj = {
  f(x, y) { return x + y; },
};

const g = obj.f(?, 3);
g(1) // 4
```

### 注意點

函數的部分執行有一些特別注意的地方。

（1）函數的部分執行是基於原函數的。如果原函數發生變化，部分執行生成的新函數也會立即反映這種變化。

```javascript
let f = (x, y) => x + y;

const g = f(?, 3);
g(1); // 4

// 替換函數 f
f = (x, y) => x * y;

g(1); // 3
```

上面代碼中，定義了函數的部分執行以後，更換原函數會立即影響到新函數。

（2）如果預先提供的那個值是一個表達式，那麼這個表達式並不會在定義時求值，而是在每次調用時求值。

```javascript
let a = 3;
const f = (x, y) => x + y;

const g = f(?, a);
g(1); // 4

// 改變 a 的值
a = 10;
g(1); // 11
```

上面代碼中，預先提供的參數是變數`a`，那麼每次調用函數`g`的時候，才會對`a`進行求值。

（3）如果新函數的參數多於佔位符的數量，那麼多餘的參數將被忽略。

```javascript
const f = (x, ...y) => [x, ...y];
const g = f(?, 1);
g(2, 3, 4); // [2, 1]
```

上面代碼中，函數`g`只有一個佔位符，也就意味著它只能接受一個參數，多餘的參數都會被忽略。

寫成下面這樣，多餘的參數就沒有問題。

```javascript
const f = (x, ...y) => [x, ...y];
const g = f(?, 1, ...);
g(2, 3, 4); // [2, 1, 3, 4];
```

（4）`...`只會被採集一次，如果函數的部分執行使用了多個`...`，那麼每個`...`的值都將相同。

```javascript
const f = (...x) => x;
const g = f(..., 9, ...);
g(1, 2, 3); // [1, 2, 3, 9, 1, 2, 3]
```

上面代碼中，`g`定義了兩個`...`佔位符，真正執行的時候，它們的值是一樣的。

## 管道運算符

Unix 操作系統有一個管道機制（pipeline），可以把前一個操作的值傳給後一個操作。這個機制非常有用，使得簡單的操作可以組合成為複雜的操作。許多語言都有管道的實現，現在有一個[提案](https://github.com/tc39/proposal-pipeline-operator)，讓 JavaScript 也擁有管道機制。

JavaScript 的管道是一個運算符，寫作`|>`。它的左邊是一個表達式，右邊是一個函數。管道運算符把左邊表達式的值，傳入右邊的函數進行求值。

```javascript
x |> f
// 等同於
f(x)
```

管道運算符最大的好處，就是可以把嵌套的函數，寫成從左到右的鏈式表達式。

```javascript
function doubleSay (str) {
  return str + ", " + str;
}

function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}

function exclaim (str) {
  return str + '!';
}
```

上面是三個簡單的函數。如果要嵌套執行，傳統的寫法和管道的寫法分別如下。

```javascript
// 傳統的寫法
exclaim(capitalize(doubleSay('hello')))
// "Hello, hello!"

// 管道的寫法
'hello'
  |> doubleSay
  |> capitalize
  |> exclaim
// "Hello, hello!"
```

管道運算符只能傳遞一個值，這意味著它右邊的函數必須是一個單參數函數。如果是多參數函數，就必須進行柯裡化，改成單參數的版本。

```javascript
function double (x) { return x + x; }
function add (x, y) { return x + y; }

let person = { score: 25 };
person.score
  |> double
  |> (_ => add(7, _))
// 57
```

上面代碼中，`add`函數需要兩個參數。但是，管道運算符只能傳入一個值，因此需要事先提供另一個參數，並將其改成單參數的箭頭函數`_ => add(7, _)`。這個函數裡面的下劃線並沒有特別的含義，可以用其他符號代替，使用下劃線只是因為，它能夠形象地表示這裡是佔位符。

管道運算符對於`await`函數也適用。

```javascript
x |> await f
// 等同於
await f(x)

const userAge = userId |> await fetchUserById |> getAgeFromUser;
// 等同於
const userAge = getAgeFromUser(await fetchUserById(userId));
```

## 數值分隔符

歐美語言中，較長的數值允許每三位添加一個分隔符（通常是一個逗號），增加數值的可讀性。比如，`1000`可以寫作`1,000`。

現在有一個[提案](https://github.com/tc39/proposal-numeric-separator)，允許 JavaScript 的數值使用下劃線（`_`）作為分隔符。

```javascript
let budget = 1_000_000_000_000;
budget === 10 ** 12 // true
```

JavaScript 的數值分隔符沒有指定間隔的位數，也就是說，可以每三位添加一個分隔符，也可以每一位、每兩位、每四位添加一個。

```javascript
123_00 === 12_300 // true

12345_00 === 123_4500 // true
12345_00 === 1_234_500 // true
```

小數和科學計數法也可以使用數值分隔符。

```javascript
// 小數
0.000_001
// 科學計數法
1e10_000
```

數值分隔符有幾個使用注意點。

- 不能在數值的最前面（leading）或最後面（trailing）。
- 不能兩個或兩個以上的分隔符連在一起。
- 小數點的前後不能有分隔符。
- 科學計數法裡面，表示指數的`e`或`E`前後不能有分隔符。

下面的寫法都會報錯。

```javascript
// 全部報錯
3_.141
3._141
1_e12
1e_12
123__456
_1464301
1464301_
```

除了十進制，其他進制的數值也可以使用分隔符。

```javascript
// 二進制
0b1010_0001_1000_0101
// 十六進制
0xA0_B0_C0
```

注意，分隔符不能緊跟著進制的前綴`0b`、`0B`、`0o`、`0O`、`0x`、`0X`。

```javascript
// 報錯
0_b111111000
0b_111111000
```

下面三個將字符串轉成數值的函數，不支持數值分隔符。主要原因是提案的設計者認為，數值分隔符主要是為了編碼時書寫數值的方便，而不是為了處理外部輸入的數據。

- Number()
- parseInt()
- parseFloat()

```javascript
Number('123_456') // NaN
parseInt('123_456') // 123
```

## BigInt 數據類型

### 簡介

JavaScript 所有數字都保存成 64 位浮點數，這給數值的表示帶來了兩大限制。一是數值的精度只能到 53 個二進制位（相當於 16 個十進制位），大於這個範圍的整數，JavaScript 是無法精確表示的，這使得 JavaScript 不適合進行科學和金融方面的精確計算。二是大於或等於2的1024次方的數值，JavaScript 無法表示，會返回`Infinite`。

```javascript
// 超過 53 個二進制位的數值，無法保持精度
Math.pow(2, 53) === Math.pow(2, 53) + 1 // true

// 超過 2 的 1024 次方的數值，無法表示
Math.pow(2, 1024) // Infinity
```

現在有一個[提案](https://github.com/tc39/proposal-bigint)，引入了一種新的數據類型 BigInt（大整數），來解決這個問題。BigInt 只用來表示整數，沒有位數的限制，任何位數的整數都可以精確表示。

```javascript
const a = 2172141653n;
const b = 15346349309n;
a * b // 33334444555566667777n
Number(a) * Number(b) // 33334444555566670000
```

為了與 Number 類型區別，BigInt 類型的數據必須使用後綴`n`表示。

```javascript
1234n
1n + 2n // 3n
```

BigInt 同樣可以使用各種進製表示，都要加上後綴`n`。

```javascript
0b1101n // 二進制
0o777n // 八進制
0xFFn // 十六進制
```

`typeof`運算符對於 BigInt 類型的數據返回`bigint`。

```javascript
typeof 123n // 'bigint'
```

### BigInt 物件

JavaScript 原生提供`BigInt`物件，可以用作構造函數生成 BitInt 類型的數值。轉換規則基本與`Number()`一致，將別的類型的值轉為 BigInt。

```javascript
BigInt(123) // 123n
BigInt('123') // 123n
BitInt(false) // 0n
BitInt(true) // 1n
```

`BitInt`構造函數必須有參數，而且參數必須可以正常轉為數值，下面的用法都會報錯。

```javascript
new BitInt() // TypeError
BigInt(undefined) //TypeError
BigInt(null) // TypeError
BigInt('123n') // SyntaxError
BigInt('abc') // SyntaxError
```

上面代碼中，尤其值得注意字符串`123n`無法解析成 Number 類型，所以會報錯。

BigInt 物件繼承了 Object 提供的實例方法。

- `BigInt.prototype.toLocaleString()`
- `BigInt.prototype.toString()`
- `BigInt.prototype.valueOf()`

此外，還提供了三個靜態方法。

- `BigInt.asUintN(width, BigInt)`： 對給定的大整數，返回 0 到 2<sup>width</sup> - 1 之間的大整數形式。
- `BigInt.asIntN(width, BigInt)`：對給定的大整數，返回 -2<sup>width - 1</sup> 到 2<sup>width - 1</sup> - 1 之間的大整數形式。
- `BigInt.parseInt(string[, radix])`：近似於`Number.parseInt`，將一個字符串轉換成指定進制的大整數。

```javascript
// 將一個大整數轉為 64 位整數的形式
const int64a = BigInt.asUintN(64, 12345n);

// Number.parseInt 與 BigInt.parseInt 的對比
Number.parseInt('9007199254740993', 10)
// 9007199254740992
BigInt.parseInt('9007199254740993', 10)
// 9007199254740993n
```

上面代碼中，由於有效數字超出了最大限度，`Number.parseInt`方法返回的結果是不精確的，而`BigInt.parseInt`方法正確返回了對應的大整數。

對於二進制陣列，BigInt 新增了兩個類型`BigUint64Array`和`BigInt64Array`，這兩種數據類型返回的都是大整數。`DataView`物件的實例方法`DataView.prototype.getBigInt64`和`DataView.prototype.getBigUint64`，返回的也是大整數。

### 運算

數學運算方面，BigInt 類型的`+`、`-`、`*`和`**`這四個二元運算符，與 Number 類型的行為一致。除法運算`/`會捨去小數部分，返回一個整數。

```javascript
9n / 5n
// 1n
```

幾乎所有的 Number 運算符都可以用在 BigInt，但是有兩個除外：不帶符號的右移位運算符`>>>`和一元的求正運算符`+`，使用時會報錯。前者是因為`>>>`運算符是不帶符號的，但是 BigInt 總是帶有符號的，導致該運算無意義，完全等同於右移運算符`>>`。後者是因為一元運算符`+`在 asm.js 裡面總是返回 Number 類型，為了不破壞 asm.js 就規定`+1n`會報錯。

Integer 類型不能與 Number 類型進行混合運算。

```javascript
1n + 1.3 // 報錯
```

上面代碼報錯是因為無論返回的是 BigInt 或 Number，都會導致丟失信息。比如`(2n**53n + 1n) + 0.5`這個表達式，如果返回 BigInt 類型，`0.5`這個小數部分會丟失；如果返回 Number 類型，有效精度只能保持 53 位，導致精度下降。

同樣的原因，如果一個標準庫函數的參數預期是 Number 類型，但是得到的是一個 BigInt，就會報錯。

```javascript
// 錯誤的寫法
Math.sqrt(4n) // 報錯

// 正確的寫法
Math.sqrt(Number(4n)) // 2
```

上面代碼中，`Math.sqrt`的參數預期是 Number 類型，如果是 BigInt 就會報錯，必須先用`Number`方法轉一下類型，才能進行計算。

asm.js 裡面，`|0`跟在一個數值的後面會返回一個32位整數。根據不能與 Number 類型混合運算的規則，BigInt 如果與`|0`進行運算會報錯。

```javascript
1n | 0 // 報錯
```

比較運算符（比如`>`）和相等運算符（`==`）允許 BigInt 與其他類型的值混合計算，因為這樣做不會損失精度。

```javascript
0n < 1 // true
0n < true // true
0n == 0 // true
0n == false // true
```

同理，精確相等運算符（`===`）也可以混合使用。

```javascript
0n === 0 // false
```

上面代碼中，由於`0n`與`0`的數據類型不同，所以返回`false`。

大整數可以轉為其他數據類型。

```javascript
Boolean(0n) // false
Boolean(1n) // true
Number(1n)  // 1
String(1n)  // "1"

!0n // true
!1n // false
```

大整數也可以與字符串混合運算。

```javascript
'' + 123n // "123"
```

## Math.signbit()

`Math.sign()`用來判斷一個值的正負，但是如果參數是`-0`，它會返回`-0`。

```javascript
Math.sign(-0) // -0
```

這導致對於判斷符號位的正負，`Math.sign()`不是很有用。JavaScript 內部使用 64 位浮點數（國際標準 IEEE 754）表示數值，IEEE 754 規定第一位是符號位，`0`表示正數，`1`表示負數。所以會有兩種零，`+0`是符號位為`0`時的零值，`-0`是符號位為`1`時的零值。實際編程中，判斷一個值是`+0`還是`-0`非常麻煩，因為它們是相等的。

```javascript
+0 === -0 // true
```

目前，有一個[提案](http://jfbastien.github.io/papers/Math.signbit.html)，引入了`Math.signbit()`方法判斷一個數的符號位是否設置了。

```javascript
Math.signbit(2) //false
Math.signbit(-2) //true
Math.signbit(0) //false
Math.signbit(-0) //true
```

可以看到，該方法正確返回了`-0`的符號位是設置了的。

該方法的算法如下。

- 如果參數是`NaN`，返回`false`
- 如果參數是`-0`，返回`true`
- 如果參數是負值，返回`true`
- 其他情況返回`false`

# 讀懂 ECMAScript 規格

## 概述

規格文件是計算機語言的官方標準，詳細描述語法規則和實現方法。

一般來說，沒有必要閱讀規格，除非你要寫編譯器。因為規格寫得非常抽象和精煉，又缺乏實例，不容易理解，而且對於解決實際的應用問題，幫助不大。但是，如果你遇到疑難的語法問題，實在找不到答案，這時可以去查看規格文件，瞭解語言標準是怎麼說的。規格是解決問題的“最後一招”。

這對 JavaScript 語言很有必要。因為它的使用場景複雜，語法規則不統一，例外很多，各種運行環境的行為不一致，導致奇怪的語法問題層出不窮，任何語法書都不可能囊括所有情況。查看規格，不失為一種解決語法問題的最可靠、最權威的終極方法。

本章介紹如何讀懂 ECMAScript 6 的規格文件。

ECMAScript 6 的規格，可以在 ECMA 國際標準組織的官方網站（[www.ecma-international.org/ecma-262/6.0/](http://www.ecma-international.org/ecma-262/6.0/)）免費下載和在線閱讀。

這個規格文件相當龐大，一共有 26 章，A4 打印的話，足足有 545 頁。它的特點就是規定得非常細緻，每一個語法行為、每一個函數的實現都做了詳盡的清晰的描述。基本上，編譯器作者只要把每一步翻譯成代碼就可以了。這很大程度上，保證了所有 ES6 實現都有一致的行為。

ECMAScript 6 規格的 26 章之中，第 1 章到第 3 章是對文件本身的介紹，與語言關係不大。第 4 章是對這門語言總體設計的描述，有興趣的讀者可以讀一下。第 5 章到第 8 章是語言宏觀層面的描述。第 5 章是規格的名詞解釋和寫法的介紹，第 6 章介紹數據類型，第 7 章介紹語言內部用到的抽象操作，第 8 章介紹代碼如何運行。第 9 章到第 26 章介紹具體的語法。

對於一般用戶來說，除了第 4 章，其他章節都涉及某一方面的細節，不用通讀，只要在用到的時候，查閱相關章節即可。

## 術語

ES6 規格使用了一些專門的術語，瞭解這些術語，可以幫助你讀懂規格。本節介紹其中的幾個。

### 抽象操作

所謂”抽象操作“（abstract operations）就是引擎的一些內部方法，外部不能調用。規格定義了一系列的抽象操作，規定了它們的行為，留給各種引擎自己去實現。

舉例來說，`Boolean(value)`的算法，第一步是這樣的。

> 1. Let `b` be `ToBoolean(value)`.

這裡的`ToBoolean`就是一個抽象操作，是引擎內部求出布爾值的算法。

許多函數的算法都會多次用到同樣的步驟，所以 ES6 規格將它們抽出來，定義成”抽象操作“，方便描述。

### Record 和 field

ES6 規格將鍵值對（key-value map）的數據結構稱為 Record，其中的每一組鍵值對稱為 field。這就是說，一個 Record 由多個 field 組成，而每個 field 都包含一個鍵名（key）和一個鍵值（value）。

### [[Notation]]

ES6 規格大量使用`[[Notation]]`這種書寫法，比如`[[Value]]`、`[[Writable]]`、`[[Get]]`、`[[Set]]`等等。它用來指代 field 的鍵名。

舉例來說，`obj`是一個 Record，它有一個`Prototype`屬性。ES6 規格不會寫`obj.Prototype`，而是寫`obj.[[Prototype]]`。一般來說，使用`[[Notation]]`這種書寫法的屬性，都是物件的內部屬性。

所有的 JavaScript 函數都有一個內部屬性`[[Call]]`，用來運行該函數。

```javascript
F.[[Call]](V, argumentsList)
```

上面代碼中，`F`是一個函數物件，`[[Call]]`是它的內部方法，`F.[[call]]()`表示運行該函數，`V`表示`[[Call]]`運行時`this`的值，`argumentsList`則是調用時傳入函數的參數。

### Completion Record

每一個語句都會返回一個 Completion Record，表示運行結果。每個 Completion Record 有一個`[[Type]]`屬性，表示運行結果的類型。

`[[Type]]`屬性有五種可能的值。

- normal
- return
- throw
- break
- continue

如果`[[Type]]`的值是`normal`，就稱為 normal completion，表示運行正常。其他的值，都稱為 abrupt completion。其中，開發者只需要關注`[[Type]]`為`throw`的情況，即運行出錯；`break`、`continue`、`return`這三個值都只出現在特定場景，可以不用考慮。

## 抽象操作的標準流程

抽象操作的運行流程，一般是下面這樣。

> 1. Let `resultCompletionRecord` be `AbstractOp()`.
> 1. If `resultCompletionRecord` is an abrupt completion, return `resultCompletionRecord`.
> 1. Let `result` be `resultCompletionRecord.[[Value]]`.
> 1. return `result`.

上面的第一步是調用抽象操作`AbstractOp()`，得到`resultCompletionRecord`，這是一個 Completion Record。第二步，如果這個 Record 屬於 abrupt completion，就將`resultCompletionRecord`返回給用戶。如果此處沒有返回，就表示運行結果正常，所得的值存放在`resultCompletionRecord.[[Value]]`屬性。第三步，將這個值記為`result`。第四步，將`result`返回給用戶。

ES6 規格將這個標準流程，使用簡寫的方式表達。

> 1. Let `result` be `AbstractOp()`.
> 1. `ReturnIfAbrupt(result)`.
> 1. return `result`.

這個簡寫方式裡面的`ReturnIfAbrupt(result)`，就代表了上面的第二步和第三步，即如果有報錯，就返回錯誤，否則取出值。

甚至還有進一步的簡寫格式。

> 1. Let `result` be `? AbstractOp()`.
> 1. return `result`.

上面流程的`?`，就代表`AbstractOp()`可能會報錯。一旦報錯，就返回錯誤，否則取出值。

除了`?`，ES 6 規格還使用另一個簡寫符號`!`。

> 1. Let `result` be `! AbstractOp()`.
> 1. return `result`.

上面流程的`!`，代表`AbstractOp()`不會報錯，返回的一定是 normal completion，總是可以取出值。

## 相等運算符

下面通過一些例子，介紹如何使用這份規格。

相等運算符（`==`）是一個很讓人頭痛的運算符，它的語法行為多變，不符合直覺。這個小節就看看規格怎麼規定它的行為。

請看下面這個表達式，請問它的值是多少。

```javascript
0 == null
```

如果你不確定答案，或者想知道語言內部怎麼處理，就可以去查看規格，[7.2.12 小節](http://www.ecma-international.org/ecma-262/6.0/#sec-abstract-equality-comparison)是對相等運算符（`==`）的描述。

規格對每一種語法行為的描述，都分成兩部分：先是總體的行為描述，然後是實現的算法細節。相等運算符的總體描述，只有一句話。

> “The comparison `x == y`, where `x` and `y` are values, produces `true` or `false`.”

上面這句話的意思是，相等運算符用於比較兩個值，返回`true`或`false`。

下面是算法細節。

> 1. ReturnIfAbrupt(x).
> 1. ReturnIfAbrupt(y).
> 1. If `Type(x)` is the same as `Type(y)`, then
>    1. Return the result of performing Strict Equality Comparison `x === y`.
> 1. If `x` is `null` and `y` is `undefined`, return `true`.
> 1. If `x` is `undefined` and `y` is `null`, return `true`.
> 1. If `Type(x)` is Number and `Type(y)` is String, 
>    return the result of the comparison `x == ToNumber(y)`.
> 1. If `Type(x)` is String and `Type(y)` is Number, 
>    return the result of the comparison `ToNumber(x) == y`.
> 1. If `Type(x)` is Boolean, return the result of the comparison `ToNumber(x) == y`.
> 1. If `Type(y)` is Boolean, return the result of the comparison `x == ToNumber(y)`.
> 1. If `Type(x)` is either String, Number, or Symbol and `Type(y)` is Object, then 
>    return the result of the comparison `x == ToPrimitive(y)`.
> 1. If `Type(x)` is Object and `Type(y)` is either String, Number, or Symbol, then 
>    return the result of the comparison `ToPrimitive(x) == y`.
> 1. Return `false`.

上面這段算法，一共有 12 步，翻譯如下。

> 1. 如果`x`不是正常值（比如拋出一個錯誤），中斷執行。
> 1. 如果`y`不是正常值，中斷執行。
> 1. 如果`Type(x)`與`Type(y)`相同，執行嚴格相等運算`x === y`。
> 1. 如果`x`是`null`，`y`是`undefined`，返回`true`。
> 1. 如果`x`是`undefined`，`y`是`null`，返回`true`。
> 1. 如果`Type(x)`是數值，`Type(y)`是字符串，返回`x == ToNumber(y)`的結果。
> 1. 如果`Type(x)`是字符串，`Type(y)`是數值，返回`ToNumber(x) == y`的結果。
> 1. 如果`Type(x)`是布爾值，返回`ToNumber(x) == y`的結果。
> 1. 如果`Type(y)`是布爾值，返回`x == ToNumber(y)`的結果。
> 1. 如果`Type(x)`是字符串或數值或`Symbol`值，`Type(y)`是物件，返回`x == ToPrimitive(y)`的結果。
> 1. 如果`Type(x)`是物件，`Type(y)`是字符串或數值或`Symbol`值，返回`ToPrimitive(x) == y`的結果。
> 1. 返回`false`。

由於`0`的類型是數值，`null`的類型是 Null（這是規格[4.3.13 小節](http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-null-type)的規定，是內部 Type 運算的結果，跟`typeof`運算符無關）。因此上面的前 11 步都得不到結果，要到第 12 步才能得到`false`。

```javascript
0 == null // false
```

## 陣列的空位

下面再看另一個例子。

```javascript
const a1 = [undefined, undefined, undefined];
const a2 = [, , ,];

a1.length // 3
a2.length // 3

a1[0] // undefined
a2[0] // undefined

a1[0] === a2[0] // true
```

上面代碼中，陣列`a1`的成員是三個`undefined`，陣列`a2`的成員是三個空位。這兩個陣列很相似，長度都是 3，每個位置的成員讀取出來都是`undefined`。

但是，它們實際上存在重大差異。

```javascript
0 in a1 // true
0 in a2 // false

a1.hasOwnProperty(0) // true
a2.hasOwnProperty(0) // false

Object.keys(a1) // ["0", "1", "2"]
Object.keys(a2) // []

a1.map(n => 1) // [1, 1, 1]
a2.map(n => 1) // [, , ,]
```

上面代碼一共列出了四種運算，陣列`a1`和`a2`的結果都不一樣。前三種運算（`in`運算符、陣列的`hasOwnProperty`方法、`Object.keys`方法）都說明，陣列`a2`取不到屬性名。最後一種運算（陣列的`map`方法）說明，陣列`a2`沒有發生遍歷。

為什麼`a1`與`a2`成員的行為不一致？陣列的成員是`undefined`或空位，到底有什麼不同？

規格的[12.2.5 小節《陣列的初始化》](http://www.ecma-international.org/ecma-262/6.0/#sec-array-initializer)給出了答案。

> “Array elements may be elided at the beginning, middle or end of the element list. Whenever a comma in the element list is not preceded by an AssignmentExpression (i.e., a comma at the beginning or after another comma), the missing array element contributes to the length of the Array and increases the index of subsequent elements. Elided array elements are not defined. If an element is elided at the end of an array, that element does not contribute to the length of the Array.”

翻譯如下。

> "陣列成員可以省略。只要逗號前面沒有任何表達式，陣列的`length`屬性就會加 1，並且相應增加其後成員的位置索引。被省略的成員不會被定義。如果被省略的成員是陣列最後一個成員，則不會導致陣列`length`屬性增加。”

上面的規格說得很清楚，陣列的空位會反映在`length`屬性，也就是說空位有自己的位置，但是這個位置的值是未定義，即這個值是不存在的。如果一定要讀取，結果就是`undefined`（因為`undefined`在 JavaScript 語言中表示不存在）。

這就解釋了為什麼`in`運算符、陣列的`hasOwnProperty`方法、`Object.keys`方法，都取不到空位的屬性名。因為這個屬性名根本就不存在，規格里面沒說要為空位分配屬性名(位置索引），只說要為下一個元素的位置索引加 1。

至於為什麼陣列的`map`方法會跳過空位，請看下一節。

## 陣列的 map 方法

規格的[22.1.3.15 小節](http://www.ecma-international.org/ecma-262/6.0/#sec-array.prototype.map)定義了陣列的`map`方法。該小節先是總體描述`map`方法的行為，裡面沒有提到陣列空位。

後面的算法描述是這樣的。

> 1. Let `O` be `ToObject(this value)`.
> 1. `ReturnIfAbrupt(O)`.
> 1. Let `len` be `ToLength(Get(O, "length"))`.
> 1. `ReturnIfAbrupt(len)`.
> 1. If `IsCallable(callbackfn)` is `false`, throw a TypeError exception.
> 1. If `thisArg` was supplied, let `T` be `thisArg`; else let `T` be `undefined`.
> 1. Let `A` be `ArraySpeciesCreate(O, len)`.
> 1. `ReturnIfAbrupt(A)`.
> 1. Let `k` be 0.
> 1. Repeat, while `k` < `len`
>    1. Let `Pk` be `ToString(k)`.
>    1. Let `kPresent` be `HasProperty(O, Pk)`.
>    1. `ReturnIfAbrupt(kPresent)`.
>    1. If `kPresent` is `true`, then
>       1. Let `kValue` be `Get(O, Pk)`.
>       1. `ReturnIfAbrupt(kValue)`.
>       1. Let `mappedValue` be `Call(callbackfn, T, «kValue, k, O»)`.
>       1. `ReturnIfAbrupt(mappedValue)`.
>       1. Let `status` be `CreateDataPropertyOrThrow (A, Pk, mappedValue)`.
>       1. `ReturnIfAbrupt(status)`.
>    1. Increase `k` by 1.
> 1. Return `A`.

翻譯如下。

> 1. 得到當前陣列的`this`物件
> 1. 如果報錯就返回
> 1. 求出當前陣列的`length`屬性
> 1. 如果報錯就返回
> 1. 如果 map 方法的參數`callbackfn`不可執行，就報錯
> 1. 如果 map 方法的參數之中，指定了`this`，就讓`T`等於該參數，否則`T`為`undefined`
> 1. 生成一個新的陣列`A`，跟當前陣列的`length`屬性保持一致
> 1. 如果報錯就返回
> 1. 設定`k`等於 0
> 1. 只要`k`小於當前陣列的`length`屬性，就重複下面步驟
>    1. 設定`Pk`等於`ToString(k)`，即將`K`轉為字符串
>    1. 設定`kPresent`等於`HasProperty(O, Pk)`，即求當前陣列有沒有指定屬性
>    1. 如果報錯就返回
>    1. 如果`kPresent`等於`true`，則進行下面步驟
>       1. 設定`kValue`等於`Get(O, Pk)`，取出當前陣列的指定屬性
>       1. 如果報錯就返回
>       1. 設定`mappedValue`等於`Call(callbackfn, T, «kValue, k, O»)`，即執行回調函數
>       1. 如果報錯就返回
>       1. 設定`status`等於`CreateDataPropertyOrThrow (A, Pk, mappedValue)`，即將回調函數的值放入`A`陣列的指定位置
>       1. 如果報錯就返回
>    1. `k`增加 1
> 1. 返回`A`

仔細查看上面的算法，可以發現，當處理一個全是空位的陣列時，前面步驟都沒有問題。進入第 10 步中第 2 步時，`kPresent`會報錯，因為空位對應的屬性名，對於陣列來說是不存在的，因此就會返回，不會進行後面的步驟。

```javascript
const arr = [, , ,];
arr.map(n => {
  console.log(n);
  return 1;
}) // [, , ,]
```

上面代碼中，`arr`是一個全是空位的陣列，`map`方法遍歷成員時，發現是空位，就直接跳過，不會進入回調函數。因此，回調函數裡面的`console.log`語句根本不會執行，整個`map`方法返回一個全是空位的新陣列。

V8 引擎對`map`方法的[實現](https://github.com/v8/v8/blob/44c44521ae11859478b42004f57ea93df52526ee/src/js/array.js#L1347)如下，可以看到跟規格的算法描述完全一致。

```javascript
function ArrayMap(f, receiver) {
  CHECK_OBJECT_COERCIBLE(this, "Array.prototype.map");

  // Pull out the length so that modifications to the length in the
  // loop will not affect the looping and side effects are visible.
  var array = TO_OBJECT(this);
  var length = TO_LENGTH_OR_UINT32(array.length);
  return InnerArrayMap(f, receiver, array, length);
}

function InnerArrayMap(f, receiver, array, length) {
  if (!IS_CALLABLE(f)) throw MakeTypeError(kCalledNonCallable, f);

  var accumulator = new InternalArray(length);
  var is_array = IS_ARRAY(array);
  var stepping = DEBUG_IS_STEPPING(f);
  for (var i = 0; i < length; i++) {
    if (HAS_INDEX(array, i, is_array)) {
      var element = array[i];
      // Prepare break slots for debugger step in.
      if (stepping) %DebugPrepareStepInIfStepping(f);
      accumulator[i] = %_Call(f, receiver, element, i, array);
    }
  }
  var result = new GlobalArray();
  %MoveArrayContents(accumulator, result);
  return result;
}
```
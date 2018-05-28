# 字符串的擴展

ES6 加強了對 Unicode 的支持，並且擴展了字符串物件。

## 字符的 Unicode 表示法

JavaScript 允許採用`\uxxxx`形式表示一個字符，其中`xxxx`表示字符的 Unicode 碼點。

```javascript
"\u0061"
// "a"
```

但是，這種表示法只限於碼點在`\u0000`~`\uFFFF`之間的字符。超出這個範圍的字符，必須用兩個雙字節的形式表示。

```javascript
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// " 7"
```

上面代碼表示，如果直接在`\u`後面跟上超過`0xFFFF`的數值（比如`\u20BB7`），JavaScript 會理解成`\u20BB+7`。由於`\u20BB`是一個不可打印字符，所以只會顯示一個空格，後面跟著一個`7`。

ES6 對這一點做出了改進，只要將碼點放入大括號，就能正確解讀該字符。

```javascript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

上面代碼中，最後一個例子表明，大括號表示法與四字節的 UTF-16 編碼是等價的。

有了這種表示法之後，JavaScript 共有 6 種方法可以表示一個字符。

```javascript
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

## codePointAt()

JavaScript 內部，字符以 UTF-16 的格式儲存，每個字符固定為`2`個字節。對於那些需要`4`個字節儲存的字符（Unicode 碼點大於`0xFFFF`的字符），JavaScript 會認為它們是兩個字符。

```javascript
var s = "𠮷";

s.length // 2
s.charAt(0) // ''
s.charAt(1) // ''
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271
```

上面代碼中，漢字“𠮷”（注意，這個字不是“吉祥”的“吉”）的碼點是`0x20BB7`，UTF-16 編碼為`0xD842 0xDFB7`（十進製為`55362 57271`），需要`4`個字節儲存。對於這種`4`個字節的字符，JavaScript 不能正確處理，字符串長度會誤判為`2`，而且`charAt`方法無法讀取整個字符，`charCodeAt`方法只能分別返回前兩個字節和後兩個字節的值。

ES6 提供了`codePointAt`方法，能夠正確處理 4 個字節儲存的字符，返回一個字符的碼點。

```javascript
let s = '𠮷a';

s.codePointAt(0) // 134071
s.codePointAt(1) // 57271

s.codePointAt(2) // 97
```

`codePointAt`方法的參數，是字符在字符串中的位置（從 0 開始）。上面代碼中，JavaScript 將“𠮷a”視為三個字符，codePointAt 方法在第一個字符上，正確地識別了“𠮷”，返回了它的十進制碼點 134071（即十六進制的`20BB7`）。在第二個字符（即“𠮷”的後兩個字節）和第三個字符“a”上，`codePointAt`方法的結果與`charCodeAt`方法相同。

總之，`codePointAt`方法會正確返回 32 位的 UTF-16 字符的碼點。對於那些兩個字節儲存的常規字符，它的返回結果與`charCodeAt`方法相同。

`codePointAt`方法返回的是碼點的十進制值，如果想要十六進制的值，可以使用`toString`方法轉換一下。

```javascript
let s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```

你可能注意到了，`codePointAt`方法的參數，仍然是不正確的。比如，上面代碼中，字符`a`在字符串`s`的正確位置序號應該是 1，但是必須向`codePointAt`方法傳入 2。解決這個問題的一個辦法是使用`for...of`循環，因為它會正確識別 32 位的 UTF-16 字符。

```javascript
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

`codePointAt`方法是測試一個字符由兩個字節還是由四個字節組成的最簡單方法。

```javascript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

## String.fromCodePoint()

ES5 提供`String.fromCharCode`方法，用於從碼點返回對應字符，但是這個方法不能識別 32 位的 UTF-16 字符（Unicode 編號大於`0xFFFF`）。

```javascript
String.fromCharCode(0x20BB7)
// "ஷ"
```

上面代碼中，`String.fromCharCode`不能識別大於`0xFFFF`的碼點，所以`0x20BB7`就發生了溢出，最高位`2`被捨棄了，最後返回碼點`U+0BB7`對應的字符，而不是碼點`U+20BB7`對應的字符。

ES6 提供了`String.fromCodePoint`方法，可以識別大於`0xFFFF`的字符，彌補了`String.fromCharCode`方法的不足。在作用上，正好與`codePointAt`方法相反。

```javascript
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// true
```

上面代碼中，如果`String.fromCodePoint`方法有多個參數，則它們會被合併成一個字符串返回。

注意，`fromCodePoint`方法定義在`String`物件上，而`codePointAt`方法定義在字符串的實例物件上。

## 字符串的遍歷器接口

ES6 為字符串添加了遍歷器接口（詳見《Iterator》一章），使得字符串可以被`for...of`循環遍歷。

```javascript
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

除了遍歷字符串，這個遍歷器最大的優點是可以識別大於`0xFFFF`的碼點，傳統的`for`循環無法識別這樣的碼點。

```javascript
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```

上面代碼中，字符串`text`只有一個字符，但是`for`循環會認為它包含兩個字符（都不可打印），而`for...of`循環會正確識別出這一個字符。

## at()

ES5 對字符串物件提供`charAt`方法，返回字符串給定位置的字符。該方法不能識別碼點大於`0xFFFF`的字符。

```javascript
'abc'.charAt(0) // "a"
'𠮷'.charAt(0) // "\uD842"
```

上面代碼中的第二條語句，`charAt`方法期望返回的是用2個字節表示的字符，但漢字“𠮷”佔用了4個字節，`charAt(0)`表示獲取這4個字節中的前2個字節，很顯然，這是無法正常顯示的。

目前，有一個提案，提出字符串實例的`at`方法，可以識別 Unicode 編號大於`0xFFFF`的字符，返回正確的字符。

```javascript
'abc'.at(0) // "a"
'𠮷'.at(0) // "𠮷"
```

這個方法可以通過[墊片庫](https://github.com/es-shims/String.prototype.at)實現。

## normalize()

許多歐洲語言有語調符號和重音符號。為了表示它們，Unicode 提供了兩種方法。一種是直接提供帶重音符號的字符，比如`Ǒ`（\u01D1）。另一種是提供合成符號（combining character），即原字符與重音符號的合成，兩個字符合成一個字符，比如`O`（\u004F）和`ˇ`（\u030C）合成`Ǒ`（\u004F\u030C）。

這兩種表示方法，在視覺和語義上都等價，但是 JavaScript 不能識別。

```javascript
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```

上面代碼表示，JavaScript 將合成字符視為兩個字符，導致兩種表示方法不相等。

ES6 提供字符串實例的`normalize()`方法，用來將字符的不同表示方法統一為同樣的形式，這稱為 Unicode 正規化。

```javascript
'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```

`normalize`方法可以接受一個參數來指定`normalize`的方式，參數的四個可選值如下。

- `NFC`，默認參數，表示“標準等價合成”（Normalization Form Canonical Composition），返回多個簡單字符的合成字符。所謂“標準等價”指的是視覺和語義上的等價。
- `NFD`，表示“標準等價分解”（Normalization Form Canonical Decomposition），即在標準等價的前提下，返回合成字符分解的多個簡單字符。
- `NFKC`，表示“兼容等價合成”（Normalization Form Compatibility Composition），返回合成字符。所謂“兼容等價”指的是語義上存在等價，但視覺上不等價，比如“囍”和“喜喜”。（這只是用來舉例，`normalize`方法不能識別中文。）
- `NFKD`，表示“兼容等價分解”（Normalization Form Compatibility Decomposition），即在兼容等價的前提下，返回合成字符分解的多個簡單字符。

```javascript
'\u004F\u030C'.normalize('NFC').length // 1
'\u004F\u030C'.normalize('NFD').length // 2
```

上面代碼表示，`NFC`參數返回字符的合成形式，`NFD`參數返回字符的分解形式。

不過，`normalize`方法目前不能識別三個或三個以上字符的合成。這種情況下，還是只能使用正則表達式，通過 Unicode 編號區間判斷。

## includes(), startsWith(), endsWith()

傳統上，JavaScript 只有`indexOf`方法，可以用來確定一個字符串是否包含在另一個字符串中。ES6 又提供了三種新方法。

- **includes()**：返回布爾值，表示是否找到了參數字符串。
- **startsWith()**：返回布爾值，表示參數字符串是否在原字符串的頭部。
- **endsWith()**：返回布爾值，表示參數字符串是否在原字符串的尾部。

```javascript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```

這三個方法都支持第二個參數，表示開始搜索的位置。

```javascript
let s = 'Hello world!';

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```

上面代碼表示，使用第二個參數`n`時，`endsWith`的行為與其他兩個方法有所不同。它針對前`n`個字符，而其他兩個方法針對從第`n`個位置直到字符串結束。

## repeat()

`repeat`方法返回一個新字符串，表示將原字符串重複`n`次。

```javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```

參數如果是小數，會被取整。

```javascript
'na'.repeat(2.9) // "nana"
```

如果`repeat`的參數是負數或者`Infinity`，會報錯。

```javascript
'na'.repeat(Infinity)
// RangeError
'na'.repeat(-1)
// RangeError
```

但是，如果參數是 0 到-1 之間的小數，則等同於 0，這是因為會先進行取整運算。0 到-1 之間的小數，取整以後等於`-0`，`repeat`視同為 0。

```javascript
'na'.repeat(-0.9) // ""
```

參數`NaN`等同於 0。

```javascript
'na'.repeat(NaN) // ""
```

如果`repeat`的參數是字符串，則會先轉換成數字。

```javascript
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```

## padStart()，padEnd()

ES2017 引入了字符串補全長度的功能。如果某個字符串不夠指定長度，會在頭部或尾部補全。`padStart()`用於頭部補全，`padEnd()`用於尾部補全。

```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'
```

上面代碼中，`padStart`和`padEnd`一共接受兩個參數，第一個參數用來指定字符串的最小長度，第二個參數是用來補全的字符串。

如果原字符串的長度，等於或大於指定的最小長度，則返回原字符串。

```javascript
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
```

如果用來補全的字符串與原字符串，兩者的長度之和超過了指定的最小長度，則會截去超出位數的補全字符串。

```javascript
'abc'.padStart(10, '0123456789')
// '0123456abc'
```

如果省略第二個參數，默認使用空格補全長度。

```javascript
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```

`padStart`的常見用途是為數值補全指定位數。下面代碼生成 10 位的數值字符串。

```javascript
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```

另一個用途是提示字符串格式。

```javascript
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

## matchAll()

`matchAll`方法返回一個正則表達式在當前字符串的所有匹配，詳見《正則的擴展》的一章。

## 模板字符串

傳統的 JavaScript 語言，輸出模板通常是這樣寫的。

```javascript
$('#result').append(
  'There are <b>' + basket.count + '</b> ' +
  'items in your basket, ' +
  '<em>' + basket.onSale +
  '</em> are on sale!'
);
```

上面這種寫法相當繁瑣不方便，ES6 引入了模板字符串解決這個問題。

```javascript
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```

模板字符串（template string）是增強版的字符串，用反引號（&#96;）標識。它可以當作普通字符串使用，也可以用來定義多行字符串，或者在字符串中嵌入變數。

```javascript
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入變數
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

上面代碼中的模板字符串，都是用反引號表示。如果在模板字符串中需要使用反引號，則前面要用反斜槓轉義。

```javascript
let greeting = `\`Yo\` World!`;
```

如果使用模板字符串表示多行字符串，所有的空格和縮進都會被保留在輸出之中。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

上面代碼中，所有模板字符串的空格和換行，都是被保留的，比如`<ul>`標籤前面會有一個換行。如果你不想要這個換行，可以使用`trim`方法消除它。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```

模板字符串中嵌入變數，需要將變數名寫在`${}`之中。

```javascript
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 傳統寫法為
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```

大括號內部可以放入任意的 JavaScript 表達式，可以進行運算，以及引用物件屬性。

```javascript
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字符串之中還能調用函數。

```javascript
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

如果大括號中的值不是字符串，將按照一般的規則轉為字符串。比如，大括號中是一個物件，將默認調用物件的`toString`方法。

如果模板字符串中的變數沒有聲明，將報錯。

```javascript
// 變數place沒有聲明
let msg = `Hello, ${place}`;
// 報錯
```

由於模板字符串的大括號內部，就是執行 JavaScript 代碼，因此如果大括號內部是一個字符串，將會原樣輸出。

```javascript
`Hello ${'World'}`
// "Hello World"
```

模板字符串甚至還能嵌套。

```javascript
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
```

上面代碼中，模板字符串的變數之中，又嵌入了另一個模板字符串，使用方法如下。

```javascript
const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```

如果需要引用模板字符串本身，在需要時執行，可以像下面這樣寫。

```javascript
// 寫法一
let str = 'return ' + '`Hello ${name}!`';
let func = new Function('name', str);
func('Jack') // "Hello Jack!"

// 寫法二
let str = '(name) => `Hello ${name}!`';
let func = eval.call(null, str);
func('Jack') // "Hello Jack!"
```

## 實例：模板編譯

下面，我們來看一個通過模板字符串，生成正式模板的實例。

```javascript
let template = `
<ul>
  <% for(let i=0; i < data.supplies.length; i++) { %>
    <li><%= data.supplies[i] %></li>
  <% } %>
</ul>
`;
```

上面代碼在模板字符串之中，放置了一個常規模板。該模板使用`<%...%>`放置 JavaScript 代碼，使用`<%= ... %>`輸出 JavaScript 表達式。

怎麼編譯這個模板字符串呢？

一種思路是將其轉換為 JavaScript 表達式字符串。

```javascript
echo('<ul>');
for(let i=0; i < data.supplies.length; i++) {
  echo('<li>');
  echo(data.supplies[i]);
  echo('</li>');
};
echo('</ul>');
```

這個轉換使用正則表達式就行了。

```javascript
let evalExpr = /<%=(.+?)%>/g;
let expr = /<%([\s\S]+?)%>/g;

template = template
  .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
  .replace(expr, '`); \n $1 \n  echo(`');

template = 'echo(`' + template + '`);';
```

然後，將`template`封裝在一個函數裡面返回，就可以了。

```javascript
let script =
`(function parse(data){
  let output = "";

  function echo(html){
    output += html;
  }

  ${ template }

  return output;
})`;

return script;
```

將上面的內容拼裝成一個模板編譯函數`compile`。

```javascript
function compile(template){
  const evalExpr = /<%=(.+?)%>/g;
  const expr = /<%([\s\S]+?)%>/g;

  template = template
    .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
    .replace(expr, '`); \n $1 \n  echo(`');

  template = 'echo(`' + template + '`);';

  let script =
  `(function parse(data){
    let output = "";

    function echo(html){
      output += html;
    }

    ${ template }

    return output;
  })`;

  return script;
}
```

`compile`函數的用法如下。

```javascript
let parse = eval(compile(template));
div.innerHTML = parse({ supplies: [ "broom", "mop", "cleaner" ] });
//   <ul>
//     <li>broom</li>
//     <li>mop</li>
//     <li>cleaner</li>
//   </ul>
```

## 標籤模板

模板字符串的功能，不僅僅是上面這些。它可以緊跟在一個函數名後面，該函數將被調用來處理這個模板字符串。這被稱為“標籤模板”功能（tagged template）。

```javascript
alert`123`
// 等同於
alert(123)
```

標籤模板其實不是模板，而是函數調用的一種特殊形式。“標籤”指的就是函數，緊跟在後面的模板字符串就是它的參數。

但是，如果模板字符裡面有變數，就不是簡單的調用了，而是會將模板字符串先處理成多個參數，再調用函數。

```javascript
let a = 5;
let b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同於
tag(['Hello ', ' world ', ''], 15, 50);
```

上面代碼中，模板字符串前面有一個標識名`tag`，它是一個函數。整個表達式的返回值，就是`tag`函數處理模板字符串後的返回值。

函數`tag`依次會接收到多個參數。

```javascript
function tag(stringArr, value1, value2){
  // ...
}

// 等同於

function tag(stringArr, ...values){
  // ...
}
```

`tag`函數的第一個參數是一個陣列，該陣列的成員是模板字符串中那些沒有變數替換的部分，也就是說，變數替換隻發生在陣列的第一個成員與第二個成員之間、第二個成員與第三個成員之間，以此類推。

`tag`函數的其他參數，都是模板字符串各個變數被替換後的值。由於本例中，模板字符串含有兩個變數，因此`tag`會接受到`value1`和`value2`兩個參數。

`tag`函數所有參數的實際值如下。

- 第一個參數：`['Hello ', ' world ', '']`
- 第二個參數: 15
- 第三個參數：50

也就是說，`tag`函數實際上以下面的形式調用。

```javascript
tag(['Hello ', ' world ', ''], 15, 50)
```

我們可以按照需要編寫`tag`函數的代碼。下面是`tag`函數的一種寫法，以及運行結果。

```javascript
let a = 5;
let b = 10;

function tag(s, v1, v2) {
  console.log(s[0]);
  console.log(s[1]);
  console.log(s[2]);
  console.log(v1);
  console.log(v2);

  return "OK";
}

tag`Hello ${ a + b } world ${ a * b}`;
// "Hello "
// " world "
// ""
// 15
// 50
// "OK"
```

下面是一個更複雜的例子。

```javascript
let total = 30;
let msg = passthru`The total is ${total} (${total*1.05} with tax)`;

function passthru(literals) {
  let result = '';
  let i = 0;

  while (i < literals.length) {
    result += literals[i++];
    if (i < arguments.length) {
      result += arguments[i];
    }
  }

  return result;
}

msg // "The total is 30 (31.5 with tax)"
```

上面這個例子展示了，如何將各個參數按照原來的位置拼合回去。

`passthru`函數採用 rest 參數的寫法如下。

```javascript
function passthru(literals, ...values) {
  let output = "";
  let index;
  for (index = 0; index < values.length; index++) {
    output += literals[index] + values[index];
  }

  output += literals[index]
  return output;
}
```

“標籤模板”的一個重要應用，就是過濾 HTML 字符串，防止用戶輸入惡意內容。

```javascript
let message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```

上面代碼中，`sender`變數往往是用戶提供的，經過`SaferHTML`函數處理，裡面的特殊字符都會被轉義。

```javascript
let sender = '<script>alert("abc")</script>'; // 惡意代碼
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;

message
// <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
```

標籤模板的另一個應用，就是多語言轉換（國際化處理）。

```javascript
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "歡迎訪問xxx，您是第xxxx位訪問者！"
```

模板字符串本身並不能取代 Mustache 之類的模板庫，因為沒有條件判斷和循環處理功能，但是通過標籤函數，你可以自己添加這些功能。

```javascript
// 下面的hashTemplate函數
// 是一個自定義的模板處理函數
let libraryHtml = hashTemplate`
  <ul>
    #for book in ${myBooks}
      <li><i>#{book.title}</i> by #{book.author}</li>
    #end
  </ul>
`;
```

除此之外，你甚至可以使用標籤模板，在 JavaScript 語言之中嵌入其他語言。

```javascript
jsx`
  <div>
    <input
      ref='input'
      onChange='${this.handleChange}'
      defaultValue='${this.state.value}' />
      ${this.state.value}
   </div>
`
```

上面的代碼通過`jsx`函數，將一個 DOM 字符串轉為 React 物件。你可以在 Github 找到`jsx`函數的[具體實現](https://gist.github.com/lygaret/a68220defa69174bdec5)。

下面則是一個假想的例子，通過`java`函數，在 JavaScript 代碼之中運行 Java 代碼。

```javascript
java`
class HelloWorldApp {
  public static void main(String[] args) {
    System.out.println(“Hello World!”); // Display the string.
  }
}
`
HelloWorldApp.main();
```

模板處理函數的第一個參數（模板字符串陣列），還有一個`raw`屬性。

```javascript
console.log`123`
// ["123", raw: Array[1]]
```

上面代碼中，`console.log`接受的參數，實際上是一個陣列。該陣列有一個`raw`屬性，保存的是轉義後的原字符串。

請看下面的例子。

```javascript
tag`First line\nSecond line`

function tag(strings) {
  console.log(strings.raw[0]);
  // strings.raw[0] 為 "First line\\nSecond line"
  // 打印輸出 "First line\nSecond line"
}
```

上面代碼中，`tag`函數的第一個參數`strings`，有一個`raw`屬性，也指向一個陣列。該陣列的成員與`strings`陣列完全一致。比如，`strings`陣列是`["First line\nSecond line"]`，那麼`strings.raw`陣列就是`["First line\\nSecond line"]`。兩者唯一的區別，就是字符串裡面的斜槓都被轉義了。比如，strings.raw 陣列會將`\n`視為`\\`和`n`兩個字符，而不是換行符。這是為了方便取得轉義之前的原始模板而設計的。

## String.raw()

ES6 還為原生的 String 物件，提供了一個`raw`方法。

`String.raw`方法，往往用來充當模板字符串的處理函數，返回一個斜槓都被轉義（即斜槓前面再加一個斜槓）的字符串，對應於替換變數後的模板字符串。

```javascript
String.raw`Hi\n${2+3}!`;
// 返回 "Hi\\n5!"

String.raw`Hi\u000A!`;
// 返回 "Hi\\u000A!"
```

如果原字符串的斜槓已經轉義，那麼`String.raw`會進行再次轉義。

```javascript
String.raw`Hi\\n`
// 返回 "Hi\\\\n"
```

`String.raw`方法可以作為處理模板字符串的基本方法，它會將所有變數替換，而且對斜槓進行轉義，方便下一步作為字符串來使用。

`String.raw`方法也可以作為正常的函數使用。這時，它的第一個參數，應該是一個具有`raw`屬性的物件，且`raw`屬性的值應該是一個陣列。

```javascript
String.raw({ raw: 'test' }, 0, 1, 2);
// 't0e1s2t'

// 等同於
String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
```

作為函數，`String.raw`的代碼實現基本如下。

```javascript
String.raw = function (strings, ...values) {
  let output = '';
  let index;
  for (index = 0; index < values.length; index++) {
    output += strings.raw[index] + values[index];
  }

  output += strings.raw[index]
  return output;
}
```

## 模板字符串的限制

前面提到標籤模板裡面，可以內嵌其他語言。但是，模板字符串默認會將字符串轉義，導致無法嵌入其他語言。

舉例來說，標籤模板裡面可以嵌入 LaTEX 語言。

```javascript
function latex(strings) {
  // ...
}

let document = latex`
\newcommand{\fun}{\textbf{Fun!}}  // 正常工作
\newcommand{\unicode}{\textbf{Unicode!}} // 報錯
\newcommand{\xerxes}{\textbf{King!}} // 報錯

Breve over the h goes \u{h}ere // 報錯
`
```

上面代碼中，變數`document`內嵌的模板字符串，對於 LaTEX 語言來說完全是合法的，但是 JavaScript 引擎會報錯。原因就在於字符串的轉義。

模板字符串會將`\u00FF`和`\u{42}`當作 Unicode 字符進行轉義，所以`\unicode`解析時報錯；而`\x56`會被當作十六進制字符串轉義，所以`\xerxes`會報錯。也就是說，`\u`和`\x`在 LaTEX 裡面有特殊含義，但是 JavaScript 將它們轉義了。

為瞭解決這個問題，ES2018 [放鬆](https://tc39.github.io/proposal-template-literal-revision/)了對標籤模板裡面的字符串轉義的限制。如果遇到不合法的字符串轉義，就返回`undefined`，而不是報錯，並且從`raw`屬性上面可以得到原始字符串。

```javascript
function tag(strs) {
  strs[0] === undefined
  strs.raw[0] === "\\unicode and \\u{55}";
}
tag`\unicode and \u{55}`
```

上面代碼中，模板字符串原本是應該報錯的，但是由於放鬆了對字符串轉義的限制，所以不報錯了，JavaScript 引擎將第一個字符設置為`undefined`，但是`raw`屬性依然可以得到原始字符串，因此`tag`函數還是可以對原字符串進行處理。

注意，這種對字符串轉義的放鬆，只在標籤模板解析字符串時生效，不是標籤模板的場合，依然會報錯。

```javascript
let bad = `bad escape sequence: \unicode`; // 報錯
```
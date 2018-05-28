# Module 的加載實現

上一章介紹了模塊的語法，本章介紹如何在瀏覽器和 Node 之中加載 ES6 模塊，以及實際開發中經常遇到的一些問題（比如循環加載）。

## 瀏覽器加載

### 傳統方法

HTML 網頁中，瀏覽器通過`<script>`標籤加載 JavaScript 腳本。

```html
<!-- 頁面內嵌的腳本 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部腳本 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>
```

上面代碼中，由於瀏覽器腳本的默認語言是 JavaScript，因此`type="application/javascript"`可以省略。

默認情況下，瀏覽器是同步加載 JavaScript 腳本，即渲染引擎遇到`<script>`標籤就會停下來，等到執行完腳本，再繼續向下渲染。如果是外部腳本，還必須加入腳本下載的時間。

如果腳本體積很大，下載和執行的時間就會很長，因此造成瀏覽器堵塞，用戶會感覺到瀏覽器“卡死”了，沒有任何響應。這顯然是很不好的體驗，所以瀏覽器允許腳本異步加載，下面就是兩種異步加載的語法。

```html
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

上面代碼中，`<script>`標籤打開`defer`或`async`屬性，腳本就會異步加載。渲染引擎遇到這一行命令，就會開始下載外部腳本，但不會等它下載和執行，而是直接執行後面的命令。

`defer`與`async`的區別是：`defer`要等到整個頁面在內存中正常渲染結束（DOM 結構完全生成，以及其他腳本執行完成），才會執行；`async`一旦下載完，渲染引擎就會中斷渲染，執行這個腳本以後，再繼續渲染。一句話，`defer`是“渲染完再執行”，`async`是“下載完就執行”。另外，如果有多個`defer`腳本，會按照它們在頁面出現的順序加載，而多個`async`腳本是不能保證加載順序的。

### 加載規則

瀏覽器加載 ES6 模塊，也使用`<script>`標籤，但是要加入`type="module"`屬性。

```html
<script type="module" src="./foo.js"></script>
```

上面代碼在網頁中插入一個模塊`foo.js`，由於`type`屬性設為`module`，所以瀏覽器知道這是一個 ES6 模塊。

瀏覽器對於帶有`type="module"`的`<script>`，都是異步加載，不會造成堵塞瀏覽器，即等到整個頁面渲染完，再執行模塊腳本，等同於打開了`<script>`標籤的`defer`屬性。

```html
<script type="module" src="./foo.js"></script>
<!-- 等同於 -->
<script type="module" src="./foo.js" defer></script>
```

如果網頁有多個`<script type="module">`，它們會按照在頁面出現的順序依次執行。

`<script>`標籤的`async`屬性也可以打開，這時只要加載完成，渲染引擎就會中斷渲染立即執行。執行完成後，再恢復渲染。

```html
<script type="module" src="./foo.js" async></script>
```

一旦使用了`async`屬性，`<script type="module">`就不會按照在頁面出現的順序執行，而是只要該模塊加載完成，就執行該模塊。

ES6 模塊也允許內嵌在網頁中，語法行為與加載外部腳本完全一致。

```html
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```

對於外部的模塊腳本（上例是`foo.js`），有幾點需要注意。

- 代碼是在模塊作用域之中運行，而不是在全局作用域運行。模塊內部的頂層變數，外部不可見。
- 模塊腳本自動採用嚴格模式，不管有沒有聲明`use strict`。
- 模塊之中，可以使用`import`命令加載其他模塊（`.js`後綴不可省略，需要提供絕對 URL 或相對 URL），也可以使用`export`命令輸出對外接口。
- 模塊之中，頂層的`this`關鍵字返回`undefined`，而不是指向`window`。也就是說，在模塊頂層使用`this`關鍵字，是無意義的。
- 同一個模塊如果加載多次，將只執行一次。

下面是一個示例模塊。

```javascript
import utils from 'https://example.com/js/utils.js';

const x = 1;

console.log(x === window.x); //false
console.log(this === undefined); // true
```

利用頂層的`this`等於`undefined`這個語法點，可以偵測當前代碼是否在 ES6 模塊之中。

```javascript
const isNotModuleScript = this !== undefined;
```

## ES6 模塊與 CommonJS 模塊的差異

討論 Node 加載 ES6 模塊之前，必須瞭解 ES6 模塊與 CommonJS 模塊完全不同。

它們有兩個重大差異。

- CommonJS 模塊輸出的是一個值的拷貝，ES6 模塊輸出的是值的引用。
- CommonJS 模塊是運行時加載，ES6 模塊是編譯時輸出接口。

第二個差異是因為 CommonJS 加載的是一個物件（即`module.exports`屬性），該物件只有在腳本運行完才會生成。而 ES6 模塊不是物件，它的對外接口只是一種靜態定義，在代碼靜態解析階段就會生成。

下面重點解釋第一個差異。

CommonJS 模塊輸出的是值的拷貝，也就是說，一旦輸出一個值，模塊內部的變化就影響不到這個值。請看下面這個模塊文件`lib.js`的例子。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
```

上面代碼輸出內部變數`counter`和改寫這個變數的內部方法`incCounter`。然後，在`main.js`裡面加載這個模塊。

```javascript
// main.js
var mod = require('./lib');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

上面代碼說明，`lib.js`模塊加載以後，它的內部變化就影響不到輸出的`mod.counter`了。這是因為`mod.counter`是一個原始類型的值，會被緩存。除非寫成一個函數，才能得到內部變動後的值。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```

上面代碼中，輸出的`counter`屬性實際上是一個取值器函數。現在再執行`main.js`，就可以正確讀取內部變數`counter`的變動了。

```bash
$ node main.js
3
4
```

ES6 模塊的運行機制與 CommonJS 不一樣。JS 引擎對腳本靜態分析的時候，遇到模塊加載命令`import`，就會生成一個只讀引用。等到腳本真正執行時，再根據這個只讀引用，到被加載的那個模塊裡面去取值。換句話說，ES6 的`import`有點像 Unix 系統的“符號連接”，原始值變了，`import`加載的值也會跟著變。因此，ES6 模塊是動態引用，並且不會緩存值，模塊裡面的變數綁定其所在的模塊。

還是舉上面的例子。

```javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

上面代碼說明，ES6 模塊輸入的變數`counter`是活的，完全反應其所在模塊`lib.js`內部的變化。

再舉一個出現在`export`一節中的例子。

```javascript
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);

// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);
```

上面代碼中，`m1.js`的變數`foo`，在剛加載時等於`bar`，過了 500 毫秒，又變為等於`baz`。

讓我們看看，`m2.js`能否正確讀取這個變化。

```bash
$ babel-node m2.js

bar
baz
```

上面代碼表明，ES6 模塊不會緩存運行結果，而是動態地去被加載的模塊取值，並且變數總是綁定其所在的模塊。

由於 ES6 輸入的模塊變數，只是一個“符號連接”，所以這個變數是只讀的，對它進行重新賦值會報錯。

```javascript
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```

上面代碼中，`main.js`從`lib.js`輸入變數`obj`，可以對`obj`添加屬性，但是重新賦值就會報錯。因為變數`obj`指向的地址是只讀的，不能重新賦值，這就好比`main.js`創造了一個名為`obj`的`const`變數。

最後，`export`通過接口，輸出的是同一個值。不同的腳本加載這個接口，得到的都是同樣的實例。

```javascript
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}

export let c = new C();
```

上面的腳本`mod.js`，輸出的是一個`C`的實例。不同的腳本加載這個模塊，得到的都是同一個實例。

```javascript
// x.js
import {c} from './mod';
c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';
```

現在執行`main.js`，輸出的是`1`。

```bash
$ babel-node main.js
1
```

這就證明了`x.js`和`y.js`加載的都是`C`的同一個實例。

## Node 加載

### 概述

Node 對 ES6 模塊的處理比較麻煩，因為它有自己的 CommonJS 模塊格式，與 ES6 模塊格式是不兼容的。目前的解決方案是，將兩者分開，ES6 模塊和 CommonJS 採用各自的加載方案。

Node 要求 ES6 模塊採用`.mjs`後綴文件名。也就是說，只要腳本文件裡面使用`import`或者`export`命令，那麼就必須採用`.mjs`後綴名。`require`命令不能加載`.mjs`文件，會報錯，只有`import`命令才可以加載`.mjs`文件。反過來，`.mjs`文件裡面也不能使用`require`命令，必須使用`import`。

目前，這項功能還在試驗階段。安裝 Node v8.5.0 或以上版本，要用`--experimental-modules`參數才能打開該功能。

```bash
$ node --experimental-modules my-app.mjs
```

為了與瀏覽器的`import`加載規則相同，Node 的`.mjs`文件支持 URL 路徑。

```javascript
import './foo?query=1'; // 加載 ./foo 傳入參數 ?query=1
```

上面代碼中，腳本路徑帶有參數`?query=1`，Node 會按 URL 規則解讀。同一個腳本只要參數不同，就會被加載多次，並且保存成不同的緩存。由於這個原因，只要文件名中含有`:`、`%`、`#`、`?`等特殊字符，最好對這些字符進行轉義。

目前，Node 的`import`命令只支持加載本地模塊（`file:`協議），不支持加載遠程模塊。

如果模塊名不含路徑，那麼`import`命令會去`node_modules`目錄尋找這個模塊。

```javascript
import 'baz';
import 'abc/123';
```

如果模塊名包含路徑，那麼`import`命令會按照路徑去尋找這個名字的腳本文件。

```javascript
import 'file:///etc/config/app.json';
import './foo';
import './foo?search';
import '../bar';
import '/baz';
```

如果腳本文件省略了後綴名，比如`import './foo'`，Node 會依次嘗試四個後綴名：`./foo.mjs`、`./foo.js`、`./foo.json`、`./foo.node`。如果這些腳本文件都不存在，Node 就會去加載`./foo/package.json`的`main`資料欄位指定的腳本。如果`./foo/package.json`不存在或者沒有`main`資料欄位，那麼就會依次加載`./foo/index.mjs`、`./foo/index.js`、`./foo/index.json`、`./foo/index.node`。如果以上四個文件還是都不存在，就會拋出錯誤。

最後，Node 的`import`命令是異步加載，這一點與瀏覽器的處理方法相同。

### 內部變數

ES6 模塊應該是通用的，同一個模塊不用修改，就可以用在瀏覽器環境和服務器環境。為了達到這個目標，Node 規定 ES6 模塊之中不能使用 CommonJS 模塊的特有的一些內部變數。

首先，就是`this`關鍵字。ES6 模塊之中，頂層的`this`指向`undefined`；CommonJS 模塊的頂層`this`指向當前模塊，這是兩者的一個重大差異。

其次，以下這些頂層變數在 ES6 模塊之中都是不存在的。

- `arguments`
- `require`
- `module`
- `exports`
- `__filename`
- `__dirname`

如果你一定要使用這些變數，有一個變通方法，就是寫一個 CommonJS 模塊輸出這些變數，然後再用 ES6 模塊加載這個 CommonJS 模塊。但是這樣一來，該 ES6 模塊就不能直接用於瀏覽器環境了，所以不推薦這樣做。

```javascript
// expose.js
module.exports = {__dirname};

// use.mjs
import expose from './expose.js';
const {__dirname} = expose;
```

上面代碼中，`expose.js`是一個 CommonJS 模塊，輸出變數`__dirname`，該變數在 ES6 模塊之中不存在。ES6 模塊加載`expose.js`，就可以得到`__dirname`。

### ES6 模塊加載 CommonJS 模塊

CommonJS 模塊的輸出都定義在`module.exports`這個屬性上面。Node 的`import`命令加載 CommonJS 模塊，Node 會自動將`module.exports`屬性，當作模塊的默認輸出，即等同於`export default xxx`。

下面是一個 CommonJS 模塊。

```javascript
// a.js
module.exports = {
  foo: 'hello',
  bar: 'world'
};

// 等同於
export default {
  foo: 'hello',
  bar: 'world'
};
```

`import`命令加載上面的模塊，`module.exports`會被視為默認輸出，即`import`命令實際上輸入的是這樣一個物件`{ default: module.exports }`。

所以，一共有三種寫法，可以拿到 CommonJS 模塊的`module.exports`。

```javascript
// 寫法一
import baz from './a';
// baz = {foo: 'hello', bar: 'world'};

// 寫法二
import {default as baz} from './a';
// baz = {foo: 'hello', bar: 'world'};

// 寫法三
import * as baz from './a';
// baz = {
//   get default() {return module.exports;},
//   get foo() {return this.default.foo}.bind(baz),
//   get bar() {return this.default.bar}.bind(baz)
// }
```

上面代碼的第三種寫法，可以通過`baz.default`拿到`module.exports`。`foo`屬性和`bar`屬性就是可以通過這種方法拿到了`module.exports`。

下面是一些例子。

```javascript
// b.js
module.exports = null;

// es.js
import foo from './b';
// foo = null;

import * as bar from './b';
// bar = { default:null };
```

上面代碼中，`es.js`採用第二種寫法時，要通過`bar.default`這樣的寫法，才能拿到`module.exports`。

```javascript
// c.js
module.exports = function two() {
  return 2;
};

// es.js
import foo from './c';
foo(); // 2

import * as bar from './c';
bar.default(); // 2
bar(); // throws, bar is not a function
```

上面代碼中，`bar`本身是一個物件，不能當作函數調用，只能通過`bar.default`調用。

CommonJS 模塊的輸出緩存機制，在 ES6 加載方式下依然有效。

```javascript
// foo.js
module.exports = 123;
setTimeout(_ => module.exports = null);
```

上面代碼中，對於加載`foo.js`的腳本，`module.exports`將一直是`123`，而不會變成`null`。

由於 ES6 模塊是編譯時確定輸出接口，CommonJS 模塊是運行時確定輸出接口，所以採用`import`命令加載 CommonJS 模塊時，不允許採用下面的寫法。

```javascript
// 不正確
import { readFile } from 'fs';
```

上面的寫法不正確，因為`fs`是 CommonJS 格式，只有在運行時才能確定`readFile`接口，而`import`命令要求編譯時就確定這個接口。解決方法就是改為整體輸入。

```javascript
// 正確的寫法一
import * as express from 'express';
const app = express.default();

// 正確的寫法二
import express from 'express';
const app = express();
```

### CommonJS 模塊加載 ES6 模塊

CommonJS 模塊加載 ES6 模塊，不能使用`require`命令，而要使用`import()`函數。ES6 模塊的所有輸出接口，會成為輸入物件的屬性。

```javascript
// es.mjs
let foo = { bar: 'my-default' };
export default foo;

// cjs.js
const es_namespace = await import('./es.mjs');
// es_namespace = {
//   get default() {
//     ...
//   }
// }
console.log(es_namespace.default);
// { bar:'my-default' }
```

上面代碼中，`default`接口變成了`es_namespace.default`屬性。

下面是另一個例子。

```javascript
// es.js
export let foo = { bar:'my-default' };
export { foo as bar };
export function f() {};
export class c {};

// cjs.js
const es_namespace = await import('./es');
// es_namespace = {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```

## 循環加載

“循環加載”（circular dependency）指的是，`a`腳本的執行依賴`b`腳本，而`b`腳本的執行又依賴`a`腳本。

```javascript
// a.js
var b = require('b');

// b.js
var a = require('a');
```

通常，“循環加載”表示存在強耦合，如果處理不好，還可能導致遞歸加載，使得程序無法執行，因此應該避免出現。

但是實際上，這是很難避免的，尤其是依賴關係複雜的大專案，很容易出現`a`依賴`b`，`b`依賴`c`，`c`又依賴`a`這樣的情況。這意味著，模塊加載機制必須考慮“循環加載”的情況。

對於 JavaScript 語言來說，目前最常見的兩種模塊格式 CommonJS 和 ES6，處理“循環加載”的方法是不一樣的，返回的結果也不一樣。

### CommonJS 模塊的加載原理

介紹 ES6 如何處理“循環加載”之前，先介紹目前最流行的 CommonJS 模塊格式的加載原理。

CommonJS 的一個模塊，就是一個腳本文件。`require`命令第一次加載該腳本，就會執行整個腳本，然後在內存生成一個物件。

```javascript
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

上面代碼就是 Node 內部加載模塊後生成的一個物件。該物件的`id`屬性是模塊名，`exports`屬性是模塊輸出的各個接口，`loaded`屬性是一個布爾值，表示該模塊的腳本是否執行完畢。其他還有很多屬性，這裡都省略了。

以後需要用到這個模塊的時候，就會到`exports`屬性上面取值。即使再次執行`require`命令，也不會再次執行該模塊，而是到緩存之中取值。也就是說，CommonJS 模塊無論加載多少次，都只會在第一次加載時運行一次，以後再加載，就返回第一次運行的結果，除非手動清除系統緩存。

### CommonJS 模塊的循環加載

CommonJS 模塊的重要特性是加載時執行，即腳本代碼在`require`的時候，就會全部執行。一旦出現某個模塊被"循環加載"，就只輸出已經執行的部分，還未執行的部分不會輸出。

讓我們來看，Node [官方文檔](https://nodejs.org/api/modules.html#modules_cycles)裡面的例子。腳本文件`a.js`代碼如下。

```javascript
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 執行完畢');
```

上面代碼之中，`a.js`腳本先輸出一個`done`變數，然後加載另一個腳本文件`b.js`。注意，此時`a.js`代碼就停在這裡，等待`b.js`執行完畢，再往下執行。

再看`b.js`的代碼。

```javascript
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 執行完畢');
```

上面代碼之中，`b.js`執行到第二行，就會去加載`a.js`，這時，就發生了“循環加載”。系統會去`a.js`模塊對應物件的`exports`屬性取值，可是因為`a.js`還沒有執行完，從`exports`屬性只能取回已經執行的部分，而不是最後的值。

`a.js`已經執行的部分，只有一行。

```javascript
exports.done = false;
```

因此，對於`b.js`來說，它從`a.js`只輸入一個變數`done`，值為`false`。

然後，`b.js`接著往下執行，等到全部執行完畢，再把執行權交還給`a.js`。於是，`a.js`接著往下執行，直到執行完畢。我們寫一個腳本`main.js`，驗證這個過程。

```javascript
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```

執行`main.js`，運行結果如下。

```bash
$ node main.js

在 b.js 之中，a.done = false
b.js 執行完畢
在 a.js 之中，b.done = true
a.js 執行完畢
在 main.js 之中, a.done=true, b.done=true
```

上面的代碼證明了兩件事。一是，在`b.js`之中，`a.js`沒有執行完畢，只執行了第一行。二是，`main.js`執行到第二行時，不會再次執行`b.js`，而是輸出緩存的`b.js`的執行結果，即它的第四行。

```javascript
exports.done = true;
```

總之，CommonJS 輸入的是被輸出值的拷貝，不是引用。

另外，由於 CommonJS 模塊遇到循環加載時，返回的是當前已經執行的部分的值，而不是代碼全部執行後的值，兩者可能會有差異。所以，輸入變數的時候，必須非常小心。

```javascript
var a = require('a'); // 安全的寫法
var foo = require('a').foo; // 危險的寫法

exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};

exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一個部分加載時的值
};
```

上面代碼中，如果發生循環加載，`require('a').foo`的值很可能後面會被改寫，改用`require('a')`會更保險一點。

### ES6 模塊的循環加載

ES6 處理“循環加載”與 CommonJS 有本質的不同。ES6 模塊是動態引用，如果使用`import`從一個模塊加載變數（即`import foo from 'foo'`），那些變數不會被緩存，而是成為一個指向被加載模塊的引用，需要開發者自己保證，真正取值的時候能夠取到值。

請看下面這個例子。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';
```

上面代碼中，`a.mjs`加載`b.mjs`，`b.mjs`又加載`a.mjs`，構成循環加載。執行`a.mjs`，結果如下。

```bash
$ node --experimental-modules a.mjs
b.mjs
ReferenceError: foo is not defined
```

上面代碼中，執行`a.mjs`以後會報錯，`foo`變數未定義，這是為什麼？

讓我們一行行來看，ES6 循環加載是怎麼處理的。首先，執行`a.mjs`以後，引擎發現它加載了`b.mjs`，因此會優先執行`b.mjs`，然後再執行`a.mjs`。接著，執行`b.mjs`的時候，已知它從`a.mjs`輸入了`foo`接口，這時不會去執行`a.mjs`，而是認為這個接口已經存在了，繼續往下執行。執行到第三行`console.log(foo)`的時候，才發現這個接口根本沒定義，因此報錯。

解決這個問題的方法，就是讓`b.mjs`運行的時候，`foo`已經有定義了。這可以通過將`foo`寫成函數來解決。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
```

這時再執行`a.mjs`就可以得到預期結果。

```bash
$ node --experimental-modules a.mjs
b.mjs
foo
a.mjs
bar
```

這是因為函數具有提升作用，在執行`import {bar} from './b'`時，函數`foo`就已經有定義了，所以`b.mjs`加載的時候不會報錯。這也意味著，如果把函數`foo`改寫成函數表達式，也會報錯。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
const foo = () => 'foo';
export {foo};
```

上面代碼的第四行，改成了函數表達式，就不具有提升作用，執行就會報錯。

我們再來看 ES6 模塊加載器[SystemJS](https://github.com/ModuleLoader/es6-module-loader/blob/master/docs/circular-references-bindings.md)給出的一個例子。

```javascript
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n === 0 || odd(n - 1);
}

// odd.js
import { even } from './even';
export function odd(n) {
  return n !== 0 && even(n - 1);
}
```

上面代碼中，`even.js`裡面的函數`even`有一個參數`n`，只要不等於 0，就會減去 1，傳入加載的`odd()`。`odd.js`也會做類似操作。

運行上面這段代碼，結果如下。

```javascript
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
```

上面代碼中，參數`n`從 10 變為 0 的過程中，`even()`一共會執行 6 次，所以變數`counter`等於 6。第二次調用`even()`時，參數`n`從 20 變為 0，`even()`一共會執行 11 次，加上前面的 6 次，所以變數`counter`等於 17。

這個例子要是改寫成 CommonJS，就根本無法執行，會報錯。

```javascript
// even.js
var odd = require('./odd');
var counter = 0;
exports.counter = counter;
exports.even = function (n) {
  counter++;
  return n == 0 || odd(n - 1);
}

// odd.js
var even = require('./even').even;
module.exports = function (n) {
  return n != 0 && even(n - 1);
}
```

上面代碼中，`even.js`加載`odd.js`，而`odd.js`又去加載`even.js`，形成“循環加載”。這時，執行引擎就會輸出`even.js`已經執行的部分（不存在任何結果），所以在`odd.js`之中，變數`even`等於`undefined`，等到後面調用`even(n - 1)`就會報錯。

```bash
$ node
> var m = require('./even');
> m.even(10)
TypeError: even is not a function
```

## ES6 模塊的轉碼

瀏覽器目前還不支持 ES6 模塊，為了現在就能使用，可以將轉為 ES5 的寫法。除了 Babel 可以用來轉碼之外，還有以下兩個方法，也可以用來轉碼。

### ES6 module transpiler

[ES6 module transpiler](https://github.com/esnext/es6-module-transpiler)是 square 公司開源的一個轉碼器，可以將 ES6 模塊轉為 CommonJS 模塊或 AMD 模塊的寫法，從而在瀏覽器中使用。

首先，安裝這個轉碼器。

```bash
$ npm install -g es6-module-transpiler
```

然後，使用`compile-modules convert`命令，將 ES6 模塊文件轉碼。

```bash
$ compile-modules convert file1.js file2.js
```

`-o`參數可以指定轉碼後的文件名。

```bash
$ compile-modules convert -o out.js file1.js
```

### SystemJS

另一種解決方法是使用 [SystemJS](https://github.com/systemjs/systemjs)。它是一個墊片庫（polyfill），可以在瀏覽器內加載 ES6 模塊、AMD 模塊和 CommonJS 模塊，將其轉為 ES5 格式。它在後台調用的是 Google 的 Traceur 轉碼器。

使用時，先在網頁內載入`system.js`文件。

```html
<script src="system.js"></script>
```

然後，使用`System.import`方法加載模塊文件。

```html
<script>
  System.import('./app.js');
</script>
```

上面代碼中的`./app`，指的是當前目錄下的 app.js 文件。它可以是 ES6 模塊文件，`System.import`會自動將其轉碼。

需要注意的是，`System.import`使用異步加載，返回一個 Promise 物件，可以針對這個物件編程。下面是一個模塊文件。

```javascript
// app/es6-file.js:

export class q {
  constructor() {
    this.es6 = 'hello';
  }
}
```

然後，在網頁內加載這個模塊文件。

```html
<script>

System.import('app/es6-file').then(function(m) {
  console.log(new m.q().es6); // hello
});

</script>
```

上面代碼中，`System.import`方法返回的是一個 Promise 物件，所以可以用`then`方法指定回調函數。
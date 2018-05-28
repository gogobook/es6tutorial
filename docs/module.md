# Module 的語法

## 概述

歷史上，JavaScript 一直沒有模塊（module）體系，無法將一個大程序拆分成互相依賴的小文件，再用簡單的方法拼裝起來。其他語言都有這項功能，比如 Ruby 的`require`、Python 的`import`，甚至就連 CSS 都有`@import`，但是 JavaScript 任何這方面的支持都沒有，這對開發大型的、複雜的專案形成了巨大障礙。

在 ES6 之前，社區制定了一些模塊加載方案，最主要的有 CommonJS 和 AMD 兩種。前者用於服務器，後者用於瀏覽器。ES6 在語言標準的層面上，實現了模塊功能，而且實現得相當簡單，完全可以取代 CommonJS 和 AMD 規範，成為瀏覽器和服務器通用的模塊解決方案。

ES6 模塊的設計思想是儘量的靜態化，使得編譯時就能確定模塊的依賴關係，以及輸入和輸出的變數。CommonJS 和 AMD 模塊，都只能在運行時確定這些東西。比如，CommonJS 模塊就是物件，輸入時必須查找物件屬性。

```javascript
// CommonJS模塊
let { stat, exists, readFile } = require('fs');

// 等同於
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

上面代碼的實質是整體加載`fs`模塊（即加載`fs`的所有方法），生成一個物件（`_fs`），然後再從這個物件上面讀取 3 個方法。這種加載稱為“運行時加載”，因為只有運行時才能得到這個物件，導致完全沒辦法在編譯時做“靜態優化”。

ES6 模塊不是物件，而是通過`export`命令顯式指定輸出的代碼，再通過`import`命令輸入。

```javascript
// ES6模塊
import { stat, exists, readFile } from 'fs';
```

上面代碼的實質是從`fs`模塊加載 3 個方法，其他方法不加載。這種加載稱為“編譯時加載”或者靜態加載，即 ES6 可以在編譯時就完成模塊加載，效率要比 CommonJS 模塊的加載方式高。當然，這也導致了沒法引用 ES6 模塊本身，因為它不是物件。

由於 ES6 模塊是編譯時加載，使得靜態分析成為可能。有了它，就能進一步拓寬 JavaScript 的語法，比如引入宏（macro）和類型檢驗（type system）這些只能靠靜態分析實現的功能。

除了靜態加載帶來的各種好處，ES6 模塊還有以下好處。

- 不再需要`UMD`模塊格式了，將來服務器和瀏覽器都會支持 ES6 模塊格式。目前，通過各種工具庫，其實已經做到了這一點。
- 將來瀏覽器的新 API 就能用模塊格式提供，不再必須做成全局變數或者`navigator`物件的屬性。
- 不再需要物件作為命名空間（比如`Math`物件），未來這些功能可以通過模塊提供。

本章介紹 ES6 模塊的語法，下一章介紹如何在瀏覽器和 Node 之中，加載 ES6 模塊。

## 嚴格模式

ES6 的模塊自動採用嚴格模式，不管你有沒有在模塊頭部加上`"use strict";`。

嚴格模式主要有以下限制。

- 變數必須聲明後再使用
- 函數的參數不能有同名屬性，否則報錯
- 不能使用`with`語句
- 不能對只讀屬性賦值，否則報錯
- 不能使用前綴 0 表示八進制數，否則報錯
- 不能刪除不可刪除的屬性，否則報錯
- 不能刪除變數`delete prop`，會報錯，只能刪除屬性`delete global[prop]`
- `eval`不會在它的外層作用域引入變數
- `eval`和`arguments`不能被重新賦值
- `arguments`不會自動反映函數參數的變化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局物件
- 不能使用`fn.caller`和`fn.arguments`獲取函數調用的堆棧
- 增加了保留字（比如`protected`、`static`和`interface`）

上面這些限制，模塊都必須遵守。由於嚴格模式是 ES5 引入的，不屬於 ES6，所以請參閱相關 ES5 書籍，本書不再詳細介紹了。

其中，尤其需要注意`this`的限制。ES6 模塊之中，頂層的`this`指向`undefined`，即不應該在頂層代碼使用`this`。

## export 命令

模塊功能主要由兩個命令構成：`export`和`import`。`export`命令用於規定模塊的對外接口，`import`命令用於輸入其他模塊提供的功能。

一個模塊就是一個獨立的文件。該文件內部的所有變數，外部無法獲取。如果你希望外部能夠讀取模塊內部的某個變數，就必須使用`export`關鍵字輸出該變數。下面是一個 JS 文件，裡面使用`export`命令輸出變數。

```javascript
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

上面代碼是`profile.js`文件，保存了用戶信息。ES6 將其視為一個模塊，裡面用`export`命令對外部輸出了三個變數。

`export`的寫法，除了像上面這樣，還有另外一種。

```javascript
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

上面代碼在`export`命令後面，使用大括號指定所要輸出的一組變數。它與前一種寫法（直接放置在`var`語句前）是等價的，但是應該優先考慮使用這種寫法。因為這樣就可以在腳本尾部，一眼看清楚輸出了哪些變數。

`export`命令除了輸出變數，還可以輸出函數或類（class）。

```javascript
export function multiply(x, y) {
  return x * y;
};
```

上面代碼對外輸出一個函數`multiply`。

通常情況下，`export`輸出的變數就是本來的名字，但是可以使用`as`關鍵字重命名。

```javascript
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

上面代碼使用`as`關鍵字，重命名了函數`v1`和`v2`的對外接口。重命名後，`v2`可以用不同的名字輸出兩次。

需要特別注意的是，`export`命令規定的是對外的接口，必須與模塊內部的變數建立一一對應關係。

```javascript
// 報錯
export 1;

// 報錯
var m = 1;
export m;
```

上面兩種寫法都會報錯，因為沒有提供對外的接口。第一種寫法直接輸出 1，第二種寫法通過變數`m`，還是直接輸出 1。`1`只是一個值，不是接口。正確的寫法是下面這樣。

```javascript
// 寫法一
export var m = 1;

// 寫法二
var m = 1;
export {m};

// 寫法三
var n = 1;
export {n as m};
```

上面三種寫法都是正確的，規定了對外的接口`m`。其他腳本可以通過這個接口，取到值`1`。它們的實質是，在接口名與模塊內部變數之間，建立了一一對應的關係。

同樣的，`function`和`class`的輸出，也必須遵守這樣的寫法。

```javascript
// 報錯
function f() {}
export f;

// 正確
export function f() {};

// 正確
function f() {}
export {f};
```

另外，`export`語句輸出的接口，與其對應的值是動態綁定關係，即通過該接口，可以取到模塊內部實時的值。

```javascript
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

上面代碼輸出變數`foo`，值為`bar`，500 毫秒之後變成`baz`。

這一點與 CommonJS 規範完全不同。CommonJS 模塊輸出的是值的緩存，不存在動態更新，詳見下文《Module 的加載實現》一節。

最後，`export`命令可以出現在模塊的任何位置，只要處於模塊頂層就可以。如果處於塊級作用域內，就會報錯，下一節的`import`命令也是如此。這是因為處於條件代碼塊之中，就沒法做靜態優化了，違背了 ES6 模塊的設計初衷。

```javascript
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```

上面代碼中，`export`語句放在函數之中，結果報錯。

## import 命令

使用`export`命令定義了模塊的對外接口以後，其他 JS 文件就可以通過`import`命令加載這個模塊。

```javascript
// main.js
import {firstName, lastName, year} from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```

上面代碼的`import`命令，用於加載`profile.js`文件，並從中輸入變數。`import`命令接受一對大括號，裡面指定要從其他模塊導入的變數名。大括號裡面的變數名，必須與被導入模塊（`profile.js`）對外接口的名稱相同。

如果想為輸入的變數重新取一個名字，`import`命令要使用`as`關鍵字，將輸入的變數重命名。

```javascript
import { lastName as surname } from './profile.js';
```

`import`命令輸入的變數都是只讀的，因為它的本質是輸入接口。也就是說，不允許在加載模塊的腳本裡面，改寫接口。

```javascript
import {a} from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;
```

上面代碼中，腳本加載了變數`a`，對其重新賦值就會報錯，因為`a`是一個只讀的接口。但是，如果`a`是一個物件，改寫`a`的屬性是允許的。

```javascript
import {a} from './xxx.js'

a.foo = 'hello'; // 合法操作
```

上面代碼中，`a`的屬性可以成功改寫，並且其他模塊也可以讀到改寫後的值。不過，這種寫法很難查錯，建議凡是輸入的變數，都當作完全只讀，輕易不要改變它的屬性。

`import`後面的`from`指定模塊文件的位置，可以是相對路徑，也可以是絕對路徑，`.js`後綴可以省略。如果只是模塊名，不帶有路徑，那麼必須有配置文件，告訴 JavaScript 引擎該模塊的位置。

```javascript
import {myMethod} from 'util';
```

上面代碼中，`util`是模塊文件名，由於不帶有路徑，必須通過配置，告訴引擎怎麼取到這個模塊。

注意，`import`命令具有提升效果，會提升到整個模塊的頭部，首先執行。

```javascript
foo();

import { foo } from 'my_module';
```

上面的代碼不會報錯，因為`import`的執行早於`foo`的調用。這種行為的本質是，`import`命令是編譯階段執行的，在代碼運行之前。

由於`import`是靜態執行，所以不能使用表達式和變數，這些只有在運行時才能得到結果的語法結構。

```javascript
// 報錯
import { 'f' + 'oo' } from 'my_module';

// 報錯
let module = 'my_module';
import { foo } from module;

// 報錯
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

上面三種寫法都會報錯，因為它們用到了表達式、變數和`if`結構。在靜態分析階段，這些語法都是沒法得到值的。

最後，`import`語句會執行所加載的模塊，因此可以有下面的寫法。

```javascript
import 'lodash';
```

上面代碼僅僅執行`lodash`模塊，但是不輸入任何值。

如果多次重複執行同一句`import`語句，那麼只會執行一次，而不會執行多次。

```javascript
import 'lodash';
import 'lodash';
```

上面代碼加載了兩次`lodash`，但是只會執行一次。

```javascript
import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同於
import { foo, bar } from 'my_module';
```

上面代碼中，雖然`foo`和`bar`在兩個語句中加載，但是它們對應的是同一個`my_module`實例。也就是說，`import`語句是 Singleton 模式。

目前階段，通過 Babel 轉碼，CommonJS 模塊的`require`命令和 ES6 模塊的`import`命令，可以寫在同一個模塊裡面，但是最好不要這樣做。因為`import`在靜態解析階段執行，所以它是一個模塊之中最早執行的。下面的代碼可能不會得到預期結果。

```javascript
require('core-js/modules/es6.symbol');
require('core-js/modules/es6.promise');
import React from 'React';
```

## 模塊的整體加載

除了指定加載某個輸出值，還可以使用整體加載，即用星號（`*`）指定一個物件，所有輸出值都加載在這個物件上面。

下面是一個`circle.js`文件，它輸出兩個方法`area`和`circumference`。

```javascript
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
```

現在，加載這個模塊。

```javascript
// main.js

import { area, circumference } from './circle';

console.log('圓面積：' + area(4));
console.log('圓周長：' + circumference(14));
```

上面寫法是逐一指定要加載的方法，整體加載的寫法如下。

```javascript
import * as circle from './circle';

console.log('圓面積：' + circle.area(4));
console.log('圓周長：' + circle.circumference(14));
```

注意，模塊整體加載所在的那個物件（上例是`circle`），應該是可以靜態分析的，所以不允許運行時改變。下面的寫法都是不允許的。

```javascript
import * as circle from './circle';

// 下面兩行都是不允許的
circle.foo = 'hello';
circle.area = function () {};
```

## export default 命令

從前面的例子可以看出，使用`import`命令的時候，用戶需要知道所要加載的變數名或函數名，否則無法加載。但是，用戶肯定希望快速上手，未必願意閱讀文檔，去瞭解模塊有哪些屬性和方法。

為了給用戶提供方便，讓他們不用閱讀文檔就能加載模塊，就要用到`export default`命令，為模塊指定默認輸出。

```javascript
// export-default.js
export default function () {
  console.log('foo');
}
```

上面代碼是一個模塊文件`export-default.js`，它的默認輸出是一個函數。

其他模塊加載該模塊時，`import`命令可以為該匿名函數指定任意名字。

```javascript
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

上面代碼的`import`命令，可以用任意名稱指向`export-default.js`輸出的方法，這時就不需要知道原模塊輸出的函數名。需要注意的是，這時`import`命令後面，不使用大括號。

`export default`命令用在非匿名函數前，也是可以的。

```javascript
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者寫成

function foo() {
  console.log('foo');
}

export default foo;
```

上面代碼中，`foo`函數的函數名`foo`，在模塊外部是無效的。加載的時候，視同匿名函數加載。

下面比較一下默認輸出和正常輸出。

```javascript
// 第一組
export default function crc32() { // 輸出
  // ...
}

import crc32 from 'crc32'; // 輸入

// 第二組
export function crc32() { // 輸出
  // ...
};

import {crc32} from 'crc32'; // 輸入
```

上面代碼的兩組寫法，第一組是使用`export default`時，對應的`import`語句不需要使用大括號；第二組是不使用`export default`時，對應的`import`語句需要使用大括號。

`export default`命令用於指定模塊的默認輸出。顯然，一個模塊只能有一個默認輸出，因此`export default`命令只能使用一次。所以，import命令後面才不用加大括號，因為只可能唯一對應`export default`命令。

本質上，`export default`就是輸出一個叫做`default`的變數或方法，然後系統允許你為它取任意名字。所以，下面的寫法是有效的。

```javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同於
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同於
// import foo from 'modules';
```

正是因為`export default`命令其實只是輸出一個叫做`default`的變數，所以它後面不能跟變數聲明語句。

```javascript
// 正確
export var a = 1;

// 正確
var a = 1;
export default a;

// 錯誤
export default var a = 1;
```

上面代碼中，`export default a`的含義是將變數`a`的值賦給變數`default`。所以，最後一種寫法會報錯。

同樣地，因為`export default`命令的本質是將後面的值，賦給`default`變數，所以可以直接將一個值寫在`export default`之後。

```javascript
// 正確
export default 42;

// 報錯
export 42;
```

上面代碼中，後一句報錯是因為沒有指定對外的接口，而前一句指定外對接口為`default`。

有了`export default`命令，輸入模塊時就非常直觀了，以輸入 lodash 模塊為例。

```javascript
import _ from 'lodash';
```

如果想在一條`import`語句中，同時輸入默認方法和其他接口，可以寫成下面這樣。

```javascript
import _, { each, each as forEach } from 'lodash';
```

對應上面代碼的`export`語句如下。

```javascript
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach };
```

上面代碼的最後一行的意思是，暴露出`forEach`接口，默認指向`each`接口，即`forEach`和`each`指向同一個方法。

`export default`也可以用來輸出類。

```javascript
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

## export 與 import 的復合寫法

如果在一個模塊之中，先輸入後輸出同一個模塊，`import`語句可以與`export`語句寫在一起。

```javascript
export { foo, bar } from 'my_module';

// 可以簡單理解為
import { foo, bar } from 'my_module';
export { foo, bar };
```

上面代碼中，`export`和`import`語句可以結合在一起，寫成一行。但需要注意的是，寫成一行以後，`foo`和`bar`實際上並沒有被導入當前模塊，只是相當於對外轉發了這兩個接口，導致當前模塊不能直接使用`foo`和`bar`。

模塊的接口改名和整體輸出，也可以採用這種寫法。

```javascript
// 接口改名
export { foo as myFoo } from 'my_module';

// 整體輸出
export * from 'my_module';
```

默認接口的寫法如下。

```javascript
export { default } from 'foo';
```

具名接口改為默認接口的寫法如下。

```javascript
export { es6 as default } from './someModule';

// 等同於
import { es6 } from './someModule';
export default es6;
```

同樣地，默認接口也可以改名為具名接口。

```javascript
export { default as es6 } from './someModule';
```

下面三種`import`語句，沒有對應的復合寫法。

```javascript
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```

為了做到形式的對稱，現在有[提案](https://github.com/leebyron/ecmascript-export-default-from)，提出補上這三種復合寫法。

```javascript
export * as someIdentifier from "someModule";
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```

## 模塊的繼承

模塊之間也可以繼承。

假設有一個`circleplus`模塊，繼承了`circle`模塊。

```javascript
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
```

上面代碼中的`export *`，表示再輸出`circle`模塊的所有屬性和方法。注意，`export *`命令會忽略`circle`模塊的`default`方法。然後，上面代碼又輸出了自定義的`e`變數和默認方法。

這時，也可以將`circle`的屬性或方法，改名後再輸出。

```javascript
// circleplus.js

export { area as circleArea } from 'circle';
```

上面代碼表示，只輸出`circle`模塊的`area`方法，且將其改名為`circleArea`。

加載上面模塊的寫法如下。

```javascript
// main.js

import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));
```

上面代碼中的`import exp`表示，將`circleplus`模塊的默認方法加載為`exp`方法。

## 跨模塊常量

本書介紹`const`命令的時候說過，`const`聲明的常量只在當前代碼塊有效。如果想設置跨模塊的常量（即跨多個文件），或者說一個值要被多個模塊共享，可以採用下面的寫法。

```javascript
// constants.js 模塊
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模塊
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模塊
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一個專門的`constants`目錄，將各種常量寫在不同的文件裡面，保存在該目錄下。

```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然後，將這些文件輸出的常量，合併在`index.js`裡面。

```javascript
// constants/index.js
export {db} from './db';
export {users} from './users';
```

使用的時候，直接加載`index.js`就可以了。

```javascript
// script.js
import {db, users} from './index';
```

## import()

### 簡介

前面介紹過，`import`命令會被 JavaScript 引擎靜態分析，先於模塊內的其他語句執行（`import`命令叫做“連接” binding 其實更合適）。所以，下面的代碼會報錯。

```javascript
// 報錯
if (x === 2) {
  import MyModual from './myModual';
}
```

上面代碼中，引擎處理`import`語句是在編譯時，這時不會去分析或執行`if`語句，所以`import`語句放在`if`代碼塊之中毫無意義，因此會報句法錯誤，而不是執行時錯誤。也就是說，`import`和`export`命令只能在模塊的頂層，不能在代碼塊之中（比如，在`if`代碼塊之中，或在函數之中）。

這樣的設計，固然有利於編譯器提高效率，但也導致無法在運行時加載模塊。在語法上，條件加載就不可能實現。如果`import`命令要取代 Node 的`require`方法，這就形成了一個障礙。因為`require`是運行時加載模塊，`import`命令無法取代`require`的動態加載功能。

```javascript
const path = './' + fileName;
const myModual = require(path);
```

上面的語句就是動態加載，`require`到底加載哪一個模塊，只有運行時才知道。`import`命令做不到這一點。

因此，有一個[提案](https://github.com/tc39/proposal-dynamic-import)，建議引入`import()`函數，完成動態加載。

```javascript
import(specifier)
```

上面代碼中，`import`函數的參數`specifier`，指定所要加載的模塊的位置。`import`命令能夠接受什麼參數，`import()`函數就能接受什麼參數，兩者區別主要是後者為動態加載。

`import()`返回一個 Promise 物件。下面是一個例子。

```javascript
const main = document.querySelector('main');

import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```

`import()`函數可以用在任何地方，不僅僅是模塊，非模塊的腳本也可以使用。它是運行時執行，也就是說，什麼時候運行到這一句，就會加載指定的模塊。另外，`import()`函數與所加載的模塊沒有靜態連接關係，這點也是與`import`語句不相同。`import()`類似於 Node 的`require`方法，區別主要是前者是異步加載，後者是同步加載。

### 適用場合

下面是`import()`的一些適用場合。

（1）按需加載。

`import()`可以在需要的時候，再加載某個模塊。

```javascript
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
```

上面代碼中，`import()`方法放在`click`事件的監聽函數之中，只有用戶點擊了按鈕，才會加載這個模塊。

（2）條件加載

`import()`可以放在`if`代碼塊，根據不同的情況，加載不同的模塊。

```javascript
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}
```

上面代碼中，如果滿足條件，就加載模塊 A，否則加載模塊 B。

（3）動態的模塊路徑

`import()`允許模塊路徑動態生成。

```javascript
import(f())
.then(...);
```

上面代碼中，根據函數`f`的返回結果，加載不同的模塊。

### 注意點

`import()`加載模塊成功以後，這個模塊會作為一個物件，當作`then`方法的參數。因此，可以使用物件解構賦值的語法，獲取輸出接口。

```javascript
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```

上面代碼中，`export1`和`export2`都是`myModule.js`的輸出接口，可以解構獲得。

如果模塊有`default`輸出接口，可以用參數直接獲得。

```javascript
import('./myModule.js')
.then(myModule => {
  console.log(myModule.default);
});
```

上面的代碼也可以使用具名輸入的形式。

```javascript
import('./myModule.js')
.then(({default: theDefault}) => {
  console.log(theDefault);
});
```

如果想同時加載多個模塊，可以採用下面的寫法。

```javascript
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

`import()`也可以用在 async 函數之中。

```javascript
async function main() {
  const myModule = await import('./myModule.js');
  const {export1, export2} = await import('./myModule.js');
  const [module1, module2, module3] =
    await Promise.all([
      import('./module1.js'),
      import('./module2.js'),
      import('./module3.js'),
    ]);
}
main();
```
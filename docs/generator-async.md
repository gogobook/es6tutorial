# Generator 函數的異步應用

異步編程對 JavaScript 語言太重要。Javascript 語言的執行環境是“單線程”的，如果沒有異步編程，根本沒法用，非卡死不可。本章主要介紹 Generator 函數如何完成異步操作。

## 傳統方法

ES6 誕生以前，異步編程的方法，大概有下面四種。

- 回調函數
- 事件監聽
- 發佈/訂閱
- Promise 物件

Generator 函數將 JavaScript 異步編程帶入了一個全新的階段。

## 基本概念

### 異步

所謂"異步"，簡單說就是一個任務不是連續完成的，可以理解成該任務被人為分成兩段，先執行第一段，然後轉而執行其他任務，等做好了準備，再回過頭執行第二段。

比如，有一個任務是讀取文件進行處理，任務的第一段是向操作系統發出請求，要求讀取文件。然後，程序執行其他任務，等到操作系統返回文件，再接著執行任務的第二段（處理文件）。這種不連續的執行，就叫做異步。

相應地，連續的執行就叫做同步。由於是連續執行，不能插入其他任務，所以操作系統從硬盤讀取文件的這段時間，程序只能幹等著。

### 回調函數

JavaScript 語言對異步編程的實現，就是回調函數。所謂回調函數，就是把任務的第二段單獨寫在一個函數裡面，等到重新執行這個任務的時候，就直接調用這個函數。回調函數的英語名字`callback`，直譯過來就是"重新調用"。

讀取文件進行處理，是這樣寫的。

```javascript
fs.readFile('/etc/passwd', 'utf-8', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

上面代碼中，`readFile`函數的第三個參數，就是回調函數，也就是任務的第二段。等到操作系統返回了`/etc/passwd`這個文件以後，回調函數才會執行。

一個有趣的問題是，為什麼 Node 約定，回調函數的第一個參數，必須是錯誤物件`err`（如果沒有錯誤，該參數就是`null`）？

原因是執行分成兩段，第一段執行完以後，任務所在的上下文環境就已經結束了。在這以後拋出的錯誤，原來的上下文環境已經無法捕捉，只能當作參數，傳入第二段。

### Promise

回調函數本身並沒有問題，它的問題出現在多個回調函數嵌套。假定讀取`A`文件之後，再讀取`B`文件，代碼如下。

```javascript
fs.readFile(fileA, 'utf-8', function (err, data) {
  fs.readFile(fileB, 'utf-8', function (err, data) {
    // ...
  });
});
```

不難想像，如果依次讀取兩個以上的文件，就會出現多重嵌套。代碼不是縱向發展，而是橫向發展，很快就會亂成一團，無法管理。因為多個異步操作形成了強耦合，只要有一個操作需要修改，它的上層回調函數和下層回調函數，可能都要跟著修改。這種情況就稱為"回調函數地獄"（callback hell）。

Promise 物件就是為瞭解決這個問題而提出的。它不是新的語法功能，而是一種新的寫法，允許將回調函數的嵌套，改成鏈式調用。採用 Promise，連續讀取多個文件，寫法如下。

```javascript
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});
```

上面代碼中，我使用了`fs-readfile-promise`模塊，它的作用就是返回一個 Promise 版本的`readFile`函數。Promise 提供`then`方法加載回調函數，`catch`方法捕捉執行過程中拋出的錯誤。

可以看到，Promise 的寫法只是回調函數的改進，使用`then`方法以後，異步任務的兩段執行看得更清楚了，除此以外，並無新意。

Promise 的最大問題是代碼冗餘，原來的任務被 Promise 包裝了一下，不管什麼操作，一眼看去都是一堆`then`，原來的語義變得很不清楚。

那麼，有沒有更好的寫法呢？

## Generator 函數

### 協程

傳統的編程語言，早有異步編程的解決方案（其實是多任務的解決方案）。其中有一種叫做"協程"（coroutine），意思是多個線程互相協作，完成異步任務。

協程有點像函數，又有點像線程。它的運行流程大致如下。

- 第一步，協程`A`開始執行。
- 第二步，協程`A`執行到一半，進入暫停，執行權轉移到協程`B`。
- 第三步，（一段時間後）協程`B`交還執行權。
- 第四步，協程`A`恢復執行。

上面流程的協程`A`，就是異步任務，因為它分成兩段（或多段）執行。

舉例來說，讀取文件的協程寫法如下。

```javascript
function* asyncJob() {
  // ...其他代碼
  var f = yield readFile(fileA);
  // ...其他代碼
}
```

上面代碼的函數`asyncJob`是一個協程，它的奧妙就在其中的`yield`命令。它表示執行到此處，執行權將交給其他協程。也就是說，`yield`命令是異步兩個階段的分界線。

協程遇到`yield`命令就暫停，等到執行權返回，再從暫停的地方繼續往後執行。它的最大優點，就是代碼的寫法非常像同步操作，如果去除`yield`命令，簡直一模一樣。

### 協程的 Generator 函數實現

Generator 函數是協程在 ES6 的實現，最大特點就是可以交出函數的執行權（即暫停執行）。

整個 Generator 函數就是一個封裝的異步任務，或者說是異步任務的容器。異步操作需要暫停的地方，都用`yield`語句註明。Generator 函數的執行方法如下。

```javascript
function* gen(x) {
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next() // { value: 3, done: false }
g.next() // { value: undefined, done: true }
```

上面代碼中，調用 Generator 函數，會返回一個內部指針（即遍歷器）`g`。這是 Generator 函數不同於普通函數的另一個地方，即執行它不會返回結果，返回的是指針物件。調用指針`g`的`next`方法，會移動內部指針（即執行異步任務的第一段），指向第一個遇到的`yield`語句，上例是執行到`x + 2`為止。

換言之，`next`方法的作用是分階段執行`Generator`函數。每次調用`next`方法，會返回一個物件，表示當前階段的信息（`value`屬性和`done`屬性）。`value`屬性是`yield`語句後面表達式的值，表示當前階段的值；`done`屬性是一個布爾值，表示 Generator 函數是否執行完畢，即是否還有下一個階段。

### Generator 函數的數據交換和錯誤處理

Generator 函數可以暫停執行和恢復執行，這是它能封裝異步任務的根本原因。除此之外，它還有兩個特性，使它可以作為異步編程的完整解決方案：函數體內外的數據交換和錯誤處理機制。

`next`返回值的 value 屬性，是 Generator 函數向外輸出數據；`next`方法還可以接受參數，向 Generator 函數體內輸入數據。

```javascript
function* gen(x){
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next() // { value: 3, done: false }
g.next(2) // { value: 2, done: true }
```

上面代碼中，第一個`next`方法的`value`屬性，返回表達式`x + 2`的值`3`。第二個`next`方法帶有參數`2`，這個參數可以傳入 Generator 函數，作為上個階段異步任務的返回結果，被函數體內的變數`y`接收。因此，這一步的`value`屬性，返回的就是`2`（變數`y`的值）。

Generator 函數內部還可以部署錯誤處理代碼，捕獲函數體外拋出的錯誤。

```javascript
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
g.throw('出錯了');
// 出錯了
```

上面代碼的最後一行，Generator 函數體外，使用指針物件的`throw`方法拋出的錯誤，可以被函數體內的`try...catch`代碼塊捕獲。這意味著，出錯的代碼與處理錯誤的代碼，實現了時間和空間上的分離，這對於異步編程無疑是很重要的。

### 異步任務的封裝

下面看看如何使用 Generator 函數，執行一個真實的異步任務。

```javascript
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}
```

上面代碼中，Generator 函數封裝了一個異步操作，該操作先讀取一個遠程接口，然後從 JSON 格式的數據解析信息。就像前面說過的，這段代碼非常像同步操作，除了加上了`yield`命令。

執行這段代碼的方法如下。

```javascript
var g = gen();
var result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
```

上面代碼中，首先執行 Generator 函數，獲取遍歷器物件，然後使用`next`方法（第二行），執行異步任務的第一階段。由於`Fetch`模塊返回的是一個 Promise 物件，因此要用`then`方法調用下一個`next`方法。

可以看到，雖然 Generator 函數將異步操作表示得很簡潔，但是流程管理卻不方便（即何時執行第一階段、何時執行第二階段）。

## Thunk 函數

Thunk 函數是自動執行 Generator 函數的一種方法。

### 參數的求值策略

Thunk 函數早在上個世紀 60 年代就誕生了。

那時，編程語言剛剛起步，計算機學家還在研究，編譯器怎麼寫比較好。一個爭論的焦點是"求值策略"，即函數的參數到底應該何時求值。

```javascript
var x = 1;

function f(m) {
  return m * 2;
}

f(x + 5)
```

上面代碼先定義函數`f`，然後向它傳入表達式`x + 5`。請問，這個表達式應該何時求值？

一種意見是"傳值調用"（call by value），即在進入函數體之前，就計算`x + 5`的值（等於 6），再將這個值傳入函數`f`。C 語言就採用這種策略。

```javascript
f(x + 5)
// 傳值調用時，等同於
f(6)
```

另一種意見是“傳名調用”（call by name），即直接將表達式`x + 5`傳入函數體，只在用到它的時候求值。Haskell 語言採用這種策略。

```javascript
f(x + 5)
// 傳名調用時，等同於
(x + 5) * 2
```

傳值調用和傳名調用，哪一種比較好？

回答是各有利弊。傳值調用比較簡單，但是對參數求值的時候，實際上還沒用到這個參數，有可能造成性能損失。

```javascript
function f(a, b){
  return b;
}

f(3 * x * x - 2 * x - 1, x);
```

上面代碼中，函數`f`的第一個參數是一個複雜的表達式，但是函數體內根本沒用到。對這個參數求值，實際上是不必要的。因此，有一些計算機學家傾向於"傳名調用"，即只在執行時求值。

### Thunk 函數的含義

編譯器的“傳名調用”實現，往往是將參數放到一個臨時函數之中，再將這個臨時函數傳入函數體。這個臨時函數就叫做 Thunk 函數。

```javascript
function f(m) {
  return m * 2;
}

f(x + 5);

// 等同於

var thunk = function () {
  return x + 5;
};

function f(thunk) {
  return thunk() * 2;
}
```

上面代碼中，函數 f 的參數`x + 5`被一個函數替換了。凡是用到原參數的地方，對`Thunk`函數求值即可。

這就是 Thunk 函數的定義，它是“傳名調用”的一種實現策略，用來替換某個表達式。

### JavaScript 語言的 Thunk 函數

JavaScript 語言是傳值調用，它的 Thunk 函數含義有所不同。在 JavaScript 語言中，Thunk 函數替換的不是表達式，而是多參數函數，將其替換成一個只接受回調函數作為參數的單參數函數。

```javascript
// 正常版本的readFile（多參數版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（單參數版本）
var Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

上面代碼中，`fs`模塊的`readFile`方法是一個多參數函數，兩個參數分別為文件名和回調函數。經過轉換器處理，它變成了一個單參數函數，只接受回調函數作為參數。這個單參數版本，就叫做 Thunk 函數。

任何函數，只要參數有回調函數，就能寫成 Thunk 函數的形式。下面是一個簡單的 Thunk 函數轉換器。

```javascript
// ES5版本
var Thunk = function(fn){
  return function (){
    var args = Array.prototype.slice.call(arguments);
    return function (callback){
      args.push(callback);
      return fn.apply(this, args);
    }
  };
};

// ES6版本
const Thunk = function(fn) {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback);
    }
  };
};
```

使用上面的轉換器，生成`fs.readFile`的 Thunk 函數。

```javascript
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```

下面是另一個完整的例子。

```javascript
function f(a, cb) {
  cb(a);
}
const ft = Thunk(f);

ft(1)(console.log) // 1
```

### Thunkify 模塊

生產環境的轉換器，建議使用 Thunkify 模塊。

首先是安裝。

```bash
$ npm install thunkify
```

使用方式如下。

```javascript
var thunkify = require('thunkify');
var fs = require('fs');

var read = thunkify(fs.readFile);
read('package.json')(function(err, str){
  // ...
});
```

Thunkify 的源碼與上一節那個簡單的轉換器非常像。

```javascript
function thunkify(fn) {
  return function() {
    var args = new Array(arguments.length);
    var ctx = this;

    for (var i = 0; i < args.length; ++i) {
      args[i] = arguments[i];
    }

    return function (done) {
      var called;

      args.push(function () {
        if (called) return;
        called = true;
        done.apply(null, arguments);
      });

      try {
        fn.apply(ctx, args);
      } catch (err) {
        done(err);
      }
    }
  }
};
```

它的源碼主要多了一個檢查機制，變數`called`確保回調函數只運行一次。這樣的設計與下文的 Generator 函數相關。請看下面的例子。

```javascript
function f(a, b, callback){
  var sum = a + b;
  callback(sum);
  callback(sum);
}

var ft = thunkify(f);
var print = console.log.bind(console);
ft(1, 2)(print);
// 3
```

上面代碼中，由於`thunkify`只允許回調函數執行一次，所以只輸出一行結果。

### Generator 函數的流程管理

你可能會問， Thunk 函數有什麼用？回答是以前確實沒什麼用，但是 ES6 有了 Generator 函數，Thunk 函數現在可以用於 Generator 函數的自動流程管理。

Generator 函數可以自動執行。

```javascript
function* gen() {
  // ...
}

var g = gen();
var res = g.next();

while(!res.done){
  console.log(res.value);
  res = g.next();
}
```

上面代碼中，Generator 函數`gen`會自動執行完所有步驟。

但是，這不適合異步操作。如果必須保證前一步執行完，才能執行後一步，上面的自動執行就不可行。這時，Thunk 函數就能派上用處。以讀取文件為例。下面的 Generator 函數封裝了兩個異步操作。

```javascript
var fs = require('fs');
var thunkify = require('thunkify');
var readFileThunk = thunkify(fs.readFile);

var gen = function* (){
  var r1 = yield readFileThunk('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFileThunk('/etc/shells');
  console.log(r2.toString());
};
```

上面代碼中，`yield`命令用於將程序的執行權移出 Generator 函數，那麼就需要一種方法，將執行權再交還給 Generator 函數。

這種方法就是 Thunk 函數，因為它可以在回調函數裡，將執行權交還給 Generator 函數。為了便於理解，我們先看如何手動執行上面這個 Generator 函數。

```javascript
var g = gen();

var r1 = g.next();
r1.value(function (err, data) {
  if (err) throw err;
  var r2 = g.next(data);
  r2.value(function (err, data) {
    if (err) throw err;
    g.next(data);
  });
});
```

上面代碼中，變數`g`是 Generator 函數的內部指針，表示目前執行到哪一步。`next`方法負責將指針移動到下一步，並返回該步的信息（`value`屬性和`done`屬性）。

仔細查看上面的代碼，可以發現 Generator 函數的執行過程，其實是將同一個回調函數，反覆傳入`next`方法的`value`屬性。這使得我們可以用遞歸來自動完成這個過程。

### Thunk 函數的自動流程管理

Thunk 函數真正的威力，在於可以自動執行 Generator 函數。下面就是一個基於 Thunk 函數的 Generator 執行器。

```javascript
function run(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }

  next();
}

function* g() {
  // ...
}

run(g);
```

上面代碼的`run`函數，就是一個 Generator 函數的自動執行器。內部的`next`函數就是 Thunk 的回調函數。`next`函數先將指針移到 Generator 函數的下一步（`gen.next`方法），然後判斷 Generator 函數是否結束（`result.done`屬性），如果沒結束，就將`next`函數再傳入 Thunk 函數（`result.value`屬性），否則就直接退出。

有了這個執行器，執行 Generator 函數方便多了。不管內部有多少個異步操作，直接把 Generator 函數傳入`run`函數即可。當然，前提是每一個異步操作，都要是 Thunk 函數，也就是說，跟在`yield`命令後面的必須是 Thunk 函數。

```javascript
var g = function* (){
  var f1 = yield readFileThunk('fileA');
  var f2 = yield readFileThunk('fileB');
  // ...
  var fn = yield readFileThunk('fileN');
};

run(g);
```

上面代碼中，函數`g`封裝了`n`個異步的讀取文件操作，只要執行`run`函數，這些操作就會自動完成。這樣一來，異步操作不僅可以寫得像同步操作，而且一行代碼就可以執行。

Thunk 函數並不是 Generator 函數自動執行的唯一方案。因為自動執行的關鍵是，必須有一種機制，自動控制 Generator 函數的流程，接收和交還程序的執行權。回調函數可以做到這一點，Promise 物件也可以做到這一點。

## co 模塊

### 基本用法

[co 模塊](https://github.com/tj/co)是著名程序員 TJ Holowaychuk 於 2013 年 6 月發佈的一個小工具，用於 Generator 函數的自動執行。

下面是一個 Generator 函數，用於依次讀取兩個文件。

```javascript
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

co 模塊可以讓你不用編寫 Generator 函數的執行器。

```javascript
var co = require('co');
co(gen);
```

上面代碼中，Generator 函數只要傳入`co`函數，就會自動執行。

`co`函數返回一個`Promise`物件，因此可以用`then`方法添加回調函數。

```javascript
co(gen).then(function (){
  console.log('Generator 函數執行完成');
});
```

上面代碼中，等到 Generator 函數執行結束，就會輸出一行提示。

### co 模塊的原理

為什麼 co 可以自動執行 Generator 函數？

前面說過，Generator 就是一個異步操作的容器。它的自動執行需要一種機制，當異步操作有了結果，能夠自動交回執行權。

兩種方法可以做到這一點。

（1）回調函數。將異步操作包裝成 Thunk 函數，在回調函數裡面交回執行權。

（2）Promise 物件。將異步操作包裝成 Promise 物件，用`then`方法交回執行權。

co 模塊其實就是將兩種自動執行器（Thunk 函數和 Promise 物件），包裝成一個模塊。使用 co 的前提條件是，Generator 函數的`yield`命令後面，只能是 Thunk 函數或 Promise 物件。如果陣列或物件的成員，全部都是 Promise 物件，也可以使用 co，詳見後文的例子。

上一節已經介紹了基於 Thunk 函數的自動執行器。下面來看，基於 Promise 物件的自動執行器。這是理解 co 模塊必須的。

### 基於 Promise 物件的自動執行

還是沿用上面的例子。首先，把`fs`模塊的`readFile`方法包裝成一個 Promise 物件。

```javascript
var fs = require('fs');

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

然後，手動執行上面的 Generator 函數。

```javascript
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
});
```

手動執行其實就是用`then`方法，層層添加回調函數。理解了這一點，就可以寫出一個自動執行器。

```javascript
function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

上面代碼中，只要 Generator 函數還沒執行到最後一步，`next`函數就調用自身，以此實現自動執行。

### co 模塊的源碼

co 就是上面那個自動執行器的擴展，它的源碼只有幾十行，非常簡單。

首先，co 函數接受 Generator 函數作為參數，返回一個 Promise 物件。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
  });
}
```

在返回的 Promise 物件裡面，co 先檢查參數`gen`是否為 Generator 函數。如果是，就執行該函數，得到一個內部指針物件；如果不是就返回，並將 Promise 物件的狀態改為`resolved`。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);
  });
}
```

接著，co 將 Generator 函數的內部指針物件的`next`方法，包裝成`onFulfilled`函數。這主要是為了能夠捕捉拋出的錯誤。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled();
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }
  });
}
```

最後，就是關鍵的`next`函數，它會反覆調用自身。

```javascript
function next(ret) {
  if (ret.done) return resolve(ret.value);
  var value = toPromise.call(ctx, ret.value);
  if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
  return onRejected(
    new TypeError(
      'You may only yield a function, promise, generator, array, or object, '
      + 'but the following object was passed: "'
      + String(ret.value)
      + '"'
    )
  );
}
```

上面代碼中，`next`函數的內部代碼，一共只有四行命令。

第一行，檢查當前是否為 Generator 函數的最後一步，如果是就返回。

第二行，確保每一步的返回值，是 Promise 物件。

第三行，使用`then`方法，為返回值加上回調函數，然後通過`onFulfilled`函數再次調用`next`函數。

第四行，在參數不符合要求的情況下（參數非 Thunk 函數和 Promise 物件），將 Promise 物件的狀態改為`rejected`，從而終止執行。

### 處理並發的異步操作

co 支持並發的異步操作，即允許某些操作同時進行，等到它們全部完成，才進行下一步。

這時，要把並發的操作都放在陣列或物件裡面，跟在`yield`語句後面。

```javascript
// 陣列的寫法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 物件的寫法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res);
}).catch(onerror);
```

下面是另一個例子。

```javascript
co(function* () {
  var values = [n1, n2, n3];
  yield values.map(somethingAsync);
});

function* somethingAsync(x) {
  // do something async
  return y
}
```

上面的代碼允許並發三個`somethingAsync`異步操作，等到它們全部完成，才會進行下一步。

### 實例：處理 Stream

Node 提供 Stream 模式讀寫數據，特點是一次只處理數據的一部分，數據分成一塊塊依次處理，就好像“數據流”一樣。這對於處理大規模數據非常有利。Stream 模式使用 EventEmitter API，會釋放三個事件。

- `data`事件：下一塊數據塊已經準備好了。
- `end`事件：整個“數據流”處理“完了。
- `error`事件：發生錯誤。

使用`Promise.race()`函數，可以判斷這三個事件之中哪一個最先發生，只有當`data`事件最先發生時，才進入下一個數據塊的處理。從而，我們可以通過一個`while`循環，完成所有數據的讀取。

```javascript
const co = require('co');
const fs = require('fs');

const stream = fs.createReadStream('./les_miserables.txt');
let valjeanCount = 0;

co(function*() {
  while(true) {
    const res = yield Promise.race([
      new Promise(resolve => stream.once('data', resolve)),
      new Promise(resolve => stream.once('end', resolve)),
      new Promise((resolve, reject) => stream.once('error', reject))
    ]);
    if (!res) {
      break;
    }
    stream.removeAllListeners('data');
    stream.removeAllListeners('end');
    stream.removeAllListeners('error');
    valjeanCount += (res.toString().match(/valjean/ig) || []).length;
  }
  console.log('count:', valjeanCount); // count: 1120
});
```

上面代碼採用 Stream 模式讀取《悲慘世界》的文本文件，對於每個數據塊都使用`stream.once`方法，在`data`、`end`、`error`三個事件上添加一次性回調函數。變數`res`只有在`data`事件發生時才有值，然後累加每個數據塊之中`valjean`這個詞出現的次數。
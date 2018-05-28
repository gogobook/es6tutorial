# async 函數

## 含義

ES2017 標準引入了 async 函數，使得異步操作變得更加方便。

async 函數是什麼？一句話，它就是 Generator 函數的語法糖。

前文有一個 Generator 函數，依次讀取兩個文件。

```javascript
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

寫成`async`函數，就是下面這樣。

```javascript
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

一比較就會發現，`async`函數就是將 Generator 函數的星號（`*`）替換成`async`，將`yield`替換成`await`，僅此而已。

`async`函數對 Generator 函數的改進，體現在以下四點。

（1）內置執行器。

Generator 函數的執行必須靠執行器，所以才有了`co`模塊，而`async`函數自帶執行器。也就是說，`async`函數的執行，與普通函數一模一樣，只要一行。

```javascript
asyncReadFile();
```

上面的代碼調用了`asyncReadFile`函數，然後它就會自動執行，輸出最後結果。這完全不像 Generator 函數，需要調用`next`方法，或者用`co`模塊，才能真正執行，得到最後結果。

（2）更好的語義。

`async`和`await`，比起星號和`yield`，語義更清楚了。`async`表示函數裡有異步操作，`await`表示緊跟在後面的表達式需要等待結果。

（3）更廣的適用性。

`co`模塊約定，`yield`命令後面只能是 Thunk 函數或 Promise 物件，而`async`函數的`await`命令後面，可以是 Promise 物件和原始類型的值（數值、字符串和布爾值，但這時等同於同步操作）。

（4）返回值是 Promise。

`async`函數的返回值是 Promise 物件，這比 Generator 函數的返回值是 Iterator 物件方便多了。你可以用`then`方法指定下一步的操作。

進一步說，`async`函數完全可以看作多個異步操作，包裝成的一個 Promise 物件，而`await`命令就是內部`then`命令的語法糖。

## 基本用法

`async`函數返回一個 Promise 物件，可以使用`then`方法添加回調函數。當函數執行的時候，一旦遇到`await`就會先返回，等到異步操作完成，再接著執行函數體內後面的語句。

下面是一個例子。

```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

上面代碼是一個獲取股票報價的函數，函數前面的`async`關鍵字，表明該函數內部有異步操作。調用該函數時，會立即返回一個`Promise`物件。

下面是另一個例子，指定多少毫秒後輸出一個值。

```javascript
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

上面代碼指定 50 毫秒以後，輸出`hello world`。

由於`async`函數返回的是 Promise 物件，可以作為`await`命令的參數。所以，上面的例子也可以寫成下面的形式。

```javascript
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

async 函數有多種使用形式。

```javascript
// 函數聲明
async function foo() {}

// 函數表達式
const foo = async function () {};

// 物件的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭頭函數
const foo = async () => {};
```

## 語法

`async`函數的語法規則總體上比較簡單，難點是錯誤處理機制。

### 返回 Promise 物件

`async`函數返回一個 Promise 物件。

`async`函數內部`return`語句返回的值，會成為`then`方法回調函數的參數。

```javascript
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

上面代碼中，函數`f`內部`return`命令返回的值，會被`then`方法回調函數接收到。

`async`函數內部拋出錯誤，會導致返回的 Promise 物件變為`reject`狀態。拋出的錯誤物件會被`catch`方法回調函數接收到。

```javascript
async function f() {
  throw new Error('出錯了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出錯了
```

### Promise 物件的狀態變化

`async`函數返回的 Promise 物件，必須等到內部所有`await`命令後面的 Promise 物件執行完，才會發生狀態改變，除非遇到`return`語句或者拋出錯誤。也就是說，只有`async`函數內部的異步操作執行完，才會執行`then`方法指定的回調函數。

下面是一個例子。

```javascript
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

上面代碼中，函數`getTitle`內部有三個操作：抓取網頁、取出文本、匹配頁面標題。只有這三個操作全部完成，才會執行`then`方法裡面的`console.log`。

### await 命令

正常情況下，`await`命令後面是一個 Promise 物件。如果不是，會被轉成一個立即`resolve`的 Promise 物件。

```javascript
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123
```

上面代碼中，`await`命令的參數是數值`123`，它被轉成 Promise 物件，並立即`resolve`。

`await`命令後面的 Promise 物件如果變為`reject`狀態，則`reject`的參數會被`catch`方法的回調函數接收到。

```javascript
async function f() {
  await Promise.reject('出錯了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出錯了
```

注意，上面代碼中，`await`語句前面沒有`return`，但是`reject`方法的參數依然傳入了`catch`方法的回調函數。這裡如果在`await`前面加上`return`，效果是一樣的。

只要一個`await`語句後面的 Promise 變為`reject`，那麼整個`async`函數都會中斷執行。

```javascript
async function f() {
  await Promise.reject('出錯了');
  await Promise.resolve('hello world'); // 不會執行
}
```

上面代碼中，第二個`await`語句是不會執行的，因為第一個`await`語句狀態變成了`reject`。

有時，我們希望即使前一個異步操作失敗，也不要中斷後面的異步操作。這時可以將第一個`await`放在`try...catch`結構裡面，這樣不管這個異步操作是否成功，第二個`await`都會執行。

```javascript
async function f() {
  try {
    await Promise.reject('出錯了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
```

另一種方法是`await`後面的 Promise 物件再跟一個`catch`方法，處理前面可能出現的錯誤。

```javascript
async function f() {
  await Promise.reject('出錯了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出錯了
// hello world
```

### 錯誤處理

如果`await`後面的異步操作出錯，那麼等同於`async`函數返回的 Promise 物件被`reject`。

```javascript
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出錯了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出錯了
```

上面代碼中，`async`函數`f`執行後，`await`後面的 Promise 物件會拋出一個錯誤物件，導致`catch`方法的回調函數被調用，它的參數就是拋出的錯誤物件。具體的執行機制，可以參考後文的“async 函數的實現原理”。

防止出錯的方法，也是將其放在`try...catch`代碼塊之中。

```javascript
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出錯了');
    });
  } catch(e) {
  }
  return await('hello world');
}
```

如果有多個`await`命令，可以統一放在`try...catch`結構中。

```javascript
async function main() {
  try {
    const val1 = await firstStep();
    const val2 = await secondStep(val1);
    const val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
```

下面的例子使用`try...catch`結構，實現多次重複嘗試。

```javascript
const superagent = require('superagent');
const NUM_RETRIES = 3;

async function test() {
  let i;
  for (i = 0; i < NUM_RETRIES; ++i) {
    try {
      await superagent.get('http://google.com/this-throws-an-error');
      break;
    } catch(err) {}
  }
  console.log(i); // 3
}

test();
```

上面代碼中，如果`await`操作成功，就會使用`break`語句退出循環；如果失敗，會被`catch`語句捕捉，然後進入下一輪循環。

### 使用注意點

第一點，前面已經說過，`await`命令後面的`Promise`物件，運行結果可能是`rejected`，所以最好把`await`命令放在`try...catch`代碼塊中。

```javascript
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一種寫法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二點，多個`await`命令後面的異步操作，如果不存在繼發關係，最好讓它們同時觸發。

```javascript
let foo = await getFoo();
let bar = await getBar();
```

上面代碼中，`getFoo`和`getBar`是兩個獨立的異步操作（即互不依賴），被寫成繼發關係。這樣比較耗時，因為只有`getFoo`完成以後，才會執行`getBar`，完全可以讓它們同時觸發。

```javascript
// 寫法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 寫法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

上面兩種寫法，`getFoo`和`getBar`都是同時觸發，這樣就會縮短程序的執行時間。

第三點，`await`命令只能用在`async`函數之中，如果用在普通函數，就會報錯。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 報錯
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```

上面代碼會報錯，因為`await`用在普通函數之中了。但是，如果將`forEach`方法的參數改成`async`函數，也有問題。

```javascript
function dbFuc(db) { //這裡不需要 async
  let docs = [{}, {}, {}];

  // 可能得到錯誤結果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
```

上面代碼可能不會正常工作，原因是這時三個`db.post`操作將是並發執行，也就是同時執行，而不是繼發執行。正確的寫法是採用`for`循環。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```

如果確實希望多個請求並發執行，可以使用`Promise.all`方法。當三個請求都會`resolved`時，下面兩種寫法效果相同。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的寫法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

目前，[`@std/esm`](https://www.npmjs.com/package/@std/esm)模塊加載器支持頂層`await`，即`await`命令可以不放在 async 函數裡面，直接使用。

```javascript
// async 函數的寫法
const start = async () => {
  const res = await fetch('google.com');
  return res.text();
};

start().then(console.log);

// 頂層 await 的寫法
const res = await fetch('google.com');
console.log(await res.text());
```

上面代碼中，第二種寫法的腳本必須使用`@std/esm`加載器，才會生效。

## async 函數的實現原理

async 函數的實現原理，就是將 Generator 函數和自動執行器，包裝在一個函數裡。

```javascript
async function fn(args) {
  // ...
}

// 等同於

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
```

所有的`async`函數都可以寫成上面的第二種形式，其中的`spawn`函數就是自動執行器。

下面給出`spawn`函數的實現，基本就是前文自動執行器的翻版。

```javascript
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```

## 與其他異步處理方法的比較

我們通過一個例子，來看 async 函數與 Promise、Generator 函數的比較。

假定某個 DOM 元素上面，部署了一系列的動畫，前一個動畫結束，才能開始後一個。如果當中有一個動畫出錯，就不再往下執行，返回上一個成功執行的動畫的返回值。

首先是 Promise 的寫法。

```javascript
function chainAnimationsPromise(elem, animations) {

  // 變數ret用來保存上一個動畫的返回值
  let ret = null;

  // 新建一個空的Promise
  let p = Promise.resolve();

  // 使用then方法，添加所有動畫
  for(let anim of animations) {
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    });
  }

  // 返回一個部署了錯誤捕捉機制的Promise
  return p.catch(function(e) {
    /* 忽略錯誤，繼續執行 */
  }).then(function() {
    return ret;
  });

}
```

雖然 Promise 的寫法比回調函數的寫法大大改進，但是一眼看上去，代碼完全都是 Promise 的 API（`then`、`catch`等等），操作本身的語義反而不容易看出來。

接著是 Generator 函數的寫法。

```javascript
function chainAnimationsGenerator(elem, animations) {

  return spawn(function*() {
    let ret = null;
    try {
      for(let anim of animations) {
        ret = yield anim(elem);
      }
    } catch(e) {
      /* 忽略錯誤，繼續執行 */
    }
    return ret;
  });

}
```

上面代碼使用 Generator 函數遍歷了每個動畫，語義比 Promise 寫法更清晰，用戶定義的操作全部都出現在`spawn`函數的內部。這個寫法的問題在於，必須有一個任務運行器，自動執行 Generator 函數，上面代碼的`spawn`函數就是自動執行器，它返回一個 Promise 物件，而且必須保證`yield`語句後面的表達式，必須返回一個 Promise。

最後是 async 函數的寫法。

```javascript
async function chainAnimationsAsync(elem, animations) {
  let ret = null;
  try {
    for(let anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) {
    /* 忽略錯誤，繼續執行 */
  }
  return ret;
}
```

可以看到 Async 函數的實現最簡潔，最符合語義，幾乎沒有語義不相關的代碼。它將 Generator 寫法中的自動執行器，改在語言層面提供，不暴露給用戶，因此代碼量最少。如果使用 Generator 寫法，自動執行器需要用戶自己提供。

## 實例：按順序完成異步操作

實際開發中，經常遇到一組異步操作，需要按照順序完成。比如，依次遠程讀取一組 URL，然後按照讀取的順序輸出結果。

Promise 的寫法如下。

```javascript
function logInOrder(urls) {
  // 遠程讀取所有URL
  const textPromises = urls.map(url => {
    return fetch(url).then(response => response.text());
  });

  // 按次序輸出
  textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}
```

上面代碼使用`fetch`方法，同時遠程讀取一組 URL。每個`fetch`操作都返回一個 Promise 物件，放入`textPromises`陣列。然後，`reduce`方法依次處理每個 Promise 物件，然後使用`then`，將所有 Promise 物件連起來，因此就可以依次輸出結果。

這種寫法不太直觀，可讀性比較差。下面是 async 函數實現。

```javascript
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```

上面代碼確實大大簡化，問題是所有遠程操作都是繼發。只有前一個 URL 返回結果，才會去讀取下一個 URL，這樣做效率很差，非常浪費時間。我們需要的是並發發出遠程請求。

```javascript
async function logInOrder(urls) {
  // 並發讀取遠程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序輸出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```

上面代碼中，雖然`map`方法的參數是`async`函數，但它是並發執行的，因為只有`async`函數內部是繼發執行，外部不受影響。後面的`for..of`循環內部使用了`await`，因此實現了按順序輸出。

## 異步遍歷器

《遍歷器》一章說過，Iterator 接口是一種數據遍歷的協議，只要調用遍歷器物件的`next`方法，就會得到一個物件，表示當前遍歷指針所在的那個位置的信息。`next`方法返回的物件的結構是`{value, done}`，其中`value`表示當前的數據的值，`done`是一個布爾值，表示遍歷是否結束。

這裡隱含著一個規定，`next`方法必須是同步的，只要調用就必須立刻返回值。也就是說，一旦執行`next`方法，就必須同步地得到`value`和`done`這兩個屬性。如果遍歷指針正好指向同步操作，當然沒有問題，但對於異步操作，就不太合適了。目前的解決方法是，Generator 函數裡面的異步操作，返回一個 Thunk 函數或者 Promise 物件，即`value`屬性是一個 Thunk 函數或者 Promise 物件，等待以後返回真正的值，而`done`屬性則還是同步產生的。

ES2018 [引入](https://github.com/tc39/proposal-async-iteration)了”異步遍歷器“（Async Iterator），為異步操作提供原生的遍歷器接口，即`value`和`done`這兩個屬性都是異步產生。

### 異步遍歷的接口

異步遍歷器的最大的語法特點，就是調用遍歷器的`next`方法，返回的是一個 Promise 物件。

```javascript
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```

上面代碼中，`asyncIterator`是一個異步遍歷器，調用`next`方法以後，返回一個 Promise 物件。因此，可以使用`then`方法指定，這個 Promise 物件的狀態變為`resolve`以後的回調函數。回調函數的參數，則是一個具有`value`和`done`兩個屬性的物件，這個跟同步遍歷器是一樣的。

我們知道，一個物件的同步遍歷器的接口，部署在`Symbol.iterator`屬性上面。同樣地，物件的異步遍歷器接口，部署在`Symbol.asyncIterator`屬性上面。不管是什麼樣的物件，只要它的`Symbol.asyncIterator`屬性有值，就表示應該對它進行異步遍歷。

下面是一個異步遍歷器的例子。

```javascript
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
.next()
.then(iterResult1 => {
  console.log(iterResult1); // { value: 'a', done: false }
  return asyncIterator.next();
})
.then(iterResult2 => {
  console.log(iterResult2); // { value: 'b', done: false }
  return asyncIterator.next();
})
.then(iterResult3 => {
  console.log(iterResult3); // { value: undefined, done: true }
});
```

上面代碼中，異步遍歷器其實返回了兩次值。第一次調用的時候，返回一個 Promise 物件；等到 Promise 物件`resolve`了，再返回一個表示當前數據成員信息的物件。這就是說，異步遍歷器與同步遍歷器最終行為是一致的，只是會先返回 Promise 物件，作為中介。

由於異步遍歷器的`next`方法，返回的是一個 Promise 物件。因此，可以把它放在`await`命令後面。

```javascript
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}
```

上面代碼中，`next`方法用`await`處理以後，就不必使用`then`方法了。整個流程已經很接近同步處理了。

注意，異步遍歷器的`next`方法是可以連續調用的，不必等到上一步產生的 Promise 物件`resolve`以後再調用。這種情況下，`next`方法會累積起來，自動按照每一步的順序運行下去。下面是一個例子，把所有的`next`方法放在`Promise.all`方法裡面。

```javascript
const asyncGenObj = createAsyncIterable(['a', 'b']);
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncGenObj.next(), asyncGenObj.next()
]);

console.log(v1, v2); // a b
```

另一種用法是一次性調用所有的`next`方法，然後`await`最後一步操作。

```javascript
async function runner() {
  const writer = openFile('someFile.txt');
  writer.next('hello');
  writer.next('world');
  await writer.return();
}

runner();
```

### for await...of

前面介紹過，`for...of`循環用於遍歷同步的 Iterator 接口。新引入的`for await...of`循環，則是用於遍歷異步的 Iterator 接口。

```javascript
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```

上面代碼中，`createAsyncIterable()`返回一個擁有異步遍歷器接口的物件，`for...of`循環自動調用這個物件的異步遍歷器的`next`方法，會得到一個 Promise 物件。`await`用來處理這個 Promise 物件，一旦`resolve`，就把得到的值（`x`）傳入`for...of`的循環體。

`for await...of`循環的一個用途，是部署了 asyncIterable 操作的異步接口，可以直接放入這個循環。

```javascript
let body = '';

async function f() {
  for await(const data of req) body += data;
  const parsed = JSON.parse(body);
  console.log('got', parsed);
}
```

上面代碼中，`req`是一個 asyncIterable 物件，用來異步讀取數據。可以看到，使用`for await...of`循環以後，代碼會非常簡潔。

如果`next`方法返回的 Promise 物件被`reject`，`for await...of`就會報錯，要用`try...catch`捕捉。

```javascript
async function () {
  try {
    for await (const x of createRejectingIterable()) {
      console.log(x);
    }
  } catch (e) {
    console.error(e);
  }
}
```

注意，`for await...of`循環也可以用於同步遍歷器。

```javascript
(async function () {
  for await (const x of ['a', 'b']) {
    console.log(x);
  }
})();
// a
// b
```

Node v10 支持異步遍歷器，Stream 就部署了這個接口。下面是讀取文件的傳統寫法與異步遍歷器寫法的差異。

```javascript
// 傳統寫法
function main(inputFilePath) {
  const readStream = fs.createReadStream(
    inputFilePath,
    { encoding: 'utf8', highWaterMark: 1024 }
  );
  readStream.on('data', (chunk) => {
    console.log('>>> '+chunk);
  });
  readStream.on('end', () => {
    console.log('### DONE ###');
  });
}

// 異步遍歷器寫法
async function main(inputFilePath) {
  const readStream = fs.createReadStream(
    inputFilePath,
    { encoding: 'utf8', highWaterMark: 1024 }
  );

  for await (const chunk of readStream) {
    console.log('>>> '+chunk);
  }
  console.log('### DONE ###');
}
```

### 異步 Generator 函數

就像 Generator 函數返回一個同步遍歷器物件一樣，異步 Generator 函數的作用，是返回一個異步遍歷器物件。

在語法上，異步 Generator 函數就是`async`函數與 Generator 函數的結合。

```javascript
async function* gen() {
  yield 'hello';
}
const genObj = gen();
genObj.next().then(x => console.log(x));
// { value: 'hello', done: false }
```

上面代碼中，`gen`是一個異步 Generator 函數，執行後返回一個異步 Iterator 物件。對該物件調用`next`方法，返回一個 Promise 物件。

異步遍歷器的設計目的之一，就是 Generator 函數處理同步操作和異步操作時，能夠使用同一套接口。

```javascript
// 同步 Generator 函數
function* map(iterable, func) {
  const iter = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iter.next();
    if (done) break;
    yield func(value);
  }
}

// 異步 Generator 函數
async function* map(iterable, func) {
  const iter = iterable[Symbol.asyncIterator]();
  while (true) {
    const {value, done} = await iter.next();
    if (done) break;
    yield func(value);
  }
}
```

上面代碼中，`map`是一個 Generator 函數，第一個參數是可遍歷物件`iterable`，第二個參數是一個回調函數`func`。`map`的作用是將`iterable`每一步返回的值，使用`func`進行處理。上面有兩個版本的`map`，前一個處理同步遍歷器，後一個處理異步遍歷器，可以看到兩個版本的寫法基本上是一致的。

下面是另一個異步 Generator 函數的例子。

```javascript
async function* readLines(path) {
  let file = await fileOpen(path);

  try {
    while (!file.EOF) {
      yield await file.readLine();
    }
  } finally {
    await file.close();
  }
}
```

上面代碼中，異步操作前面使用`await`關鍵字標明，即`await`後面的操作，應該返回 Promise 物件。凡是使用`yield`關鍵字的地方，就是`next`方法停下來的地方，它後面的表達式的值（即`await file.readLine()`的值），會作為`next()`返回物件的`value`屬性，這一點是與同步 Generator 函數一致的。

異步 Generator 函數內部，能夠同時使用`await`和`yield`命令。可以這樣理解，`await`命令用於將外部操作產生的值輸入函數內部，`yield`命令用於將函數內部的值輸出。

上面代碼定義的異步 Generator 函數的用法如下。

```javascript
(async function () {
  for await (const line of readLines(filePath)) {
    console.log(line);
  }
})()
```

異步 Generator 函數可以與`for await...of`循環結合起來使用。

```javascript
async function* prefixLines(asyncIterable) {
  for await (const line of asyncIterable) {
    yield '> ' + line;
  }
}
```

異步 Generator 函數的返回值是一個異步 Iterator，即每次調用它的`next`方法，會返回一個 Promise 物件，也就是說，跟在`yield`命令後面的，應該是一個 Promise 物件。如果像上面那個例子那樣，`yield`命令後面是一個字符串，會被自動包裝成一個 Promise 物件。

```javascript
function fetchRandom() {
  const url = 'https://www.random.org/decimal-fractions/'
    + '?num=1&dec=10&col=1&format=plain&rnd=new';
  return fetch(url);
}

async function* asyncGenerator() {
  console.log('Start');
  const result = await fetchRandom(); // (A)
  yield 'Result: ' + await result.text(); // (B)
  console.log('Done');
}

const ag = asyncGenerator();
ag.next().then(({value, done}) => {
  console.log(value);
})
```

上面代碼中，`ag`是`asyncGenerator`函數返回的異步遍歷器物件。調用`ag.next()`以後，上面代碼的執行順序如下。

1. `ag.next()`立刻返回一個 Promise 物件。
1. `asyncGenerator`函數開始執行，打印出`Start`。
1. `await`命令返回一個 Promise 物件，`asyncGenerator`函數停在這裡。
1. A 處變成 fulfilled 狀態，產生的值放入`result`變數，`asyncGenerator`函數繼續往下執行。
1. 函數在 B 處的`yield`暫停執行，一旦`yield`命令取到值，`ag.next()`返回的那個 Promise 物件變成 fulfilled 狀態。
1. `ag.next()`後面的`then`方法指定的回調函數開始執行。該回調函數的參數是一個物件`{value, done}`，其中`value`的值是`yield`命令後面的那個表達式的值，`done`的值是`false`。

A 和 B 兩行的作用類似於下面的代碼。

```javascript
return new Promise((resolve, reject) => {
  fetchRandom()
  .then(result => result.text())
  .then(result => {
     resolve({
       value: 'Result: ' + result,
       done: false,
     });
  });
});
```

如果異步 Generator 函數拋出錯誤，會導致 Promise 物件的狀態變為`reject`，然後拋出的錯誤被`catch`方法捕獲。

```javascript
async function* asyncGenerator() {
  throw new Error('Problem!');
}

asyncGenerator()
.next()
.catch(err => console.log(err)); // Error: Problem!
```

注意，普通的 async 函數返回的是一個 Promise 物件，而異步 Generator 函數返回的是一個異步 Iterator 物件。可以這樣理解，async 函數和異步 Generator 函數，是封裝異步操作的兩種方法，都用來達到同一種目的。區別在於，前者自帶執行器，後者通過`for await...of`執行，或者自己編寫執行器。下面就是一個異步 Generator 函數的執行器。

```javascript
async function takeAsync(asyncIterable, count = Infinity) {
  const result = [];
  const iterator = asyncIterable[Symbol.asyncIterator]();
  while (result.length < count) {
    const {value, done} = await iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
}
```

上面代碼中，異步 Generator 函數產生的異步遍歷器，會通過`while`循環自動執行，每當`await iterator.next()`完成，就會進入下一輪循環。一旦`done`屬性變為`true`，就會跳出循環，異步遍歷器執行結束。

下面是這個自動執行器的一個使用實例。

```javascript
async function f() {
  async function* gen() {
    yield 'a';
    yield 'b';
    yield 'c';
  }

  return await takeAsync(gen());
}

f().then(function (result) {
  console.log(result); // ['a', 'b', 'c']
})
```

異步 Generator 函數出現以後，JavaScript 就有了四種函數形式：普通函數、async 函數、Generator 函數和異步 Generator 函數。請注意區分每種函數的不同之處。基本上，如果是一系列按照順序執行的異步操作（比如讀取文件，然後寫入新內容，再存入硬盤），可以使用 async 函數；如果是一系列產生相同數據結構的異步操作（比如一行一行讀取文件），可以使用異步 Generator 函數。

異步 Generator 函數也可以通過`next`方法的參數，接收外部傳入的數據。

```javascript
const writer = openFile('someFile.txt');
writer.next('hello'); // 立即執行
writer.next('world'); // 立即執行
await writer.return(); // 等待寫入結束
```

上面代碼中，`openFile`是一個異步 Generator 函數。`next`方法的參數，向該函數內部的操作傳入數據。每次`next`方法都是同步執行的，最後的`await`命令用於等待整個寫入操作結束。

最後，同步的數據結構，也可以使用異步 Generator 函數。

```javascript
async function* createAsyncIterable(syncIterable) {
  for (const elem of syncIterable) {
    yield elem;
  }
}
```

上面代碼中，由於沒有異步操作，所以也就沒有使用`await`關鍵字。

### yield\* 語句

`yield*`語句也可以跟一個異步遍歷器。

```javascript
async function* gen1() {
  yield 'a';
  yield 'b';
  return 2;
}

async function* gen2() {
  // result 最終會等於 2
  const result = yield* gen1();
}
```

上面代碼中，`gen2`函數裡面的`result`變數，最後的值是`2`。

與同步 Generator 函數一樣，`for await...of`循環會展開`yield*`。

```javascript
(async function () {
  for await (const x of gen2()) {
    console.log(x);
  }
})();
// a
// b
```
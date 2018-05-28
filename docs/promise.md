# Promise 物件

## Promise 的含義

Promise 是異步編程的一種解決方案，比傳統的解決方案——回調函數和事件——更合理和更強大。它由社區最早提出和實現，ES6 將其寫進了語言標準，統一了用法，原生提供了`Promise`物件。

所謂`Promise`，簡單說就是一個容器，裡面保存著某個未來才會結束的事件（通常是一個異步操作）的結果。從語法上說，Promise 是一個物件，從它可以獲取異步操作的消息。Promise 提供統一的 API，各種異步操作都可以用同樣的方法進行處理。

`Promise`物件有以下兩個特點。

（1）物件的狀態不受外界影響。`Promise`物件代表一個異步操作，有三種狀態：`pending`（進行中）、`fulfilled`（已成功）和`rejected`（已失敗）。只有異步操作的結果，可以決定當前是哪一種狀態，任何其他操作都無法改變這個狀態。這也是`Promise`這個名字的由來，它的英語意思就是“承諾”，表示其他手段無法改變。

（2）一旦狀態改變，就不會再變，任何時候都可以得到這個結果。`Promise`物件的狀態改變，只有兩種可能：從`pending`變為`fulfilled`和從`pending`變為`rejected`。只要這兩種情況發生，狀態就凝固了，不會再變了，會一直保持這個結果，這時就稱為 resolved（已定型）。如果改變已經發生了，你再對`Promise`物件添加回調函數，也會立即得到這個結果。這與事件（Event）完全不同，事件的特點是，如果你錯過了它，再去監聽，是得不到結果的。

注意，為了行文方便，本章後面的`resolved`統一隻指`fulfilled`狀態，不包含`rejected`狀態。

有了`Promise`物件，就可以將異步操作以同步操作的流程表達出來，避免了層層嵌套的回調函數。此外，`Promise`物件提供統一的接口，使得控制異步操作更加容易。

`Promise`也有一些缺點。首先，無法取消`Promise`，一旦新建它就會立即執行，無法中途取消。其次，如果不設置回調函數，`Promise`內部拋出的錯誤，不會反應到外部。第三，當處於`pending`狀態時，無法得知目前進展到哪一個階段（剛剛開始還是即將完成）。

如果某些事件不斷地反覆發生，一般來說，使用 [Stream](https://nodejs.org/api/stream.html) 模式是比部署`Promise`更好的選擇。

## 基本用法

ES6 規定，`Promise`物件是一個構造函數，用來生成`Promise`實例。

下面代碼創造了一個`Promise`實例。

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 異步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

`Promise`構造函數接受一個函數作為參數，該函數的兩個參數分別是`resolve`和`reject`。它們是兩個函數，由 JavaScript 引擎提供，不用自己部署。

`resolve`函數的作用是，將`Promise`物件的狀態從“未完成”變為“成功”（即從 pending 變為 resolved），在異步操作成功時調用，並將異步操作的結果，作為參數傳遞出去；`reject`函數的作用是，將`Promise`物件的狀態從“未完成”變為“失敗”（即從 pending 變為 rejected），在異步操作失敗時調用，並將異步操作報出的錯誤，作為參數傳遞出去。

`Promise`實例生成以後，可以用`then`方法分別指定`resolved`狀態和`rejected`狀態的回調函數。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

`then`方法可以接受兩個回調函數作為參數。第一個回調函數是`Promise`物件的狀態變為`resolved`時調用，第二個回調函數是`Promise`物件的狀態變為`rejected`時調用。其中，第二個函數是可選的，不一定要提供。這兩個函數都接受`Promise`物件傳出的值作為參數。

下面是一個`Promise`物件的簡單例子。

```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

上面代碼中，`timeout`方法返回一個`Promise`實例，表示一段時間以後才會發生的結果。過了指定的時間（`ms`參數）以後，`Promise`實例的狀態變為`resolved`，就會觸發`then`方法綁定的回調函數。

Promise 新建後就會立即執行。

```javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

上面代碼中，Promise 新建後立即執行，所以首先輸出的是`Promise`。然後，`then`方法指定的回調函數，將在當前腳本所有同步任務執行完才會執行，所以`resolved`最後輸出。

下面是異步加載圖片的例子。

```javascript
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```

上面代碼中，使用`Promise`包裝了一個圖片加載的異步操作。如果加載成功，就調用`resolve`方法，否則就調用`reject`方法。

下面是一個用`Promise`物件實現的 Ajax 操作的例子。

```javascript
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出錯了', error);
});
```

上面代碼中，`getJSON`是對 XMLHttpRequest 物件的封裝，用於發出一個針對 JSON 數據的 HTTP 請求，並且返回一個`Promise`物件。需要注意的是，在`getJSON`內部，`resolve`函數和`reject`函數調用時，都帶有參數。

如果調用`resolve`函數和`reject`函數時帶有參數，那麼它們的參數會被傳遞給回調函數。`reject`函數的參數通常是`Error`物件的實例，表示拋出的錯誤；`resolve`函數的參數除了正常的值以外，還可能是另一個 Promise 實例，比如像下面這樣。

```javascript
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```

上面代碼中，`p1`和`p2`都是 Promise 的實例，但是`p2`的`resolve`方法將`p1`作為參數，即一個異步操作的結果是返回另一個異步操作。

注意，這時`p1`的狀態就會傳遞給`p2`，也就是說，`p1`的狀態決定了`p2`的狀態。如果`p1`的狀態是`pending`，那麼`p2`的回調函數就會等待`p1`的狀態改變；如果`p1`的狀態已經是`resolved`或者`rejected`，那麼`p2`的回調函數將會立刻執行。

```javascript
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

上面代碼中，`p1`是一個 Promise，3 秒之後變為`rejected`。`p2`的狀態在 1 秒之後改變，`resolve`方法返回的是`p1`。由於`p2`返回的是另一個 Promise，導致`p2`自己的狀態無效了，由`p1`的狀態決定`p2`的狀態。所以，後面的`then`語句都變成針對後者（`p1`）。又過了 2 秒，`p1`變為`rejected`，導致觸發`catch`方法指定的回調函數。

注意，調用`resolve`或`reject`並不會終結 Promise 的參數函數的執行。

```javascript
new Promise((resolve, reject) => {
  resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 2
// 1
```

上面代碼中，調用`resolve(1)`以後，後面的`console.log(2)`還是會執行，並且會首先打印出來。這是因為立即 resolved 的 Promise 是在本輪事件循環的末尾執行，總是晚於本輪循環的同步任務。

一般來說，調用`resolve`或`reject`以後，Promise 的使命就完成了，後繼操作應該放到`then`方法裡面，而不應該直接寫在`resolve`或`reject`的後面。所以，最好在它們前面加上`return`語句，這樣就不會有意外。

```javascript
new Promise((resolve, reject) => {
  return resolve(1);
  // 後面的語句不會執行
  console.log(2);
})
```

## Promise.prototype.then()

Promise 實例具有`then`方法，也就是說，`then`方法是定義在原型物件`Promise.prototype`上的。它的作用是為 Promise 實例添加狀態改變時的回調函數。前面說過，`then`方法的第一個參數是`resolved`狀態的回調函數，第二個參數（可選）是`rejected`狀態的回調函數。

`then`方法返回的是一個新的`Promise`實例（注意，不是原來那個`Promise`實例）。因此可以採用鏈式寫法，即`then`方法後面再調用另一個`then`方法。

```javascript
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```

上面的代碼使用`then`方法，依次指定了兩個回調函數。第一個回調函數完成以後，會將返回結果作為參數，傳入第二個回調函數。

採用鏈式的`then`，可以指定一組按照次序調用的回調函數。這時，前一個回調函數，有可能返回的還是一個`Promise`物件（即有異步操作），這時後一個回調函數，就會等待該`Promise`物件的狀態發生變化，才會被調用。

```javascript
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("resolved: ", comments);
}, function funcB(err){
  console.log("rejected: ", err);
});
```

上面代碼中，第一個`then`方法指定的回調函數，返回的是另一個`Promise`物件。這時，第二個`then`方法指定的回調函數，就會等待這個新的`Promise`物件狀態發生變化。如果變為`resolved`，就調用`funcA`，如果狀態變為`rejected`，就調用`funcB`。

如果採用箭頭函數，上面的代碼可以寫得更簡潔。

```javascript
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("resolved: ", comments),
  err => console.log("rejected: ", err)
);
```

## Promise.prototype.catch()

`Promise.prototype.catch`方法是`.then(null, rejection)`的別名，用於指定發生錯誤時的回調函數。

```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 處理 getJSON 和 前一個回調函數運行時發生的錯誤
  console.log('發生錯誤！', error);
});
```

上面代碼中，`getJSON`方法返回一個 Promise 物件，如果該物件狀態變為`resolved`，則會調用`then`方法指定的回調函數；如果異步操作拋出錯誤，狀態就會變為`rejected`，就會調用`catch`方法指定的回調函數，處理這個錯誤。另外，`then`方法指定的回調函數，如果運行中拋出錯誤，也會被`catch`方法捕獲。

```javascript
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同於
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
```

下面是一個例子。

```javascript
const promise = new Promise(function(resolve, reject) {
  throw new Error('test');
});
promise.catch(function(error) {
  console.log(error);
});
// Error: test
```

上面代碼中，`promise`拋出一個錯誤，就被`catch`方法指定的回調函數捕獲。注意，上面的寫法與下面兩種寫法是等價的。

```javascript
// 寫法一
const promise = new Promise(function(resolve, reject) {
  try {
    throw new Error('test');
  } catch(e) {
    reject(e);
  }
});
promise.catch(function(error) {
  console.log(error);
});

// 寫法二
const promise = new Promise(function(resolve, reject) {
  reject(new Error('test'));
});
promise.catch(function(error) {
  console.log(error);
});
```

比較上面兩種寫法，可以發現`reject`方法的作用，等同於拋出錯誤。

如果 Promise 狀態已經變成`resolved`，再拋出錯誤是無效的。

```javascript
const promise = new Promise(function(resolve, reject) {
  resolve('ok');
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });
// ok
```

上面代碼中，Promise 在`resolve`語句後面，再拋出錯誤，不會被捕獲，等於沒有拋出。因為 Promise 的狀態一旦改變，就永久保持該狀態，不會再變了。

Promise 物件的錯誤具有“冒泡”性質，會一直向後傳遞，直到被捕獲為止。也就是說，錯誤總是會被下一個`catch`語句捕獲。

```javascript
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 處理前面三個Promise產生的錯誤
});
```

上面代碼中，一共有三個 Promise 物件：一個由`getJSON`產生，兩個由`then`產生。它們之中任何一個拋出的錯誤，都會被最後一個`catch`捕獲。

一般來說，不要在`then`方法裡面定義 Reject 狀態的回調函數（即`then`的第二個參數），總是使用`catch`方法。

```javascript
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

上面代碼中，第二種寫法要好於第一種寫法，理由是第二種寫法可以捕獲前面`then`方法執行中的錯誤，也更接近同步的寫法（`try/catch`）。因此，建議總是使用`catch`方法，而不使用`then`方法的第二個參數。

跟傳統的`try/catch`代碼塊不同的是，如果沒有使用`catch`方法指定錯誤處理的回調函數，Promise 物件拋出的錯誤不會傳遞到外層代碼，即不會有任何反應。

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行會報錯，因為x沒有聲明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  console.log('everything is great');
});

setTimeout(() => { console.log(123) }, 2000);
// Uncaught (in promise) ReferenceError: x is not defined
// 123
```

上面代碼中，`someAsyncThing`函數產生的 Promise 物件，內部有語法錯誤。瀏覽器運行到這一行，會打印出錯誤提示`ReferenceError: x is not defined`，但是不會退出進程、終止腳本執行，2 秒之後還是會輸出`123`。這就是說，Promise 內部的錯誤不會影響到 Promise 外部的代碼，通俗的說法就是“Promise 會吃掉錯誤”。

這個腳本放在服務器執行，退出碼就是`0`（即表示執行成功）。不過，Node 有一個`unhandledRejection`事件，專門監聽未捕獲的`reject`錯誤，上面的腳本會觸發這個事件的監聽函數，可以在監聽函數裡面拋出錯誤。

```javascript
process.on('unhandledRejection', function (err, p) {
  throw err;
});
```

上面代碼中，`unhandledRejection`事件的監聽函數有兩個參數，第一個是錯誤物件，第二個是報錯的 Promise 實例，它可以用來瞭解發生錯誤的環境信息。

注意，Node 有計畫在未來廢除`unhandledRejection`事件。如果 Promise 內部有未捕獲的錯誤，會直接終止進程，並且進程的退出碼不為 0。

再看下面的例子。

```javascript
const promise = new Promise(function (resolve, reject) {
  resolve('ok');
  setTimeout(function () { throw new Error('test') }, 0)
});
promise.then(function (value) { console.log(value) });
// ok
// Uncaught Error: test
```

上面代碼中，Promise 指定在下一輪“事件循環”再拋出錯誤。到了那個時候，Promise 的運行已經結束了，所以這個錯誤是在 Promise 函數體外拋出的，會冒泡到最外層，成了未捕獲的錯誤。

一般總是建議，Promise 物件後面要跟`catch`方法，這樣可以處理 Promise 內部發生的錯誤。`catch`方法返回的還是一個 Promise 物件，因此後面還可以接著調用`then`方法。

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行會報錯，因為x沒有聲明
    resolve(x + 2);
  });
};

someAsyncThing()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
// oh no [ReferenceError: x is not defined]
// carry on
```

上面代碼運行完`catch`方法指定的回調函數，會接著運行後面那個`then`方法指定的回調函數。如果沒有報錯，則會跳過`catch`方法。

```javascript
Promise.resolve()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
// carry on
```

上面的代碼因為沒有報錯，跳過了`catch`方法，直接執行後面的`then`方法。此時，要是`then`方法裡面報錯，就與前面的`catch`無關了。

`catch`方法之中，還能再拋出錯誤。

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行會報錯，因為x沒有聲明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行會報錯，因為 y 沒有聲明
  y + 2;
}).then(function() {
  console.log('carry on');
});
// oh no [ReferenceError: x is not defined]
```

上面代碼中，`catch`方法拋出一個錯誤，因為後面沒有別的`catch`方法了，導致這個錯誤不會被捕獲，也不會傳遞到外層。如果改寫一下，結果就不一樣了。

```javascript
someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行會報錯，因為y沒有聲明
  y + 2;
}).catch(function(error) {
  console.log('carry on', error);
});
// oh no [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```

上面代碼中，第二個`catch`方法用來捕獲前一個`catch`方法拋出的錯誤。

## Promise.prototype.finally()

`finally`方法用於指定不管 Promise 物件最後狀態如何，都會執行的操作。該方法是 ES2018 引入標準的。

```javascript
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

上面代碼中，不管`promise`最後的狀態，在執行完`then`或`catch`指定的回調函數以後，都會執行`finally`方法指定的回調函數。

下面是一個例子，服務器使用 Promise 處理請求，然後使用`finally`方法關掉服務器。

```javascript
server.listen(port)
  .then(function () {
    // ...
  })
  .finally(server.stop);
```

`finally`方法的回調函數不接受任何參數，這意味著沒有辦法知道，前面的 Promise 狀態到底是`fulfilled`還是`rejected`。這表明，`finally`方法裡面的操作，應該是與狀態無關的，不依賴於 Promise 的執行結果。

`finally`本質上是`then`方法的特例。

```javascript
promise
.finally(() => {
  // 語句
});

// 等同於
promise
.then(
  result => {
    // 語句
    return result;
  },
  error => {
    // 語句
    throw error;
  }
);
```

上面代碼中，如果不使用`finally`方法，同樣的語句需要為成功和失敗兩種情況各寫一次。有了`finally`方法，則只需要寫一次。

它的實現也很簡單。

```javascript
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

上面代碼中，不管前面的 Promise 是`fulfilled`還是`rejected`，都會執行回調函數`callback`。

從上面的實現還可以看到，`finally`方法總是會返回原來的值。

```javascript
// resolve 的值是 undefined
Promise.resolve(2).then(() => {}, () => {})

// resolve 的值是 2
Promise.resolve(2).finally(() => {})

// reject 的值是 undefined
Promise.reject(3).then(() => {}, () => {})

// reject 的值是 3
Promise.reject(3).finally(() => {})
```

## Promise.all()

`Promise.all`方法用於將多個 Promise 實例，包裝成一個新的 Promise 實例。

```javascript
const p = Promise.all([p1, p2, p3]);
```

上面代碼中，`Promise.all`方法接受一個陣列作為參數，`p1`、`p2`、`p3`都是 Promise 實例，如果不是，就會先調用下面講到的`Promise.resolve`方法，將參數轉為 Promise 實例，再進一步處理。（`Promise.all`方法的參數可以不是陣列，但必須具有 Iterator 接口，且返回的每個成員都是 Promise 實例。）

`p`的狀態由`p1`、`p2`、`p3`決定，分成兩種情況。

（1）只有`p1`、`p2`、`p3`的狀態都變成`fulfilled`，`p`的狀態才會變成`fulfilled`，此時`p1`、`p2`、`p3`的返回值組成一個陣列，傳遞給`p`的回調函數。

（2）只要`p1`、`p2`、`p3`之中有一個被`rejected`，`p`的狀態就變成`rejected`，此時第一個被`reject`的實例的返回值，會傳遞給`p`的回調函數。

下面是一個具體的例子。

```javascript
// 生成一個Promise物件的陣列
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

上面代碼中，`promises`是包含 6 個 Promise 實例的陣列，只有這 6 個實例的狀態都變成`fulfilled`，或者其中有一個變為`rejected`，才會調用`Promise.all`方法後面的回調函數。

下面是另一個例子。

```javascript
const databasePromise = connectDatabase();

const booksPromise = databasePromise
  .then(findAllBooks);

const userPromise = databasePromise
  .then(getCurrentUser);

Promise.all([
  booksPromise,
  userPromise
])
.then(([books, user]) => pickTopRecommentations(books, user));
```

上面代碼中，`booksPromise`和`userPromise`是兩個異步操作，只有等到它們的結果都返回了，才會觸發`pickTopRecommentations`這個回調函數。

注意，如果作為參數的 Promise 實例，自己定義了`catch`方法，那麼它一旦被`rejected`，並不會觸發`Promise.all()`的`catch`方法。

```javascript
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('報錯了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 報錯了]
```

上面代碼中，`p1`會`resolved`，`p2`首先會`rejected`，但是`p2`有自己的`catch`方法，該方法返回的是一個新的 Promise 實例，`p2`指向的實際上是這個實例。該實例執行完`catch`方法後，也會變成`resolved`，導致`Promise.all()`方法參數里面的兩個實例都會`resolved`，因此會調用`then`方法指定的回調函數，而不會調用`catch`方法指定的回調函數。

如果`p2`沒有自己的`catch`方法，就會調用`Promise.all()`的`catch`方法。

```javascript
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result);

const p2 = new Promise((resolve, reject) => {
  throw new Error('報錯了');
})
.then(result => result);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// Error: 報錯了
```

## Promise.race()

`Promise.race`方法同樣是將多個 Promise 實例，包裝成一個新的 Promise 實例。

```javascript
const p = Promise.race([p1, p2, p3]);
```

上面代碼中，只要`p1`、`p2`、`p3`之中有一個實例率先改變狀態，`p`的狀態就跟著改變。那個率先改變的 Promise 實例的返回值，就傳遞給`p`的回調函數。

`Promise.race`方法的參數與`Promise.all`方法一樣，如果不是 Promise 實例，就會先調用下面講到的`Promise.resolve`方法，將參數轉為 Promise 實例，再進一步處理。

下面是一個例子，如果指定時間內沒有獲得結果，就將 Promise 的狀態變為`reject`，否則變為`resolve`。

```javascript
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p
.then(console.log)
.catch(console.error);
```

上面代碼中，如果 5 秒之內`fetch`方法無法返回結果，變數`p`的狀態就會變為`rejected`，從而觸發`catch`方法指定的回調函數。

## Promise.resolve()

有時需要將現有物件轉為 Promise 物件，`Promise.resolve`方法就起到這個作用。

```javascript
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

上面代碼將 jQuery 生成的`deferred`物件，轉為一個新的 Promise 物件。

`Promise.resolve`等價於下面的寫法。

```javascript
Promise.resolve('foo')
// 等價於
new Promise(resolve => resolve('foo'))
```

`Promise.resolve`方法的參數分成四種情況。

**（1）參數是一個 Promise 實例**

如果參數是 Promise 實例，那麼`Promise.resolve`將不做任何修改、原封不動地返回這個實例。

**（2）參數是一個`thenable`物件**

`thenable`物件指的是具有`then`方法的物件，比如下面這個物件。

```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```

`Promise.resolve`方法會將這個物件轉為 Promise 物件，然後就立即執行`thenable`物件的`then`方法。

```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```

上面代碼中，`thenable`物件的`then`方法執行後，物件`p1`的狀態就變為`resolved`，從而立即執行最後那個`then`方法指定的回調函數，輸出 42。

**（3）參數不是具有`then`方法的物件，或根本就不是物件**

如果參數是一個原始值，或者是一個不具有`then`方法的物件，則`Promise.resolve`方法返回一個新的 Promise 物件，狀態為`resolved`。

```javascript
const p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```

上面代碼生成一個新的 Promise 物件的實例`p`。由於字符串`Hello`不屬於異步操作（判斷方法是字符串物件不具有 then 方法），返回 Promise 實例的狀態從一生成就是`resolved`，所以回調函數會立即執行。`Promise.resolve`方法的參數，會同時傳給回調函數。

**（4）不帶有任何參數**

`Promise.resolve`方法允許調用時不帶參數，直接返回一個`resolved`狀態的 Promise 物件。

所以，如果希望得到一個 Promise 物件，比較方便的方法就是直接調用`Promise.resolve`方法。

```javascript
const p = Promise.resolve();

p.then(function () {
  // ...
});
```

上面代碼的變數`p`就是一個 Promise 物件。

需要注意的是，立即`resolve`的 Promise 物件，是在本輪“事件循環”（event loop）的結束時，而不是在下一輪“事件循環”的開始時。

```javascript
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```

上面代碼中，`setTimeout(fn, 0)`在下一輪“事件循環”開始時執行，`Promise.resolve()`在本輪“事件循環”結束時執行，`console.log('one')`則是立即執行，因此最先輸出。

## Promise.reject()

`Promise.reject(reason)`方法也會返回一個新的 Promise 實例，該實例的狀態為`rejected`。

```javascript
const p = Promise.reject('出錯了');
// 等同於
const p = new Promise((resolve, reject) => reject('出錯了'))

p.then(null, function (s) {
  console.log(s)
});
// 出錯了
```

上面代碼生成一個 Promise 物件的實例`p`，狀態為`rejected`，回調函數會立即執行。

注意，`Promise.reject()`方法的參數，會原封不動地作為`reject`的理由，變成後續方法的參數。這一點與`Promise.resolve`方法不一致。

```javascript
const thenable = {
  then(resolve, reject) {
    reject('出錯了');
  }
};

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true
```

上面代碼中，`Promise.reject`方法的參數是一個`thenable`物件，執行以後，後面`catch`方法的參數不是`reject`拋出的“出錯了”這個字符串，而是`thenable`物件。

## 應用

### 加載圖片

我們可以將圖片的加載寫成一個`Promise`，一旦加載完成，`Promise`的狀態就發生變化。

```javascript
const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    const image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
```

### Generator 函數與 Promise 的結合

使用 Generator 函數管理流程，遇到異步操作的時候，通常返回一個`Promise`物件。

```javascript
function getFoo () {
  return new Promise(function (resolve, reject){
    resolve('foo');
  });
}

const g = function* () {
  try {
    const foo = yield getFoo();
    console.log(foo);
  } catch (e) {
    console.log(e);
  }
};

function run (generator) {
  const it = generator();

  function go(result) {
    if (result.done) return result.value;

    return result.value.then(function (value) {
      return go(it.next(value));
    }, function (error) {
      return go(it.throw(error));
    });
  }

  go(it.next());
}

run(g);
```

上面代碼的 Generator 函數`g`之中，有一個異步操作`getFoo`，它返回的就是一個`Promise`物件。函數`run`用來處理這個`Promise`物件，並調用下一個`next`方法。

## Promise.try()

實際開發中，經常遇到一種情況：不知道或者不想區分，函數`f`是同步函數還是異步操作，但是想用 Promise 來處理它。因為這樣就可以不管`f`是否包含異步操作，都用`then`方法指定下一步流程，用`catch`方法處理`f`拋出的錯誤。一般就會採用下面的寫法。

```javascript
Promise.resolve().then(f)
```

上面的寫法有一個缺點，就是如果`f`是同步函數，那麼它會在本輪事件循環的末尾執行。

```javascript
const f = () => console.log('now');
Promise.resolve().then(f);
console.log('next');
// next
// now
```

上面代碼中，函數`f`是同步的，但是用 Promise 包裝了以後，就變成異步執行了。

那麼有沒有一種方法，讓同步函數同步執行，異步函數異步執行，並且讓它們具有統一的 API 呢？回答是可以的，並且還有兩種寫法。第一種寫法是用`async`函數來寫。

```javascript
const f = () => console.log('now');
(async () => f())();
console.log('next');
// now
// next
```

上面代碼中，第二行是一個立即執行的匿名函數，會立即執行裡面的`async`函數，因此如果`f`是同步的，就會得到同步的結果；如果`f`是異步的，就可以用`then`指定下一步，就像下面的寫法。

```javascript
(async () => f())()
.then(...)
```

需要注意的是，`async () => f()`會吃掉`f()`拋出的錯誤。所以，如果想捕獲錯誤，要使用`promise.catch`方法。

```javascript
(async () => f())()
.then(...)
.catch(...)
```

第二種寫法是使用`new Promise()`。

```javascript
const f = () => console.log('now');
(
  () => new Promise(
    resolve => resolve(f())
  )
)();
console.log('next');
// now
// next
```

上面代碼也是使用立即執行的匿名函數，執行`new Promise()`。這種情況下，同步函數也是同步執行的。

鑑於這是一個很常見的需求，所以現在有一個[提案](https://github.com/ljharb/proposal-promise-try)，提供`Promise.try`方法替代上面的寫法。

```javascript
const f = () => console.log('now');
Promise.try(f);
console.log('next');
// now
// next
```

事實上，`Promise.try`存在已久，Promise 庫[`Bluebird`](http://bluebirdjs.com/docs/api/promise.try.html)、[`Q`](https://github.com/kriskowal/q/wiki/API-Reference#promisefcallargs)和[`when`](https://github.com/cujojs/when/blob/master/docs/api.md#whentry)，早就提供了這個方法。

由於`Promise.try`為所有操作提供了統一的處理機制，所以如果想用`then`方法管理流程，最好都用`Promise.try`包裝一下。這樣有[許多好處](http://cryto.net/~joepie91/blog/2016/05/11/what-is-promise-try-and-why-does-it-matter/)，其中一點就是可以更好地管理異常。

```javascript
function getUsername(userId) {
  return database.users.get({id: userId})
  .then(function(user) {
    return user.name;
  });
}
```

上面代碼中，`database.users.get()`返回一個 Promise 物件，如果拋出異步錯誤，可以用`catch`方法捕獲，就像下面這樣寫。

```javascript
database.users.get({id: userId})
.then(...)
.catch(...)
```

但是`database.users.get()`可能還會拋出同步錯誤（比如數據庫連接錯誤，具體要看實現方法），這時你就不得不用`try...catch`去捕獲。

```javascript
try {
  database.users.get({id: userId})
  .then(...)
  .catch(...)
} catch (e) {
  // ...
}
```

上面這樣的寫法就很笨拙了，這時就可以統一用`promise.catch()`捕獲所有同步和異步的錯誤。

```javascript
Promise.try(database.users.get({id: userId}))
  .then(...)
  .catch(...)
```

事實上，`Promise.try`就是模擬`try`代碼塊，就像`promise.catch`模擬的是`catch`代碼塊。
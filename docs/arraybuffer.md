# ArrayBuffer

`ArrayBuffer`物件、`TypedArray`視圖和`DataView`視圖是 JavaScript 操作二進制數據的一個接口。這些物件早就存在，屬於獨立的規格（2011 年 2 月發佈），ES6 將它們納入了 ECMAScript 規格，並且增加了新的方法。它們都是以陣列的語法處理二進制數據，所以統稱為二進制陣列。

這個接口的原始設計目的，與 WebGL 專案有關。所謂 WebGL，就是指瀏覽器與顯卡之間的通信接口，為了滿足 JavaScript 與顯卡之間大量的、實時的數據交換，它們之間的數據通信必須是二進制的，而不能是傳統的文本格式。文本格式傳遞一個 32 位整數，兩端的 JavaScript 腳本與顯卡都要進行格式轉化，將非常耗時。這時要是存在一種機制，可以像 C 語言那樣，直接操作字節，將 4 個字節的 32 位整數，以二進制形式原封不動地送入顯卡，腳本的性能就會大幅提升。

二進制陣列就是在這種背景下誕生的。它很像 C 語言的陣列，允許開發者以陣列下標的形式，直接操作內存，大大增強了 JavaScript 處理二進制數據的能力，使得開發者有可能通過 JavaScript 與操作系統的原生接口進行二進制通信。

二進制陣列由三類物件組成。

**（1）`ArrayBuffer`物件**：代表內存之中的一段二進制數據，可以通過“視圖”進行操作。“視圖”部署了陣列接口，這意味著，可以用陣列的方法操作內存。

**（2）`TypedArray`視圖**：共包括 9 種類型的視圖，比如`Uint8Array`（無符號 8 位整數）陣列視圖, `Int16Array`（16 位整數）陣列視圖, `Float32Array`（32 位浮點數）陣列視圖等等。

**（3）`DataView`視圖**：可以自定義復合格式的視圖，比如第一個字節是 Uint8（無符號 8 位整數）、第二、三個字節是 Int16（16 位整數）、第四個字節開始是 Float32（32 位浮點數）等等，此外還可以自定義字節序。

簡單說，`ArrayBuffer`物件代表原始的二進制數據，TypedArray 視圖用來讀寫簡單類型的二進制數據，`DataView`視圖用來讀寫複雜類型的二進制數據。

TypedArray 視圖支持的數據類型一共有 9 種（`DataView`視圖支持除`Uint8C`以外的其他 8 種）。

| 數據類型 | 字節長度 | 含義                             | 對應的 C 語言類型 |
| -------- | -------- | -------------------------------- | ----------------- |
| Int8     | 1        | 8 位帶符號整數                   | signed char       |
| Uint8    | 1        | 8 位不帶符號整數                 | unsigned char     |
| Uint8C   | 1        | 8 位不帶符號整數（自動過濾溢出） | unsigned char     |
| Int16    | 2        | 16 位帶符號整數                  | short             |
| Uint16   | 2        | 16 位不帶符號整數                | unsigned short    |
| Int32    | 4        | 32 位帶符號整數                  | int               |
| Uint32   | 4        | 32 位不帶符號的整數              | unsigned int      |
| Float32  | 4        | 32 位浮點數                      | float             |
| Float64  | 8        | 64 位浮點數                      | double            |

注意，二進制陣列並不是真正的陣列，而是類似陣列的物件。

很多瀏覽器操作的 API，用到了二進制陣列操作二進制數據，下面是其中的幾個。

- File API
- XMLHttpRequest
- Fetch API
- Canvas
- WebSockets

## ArrayBuffer 物件

### 概述

`ArrayBuffer`物件代表儲存二進制數據的一段內存，它不能直接讀寫，只能通過視圖（`TypedArray`視圖和`DataView`視圖)來讀寫，視圖的作用是以指定格式解讀二進制數據。

`ArrayBuffer`也是一個構造函數，可以分配一段可以存放數據的連續內存區域。

```javascript
const buf = new ArrayBuffer(32);
```

上面代碼生成了一段 32 字節的內存區域，每個字節的值默認都是 0。可以看到，`ArrayBuffer`構造函數的參數是所需要的內存大小（單位字節）。

為了讀寫這段內容，需要為它指定視圖。`DataView`視圖的創建，需要提供`ArrayBuffer`物件實例作為參數。

```javascript
const buf = new ArrayBuffer(32);
const dataView = new DataView(buf);
dataView.getUint8(0) // 0
```

上面代碼對一段 32 字節的內存，建立`DataView`視圖，然後以不帶符號的 8 位整數格式，從頭讀取 8 位二進制數據，結果得到 0，因為原始內存的`ArrayBuffer`物件，默認所有位都是 0。

另一種 TypedArray 視圖，與`DataView`視圖的一個區別是，它不是一個構造函數，而是一組構造函數，代表不同的數據格式。

```javascript
const buffer = new ArrayBuffer(12);

const x1 = new Int32Array(buffer);
x1[0] = 1;
const x2 = new Uint8Array(buffer);
x2[0]  = 2;

x1[0] // 2
```

上面代碼對同一段內存，分別建立兩種視圖：32 位帶符號整數（`Int32Array`構造函數）和 8 位不帶符號整數（`Uint8Array`構造函數）。由於兩個視圖對應的是同一段內存，一個視圖修改底層內存，會影響到另一個視圖。

TypedArray 視圖的構造函數，除了接受`ArrayBuffer`實例作為參數，還可以接受普通陣列作為參數，直接分配內存生成底層的`ArrayBuffer`實例，並同時完成對這段內存的賦值。

```javascript
const typedArray = new Uint8Array([0,1,2]);
typedArray.length // 3

typedArray[0] = 5;
typedArray // [5, 1, 2]
```

上面代碼使用 TypedArray 視圖的`Uint8Array`構造函數，新建一個不帶符號的 8 位整數視圖。可以看到，`Uint8Array`直接使用普通陣列作為參數，對底層內存的賦值同時完成。

### ArrayBuffer.prototype.byteLength

`ArrayBuffer`實例的`byteLength`屬性，返回所分配的內存區域的字節長度。

```javascript
const buffer = new ArrayBuffer(32);
buffer.byteLength
// 32
```

如果要分配的內存區域很大，有可能分配失敗（因為沒有那麼多的連續空餘內存），所以有必要檢查是否分配成功。

```javascript
if (buffer.byteLength === n) {
  // 成功
} else {
  // 失敗
}
```

### ArrayBuffer.prototype.slice()

`ArrayBuffer`實例有一個`slice`方法，允許將內存區域的一部分，拷貝生成一個新的`ArrayBuffer`物件。

```javascript
const buffer = new ArrayBuffer(8);
const newBuffer = buffer.slice(0, 3);
```

上面代碼拷貝`buffer`物件的前 3 個字節（從 0 開始，到第 3 個字節前面結束），生成一個新的`ArrayBuffer`物件。`slice`方法其實包含兩步，第一步是先分配一段新內存，第二步是將原來那個`ArrayBuffer`物件拷貝過去。

`slice`方法接受兩個參數，第一個參數表示拷貝開始的字節序號（含該字節），第二個參數表示拷貝截止的字節序號（不含該字節）。如果省略第二個參數，則默認到原`ArrayBuffer`物件的結尾。

除了`slice`方法，`ArrayBuffer`物件不提供任何直接讀寫內存的方法，只允許在其上方建立視圖，然後通過視圖讀寫。

### ArrayBuffer.isView()

`ArrayBuffer`有一個靜態方法`isView`，返回一個布爾值，表示參數是否為`ArrayBuffer`的視圖實例。這個方法大致相當於判斷參數，是否為 TypedArray 實例或`DataView`實例。

```javascript
const buffer = new ArrayBuffer(8);
ArrayBuffer.isView(buffer) // false

const v = new Int32Array(buffer);
ArrayBuffer.isView(v) // true
```

## TypedArray 視圖

### 概述

`ArrayBuffer`物件作為內存區域，可以存放多種類型的數據。同一段內存，不同數據有不同的解讀方式，這就叫做“視圖”（view）。`ArrayBuffer`有兩種視圖，一種是 TypedArray 視圖，另一種是`DataView`視圖。前者的陣列成員都是同一個數據類型，後者的陣列成員可以是不同的數據類型。

目前，TypedArray 視圖一共包括 9 種類型，每一種視圖都是一種構造函數。

- **`Int8Array`**：8 位有符號整數，長度 1 個字節。
- **`Uint8Array`**：8 位無符號整數，長度 1 個字節。
- **`Uint8ClampedArray`**：8 位無符號整數，長度 1 個字節，溢出處理不同。
- **`Int16Array`**：16 位有符號整數，長度 2 個字節。
- **`Uint16Array`**：16 位無符號整數，長度 2 個字節。
- **`Int32Array`**：32 位有符號整數，長度 4 個字節。
- **`Uint32Array`**：32 位無符號整數，長度 4 個字節。
- **`Float32Array`**：32 位浮點數，長度 4 個字節。
- **`Float64Array`**：64 位浮點數，長度 8 個字節。

這 9 個構造函數生成的陣列，統稱為 TypedArray 視圖。它們很像普通陣列，都有`length`屬性，都能用方括號運算符（`[]`）獲取單個元素，所有陣列的方法，在它們上面都能使用。普通陣列與 TypedArray 陣列的差異主要在以下方面。

- TypedArray 陣列的所有成員，都是同一種類型。
- TypedArray 陣列的成員是連續的，不會有空位。
- TypedArray 陣列成員的默認值為 0。比如，`new Array(10)`返回一個普通陣列，裡面沒有任何成員，只是 10 個空位；`new Uint8Array(10)`返回一個 TypedArray 陣列，裡面 10 個成員都是 0。
- TypedArray 陣列只是一層視圖，本身不儲存數據，它的數據都儲存在底層的`ArrayBuffer`物件之中，要獲取底層物件必須使用`buffer`屬性。

### 構造函數

TypedArray 陣列提供 9 種構造函數，用來生成相應類型的陣列實例。

構造函數有多種用法。

**（1）TypedArray(buffer, byteOffset=0, length?)**

同一個`ArrayBuffer`物件之上，可以根據不同的數據類型，建立多個視圖。

```javascript
// 創建一個8字節的ArrayBuffer
const b = new ArrayBuffer(8);

// 創建一個指向b的Int32視圖，開始於字節0，直到緩衝區的末尾
const v1 = new Int32Array(b);

// 創建一個指向b的Uint8視圖，開始於字節2，直到緩衝區的末尾
const v2 = new Uint8Array(b, 2);

// 創建一個指向b的Int16視圖，開始於字節2，長度為2
const v3 = new Int16Array(b, 2, 2);
```

上面代碼在一段長度為 8 個字節的內存（`b`）之上，生成了三個視圖：`v1`、`v2`和`v3`。

視圖的構造函數可以接受三個參數：

- 第一個參數（必需）：視圖對應的底層`ArrayBuffer`物件。
- 第二個參數（可選）：視圖開始的字節序號，默認從 0 開始。
- 第三個參數（可選）：視圖包含的數據個數，默認直到本段內存區域結束。

因此，`v1`、`v2`和`v3`是重疊的：`v1[0]`是一個 32 位整數，指向字節 0 ～字節 3；`v2[0]`是一個 8 位無符號整數，指向字節 2；`v3[0]`是一個 16 位整數，指向字節 2 ～字節 3。只要任何一個視圖對內存有所修改，就會在另外兩個視圖上反應出來。

注意，`byteOffset`必須與所要建立的數據類型一致，否則會報錯。

```javascript
const buffer = new ArrayBuffer(8);
const i16 = new Int16Array(buffer, 1);
// Uncaught RangeError: start offset of Int16Array should be a multiple of 2
```

上面代碼中，新生成一個 8 個字節的`ArrayBuffer`物件，然後在這個物件的第一個字節，建立帶符號的 16 位整數視圖，結果報錯。因為，帶符號的 16 位整數需要兩個字節，所以`byteOffset`參數必須能夠被 2 整除。

如果想從任意字節開始解讀`ArrayBuffer`物件，必須使用`DataView`視圖，因為 TypedArray 視圖只提供 9 種固定的解讀格式。

**（2）TypedArray(length)**

視圖還可以不通過`ArrayBuffer`物件，直接分配內存而生成。

```javascript
const f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```

上面代碼生成一個 8 個成員的`Float64Array`陣列（共 64 字節），然後依次對每個成員賦值。這時，視圖構造函數的參數就是成員的個數。可以看到，視圖陣列的賦值操作與普通陣列的操作毫無兩樣。

**（3）TypedArray(typedArray)**

TypedArray 陣列的構造函數，可以接受另一個 TypedArray 實例作為參數。

```javascript
const typedArray = new Int8Array(new Uint8Array(4));
```

上面代碼中，`Int8Array`構造函數接受一個`Uint8Array`實例作為參數。

注意，此時生成的新陣列，只是複製了參數陣列的值，對應的底層內存是不一樣的。新陣列會開闢一段新的內存儲存數據，不會在原陣列的內存之上建立視圖。

```javascript
const x = new Int8Array([1, 1]);
const y = new Int8Array(x);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 1
```

上面代碼中，陣列`y`是以陣列`x`為模板而生成的，當`x`變動的時候，`y`並沒有變動。

如果想基於同一段內存，構造不同的視圖，可以採用下面的寫法。

```javascript
const x = new Int8Array([1, 1]);
const y = new Int8Array(x.buffer);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 2
```

**（4）TypedArray(arrayLikeObject)**

構造函數的參數也可以是一個普通陣列，然後直接生成 TypedArray 實例。

```javascript
const typedArray = new Uint8Array([1, 2, 3, 4]);
```

注意，這時 TypedArray 視圖會重新開闢內存，不會在原陣列的內存上建立視圖。

上面代碼從一個普通的陣列，生成一個 8 位無符號整數的 TypedArray 實例。

TypedArray 陣列也可以轉換回普通陣列。

```javascript
const normalArray = [...typedArray];
// or
const normalArray = Array.from(typedArray);
// or
const normalArray = Array.prototype.slice.call(typedArray);
```

### 陣列方法

普通陣列的操作方法和屬性，對 TypedArray 陣列完全適用。

- `TypedArray.prototype.copyWithin(target, start[, end = this.length])`
- `TypedArray.prototype.entries()`
- `TypedArray.prototype.every(callbackfn, thisArg?)`
- `TypedArray.prototype.fill(value, start=0, end=this.length)`
- `TypedArray.prototype.filter(callbackfn, thisArg?)`
- `TypedArray.prototype.find(predicate, thisArg?)`
- `TypedArray.prototype.findIndex(predicate, thisArg?)`
- `TypedArray.prototype.forEach(callbackfn, thisArg?)`
- `TypedArray.prototype.indexOf(searchElement, fromIndex=0)`
- `TypedArray.prototype.join(separator)`
- `TypedArray.prototype.keys()`
- `TypedArray.prototype.lastIndexOf(searchElement, fromIndex?)`
- `TypedArray.prototype.map(callbackfn, thisArg?)`
- `TypedArray.prototype.reduce(callbackfn, initialValue?)`
- `TypedArray.prototype.reduceRight(callbackfn, initialValue?)`
- `TypedArray.prototype.reverse()`
- `TypedArray.prototype.slice(start=0, end=this.length)`
- `TypedArray.prototype.some(callbackfn, thisArg?)`
- `TypedArray.prototype.sort(comparefn)`
- `TypedArray.prototype.toLocaleString(reserved1?, reserved2?)`
- `TypedArray.prototype.toString()`
- `TypedArray.prototype.values()`

上面所有方法的用法，請參閱陣列方法的介紹，這裡不再重複了。

注意，TypedArray 陣列沒有`concat`方法。如果想要合併多個 TypedArray 陣列，可以用下面這個函數。

```javascript
function concatenate(resultConstructor, ...arrays) {
  let totalLength = 0;
  for (let arr of arrays) {
    totalLength += arr.length;
  }
  let result = new resultConstructor(totalLength);
  let offset = 0;
  for (let arr of arrays) {
    result.set(arr, offset);
    offset += arr.length;
  }
  return result;
}

concatenate(Uint8Array, Uint8Array.of(1, 2), Uint8Array.of(3, 4))
// Uint8Array [1, 2, 3, 4]
```

另外，TypedArray 陣列與普通陣列一樣，部署了 Iterator 接口，所以可以被遍歷。

```javascript
let ui8 = Uint8Array.of(0, 1, 2);
for (let byte of ui8) {
  console.log(byte);
}
// 0
// 1
// 2
```

### 字節序

字節序指的是數值在內存中的表示方式。

```javascript
const buffer = new ArrayBuffer(16);
const int32View = new Int32Array(buffer);

for (let i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}
```

上面代碼生成一個 16 字節的`ArrayBuffer`物件，然後在它的基礎上，建立了一個 32 位整數的視圖。由於每個 32 位整數佔據 4 個字節，所以一共可以寫入 4 個整數，依次為 0，2，4，6。

如果在這段數據上接著建立一個 16 位整數的視圖，則可以讀出完全不一樣的結果。

```javascript
const int16View = new Int16Array(buffer);

for (let i = 0; i < int16View.length; i++) {
  console.log("Entry " + i + ": " + int16View[i]);
}
// Entry 0: 0
// Entry 1: 0
// Entry 2: 2
// Entry 3: 0
// Entry 4: 4
// Entry 5: 0
// Entry 6: 6
// Entry 7: 0
```

由於每個 16 位整數佔據 2 個字節，所以整個`ArrayBuffer`物件現在分成 8 段。然後，由於 x86 體系的計算機都採用小端字節序（little endian），相對重要的字節排在後面的內存地址，相對不重要字節排在前面的內存地址，所以就得到了上面的結果。

比如，一個佔據四個字節的 16 進制數`0x12345678`，決定其大小的最重要的字節是“12”，最不重要的是“78”。小端字節序將最不重要的字節排在前面，儲存順序就是`78563412`；大端字節序則完全相反，將最重要的字節排在前面，儲存順序就是`12345678`。目前，所有個人電腦幾乎都是小端字節序，所以 TypedArray 陣列內部也採用小端字節序讀寫數據，或者更準確的說，按照本機操作系統設定的字節序讀寫數據。

這並不意味大端字節序不重要，事實上，很多網絡設備和特定的操作系統採用的是大端字節序。這就帶來一個嚴重的問題：如果一段數據是大端字節序，TypedArray 陣列將無法正確解析，因為它只能處理小端字節序！為瞭解決這個問題，JavaScript 引入`DataView`物件，可以設定字節序，下文會詳細介紹。

下面是另一個例子。

```javascript
// 假定某段buffer包含如下字節 [0x02, 0x01, 0x03, 0x07]
const buffer = new ArrayBuffer(4);
const v1 = new Uint8Array(buffer);
v1[0] = 2;
v1[1] = 1;
v1[2] = 3;
v1[3] = 7;

const uInt16View = new Uint16Array(buffer);

// 計算機採用小端字節序
// 所以頭兩個字節等於258
if (uInt16View[0] === 258) {
  console.log('OK'); // "OK"
}

// 賦值運算
uInt16View[0] = 255;    // 字節變為[0xFF, 0x00, 0x03, 0x07]
uInt16View[0] = 0xff05; // 字節變為[0x05, 0xFF, 0x03, 0x07]
uInt16View[1] = 0x0210; // 字節變為[0x05, 0xFF, 0x10, 0x02]
```

下面的函數可以用來判斷，當前視圖是小端字節序，還是大端字節序。

```javascript
const BIG_ENDIAN = Symbol('BIG_ENDIAN');
const LITTLE_ENDIAN = Symbol('LITTLE_ENDIAN');

function getPlatformEndianness() {
  let arr32 = Uint32Array.of(0x12345678);
  let arr8 = new Uint8Array(arr32.buffer);
  switch ((arr8[0]*0x1000000) + (arr8[1]*0x10000) + (arr8[2]*0x100) + (arr8[3])) {
    case 0x12345678:
      return BIG_ENDIAN;
    case 0x78563412:
      return LITTLE_ENDIAN;
    default:
      throw new Error('Unknown endianness');
  }
}
```

總之，與普通陣列相比，TypedArray 陣列的最大優點就是可以直接操作內存，不需要數據類型轉換，所以速度快得多。

### BYTES_PER_ELEMENT 屬性

每一種視圖的構造函數，都有一個`BYTES_PER_ELEMENT`屬性，表示這種數據類型佔據的字節數。

```javascript
Int8Array.BYTES_PER_ELEMENT // 1
Uint8Array.BYTES_PER_ELEMENT // 1
Int16Array.BYTES_PER_ELEMENT // 2
Uint16Array.BYTES_PER_ELEMENT // 2
Int32Array.BYTES_PER_ELEMENT // 4
Uint32Array.BYTES_PER_ELEMENT // 4
Float32Array.BYTES_PER_ELEMENT // 4
Float64Array.BYTES_PER_ELEMENT // 8
```

這個屬性在 TypedArray 實例上也能獲取，即有`TypedArray.prototype.BYTES_PER_ELEMENT`。

### ArrayBuffer 與字符串的互相轉換

`ArrayBuffer`轉為字符串，或者字符串轉為`ArrayBuffer`，有一個前提，即字符串的編碼方法是確定的。假定字符串採用 UTF-16 編碼（JavaScript 的內部編碼方式），可以自己編寫轉換函數。

```javascript
// ArrayBuffer 轉為字符串，參數為 ArrayBuffer 物件
function ab2str(buf) {
  // 注意，如果是大型二進制陣列，為了避免溢出，
  // 必須一個一個字符地轉
  if (buf && buf.byteLength < 1024) {
    return String.fromCharCode.apply(null, new Uint16Array(buf));
  }

  const bufView = new Uint16Array(buf);
  const len =  bufView.length;
  const bstr = new Array(len);
  for (let i = 0; i < len; i++) {
    bstr[i] = String.fromCharCode.call(null, bufView[i]);
  }
  return bstr.join('');
}

// 字符串轉為 ArrayBuffer 物件，參數為字符串
function str2ab(str) {
  const buf = new ArrayBuffer(str.length * 2); // 每個字符佔用2個字節
  const bufView = new Uint16Array(buf);
  for (let i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

### 溢出

不同的視圖類型，所能容納的數值範圍是確定的。超出這個範圍，就會出現溢出。比如，8 位視圖只能容納一個 8 位的二進制值，如果放入一個 9 位的值，就會溢出。

TypedArray 陣列的溢出處理規則，簡單來說，就是拋棄溢出的位，然後按照視圖類型進行解釋。

```javascript
const uint8 = new Uint8Array(1);

uint8[0] = 256;
uint8[0] // 0

uint8[0] = -1;
uint8[0] // 255
```

上面代碼中，`uint8`是一個 8 位視圖，而 256 的二進制形式是一個 9 位的值`100000000`，這時就會發生溢出。根據規則，只會保留後 8 位，即`00000000`。`uint8`視圖的解釋規則是無符號的 8 位整數，所以`00000000`就是`0`。

負數在計算機內部採用“2 的補碼”表示，也就是說，將對應的正數值進行否運算，然後加`1`。比如，`-1`對應的正值是`1`，進行否運算以後，得到`11111110`，再加上`1`就是補碼形式`11111111`。`uint8`按照無符號的 8 位整數解釋`11111111`，返回結果就是`255`。

一個簡單轉換規則，可以這樣表示。

- 正向溢出（overflow）：當輸入值大於當前數據類型的最大值，結果等於當前數據類型的最小值加上余值，再減去 1。
- 負向溢出（underflow）：當輸入值小於當前數據類型的最小值，結果等於當前數據類型的最大值減去余值，再加上 1。

上面的“余值”就是模運算的結果，即 JavaScript 裡面的`%`運算符的結果。

```javascript
12 % 4 // 0
12 % 5 // 2
```

上面代碼中，12 除以 4 是沒有餘值的，而除以 5 會得到余值 2。

請看下面的例子。

```javascript
const int8 = new Int8Array(1);

int8[0] = 128;
int8[0] // -128

int8[0] = -129;
int8[0] // 127
```

上面例子中，`int8`是一個帶符號的 8 位整數視圖，它的最大值是 127，最小值是-128。輸入值為`128`時，相當於正向溢出`1`，根據“最小值加上余值（128 除以 127 的余值是 1），再減去 1”的規則，就會返回`-128`；輸入值為`-129`時，相當於負向溢出`1`，根據“最大值減去余值（-129 除以-128 的余值是 1），再加上 1”的規則，就會返回`127`。

`Uint8ClampedArray`視圖的溢出規則，與上面的規則不同。它規定，凡是發生正向溢出，該值一律等於當前數據類型的最大值，即 255；如果發生負向溢出，該值一律等於當前數據類型的最小值，即 0。

```javascript
const uint8c = new Uint8ClampedArray(1);

uint8c[0] = 256;
uint8c[0] // 255

uint8c[0] = -1;
uint8c[0] // 0
```

上面例子中，`uint8C`是一個`Uint8ClampedArray`視圖，正向溢出時都返回 255，負向溢出都返回 0。

### TypedArray.prototype.buffer

TypedArray 實例的`buffer`屬性，返回整段內存區域對應的`ArrayBuffer`物件。該屬性為只讀屬性。

```javascript
const a = new Float32Array(64);
const b = new Uint8Array(a.buffer);
```

上面代碼的`a`視圖物件和`b`視圖物件，對應同一個`ArrayBuffer`物件，即同一段內存。

### TypedArray.prototype.byteLength，TypedArray.prototype.byteOffset

`byteLength`屬性返回 TypedArray 陣列佔據的內存長度，單位為字節。`byteOffset`屬性返回 TypedArray 陣列從底層`ArrayBuffer`物件的哪個字節開始。這兩個屬性都是只讀屬性。

```javascript
const b = new ArrayBuffer(8);

const v1 = new Int32Array(b);
const v2 = new Uint8Array(b, 2);
const v3 = new Int16Array(b, 2, 2);

v1.byteLength // 8
v2.byteLength // 6
v3.byteLength // 4

v1.byteOffset // 0
v2.byteOffset // 2
v3.byteOffset // 2
```

### TypedArray.prototype.length

`length`屬性表示 TypedArray 陣列含有多少個成員。注意將`byteLength`屬性和`length`屬性區分，前者是字節長度，後者是成員長度。

```javascript
const a = new Int16Array(8);

a.length // 8
a.byteLength // 16
```

### TypedArray.prototype.set()

TypedArray 陣列的`set`方法用於複製陣列（普通陣列或 TypedArray 陣列），也就是將一段內容完全複製到另一段內存。

```javascript
const a = new Uint8Array(8);
const b = new Uint8Array(8);

b.set(a);
```

上面代碼複製`a`陣列的內容到`b`陣列，它是整段內存的複製，比一個個拷貝成員的那種複製快得多。

`set`方法還可以接受第二個參數，表示從`b`物件的哪一個成員開始複製`a`物件。

```javascript
const a = new Uint16Array(8);
const b = new Uint16Array(10);

b.set(a, 2)
```

上面代碼的`b`陣列比`a`陣列多兩個成員，所以從`b[2]`開始複製。

### TypedArray.prototype.subarray()

`subarray`方法是對於 TypedArray 陣列的一部分，再建立一個新的視圖。

```javascript
const a = new Uint16Array(8);
const b = a.subarray(2,3);

a.byteLength // 16
b.byteLength // 2
```

`subarray`方法的第一個參數是起始的成員序號，第二個參數是結束的成員序號（不含該成員），如果省略則包含剩餘的全部成員。所以，上面代碼的`a.subarray(2,3)`，意味著 b 只包含`a[2]`一個成員，字節長度為 2。

### TypedArray.prototype.slice()

TypeArray 實例的`slice`方法，可以返回一個指定位置的新的 TypedArray 實例。

```javascript
let ui8 = Uint8Array.of(0, 1, 2);
ui8.slice(-1)
// Uint8Array [ 2 ]
```

上面代碼中，`ui8`是 8 位無符號整數陣列視圖的一個實例。它的`slice`方法可以從當前視圖之中，返回一個新的視圖實例。

`slice`方法的參數，表示原陣列的具體位置，開始生成新陣列。負值表示逆向的位置，即-1 為倒數第一個位置，-2 表示倒數第二個位置，以此類推。

### TypedArray.of()

TypedArray 陣列的所有構造函數，都有一個靜態方法`of`，用於將參數轉為一個 TypedArray 實例。

```javascript
Float32Array.of(0.151, -8, 3.7)
// Float32Array [ 0.151, -8, 3.7 ]
```

下面三種方法都會生成同樣一個 TypedArray 陣列。

```javascript
// 方法一
let tarr = new Uint8Array([1,2,3]);

// 方法二
let tarr = Uint8Array.of(1,2,3);

// 方法三
let tarr = new Uint8Array(3);
tarr[0] = 1;
tarr[1] = 2;
tarr[2] = 3;
```

### TypedArray.from()

靜態方法`from`接受一個可遍歷的數據結構（比如陣列）作為參數，返回一個基於這個結構的 TypedArray 實例。

```javascript
Uint16Array.from([0, 1, 2])
// Uint16Array [ 0, 1, 2 ]
```

這個方法還可以將一種 TypedArray 實例，轉為另一種。

```javascript
const ui16 = Uint16Array.from(Uint8Array.of(0, 1, 2));
ui16 instanceof Uint16Array // true
```

`from`方法還可以接受一個函數，作為第二個參數，用來對每個元素進行遍歷，功能類似`map`方法。

```javascript
Int8Array.of(127, 126, 125).map(x => 2 * x)
// Int8Array [ -2, -4, -6 ]

Int16Array.from(Int8Array.of(127, 126, 125), x => 2 * x)
// Int16Array [ 254, 252, 250 ]
```

上面的例子中，`from`方法沒有發生溢出，這說明遍歷不是針對原來的 8 位整數陣列。也就是說，`from`會將第一個參數指定的 TypedArray 陣列，拷貝到另一段內存之中，處理之後再將結果轉成指定的陣列格式。

## 復合視圖

由於視圖的構造函數可以指定起始位置和長度，所以在同一段內存之中，可以依次存放不同類型的數據，這叫做“復合視圖”。

```javascript
const buffer = new ArrayBuffer(24);

const idView = new Uint32Array(buffer, 0, 1);
const usernameView = new Uint8Array(buffer, 4, 16);
const amountDueView = new Float32Array(buffer, 20, 1);
```

上面代碼將一個 24 字節長度的`ArrayBuffer`物件，分成三個部分：

- 字節 0 到字節 3：1 個 32 位無符號整數
- 字節 4 到字節 19：16 個 8 位整數
- 字節 20 到字節 23：1 個 32 位浮點數

這種數據結構可以用如下的 C 語言描述：

```c
struct someStruct {
  unsigned long id;
  char username[16];
  float amountDue;
};
```

## DataView 視圖

如果一段數據包括多種類型（比如服務器傳來的 HTTP 數據），這時除了建立`ArrayBuffer`物件的復合視圖以外，還可以通過`DataView`視圖進行操作。

`DataView`視圖提供更多操作選項，而且支持設定字節序。本來，在設計目的上，`ArrayBuffer`物件的各種 TypedArray 視圖，是用來向網卡、聲卡之類的本機設備傳送數據，所以使用本機的字節序就可以了；而`DataView`視圖的設計目的，是用來處理網絡設備傳來的數據，所以大端字節序或小端字節序是可以自行設定的。

`DataView`視圖本身也是構造函數，接受一個`ArrayBuffer`物件作為參數，生成視圖。

```javascript
DataView(ArrayBuffer buffer [, 字節起始位置 [, 長度]]);
```

下面是一個例子。

```javascript
const buffer = new ArrayBuffer(24);
const dv = new DataView(buffer);
```

`DataView`實例有以下屬性，含義與 TypedArray 實例的同名方法相同。

- `DataView.prototype.buffer`：返回對應的 ArrayBuffer 物件
- `DataView.prototype.byteLength`：返回佔據的內存字節長度
- `DataView.prototype.byteOffset`：返回當前視圖從對應的 ArrayBuffer 物件的哪個字節開始

`DataView`實例提供 8 個方法讀取內存。

- **`getInt8`**：讀取 1 個字節，返回一個 8 位整數。
- **`getUint8`**：讀取 1 個字節，返回一個無符號的 8 位整數。
- **`getInt16`**：讀取 2 個字節，返回一個 16 位整數。
- **`getUint16`**：讀取 2 個字節，返回一個無符號的 16 位整數。
- **`getInt32`**：讀取 4 個字節，返回一個 32 位整數。
- **`getUint32`**：讀取 4 個字節，返回一個無符號的 32 位整數。
- **`getFloat32`**：讀取 4 個字節，返回一個 32 位浮點數。
- **`getFloat64`**：讀取 8 個字節，返回一個 64 位浮點數。

這一系列`get`方法的參數都是一個字節序號（不能是負數，否則會報錯），表示從哪個字節開始讀取。

```javascript
const buffer = new ArrayBuffer(24);
const dv = new DataView(buffer);

// 從第1個字節讀取一個8位無符號整數
const v1 = dv.getUint8(0);

// 從第2個字節讀取一個16位無符號整數
const v2 = dv.getUint16(1);

// 從第4個字節讀取一個16位無符號整數
const v3 = dv.getUint16(3);
```

上面代碼讀取了`ArrayBuffer`物件的前 5 個字節，其中有一個 8 位整數和兩個十六位整數。

如果一次讀取兩個或兩個以上字節，就必須明確數據的存儲方式，到底是小端字節序還是大端字節序。默認情況下，`DataView`的`get`方法使用大端字節序解讀數據，如果需要使用小端字節序解讀，必須在`get`方法的第二個參數指定`true`。

```javascript
// 小端字節序
const v1 = dv.getUint16(1, true);

// 大端字節序
const v2 = dv.getUint16(3, false);

// 大端字節序
const v3 = dv.getUint16(3);
```

DataView 視圖提供 8 個方法寫入內存。

- **`setInt8`**：寫入 1 個字節的 8 位整數。
- **`setUint8`**：寫入 1 個字節的 8 位無符號整數。
- **`setInt16`**：寫入 2 個字節的 16 位整數。
- **`setUint16`**：寫入 2 個字節的 16 位無符號整數。
- **`setInt32`**：寫入 4 個字節的 32 位整數。
- **`setUint32`**：寫入 4 個字節的 32 位無符號整數。
- **`setFloat32`**：寫入 4 個字節的 32 位浮點數。
- **`setFloat64`**：寫入 8 個字節的 64 位浮點數。

這一系列`set`方法，接受兩個參數，第一個參數是字節序號，表示從哪個字節開始寫入，第二個參數為寫入的數據。對於那些寫入兩個或兩個以上字節的方法，需要指定第三個參數，`false`或者`undefined`表示使用大端字節序寫入，`true`表示使用小端字節序寫入。

```javascript
// 在第1個字節，以大端字節序寫入值為25的32位整數
dv.setInt32(0, 25, false);

// 在第5個字節，以大端字節序寫入值為25的32位整數
dv.setInt32(4, 25);

// 在第9個字節，以小端字節序寫入值為2.5的32位浮點數
dv.setFloat32(8, 2.5, true);
```

如果不確定正在使用的計算機的字節序，可以採用下面的判斷方式。

```javascript
const littleEndian = (function() {
  const buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true);
  return new Int16Array(buffer)[0] === 256;
})();
```

如果返回`true`，就是小端字節序；如果返回`false`，就是大端字節序。

## 二進制陣列的應用

大量的 Web API 用到了`ArrayBuffer`物件和它的視圖物件。

### AJAX

傳統上，服務器通過 AJAX 操作只能返回文本數據，即`responseType`屬性默認為`text`。`XMLHttpRequest`第二版`XHR2`允許服務器返回二進制數據，這時分成兩種情況。如果明確知道返回的二進制數據類型，可以把返回類型（`responseType`）設為`arraybuffer`；如果不知道，就設為`blob`。

```javascript
let xhr = new XMLHttpRequest();
xhr.open('GET', someUrl);
xhr.responseType = 'arraybuffer';

xhr.onload = function () {
  let arrayBuffer = xhr.response;
  // ···
};

xhr.send();
```

如果知道傳回來的是 32 位整數，可以像下面這樣處理。

```javascript
xhr.onreadystatechange = function () {
  if (req.readyState === 4 ) {
    const arrayResponse = xhr.response;
    const dataView = new DataView(arrayResponse);
    const ints = new Uint32Array(dataView.byteLength / 4);

    xhrDiv.style.backgroundColor = "#00FF00";
    xhrDiv.innerText = "Array is " + ints.length + "uints long";
  }
}
```

### Canvas

網頁`Canvas`元素輸出的二進制像素數據，就是 TypedArray 陣列。

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const uint8ClampedArray = imageData.data;
```

需要注意的是，上面代碼的`uint8ClampedArray`雖然是一個 TypedArray 陣列，但是它的視圖類型是一種針對`Canvas`元素的專有類型`Uint8ClampedArray`。這個視圖類型的特點，就是專門針對顏色，把每個字節解讀為無符號的 8 位整數，即只能取值 0 ～ 255，而且發生運算的時候自動過濾高位溢出。這為圖像處理帶來了巨大的方便。

舉例來說，如果把像素的顏色值設為`Uint8Array`類型，那麼乘以一個 gamma 值的時候，就必須這樣計算：

```javascript
u8[i] = Math.min(255, Math.max(0, u8[i] * gamma));
```

因為`Uint8Array`類型對於大於 255 的運算結果（比如`0xFF+1`），會自動變為`0x00`，所以圖像處理必須要像上面這樣算。這樣做很麻煩，而且影響性能。如果將顏色值設為`Uint8ClampedArray`類型，計算就簡化許多。

```javascript
pixels[i] *= gamma;
```

`Uint8ClampedArray`類型確保將小於 0 的值設為 0，將大於 255 的值設為 255。注意，IE 10 不支持該類型。

### WebSocket

`WebSocket`可以通過`ArrayBuffer`，發送或接收二進制數據。

```javascript
let socket = new WebSocket('ws://127.0.0.1:8081');
socket.binaryType = 'arraybuffer';

// Wait until socket is open
socket.addEventListener('open', function (event) {
  // Send binary data
  const typedArray = new Uint8Array(4);
  socket.send(typedArray.buffer);
});

// Receive binary data
socket.addEventListener('message', function (event) {
  const arrayBuffer = event.data;
  // ···
});
```

### Fetch API

Fetch API 取回的數據，就是`ArrayBuffer`物件。

```javascript
fetch(url)
.then(function(response){
  return response.arrayBuffer()
})
.then(function(arrayBuffer){
  // ...
});
```

### File API

如果知道一個文件的二進制數據類型，也可以將這個文件讀取為`ArrayBuffer`物件。

```javascript
const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];
const reader = new FileReader();
reader.readAsArrayBuffer(file);
reader.onload = function () {
  const arrayBuffer = reader.result;
  // ···
};
```

下面以處理 bmp 文件為例。假定`file`變數是一個指向 bmp 文件的文件物件，首先讀取文件。

```javascript
const reader = new FileReader();
reader.addEventListener("load", processimage, false);
reader.readAsArrayBuffer(file);
```

然後，定義處理圖像的回調函數：先在二進制數據之上建立一個`DataView`視圖，再建立一個`bitmap`物件，用於存放處理後的數據，最後將圖像展示在`Canvas`元素之中。

```javascript
function processimage(e) {
  const buffer = e.target.result;
  const datav = new DataView(buffer);
  const bitmap = {};
  // 具體的處理步驟
}
```

具體處理圖像數據時，先處理 bmp 的文件頭。具體每個文件頭的格式和定義，請參閱有關資料。

```javascript
bitmap.fileheader = {};
bitmap.fileheader.bfType = datav.getUint16(0, true);
bitmap.fileheader.bfSize = datav.getUint32(2, true);
bitmap.fileheader.bfReserved1 = datav.getUint16(6, true);
bitmap.fileheader.bfReserved2 = datav.getUint16(8, true);
bitmap.fileheader.bfOffBits = datav.getUint32(10, true);
```

接著處理圖像元信息部分。

```javascript
bitmap.infoheader = {};
bitmap.infoheader.biSize = datav.getUint32(14, true);
bitmap.infoheader.biWidth = datav.getUint32(18, true);
bitmap.infoheader.biHeight = datav.getUint32(22, true);
bitmap.infoheader.biPlanes = datav.getUint16(26, true);
bitmap.infoheader.biBitCount = datav.getUint16(28, true);
bitmap.infoheader.biCompression = datav.getUint32(30, true);
bitmap.infoheader.biSizeImage = datav.getUint32(34, true);
bitmap.infoheader.biXPelsPerMeter = datav.getUint32(38, true);
bitmap.infoheader.biYPelsPerMeter = datav.getUint32(42, true);
bitmap.infoheader.biClrUsed = datav.getUint32(46, true);
bitmap.infoheader.biClrImportant = datav.getUint32(50, true);
```

最後處理圖像本身的像素信息。

```javascript
const start = bitmap.fileheader.bfOffBits;
bitmap.pixels = new Uint8Array(buffer, start);
```

至此，圖像文件的數據全部處理完成。下一步，可以根據需要，進行圖像變形，或者轉換格式，或者展示在`Canvas`網頁元素之中。

## SharedArrayBuffer

JavaScript 是單線程的，Web worker 引入了多線程：主線程用來與用戶互動，Worker 線程用來承擔計算任務。每個線程的數據都是隔離的，通過`postMessage()`通信。下面是一個例子。

```javascript
// 主線程
const w = new Worker('myworker.js');
```

上面代碼中，主線程新建了一個 Worker 線程。該線程與主線程之間會有一個通信渠道，主線程通過`w.postMessage`向 Worker 線程發消息，同時通過`message`事件監聽 Worker 線程的回應。

```javascript
// 主線程
w.postMessage('hi');
w.onmessage = function (ev) {
  console.log(ev.data);
}
```

上面代碼中，主線程先發一個消息`hi`，然後在監聽到 Worker 線程的回應後，就將其打印出來。

Worker 線程也是通過監聽`message`事件，來獲取主線程發來的消息，並作出反應。

```javascript
// Worker 線程
onmessage = function (ev) {
  console.log(ev.data);
  postMessage('ho');
}
```

線程之間的數據交換可以是各種格式，不僅僅是字符串，也可以是二進制數據。這種交換採用的是複製機制，即一個進程將需要分享的數據複製一份，通過`postMessage`方法交給另一個進程。如果數據量比較大，這種通信的效率顯然比較低。很容易想到，這時可以留出一塊內存區域，由主線程與 Worker 線程共享，兩方都可以讀寫，那麼就會大大提高效率，協作起來也會比較簡單（不像`postMessage`那麼麻煩）。

ES2017 引入[`SharedArrayBuffer`](https://github.com/tc39/ecmascript_sharedmem/blob/master/TUTORIAL.md)，允許 Worker 線程與主線程共享同一塊內存。`SharedArrayBuffer`的 API 與`ArrayBuffer`一模一樣，唯一的區別是後者無法共享。

```javascript
// 主線程

// 新建 1KB 共享內存
const sharedBuffer = new SharedArrayBuffer(1024);

// 主線程將共享內存的地址發送出去
w.postMessage(sharedBuffer);

// 在共享內存上建立視圖，供寫入數據
const sharedArray = new Int32Array(sharedBuffer);
```

上面代碼中，`postMessage`方法的參數是`SharedArrayBuffer`物件。

Worker 線程從事件的`data`屬性上面取到數據。

```javascript
// Worker 線程
onmessage = function (ev) {
  // 主線程共享的數據，就是 1KB 的共享內存
  const sharedBuffer = ev.data;

  // 在共享內存上建立視圖，方便讀寫
  const sharedArray = new Int32Array(sharedBuffer);

  // ...
};
```

共享內存也可以在 Worker 線程創建，發給主線程。

`SharedArrayBuffer`與`ArrayBuffer`一樣，本身是無法讀寫的，必須在上面建立視圖，然後通過視圖讀寫。

```javascript
// 分配 10 萬個 32 位整數佔據的內存空間
const sab = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 100000);

// 建立 32 位整數視圖
const ia = new Int32Array(sab);  // ia.length == 100000

// 新建一個質數生成器
const primes = new PrimeGenerator();

// 將 10 萬個質數，寫入這段內存空間
for ( let i=0 ; i < ia.length ; i++ )
  ia[i] = primes.next();

// 向 Worker 線程發送這段共享內存
w.postMessage(ia);
```

Worker 線程收到數據後的處理如下。

```javascript
// Worker 線程
let ia;
onmessage = function (ev) {
  ia = ev.data;
  console.log(ia.length); // 100000
  console.log(ia[37]); // 輸出 163，因為這是第38個質數
};
```

## Atomics 物件

多線程共享內存，最大的問題就是如何防止兩個線程同時修改某個地址，或者說，當一個線程修改共享內存以後，必須有一個機制讓其他線程同步。SharedArrayBuffer API 提供`Atomics`物件，保證所有共享內存的操作都是“原子性”的，並且可以在所有線程內同步。

什麼叫“原子性操作”呢？現代編程語言中，一條普通的命令被編譯器處理以後，會變成多條機器指令。如果是單線程運行，這是沒有問題的；多線程環境並且共享內存時，就會出問題，因為這一組機器指令的運行期間，可能會插入其他線程的指令，從而導致運行結果出錯。請看下面的例子。

```javascript
// 主線程
ia[42] = 314159;  // 原先的值 191
ia[37] = 123456;  // 原先的值 163

// Worker 線程
console.log(ia[37]);
console.log(ia[42]);
// 可能的結果
// 123456
// 191
```

上面代碼中，主線程的原始順序是先對 42 號位置賦值，再對 37 號位置賦值。但是，編譯器和 CPU 為了優化，可能會改變這兩個操作的執行順序（因為它們之間互不依賴），先對 37 號位置賦值，再對 42 號位置賦值。而執行到一半的時候，Worker 線程可能就會來讀取數據，導致打印出`123456`和`191`。

下面是另一個例子。

```javascript
// 主線程
const sab = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 100000);
const ia = new Int32Array(sab);

for (let i = 0; i < ia.length; i++) {
  ia[i] = primes.next(); // 將質數放入 ia
}

// worker 線程
ia[112]++; // 錯誤
Atomics.add(ia, 112, 1); // 正確
```

上面代碼中，Worker 線程直接改寫共享內存`ia[112]++`是不正確的。因為這行語句會被編譯成多條機器指令，這些指令之間無法保證不會插入其他進程的指令。請設想如果兩個線程同時`ia[112]++`，很可能它們得到的結果都是不正確的。

`Atomics`物件就是為瞭解決這個問題而提出，它可以保證一個操作所對應的多條機器指令，一定是作為一個整體運行的，中間不會被打斷。也就是說，它所涉及的操作都可以看作是原子性的單操作，這可以避免線程競爭，提高多線程共享內存時的操作安全。所以，`ia[112]++`要改寫成`Atomics.add(ia, 112, 1)`。

`Atomics`物件提供多種方法。

**（1）Atomics.store()，Atomics.load()**

`store()`方法用來向共享內存寫入數據，`load()`方法用來從共享內存讀出數據。比起直接的讀寫操作，它們的好處是保證了讀寫操作的原子性。

此外，它們還用來解決一個問題：多個線程使用共享內存的某個位置作為開關（flag），一旦該位置的值變了，就執行特定操作。這時，必須保證該位置的賦值操作，一定是在它前面的所有可能會改寫內存的操作結束後執行；而該位置的取值操作，一定是在它後面所有可能會讀取該位置的操作開始之前執行。`store`方法和`load`方法就能做到這一點，編譯器不會為了優化，而打亂機器指令的執行順序。

```javascript
Atomics.load(array, index)
Atomics.store(array, index, value)
```

`store`方法接受三個參數：SharedBuffer 的視圖、位置索引和值，返回`sharedArray[index]`的值。`load`方法只接受兩個參數：SharedBuffer 的視圖和位置索引，也是返回`sharedArray[index]`的值。

```javascript
// 主線程 main.js
ia[42] = 314159;  // 原先的值 191
Atomics.store(ia, 37, 123456);  // 原先的值是 163

// Worker 線程 worker.js
while (Atomics.load(ia, 37) == 163);
console.log(ia[37]);  // 123456
console.log(ia[42]);  // 314159
```

上面代碼中，主線程的`Atomics.store`向 42 號位置的賦值，一定是早於 37 位置的賦值。只要 37 號位置等於 163，Worker 線程就不會終止循環，而對 37 號位置和 42 號位置的取值，一定是在`Atomics.load`操作之後。

**（2）Atomics.wait()，Atomics.wake()**

使用`while`循環等待主線程的通知，不是很高效，如果用在主線程，就會造成卡頓，`Atomics`物件提供了`wait()`和`wake()`兩個方法用於等待通知。這兩個方法相當於鎖內存，即在一個線程進行操作時，讓其他線程休眠（建立鎖），等到操作結束，再喚醒那些休眠的線程（解除鎖）。

```javascript
Atomics.wait(sharedArray, index, value, time)
```

`Atomics.wait`用於當`sharedArray[index]`不等於`value`，就返回`not-equal`，否則就進入休眠，只有使用`Atomics.wake()`或者`time`毫秒以後才能喚醒。被`Atomics.wake()`喚醒時，返回`ok`，超時喚醒時返回`timed-out`。

```javascript
Atomics.wake(sharedArray, index, count)
```

`Atomics.wake`用於喚醒`count`數目在`sharedArray[index]`位置休眠的線程，讓它繼續往下運行。

下面請看一個例子。

```javascript
// 線程一
console.log(ia[37]);  // 163
Atomics.store(ia, 37, 123456);
Atomics.wake(ia, 37, 1);

// 線程二
Atomics.wait(ia, 37, 163);
console.log(ia[37]);  // 123456
```

上面代碼中，共享內存視圖`ia`的第 37 號位置，原來的值是`163`。進程二使用`Atomics.wait()`方法，指定只要`ia[37]`等於`163`，就進入休眠狀態。進程一使用`Atomics.store()`方法，將`123456`放入`ia[37]`，然後使用`Atomics.wake()`方法將監視`ia[37]`的休眠線程喚醒。

另外，基於`wait`和`wake`這兩個方法的鎖內存實現，可以看 Lars T Hansen 的 [js-lock-and-condition](https://github.com/lars-t-hansen/js-lock-and-condition) 這個庫。

注意，瀏覽器的主線程有權“拒絕”休眠，這是為了防止用戶失去響應。

**（3）運算方法**

共享內存上面的某些運算是不能被打斷的，即不能在運算過程中，讓其他線程改寫內存上面的值。Atomics 物件提供了一些運算方法，防止數據被改寫。

```javascript
Atomics.add(sharedArray, index, value)
```

`Atomics.add`用於將`value`加到`sharedArray[index]`，返回`sharedArray[index]`舊的值。

```javascript
Atomics.sub(sharedArray, index, value)
```

`Atomics.sub`用於將`value`從`sharedArray[index]`減去，返回`sharedArray[index]`舊的值。

```javascript
Atomics.and(sharedArray, index, value)
```

`Atomics.and`用於將`value`與`sharedArray[index]`進行位運算`and`，放入`sharedArray[index]`，並返回舊的值。

```javascript
Atomics.or(sharedArray, index, value)
```

`Atomics.or`用於將`value`與`sharedArray[index]`進行位運算`or`，放入`sharedArray[index]`，並返回舊的值。

```javascript
Atomics.xor(sharedArray, index, value)
```

`Atomic.xor`用於將`vaule`與`sharedArray[index]`進行位運算`xor`，放入`sharedArray[index]`，並返回舊的值。

**（4）其他方法**

`Atomics`物件還有以下方法。

- `Atomics.compareExchange(sharedArray, index, oldval, newval)`：如果`sharedArray[index]`等於`oldval`，就寫入`newval`，返回`oldval`。
- `Atomics.exchange(sharedArray, index, value)`：設置`sharedArray[index]`的值，返回舊的值。
- `Atomics.isLockFree(size)`：返回一個布爾值，表示`Atomics`物件是否可以處理某個`size`的內存鎖定。如果返回`false`，應用程序就需要自己來實現鎖定。

`Atomics.compareExchange`的一個用途是，從 SharedArrayBuffer 讀取一個值，然後對該值進行某個操作，操作結束以後，檢查一下 SharedArrayBuffer 裡面原來那個值是否發生變化（即被其他線程改寫過）。如果沒有改寫過，就將它寫回原來的位置，否則讀取新的值，再重頭進行一次操作。
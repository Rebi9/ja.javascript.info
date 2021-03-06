# ArrayBuffer, binary arrays

Web 開発では、ファイル(作成、更新、ダウンロード)を処理するときにバイナリデータに出くわします。その他の典型的なユースケースは画像処理です。

これらはすべて JavaScript で可能です。また、バイナリ操作も高性能です。

ですが、多くのクラスが存在するため少し混乱しやすいです。いくつか例を挙げます:
- `ArrayBuffer`, `Uint8Array`, `DataView`, `Blob`, `File`, etc.

JavaScript でのバイナリデータは、他の言語と比べて非標準的な方法で実装されています。しかし、一度整理できれば、すべてがとても簡単になります。

**基本となるバイナリオブジェクトは `ArrayBuffer` です。これは固定長の連続したメモリ領域への参照です。**

次のようにして生成できます:
```js run
let buffer = new ArrayBuffer(16); // 長さ 16 のバッファを作成
alert(buffer.byteLength); // 16
```

16バイトの連続したメモリ領域が割り当てられ、事前にゼロで埋められます。

```warn header="`ArrayBuffer` は何かの配列ではありません"
混乱の元は排除しましょう。`ArrayBuffer` は `Array` とは何の共通点もありません。:
- 長さは固定で、増減することはできません。
- メモリの中で、長さちょうどのスペースをとります。
- 個々のバイトにアクセスするには、`buffer[index]` ではなく、別の "view" オブジェクトが必要です。
```

`ArrayBuffer` はメモリ領域です。何が格納されているでしょう？それは分かりません。単に生のバイト列です。

**`ArrayBuffer` を操作するには、"ビュー(view)" オブジェクトを使用する必要があります。**

ビューオブジェクトは自身には何も格納しません。それは `ArrayBuffer` に格納されたバイトへの解釈を与える "メガネ" です。

例:

- **`Uint8Array`** -- `ArrayBuffer` にある各バイトを、 0 から 255 (1バイトは8ビット)までの値となる、別々の数として扱います。このような値は "8ビット符号なし整数(8-bit unsigned integer)" と呼ばれます。
- **`Uint16Array`** -- 各2バイトを、0 から 65535 までの値となる整数として扱います。これは "16ビット符号なし整数" と呼ばれます。
- **`Uint32Array`** -- 各4バイトを 0 から 4294967295 までの値となる整数として扱います。これは "32ビット符号なし整数" と呼ばれます。
- **`Float64Array`** -- 各8バイトを <code>5.0x10<sup>-324</sup></code> から <code>1.8x10<sup>308</sup></code> までの値となる浮動小数点として扱います。

したがって、16バイトの `ArrayBuffer` にあるバイナリデータは 16 個の "小さい数字", あるいは 8 個のより大きい数字(2バイトずつ), または 4 個のより大きな数字(4バイトずつ), 高精度の2個の浮動小数点値(8バイトずつとして解釈できます。

![](arraybuffer-views.svg)

`ArrayBuffer` はコアとなるオブジェクト、すべてのもののルート、生のバイナリデータです。

ですが、そこに書き込んだり反復しようとする場合、基本的にほぼすべての操作に対して、ビューを使用する必要があります。e.g:

```js run
let buffer = new ArrayBuffer(16); // 長さ 16 のバッファを作成

*!*
let view = new Uint32Array(buffer); // 32 ビット整数列としてバッファを扱います

alert(Uint32Array.BYTES_PER_ELEMENT); // 4 バイト毎の整数
*/!*

alert(view.length); // 4, 多くの整数が格納されます
alert(view.byteLength); // 16, バイトサイズ

// 値の書き込み
view[0] = 123456;

// 値のイテレート
for(let num of view) {
  alert(num); // 123456, 次に 0, 0, 0 (全部で 4 つの値)
}

```

## TypedArray

これらすべてのビュー (`Uint8Array`, `Uint32Array`, etc) の共通の用語は、[TypedArray](https://tc39.github.io/ecma262/#sec-typedarray-objects) です。これらは同じメソッドとプロパティのセットを共有します。

これらは通常の配列によく似ています: インデックスがあり、反復可能(iterable)です。

型付き配列のコンストラクタ (`Int8Array` でも `Float64Array` でも構いません)は、引数の種類に応じて異なる振る舞いをします。

5 つの引数のパターンがあります:

```js
new TypedArray(buffer, [byteOffset], [length]);
new TypedArray(object);
new TypedArray(typedArray);
new TypedArray(length);
new TypedArray();
```

1. `ArrayBuffer` の引数が与えられると、それに対するビューが作られます。我々はすでにその構文を使いました。

    オプションで、`byteOffset` で開始位置(デフォルトは 0)、`length` で長さ(デフォルトではバッファの終わりまで) を指定することができ、その場合はビューは `buffer` の一部だけをカバーします。 

2. `Array` または配列ライクなオブジェクトが与えられた場合は、同じ長さの型付き配列を生成し、内容をコピーします。

    これを使って配列にデータを事前に埋め込むことができます:
    ```js run
    *!*
    let arr = new Uint8Array([0, 1, 2, 3]);
    */!*
    alert( arr.length ); // 4
    alert( arr[1] ); // 1
    ```
3. 別の `TypedArray` が与えられた場合も同じです: 同じ長さの型付き配列を生成し、値をコピーします。値はその処理の中で新しい型に変換されます。
    ```js run
    let arr16 = new Uint16Array([1, 1000]);
    *!*
    let arr8 = new Uint8Array(arr16);
    */!*
    alert( arr8[0] ); // 1
    alert( arr8[1] ); // 232 (1000 をコピーしようとしますが、1000 は 8 ビットには収まりません(1000 は 11 1110 1000, 232 は 1110 1000)
    ```

4. 数値引数の場合は `length` であり、その数の要素を含む型付き配列を作成します。そのバイト長は、一つあたりのアイテムのバイト数 `TypedArray.BYTES_PER_ELEMENT` が `length` 倍されたものになります。:
    ```js run
    let arr = new Uint16Array(4); // 長さ 4 の型付き配列を作成します。
    alert( Uint16Array.BYTES_PER_ELEMENT ); // 要素あたり 2 バイトです
    alert( arr.byteLength ); // 8 (バイトサイズ)
    ```

5. 引数がなければ、長さゼロの型付き配列を作成します。

`ArrayBuffer` に言及することなく、直接 `TypedArray` を作成することができます。しかし、ビューはその根底にある `ArrayBuffer` なしでは存在できないため、最初のケース(`ArrayBuffer` が与えられた場合)を除く、すべての場合に自動的に作成されます。

`ArrayBuffer` にアクセスするには、次のプロパティを使います:
- `arr.buffer` -- `ArrayBuffer` への参照です。
- `arr.byteLength` -- `ArrayBuffer` の長さです。

そのため、いつでもあるビューから別のビューへ移動させることができます。:
```js
let arr8 = new Uint8Array([0, 1, 2, 3]);

// 同じデータを別のビューで
let arr16 = new Uint16Array(arr8.buffer);
```

型付き配列の一覧です:

- `Uint8Array`, `Uint16Array`, `Uint32Array` -- 8, 16, 32 ビット整数の場合
  - `Uint8ClampedArray` -- 8ビット整数の場合、代入時に "クランプ" します (下記参照)。
- `Int8Array`, `Int16Array`, `Int32Array` -- 符号付き整数の場合(負の値になることもあります）。
- `Float32Array`, `Float64Array` -- 32, 64 ビット符号付き浮動小数点の場合。

```warn header="`int8` またはそれに類似した単一値の型はありません"
注意してください、`Int8Array` のような名前にもかかわらず、JavaScript には `int` あるいは `int8` のような単一値の型はありません。

繰り返しますが、`Int8Array` はこれらの個々の値の配列ではなく、`ArrayBuffer` のビューです。
```

### 範囲外の動作

もし範囲外の値を型付き配列に書き込もうとするとどうなるでしょう？エラーにはなりませんが、余分なビットが切り捨てられます。

例えば、256 を `Uint8Array` に置くことを考えてみましょう。二進法(バイナリ形式)では、256 は `100000000` (9ビット)ですが、`Uint8Array` は値ごとに 8 ビットしか提供せず、0 から 255 までの範囲です。

範囲を超えるようなより大きい数値の場合、(重要度の低い)右端の8ビットだけが格納され、残りは切り捨てられます。:

![](8bit-integer-256.svg)

そのため、ゼロになります。

257 の場合、二進法は `100000001` (9ビット)で、右端の 8ビットが格納されるので、配列には `1` が入ります。:

![](8bit-integer-257.svg)

デモです:

```js run
let uint8array = new Uint8Array(16);

let num = 256;
alert(num.toString(2)); // 100000000 (二進表現)

uint8array[0] = 256;
uint8array[1] = 257;

alert(uint8array[0]); // 0
alert(uint8array[1]); // 1
```

`Uint8ClampedArray` はこの点で特別で異なる振る舞いをします。これは、255 よりも大きな値の場合は 255 を、負の値の場合には 0 を格納します。これは画像処理の場合に役立つ振る舞いです。

## TypedArray メソッド

`TypedArray` は通常の `Array` のメソッドを持っていますが、注目すべき例外があります。

`map`, `slice`, `find`, `reduce` 等を使用してイテレートすることができます。

ですが、できないことがいくつかあります:

- `splice` はありません。型付き配列は buffer のビューであり、これらは固定された連続したメモリ領域であるため、値を "削除" することはできません。できることは、ゼロの代入です。
- `concat` メソッドはありません。

2つの追加のメソッドがあります:

- `arr.set(fromArr, [offset])` は `fromArr` のすべての要素を `arr` へ、位置 `offset` (デフォルトは 0) を開始点としてコピーします。
- `arr.subarray([begin, end])` は `begin` から `end` (排他的)までの同じ型のビューを作成します。これは `slice` メソッドに似ています(このメソッドもサポートされています)が、何もコピーしません。単に指定されたデータ範囲を操作するための新しいビューを作成するだけです。

これらのメソッドにより、型付き配列のコピーしたり、ミックスしたり、既存の配列から新しい配列を作成したりすることができます。

## DataView

[DataView](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView) は非常に柔軟な `ArrayBuffer` に対する "型指定のない" ビューです。あらゆるフォーマットの任意のオフセットのデータにアクセスすることができます。

- 型付き配列の場合、コンストラクタはフォーマットが何であるかを決定します。配列全体は不変になります。i 番目の数値は `arr[i]` です。
- `DataView` では、`.getUint8(i)` や `.getUint16(i)` メソッドでデータにアクセスします。フォーマットはコンストラクタ時の代わりに、メソッド呼び出し時に指定します。

構文:

```js
new DataView(buffer, [byteOffset], [byteLength])
```

- **`buffer`** -- 基礎となる `ArrayBuffer` です。型付き配列とは異なり、`DataView` はそれ自身では buffer は作成しません。事前に用意する必要があります。
- **`byteOffset`** -- ビューの開始バイト位置(デフォルトは 0)です。
- **`byteLength`** -- ビューのバイト長(デフォルトは `buffer` の最後まで)です。

例えば、ここでは同じバッファから異なるフォーマットで数値を取り出します。:

```js run
let buffer = new Uint8Array([255, 255, 255, 255]).buffer;

let dataView = new DataView(buffer);

// オフセット 0 で、8ビットの数値を取得します
alert( dataView.getUint8(0) ); // 255

// オフセット 0 で、16ビットの数値を取得します、つまり 2バイトです
alert( dataView.getUint16(0) ); // 65535 (16ビット符号なし整数の最大値)

// オフセット 0 で、32ビットの数値を取得します
alert( dataView.getUint32(0) ); // 4294967295 (32ビット符号なし整数の最大値)

dataView.setUint32(0, 0); // 4バイトの数値を 0 にセット
```

`DataView` はフォーマットが混在したデータを同じバッファに格納するときに役立ちます。E.g 一連のペア(16ビット整数、32ビット float)を格納します。その後、`DataView` は簡単にそれらにアクセスすることができます。

## サマリ

`ArrayBuffer` は基本となるオブジェクトで、固定長の連続したメモリ領域への参照です。

`ArrayBuffer` に対するほとんどの操作は、次のビューが必要です。

- `TypedArray`:
    - `Uint8Array`, `Uint16Array`, `Uint32Array` -- 8, 16 あるいは 32 ビットの符号なし整数の場合
    - `Uint8ClampedArray` -- 8 ビット整数の場合。代入時に "クランプ" します 
    - `Int8Array`, `Int16Array`, `Int32Array` -- 符号あり整数値の場合(負の値もあり)
    - `Float32Array`, `Float64Array` -- 32, 64 ビット符号あり浮動小数点
- もしくは `DataView` -- フォーマットを指定するメソッドを使用するビューです。例えば、`getUint8(offset)`.

多くの場合、`ArrayBuffer` を隠したまま、直接型付き配列を作成し操作します。`.buffer` でアクセスすることができ、必要に応じて別のビューを作成することができます。

その他に2つの用語があります:
- `ArrayBufferView` はこれらすべての種類のビューの総称です。
- `BufferSource` は `ArrayBuffer` もしくは `ArrayBufferView` の総称です。

これらはバイナリデータを操作するメソッドの説明で使用されています。`BufferSource` は最も一般的な用語で、"あらゆる種類のバイナリデータ" -- `ArrayBuffer` またはそれを覆うビュー、を意味します。

チートシートです:

![](arraybuffer-view-buffersource.svg)

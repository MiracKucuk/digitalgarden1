---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/7-js-variables/","created":"2025-06-12T23:06:10.305+03:00","updated":"2025-06-12T23:46:35.068+03:00"}
---


Değişkenler Veri Depolamak için Konteynerlerdir

JavaScript Değişkenleri 4 şekilde bildirilebilir:

* Otomatik olarak
* `var` kullanarak
* `let` kullanarak
* `const` kullanarak

Bu ilk örnekte, `x`, `y` ve `z` tanımlanmamış değişkenlerdir.
İlk kullanıldıklarında otomatik olarak bildirilirler:

```js
x = 5;  
y = 6;  
z = x + y;
```

* Değişkenleri kullanmadan önce her zaman bildirmek iyi bir programlama uygulaması olarak kabul edilir.

Örneklerden tahmin edebilirsiniz:
* x 5 değerini saklar
* y 6 değerini saklar
* z 11 değerini saklar

### Example using `var`

```js
var x = 5;  
var y = 6;  
var z = x + y;
```

## Note

* `var` keyword'ü 1995'ten 2015'e kadar tüm JavaScript kodlarında kullanılmıştır.
* `let` ve `const` keywordleri JavaScript'e 2015 yılında eklenmiştir.
* `var` keyword'ü yalnızca eski tarayıcılar için yazılmış kodlarda kullanılmalıdır.

### Example using `let`

<div id="protectedContent" style="display: none;">
  ## Gizli Sayfa

  Bu içerik yalnızca doğru şifre girilirse görünür.

  - Özel notlar
  - Gizli video
</div>
```js
let x = 5;  
let y = 6;  
let z = x + y;
```

### Example using `const`

```js
const x = 5;  
const y = 6;  
const z = x + y;
```

### Mixed Example

```js
const price1 = 5;  
const price2 = 6;  
let total = price1 + price2;
```

* İki değişken `price1` ve `price2` `const` keyword ile bildirilir.
* Bunlar sabit değerlerdir ve `değiştirilemezler`.
* `Total` değişkeni `let` keyword ile bildirilir.
* `Total` değeri `değiştirilebilir`.

## When to Use `var`, `let`, or `const`?

1. Her zaman değişkenleri bildirin
2. Değerin değiştirilmemesi gerekiyorsa her zaman `const` kullanın
3. Türün değiştirilmemesi gerekiyorsa her zaman `const` kullanın (Arrays ve Objects)
4. Sadece `const` kullanamıyorsanız `let` kullanın
5. `Var`'ı yalnızca eski tarayıcıları desteklemeniz gerekiyorsa kullanın.

Tıpkı cebirde olduğu gibi, değişkenler değerleri tutar:

```js
let x = 5;  
let y = 6;
```

Tıpkı cebirde olduğu gibi, değişkenler expression'larda kullanılır:

```js
let z = x + y;
```

Yukarıdaki örnekten, toplamın 11 olarak hesaplandığını tahmin edebilirsiniz.

Değişkenler, değerleri saklamak için kullanılan konteynerlerdir.

## JavaScript Identifiers

* Tüm JavaScript değişkenleri benzersiz isimlerle tanımlanmalıdır.
* Bu benzersiz adlara identifier'lar (tanımlayıcılar) denir.
* Tanımlayıcılar kısa isimler (x ve y gibi) veya daha açıklayıcı isimler (`age`, `sum`, `totalVolume`) olabilir.

Değişkenler (benzersiz tanımlayıcılar) için isim oluşturmaya yönelik genel kurallar şunlardır:

* İsimler harf, rakam, alt çizgi ve dolar işareti içerebilir.
* İsimler bir harfle başlamalıdır.
* İsimler `$` ve `_` ile de başlayabilir (ancak bu derste kullanmayacağız).
* İsimler büyük/küçük harfe duyarlıdır (y ve Y farklı değişkenlerdir).
* Rezerve edilmiş kelimeler (JavaScript keywords gibi) isim olarak kullanılamaz.

JavaScript tanımlayıcıları büyük/küçük harfe duyarlıdır.

## The Assignment Operator (Atama Operatörü)

JavaScript'te eşittir işareti (`=`) bir “`atama`” operatörüdür, “eşittir” operatörü değildir.

Bu cebirden farklıdır. Aşağıdakiler cebirde bir anlam ifade etmez:

```js
x = x + 5
```

Ancak JavaScript'te bu mükemmel bir anlam ifade eder: `x + 5` değerini `x`'e atar.

(`x + 5` değerini hesaplar ve sonucu `x`'in içine koyar. `x`'in değeri `5` artırılır).

Not : “`equal to`” operatörü JavaScript'te `==` şeklinde yazılır.

## JavaScript Data Types

* JavaScript değişkenleri `100` gibi sayıları ve “`John Doe`” gibi `text` değerlerini tutabilir.
* Programlamada `text` değerlerine text `string` denir.
* JavaScript birçok veri türünü işleyebilir, ancak şimdilik sadece numaraları ve stringleri düşünün.
* Stringler çift veya tek tırnak içinde yazılır. Sayılar tırnak işaretleri olmadan yazılır.
* Tırnak içine bir sayı koyarsanız, bu bir text string olarak değerlendirilir.

```js
const pi = 3.14;  
let person = "John Doe";  
let answer = 'Yes I am!';
```

## Declaring a JavaScript Variable (Değişkeni Bildirme)

* JavaScript'te bir değişken oluşturmaya değişken “`declaring` (bildirmek)” denir.
* Bir JavaScript değişkenini `var` veya `let` keyword'ü ile bildirirsiniz:

```js
var carName;
```

veya ; 

```js
let carName;
```

* Bildirimden sonra değişkenin değeri yoktur (teknik olarak tanımsızdır).
* Değişkene bir değer atamak için eşittir işaretini kullanın:

```js
carName = "Volvo";
```

Değişkeni bildirdiğinizde ona bir değer de atayabilirsiniz:

```js
let carName = "Volvo";
```

Aşağıdaki örnekte, `carName` adında bir değişken yaratıyoruz ve ona “`Volvo`” değerini atıyoruz.

Aşağıdaki örnekte, `carName` adında bir değişken oluşturuyoruz ve ona “`Volvo`” değerini atıyoruz.

Daha sonra bu değeri `id="demo"` ile bir HTML paragrafı içinde “`output`” ediyoruz:

```js
<p id="demo"></p>  
  
<script>  
let carName = "Volvo";  
document.getElementById("demo").innerHTML = carName;  
</script>
```

Note: Tüm değişkenleri bir kodun başında bildirmek iyi bir programlama uygulamasıdır.

## One Statement, Many Variables

* Bir statement'da birçok değişken bildirebilirsiniz.
* statement'ı `let` ile başlatın ve değişkenleri `virgülle` ayırın:

```js
<p id="demo"></p>

<script>
let person = "John Doe", carName = "Volvo", price = 200;
document.getElementById("demo").innerHTML = carName;
</script>
```

Bir bildirim birden fazla satıra yayılabilir:

```js
<p id="demo"></p>

<script>
let person = "John Doe",
carName = "Volvo",
price = 200;
document.getElementById("demo").innerHTML = carName;
</script>
```

## Value = undefined

* Bilgisayar programlarında, değişkenler genellikle bir değer olmadan bildirilir. Değer, hesaplanması gereken bir şey ya da kullanıcı input'u gibi daha sonra sağlanacak bir şey olabilir.
* Değeri olmadan bildirilen bir değişkenin değeri `undefined` olacaktır.
* `carName` değişkeni, bu statement yürütüldükten sonra `undefined` değerine sahip olacaktır:

```js
let carName;
```

```js
<script>
let carName;
document.getElementById("demo").innerHTML = carName;
</script>
```

```
undefined
```


## Re-Declaring JavaScript Variables (Değişkenlerini Yeniden Bildirme)

* `var` ile bildirilen bir JavaScript değişkenini re-declare ederseniz, değişken değerini kaybetmez.

* `carName` değişkeni, bu statement'ların yürütülmesinden sonra hala “`Volvo`” değerine sahip olacaktır:

```js
var carName = "Volvo";  
var carName;
```

```js
<script>
var carName = "Volvo";
var carName;
document.getElementById("demo").innerHTML = carName;
</script>
```

```
Volvo
```

#### Not

* `let` veya `const` ile bildirilmiş bir değişkeni yeniden bildiremezsiniz. Bu işe yaramayacaktır:

```js
let carName = "Volvo";  
let carName;
```

## JavaScript Arithmetic

Cebirde olduğu gibi, `=` ve `+` gibi operatörleri kullanarak JavaScript değişkenleriyle aritmetik yapabilirsiniz:

```js
<p id="demo"></p>

<script>
let x = 5 + 2 + 3;
document.getElementById("demo").innerHTML = x;
</script>
```

```Output
10
```

String de ekleyebilirsiniz, ancak stringler birleştirilecektir:

```js
let x = "John" + " " + "Doe";
```

```Output
John Doe
```

Bunu da dene:

```js
let x = "5" + 2 + 3;
```

```Output
523
```

Not : Tırnak içine bir sayı koyarsanız, sayıların geri kalanı string olarak değerlendirilir ve birleştirilir.

Şimdi bunu dene:

```js
let x = 2 + 3 + "5";
```

```Output
55
```

## JavaScript Dollar Sign `$`

JavaScript dolar işaretini harf olarak kabul ettiğinden, `$` içeren tanımlayıcılar geçerli değişken adlarıdır:

```js
let $ = "Hello World";  
let $$ = 2;  
let $myMoney = 5;
```

```js
<script>
let $$ = 2;
let $myMoney = 5;
document.getElementById("demo").innerHTML = $$ + $myMoney;
</script>
```

```
7
```

Dolar işaretinin kullanımı JavaScript'te çok yaygın değildir, ancak profesyonel programcılar bunu genellikle bir JavaScript kütüphanesindeki `main` fonksiyonu için bir takma ad olarak kullanırlar.

Örneğin `jQuery` JavaScript kütüphanesinde `$ main` fonksiyonu HTML elementlerini seçmek için kullanılır. jQuery'de `$(“p”);` “tüm p elementlerini seç” anlamına gelir.

## JavaScript Underscore (`_`)

JavaScript alt çizgiyi harf olarak kabul ettiğinden, `_` içeren tanımlayıcılar geçerli değişken adlarıdır:

```js
let _lastName = "Johnson";  
let _x = 2;  
let _100 = 5;
```

JavaScript'te alt çizgi kullanımı çok yaygın değildir, ancak profesyonel programcılar arasındaki bir gelenek, bunu “özel (gizli)” değişkenler için bir takma ad olarak kullanmaktır.


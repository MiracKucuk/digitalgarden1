---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/5-js-syntax/","created":"2025-06-12T16:29:42.540+03:00","updated":"2025-06-12T17:01:09.346+03:00"}
---

JavaScript syntax , JavaScript programlarının nasıl oluşturulduğunu gösteren kurallar bütünüdür:

```js
// Değişkenler nasıl oluşturulur?
var x;
let y;

// Değişkenler nasıl kullanılır?
x = 5;
y = 6;
let z = x + y;
```

## JavaScript Values

JavaScript syntax'ı iki tür değer tanımlar:

- Fixed values
- Variable values

Sabit değerlere `Literals` denir.
Değişken değerlere `Variables` (Değişkenler) denir.

## JavaScript Literals

Sabit değerler için en önemli iki sözdizimi kuralı şunlardır:

1. `Number`'lar ondalıklı veya ondalıksız olarak yazılır:

```
10.50  
1001
```

```js
<script>
document.getElementById("demo").innerHTML = 10.50;
</script>
```

2. String'ler çift veya tek tırnak içinde yazılmış metinlerdir:

```
"John Doe"  
  
'John Doe'
```

```js
<script>
document.getElementById("demo").innerHTML = 'John Doe';
</script>
```

## JavaScript Variables

* Bir programlama dilinde variable'lar veri değerlerini saklamak için kullanılır.
* JavaScript, değişkenleri bildirmek için `var`, `let` ve `const` keyword'lerini kullanır.
* Değişkenlere değer atamak için eşittir işareti kullanılır.
* Bu örnekte, `x` bir değişken olarak tanımlanmıştır. Ardından, `x`'e `6` değeri atanır (verilir):

```js
let x;
x = 6;
```

## JavaScript Operators

JavaScript, değerleri hesaplamak için aritmetik operatörleri ( `+ - * /` ) kullanır:

```
(5 + 6) * 10
```

JavaScript, değişkenlere değer atamak için bir atama (assignment) operatörü ( `=` ) kullanır:

```js
let x, y;
x = 5;
y = 6;
```

## JavaScript Expressions

* Expression , bir değere hesaplama yapan değerlerin, değişkenlerin ve operatörlerin bir kombinasyonudur.
* Hesaplamaya "evaluation (değerlendirme)" adı verilir.
* Örneğin, `5 * 10, 50` olarak değerlendirilir:

```
5 * 10
```

Expression'lar değişken değerler de içerebilir:

```
x * 10
```

* Values'lar, numbers ve string'ler gibi çeşitli tiplerde olabilir.
* Örneğin, `"John" + " " + "Doe"`, `"John Doe"` olarak değerlendirilir:

```js
"John" + " " + "Doe"
```


## JavaScript Keywords

* JavaScript keyword'leri gerçekleştirilecek action'ları tanımlamak için kullanılır.
* `let` keyword'ü tarayıcıya değişkenler oluşturmasını söyler:

```js
let x, y;
x = 5 + 6;
y = x * 10;
```

* Bu örneklerde `var` veya `let` kullanımı aynı sonucu verecektir.
* Bu eğitimin ilerleyen bölümlerinde `var` ve `let` hakkında daha fazla bilgi edineceksiniz.

## JavaScript Comments

* Tüm JavaScript statement'leri “yürütülmez”.
* Çift eğik çizgiden sonra `//` veya `/*` ve `*/` arasındaki kod yorum olarak değerlendirilir.
* Yorumlar yok sayılır ve yürütülmez:

```js
let x = 5;   // Bu satır çalıştırılacak
  
// x = 6;   Bu satır YORUM satırı olduğu için çalıştırılmayacak
```

Daha sonraki bir bölümde yorumlar hakkında daha fazla bilgi edineceksiniz.

## JavaScript Identifiers / Names

* Identifiers JavaScript isimleridir.
* Identifiers değişkenleri, keywordleri ve fonksiyonları adlandırmak için kullanılır.
* Uygun adlar için kurallar çoğu programlama dilinde aynıdır.
* Bir JavaScript adı ile başlamalıdır:
	* Bir harf (`A-Z` veya `a-z`)
	* Bir dolar işareti (`$`)
	* Veya bir alt çizgi (`_`)

Sonraki karakterler harf, rakam, alt çizgi veya dolar işareti olabilir.

##### Not
* İsimlerde ilk karakter olarak sayılara izin verilmez.
* Bu şekilde JavaScript tanımlayıcıları sayılardan kolayca ayırt edebilir.

## JavaScript is Case Sensitive (Büyük/Küçük Harfe Duyarlıdır)

* Tüm JavaScript tanımlayıcıları büyük/küçük harfe duyarlıdır.
* `lastName` ve `lastname` değişkenleri iki farklı değişkendir:

```js
let lastname, lastName;  
lastName = "Doe";  
lastname = "Peterson";
```

JavaScript LET veya Let keywordleri `let` olarak yorumlamaz.


## JavaScript and Camel Case

Tarihsel olarak, programcılar birden fazla kelimeyi tek bir değişken isminde birleştirmek için farklı yollar kullanmışlardır:

Kısa çizgiler:
* first-name, last-name, master-card, inter-city

JavaScript'te tire işaretlerine izin verilmez. Bunlar subtraction'lar için ayrılmıştır.

Underscore (alt çizgi): 
* `first_name`, `last_name`, `master_card`, `inter_city`.

Upper Camel Case (Pascal Case):
* `FirstName`, `LastName`, `MasterCard`, `InterCity`.

Lower Camel Case:
* JavaScript programcıları küçük harfle başlayan camel case kullanma eğilimindedir:
* `firstName`, `lastName`, `masterCard`, `interCity`.


## JavaScript Character Set

* JavaScript `Unicode` karakter setini kullanır.
* Unicode (neredeyse) dünyadaki tüm karakterleri, noktalama işaretlerini ve sembolleri kapsar.
* Daha yakından bakmak için lütfen[ Eksiksiz Unicode Referansımızı](https://www.w3schools.com/charsets/ref_html_utf8.asp) inceleyin.


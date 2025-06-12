---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/4-statements/","created":"2025-06-12T15:45:43.509+03:00","updated":"2025-06-12T16:26:42.440+03:00"}
---


```js
let x, y, z;  // Statement 1 
x = 5;        // Statement 2
y = 6;        // Statement 3
z = x + y;    // Statement 4
```

Bir bilgisayar programı, bir bilgisayar tarafından "yürütülecek" (execute) "talimatlar" (instructions) listesidir.

Bir programlama dilinde, bu programlama talimatlarına **statement** (ifadeler) denir.

Bir JavaScript programı, programlama **statements**'larının bir listesidir.

HTML'de JavaScript programları web tarayıcısı tarafından yürütülür.

## JavaScript Statements

* JavaScript statement'ları şunlardan oluşur::

* `Values` (Değerler), `Operators` (Operatörler), `Expressions` (İfadeler), `Keywords` (Anahtar Kelimeler) ve `Comments` (Yorumlar).

* Bu statement tarayıcıya `id="demo"` olan bir HTML elementinin içine `“Hello Dolly.”` yazmasını söyler:

```js
document.getElementById("demo").innerHTML = "Hello Dolly";
```

* Çoğu JavaScript programı çok sayıda JavaScript statement'ı içerir.
* Statement, yazıldıkları sırayla teker teker çalıştırılır.
* JavaScript programları (ve JavaScript statement'ları) genellikle JavaScript kodu olarak adlandırılır.

## Semicolons `;` (Noktalı Virgül)

* Noktalı virgüller JavaScript statement'larını ayırır.
* Her çalıştırılabilir statement'ın sonuna bir noktalı virgül ekleyin:

```js
let a, b, c;  // 3 değişken tanımla
a = 5;        // a değişkenine 5 değerini ata
b = 6;        // b değişkenine 6 değerini ata
c = a + b;    // a ve b'nin toplamını c değişkenine ata
```

Noktalı virgüllerle ayrıldığında, bir satırda birden fazla ifadeye izin verilir:

```js
a = 5; b = 6; c = a + b;
```

Web'de noktalı virgül içermeyen örnekler görebilirsiniz. Statement'ları noktalı virgülle bitirmek zorunlu değildir, ancak şiddetle tavsiye edilir.

## JavaScript White Space

JavaScript birden fazla boşluğu yok sayar. Daha okunabilir hale getirmek için kodunuza white space ekleyebilirsiniz.

Aşağıdaki satırlar eşdeğerdir:

```js
let person = "Hege";
let person ="Hege";
```

İyi bir uygulama, operatörlerin ( `= + - * /` ) etrafına boşluk koymaktır:

```js
let x = y + z;
```

## JavaScript Line Length and Line Breaks (JavaScript Satır Uzunluğu ve Satır Kesmeleri)

* En iyi okunabilirlik için, programcılar genellikle `80` karakterden uzun kod satırlarından kaçınmak isterler.
* Bir JavaScript statement'ı tek bir satıra sığmıyorsa, onu kesmek için en iyi yer bir operatörden sonrasıdır:

```js
document.getElementById("demo").innerHTML =  
"Hello Dolly!";
```

## JavaScript Code Blocks

* JavaScript statement'leri, küme parantezleri `{...}` içinde kod blokları halinde gruplandırılabilir.
* Kod bloklarının amacı, birlikte yürütülecek statement'leri tanımlamaktır.
* Statement'ları bloklar halinde gruplandırılmış olarak bulabileceğiniz bir yer de JavaScript fonksiyonlarıdır:

```js
function myFunction() {
  document.getElementById("demo1").innerHTML = "Hello Dolly!";
  document.getElementById("demo2").innerHTML = "How are you?";
}
```

Bu derste kod blokları için 2 boşlukluk girinti kullanacağız. Bu eğitimin ilerleyen bölümlerinde fonksiyonlar hakkında daha fazla bilgi edineceksiniz.

## JavaScript Keywords

* JavaScript statement'ları genellikle gerçekleştirilecek JavaScript action'ını tanımlamak için bir `keyword` ile başlar.
* [Reserved Words Referansımız ](https://www.w3schools.com/js/js_reserved.asp)tüm JavaScript keyword'lerini listeler.
* İşte bu eğitimde öğreneceğiniz bazı keyword'lerin bir listesi:

| Keyword    | Description                                                             |
| ---------- | ----------------------------------------------------------------------- |
| `var`      | Bir değişken bildirir                                                   |
| `let`      | Bir blok değişkeni bildirir                                             |
| `const`    | Bir blok sabiti bildirir                                                |
| `if`       | Bir koşula bağlı olarak yürütülecek bir statement'lar bloğunu işaretler |
| `switch`   | Farklı durumlarda çalıştırılacak bir statement'ler bloğunu işaretler    |
| `for`      | Bir döngü içinde yürütülecek bir statements bloğunu işaretler           |
| `function` | Bir fonksiyon bildirir                                                  |
| `return`   | Bir fonksiyondan çıkar                                                  |
| `try`      | Bir statement'lar bloğuna handling (hata işleme) uygular                |

JavaScript keyword'leri ayrılmış kelimelerdir. Ayrılmış sözcükler değişken adı olarak kullanılamaz.

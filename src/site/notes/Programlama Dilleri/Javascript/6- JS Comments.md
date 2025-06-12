---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/6-js-comments/","created":"2025-06-12T22:42:07.477+03:00","updated":"2025-06-12T23:05:17.058+03:00"}
---


* JavaScript yorumları, JavaScript kodunu açıklamak ve daha okunabilir hale getirmek için kullanılabilir.
* JavaScript yorumları, alternatif kodu test ederken yürütmeyi önlemek için de kullanılabilir.

## Single Line Comments

* Tek satırlık yorumlar `//` ile başlar.
* `//` ile satır sonu arasındaki tüm metinler JavaScript tarafından yok sayılır (yürütülmez).
* Bu örnek, her kod satırından önce tek satırlık bir yorum kullanır:

```js
// Change heading:  
document.getElementById("myH").innerHTML = "My First Page";  
  
// Change paragraph:  
document.getElementById("myP").innerHTML = "My first paragraph.";
```

Bu örnek, kodu açıklamak için her satırın sonunda tek satırlık bir yorum kullanır:

```js
let x = 5;       // x değişkenini tanımla, ona 5 değerini ata
let y = x + 2;   // y değişkenini tanımla, ona x + 2 değerini ata
```

## Multi-line Comments

* Çok satırlı yorumlar `/*` ile başlar ve `*/` ile biter.
* `/*` ve `*/` arasındaki tüm metinler JavaScript tarafından yok sayılır.
* Bu örnekte kodu açıklamak için çok satırlı bir yorum (bir yorum bloğu) kullanılmıştır:

```js
/*
Aşağıdaki kod, web sayfamdaki
id'si "myH" olan başlığı
ve id'si "myP" olan paragrafı değiştirecek:
*/
document.getElementById("myH").innerHTML = "İlk Sayfam";
document.getElementById("myP").innerHTML = "İlk paragrafım.";
```

En yaygın kullanım tek satırlık yorumlardır. Blok yorumlar genellikle resmi dokümantasyon için kullanılır.

## Yorum Satırlarını Kullanarak Kodun Çalışmasını Engellemek

* Kodun yürütülmesini önlemek için yorumların kullanılması kod testi için uygundur.
* Bir kod satırının önüne `//` eklemek, kod satırlarını çalıştırılabilir bir satırdan bir yoruma dönüştürür.
* Bu örnekte, kod satırlarından birinin yürütülmesini engellemek için `//` kullanılır:

```js
//document.getElementById("myH").innerHTML = "My First Page";  
document.getElementById("myP").innerHTML = "My first paragraph.";
```

Bu örnek, birden fazla satırın yürütülmesini önlemek için bir yorum bloğu kullanır:

```js
/*  
document.getElementById("myH").innerHTML = "My First Page";  
document.getElementById("myP").innerHTML = "My first paragraph.";  
*/
```



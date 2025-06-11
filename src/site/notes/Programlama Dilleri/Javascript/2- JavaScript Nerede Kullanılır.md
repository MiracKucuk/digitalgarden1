---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/2-java-script-nerede-kullanilir/","created":"2025-06-12T00:29:00.084+03:00","updated":"2025-06-12T01:16:26.362+03:00"}
---


## `script` Tag

HTML'de JavaScript kodu `<script>` ve `</script>` tagları arasına eklenir.

```js
<script>
document.getElementById("demo").innerHTML = "My First Javascript"
</script>
```

Eski JavaScript örnekleri **`type`** özelliğini kullanabilir:

```js
<script type="text/javascript">
```

**`type`** özelliği gerekli değildir. HTML'de varsayılan script dili JavaScript'tir.


## JavaScript Fonksiyonları ve Eventler

* Bir JavaScript `function`, “call edildiğinde” çalıştırılabilen bir JavaScript code bloğudur.

* Örneğin, kullanıcı bir butona tıkladığında olduğu gibi bir `event` gerçekleştiğinde bir fonksiyon çağrılabilir.

* İlerleyen bölümlerde fonksiyonlar ve event'ler hakkında çok daha fazla şey öğreneceksiniz.

## `head` veya `body` içinde JavaScript

* Bir HTML document'e istediğiniz sayıda script yerleştirebilirsiniz.

* Scriptler bir HTML sayfasının `<body>` veya `<head>` bölümüne ya da her ikisine birden yerleştirilebilir.

## `head` içinde JavaScript

* Bu örnekte, bir HTML sayfasının `<head>` bölümüne bir JavaScript `fonksiyonu` yerleştirilmiştir.

* Bir buton tıklandığında fonksiyon çağrılır (çağırılır):

```html
<!DOCTYPE html>
<html>
<head>
    <script>
        function myFunction() {
            document.getElementById("demo").innerHTML = "Paragraph changed.";
        }
    </script>
</head>
<body>

    <h2>Demo JavaScript in Head</h2>

    <p id="demo">A Paragraph</p>
    <button type="button" onclick="myFunction()">Try it</button>

</body>
</html>

```


## `body` içinde JavaScript


```html
<!DOCTYPE html>
<html>
<body>

    <h2>Demo JavaScript in Body</h2>

    <p id="demo">A Paragraph</p>

    <button type="button" onclick="myFunction()">Try it</button>

    <script>
        function myFunction() {
            document.getElementById("demo").innerHTML = "Paragraph changed.";
        }
    </script>

</body>
</html>
```

`<body>` tag'inin altına script yerleştirmek, ekranın daha hızlı görüntülenmesini sağlar, çünkü script yorumlanması görüntülemeyi yavaşlatır. Eğer `<script>` tagı sayfanın **başında** (örneğin `<head>` kısmında) yer alırsa; Tarayıcı önce JavaScript dosyasını indirip çalıştırır. O sırada sayfa içeriği (başlık, yazı, resimler vs.) yüklenmez. Çözüm; Script'i `<body>`'nin en altına koymak . HTML elementleri **önce yüklenir ve görüntülenir**, sonra JavaScript çalışır. Böylece sayfa daha **hızlı görünür**. Alternatif Modern Yöntem; Script tagı `head` içinde bile olsa aşağıdaki gibi yazarsan, sayfanın çizimi yine durmaz:

```js
<script src="script.js" defer></script>
```

`defer`: HTML tamamen yüklendikten sonra script çalışır.

## External JavaScript

Scriptler external file'lara da yerleştirilebilir:

```js
function myFunction() {  
  document.getElementById("demo").innerHTML = "Paragraph changed.";  
}
```

* **External** script'ler, aynı kodun birçok farklı web sayfasında kullanıldığı durumlarda pratiktir.

* JavaScript dosyaları **`.js`** uzantısına sahiptir.

* Bir **external** script kullanmak için, script dosyasının adını **`src`** (source) özelliğine ekleyerek **`<script>`** tag'inde belirtin:

```html
<p id="demo">A Paragraph.</p>

<button type="button" onclick="myFunction()">Try it</button>

<p>Bu örnek “myScript.js” dosyasına link vermektedir.</p>
<p>(myFunction “myScript.js” içinde saklanır)</p>

<script src="myScript.js"></script>
```


* Bir **external** script referansını **`<head>`** veya **`<body>`** tag'ine istediğiniz gibi yerleştirebilirsiniz.

* Script, **`<script>`** tag'inin bulunduğu konumda yer alıyormuş gibi çalışacaktır.

* **External** script'ler içinde **`<script>`** tag'leri bulunamaz. 

```js
// main.js içinde şöyle bir şey yazarsan:
<script>
  alert("Merhaba");
</script>
```

Bu yanlıştır. Çünkü `.js` dosyalarının içinde **HTML tagı olan `<script>` kullanılamaz.**

## External JavaScript Avantajları

Script'leri **external** dosyalara yerleştirmenin bazı avantajları vardır:

- HTML ve kodu ayırır.
- HTML ve JavaScript'in okunmasını ve bakımını kolaylaştırır.
- **Cache**'lenmiş JavaScript dosyaları, sayfa yüklenme süresini hızlandırabilir.

Bir sayfaya birden fazla script dosyası eklemek için birden fazla **`<script>`** tag'i kullanın:

```js
<script src="myScript1.js"></script>  
<script src="myScript2.js"></script>
```


## External References

Bir **external** script 3 farklı şekilde referans verilebilir:

- Tam bir URL ile (tam web adresi)
- Bir dosya yolu ile (örneğin, **`/js/`**)
- Herhangi bir yol belirtmeden

Bu örnek, **`myScript.js`** dosyasına tam bir URL ile bağlantı verir:

```js
<script src="https://www.w3schools.com/js/myScript.js"></script>
```

Bu örnek `myScript.js` dosyasına `link` vermek için bir `file path` kullanır:

```js
<script src="/js/myScript.js"></script>
```

Bu örnekte `myScript.js`'ye bağlantı için herhangi bir path kullanılmamıştır:

```js
<script src="myScript.js"></script>
```


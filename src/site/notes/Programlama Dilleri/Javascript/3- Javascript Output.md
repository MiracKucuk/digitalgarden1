---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/3-javascript-output/","created":"2025-06-12T00:52:26.345+03:00","updated":"2025-06-12T01:35:38.639+03:00"}
---


# JavaScript Output

## JavaScript Display Possibilities

JavaScript verileri farklı şekillerde “görüntüleyebilir”:

* `innerHTML` veya `innerText` kullanarak bir HTML elementine yazma.
* `document.write()` kullanarak HTML çıktısına yazma.
* `window.alert()` kullanarak bir alert box içine yazma.
* `console.log()` kullanarak tarayıcı konsoluna yazma.

## Using innerHTML

* Bir HTML elementine erişmek için `document.getElementById(id)` metodunu kullanabilirsiniz.
* HTML elementini tanımlamak için `id` attribute'unu kullanın.
* Ardından, HTML elementinin HTML içeriğini değiştirmek için `innerHTML` özelliğini kullanın:

```html
<p id="demo"></p>  
  
<script>  
document.getElementById("demo").innerHTML = "<h2>Hello World</h2>";  
</script>
```

Output : 

```html
<p id="demo">
	<h2>Hello World</h2>
</p>
```

Not : Bir HTML elementinin `innerHTML` özelliğini değiştirmek, HTML'de veri görüntülemenin en yaygın yoludur.


## Using innerText

* Bir HTML elementine erişmek için `document.getElementById(id)` metodunu kullanın.

* Ardından, HTML elementinin içindeki metni değiştirmek için `innerText` özelliğini kullanın:

```html
<p id="demo"></p>  
  
<script>  
document.getElementById("demo").innerText = "Hello World";  
</script>
```

Output : 

```html
<p id="demo">Hello World<p>
```

## Using `document.write()`

Test amacıyla `document.write()` fonksiyonunu kullanmak uygundur:

```js
<script>
document.write(5 + 6);
</script>
```

```output
11
```

* `Document.write` dosyasını asla doküman yüklendikten sonra çağırmayın. Tüm dokümanın üzerine yazacaktır.

```html
<!DOCTYPE html>  
<html>  
<body>  
  
<h1>My First Web Page</h1>  
<p>My first paragraph.</p>  
  
<button type="button" onclick="document.write(5 + 6)">Try it</button>  
  
</body>  
</html>
```

![Pasted image 20250612012857.png](/img/user/Pasted%20image%2020250612012857.png)

`Try it` butonuna tıkladığımız az `onclick` event'i tetiklenecek ve `document.write` fonksiyonunu yazacaktır.

```output
11
```

`document.write()` metodu sadece test için kullanılmalıdır.

## Using `window.alert()`

Verileri görüntülemek için bir alert box kullanabilirsiniz:

```js
<script>  
window.alert(5 + 6);  
</script>
```

![Pasted image 20250612013146.png](/img/user/Pasted%20image%2020250612013146.png)

`Window` anahtar sözcüğünü atlayabilirsiniz.

JavaScript'te, window objesi `global` scope objesidir. Bu, değişkenlerin, özelliklerin ve methodların varsayılan olarak window object'e ait olduğu anlamına gelir. Bu aynı zamanda `window` keyword'ünü belirtmenin isteğe bağlı olduğu anlamına gelir:

```js
<script>  
alert(5 + 6);  
</script>
```

## Using `console.log()`

* Debugging amacıyla, verileri görüntülemek için tarayıcıda `console.log()` metodunu çağırabilirsiniz.

* Daha sonraki bir bölümde debugging hakkında daha fazla bilgi edineceksiniz.

```js
<script>  
console.log(5 + 6);  
</script>
```

![Pasted image 20250505002024.png](/img/user/resimler/Pasted%20image%2020250505002024.png)

## JavaScript Print

* JavaScript'in herhangi bir print object'i veya print metodu yoktur.

* JavaScript'ten output device'lara erişemezsiniz.

* Bunun tek istisnası, geçerli window'un içeriğini yazdırmak için tarayıcıda `window.print()` metodunu çağırabilmenizdir.

```js
<button onclick="window.print()">Print this page</button>
```

Butona bastığımız anda `print()` fonksiyonu çalışacak .

![Pasted image 20250505002501.png](/img/user/resimler/Pasted%20image%2020250505002501.png)

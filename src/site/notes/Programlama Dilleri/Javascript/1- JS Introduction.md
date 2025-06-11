---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/1-js-introduction/","created":"2025-06-11T23:47:35.518+03:00","updated":"2025-06-12T00:20:00.611+03:00"}
---

### Neden JavaScript Çalışmalı?

JavaScript, tüm web geliştiricilerinin öğrenmesi gereken 3 dilden biridir:

   1. Web sayfalarının içeriğini tanımlamak için HTML
   2. Web sayfalarının düzenini belirtmek için CSS
   3. Web sayfalarının davranışını programlamak için JavaScript


### JavaScript HTML İçeriğini Değiştirebilir

JavaScript HTML methodlarından biri olan `getElementById()` methodudur. -> [[Bağlantılar/getElementById() syntax\|getElementById() syntax]]

Aşağıdaki örnek, bir HTML element'ini (`id="demo"` olan) "erişir" ve bu element'in içeriğini (**`innerHTML`**) "`Hello JavaScript`" olarak değiştirir:

```js
document.getElementById("id_degeri")
```

Örnek : 

```html
<p id="demo">JavaScript HTML içeriğini değiştirebilir.</p>

<button type="button" onclick="document.getElementById("demo").innerHTML = "Hello Javascript!">Click Me!</button>
```

Output : 

![Pasted image 20250126152950.png](/img/user/resimler/Pasted%20image%2020250126152950.png)

**`id="demo"`**: `id` bir **attribute**’tır. Bu `p` öğesine `"demo"` adını verir.

JavaScript ile bu öğeye erişmek için kullanılır: `document.getElementById("demo")`.

`<button type="button" onclick="...">Click Me!</button>`

 * `type="button"`

	- Bir **attribute**. Tarayıcıya bu butonun bir “form submit” butonu olmadığını söyler.

 * `onclick="..."`

- `onclick` bir **event attribute**’tur. 

	- "Click" event'i meydana geldiğinde çalışacak **JavaScript kodunu** belirtir.

	- Bu bir **HTML Event Attribute** olsa da, JavaScript’te de şu şekilde kullanılabilir:

```js
buttonElement.onclick = function() { ... };  // Anlamadıysanız bu kısmı geçin ileriki bölümlerde açıklanacak.
```


`document.getElementById("demo")`

- **`document`**:
    - Web sayfasının temsilidir (DOM: Document Object Model).
    - Tarayıcı, HTML’yi bu objectle temsil eder.
    - `document` global bir objedir, her HTML sayfasında otomatik vardır.
        
- **`.getElementById("demo")`**:
    - `id`’si `"demo"` olan element'i döndürür.
    - Sonuç: DOM öğesi (örnek: `<p>` tagı) elde edilir.

`.innerHTML = "Hello Javascript!"`

`innerHTML`:

- HTML öğesinin **içeriğini** temsil eder (etiketler dahil metin).
- Bu örnekte, `<p>` tagının içindeki yazı değiştiriliyor.

##### Özetle 

| Kısım                 | Türü               | Anlamı                                       |
| --------------------- | ------------------ | -------------------------------------------- |
| `onclick`             | Event Attribute    | Tıklanınca çalışacak JS kodunu belirler      |
| `document`            | Global Object      | Sayfadaki HTML’ye erişmeyi sağlar            |
| `getElementById(...)` | DOM Methodu        | ID’ye göre HTML öğesi bulur                  |
| `innerHTML`           | Property (özellik) | HTML öğesinin içeriğini okur veya değiştirir |

Not : JavaScript hem çift hem de tek tırnak işaretlerini kabul eder:

```js
document.getElementById('demo').innerHTML = 'Hello javascript'
```


## JavaScript HTML Attribute Değerlerini Değiştirebilir

Bu örnekte JavaScript, bir `<img>` tagının src (kaynak) attribute'unun değerini değiştirir:

```html
<button onclick="document.getElementById('myImage').src='pic_bulbon.gif'">Işığı açın</button>

<img id ="myImage" src="pic_bulboff.gif" style="width:100px">

<button onclick="document.getElementById('myImage').src='pic_bulboff.gif'">Işığı Kapat</button>
```

 
 `.src`

- **`src`** bir HTML `<img>` öğesinin kaynağını (görselin yolunu) belirten attribute’tur.
- JavaScript ile `.src` şeklinde erişilir ve değiştirilebilir.
- Yani bu satır, görselin dosyasını `pic_bulbon.gif` ile değiştirir (ışık yanar).

`<img id="myImage" src="pic_bulboff.gif" ... >`

- Başlangıçta görsel olarak `pic_bulboff.gif` (ışık kapalı hali) yükleniyor.
- `id="myImage"` sayesinde bu element JavaScript tarafından kontrol edilebilir.

|Özellik|Açıklama|
|---|---|
|`.src`|`img` etiketinin görsel yolunu değiştirmek için kullanılır.|
|`<img>`|Görsel öğesidir, JavaScript ile dinamik olarak değiştirilebilir.|

![Pasted image 20250612001143.png](/img/user/Pasted%20image%2020250612001143.png)


## JavaScript HTML Stillerini (CSS) Değiştirebilir

Bir HTML element'inin stilini değiştirmek, bir HTML attribute'unu değiştirmenin bir çeşididir:

```js
document.getElementById("demo").style.fontSize = "35px";
```

## JavaScript HTML Elementlerini Gizleyebilir

HTML elementlerini gizlemek, `display` stilini değiştirerek yapılabilir:

```js
document.getElementById("demo").style.display = "none";
```

## JavaScript HTML Elementlerini Gösterebilir

Gizli HTML elementlerini göstermek, `display` style'ı değiştirerek de yapılabilir:

```js
document.getElementById("demo").style.display = "block";
```



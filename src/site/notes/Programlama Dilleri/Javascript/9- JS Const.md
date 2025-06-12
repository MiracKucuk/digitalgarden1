---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/9-js-const/","created":"2025-06-13T01:24:13.365+03:00","updated":"2025-06-13T01:52:08.693+03:00"}
---


* `const` keyword ES6'da (2015) kullanıma sunulmuştur
* `const` ile tanımlanan değişkenler yeniden bildirilemez
* `const` ile tanımlanan değişkenler Yeniden Atanamaz
* `const` ile tanımlanan değişkenler Blok Kapsamına sahiptir

## Cannot be Reassigned (Yeniden Atanamaz)

`const` keyword'ü ile tanımlanan bir değişken yeniden atanamaz:

```js
const PI = 3.141592653589793;  
PI = 3.14;       // Bu bir hata verecektir  
PI = PI + 10;    // Bu da bir hata verecektir
```

```js
<p id="demo"></p>

<script>
try {
  const PI = 3.141592653589793;
  PI = 3.14;
}
catch (err) {
  document.getElementById("demo").innerHTML = err;
}
</script>
```

```Output
TypeError: Assignment to constant variable.
```

## Must be Assigned (Atanmalıdır)

#### Correct 

```js
const PI = 3.14159265359;
```

#### Incorrect

```js
const PI;  
PI = 3.14159265359;
```

#### JavaScript const ne zaman kullanılır?

Değerin değişmemesi gerektiğini biliyorsanız, her zaman değişkeni **`const`** ile tanımlayın.

**Şu durumlarda `const` kullanın:**

- Yeni bir Array tanımlarken
- Yeni bir Object tanımlarken
- Yeni bir fonksiyon tanımlarken
- Yeni bir **RegExp** tanımlarken

## Constant (Sabit) Objects ve Arrays

**`const`** keyword'ü biraz yanıltıcı olabilir.

Bu keyword, sabit bir değer değil, **sabit bir referans** tanımlar.

Bu yüzden **yapamazsınız:**

- Sabit bir değeri yeniden atayamazsınız
- Sabit bir array'i yeniden atayamazsınız
- Sabit bir object'i yeniden atayamazsınız

Ama **yapabilirsiniz:**

- Sabit bir array'in elemanlarını değiştirebilirsiniz
- Sabit bir object'in özelliklerini değiştirebilirsiniz

## Constant Arrays

Sabit bir array'in elemanlarını değiştirebilirsiniz:

```js
// Sabit (const) bir dizi oluşturabilirsiniz:
const cars = ["Saab", "Volvo", "BMW"];

// Bir elemanı değiştirebilirsiniz:
cars[0] = "Toyota"

// Yeni bir eleman ekleyebilirsiniz:
cars.push("Audi");
```

Sabit (`const`) bir array tanımlamak, array'in elemanlarını değiştirilemez yapmaz.

Ancak array'i yeniden atayamazsınız:

```js
const cars = ["Saab", "Volvo", "BMW"];

cars = ["Toyota", "Volvo", "Audi"]; //Error
```

## Constant(Sabit) Objects

Sabit bir object'in özelliklerini değiştirebilirsiniz:

```js
// Sabit (const) bir object oluşturabilirsiniz:
const car = {type:"Fiat", model"500", color:"white"};

// Bir özelliği değiştirebilirsiniz:
car.color = "red";

// Yeni bir özellik ekleyebilirsiniz:
car.owner = "Johnson";
```

Sabit (const) bir **object** tanımlamak, **object**'in özelliklerini değiştirilemez yapmaz.

```js
const car = {type:"Fiat", model:"500", color:"white"};

car = {type:"Volvo", model:"EX60", color:"red"}; //Error
```

## Difference Between var, let and const

|       | Scope | Redeclare | Reassign | Hoisted | Binds this |
| ----- | ----- | --------- | -------- | ------- | ---------- |
| var   | No    | Yes       | Yes      | Yes     | Yes        |
| let   | Yes   | No        | Yes      | No      | No         |
| const | Yes   | No        | No       | No      | No         |

## What is Good?

- `let` ve `const` blok kapsamına sahiptir.
- `let` ve `const` yeniden deklarasyon yapılamaz.
- `let` ve `const` kullanılmadan önce deklarasyon yapılmalıdır.
- `let` ve `const`, `this`'e bağlanmaz.
- `let` ve `const` hoisted (yukarıya alınmaz) değildir.

## What is Not Good?

- `var` deklarasyon (bildirim) yapılmak zorunda değildir.
- `var` hoisted (yukarıya alınır).
- `var` `this`'e bağlanır.


## Browser Support

* `let` ve `const` keyword'leri Internet Explorer 11 veya önceki sürümlerde desteklenmez.
* Aşağıdaki tablo, tam desteğe sahip ilk tarayıcı sürümlerini tanımlamaktadır:

![Pasted image 20250505221618.png](/img/user/resimler/Pasted%20image%2020250505221618.png)

## Block Scope

* Bir değişkeni `const` ile bildirmek, Blok Scope'u söz konusu olduğunda `let`'e benzer.
* Bu örnekte blok içinde bildirilen x, blok dışında bildirilen x ile aynı değildir:

```js
const x = 10;  
// Burada x 10'dur
  
{  
const x = 2;  
// Burada x 2'dir 
}  
  
// Burada x 10'dur
```

## Redeclaring (Yeniden bildirim)

Bir JavaScript var değişkeninin programın herhangi bir yerinde yeniden bildirilmesine izin verilir:

```js
var x = 2;     // Allowed  
var x = 3;     // Allowed  
x = 4;         // Allowed
```

Mevcut bir `var` veya `let` değişkeninin aynı scope içinde `const` olarak yeniden bildirilmesine izin verilmez:

```js
var x = 2;     // İzin verilir
const x = 2;   // İzin verilmez

{
  let x = 2;     // İzin verilir
  const x = 2;   // İzin verilmez
}

{
  const x = 2;   // İzin verilir
  const x = 2;   // İzin verilmez
}
```

Aynı scope'da mevcut bir `const` değişkenin yeniden atanmasına izin verilmez:

```js
const x = 2;     // İzin verilir
x = 2;           // İzin verilmez
var x = 2;       // İzin verilmez
let x = 2;       // İzin verilmez
const x = 2;     // İzin verilmez

{
  const x = 2;   // İzin verilir
  x = 2;         // İzin verilmez
  var x = 2;     // İzin verilmez
  let x = 2;     // İzin verilmez
  const x = 2;   // İzin verilmez
}
```

Bir değişkenin `const` ile, başka bir scope'da veya başka bir blokta yeniden bildirilmesine izin verilir:

```js
const x = 2;       // İzin verilir

{
  const x = 3;   // İzin verilir
}

{
  const x = 4;   // İzin verilir
}
```

## Hoisting

`var` ile tanımlanan değişkenler en üste (**hoisted**) taşınır ve herhangi bir zamanda başlatılabilir.

Anlamı: Değişkeni, tanımlanmadan önce kullanabilirsiniz.

```js
carName = "Volvo";  
var carName;
```

* `const` ile tanımlanan değişkenler de en üste taşınır (hoisted), ancak başlatılmazlar.
* Anlamı: Bir `const` değişkeni tanımlanmadan önce kullanmak **ReferenceError** (Başvuru Hatası) ile sonuçlanır.

```js
alert (carName);  
const carName = "Volvo"; 
```

```js
<script>

try {
  alert(carName);
  const carName = "Volvo";
}
catch (err) {
  document.getElementById("demo").innerHTML = err;
}

</script>
```

```Output
ReferenceError: Cannot access 'carName' before initialization
```
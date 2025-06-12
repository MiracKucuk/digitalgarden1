---
{"dg-publish":true,"permalink":"/programlama-dilleri/javascript/8-js-let/","created":"2025-06-13T00:19:20.016+03:00","updated":"2025-06-13T01:13:22.731+03:00"}
---

* `let` keyword ES6'da (2015) tanıtıldı
* `let` ile bildirilen değişkenler Blok Kapsamına sahiptir
* `let` ile bildirilen değişkenler kullanılmadan önce bildirilmelidir
* `let` ile bildirilen değişkenler aynı kapsamda yeniden bildirilemez

## Block Scope

* ES6'dan (2015) önce JavaScript'te Blok Scope yoktu.
* JavaScript'te Global Scope ve Function Scope vardı.
* ES6 iki yeni JavaScript keyword'ünü tanıttı: `let` ve `const`.
* Bu iki keyword JavaScript'te Blok Kapsamı sağladı:

### Example

Bir `{ }` bloğu içinde bildirilen değişkenlere blok dışından erişilemez:

```js
{  
  let x = 2;  
}  
// x burada KULLANILAMAZ
```

## Global Scope

* `var` ile bildirilen değişkenler her zaman `Global Scope`'a sahiptir.
* `var` keyword'ü ile bildirilen değişkenler blok kapsamına sahip OLAMAZ:

### Example

Bir `{ }` bloğu içinde `var` ile bildirilen değişkenlere blok dışından erişilebilir:

```js
{  
  var x = 2;  
}  
// x burada kullanılabilir
```

## Cannot be Redeclared (Yeniden İlan Edilemez)

* `let` ile tanımlanan değişkenler yeniden bildirilemez.
* `let` ile bildirilmiş bir değişkeni yanlışlıkla yeniden bildiremezsiniz.

`let` ile bunu yapamazsınız:

```js
let x = "John Doe";  
let x = 0;
```

`var` ile tanımlanan değişkenler yeniden bildirilebilir.

```js
var x = "John Doe"; 
var x = 0;
```

## Redeclaring Variables

* `var` keyword'ünü kullanarak bir değişkeni yeniden bildirmek sorunlara yol açabilir.
* Bir blok içinde bir değişkenin yeniden bildirilmesi, değişkeni blok dışında da yeniden bildirecektir:

```js
var x = 10;  
// Burada x 10'dur
  
{  
var x = 2;  
// Burada x 2'dir
}  
  
// Burada x 2'dir
```

* `let` keyword'ünü kullanarak bir değişkeni yeniden bildirmek bu sorunu çözebilir.
* Bir blok içinde bir değişkeni yeniden bildirmek, blok dışındaki değişkeni yeniden bildirmeyecektir:

```js
let x = 10;  
// Burada x 10'dur
  
{  
let x = 2;  
// Burada x 2'dir
}  
  
// Burada x 10'dur
```

## Var, let ve const arasındaki farklar

|       | Scope | Redeclare | Reassign | Hoisted | Binds this |
| ----- | ----- | --------- | -------- | ------- | ---------- |
| var   | No    | Yes       | Yes      | Yes     | Yes        |
| let   | Yes   | No        | Yes      | No      | No         |
| const | Yes   | No        | No       | No      | No         |

## What is Good?

- **let** ve **const** blok kapsamına sahiptir.
- **let** ve **const** yeniden bildirilemez.
- **let** ve **const** kullanılmadan önce tanımlanmalıdır.
- **let** ve **const**, `this` ile bağlanmaz.
- **let** ve **const** hoist (yukarı taşınma) edilmez.

## What is Not Good?

- **var** değişkeni tanımlanmak zorunda değildir.
- **var** hoist edilir (yukarı taşınır).
- **var**, `this` context'ine bağlanabilir.

## Browser Support

* `let` ve `const` keyword'leri Internet Explorer 11 veya önceki sürümlerde desteklenmez.
* Aşağıdaki tablo, tam desteğe sahip ilk tarayıcı sürümlerini tanımlamaktadır:

| Tarayıcı | Sürüm | Çıkış Tarihi |
| -------- | ----- | ------------ |
| Chrome   | 49    | Mart 2016    |
| Edge     | 12    | Temmuz 2015  |
| Firefox  | 36    | Ocak 2015    |
| Safari   | 11    | Eylül 2017   |
| Opera    | 36    | Mart 2016    |

## Redeclaring (Yeniden bildirim)

Bir JavaScript değişkeninin `var` ile yeniden bildirilmesine programın herhangi bir yerinde izin verilir:

```js
var x = 2;  
// Şimdi x 2
  
var x = 3;  
// Şimdi x 3
```

`let` ile, aynı blok içinde bir değişkenin yeniden bildirilmesine izin VERİLMEZ:

```js
var x = 2;   // İzin verilir
let x = 3;   // İzin verilmez
  
{  
let x = 2;   // İzin verilir
let x = 3;   // İzin verilmez  
}  
  
{  
let x = 2;   // İzin verilir
var x = 3;   // İzin verilmez
}
```

Bir değişkenin başka bir blokta `let` ile yeniden bildirilmesine izin verilir:

```js
let x = 2;   // İzin verilir
  
{  
let x = 3;   // İzin verilir
}  
  
{  
let x = 4;    // İzin verilir
}
```


## Let Hoisting

**`var`** ile tanımlanan değişkenler en üste (kapsamın başına) hoist edilir ve herhangi bir zamanda başlatılabilir (initialize edilebilir).

**Anlamı:** Değişkeni, tanımlanmadan önce kullanabilirsiniz.

### Example

```js
carName = "Volvo";  
var carName;
```

* **`let`** ile tanımlanan değişkenler de bloğun en üstüne `hoist` edilir, ancak başlatılmaz (initialize edilmez).
* **Anlamı:** Bir **`let`** değişkenini tanımlanmadan önce kullanmak **`ReferenceError`** hatasına neden olur.

```js
carName = "Saab";  
let carName = "Volvo";
```

```Output
ReferenceError: Cannot access 'carName' before initialization
```


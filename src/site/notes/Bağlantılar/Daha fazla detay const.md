---
{"dg-publish":true,"permalink":"/baglantilar/daha-fazla-detay-const/"}
---


1. **Tekrar Atama Yapılamaz:**  

* Bir `const` değişkenine atanan değer daha sonra değiştirilemez.  

Örneğin:

![Pasted image 20241206185809.png](/img/user/resimler/Pasted%20image%2020241206185809.png)

2. İlk Değer Atanması Zorunludur
* `const` ile bir değişken tanımlıyorsanız, onu hemen bir değerle başlatmanız gerekir.

![Pasted image 20241206185836.png](/img/user/resimler/Pasted%20image%2020241206185836.png)

3. Referans Tiplerde Değer Değişebilir:
* Eğer bir `const` değişkeni bir nesne (object) veya dizi (array) içeriyorsa, bu nesnenin veya dizinin içeriği değiştirilebilir, ancak değişkenin kendisi başka bir referansa atanamaz.  

Örneğin:

![Pasted image 20241206190000.png](/img/user/resimler/Pasted%20image%2020241206190000.png)

### Neden Kullanılır?

1. **Sabit Değerler İçin:**  
* Değeri değişmemesi gereken bir değişken tanımlamak istiyorsanız `const` kullanılır.  

Örnek:

![Pasted image 20241206190031.png](/img/user/resimler/Pasted%20image%2020241206190031.png)


2. Kodun Okunabilirliğini Artırır:
* Kod yazarken hangi değişkenlerin sabit olduğunu görmek kolaylaşır.

`const` ile `let` ve `var` Karşılaştırması

![Pasted image 20241206190101.png](/img/user/resimler/Pasted%20image%2020241206190101.png)


### Örnek Kullanımlar

Sabit Değerler:

![Pasted image 20241206190128.png](/img/user/resimler/Pasted%20image%2020241206190128.png)


Nesnelerle Kullanım:

![Pasted image 20241206190138.png](/img/user/resimler/Pasted%20image%2020241206190138.png)


Dizilerle Kullanım:

![Pasted image 20241206190203.png](/img/user/resimler/Pasted%20image%2020241206190203.png)


### **Özet:**

`const`, bir değişkene sabit bir değer atanmasını sağlar. Değeri değiştirilemez, ancak nesne ve diziler gibi yapılarla kullanıldığında içerikleri değiştirilebilir. Sabit ve güvenilir değerler için tercih edilir.
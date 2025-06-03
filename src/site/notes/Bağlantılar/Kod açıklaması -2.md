---
{"dg-publish":true,"permalink":"/baglantilar/kod-aciklamasi-2/"}
---

1. Başlangıç ve Modüller

![Pasted image 20241213092054.png](/img/user/resimler/Pasted%20image%2020241213092054.png)

- `#!/usr/bin/python3`: Script'in Python 3 ile çalıştırılması gerektiğini belirtir.
- `import requests`: HTTP isteklerini göndermek ve sunucudan yanıt almak için kullanılan bir modül.
- `import json`: JSON formatındaki verileri işlemek için kullanılan bir modül.
- `import sys`: Sistemle etkileşim için kullanılan bir modül (bu kodda kullanılmamış).
- `from urllib.parse import quote_plus`: URL içindeki özel karakterleri doğru bir şekilde kodlamak için bir yöntem.

2. Hedef Kullanıcı

![Pasted image 20241213092122.png](/img/user/resimler/Pasted%20image%2020241213092122.png)

`target`: SQL injection yaparken kullanılacak hedef kullanıcı adı. Bu, sorgulara dahil edilecek. Hedef kullanıcının adı "maria" olarak ayarlanmış.


### 3. **`oracle` Fonksiyonu**

Bu fonksiyon, SQL injection sorgularının sonucunu kontrol eder.

![Pasted image 20241213092150.png](/img/user/resimler/Pasted%20image%2020241213092150.png)

`q`: Bu fonksiyon, SQL sorgusunun bir parçası olan koşulu (**query**) alır. Bu koşul, SQL sorgusunun `true` ya da `false` döndürmesini sağlar.


3.2. SQL Injection Parametresi Hazırlama

![Pasted image 20241213092229.png](/img/user/resimler/Pasted%20image%2020241213092229.png)

- **`target`**: Kullanıcı adı, sorguya dahil edilir. Bu durumda hedef kullanıcı adı `"maria"`.
- **`quote_plus`**: Bu işlev, URL'deki özel karakterleri kodlar. Örneğin, boşluklar `%20` olarak değiştirilir. SQL injection güvenilir şekilde URL'ye eklenmiş olur.
- **Injection Detayı**:
    - `"maria' AND ({q})-- -"`:
        - Kullanıcı adı `"maria"`'dır.
        - `' AND ({q})-- -` ile kullanıcı adı sorgusunun yanına bir koşul (örneğin `1=1`) eklenir.
        - `-- -` kısmı, SQL'de yorum satırı başlatır ve geri kalan kısmı yok sayar.


3.3. HTTP GET İsteği Gönderme

![Pasted image 20241213092338.png](/img/user/resimler/Pasted%20image%2020241213092338.png)

- `requests.get`: Belirtilen URL'ye bir GET isteği gönderir.
    - **URL Detayı**:
        - `http://192.168.43.37/api/check-username.php`: Hedef API adresi.
        - `u={p}`: SQL injection sorgusunun bulunduğu parametre.
- Örneğin:
    - `u=maria' AND (1=1)-- -`:
        - Bu sorgu, SQL'de **true** dönerse `status: taken` dönecek.
    - `u=maria' AND (1=0)-- -`:
        - Bu sorgu, SQL'de **false** dönerse `status: available` dönecek.

3.4. JSON Yanıtını İşleme

![Pasted image 20241213092421.png](/img/user/resimler/Pasted%20image%2020241213092421.png)

- **`r.text`**: Sunucudan gelen yanıt metnini alır.
- **`json.loads`**: Bu metni JSON formatında ayrıştırır ve bir Python sözlüğüne çevirir


3.5. Boolean Karşılaştırması

![Pasted image 20241213092442.png](/img/user/resimler/Pasted%20image%2020241213092442.png)

- **`j['status']`**: Sunucudan dönen JSON yanıtındaki `status` alanını kontrol eder.
- Eğer `status` değeri `"taken"` ise fonksiyon **True**, aksi halde **False** döner.
- Bu, sorgunun **doğru** ya da **yanlış** döndüğünü anlamayı sağlar.


4. Doğrulama

![Pasted image 20241213092503.png](/img/user/resimler/Pasted%20image%2020241213092503.png)

- **`assert`**: Bir koşulun doğru olduğunu test eder. Koşul yanlışsa bir hata verir ve program durur.
- **`oracle("1=1")`**:
    - SQL sorgusu `1=1` her zaman doğru döneceğinden bu fonksiyon **True** döner.
- **`oracle("1=0")`**:
    - SQL sorgusu `1=0` her zaman yanlış döneceğinden bu fonksiyon **False** döner.

Bu satırlar, fonksiyonun doğru çalışıp çalışmadığını test eder.


### **Özet**

Bu script şunları yapar:

1. Kullanıcı adı olarak `"maria"` belirlenmiş.
2. **oracle** fonksiyonu ile SQL injection koşulu hazırlanır ve hedef API'ye gönderilir.
3. Sunucudan gelen yanıt incelenerek, sorgunun doğru (**True**) mu yanlış (**False**) mı olduğu belirlenir.
4. `assert` ifadeleri ile, bu mekanizmanın doğru çalışıp çalışmadığı kontrol edilir.

Bu yöntem, hedef sistemin SQL injection'a karşı savunmasız olup olmadığını test etmek için kullanılır. **Boolean-based SQL injection** kullanılarak sorgunun durumu (doğru/yanlış) yanıt içeriğinden anlaşılır.



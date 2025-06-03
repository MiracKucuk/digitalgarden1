---
{"dg-publish":true,"permalink":"/baglantilar/kodun-detayli-aciklamasi/"}
---


1. Başlangıç: Gerekli Kütüphaneler ve Ayarlar

![Pasted image 20241206185458.png](/img/user/resimler/Pasted%20image%2020241206185458.png)

### **`const` Nedir?**

**`const`**, JavaScript'te bir **değişken tanımlama** anahtar kelimesidir. Ancak bu değişken **sabit** (constant) olarak tanımlanır, yani bir kez değer atandıktan sonra değeri **değiştirilemez**.

[[Bağlantılar/Daha fazla detay const\|Daha fazla detay const]]



**`require('express')`:**  
Bu komut, Node.js'in _Express_ adında bir kütüphanesini projeye dahil eder.

- **Express nedir?**  
    Express, bir "web uygulama çatısıdır" (framework). Web sunucuları oluşturmayı ve yönetmeyi kolaylaştırır.
- **`const app = express();`:**  
    Bu, Express'i başlatır ve uygulama nesnesi oluşturur. Bu nesne, tüm uygulama ayarlarını ve yönlendirmelerini (routes) yönetir.

![Pasted image 20241206190622.png](/img/user/resimler/Pasted%20image%2020241206190622.png)

Bu satır, Express'e "JSON formatında gelen verileri kabul et ve işle" talimatı verir.

- JSON nedir?  
    **JSON (JavaScript Object Notation)**, verileri saklamak ve aktarmak için kullanılan basit bir format. Örnek:

![Pasted image 20241206190728.png](/img/user/resimler/Pasted%20image%2020241206190728.png)


2. MongoDB Ayarları

![Pasted image 20241206190754.png](/img/user/resimler/Pasted%20image%2020241206190754.png)

**MongoDB nedir?**  
MongoDB, bir veritabanıdır. Verileri JSON benzeri bir yapıda saklar.

- **`MongoClient`:**  
    Bu, MongoDB ile iletişim kurmamızı sağlayan bir araçtır.

![Pasted image 20241206190820.png](/img/user/resimler/Pasted%20image%2020241206190820.png)

**`uri`:**  
Bu, veritabanımıza bağlanmamız için gereken adres.

- `127.0.0.1`: Bilgisayarın kendi adresi (localhost).
- `27017`: MongoDB'nin varsayılan port numarası.
- `test`: Kullanılacak veritabanı ismi.


![Pasted image 20241206190838.png](/img/user/resimler/Pasted%20image%2020241206190838.png)

Bu satırda, **MongoDB ile bağlantı kurmak** için bir istemci (client) oluşturuyoruz.


3. Bir API Yönlendirme (Route) Tanımlama

![Pasted image 20241206190857.png](/img/user/resimler/Pasted%20image%2020241206190857.png)

- **`app.post(...)`:**  
    Bu, bir POST isteğini kabul eden bir yönlendirme oluşturur.
    
    - **POST nedir?**  
        POST, web üzerinden veri göndermek için kullanılan bir HTTP metodudur.
    - **`/api/v1/getUser`:**  
        Bu, isteğin gönderileceği URL'dir.
- **`(req, res) => { ... }`:**  
    Bu bir fonksiyondur. Gelen istekleri (request) işler ve yanıt (response) döner.
    
    - **`req`:** Kullanıcının gönderdiği veri.
    - **`res`:** Kullanıcıya geri gönderilecek yanıt.


4. MongoDB'ye Bağlanma ve Veri Çekme

![Pasted image 20241206190922.png](/img/user/resimler/Pasted%20image%2020241206190922.png)

**`client.connect`:**  
Bu, MongoDB veritabanına bağlanır.

- **`(_, con)`:**
    - İlk parametre hata mesajıdır (`_` ile görmezden geliyoruz).
    - İkinci parametre, bağlantı nesnesidir (`con`).

Veritabanı ve Koleksiyona Erişim:

![Pasted image 20241206190943.png](/img/user/resimler/Pasted%20image%2020241206190943.png)

- **`con.db("example")`:**  
    `example` isimli veritabanına erişiyoruz.
    
- **`.collection("users")`:**  
    `users` isimli koleksiyona (tabloya) erişiyoruz.
    
- **`.find(...)`:**  
    `find`, belirtilen koşullara uyan tüm kayıtları (veri) bulur.
    
    - **`{username: req.body['username']}`:**  
        `req.body['username']` kullanıcının gönderdiği `username` değeridir. Bu sorgu, kullanıcı adı belirtilen değere eşit olan kayıtları bulur.

Bulunan Verileri Listeleme:

![Pasted image 20241206191002.png](/img/user/resimler/Pasted%20image%2020241206191002.png)

- **`cursor.toArray`:**  
    Bulunan kayıtları bir diziye (listeye) dönüştürür.
    
    - **`(_, result)`:**
        - İlk parametre hata mesajıdır (`_` ile görmezden geliyoruz).
        - İkinci parametre, sonuçların listesidir (`result`).
- **`res.send(result)`:**  
    Bulunan verileri kullanıcıya JSON formatında geri gönderir.


5. Sunucuyu Başlatma

![Pasted image 20241206191020.png](/img/user/resimler/Pasted%20image%2020241206191020.png)

- **`app.listen(3000)`:**  
    Sunucu, 3000 numaralı port üzerinden çalışır.
    
    - Port, bir bilgisayarın veri alışverişi için kullandığı kapıdır.
- **`console.log('Listening...')`:**  
    Bu mesaj, sunucunun başarılı bir şekilde çalıştığını belirtir.



### Genel Akış

Kullanıcı, `POST /api/v1/getUser` adresine bir `username` gönderir.  
Örnek:

![Pasted image 20241206191103.png](/img/user/resimler/Pasted%20image%2020241206191103.png)

- Sunucu, MongoDB'ye bağlanır ve `username` eşleşen kayıtları arar.
    
- Bulunan kayıtları listeye dönüştürür ve kullanıcıya geri gönderir.  
    Örnek yanıt:

![Pasted image 20241206191117.png](/img/user/resimler/Pasted%20image%2020241206191117.png)

Bu kod parçası, temel bir **kullanıcı sorgulama API'si** oluşturur. Her adımda veritabanına bağlanma, kullanıcıdan gelen veriyi işleme ve sonuç döndürme gibi işlemleri barındırır.


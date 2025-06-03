---
{"dg-publish":true,"permalink":"/web-penters/1-web-request/"}
---

 `www.hackthebox.com` gibi istenen web sitesine ulaşmak için bir ==Uniform Resource Locator (`URL`)== olarak ==Fully Qualified Domain Name (`FQDN`)== giriyoruz.


## URL

Şimdi bir URL'nin yapısına bakalım:

![Pasted image 20241224195805.png](/img/user/resimler/Pasted%20image%2020241224195805.png)

| **Bileşen**      | **Örnek**             | **Açıklama**                                                                                                                                                                                   |
| ---------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Schema**       | `http://`, `https://` | Client tarafından erişilen protokolü tanımlamak için kullanılır ve bir iki nokta üst üste ve çift eğik çizgi (`://`) ile sona erer.                                                            |
| **User Info**    | `admin:password@`     | Host'a kimlik doğrulaması yapmak için kullanılan kimlik bilgilerini (iki nokta üst üste `:` ile ayrılır) içeren isteğe bağlı bir bileşendir ve host'tan bir at işareti `@` ile ayrılır.        |
| **Host**         | `inlanefreight.com`   | Kaynağın yerini belirten host adını veya bir IP adresini ifade eder.                                                                                                                           |
| **Port**         | `:80`                 | Port, Host'tan bir iki nokta üst üste `:` ile ayrılır. Eğer bir port belirtilmemişse, `http` şeması varsayılan olarak 80 portunu, `https` ise varsayılan olarak 443 portunu kullanır.          |
| **Path**         | `/dashboard.php`      | Erişilen kaynağa işaret eder ve bu bir dosya veya klasör olabilir. Eğer bir path belirtilmemişse, sunucu varsayılan index dosyasını döndürür (ör. `index.html`).                               |
| **Query String** | `?login=true`         | Query stringi bir soru işareti `?` ile başlar ve bir parametre (ör. `login`) ile bir değer (ör. `true`) içerir. Birden fazla parametre, `&` işareti ile ayrılabilir. (`login=true&user=admin`) |
| **Fragments**    | `#status`             | Fragments, client tarafında tarayıcılar tarafından işlenir ve birincil kaynağın içinde bir bölümün (ör. bir başlık veya sayfa bölümü) yerini bulmak için kullanılır.                           |

Bir kaynağa erişmek için tüm bileşenler gerekli değildir. Ana zorunlu alanlar ==scheme== ve ==host=='tur, bunlar olmadan request'te bulunulacak kaynak olmaz.


## HTTP Flow

![Pasted image 20241224201437.png](/img/user/resimler/Pasted%20image%2020241224201437.png)

Bir kullanıcı URL'yi (ör. `inlanefreight.com`) tarayıcıya girdiğinde, tarayıcı bir **DNS** sunucusuna request göndererek domain adını çözümler ve IP adresini alır. Öncelikle tarayıcı, local `/etc/hosts` dosyasını kontrol eder, ardından gerekirse DNS sunucusuna yönelir.

IP adresi alındıktan sonra, tarayıcı varsayılan HTTP portuna (ör. `80`) bir **GET** isteği gönderir ve root (`/`) dizinini talep eder. Web sunucusu isteği alır, varsayılan **`index.html`** dosyasını okur ve bir HTTP yanıtıyla birlikte (`200 OK` status kodu dahil) döndürür. Tarayıcı bu içeriği işler ve kullanıcıya sunar.

## cURL

```shell-session
M1R4CKCK@htb[/htb]$ curl inlanefreight.com

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

cURL'ü bir sayfayı veya dosyayı indirmek ve `-O` bayrağını kullanarak içeriği bir dosyaya çıktı olarak almak için de kullanabiliriz. Eğer çıktı dosyasının ismini belirtmek istiyorsak `-o` bayrağını kullanabilir ve ismi belirtebiliriz. Aksi takdirde, `-O`'yu kullanabiliriz ve cURL aşağıdaki gibi uzaktaki dosya adını kullanacaktır:

```shell-session
M1R4CKCK@htb[/htb]$ curl -O inlanefreight.com/index.html
M1R4CKCK@htb[/htb]$ ls
index.html
```

Gördüğümüz gibi, çıktı bu sefer yazdırılmadı, bunun yerine `index.html` dosyasına kaydedildi. Request işlenirken cURL'ün hala bazı durumları yazdırdığını fark ettik. Durumu `-s` bayrağı ile aşağıdaki gibi susturabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s -O inlanefreight.com/index.html
```

```shell-session
M1R4CKCK@htb[/htb]$ curl -h
Usage: curl [options...] <url>
 -d, --data <data>   HTTP POST data
 -h, --help <category> Get help for commands
 -i, --include       Include protocol response headers in the output
 -o, --output <file> Write to file instead of stdout
 -O, --remote-name   Write output to a file named as the remote file
 -s, --silent        Silent mode
 -u, --user <user:password> Server user and password
 -A, --user-agent <name> Send User-Agent <name> to server
 -v, --verbose       Make the operation more talkative

This is not the full help, this menu is stripped into categories.
Use "--help category" to get an overview of all categories.
Use the user manual `man curl` or the "--help all" flag for all options.
```

Detaylı yardım için `--help all`, belirli bir bayrak için `--help category` kullanın. Dokümantasyon için `man curl`.

----

Soru : Flag'i almak için yukarıdaki alıştırmayı başlatın, ardından yukarıda gösterilen sunucuda '`/download.php`' tarafından döndürülen dosyayı indirmek için cURL kullanın.

Cevap : 

![Pasted image 20241224202430.png](/img/user/resimler/Pasted%20image%2020241224202430.png)

---


# Hypertext Transfer Protocol Secure (HTTPS)

HTTPS, iletişimi şifreleyerek üçüncü tarafların verilere erişimini engeller. Bu nedenle, HTTPS web sitelerinde standart haline gelmiş, HTTP ise aşamalı olarak kaldırılmakta ve tarayıcılar yakında HTTP sitelerine erişimi engelleyecektir.

![Pasted image 20241224202601.png](/img/user/resimler/Pasted%20image%2020241224202601.png)

![Pasted image 20241224202618.png](/img/user/resimler/Pasted%20image%2020241224202618.png)

HTTPS uygulayan web siteleri, URL'lerindeki `https://` (örneğin `https://www.google.com`) ve web tarayıcısının adres çubuğunda URL'nin solundaki kilit simgesiyle tanımlanabilir:

![Pasted image 20241224202642.png](/img/user/resimler/Pasted%20image%2020241224202642.png)

Not: HTTPS protokolü aracılığıyla aktarılan veriler şifrelenmiş olsa da, istek ==clear-text== bir DNS sunucusuyla iletişime geçerse ziyaret edilen URL'yi yine de açığa çıkarabilir. Bu nedenle, şifreli DNS sunucularının (örn. `8.8.8.8` veya `1.1.1.1`) kullanılması veya tüm trafiğin düzgün bir şekilde şifrelendiğinden emin olmak için bir VPN servislerinden yararlanılması önerilir.

## HTTPS Flow

![Pasted image 20241224202749.png](/img/user/resimler/Pasted%20image%2020241224202749.png)

Eğer HTTPS yerine bir web sitesine **`http://`** ile erişmeye çalışırsak ve site HTTPS zorunluluğu uygularsa, tarayıcı önce domain'i çözümler ve hedef web sitesini barındıran web sunucusuna yönlendirme yapar. İlk olarak, şifrelenmemiş HTTP protokolü üzerinden port `80`'e bir request gönderilir. Sunucu bunu algılar ve client'i, ==`301` `Moved Permanently`== response kodu ile güvenli ==HTTPS portu `443`=='e yönlendirir. 

Sonrasında, client (web tarayıcısı) kendisi hakkında bilgi veren bir "==`client hello`==" paketi gönderir. Buna karşılık, sunucu bir "==`server hello`==" ile yanıt verir ve SSL sertifikalarını değiştirmek için bir [anahtar değişimi (key exchange)](https://en.wikipedia.org/wiki/Key_exchange) başlatır. Client bu anahtarı/sertifikayı doğrular ve kendi sertifikasını gönderir. Ardından, şifrelenmiş bir [handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake) gerçekleştirilir ve şifreleme ile veri aktarımının düzgün çalışıp çalışmadığı doğrulanır.

**Not:** Belirli durumlarda, bir saldırgan **`HTTP Downgrade`** saldırısı gerçekleştirebilir. Bu saldırı, HTTPS iletişimini HTTP'ye indirger ve veri clear text olarak aktarılır. Bu, kullanıcı farkında olmadan tüm trafiği saldırganın host'u üzerinden yönlendirmek için bir **Man-In-The-Middle (MITM)** proxy kurulumu ile yapılır. Ancak, modern tarayıcılar, sunucular ve web uygulamaları bu tür saldırılara karşı koruma sağlamaktadır.


## cURL for HTTPS

cURL tüm HTTPS iletişim standartlarını otomatik olarak ele almalı ve güvenli bir handshake gerçekleştirmeli ve ardından verileri otomatik olarak şifrelemeli ve şifresini çözmelidir. Ancak, geçersiz veya güncel olmayan bir SSL sertifikasına sahip bir web sitesiyle iletişime geçersek, cURL varsayılan olarak daha önce bahsedilen MITM saldırılarına karşı koruma sağlamak için iletişime devam etmeyecektir:

```shell-session
M1R4CKCK@htb[/htb]$ curl https://inlanefreight.com

curl: (60) SSL certificate problem: Invalid certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html
...SNIP...
```

Modern web tarayıcıları da aynı şeyi yaparak kullanıcıyı geçersiz SSL sertifikasına sahip bir web sitesini ziyaret etmemesi konusunda uyarır.

Yerel bir web uygulamasını test ederken veya uygulama amacıyla barındırılan bir web uygulamasında böyle bir sorunla karşılaşabiliriz, çünkü bu tür web uygulamaları henüz geçerli bir SSL sertifikası uygulamamış olabilir. cURL ile sertifika kontrolünü atlamak için `-k` bayrağını kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -k https://inlanefreight.com

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

# HTTP Requests and Responses

## HTTP Request

![Pasted image 20241224204742.png](/img/user/resimler/Pasted%20image%2020241224204742.png)

Yukarıdaki görüntü URL'ye yapılan bir HTTP GET isteğini göstermektedir:

- `http://inlanefreight.com/users/login.html`

Herhangi bir HTTP isteğinin ilk satırı 'boşluklarla ayrılmış' üç ana alan içerir:

| **Alan**    | **Örnek**           | **Açıklama**                                                                                                    |
| ----------- | ------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Method**  | `GET`               | HTTP method'u veya verb'ü, gerçekleştirilecek işlem türünü belirtir.                                            |
| **Path**    | `/users/login.html` | Erişilen kaynağın yolunu ifade eder. Bu alan, bir query stringiyle de sonlandırılabilir (ör. `?username=user`). |
| **Version** | `HTTP/1.1`          | Üçüncü ve son alan, kullanılan HTTP versiyonunu belirtir.                                                       |

Sonraki satır kümesi ==`Host`==, ==`User-Agent`==, ==`Cookie`== ve diğer birçok olası header gibi HTTP header değer çiftlerini içerir. Bu header'lar bir request'in çeşitli attribute'lerini belirtmek için kullanılır. Headerlar, sunucunun isteği doğrulaması için gerekli olan yeni bir satırla sonlandırılır. Son olarak, bir request, request body ve data ile bitebilir.

Not: HTTP sürüm `1.X` request'leri açık metin olarak gönderir ve farklı alanları ve farklı istekleri ayırmak için yeni satır karakteri kullanır. HTTP sürüm `2.X` ise istekleri sözlük biçiminde `binary` data olarak gönderir.

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

```
0000 0001 1000 0010 1010 1100 0101 1110 ...
```


## HTTP Response

![Pasted image 20241224205735.png](/img/user/resimler/Pasted%20image%2020241224205735.png)

Bir HTTP response'unun ilk satırı boşluklarla ayrılmış iki alan içerir. Bunlardan ilki ==HTTP sürümü== (örn. `HTTP/1.1`), ikincisi ise ==HTTP response kodudur== (örn. `200 OK`).

Response kodları, daha sonraki bir bölümde tartışılacağı gibi, isteğin durumunu belirlemek için kullanılır. İlk satırdan sonra yanıt, HTTP isteğine benzer şekilde başlıklarını listeler. Hem request hem de response header'ları bir sonraki bölümde ele alınacaktır.

Son olarak, response, headerlarından sonra yeni bir satırla ayrılan bir response body ile bitebilir. Response body genellikle HTML kodu olarak tanımlanır. Ancak, JSON gibi diğer kod türleri, resimler, stil sayfaları veya komut dosyaları gibi web sitesi kaynakları veya hatta web sunucusunda barındırılan PDF belgesi gibi bir belge ile de yanıt verebilir.


## cURL

cURL ile yalnızca URL belirterek response body'sini alabiliyoruz. Ancak, `-v` (verbose) bayrağı ekleyerek HTTP request ve response'un tamamını görüntüleyebiliriz. Bu, özellikle web penetrasyon testleri ve exploit yazarken oldukça kullanışlıdır.

```shell-session
M1R4CKCK@htb[/htb]$ curl inlanefreight.com -v

*   Trying SERVER_IP:80...
* TCP_NODELAY set
* Connected to inlanefreight.com (SERVER_IP) port 80 (#0)
> GET / HTTP/1.1
> Host: inlanefreight.com
> User-Agent: curl/7.65.3
> Accept: */*
> Connection: close
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Unauthorized
< Date: Tue, 21 Jul 2020 05:20:15 GMT
< Server: Apache/X.Y.ZZ (Ubuntu)
< WWW-Authenticate: Basic realm="Restricted Content"
< Content-Length: 464
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>

...SNIP...
```

Request basitçe `Host`, `User-Agent` ve `Accept` başlıklarıyla birlikte `GET / HTTP/1.1` gönderdi. Buna karşılık, HTTP yanıtı `HTTP/1.1 401` `Unauthorized`'ı içeriyordu

Request olduğu gibi, Response da `Date`, `Content-Length` ve `Content-Type` dahil olmak üzere server tarafından gönderilen birkaç header içeriyordu.

Not : `-vvv` bayrağı daha da ayrıntılı bir çıktı gösterir.

## Browser DevTools

Modern tarayıcılardaki geliştirici araçları (**`DevTools`**), web değerlendirmelerinde kritik bir araçtır. `[CTRL+SHIFT+I]` veya `[F12]` ile açılabilen bu araçlar, özellikle **`Network`** sekmesiyle web isteklerini izlemeyi ve analiz etmeyi sağlar. Sayfayı yenileyerek tüm gönderilen istekleri görüntüleyebilirsiniz.

![Pasted image 20241224211847.png](/img/user/resimler/Pasted%20image%2020241224211847.png)

Gördüğümüz gibi, `devtools` bize bir bakışta response durumunu (yani response kodu), kullanılan request yöntemini (GET), istenen kaynağı (yani URL `/` domain) ve istenen yolu gösterir. Ayrıca, web sitesinin çok fazla istek yüklemesi durumunda, belirli bir isteği aramak için Filtre URL'lerini kullanabiliriz.

---

Soru : Request ele geçirilirken kullanılan HTTP methodu nedir? (büyük/küçük harfe duyarlı)

Cevap : `GET`


Soru :  Yukarıdaki sunucuya bir GET isteği gönderin ve sunucuda çalışan Apache sürümünü bulmak için response header'larını okuyun, ardından bunu cevap olarak gönderin. (cevap formatı: `X.Y.ZZ`)

Cevap : `2.4.41`

![Pasted image 20241224212425.png](/img/user/resimler/Pasted%20image%2020241224212425.png)


----

# HTTP Headers

Headerlar, header adından sonra bir iki nokta üst üste (`:`) ile ayrılmış bir veya birden fazla değere sahip olabilir. Headerlar şu kategorilere ayrılabilir:

- **General Headers**
- **Entity Headers**
- **Request Headers**
- **Response Headers**
- **Security Headers**


## General Headers

[General headers](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html), hem HTTP requestlerde hem de response'larda kullanılır. Mesajın içeriğinden ziyade mesajın kendisini tanımlamak için kullanılırlar.

| **Header**     | **Örnek**                             | **Açıklama**                                                                                                                                                                                                                                                                                                                           |
| -------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Date**       | `Date: Wed, 16 Feb 2022 10:38:44 GMT` | Mesajın oluşturulduğu tarih ve saati belirtir. Zamanın standart UTC saat dilimine dönüştürülmesi tercih edilir.                                                                                                                                                                                                                        |
| **Connection** | `Connection: close`                   | Request tamamlandıktan sonra mevcut ağ bağlantısının açık kalıp kalmayacağını belirtir. Bu header için yaygın iki değer `close` ve `keep-alive`'dır. `Close` değeri client veya sunucudan geldiğinde bağlantının sonlandırılmak istendiğini ifade ederken, `keep-alive` bağlantının açık kalmasını ve daha fazla veri almasını sağlar. |


## Entity Headers

General headerlara benzer şekilde, Entity Headers da hem request'de hem de responselarda ortak olabilir. Bu headerlar, bir mesaj tarafından iletilen içeriği (entity) tanımlamak için kullanılır. Genellikle yanıtlarda ve `POST` veya `PUT` isteklerinde bulunurlar.

| **Header**           | **Örnek**                     | **Açıklama**                                                                                                                                                                                                            |
| -------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Content-Type**     | `Content-Type: text/html`     | İletilen kaynağın türünü tanımlar. Değer, tarayıcılar tarafından otomatik olarak client tarafında eklenir ve server response'unda döndürülür. `charset` alanı, kullanılan kodlama standardını belirtir (örneğin UTF-8). |
| **Media-Type**       | `Media-Type: application/pdf` | `Media-Type`, `Content-Type` ile benzerdir ve iletilen veriyi tanımlar. Sunucunun girdiyi nasıl yorumlayacağına karar verirken bu header önemli bir rol oynar. `charset` alanı bu headerla da kullanılabilir.           |
| **Boundary**         | `boundary="b4e4fbd93540"`     | Aynı mesajda birden fazla içerik olduğunda, içerikleri ayıran pointer olarak kullanılır. Örneğin, form verilerinde, bu boundary `--b4e4fbd93540` olarak kullanılır ve formun farklı bölümlerini ayırır.                 |
| **Content-Length**   | `Content-Length: 385`         | Geçirilen entity'nin boyutunu belirtir. Sunucu bu headerı, mesaj body'sinden veri okumak için kullanır ve tarayıcılar ve cURL gibi araçlar tarafından otomatik olarak oluşturulur.                                      |
| **Content-Encoding** | `Content-Encoding: gzip`      | Veriler, iletilmeden önce birden fazla dönüşümden geçebilir. Örneğin, büyük veri miktarları mesaj boyutunu küçültmek için sıkıştırılabilir. Kullanılan kodlama türü, `Content-Encoding` header'ı ile belirtilir.        |

## **Request Headers**  

Client, HTTP işleminde request headerlarını gönderir. Bu headerlar, HTTP request'inde kullanılır ve mesajın içeriğiyle ilgili değildir. Aşağıdaki başlıklar, HTTP requestlerinde yaygın olarak görülür.

| **Header**        | **Örnek**                                | **Açıklama**                                                                                                                                                                                                                                     |
| ----------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Host**          | `Host: www.inlanefreight.com`            | Kaynağa sorgulama yapılan hostu belirtir. Bu bir domain veya IP adresi olabilir. HTTP sunucuları, farklı web sitelerini barındıracak şekilde yapılandırılabilir, bu da host header'ının önemli bir keşif hedefi olmasını sağlar.                 |
| **User-Agent**    | `User-Agent: curl/7.77.0`                | Request yapan client'i tanımlar. Tarayıcı, sürüm ve işletim sistemi gibi client hakkında çok şey ortaya koyabilir.                                                                                                                               |
| **Referer**       | `Referer: http://www.inlanefreight.com/` | Mevcut isteğin nereden geldiğini belirtir. Örneğin, Google arama sonuçlarından bir linke tıklamak, `https://google.com`'ı referer yapar. Bu headera güvenmek tehlikeli olabilir çünkü kolayca manipüle edilebilir.                               |
| **Accept**        | `Accept: */*`                            | Client'in anlayabileceği medya türlerini belirtir. Birden fazla medya türü virgülle ayrılarak belirtilir. `*/ *` değeri, tüm medya türlerinin kabul edildiğini gösterir.                                                                         |
| **Cookie**        | `Cookie: PHPSESSID=b4e4fbd93540`         | Ad-değer çiftlerinden oluşan cookie içerir. Cookie, client tarafında ve sunucuda depolanan, tanımlayıcı olarak kullanılan bir veri parçasıdır. Bu veriler her requestle sunucuya iletilir ve client'in erişimini sürdürür.                       |
| **Authorization** | `Authorization: BASIC cGFzc3dvcmQK`      | Serverın client'i tanımlamasının başka bir yoludur. Başarılı kimlik doğrulamanın ardından, server client'e özel bir token döndürür. Cookies'ten farklı olarak, token'lar sadece client tarafında depolanır ve her requestle sunucuya gönderilir. |

## **Response Headers**  

Response headers, HTTP response'larında kullanılır ve içeriğiyle ilgili değildir. Response header'ları, response'la ilgili daha fazla bağlam sağlamak için kullanılır. Aşağıdaki başlıklar HTTP yanıtlarında yaygın olarak görülür.

| **Header**           | **Örnek**                                   | **Açıklama**                                                                                                                                                                              |
| -------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Server**           | `Server: Apache/2.2.14 (Win32)`             | İsteği işleyen HTTP server'ına dair bilgi içerir. Server hakkında bilgi edinmek ve daha fazla keşif yapmak için kullanılabilir.                                                           |
| **Set-Cookie**       | `Set-Cookie: PHPSESSID=b4e4fbd93540`        | Client tanımlaması için gereken cookie'leri içerir. Tarayıcılar, cookie'leri ayrıştırır ve sonraki istekler için depolar. Bu header, Cookie request header'ı ile aynı formatı takip eder. |
| **WWW-Authenticate** | `WWW-Authenticate: BASIC realm="localhost"` | Client'i, istenen kaynağa erişim için gerekli olan kimlik doğrulama türü hakkında bilgilendirir.                                                                                          |

## **Security Headers**  

Son olarak, **Security Headers**'a sahibiz. Tarayıcı çeşitliliği ve web tabanlı saldırıların artışıyla birlikte, güvenliği artıran headerlar belirlemek gerekli hale geldi. HTTP güvenlik başlıkları, tarayıcının web sitesine erişirken uyması gereken belirli kurallar ve politikaları belirtmek için kullanılan yanıt başlıklarıdır.

| **Header**                    | **Örnek**                                     | **Açıklama**                                                                                                                                                                                                                                                                                 |
| ----------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Content-Security-Policy**   | `Content-Security-Policy: script-src 'self'`  | Web sitesinin dışa aktarılan kaynaklara karşı politikasını belirler. Bu, JavaScript kodu veya script kaynakları olabilir. Bu header, tarayıcıya yalnızca belirli güvenilir alanlardan gelen kaynakları kabul etmesini söyler, böylece Cross-site scripting (XSS) gibi saldırılardan korunur. |
| **Strict-Transport-Security** | `Strict-Transport-Security: max-age=31536000` | Tarayıcının, düz metin HTTP protokolü üzerinden web sitesine erişmesine engel olur ve tüm iletişimin güvenli HTTPS protokolü üzerinden yapılmasını zorunlu kılar. Bu, saldırganların web trafiğini izleyerek şifreler gibi hassas bilgilere erişmesini engeller.                             |
| **Referrer-Policy**           | `Referrer-Policy: origin`                     | Tarayıcının, Referer header'ı üzerinden belirtilen değeri içermesi gerekip gerekmediğini belirler. Bu, web sitesi gezintisi sırasında hassas URL'lerin ve bilgilerin açıklanmasını önlemeye yardımcı olabilir.                                                                               |

**Not:** Bu bölümde yalnızca yaygın olarak görülen HTTP headerlarının küçük bir alt kümesi ele alınmıştır. HTTP iletişimlerinde kullanılabilecek birçok başka context header vardır. Ayrıca, uygulamalar kendi gereksinimlerine göre custom headerlar tanımlayabilir. Standart HTTP headerların tam listesine [buradan](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) ulaşabilirsiniz.


## cURL

Eğer sadece response header'larını görmek istiyorsak, ==`-I`== bayrağını kullanarak `HEAD` request'i gönderebilir ve sadece response header'larını görüntüleyebiliriz. Ayrıca, hem header'ları hem de response body'yi (örneğin HTML kodu) görüntülemek için `-i` bayrağını kullanabiliriz. İkisi arasındaki fark, `-I` bir `HEAD` isteği gönderirken (bir sonraki bölümde göreceğimiz gibi), `-i` belirttiğimiz herhangi bir request gönderir ve header'ları da yazdırır.

```shell-session
M1R4CKCK@htb[/htb]$ curl -I https://www.inlanefreight.com

Host: www.inlanefreight.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/605.1.15 (KHTML, like Gecko)
Cookie: cookie1=298zf09hf012fh2; cookie2=u32t4o3tb3gg4
Accept: text/plain
Referer: https://www.inlanefreight.com/
Authorization: BASIC cGFzc3dvcmQK

Date: Sun, 06 Aug 2020 08:49:37 GMT
Connection: keep-alive
Content-Length: 26012
Content-Type: text/html; charset=ISO-8859-4
Content-Encoding: gzip
Server: Apache/2.2.14 (Win32)
Set-Cookie: name1=value1,name2=value2; Expires=Wed, 09 Jun 2021 10:18:14 GMT
WWW-Authenticate: BASIC realm="localhost"
Content-Security-Policy: script-src 'self'
Strict-Transport-Security: max-age=31536000
Referrer-Policy: origin
```

Headerları görüntülemenin yanı sıra, cURL, daha sonraki bir bölümde göreceğimiz gibi, ==`-H`== bayrağı ile request headerlarını ayarlamamıza da izin verir. `User-Agent` veya `Cookie` header'ları gibi bazı header'ların kendi bayrakları vardır. Örneğin, `User-Agent`'ımızı ayarlamak için `-A`'yı aşağıdaki gibi kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl https://www.inlanefreight.com -A 'Mozilla/5.0'

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```


## Browser DevTools

Son olarak, tarayıcı devtools kullanarak HTTP header'larını nasıl önizleyebileceğimizi görelim. Önceki bölümde yaptığımız gibi, sayfa tarafından yapılan farklı istekleri görmek için **`Network`** sekmesine gidiyoruz. Ayrıntıları görmek için herhangi bir isteğe tıklayabiliriz.

![Pasted image 20241224214532.png](/img/user/resimler/Pasted%20image%2020241224214532.png)

İlk Headers sekmesinde hem HTTP request hem de HTTP response header'larını görüyoruz. `Devtools` başlıkları otomatik olarak bölümler halinde düzenler, ancak ayrıntılarını `raw` formatlarda görüntülemek için ==`Raw`== butonuna tıklayabiliriz. Ayrıca, gelecek bölümde tartışılacağı gibi, request tarafından kullanılan cookie'leri görmek için `Cookies` sekmesini kontrol edebiliriz.

---

Soru : Yukarıdaki sunucu, sayfa yüklendikten sonra flag'ı yükler. Sayfa tarafından hangi requestlerin yapıldığını görmek ve bayrağa yapılan request'i bulmak için tarayıcı devtools'undaki `Network` sekmesini kullanın.

Cevap : 

![Pasted image 20241224214713.png](/img/user/resimler/Pasted%20image%2020241224214713.png)

![Pasted image 20241224214745.png](/img/user/resimler/Pasted%20image%2020241224214745.png)

![Pasted image 20241224214816.png](/img/user/resimler/Pasted%20image%2020241224214816.png)

----

# HTTP Methods and Codes

HTTP, kaynaklara erişim için farklı method'lar destekler ve tarayıcı ile server arasındaki iletişimi belirler. Test ettiğimiz HTTP isteklerinde methodlar (`GET`, `POST` vb.) ve response kodlarını cURL ile **`-v`** kullanarak veya tarayıcı **devtools**'un **`Method`** sütununda görebiliriz.


## Request Methods

| **Yöntem**  | **Açıklama**                                                                                                                                                                                                                                                                            |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GET**     | Belirli bir kaynağı talep eder. Ek veri, URL'deki query stringleriyle sunucuya iletilebilir (ör. `?param=value`).                                                                                                                                                                       |
| **POST**    | Sunucuya veri gönderir. Bu method, metin, PDF veya diğer binary veriler gibi çeşitli türlerde girişleri işleyebilir. Gönderilen veri, headerların ardından request body'sine eklenir. Genellikle `formlar/log` in işlemleri veya `resim/doküman` gibi veriler yüklemek için kullanılır. |
| **HEAD**    | GET request'e yapıldığında dönecek headerların talep eder, ancak request body'sini içermez. Genellikle kaynakların boyutunu kontrol etmek için kullanılır.                                                                                                                              |
| **PUT**     | Serverda yeni kaynaklar oluşturur. Uygun kontroller olmadan kullanılması, zararlı kaynakların yüklenmesine yol açabilir.                                                                                                                                                                |
| **DELETE**  | Web sunucusundaki mevcut bir kaynağı siler. Güvenlik önlemleri olmadan kullanıldığında, kritik dosyaların silinmesine ve Denial of Service  (DoS) saldırılarına neden olabilir.                                                                                                         |
| **OPTIONS** | Sunucu hakkında, kabul ettiği yöntemler gibi bilgileri döner.                                                                                                                                                                                                                           |
| **PATCH**   | Belirtilen kaynağa kısmi değişiklikler uygular.                                                                                                                                                                                                                                         |

Bir methodun kullanılabilirliği, sunucu ve uygulama yapılandırmasına bağlıdır. Tüm HTTP methodlarının listesi için bu [bağlantıyı](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) ziyaret edebilirsiniz.

**Not:** Modern web uygulamaları genellikle **GET** ve **POST** yöntemlerini kullanır. Ancak, `REST API` kullanan uygulamalar genellikle veri güncellemek ve silmek için **PUT** ve **DELETE** methodlarını da kullanır.

## Response Codes

| **Tür**   | **Açıklama**                                                                             |
| --------- | ---------------------------------------------------------------------------------------- |
| **`1xx`** | Bilgilendirme sağlar ve request'in işlenmesini etkilemez.                                |
| **`2xx`** | Request'in başarıyla tamamlandığını belirtir.                                            |
| **`3xx`** | Sunucunun client'e başka bir URL'ye yönlendirdiğini belirtir.                            |
| **`4xx`** | Clientden gelen hatalı requestleri ifade eder (ör. var olmayan bir kaynağı talep etmek). |
| **`5xx`** | HTTP sunucusunun kendisiyle ilgili bir sorun olduğunda döner.                            |

Aşağıda, yukarıdaki türlere ait sık karşılaşılan yanıt kodları yer almaktadır:

| **Kod**                       | **Açıklama**                                                                                                           |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **200 OK**                    | İsteğin başarılı olduğunu belirtir ve response body'sini genellikle talep edilen kaynağı içerir.                       |
| **302 Found**                 | Kullanıcıyı başka bir URL'ye yönlendirir (ör. başarılı bir girişten sonra kullanıcıyı gösterge paneline yönlendirmek). |
| **400 Bad Request**           | Eksik satır sonlandırıcılar gibi hatalı isteklerle karşılaşıldığında döner.                                            |
| **403 Forbidden**             | Client'in kaynağa erişim izni olmadığını belirtir. Ayrıca, sunucu kötü amaçlı giriş tespit ettiğinde de dönebilir.     |
| **404 Not Found**             | Client'in talep ettiği kaynağın sunucuda bulunmadığını belirtir.                                                       |
| **500 Internal Server Error** | Sunucunun isteği işleyemediği durumlarda döner.                                                                        |

Standart HTTP response kodlarının tam listesi için bu [bağlantıyı](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) ziyaret edebilirsiniz. Ayrıca, [Cloudflare](https://support.cloudflare.com/hc/en-us/articles/115003014432-HTTP-Status-Codes) veya [AWS](https://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/APIError.html) gibi sağlayıcılar kendi kodlarını uygulayabilir.


# GET

Herhangi bir URL'yi ziyaret ettiğimizde, tarayıcılarımız bu URL'de barındırılan remote kaynakları elde etmek için varsayılan olarak bir GET isteği gönderir. Tarayıcı talep ettiği ilk sayfayı aldıktan sonra; çeşitli HTTP yöntemlerini kullanarak başka talepler gönderebilir. 


## HTTP Basic Auth

Bu bölümün sonunda bulunan alıştırmayı ziyaret ettiğimizde, bizden bir username ve password girmemizi ister. Kullanıcı kimlik bilgilerini doğrulamak için HTTP parametrelerini kullanan normal oturum açma formlarının aksine (örneğin POST isteği), bu tür kimlik doğrulama, web uygulamasıyla doğrudan etkileşime girmeden belirli bir sayfayı `/` dizini korumak için doğrudan web sunucusu tarafından işlenen temel bir HTTP kimlik doğrulamasını kullanır.

Sayfaya erişmek için, bu durumda `admin:admin` olan geçerli bir kimlik bilgisi çifti girmemiz gerekir:

![Pasted image 20241224224231.png](/img/user/resimler/Pasted%20image%2020241224224231.png)

Kimlik bilgilerini girdikten sonra sayfaya erişim sağlayacağız:

![Pasted image 20241224224254.png](/img/user/resimler/Pasted%20image%2020241224224254.png)

Sayfaya cURL ile erişmeyi deneyelim ve response header'larını görüntülemek için `-i` ekleyelim:

```shell-session
M1R4CKCK@htb[/htb]$ curl -i http://<SERVER_IP>:<PORT>/
HTTP/1.1 401 Authorization Required
Date: Mon, 21 Feb 2022 13:11:46 GMT
Server: Apache/2.4.41 (Ubuntu)
Cache-Control: no-cache, must-revalidate, max-age=0
WWW-Authenticate: Basic realm="Access denied"
Content-Length: 13
Content-Type: text/html; charset=UTF-8

Access denied
```

Gördüğümüz gibi, response body'de ==`Access denied`== ve `WWW-Authenticate` header'ında Basic `realm=“Access denied”` değerlerini alıyoruz, bu da Headers bölümünde tartışıldığı gibi bu sayfanın gerçekten basic HTTP auth kullandığını doğruluyor. Kimlik bilgilerini cURL aracılığıyla sağlamak için aşağıdaki gibi `-u` bayrağını kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -u admin:admin http://<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

Bu kez response olarak sayfayı alırız. Temel HTTP auth kimlik bilgilerini sağlayabileceğimiz başka bir yöntem daha vardır, bu da ilk bölümde tartıştığımız gibi doğrudan URL aracılığıyla (`username:password@URL`) şeklindedir. Aynı şeyi cURL veya tarayıcımızla denersek, sayfaya da erişebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://admin:admin@<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```


## HTTP Authorization Header

Daha önceki cURL komutlarımızdan birine `-v` bayrağını eklersek:

```shell-session
M1R4CKCK@htb[/htb]$ curl -v http://admin:admin@<SERVER_IP>:<PORT>/

*   Trying <SERVER_IP>:<PORT>...
* Connected to <SERVER_IP> (<SERVER_IP>) port PORT (#0)
* Server auth using Basic with user 'admin'
> GET / HTTP/1.1
> Host: <SERVER_IP>
> Authorization: Basic YWRtaW46YWRtaW4=
> User-Agent: curl/7.77.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Mon, 21 Feb 2022 13:19:57 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Cache-Control: no-store, no-cache, must-revalidate
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Pragma: no-cache
< Vary: Accept-Encoding
< Content-Length: 1453
< Content-Type: text/html; charset=UTF-8
< 

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

Basic HTTP auth kullandığımız için, HTTP isteğimizin Authorization headerını `admin:admin`'in base64 kodlu değeri olan Basic `YWRtaW46YWRtaW4=` olarak ayarladığını görüyoruz. Modern bir kimlik doğrulama yöntemi (örneğin `JWT`) kullanıyor olsaydık, Authorization Bearer tipinde olacak ve daha uzun şifrelenmiş bir token içerecekti.

Sayfaya erişmemize izin verip vermediğini görmek için kimlik bilgilerini sağlamadan Authorization'ı manuel olarak ayarlamayı deneyelim. Header'ı `-H` bayrağı ile ayarlayabiliriz ve yukarıdaki HTTP isteğindeki aynı değeri kullanacağız. Birden fazla header belirtmek için `-H` bayrağını birden fazla kez ekleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/

<!DOCTYPE html
<html lang="en">

<head>
...SNIP...
```

Gördüğümüz gibi, bu da bize sayfaya erişim sağladı. Bunlar sayfaya kimlik doğrulaması yapmak için kullanabileceğimiz birkaç yöntemdir. Çoğu modern web uygulaması, kullanıcıların kimliklerini doğrulamak için HTTP POST isteklerini kullanan ve ardından kimlik doğrulamalarını sürdürmek için bir cookie döndüren back-end script dili (örneğin PHP) ile oluşturulmuş oturum açma formlarını kullanır.

## GET Parameters

Kimliğimiz doğrulandıktan sonra, bir arama terimi girebileceğimiz ve eşleşen şehirlerin bir listesini alabileceğimiz bir City Search işlevine erişebiliriz:

![Pasted image 20241224225129.png](/img/user/resimler/Pasted%20image%2020241224225129.png)

Sayfa sonuçlarını döndürürken, bilgileri almak için uzaktaki bir kaynakla iletişim kurulabilir ve ardından bu bilgiler sayfada görüntülenebilir. Bunu doğrulamak için tarayıcı **DevTools**'u açabilir ve **Network** sekmesine geçebilirsiniz. Aynı sekmeye ulaşmak için **`[CTRL+SHIFT+E]**` kısayolunu da kullanabilirsiniz.

![Pasted image 20241224225307.png](/img/user/resimler/Pasted%20image%2020241224225307.png)

![Pasted image 20241224225319.png](/img/user/resimler/Pasted%20image%2020241224225319.png)

Requeste tıkladığımızda, URL'de kullanılan GET parametresi `search=le` ile `search.php`'ye gönderilir. Bu, search fonksiyonunun sonuçlar için başka bir sayfa talep ettiğini anlamamıza yardımcı olur.

Şimdi, arama sonuçlarının tamamını almak için aynı isteği doğrudan `search.php`'ye gönderebiliriz, ancak muhtemelen yukarıdaki ekran görüntüsünde gösterilen HTML düzenine sahip olmadan bunları belirli bir biçimde (örneğin JSON) döndürecektir.

`cURL` ile bir GET isteği göndermek için, GET istekleri parametrelerini URL'ye yerleştirdiğinden, yukarıdaki ekran görüntülerinde görülen URL'nin aynısını kullanabiliriz. Ancak, tarayıcı geliştirme araçları cURL komutunu elde etmek için daha uygun bir yöntem sağlar. İsteğe sağ tıklayabilir ve ==`Copy>Copy as cURL`== seçeneğini seçebiliriz. Daha sonra, kopyalanan komutu terminalimize yapıştırabilir ve çalıştırabiliriz ve tam olarak aynı yanıtı almalıyız:

```shell-session
M1R4CKCK@htb[/htb]$ curl 'http://<SERVER_IP>:<PORT>/search.php?search=le' -H 'Authorization: Basic YWRtaW46YWRtaW4='

Leeds (UK)
Leicester (UK)
```

Not: Kopyalanan komut HTTP isteğinde kullanılan tüm header'ları içerecektir. Ancak, bunların çoğunu kaldırabilir ve yalnızca ==`Authorization`== header'ı gibi gerekli kimlik doğrulama header'larını tutabiliriz.

Aynı isteği doğrudan tarayıcı devtools'u içinde `Copy>Copy as Fetch`'i seçerek de tekrarlayabiliriz. Bu, JavaScript `Fetch kütüphanesini` kullanarak aynı HTTP isteğini kopyalayacaktır. Ardından, `[CTRL+SHIFT+K]` tuşlarına tıklayarak JavaScript `console` sekmesine gidebilir, `Fetch` komutumuzu yapıştırabilir ve isteği göndermek için enter tuşuna basabiliriz:

![Pasted image 20241224225634.png](/img/user/resimler/Pasted%20image%2020241224225634.png)

----

Soru : Yukarıdaki alıştırma yanlış sonuçlar döndürdüğü için bozuk gibi görünüyor. Arama yaptığımızda gönderdiği isteğin ne olduğunu görmek için tarayıcı devtools'u kullanın ve '`flag`' araması yapmak ve bayrağı elde etmek için cURL kullanın.

Cevap : 

![Pasted image 20241224230018.png](/img/user/resimler/Pasted%20image%2020241224230018.png)

---

# POST

Önceki bölümde **GET** isteklerinin arama ve sayfa erişimi gibi işlevler için kullanıldığını gördük. Ancak dosya aktarımı veya URL'den parametre taşımak gerektiğinde, **POST** istekleri tercih edilir.

**POST** isteklerinin avantajları:

1. **Loglama:** Büyük dosyalar aktarılırken sunucu, dosyaları URL gibi loglamak zorunda kalmaz.
2. **Kodlama:** **POST**, veriyi body'de taşıdığı için sadece ayırıcı karakterler kodlanır. Binary veriyi destekler.
3. **Daha Fazla Veri:** URL uzunluğu sınırları nedeniyle (genelde 2.000 karakter altı) fazla veri taşıyamayan **GET**'in aksine, **POST** büyük verilerle uyumludur.

Şimdi **POST** isteklerini okumak ve göndermek için **cURL** ve tarayıcı **DevTools** kullanımı örneklerine bakalım.

## Login Forms

Bu bölümün sonundaki alıştırma GET bölümünde gördüğümüz örneğe benzer. Ancak, web uygulamasını ziyaret ettiğimizde, HTTP basic auth yerine bir PHP giriş formu kullandığını görüyoruz:

![Pasted image 20241224231340.png](/img/user/resimler/Pasted%20image%2020241224231340.png)

`admin:admin`

![Pasted image 20241224231400.png](/img/user/resimler/Pasted%20image%2020241224231400.png)

![Pasted image 20241224231419.png](/img/user/resimler/Pasted%20image%2020241224231419.png)

Request'e tıklayabilir, Request sekmesine ( request body'yi gösterir) tıklayabilir ve ardından ham request verilerini göstermek için `Raw` butonuna tıklayabiliriz. POST istek verisi olarak aşağıdaki verilerin gönderildiğini görüyoruz:

```bash
username=admin&password=admin
```

Elimizdeki request verileriyle, benzer bir isteği cURL ile göndermeyi deneyebilir ve bunun da giriş yapmamıza izin verip vermeyeceğini görebiliriz. Ayrıca, önceki bölümde yaptığımız gibi, istek üzerine sağ tıklayıp `Copy>Copy as cURL` seçeneğini seçebiliriz. Bununla birlikte, POST isteklerini manuel olarak oluşturabilmek önemlidir, bu yüzden bunu yapmayı deneyelim.

Bir POST isteği göndermek için ==`-X POST`== bayrağını kullanacağız. Ardından, POST verilerimizi eklemek için `-d` bayrağını kullanabilir ve yukarıdaki verileri aşağıdaki gibi ekleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/

...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

HTML kodunu incelersek, giriş formu kodunu görmeyeceğiz, ancak arama fonksiyonu kodunu göreceğiz, bu da gerçekten kimliğimizin doğrulandığını gösterir.

İpucu: Birçok giriş formu, kimlik doğrulandıktan sonra bizi farklı bir sayfaya yönlendirir (örn. `/dashboard.php`). Eğer yönlendirmeyi cURL ile takip etmek istiyorsak `-L` bayrağını kullanabiliriz.


## Authenticated Cookies

Kimliğimiz başarıyla doğrulandıysa, tarayıcılarımızın kimlik doğrulamamızı sürdürebilmesi ve sayfayı her ziyaret ettiğimizde giriş yapmamız gerekmemesi için bir cookie almış olmamız gerekir. Kimliği doğrulanmış cookie'mizle birlikte ==`Set-Cookie`== header'ını içermesi gereken response'u görüntülemek için `-v` veya `-i` bayraklarını kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/ -i

HTTP/1.1 200 OK
Date: 
Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1; path=/

...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

Kimliği doğrulanmış cookie'miz ile artık her seferinde kimlik bilgilerimizi vermemize gerek kalmadan web uygulaması ile etkileşime geçebilmeliyiz. Bunu test etmek için, yukarıdaki cookie'i cURL'de `-b` bayrağı ile aşağıdaki gibi ayarlayabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/

...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

Gördüğümüz gibi, gerçekten de kimliğimiz doğrulandı ve search fonksiyonuna ulaştık. Cookie'yi aşağıdaki gibi bir header olarak belirtmek de mümkündür:

```bash
curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/
```

Tarayıcıda deneyelim: Oturumu kapatıp giriş sayfasına dönelim. Ardından **`[SHIFT+F9]`** ile **`DevTools > Storage`** sekmesine gidelim. Sol menüde **`Cookies`** kısmına tıklayıp sitemizi seçerek mevcut cookieleri inceleyebiliriz. Oturum kapatıldıysa, PHP cookie kimliği doğrulanmamış olur ve arama yerine oturum açma formu görünür.

![Pasted image 20241224232126.png](/img/user/resimler/Pasted%20image%2020241224232126.png)

Şimdi, daha önce kimliği doğrulanmış cookie'mizi kullanmayı deneyelim ve kimlik bilgilerimizi vermemize gerek kalmadan içeri girip giremeyeceğimizi görelim. Bunu yapmak için, cookie değerini kendi değerimizle değiştirebiliriz. Aksi takdirde, cookie'ye sağ tıklayıp `Delete All` (Tümünü Sil) seçeneğini seçebilir ve yeni bir cookie eklemek için `+` simgesine tıklayabiliriz. Bundan sonra, `=` (`PHPSESSID`)'den önceki kısım olan cookie adını ve ardından = (`c1nsa6op7vtk7kdis7bcnbadf1`)'den sonraki kısım olan cookie değerini girmemiz gerekir. Ardından, cookie'miz ayarlandıktan sonra sayfayı yenileyebiliriz ve giriş yapmamıza gerek kalmadan, sadece kimliği doğrulanmış bir cookie kullanarak gerçekten kimliğimizin doğrulandığını görürüz:

![Pasted image 20241224232226.png](/img/user/resimler/Pasted%20image%2020241224232226.png)

Gördüğümüz gibi, geçerli bir tanımlama bilgisine sahip olmak birçok web uygulamasında kimlik doğrulaması yapmak için yeterli olabilir. Bu, Cross-Site Scripting gibi bazı web saldırılarının önemli bir parçası olabilir.

## JSON Data

Son olarak, City Search fonksiyonu ile etkileşime girdiğimizde hangi request'lerin gönderildiğini görelim. Bunu yapmak için, tarayıcı devtools'unda `Network` sekmesine gideceğiz ve ardından tüm requestleri temizlemek için çöp kutusu simgesine tıklayacağız. Ardından, hangi isteklerin gönderildiğini görmek için herhangi bir arama sorgusu yapabiliriz:

![Pasted image 20241224232314.png](/img/user/resimler/Pasted%20image%2020241224232314.png)

Gördüğümüz gibi, arama formu `search.php`'ye aşağıdaki verilerle birlikte bir `POST` isteği gönderiyor:

```json
{"search":"london"}
```

POST verileri `JSON` biçiminde görünmektedir, bu nedenle isteğimiz ==`Content-Type` header'ını `application/json`== olarak belirtmiş olmalıdır. İsteğe sağ tıklayıp `Copy>Copy Request Headers`'ı seçerek bunu doğrulayabiliriz:

```bash
POST /search.php HTTP/1.1
Host: server_ip
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://server_ip/index.php
Content-Type: application/json
Origin: http://server_ip
Content-Length: 19
DNT: 1
Connection: keep-alive
Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1
```

Gerçekten de `Content-Type: application/json` headımız var. Bu isteği daha önce yaptığımız gibi tekrarlamaya çalışalım, ancak hem cookie hem de `content-type` header'larını ekleyelim ve isteğimizi `search.php`'ye gönderelim:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X POST -d '{"search":"london"}' -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' -H 'Content-Type: application/json' http://<SERVER_IP>:<PORT>/search.php
["London (UK)"]
```

`fetch` ilede yapılabilir.

![Pasted image 20241224232507.png](/img/user/resimler/Pasted%20image%2020241224232507.png)


---

Soru : Geçerli bir oturum açma yoluyla bir session cookie elde edin ve ardından '`/search.php`' adresine bir JSON POST isteği göndererek bayrağı aramak için cookie'yi cURL ile kullanın

Cevap : 

![Pasted image 20241224232855.png](/img/user/resimler/Pasted%20image%2020241224232855.png)

----

`--compressed` seçeneği, **cURL**'ün sunucuyla iletişim sırasında **sıkıştırılmış veri** transferini desteklemesini sağlar. Bu seçenek, sunucunun **`gzip`**, **`deflate`** veya benzeri sıkıştırma methodlarını kullanarak response gönderdiği durumlarda, cURL'ün bu yanıtı otomatik olarak açmasını (decompress) sağlar.

# CRUD API

Bu bölümde, bir web uygulamasının aynı işlemi API'ler aracılığıyla nasıl gerçekleştirdiğine bakacak ve API endpoint ile doğrudan etkileşim kuracağız.

### API'ler

Çeşitli API türleri vardır. Birçok API bir veritabanıyla etkileşim kurmak için kullanılır, öyle ki API sorgumuzda istenen tabloyu ve istenen satırı belirtebilir ve ardından gereken işlemi gerçekleştirmek için bir HTTP yöntemi kullanabiliriz. Örneğin, örneğimizdeki `api.php` endpoint'i için, veritabanındaki `city` tablosunu güncellemek istiyorsak ve güncelleyeceğimiz satır `london` city adına sahipse, URL aşağıdaki gibi görünecektir:

```bash
curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london ...SNIP...
```


### CRUD

Gördüğümüz gibi, bu tür API'ler aracılığıyla üzerinde işlem yapmak istediğimiz tabloyu ve satırı kolayca belirleyebiliyoruz. Daha sonra bu satır üzerinde farklı işlemler gerçekleştirmek için farklı HTTP methodları kullanabiliriz. Genel olarak API'ler talep edilen veritabanı varlığı üzerinde 4 ana işlem gerçekleştirir:

| İşlem      | HTTP Yöntemi | Açıklama                                             |
| ---------- | ------------ | ---------------------------------------------------- |
| Oluşturma  | POST         | Belirtilen veriyi veritabanı tablosuna ekler.        |
| Okuma      | GET          | Belirtilen varlığı veritabanı tablosundan okur.      |
| Güncelleme | PUT          | Belirtilen veritabanı tablosundaki veriyi günceller. |
| Silme      | DELETE       | Belirtilen satırı veritabanı tablosundan siler.      |
Bu dört işlem temel olarak yaygın olarak bilinen CRUD API'leriyle bağlantılıdır, ancak aynı ilke REST API'lerinde ve diğer bazı API türlerinde de kullanılır. Elbette, tüm API'ler aynı şekilde çalışmaz ve kullanıcı erişim kontrolü hangi eylemleri gerçekleştirebileceğimizi ve hangi sonuçları görebileceğimizi sınırlayacaktır.


## Read

Bir API ile etkileşime girerken yapacağımız ilk şey veri okumaktır. Daha önce de belirtildiği gibi, API'den sonra tablo adını (örn. `/city`) ve ardından arama terimimizi (örn. `/london`) aşağıdaki gibi belirtebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://<SERVER_IP>:<PORT>/api.php/city/london

[{"city_name":"London","country_name":"(UK)"}]
```

Sonucun bir JSON stringi olarak gönderildiğini görüyoruz. JSON formatında düzgün bir şekilde formatlamak için, çıktıyı düzgün bir şekilde biçimlendirecek olan `jq` yardımcı programına aktarabiliriz. Gereksiz cURL çıktılarını da `-s` ile aşağıdaki gibi susturacağız:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq

[
  {
    "city_name": "London",
    "country_name": "(UK)"
  }
]
```

Gördüğümüz gibi, çıktıyı güzel bir şekilde biçimlendirilmiş bir çıktı olarak aldık. Ayrıca bir search terimi sağlayabilir ve eşleşen tüm sonuçları alabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/le | jq

[
  {
    "city_name": "Leeds",
    "country_name": "(UK)"
  },
  {
    "city_name": "Dudley",
    "country_name": "(UK)"
  },
  {
    "city_name": "Leicester",
    "country_name": "(UK)"
  },
  ...SNIP...
]
```

Son olarak, tablodaki tüm girdileri almak için boş bir dize geçebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/ | jq

[
  {
    "city_name": "London",
    "country_name": "(UK)"
  },
  {
    "city_name": "Birmingham",
    "country_name": "(UK)"
  },
  {
    "city_name": "Leeds",
    "country_name": "(UK)"
  },
  ...SNIP...
]
```


## Create

Yeni bir input eklemek için, önceki bölümde gerçekleştirdiğimize oldukça benzeyen bir HTTP POST isteği kullanabiliriz. JSON verilerimizi basitçe POST edebiliriz ve tabloya eklenecektir. Bu API JSON verilerini kullandığından, `Content-Type` header'ını da aşağıdaki gibi JSON olarak ayarlayacağız:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ -d '{"city_name":"HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

Şimdi, başarıyla eklenip eklenmediğini görmek için eklediğimiz şehrin (`HTB_City`) içeriğini okuyabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/HTB_City | jq

[
  {
    "city_name": "HTB_City",
    "country_name": "HTB"
  }
]
```

Gördüğümüz gibi, daha önce var olmayan yeni bir city yaratıldı.

## Update

Artık API'ler aracılığıyla inputları nasıl okuyacağımızı ve yazacağımızı bildiğimize göre, şimdiye kadar kullanmadığımız diğer iki HTTP yöntemini tartışmaya başlayalım: `PUT` ve `DELETE`. Bölümün başında belirtildiği gibi, `PUT API` girdilerini güncellemek ve ayrıntılarını değiştirmek için kullanılırken, `DELETE` belirli bir varlığı kaldırmak için kullanılır.

Not: API girdilerini güncellemek için `PUT` yerine HTTP `PATCH` methodu da kullanılabilir. Kesin olmak gerekirse, `PATCH` bir girdiyi kısmen güncellemek için kullanılırken (yalnızca bazı verilerini değiştirmek “örneğin yalnızca city_name”), `PUT` tüm girdiyi güncellemek için kullanılır. Sunucu tarafından hangisinin kabul edildiğini görmek için `HTTP OPTIONS` yöntemini de kullanabilir ve ardından buna göre uygun yöntemi kullanabiliriz. Bu bölümde, kullanımları oldukça benzer olmasına rağmen `PUT` yöntemine odaklanacağız.

Bu durumda `PUT` kullanmak `POST`'a oldukça benzerdir, tek fark URL'de düzenlemek istediğimiz varlığın adını belirtmek zorunda olmamızdır, aksi takdirde API hangi varlığı düzenleyeceğini bilemez. Dolayısıyla, tek yapmamız gereken URL'de `city` adını belirtmek, request methodunu `PUT` olarak değiştirmek ve `POST` ile yaptığımız gibi JSON verilerini aşağıdaki gibi sağlamaktır:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london -d '{"city_name":"New_HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

Yukarıdaki örnekte ilk olarak şehrimiz olarak `/city/london` belirttiğimizi ve request verisinde `“city_name”: “New_HTB_City”` içeren bir JSON string'i ilettiğimizi görüyoruz. Dolayısıyla, `londra` şehri artık mevcut olmamalı ve `New_HTB_City` adında yeni bir şehir mevcut olmalıdır. Doğrulamak için ikisini de okumayı deneyelim:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq
```

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City | jq

[
  {
    "city_name": "New_HTB_City",
    "country_name": "HTB"
  }
]
```

Gerçekten de, eski şehir adını yeni şehirle başarıyla değiştirdik.

Not: Bazı API'lerde, `Update` işlemi yeni girişler oluşturmak için de kullanılabilir. Temel olarak, verilerimizi göndeririz ve eğer mevcut değilse, onu oluşturur. Örneğin, yukarıdaki örnekte, londra şehrine sahip bir giriş mevcut olmasa bile, ilettiğimiz ayrıntılarla yeni bir giriş oluşturacaktır. Ancak bizim örneğimizde durum böyle değildir. 

## DELETE

Son olarak, bir şehri okumak kadar kolay olan bir şehri silmeyi deneyelim. API için sadece şehir adını belirtiriz ve `HTTP DELETE` yöntemini kullanırız ve aşağıdaki gibi girişi siler:

```shell-session
M1R4CKCK@htb[/htb]$ curl -X DELETE http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City
```

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City | jq
[]
```

Gördüğümüz gibi, New_HTB_City'yi sildikten sonra, okumayı denediğimizde boş bir dizi alıyoruz, yani artık mevcut değil.

Bu sayede, cURL aracılığıyla 4 CRUD işleminin tamamını gerçekleştirebiliyoruz. Gerçek bir web uygulamasında, bu tür eylemlere tüm kullanıcılar için izin verilmeyebilir veya herhangi birinin herhangi bir girişi değiştirebilmesi veya silebilmesi bir güvenlik açığı olarak kabul edilir. Her kullanıcı neleri okuyabileceği veya yazabileceği konusunda belirli ayrıcalıklara sahip olacaktır; burada write veri ekleme, değiştirme veya silme anlamına gelir. API'yi kullanacak kullanıcımızın kimliğini doğrulamak için, daha önceki bir bölümde yaptığımız gibi, bir cookie veya bir authorization header (örn. JWT) iletmemiz gerekir. Bunun dışında, işlemler bu bölümde uyguladıklarımıza benzer.

---

Soru : İlk olarak, herhangi bir şehrin adını ''flag'' olarak güncellemeyi deneyin. Ardından, herhangi bir şehri silin. Bunu yaptıktan sonra, bayrağı almak için 'bayrak' adlı bir şehri arayın.

Cevap : 

![Pasted image 20241224235334.png](/img/user/resimler/Pasted%20image%2020241224235334.png)

![Pasted image 20241224235837.png](/img/user/resimler/Pasted%20image%2020241224235837.png)

----

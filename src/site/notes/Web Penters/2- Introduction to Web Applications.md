---
{"dg-publish":true,"permalink":"/web-penters/2-introduction-to-web-applications/","created":"2025-02-11T02:14:38.450+03:00","updated":"2025-06-04T01:07:03.752+03:00"}
---


**Web 1.0**: Statik HTML sayfalarından oluşan, kullanıcıların yalnızca içerik tüketebildiği, etkileşimin minimum olduğu internet dönemidir.

**Web 2.0**: Kullanıcıların içerik oluşturabildiği, paylaşabildiği ve etkileşimde bulunabildiği, dinamik ve sosyal medya odaklı internet dönemidir.


## Web Applications vs. Native Operating System Applications

**Web applications**, platformdan bağımsızdır ve **browser** üzerinden çalıştığı için **end user**’ın sistemine yüklenmek zorunda değildir. **Web applications**, tek bir **webserver** üzerinden güncellenebilir ve her kullanıcı aynı sürümü kullanır, bu da bakım maliyetlerini azaltır. **Native OS applications**, **native OS** kütüphanelerini ve donanımı doğrudan kullanabildiği için daha hızlı ve güçlüdür. **Hybrid** ve **progressive web applications**, modern **frameworks** ile **native OS** kaynaklarını kullanarak daha hızlı ve yetenekli hale gelmektedir.


## Web Application Distribution

Açık kaynaklı **web applications**, organizasyonların ihtiyaçlarına göre özelleştirilebilir ve yaygın örnekleri **WordPress, OpenCart, Joomla**’dır. Kapalı kaynaklı **web applications** ise genellikle belirli bir organizasyon tarafından geliştirilip satılır veya abonelik modeliyle kullanılır; yaygın örnekleri **Wix, Shopify, DotNetNuke**’tir.


## Security Risks of Web Applications

**Web application attacks**, geniş **attack surface** sundukları için büyük bir tehdit oluşturur ve ciddi veri kayıplarına yol açabilir. **Web applications**, hassas veriler içeren **servers** ve **databases** ile bağlantılı olduğundan, güvenlik testleri yapılıp **vulnerabilities** hızlıca **patch** edilmelidir. **Web application penetration testing**, güvenli kodlama ile desteklenmeli ve düzenli olarak yapılmalıdır. Test süreci, **front end** bileşenleri (**HTML, CSS, JavaScript**) analiz ederek başlar, ardından **webserver** teknolojileri incelenerek **exploitable flaws** araştırılır. Maksimum kapsama için testler hem **unauthenticated** hem de **authenticated** perspektiften gerçekleştirilir.,


## Attacking Web Applications

**Web applications**, geniş bir **attack surface** sunduğundan kritik hedeflerdir ve basit kod değişiklikleri ciddi **vulnerabilities** oluşturabilir. **SQL injection**, **file upload** ve **file inclusion** gibi açıklar, **remote code execution** ve hassas verilere erişimle sonuçlanabilir. **Active Directory authentication** kullanan **web applications**, **password spraying attack** için veri sızdırabilir ve kurumsal **network environment** içinde bir **foothold** elde edilmesine yol açabilir. Güçlü bir **web security** bilgisi, **penetration tester**’ları rakiplerinden ayırarak daha fazla **flaws** tespit etmelerini sağlar.

| **Flaw**                                      | **Real-world Scenario**                                                                                                                                                                                                                                                                                                                                                                                                          |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SQL injection**                             | **Active Directory** kullanıcı adlarını alıp **VPN** veya e-posta portalına **password spraying attack** yapmak.                                                                                                                                                                                                                                                                                                                 |
| **File Inclusion**                            | Kaynak kodu okuyarak gizli bir sayfa veya dizin bulmak, bu da **remote code execution** için kullanılabilir.                                                                                                                                                                                                                                                                                                                     |
| **Unrestricted File Upload**                  | Kullanıcıya herhangi bir dosya türünü yüklemeye izin veren bir **web application** (sadece resimler değil). Bu, zararlı kod yükleyerek **web application server** üzerinde tam kontrol sağlamaya olanak verir.                                                                                                                                                                                                                   |
| **Insecure Direct Object Referencing (IDOR)** | **Broken access control** ile birleştiğinde, başka bir kullanıcının dosyalarına veya fonksiyonlarına erişmek için kullanılabilir. Örneğin, kullanıcı profilini düzenlerken `/user/701/edit-profile` gibi bir sayfaya gitmek. Eğer `701`’i `702`’ye değiştirirsek, başka bir kullanıcının profilini düzenleyebiliriz!                                                                                                             |
| **Broken Access Control**                     | **Account registration** fonksiyonu kötü tasarlandığında, kullanıcı **privilege escalation** gerçekleştirebilir. Örneğin, yeni kullanıcı kaydederken gönderilen POST isteği `username=bjones&password=Welcome1&email=bjones@inlanefreight.local&roleid=3`. Eğer **`roleid`** parametresini `0` veya `1` olarak değiştirirsek, admin kullanıcısı kaydedebiliriz ve uygulamanın pek çok istenmeyen özelliğine erişim sağlanabilir. |

# Web Application Layout

|**Category**|**Description**|
|---|---|
|**Web Application Infrastructure**|**Web application**'ın beklenen şekilde çalışabilmesi için gereken bileşenlerin yapısını, örneğin **database**'i açıklar. **Web application**, ayrı bir sunucuda çalışacak şekilde kurulabileceği için, hangi **database server**'ına erişmesi gerektiğini bilmek önemlidir.|
|**Web Application Components**|**Web application**'ı oluşturan bileşenler, **web application**'ın etkileşimde bulunduğu tüm bileşenleri temsil eder. Bunlar aşağıdaki üç alana ayrılır: **UI/UX**, **Client**, ve **Server** bileşenleri.|
|**Web Application Architecture**|**Architecture**, farklı **web application** bileşenleri arasındaki tüm ilişkileri kapsar.|

## Web Application Infrastructure (Altyapısı)


Web uygulamaları, farklı altyapı kurulumlarına sahip olabilir. En yaygın modeller şunlardır:

**Client-Server**: Bir **server**, **web application**'ı **client**'a dağıtır. Kullanıcı, web adresini ziyaret ettiğinde, tarayıcı **server**'a HTTP isteği gönderir ve işlem tamamlandığında sonuçları geri alır.

![Pasted image 20250211133756.png](/img/user/resimler/Pasted%20image%2020250211133756.png)

**One Server**: Tüm **web application** ve bileşenleri (database dahil) tek bir **server**'da barındırılır. Bu tasarım basittir ama güvenlik risklidir. **Server**'ın ihlali tüm verileri tehlikeye atar.

![Pasted image 20250211133806.png](/img/user/resimler/Pasted%20image%2020250211133806.png)

**Many Servers - One Database**: **Database**, ayrı bir **server**'da barındırılır. **Web application**'lar, bu **database**'i kullanarak veri depolar ve alır. Bu tasarımda segmentasyon sağlanır; bir **webserver** ihlal edilirse diğerleri etkilenmez.

![Pasted image 20250211133820.png](/img/user/resimler/Pasted%20image%2020250211133820.png)

**Many Servers - Many Databases**: Her **web application**'ın verisi, ayrı bir **database**'te barındırılır. Bu tasarım, yedeklilik sağlar ve kesinti süresini azaltır, ayrıca güvenlik açısından daha sağlamdır.

![Pasted image 20250211133827.png](/img/user/resimler/Pasted%20image%2020250211133827.png)

Bu tasarım aynı zamanda yedekleme amacıyla da yaygın olarak kullanılır, böylece herhangi bir web sunucusu veya veritabanı çevrimdışı olursa, kesinti süresini mümkün olduğunca azaltmak için yerine bir yedekleme çalışır. Uygulanması daha zor olsa ve uygun şekilde çalışması için load balancer gibi araçlar gerektirse de, bu mimari uygun erişim kontrol önlemleri ve uygun asset segmentasyonu sayesinde güvenlik açısından en iyi seçeneklerden biridir.

Bu modellerin yanı sıra, [serverless](https://aws.amazon.com/lambda/serverless-architectures-learn-more) web uygulamaları veya [mikroservisler](https://aws.amazon.com/microservices) kullanan web uygulamaları gibi başka web uygulama modelleri de mevcuttur.

**Serverless**: Sunucu yönetimiyle ilgilenmeden sadece uygulama kodunu çalıştırmanıza olanak sağlayan bir mimaridir. Sunucular, altyapı sağlayıcı tarafından yönetilir ve kullanıcı yalnızca ihtiyaç duyduğu kadar kaynak kullanır.

**Mikroservisler**: Uygulamanın işlevlerini bağımsız, küçük hizmetlere ayıran bir mimaridir. Her mikroservis kendi işlevini yerine getirir ve birbirinden bağımsız olarak geliştirilip dağıtılabilir.

**Web Application Components**:

1.**Client**
2. **Server**
	- **Webserver**
	- **Web Application Logic**
	- **Database**
3. **Services (Microservices)**
	- **3rd Party Integrations**
	- **Web Application Integrations**
4. **Functions (Serverless)**

Bu bileşenler, farklı altyapı ve güvenlik ihtiyaçlarına göre düzenlenebilir.


## Web Application Architecture

Web uygulamasının bileşenleri üç farklı katmana (diğer adıyla Üç Katmanlı Mimari) ayrılır.

| Katman             | Açıklama                                                                                                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presentation Layer | Uygulama ve sistemle iletişimi sağlayan UI işlem bileşenlerinden oluşur. Müşteri, bu katmana web tarayıcısı aracılığıyla erişebilir ve HTML, JavaScript ve CSS formatında geri döner. |
| Application Layer  | Bu katman, tüm client isteklerinin (web istekleri) doğru şekilde işlenmesini sağlar. Yetkilendirme, ayrıcalıklar ve client'e iletilen veriler gibi çeşitli kriterler kontrol edilir.  |
| Data Layer         | Data katmanı, application katmanı ile yakın bir şekilde çalışarak, gerekli verilerin nerede depolandığını ve nasıl erişileceğini belirler.                                            |

Bir web uygulaması mimarisi örneği aşağıdaki gibi görünebilir:

![Pasted image 20250211135633.png](/img/user/resimler/Pasted%20image%2020250211135633.png)

Ayrıca, bazı web sunucuları IIS ISAPI veya PHP-CGI gibi işletim sistemi çağrılarını ve programlarını çalıştırabilir.

#### Microservices

Microservices'i, çoğu durumda sadece bir görev için programlanmış bağımsız web uygulaması bileşenleri olarak düşünebiliriz. Örneğin, bir online mağaza için temel görevleri aşağıdaki bileşenlere ayırabiliriz:

- Registration
- Search
- Payments
- Ratings
- Reviews


Bu bileşenler, **client** ve birbirleriyle iletişim kurar. Bu **microservices**'ler arasındaki iletişim **stateless**'tır, yani **request** ve **response** bağımsızdır. Bunun nedeni, depolanan verilerin ilgili **microservices**'lerden ayrı olarak saklanmasıdır. **Microservices** kullanımı, tek bir iş hedefine odaklanmış çeşitli otomatikleştirilmiş işlevlerin bir koleksiyonu olarak inşa edilmiş [hizmet odaklı mimari (**SOA**)](https://en.wikipedia.org/wiki/Service-oriented_architecture) olarak kabul edilir. Bununla birlikte, bu **microservices**'ler birbirlerine bağımlıdır.

Bir diğer önemli ve verimli **microservice** bileşeni, farklı programlama dillerinde yazılabilmeleridir ve yine de birbirleriyle etkileşimde bulunabilirler. **Microservices**, uygulamaların daha kolay ölçeklenmesine ve daha hızlı geliştirilmesine olanak tanır, bu da yenilikleri teşvik eder ve yeni özelliklerin pazara sunulmasını hızlandırır. **Microservices**'in bazı faydaları şunlardır:

- Agility
- Flexible scaling
- Easy deployment
- Reusable code
- Resilience

Bu [AWS beyaz kitabı](https://d1.awsstatic.com/whitepapers/microservices-on-aws.pdf), **microservice** implementasyonuna dair mükemmel bir genel bakış sunmaktadır.


#### Serverless

AWS, GCP, Azure gibi **cloud** sağlayıcıları, **serverless** mimarileri sunmaktadır. Bu platformlar, **application framework** sağlayarak, **serverless** web uygulamaları inşa etmenizi sağlar ve bu sayede sunucular hakkında endişelenmenize gerek kalmaz. Bu web uygulamaları, daha sonra **stateless** hesaplama konteynerlerinde çalışır (örneğin, Docker). Bu tür bir mimari, bir şirkete, altyapıyı yönetme zorunluluğu olmadan uygulamalar ve hizmetler inşa etme ve dağıtma esnekliği sunar; tüm sunucu yönetimi **cloud** sağlayıcısı tarafından yapılır, böylece uygulamaları ve veritabanlarını çalıştırmak için gereken sunucuları sağlama, ölçeklendirme ve bakım yapma ihtiyacı ortadan kalkar.

**Serverless** hesaplama ve çeşitli kullanım senaryoları hakkında daha fazla [bilgi](https://aws.amazon.com/serverless) edinebilirsiniz.


## Architecture Security

Web uygulamalarının mimarisi ve tasarımını anlamak, penetrasyon testi için önemlidir. Zafiyetler her zaman programlama hatasından değil, tasarım hatalarından da kaynaklanabilir. Örneğin, doğru erişim kontrolü (RBAC gibi) uygulanmamışsa, kullanıcılar yetkisiz yönetici özelliklerine veya başkalarının özel bilgilerine erişebilir. Bu tür sorunlar tasarım değişiklikleri gerektirir ve bu değişiklikler maliyetli olabilir. Ayrıca, **back-end server**'a hakim olunduğunda veritabanının ayrı bir sunucuda bulunması, birden fazla veritabanının kullanılıyor olabileceğini gösterebilir. Bu nedenle, güvenlik her aşamada dikkate alınmalı ve penetrasyon testleri sürekli yapılmalıdır.


# Front End vs. Back End

**Front end** ve **back end** web geliştirme ile **Full Stack** web geliştirme, web uygulama geliştirmesinin farklı alanlarını ifade eder. Her biri farklı işlevler üstlenir ve birbirinden ayrıdır.


## Front End

**Front end**, kullanıcının web tarayıcısı üzerinden gördüğü bileşenleri içerir ve genellikle HTML, CSS ve JavaScript'ten oluşur. Bu bileşenler, tarayıcılar tarafından gerçek zamanlı yorumlanır.

![Pasted image 20250211141206.png](/img/user/resimler/Pasted%20image%2020250211141206.png)

**Front end** web geliştirme, kullanıcının etkileşimde bulunduğu tüm öğeleri içerir, bunlar HTML, CSS ve JavaScript ile oluşturulur ve tarayıcıda gerçek zamanlı yorumlanır. Modern web uygulamaları, **front end**'in her cihaz ve tarayıcıda uyumlu olmasını gerektirir. İyi optimize edilmemiş **front end**, web uygulamasını yavaşlatabilir. Ayrıca, **front end** geliştirme, görsel tasarım, **UI** ve **UX** tasarımı gibi görevleri de kapsar.

```html
<p><strong>Welcome to Hack The Box Academy</strong><strong></strong></p>
<p></p>
<p><em>This is some italic text.</em></p>
<p></p>
<p><span style="color: #0000ff;">This is some blue text.</span></p>
<p></p>
<p></p>
```



## Back End

Bir web uygulamasının **back end**'i, uygulamanın doğru çalışmasını sağlayan tüm fonksiyonları işler. Kullanıcılar tarafından görülemez ve doğrudan etkileşime girilemez, ancak **back end** olmadan web sitesi sadece statik sayfalardan oluşur.


Web uygulamaları için dört ana back end bileşeni vardır:


|Bileşen|Açıklama|
|---|---|
|**Back end Sunucuları**|Diğer tüm bileşenleri barındıran donanım ve işletim sistemi olup genellikle Linux, Windows gibi işletim sistemlerinde veya Konteynerlerde çalıştırılır.|
|**Web Sunucuları**|Web sunucuları HTTP isteklerini ve bağlantılarını yönetir. Bazı örnekler Apache, NGINX ve IIS'tir.|
|**Veritabanları**|Veritabanları (DB'ler) web uygulaması verilerini depolar ve alır. **Relational Databases** (İlişkisel Veritabanları) örnekleri MySQL, MSSQL, Oracle, PostgreSQL iken, **Non-Relational Databases** (İlişkisel Olmayan Veritabanları) örnekleri NoSQL ve MongoDB'dir.|
|**Development Frameworks**|**Development Frameworks**, web uygulamasının temelini geliştirmek için kullanılır. Bazı popüler **framework**'ler Laravel (PHP), ASP.NET (C#), Spring (Java), Django (Python) ve Express (NodeJS JavaScript) gibi **framework**'lerdir.|

![Pasted image 20250211141620.png](/img/user/resimler/Pasted%20image%2020250211141620.png)

Her bir **back end** bileşeni, **Docker** gibi hizmetlerle izole edilmiş sunucularda veya konteynerlerde barındırılabilir. Bu, her bileşenin potansiyel zafiyetlerden izole edilmesini sağlar. Ayrıca, her bileşen ayrı sunucularda da barındırılabilir, ancak bu daha fazla kaynak ve zaman gerektirir.

**Back end** bileşenlerinin temel görevleri şunlardır:

- **Back end** mantığını ve hizmetlerini geliştirmek
- **Back end** veritabanını yönetmek
- Web uygulamasında kullanılacak kütüphaneleri geliştirmek
- **Front end** ile iletişim için **API**'leri uygulamak
- Uzak sunucuları ve **cloud** hizmetlerini entegre etmek


## Securing Front/Back End

Back end koduna erişimimiz olmasa da, web uygulamaları hala çeşitli saldırılara karşı savunmasız olabilir. Örneğin, **SQL enjeksiyonları** veya **Komut Enjeksiyonları** ile yetkisiz erişim sağlanabilir.

**Front end** koduna erişimimiz varsa, **Whitebox Pentesting** ile kod incelemesi yapabiliriz. Ancak, **back end** kaynak kodu genellikle erişilemez ve **Blackbox Pentesting** yapılması gerekir. Bazı açık kaynak uygulamalar veya **Local File Inclusion** gibi zafiyetler, **back end** kaynak kodunu elde etmemize olanak tanıyabilir.

Penetrasyon testçileri için önemli olan, geliştiricilerin yaptığı en yaygın 20 hata şunlardır:

penetration tester olarak bizim için önemli olan, web geliştiricilerinin yaptığı en yaygın 20 hata şunlardır:

- Hatalı Verilerin Veritabanına Girmesine İzin Verme
- Sisteme Genel Olarak Odaklanma
- Kişisel Geliştirilmiş Güvenlik Yöntemleri Kurma
- Güvenliği Son Adım Olarak Görme
- Düz Metin Parola Depolama Geliştirme
- Zayıf Parolalar Oluşturma
- Veritabanında Şifrelenmemiş Verileri Saklama
- Client Tarafına Aşırı Bağımlı Olma
- Aşırı İyimser Olma
- Değişkenlerin URL Yolu İsimlerinde İzin Verme
- Üçüncü Taraf Kodu Güvenerek Kullanma
- Arka Kapı Hesaplarını Sabit Kodlama
- Doğrulanmamış SQL Enjeksiyonları
- Remote File Inclusion
- Güvensiz Veri İşleme
- Veriyi Doğru Şekilde Şifrelememe
- Güvenli Bir Kriptografik Sistem Kullanamama
- Katman 8’i İhmal Etme
- Kullanıcı Eylemlerini İncelememe
- Web Uygulama Güvenlik Duvarı Yapılandırma Hataları

Bu hatalar, diğer modüllerde ele alacağımız web uygulamaları için OWASP Top 10 güvenlik açıklarına yol açmaktadır:

|**No.**|**Vulnerability**|
|---|---|
|`1.`|Broken Access Control|
|`2.`|Cryptographic Failures|
|`3.`|Injection|
|`4.`|Insecure Design|
|`5.`|Security Misconfiguration|
|`6.`|Vulnerable and Outdated Components|
|`7.`|Identification and Authentication Failures|
|`8.`|Software and Data Integrity Failures|
|`9.`|Security Logging and Monitoring Failures|
|`10.`|Server-Side Request Forgery (SSRF)|


# HTML

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>
    </head>
    <body>
        <h1>A Heading</h1>
        <p>A Paragraph</p>
    </body>
</html>
```

Bu, aşağıdakileri görüntüleyecektir:

![Pasted image 20250211143634.png](/img/user/resimler/Pasted%20image%2020250211143634.png)

Gördüğümüz gibi, HTML öğeleri XML ve diğer dillere benzer şekilde bir tree biçiminde görüntülenir:

#### HTML Structure

```shell-session
document
 - html
   -- head
      --- title
   -- body
      --- h1
      --- p
```

Her bir element diğer HTML elementlerini içerebilirken, main HTML tag'i sayfa içerisindeki diğer tüm elementleri içermelidir, bu da document kapsamına girer ve HTML ile XML dokümanları gibi diğer diller için yazılmış dokümanları birbirinden ayırır.

Yukarıdaki kodun HTML öğeleri aşağıdaki gibi görüntülenebilir:

![Pasted image 20250211143930.png](/img/user/resimler/Pasted%20image%2020250211143930.png)

Her HTML elementi, elementin türünü belirten bir tag ile açılır ve kapanır 'örneğin paragraflar için `<p>`', içerik bu taglar arasına yerleştirilir. Tag'ler aynı zamanda elementin id'sini veya sınıfını da tutabilir 'örneğin `<p id='para1'>` veya `<p id='red-paragraphs'>`', bu da CSS'in elementi düzgün bir şekilde biçimlendirmesi için gereklidir. Hem tag'ler hem de içerik elementin tamamını oluşturur.


## URL Encoding

HTML'de öğrenilmesi gereken önemli bir kavram URL Encoding ya da percent-encoding'dir. Bir tarayıcının bir sayfanın içeriğini düzgün bir şekilde görüntüleyebilmesi için kullanılan karakter kümesini bilmesi gerekir. Örneğin URL'lerde tarayıcılar yalnızca alfanümerik karakterlere ve belirli özel karakterlere izin veren ASCII kodlamasını kullanabilir. Bu nedenle, ASCII karakter kümesi dışındaki diğer tüm karakterlerin bir URL içinde kodlanması gerekir. URL kodlaması, güvenli olmayan ASCII karakterlerini % sembolü ve ardından iki hexadecimal rakam ile değiştirir.

```
Örneğin, tek tırnak karakteri ''', tarayıcılar tarafından tek tırnak olarak anlaşılabilen '%27' olarak kodlanır. URL'lerde boşluk bulunamaz ve bir boşluk yerine + (artı işareti) veya %20 kullanılır. Bazı yaygın karakter kodlamaları şunlardır:
```

|Character|Encoding|
|---|---|
|space|%20|
|!|%21|
|"|%22|
|#|%23|
|$|%24|
|%|%25|
|&|%26|
|'|%27|
|(|%28|
|)|%29|


#### Usage

`<head>` elementi genellikle sayfada doğrudan yazdırılmayan öğeleri içerir, örneğin sayfa başlığı, tüm ana sayfa öğeleri ise `<body>` altında yer alır. Diğer önemli öğeler arasında sayfanın CSS kodlarını barındıran `<style>` ve sayfanın JS kodlarını içeren `<script>` bulunur, bunları bir sonraki bölümde göreceğiz.

Bu öğelerin her biri bir DOM (Document Object Model) olarak adlandırılır. Dünya Çapında Web Konsorsiyumu (W3C), DOM'u şu şekilde tanımlar:

"W3C Document Object Model (DOM), programların ve scriptleri bir dokumanın içeriğine, yapısına ve stiline dinamik olarak erişmesini ve bunları güncellemesini sağlayan platformdan bağımsız bir arayüzdür."

DOM standardı 3 parçaya ayrılır:

- **Core DOM** - tüm belge türleri için standart model
- **XML DOM** - XML belgeleri için standart model
- **HTML DOM** - HTML belgeleri için standart model

Örneğin, yukarıdaki tree görünümünden DOM'lara `document.head` veya `document.h1` gibi atıfta bulunabiliriz.

HTML DOM yapısını anlamak, sayfada gördüğümüz her öğenin nerede bulunduğunu anlamamıza yardımcı olabilir. Bu, sayfadaki belirli bir öğenin kaynak kodunu incelememize ve potansiyel sorunları aramamıza olanak tanır. HTML öğelerini, kimlikleri (id), etiket adları (tag name) veya sınıf adları (class name) ile bulabiliriz.

Bu, aynı zamanda mevcut öğeleri manipüle etmek veya ihtiyaçlarımıza göre yeni öğeler oluşturmak için ön uç zafiyetlerini (örneğin, XSS) kullanmak istediğimizde de faydalıdır.

Soru : Bir resmi göstermek için kullanılan HTML tagı nedir?

Cevap : `<img>`



# Cascading Style Sheets (CSS)

CSS (Cascading Style Sheets), HTML ile birlikte kullanılan bir stylesheet dilidir ve HTML elementlerinin stilini belirler. Her yeni sürüm, ek biçimlendirme yetenekleri sunarken, tarayıcılar da bu özellikleri desteklemek için güncellenir. CSS, belirli HTML elementlerinin (`body`, `h1` vb.) stilini tanımlayarak yazı tipi, renk, hizalama gibi özellikleri belirler.

```css
body {
  background-color: black;
}

h1 {
  color: white;
  text-align: center;
}

p {
  font-family: helvetica;
  font-size: 10px;
}
```

Daha önce de belirtildiği gibi, bu nedenle belirli HTML elementleri için benzersiz ID'ler veya sınıf adları belirleyebiliriz, böylece daha sonra gerektiğinde CSS veya JavaScript içinde bunlara başvurabiliriz.

## Syntax

CSS, her HTML elementinin veya class’ın stilini süslü parantez `{}` içinde tanımlar ve bu parantezler içinde özellikler, değerleriyle birlikte belirtilir (örneğin, `element { property: value; }`).

Her HTML elementinin, CSS ile ayarlanabilen birçok özelliği vardır. Bunlar arasında `height`, `position`, `border`, `margin`, `padding`, `color`, `text-align`, `font-size` ve yüzlerce diğer özellik bulunur. Tüm bu özellikler birleştirilerek görsel olarak etkileyici web sayfaları tasarlanabilir.

CSS, temel nesne hareketlerinden gelişmiş 3D animasyonlara kadar birçok farklı amaç için ileri düzey animasyonlar oluşturmakta kullanılabilir. `@keyframes`, `animation`, `animation-duration`, `animation-direction` gibi birçok CSS özelliği animasyonlar için mevcuttur. Bu animasyon özellikleri hakkında daha fazla bilgi edinebilir ve [bunları](https://www.w3schools.com/css/css3_animations.asp) deneyebilirsiniz.


#### Usage

CSS, JavaScript ile birlikte kullanılarak dinamik stil ayarları, hızlı hesaplamalar ve gelişmiş animasyonlar oluşturabilir. **"Parallax Depth Cards - by Andy Merskin on CodePen"** [bunun](https://codepen.io/) güzel bir örneğidir.


### **Frameworks**

CSS’i manuel olarak tasarlamak zahmetli olabilir. Bu nedenle, önceden tanımlanmış stiller içeren CSS framework’leri geliştirilmiştir. Web uygulamaları için optimize edilen ve genellikle JavaScript ile kullanılan bazı yaygın framework’ler şunlardır:

- **Bootstrap**
- **SASS**
- **Foundation**
- **Bulma**
- **Pure**


Soru : Bir HTML elementinin metninin sola hizalanmasını sağlamak için kullanılan CSS “property: value” nedir?

Cevap : `text-align: left;`


# JavaScript

JavaScript, web ve mobil geliştirmede yaygın olarak kullanılır. Genellikle front end tarafında çalışsa da, NodeJS gibi back end uygulamaları da mevcuttur. HTML ve CSS sayfanın görünümünü belirlerken, JavaScript işlevsellik ve etkileşim kazandırır.

#### Example

Sayfa kaynak kodu içinde JavaScript kodu `<script>` tagı ile aşağıdaki gibi yüklenir:

```html
<script type="text/javascript">
..JavaScript code..
</script>
```

Bir web sayfası, aşağıdaki gibi src ve komut dosyasının bağlantısı ile remote JavaScript kodunu da yükleyebilir:

```html
<script src="./script.js"></script>
```

Bir web sayfası içinde JavaScript'in temel kullanımına bir örnek aşağıdaki gibidir:

```javascript
document.getElementById("button1").innerHTML = "Changed Text!";
```


## Usage

Web uygulamaları, JavaScript'i sayfa görünümünü gerçek zamanlı güncelleme, içerik dinamik güncellemeleri, kullanıcı girdilerini işleme gibi işlevsellikler için kullanır. Ayrıca, Ajax gibi teknolojilerle back end ile etkileşim kurar ve HTTP istekleri gönderir. JavaScript, CSS ile birlikte gelişmiş animasyonlar sağlar ve modern tarayıcılar, sayfa güncellemeleri için back end’e ihtiyaç duymadan client-side çalışabilir.


## Frameworks

Web uygulamaları gelişirken saf JavaScript yerine daha verimli framework'ler kullanılır. Bu framework'ler, gelişmiş işlevsellikleri kolaylaştıran kütüphaneler sunar ve dinamik HTML kullanır. Öne çıkan bazı JavaScript framework’leri şunlardır:

- **Angular**
- **React**
- **Vue**
- **jQuery**



# Sensitive Data Exposure

Front end bileşenler client-side çalıştığı için saldırıya uğrasalar da, genellikle back end'e doğrudan zarar vermezler. Ancak, bir front end güvenlik açığı, admin kullanıcılarına saldırarak hassas verilere erişim, yetkisiz erişim ve hizmet kesintisi gibi sonuçlara yol açabilir.

[Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure), hassas verilerin açık metin olarak son kullanıcıya sunulmasıdır. Bu genellikle web sayfasının HTML kaynak kodunda bulunur ve sayfa kaynağı sağ tıklanarak veya ctrl + u tuşlarıyla görüntülenebilir. Örneğin, google.com sayfa kaynağını görüntüleyerek HTML, JavaScript ve harici bağlantılara ulaşılabilir.

![Pasted image 20250211200924.png](/img/user/resimler/Pasted%20image%2020250211200924.png)

Bazen, bir web sayfasının kaynak kodunda veya dış JavaScript kodlarında giriş bilgileri, hash'ler veya diğer hassas veriler bulunabilir. Bunlar, web uygulamasına veya altyapısına erişim sağlamak için kullanılabilir. Bu yüzden, bir web uygulamasını değerlendirirken ilk olarak sayfa kaynak kodunu inceleyip kolayca erişilebilecek verileri aramak önemlidir.

#### Example

İlk bakışta, bu giriş formu sıra dışı bir şey gibi görünmüyor:

![Pasted image 20250211201051.png](/img/user/resimler/Pasted%20image%2020250211201051.png)

Sayfa kaynağına bir göz atalım:

```html
<form action="action_page.php" method="post">

    <div class="container">
        <label for="uname"><b>Username</b></label>
        <input type="text" required>

        <label for="psw"><b>Password</b></label>
        <input type="password" required>

        <!-- TODO: remove test credentials test:test -->

        <button type="submit">Login</button>
    </div>
</form>

</html>
```

Developer'ların kaldırmayı unuttukları ve test kimlik bilgilerini içeren bazı yorumlar eklediklerini görüyoruz:

```html
<!-- TODO: remove test credentials test:test -->
```

Yorum, test kimlik bilgilerinin kaldırılmadığını gösteriyor, bu da kimlik bilgilerinin geçerli olabileceğini gösteriyor. Geliştirici yorumlarında genellikle kimlik bilgileri bulunmaz, ancak test sayfaları, dizinler ve gizli işlevler gibi hassas bilgiler bulunabilir. Bu bilgileri tespit etmek için otomatik araçlar kullanılabilir.

## Prevention (önleme)

Front end kaynak kodu yalnızca gerekli işlevleri içermeli, gereksiz kod ve yorumlardan kaçınılmalıdır. Kaynak kodunu gözden geçirmek veya araçlarla tarayarak ifşa edilen bilgileri kontrol etmek önemlidir. Ayrıca, geliştiriciler, client-side'da hangi bilgilerin ifşa edilebileceğini belirlemeli ve gereksiz yorumları veya gizli bağlantıları kaldırmalıdır. JavaScript kodunu paketleyerek veya obfuscation yöntemiyle hassas verilerin ifşa olma riski azaltılabilir.


Soru : Açık şifreler için yukarıdaki giriş formunu kontrol edin. Şifreyi cevap olarak gönderin.

Cevap : HiddenInPlainSight

![Pasted image 20250211201438.png](/img/user/resimler/Pasted%20image%2020250211201438.png)


# HTML Injection

Front end güvenliğinde, kullanıcı girdisinin doğrulanması ve temizlenmesi önemlidir. HTML injection, filtrelenmemiş girdilerin sayfada gösterilmesiyle gerçekleşir, bu da kötü niyetli kodlar veya sayfa tasarım değişikliklerine yol açabilir. Bu tür saldırılar, şirketin itibarına zarar verebilir.

#### Example

Aşağıdaki örnek, tek bir “ Click to enter your name” (Adınızı girmek için tıklayın) düğmesi olan çok basit bir web sayfasıdır. Düğmeye tıkladığımızda, adımızı girmemizi ister ve ardından adımızı “Adınız ...” olarak görüntüler:

![Pasted image 20250211201655.png](/img/user/resimler/Pasted%20image%2020250211201655.png)

Eğer input sanitization işlemi yapılmazsa, bu durum HTML Injection ve Cross-Site Scripting (XSS) saldırıları için kolay bir hedef olabilir. Sayfa kaynak koduna baktığımızda, hiçbir input sanitization işlemi yapılmadığını ve sayfanın kullanıcı inputunu alıp doğrudan görüntülediğini görüyoruz.

```html
<!DOCTYPE html>
<html>

<body>
    <button onclick="inputFunction()">Click to enter your name</button>
    <p id="output"></p>

    <script>
        function inputFunction() {
            var input = prompt("Please enter your name", "");

            if (input != null) {
                document.getElementById("output").innerHTML = "Your name is " + input;
            }
        }
    </script>
</body>

</html>
```

HTML Injection'ı test etmek için, adımız olarak küçük bir HTML kodu parçacığı girebilir ve sayfanın bir parçası olarak görüntülenip görüntülenmediğini görebiliriz. Web sayfasının arka plan görüntüsünü değiştiren aşağıdaki kodu test edeceğiz:

```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```

![Pasted image 20250211201825.png](/img/user/resimler/Pasted%20image%2020250211201825.png)

Bu örnekte, her şey front end üzerinde yürütüldüğünden, web sayfasının yenilenmesi her şeyi normale döndürecektir.

Soru : İnput olarak aşağıdaki payload'u kullanırsak sayfada hangi metin görüntülenir:      `<a href="http://www.hackthebox.com">Click Me</a>`

Cevap : `Your name is Click Me`



# Cross-Site Scripting (XSS)

HTML Injection güvenlik açıkları, JavaScript kodu enjekte edilerek XSS saldırıları gerçekleştirmek için kullanılabilir. XSS, HTML Injection’a benzer, ancak HTML yerine JavaScript kodu enjekte edilir. XSS'in üç türü vardır:

| Tür           | Açıklama                                                                                                                                                                    |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reflected XSS | Kullanıcı inputu işlendikten sonra sayfada görüntülendiğinde meydana gelir (örneğin, arama sonucu veya hata mesajı).                                                        |
| Stored XSS    | Kullanıcı inputu backend veritabanında saklandığında ve geri alındığında görüntülendiğinde meydana gelir (örneğin, gönderiler veya yorumlar).                               |
| DOM XSS       | Kullanıcı inputu doğrudan tarayıcıda görüntülendiğinde ve bir HTML DOM objesine yazıldığında meydana gelir (örneğin, güvenlik açığı olan kullanıcı adı veya sayfa başlığı). |

HTML Injection örneğinde olduğu gibi, burada da hiçbir input sanitization yapılmamaktadır. Bu nedenle, aynı sayfa XSS saldırılarına karşı savunmasız olabilir. Şu DOM XSS JavaScript kodunu payload olarak enjekte etmeyi deneyebiliriz, bu da bize mevcut kullanıcının cookie değerini gösterecektir:

```javascript
#"><img src=/ onerror=alert(document.cookie)>
```

Payload'umuzu girip tamam tuşuna bastığımızda, içinde cookie değerinin bulunduğu bir uyarı penceresinin açıldığını görüyoruz:

![Pasted image 20250211202239.png](/img/user/resimler/Pasted%20image%2020250211202239.png)

Bu payload, HTML document tree'yi erişerek cookie object'inin değerini alır. Tarayıcı inputumuzu işlerken bunu yeni bir DOM olarak kabul edecek ve JavaScript kodumuz çalıştırılacak, cookie değerini bir popup içinde bize gösterecektir.

Bir saldırgan, bunu cookie oturumlarını çalmak ve kendisine göndermek için kullanabilir ve cookie değerini kullanarak victim'in hesabına kimlik doğrulaması yapmayı deneyebilir. Aynı saldırı, web uygulamasının kullanıcılarına karşı farklı saldırılar gerçekleştirmek için de kullanılabilir. XSS, geniş bir konu olup, sonraki modüllerde derinlemesine ele alınacaktır.

Soru : Yukarıdaki sayfada cookie değerini almak için XSS kullanmayı deneyin
Cevap : `XSSisFun`


![Pasted image 20250211202452.png](/img/user/resimler/Pasted%20image%2020250211202452.png)


# Cross-Site Request Forgery (CSRF)


**CSRF** (Cross-Site Request Forgery), filtrelenmemiş kullanıcı girdilerinden kaynaklanan bir **front end** güvenlik açığıdır. Saldırgan, **XSS** açıklarını kullanarak **API** çağrıları ve sorgular gerçekleştirebilir, böylece kurbanın kimlik doğrulamasıyla işlemler yapabilir.

Yüksek ayrıcalıklı erişim sağlamak için, saldırgan kurbanın şifresini değiştiren bir **JavaScript payload**'ı oluşturabilir. Kurban sayfayı görüntülediğinde, **JavaScript** kodu çalışarak **session cookie**'yi kullanarak şifreyi değiştirir.

**CSRF**, yöneticilerin hesaplarına da saldırarak **back-end server**'a erişim sağlanabilir. Bunun için, uzaktan bir **JavaScript** dosyası yüklenebilir.

```html
"><script src=//www.example.com/exploit.js></script>
```

**exploit.js** dosyası, kullanıcının şifresini değiştiren kötü niyetli **JavaScript** kodunu içerir. **exploit.js**'i geliştirmek, bu **web application**'ın şifre değiştirme prosedürünü ve **API**'lerini bilmek gerektirir. Saldırgan, istenilen işlevselliği taklit eden ve bunu otomatik olarak gerçekleştiren **JavaScript** kodu oluşturmak zorundadır (yani, bu **web application** için şifremizi değiştiren **JavaScript** kodu).


## Prevention

**Back end**'e ulaşmadan önce **front end**'de kullanıcı girdilerinin filtrelenmesi ve temizlenmesi önemlidir, özellikle bu kod doğrudan **client-side**'da görüntülenecekse. Kullanıcı girdisi kabul edilirken iki ana kontrol uygulanmalıdır:

| **Tür**          | **Açıklama**                                                                         |
| ---------------- | ------------------------------------------------------------------------------------ |
| **Sanitization** | Özel ve standart dışı karakterleri temizlemek.                                       |
| **Validation**   | Girdinin beklenen formatta olup olmadığını kontrol etmek (örneğin, e-posta formatı). |
Çıktı da temizlenmeli ve özel karakterler kaldırılmalıdır. **HTML Injection**, **XSS** veya **CSRF** gibi saldırılar engellenebilir. **WAF** çözümleri enjeksiyonları engellemeye yardımcı olabilir ancak atlatılabilir, bu yüzden geliştiriciler en iyi kodlama uygulamalarını takip etmelidir.

**CSRF**'ye karşı modern tarayıcılar ve **web application**'lar anti-**CSRF** önlemleri sunar, ancak bazı güvenlik önlemleri atlatılabilir. Bu yüzden bu önlemler ikincil bir güvenlik önlemi olmalı, geliştiriciler de her zaman güvenli kod yazmalıdır.


# Back End Servers

**Back-end server**, **web application**'ı çalıştıran donanım ve işletim sistemidir, tüm **process**'leri çalıştırır ve **task**'leri yerine getirir. **Back-end server**, **Data access layer**'da yer alır.

**Back-end server**, aşağıdaki 3 **back end** bileşenini içerir:

- **Web Server**
- **Database**
- **Development Framework**

![Pasted image 20250212004648.png](/img/user/resimler/Pasted%20image%2020250212004648.png)


Back end sunucusundaki diğer yazılım bileşenleri [hypervisors](https://en.wikipedia.org/wiki/Hypervisor), konteynerleri ve WAF'ları içerebilir.

Belirli bir dizi back end bileşeni içeren back end sunucular için birçok popüler “ stack ” kombinasyonu vardır. Bazı yaygın örnekler şunlardır:

|Combinations|Components|
|---|---|
|[LAMP](https://en.wikipedia.org/wiki/LAMP_\(software_bundle\))|`Linux`, `Apache`, `MySQL`, and `PHP`.|
|[WAMP](https://en.wikipedia.org/wiki/LAMP_\(software_bundle\)#WAMP)|`Windows`, `Apache`, `MySQL`, and `PHP`.|
|[WINS](https://en.wikipedia.org/wiki/Solution_stack)|`Windows`, `IIS`, `.NET`, and `SQL Server`|
|[MAMP](https://en.wikipedia.org/wiki/MAMP)|`macOS`, `Apache`, `MySQL`, and `PHP`.|
|[XAMPP](https://en.wikipedia.org/wiki/XAMPP)|Cross-Platform, `Apache`, `MySQL`, and `PHP/PERL`.|

Bu makalede Web Solution Stacks'in [kapsamlı](https://en.wikipedia.org/wiki/Solution_stack) bir listesini bulabiliriz.


## Hardware

**Back-end server**, **web application**'ın stabilitesini ve hızını belirleyen donanımı içerir. Büyük **web application**'ları, yüklerini birden fazla **back end server**'a dağıtarak aynı **task**'leri yerine getiren mimarilerle çalışır. **Web applications**, tek bir **back end server** yerine **data centers** ve **cloud hosting services** kullanarak **virtual hosts**'ları da kullanabilirler.


Soru : 'WAMP' hangi işletim sistemi ile kullanılır?

Cevap : Windows



# Web Servers

Web sunucusu, back end sunucu üzerinde çalışan, client-side tarayıcıdan gelen tüm HTTP trafiğini işleyen, istenen sayfalara yönlendiren ve son olarak client-side tarayıcıya yanıt veren bir uygulamadır. Web sunucuları genellikle 80 veya 443 numaralı TCP portlarında çalışır ve çeşitli response'larını işlemenin yanı sıra son kullanıcıları web uygulamasının çeşitli bölümlerine bağlamaktan sorumludur.


## Workflow

Tipik bir web sunucusu, client tarafından gelen HTTP isteklerini kabul eder ve başarılı bir istek için 200 OK kodu, mevcut olmayan sayfaları isterken 404 NOT FOUND kodu, kısıtlanmış sayfalara erişim isterken 403 FORBIDDEN kodu gibi farklı HTTP yanıtları ve kodlarıyla yanıt verir.

![Pasted image 20250212010242.png](/img/user/resimler/Pasted%20image%2020250212010242.png)


|**Kod**|**Açıklama**|
|---|---|
|**200 OK**|İstek başarılı oldu.|
|**301 Moved Permanently**|Talep edilen kaynağın URL'si kalıcı olarak değiştirildi.|
|**302 Found**|Talep edilen kaynağın URL'si geçici olarak değiştirildi.|
|**400 Bad Request**|Sunucu, hatalı sözdizimi nedeniyle isteği anlayamadı.|
|**401 Unauthorized**|Kimlik doğrulaması yapılmadan sayfaya erişim girişimi.|
|**403 Forbidden**|**Client**, içeriğe erişim yetkisine sahip değil.|
|**404 Not Found**|Sunucu, talep edilen kaynağı bulamıyor.|
|**405 Method Not Allowed**|Sunucu, istek metodunu biliyor ancak bu metod devre dışı bırakılmış ve kullanılamıyor.|
|**408 Request Timeout**|Bazı sunucular tarafından, **client** herhangi bir önceki istek göndermemiş olsa bile, boşta kalan bağlantılar için gönderilen yanıt.|
|**500 Internal Server Error**|Sunucu, nasıl çözeceğini bilmediği bir durumla karşılaştı.|
|**502 Bad Gateway**|Sunucu, bir **gateway** olarak çalışırken, isteği işlemek için gereken yanıtı alırken geçersiz bir yanıt aldı.|
|**504 Gateway Timeout**|Sunucu, bir **gateway** olarak çalışırken, zamanında yanıt alamadı.|

Web servers, HTTP istekleri içinde **text**, **JSON** ve hatta **binary** veri (örn. dosya yüklemeleri) gibi çeşitli **user input** türlerini kabul eder. Bir **web server**, bir **web request** aldığında, bunu hedefe yönlendirmek, gerekli **process**'leri çalıştırmak ve sonucu **client-side**'da kullanıcıya geri döndürmekle sorumludur. **Web server**'ın yönlendirdiği **pages** ve **files**, **web application core files**'ıdır.

Aşağıda, **Linux terminal**'inde **cURL utility** kullanılarak bir **page** isteğinin nasıl yapıldığını ve **-I flag** ile sadece **headers**'ın nasıl görüntülendiğini gösteren bir örnek bulunmaktadır.

```shell-session
C4RT3L@htb[/htb]$ curl -I https://academy.hackthebox.com

HTTP/2 200
date: Tue, 15 Dec 2020 19:54:29 GMT
content-type: text/html; charset=UTF-8
...SNIP...
```

Bu cURL komutu örneği bize web sayfasının kaynak kodunu gösterirken:

```shell-session
C4RT3L@htb[/htb]$ curl https://academy.hackthebox.com

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>Cyber Security Training : HTB Academy</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

Web applications için çeşitli **web server** türleri kullanılabilir. Çoğu karmaşık **HTTP request**'leri karşılayabilir ve genellikle ücretsizdir. Python, JavaScript, PHP gibi dillerle temel bir **web server** geliştirilebilir. Ancak, yüksek **web traffic**'i yönetmek için optimize edilmiş popüler **web applications** mevcut olup, bu da zaman kazandırır.

## Apache


![Pasted image 20250212010641.png](/img/user/resimler/Pasted%20image%2020250212010641.png)

Apache (**httpd**) en yaygın **web server** olup, internet sitelerinin %40’ından fazlasını barındırır. Çoğu **Linux distro**'da ön yüklü gelir ve **Windows/macOS**'a da kurulabilir. **PHP** ile yaygın kullanılır ancak **.Net, Python, Perl, Bash** gibi dilleri de destekler. **Apache modülleri** ile işlevselliği genişletilebilir (**mod_php** gibi). **Açık kaynaklı** olup, düzenli **security patch** alır ve iyi belgelenmiştir. **Kolay geliştirme** imkanı sunduğundan küçük şirketler ve yeni başlayanlar için idealdir, ancak büyük şirketler de kullanmaktadır.

| `Apple` | `Adobe` | `Baidu` |
| ------- | ------- | ------- |



## NGINX

![Pasted image 20250212010934.png](/img/user/resimler/Pasted%20image%2020250212010934.png)
NGINX, en yaygın ikinci **web server** olup, internet sitelerinin %30’unu barındırır. **Async mimarisi** sayesinde düşük **CPU/RAM** kullanarak çok sayıda eşzamanlı isteğe hizmet eder, bu da onu yüksek trafikli **web application**'lar için ideal kılar. **İlk 100.000** web sitesinin %60’ı NGINX kullanmaktadır. **Açık kaynaklı** ve güvenilirdir.

|`Google`|`Facebook`|`Twitter`|`Cisco`|`Intel`|`Netflix`|`HackTheBox`|
|---|---|---|---|---|---|---|


## IIS

![Pasted image 20250212011237.png](/img/user/resimler/Pasted%20image%2020250212011237.png)

IIS, en yaygın üçüncü **web server** olup, internet sitelerinin %15’ini barındırır. **Microsoft** tarafından geliştirilir ve genellikle **Windows Server**'da çalışır. **.NET framework** için optimize edilmiştir ancak **PHP, FTP** gibi servisleri de destekler. **Active Directory** entegrasyonu ve **Windows Auth** ile kimlik doğrulamayı kolaylaştırır. Büyük şirketler, **Windows Server** ve **Active Directory** kullandıkları için IIS tercih eder.

| `Microsoft` | `Office365` | `Skype` | `Stack Overflow` | `Dell` |
| ----------- | ----------- | ------- | ---------------- | ------ |

Bu 3 web sunucusunun yanı sıra, Java web uygulamaları için Apache Tomcat ve back end'de JavaScript kullanılarak geliştirilen web uygulamaları için Node.JS gibi yaygın olarak kullanılan başka web sunucuları da vardır.

Soru : Bir web sunucusu bir HTTP kodu 201 döndürürse, bu ne anlama gelir?

Cevir : Created


# Databases

Web applications, **back end database** kullanarak içerik, varlıklar ve kullanıcı verilerini depolar. Bu, **dinamik içerik** sağlar ve verilerin hızlı erişimini mümkün kılar. **Veritabanı türleri**, hız, ölçeklenebilirlik, boyut ve maliyet gibi faktörlere göre seçilir.


## Relational (SQL)

**Relational (SQL) databases**, verileri **tablolar**, **rows** ve **columns** şeklinde saklar. **Primary key** ile tablolar arasında ilişkiler oluşturulabilir. Örneğin, **users** tablosu **id, username** içerirken, **posts** tablosu **id, user_id, content** barındırabilir.

![Pasted image 20250212015510.png](/img/user/resimler/Pasted%20image%2020250212015510.png)

**users** tablosundaki **id**, **posts** tablosundaki **user_id** ile ilişkilendirilerek her gönderiye ait kullanıcı bilgileri, her gönderiye tüm kullanıcı verilerini eklemeye gerek kalmadan kolayca alınabilir.

Bir tablo birden fazla **key** içerebilir; başka bir sütun farklı bir tabloyla ilişkilendirmek için kullanılabilir. Örneğin, **id** sütunu **posts** tablosunu, her bir gönderiye ait **comments** içeren başka bir tabloyla bağlamak için kullanılabilir.

Veritabanındaki tablolar arasındaki ilişkiye **Schema** denir.

Bu yapı sayesinde, **relational databases** kullanılarak belirli bir öğeye ait tüm veriler tek bir **query** ile alınabilir. Bu da **relational databases**'i büyük ve iyi yapılandırılmış veri setleri için **hızlı**, **güvenilir** ve **verimli** hale getirir.

En yaygın **relational databases** şunlardır:

| **Type**       | **Description**                                                                                                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MySQL**      | İnternette en yaygın kullanılan veritabanıdır. Açık kaynaklı bir veritabanıdır ve tamamen ücretsiz kullanılabilir.                                                                                      |
| **MSSQL**      | Microsoft'un ilişkisel veritabanı uygulamasıdır. Genellikle Windows Sunucuları ve IIS web sunucuları ile yaygın olarak kullanılır.                                                                      |
| **Oracle**     | Büyük işletmeler için çok güvenilir bir veritabanıdır ve hızını ve güvenilirliğini artırmak için yenilikçi veritabanı çözümleri ile sık sık güncellenir. Büyük işletmeler için bile maliyetli olabilir. |
| **PostgreSQL** | Diğer bir ücretsiz ve açık kaynaklı ilişkisel veritabanıdır. Kolayca genişletilebilir şekilde tasarlanmıştır, böylece büyük değişiklikler yapmadan yeni gelişmiş özellikler eklemeye olanak tanır.      |

|   |
|---|
|Diğer yaygın SQL veritabanları şunlardır: SQLite, MariaDB, Amazon Aurora ve Azure SQL.|


## Non-relational (NoSQL)

İlişkisel olmayan (NoSQL) veritabanları, tablo, satır, sütun gibi yapılar kullanmaz ve esneklik sağlar. Veri yapıları belirli olmayan büyük veri setlerinde tercih edilir. NoSQL için dört yaygın depolama modeli vardır:

- **Key-Value**
- **Document-Based**
- **Wide-Column**
- **Graph**

Bu modeller, veriyi farklı şekillerde depolar; örneğin, Key-Value modeli JSON veya XML kullanarak veri depolar ve her veriyi bir anahtar-değer çifti olarak saklar.

![Pasted image 20250212025443.png](/img/user/resimler/Pasted%20image%2020250212025443.png)

Yukarıdaki örnek JSON kullanılarak aşağıdaki gibi gösterilebilir:

```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

Python veya PHP'deki sözlük/map/anahtar-değer çifti gibi yapıdadır (`{'key':'value'}`), burada anahtar bir string, değer ise string, sözlük veya class nesnesi olabilir.

**Document-Based** modeli, veriyi JSON nesnelerinde depolar ve her obje meta veri ile birlikte veri içerir.

Popüler NoSQL veritabanları şunlardır:

|**Tür**|**Açıklama**|
|---|---|
|**MongoDB**|En yaygın NoSQL veritabanı. Ücretsiz ve açık kaynaklıdır, Document-Based model kullanır ve verileri JSON nesnelerinde depolar.|
|**ElasticSearch**|Ücretsiz ve açık kaynaklı NoSQL veritabanı. Büyük veri setlerini depolamak ve analiz etmek için optimize edilmiştir. Verilere hızlı ve verimli arama yapılabilir.|
|**Apache Cassandra**|Ücretsiz ve açık kaynaklıdır. Çok ölçeklenebilir ve hatalı değerleri yönetmek için optimize edilmiştir.|

Diğer yaygın NoSQL veritabanları şunları içerir: Redis, Neo4j, CouchDB ve Amazon DynamoDB.


## Use in Web Applications

Çoğu modern web geliştirme dili ve framework'ü, çeşitli veritabanı türlerini entegre etmeyi, depolamayı ve bunlardan veri almayı kolaylaştırır. Ancak öncelikle, veritabanının back end sunucusuna kurulması ve ayarlanması gerekir ve bir kez kurulup çalışmaya başladığında, web uygulamaları veri depolamak ve almak için onu kullanmaya başlayabilir.

Örneğin, bir PHP web uygulamasında, MySQL kurulup çalıştırıldıktan sonra, veritabanı sunucusuna şu komutla bağlanabiliriz:

```php
$conn = new mysqli("localhost", "user", "pass");
```

Ardından, ile yeni bir veritabanı oluşturabiliriz:

```php
$sql = "CREATE DATABASE database1";
$conn->query($sql)
```

Bundan sonra, yeni veritabanımıza bağlanabilir ve MySQL veritabanını MySQL syntax aracılığıyla PHP içinde aşağıdaki gibi kullanmaya başlayabiliriz:

```php
$conn = new mysqli("localhost", "user", "pass", "database1");
$query = "select * from table_1";
$result = $conn->query($query);
```

Web uygulamaları, kullanıcı girdisini alarak veritabanlarında arama yapmak için kullanır. Örneğin, bir kullanıcı arama fonksiyonunu kullanarak diğer kullanıcıları aradığında, girilen veri veritabanında arama yapar.

```php
$searchInput =  $_POST['findUser'];
$query = "select * from users where name like '%$searchInput%'";
$result = $conn->query($query);
```

Son olarak, web uygulaması sonucu kullanıcıya geri gönderir:

```php
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```


Soru : Google'ın Firebase Veritabanı ne tür bir veritabanıdır?

Cevap : NoSQL


# Development Frameworks & APIs

Web uygulamalarını barındırabilen sunucuların yanı sıra, birçok yaygın **web development framework**'ü, temel web uygulama dosyalarını ve işlevselliğini geliştirmeye yardımcı olur. Web uygulamalarının karmaşıklığının artmasıyla birlikte, modern ve sofistike bir web uygulaması sıfırdan oluşturmak zorlaşır. Bu nedenle, popüler web uygulamaları genellikle **web framework**'leri kullanılarak geliştirilir.

Çoğu web uygulaması, kullanıcı kaydı gibi ortak işlevleri paylaştığından, **web framework**'leri bu işlevleri hızlıca uygulamayı ve bunları **front end** bileşenlerine bağlayarak fonksiyonel bir web uygulaması oluşturmayı kolaylaştırır. Yaygın **web development framework** örnekleri şunlardır:

Laravel (PHP): genellikle startuplar ve küçük şirketler tarafından kullanılır, çünkü güçlü olmasına rağmen geliştirilmesi kolaydır.  
Express (Node.JS): PayPal, Yahoo, Uber, IBM ve MySpace tarafından kullanılır.  
Django (Python): Google, YouTube, Instagram, Mozilla ve Pinterest tarafından kullanılır. 
Rails (Ruby): GitHub, Hulu, Twitch, Airbnb ve geçmişte Twitter tarafından kullanılmıştır.

Popüler web sitelerinin genellikle sadece bir tane yerine, birden fazla framework ve web sunucusu kullandığı belirtilmelidir.


## APIs

Back end web uygulaması geliştirmede, front end ve back end bileşenleri arasındaki veri iletimi için Web API'leri ve HTTP İstek parametreleri kullanılır. Front end, back end'den belirli bir görev istemek için API'leri kullanır, back end ise bu istekleri işleyip gerekli fonksiyonları yerine getirir ve sonrasında bir response döndürür, bu da client-side'da kullanıcı çıktısını oluşturur.

#### Query Parameters
Bir web sayfasına argüman göndermenin varsayılan yöntemi GET ve POST request parametreleridir. Bu, front end'in belirli parametreler için değerler belirtmesini ve back end'in bunları işlemesini sağlar. Örneğin, /search.php sayfası bir item parametresi alır; GET isteği '/search.php?item=apples' URL'si ile, POST parametreleri ise POST verileri ile gönderilir.

```http
POST /search.php HTTP/1.1
...SNIP...

item=apples
```

## Web APIs

API (Application Programming Interface - Uygulama Programlama Arayüzü), bir uygulamanın diğer uygulamalarla nasıl etkileşime girebileceğini belirten bir arayüzdür. Web uygulamaları için, back end fonksiyonlarına remote erişim sağlar. API'ler sadece web uygulamalarına özel değildir ve genellikle HTTP üzerinden web sunucuları aracılığıyla erişilir ve işlenir.

![Pasted image 20250212030916.png](/img/user/resimler/Pasted%20image%2020250212030916.png)

Örneğin, bir hava durumu uygulaması, şehir için mevcut hava durumunu almak amacıyla bir API kullanabilir ve şehir adını ileterek JSON objesi döndürebilir. Benzer şekilde, Twitter'ın API'si, son Tweetleri almayı veya Tweet göndermeyi sağlar. Web uygulamalarında API kullanımı için geliştiricilerin SOAP veya REST gibi API standartlarını kullanarak back end işlevselliğini geliştirmeleri gerekir.

## SOAP

SOAP ( Simple Objects Access) standardı verileri XML aracılığıyla paylaşır, burada istek bir HTTP isteği aracılığıyla XML olarak yapılır ve response da XML olarak döndürülür. Front end bileşenleri bu XML çıktısını düzgün bir şekilde ayrıştırmak için tasarlanmıştır. Aşağıda örnek bir SOAP mesajı verilmiştir:

```xml
<?xml version="1.0"?>

<soap:Envelope
xmlns:soap="http://www.example.com/soap/soap/"
soap:encodingStyle="http://www.w3.org/soap/soap-encoding">

<soap:Header>
</soap:Header>

<soap:Body>
  <soap:Fault>
  </soap:Fault>
</soap:Body>

</soap:Envelope>
```

SOAP, yapılandırılmış verilerin ve binary verilerin aktarılması için çok kullanışlıdır, özellikle front end ve back end arasında kompleks verilerin paylaşılmasını ve düzgün ayrıştırılmasını sağlar. Ayrıca, stateful objelerin paylaşımı için de kullanılır. Ancak, SOAP yeni başlayanlar için zor olabilir ve küçük sorgular için bile karmaşık istekler gerektirebilir. Bu yüzden REST API standardı daha kullanışlıdır.

## REST

REST, verileri URL yolu üzerinden paylaşır ve genellikle JSON formatında döndürür. Query Parameters'tan farklı olarak, REST API'leri genellikle URL yolundan doğrudan geçirilen input bekler. Bu, arama, sıralama veya filtreleme gibi işlemler için kullanışlıdır. REST API'ler, web uygulamalarını modüler ve ölçeklenebilir hale getirmek için daha küçük API'lere bölünür. Yanıtlar genellikle JSON formatında olup, front end bileşenleri tarafından işlenir. Diğer formatlar XML, x-www-form-urlencoded veya raw veri olabilir.

```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

REST, web uygulaması üzerinde farklı eylemler gerçekleştirmek için çeşitli HTTP yöntemleri kullanır:

* Veri almak için `GET` isteği
* Veri oluşturmak için `POST` isteği (örnek olmayan)
* Mevcut verileri oluşturmak veya değiştirmek için `PUT` isteği (idempotent)
* Verileri kaldırmak için `DELETE` isteği

soru : ID numarası 1 olan kullanıcının adını aramak için '/index.php?id=0' GET isteğini kullanın?

Cevap : superadmin

![Pasted image 20250212031933.png](/img/user/resimler/Pasted%20image%2020250212031933.png)


# Common Web Vulnerabilities

Aşağıdaki örnekler, web uygulamaları için [OWASP Top 10](https://owasp.org/www-project-top-ten/) güvenlik açıklarının bir parçası olan web uygulamaları için en yaygın güvenlik açığı türlerinden bazılarıdır.


## Broken Authentication/Access Control

Broken Authentication ve Broken Access Control, web uygulamaları için en yaygın ve tehlikeli güvenlik açıklarındandır.

**Broken Authentication**, saldırganların kimlik doğrulama işlevlerini atlamasına izin verir, bu da geçerli kimlik bilgileri olmadan oturum açmayı veya yetkisiz bir kullanıcının yönetici olmasını sağlayabilir.

**Broken Access Control**, saldırganların erişmemesi gereken sayfalara veya özelliklere erişmesine neden olur. Örneğin, normal bir kullanıcının admin paneline erişmesi.

Örneğin, College Management System 1.2, e-posta alanına `' veya 0=0 #` yazarak ve herhangi bir şifre kullanarak hesaba sahip olmadan giriş yapılmasına olanak tanır.


## Malicious File Upload

Web uygulamalarında kontrol sağlamak için bir diğer yaygın yöntem de kötü niyetli komut dosyaları yüklemektir. Eğer web uygulaması dosya yükleme özelliğine sahipse ve yüklenen dosyalar düzgün doğrulanmazsa, zararlı bir script (örneğin, PHP scripti) yükleyerek sunucuda komut çalıştırabiliriz.

Bu temel bir güvenlik açığı olsa da, birçok geliştirici bu tehdidi göz ardı eder veya doğrulamaları geçersiz kılınabilir. Örneğin, WordPress Responsive Thumbnail Slider 1.0 eklentisi, çift uzantılı dosyalarla (örn. shell.php.jpg) zararlı dosyaların yüklenmesini sağlar. Bu güvenlik açığı, Metasploit Modülü ile kolayca kullanılabilir.


## Command Injection

Birçok web uygulaması, belirli işlemleri gerçekleştirmek için lokal işletim sistemi komutları çalıştırır. Örneğin, bir web uygulaması, sağlanan plugin adını kullanarak bir plugin yükleyebilir. Eğer komutlar düzgün filtrelenmezse, saldırganlar başka bir komut enjekte edebilir, bu da sunucuda komut çalıştırmalarına ve kontrol sağlamalarına yol açar. Bu tür bir güvenlik açığına **command injection** denir.

Geliştiricilerin kullanıcı girdilerini düzgün sterilize edememesi veya zayıf testler kullanması nedeniyle bu güvenlik açığı yaygındır. Örneğin, WordPress Eklentisi Plainview Activity Monitor 20161228, saldırganların ip değerine | COMMAND... ekleyerek komut enjekte etmelerine izin verir.

## SQL Injection (SQLi)

Web uygulamalarında çok yaygın olan bir diğer güvenlik açığı da SQL İnjection güvenlik açığıdır. Command Injection zafiyetine benzer şekilde, bu zafiyet web uygulaması kullanıcı tarafından sağlanan girdiden alınan bir değer içeren bir SQL sorgusu yürüttüğünde ortaya çıkabilir.

Örneğin, veritabanı bölümünde, bir web uygulamasının belirli bir tabloda arama yapmak için kullanıcı girdisini nasıl kullanacağının bir örneğini aşağıdaki kod satırıyla gördük:

```php
$query = "select * from users where name like '%$searchInput%'";
```

Kullanıcı girdisi düzgün filtrelenmezse (Komut Enjeksiyonları gibi), SQL enjeksiyonu yapılabilir ve bu da veritabanı ve barındırma sunucusunun kontrolünü ele geçirmeye yol açar.

Örneğin, College Management System 1.2, her zaman true döndüren bir SQL sorgusu çalıştırarak başarılı bir kimlik doğrulaması sağlar, bu da uygulamaya giriş yapmamızı mümkün kılar. Bu güvenlik açığı, veritabanından veri almak veya sunucu kontrolünü ele geçirmek için de kullanılabilir.


Soru : 'CVE-2014-6271' kamu güvenlik açığı yukarıdaki kategorilerden hangisine aittir?

Cevap : Command Injection

![Pasted image 20250212032747.png](/img/user/resimler/Pasted%20image%2020250212032747.png)


# Public Vulnerabilities

En kritik back end açıkları, dışarıdan saldırılabilir ve local erişim olmadan sunucu kontrolünü ele geçirebilir. Genellikle kodlama hatalarından kaynaklanır ve temel ile sofistike güvenlik açıkları arasında değişir.

## Public CVE

Birçok web uygulaması, dünya çapında test edilerek birçok güvenlik açığı ortaya çıkarır. Bu açıklar genellikle yamanır ve CVE kaydı alır. Sızma testi uzmanları, bu açıkları test etmek için proof of concept exploit hazırlar ve bunları kamuya sunar. İlk adım, web uygulamasının sürümünü belirlemektir. Sürüm belirlendikten sonra, açık exploit'ler Google veya Exploit DB gibi veritabanlarında aranabilir.

![Pasted image 20250212033023.png](/img/user/resimler/Pasted%20image%2020250212033023.png)


## Common Vulnerability Scoring System (CVSS)

Common Vulnerability Scoring System (CVSS), güvenlik açıklarının ciddiyetini değerlendiren açık kaynaklı bir standarttır. Bu puanlama, güvenlik açıklarının önem derecelerini belirleyerek kaynakların önceliklendirilmesine yardımcı olur. CVSS, Base, Temporal ve Environmental metriklerinden oluşan bir formüle dayanır. NVD, bilinen güvenlik açıkları için CVSS puanları sağlar. CVSS v2 ve v3 arasında bazı değişiklikler vardır, özellikle Base ve Environmental metrikleri üzerine.

CVSS puanlama dereceleri, aşağıdaki tablolarda görülebileceği gibi V2 ve V3 arasında biraz farklılık göstermektedir:

|**Severity**|**CVSS V2.0 Base Score Range**|**CVSS V3.0 Base Score Range**|
|---|---|---|
|**None**|-|0.0|
|**Low**|0.0-3.9|0.1-3.9|
|**Medium**|4.0-6.9|4.0-6.9|
|**High**|7.0-10.0|7.0-8.9|
|**Critical**|-|9.0-10.0|

NVD, Temporal ve Environmental metriklerini dikkate almaz çünkü Temporal metrikleri zamanla değişebilir ve Environmental metrikleri organizasyona özgüdür. NVD, bu ek riskleri hesaplamak için CVSS v2 ve v3 hesaplayıcıları sunar. Bu hesaplayıcılar, CVSS puanını çevremize göre ince ayar yapmamızı sağlar ve her bir metriğin nasıl uygulandığını öğrenmemizi sağlar.

![Pasted image 20250212033354.png](/img/user/resimler/Pasted%20image%2020250212033354.png)


## Back-end Server Vulnerabilities

Web uygulamaları gibi, back-end bileşenlerinde de güvenlik açıklarını araştırmalıyız. En kritik açıklar, web sunucularındaki TCP üzerinden erişilebilenlerdir. Örneğin, Apache web sunucularını etkileyen `Shell-Shock`, HTTP istekleriyle back-end sunucusunda uzaktan kontrol sağlar. Back-end sunucu veya veritabanı açıkları genellikle lokal erişim sonrası kullanılır ve iç sızma testleriyle elde edilebilir. Bu açıklar doğrudan dışarıdan exploit edilemese de, yine de kritik olup, tüm web uygulamasını korumak için yamalanmalıdır.

Soru : CVE-2017-0144 genel güvenlik açığının CVSS puanı nedir?

Cevap : 9.3
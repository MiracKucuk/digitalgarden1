---
{"dg-publish":true,"permalink":"/ctf/htb-usage/"}
---



## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

![Pasted image 20241208042521.png](/img/user/resimler/Pasted%20image%2020241208042521.png)

![Pasted image 20241208042529.png](/img/user/resimler/Pasted%20image%2020241208042529.png)

OpenSSH sürümüne göre, host muhtemelen Ubuntu 22.04 [[Bağlantılar/jammy\|jammy]] çalıştırıyor.

Web sunucusunda usage.htb adresine bir yönlendirme var.


### Subdomain Fuzz - TCP 80

Domain tabanlı yönlendirme (veya virtual hosts) kullanımı göz önüne alındığında, usage.htb'nin varsayılan durumdan farklı yanıt veren tüm subdomain'lerini taramak için ffuf kullanacağım:

`ffuf -u http://10.10.11.18 -H "Host: FUZZ.usage.htb" -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac`

![Pasted image 20241208042808.png](/img/user/resimler/Pasted%20image%2020241208042808.png)

admin.usage.htb'yi buluyor. Bunları /etc/hosts dosyama ekleyeceğim:

![Pasted image 20241208042855.png](/img/user/resimler/Pasted%20image%2020241208042855.png)


### usage.htb - TCP 80

Site

Site bir giriş formu sunmaktadır:

![Pasted image 20241208042927.png](/img/user/resimler/Pasted%20image%2020241208042927.png)

En üstte, üç bağlantı bu giriş formuna (/index.php/login), kayıt formuna (/index.php/registration) ve http://admin.usage.htb/ adresine yönlendirir.

Ayrıca, e-posta adresi isteyen bir forma yönlendiren bir “ Password Reset” bağlantısı (/forgot-password) bulunmaktadır:

![Pasted image 20241208043017.png](/img/user/resimler/Pasted%20image%2020241208043017.png)

Eğer var olmayan bir e-posta girersem:

![Pasted image 20241208043048.png](/img/user/resimler/Pasted%20image%2020241208043048.png)

Kayıt olduktan sonra bu adresi girersem:

![Pasted image 20241208043146.png](/img/user/resimler/Pasted%20image%2020241208043146.png)

Kayıt formu bir isim, e-posta ve şifre alır:

![Pasted image 20241208043202.png](/img/user/resimler/Pasted%20image%2020241208043202.png)

Kayıt olmak giriş sayfasına yönlendirir ve giriş yapmak bazı gönderilerin bulunduğu bir sayfaya yönlendirir:

![Pasted image 20241208043230.png](/img/user/resimler/Pasted%20image%2020241208043230.png)

Bu yazılar yapay zeka tarafından üretilmiş gibi görünüyor, vızıltılı kelimelerle dolu ve pek bir anlamı yok. Laravel PHP'den bahsediyor.


### Tech Stack

URL yollarının index.php içerdiğini zaten fark etmiştim. Bunu görmeden önce, index uzantılarını da tahmin edebiliyordum ve giriş formunun da /index.php olarak yüklendiğini görebiliyordum.

404 sayfası mavi arka plan üzerinde gri metin ile klasik Laravel varsayılan 404 sayfasıdır:
![Pasted image 20241208043325.png](/img/user/resimler/Pasted%20image%2020241208043325.png)

Bunu fark etmeseydim, HTML'nin bazılarını aramak Laravel ile ilgili bazı sayfaları gösterirdi:

![Pasted image 20241208043349.png](/img/user/resimler/Pasted%20image%2020241208043349.png)
![Pasted image 20241208043356.png](/img/user/resimler/Pasted%20image%2020241208043356.png)


HTTP yanıt başlıkları da Laravel'i gösteren cookie'leri ayarlar:


![Pasted image 20241208043451.png](/img/user/resimler/Pasted%20image%2020241208043451.png)

Laravel her zaman bir XSRF-TOKEN ve [app]_session cookie'leri ayarlar. Varsayılan olarak [app] laravel'dir, ancak uygulama bunu değiştirebilir.


#### Directory Brute Force
Siteye karşı feroxbuster'ı çalıştıracağım ve sitenin PHP olduğunu bildiğim için -x php'yi dahil edeceğim, ancak hızlı bir şekilde bir ton hata döndürmeye başlıyor. Bu işe yaramayacak. Brute force'u yavaşlatmak için bazı şeyler yapabilirim, ancak kolay bir kutu için bu muhtemelen gerekli değildir.


### admin.usage.htb - TCP 80

Bu site farklı bir giriş sayfası sunar:

![Pasted image 20241208043642.png](/img/user/resimler/Pasted%20image%2020241208043642.png)

Diğer sitedeki yetkilerim çalışmıyor. 404 sayfası aynı ve form /index.php olarak yükleniyor, bu yüzden muhtemelen aynı uygulamanın bir parçası.


## Shell as dash

### SQL Injection


Tanımlama
Herhangi bir şeyin çöküp çökmediğini görmek için her zaman karşılaştığım her alanı tek bir tırnak işaretiyle test edeceğim. Parola sıfırlama formunda, e-posta olarak ' gönderildiğinde sayfa 500 döndürüyor:

![Pasted image 20241208043738.png](/img/user/resimler/Pasted%20image%2020241208043738.png)

Bu SQL enjeksiyonunun iyi bir göstergesidir. Muhtemelen veritabanında e-posta adresini aramak için bir sorgu yapıyor. Bunun şöyle göründüğünü tahmin edebiliyorum:

![Pasted image 20241208043754.png](/img/user/resimler/Pasted%20image%2020241208043754.png)

Eğer durum buysa, ' veya 1=1 limit 1;-- - gönderirsem, bu olur:

![Pasted image 20241208043806.png](/img/user/resimler/Pasted%20image%2020241208043806.png)

İşe yarıyor:

![Pasted image 20241208043820.png](/img/user/resimler/Pasted%20image%2020241208043820.png)

That’s SQL injection.



#### Exploitation

Gönderdiklerim geri görüntülenirken, veritabanından herhangi bir veri görüntülenmiyor gibi görünüyor. Görünüşe göre kod sadece yanıtların uzunluğunu kontrol ediyor ve gönderilen e-postayı gösteriyor.

Yani buradan veri almak için error-based ya da blind injection yapmak gerekecek. Bunun için sqlmap kullanacağım.

Burp'ta /forgot-password'e yasal (SQL enjeksiyonu olmayan) bir POST bulacağım, isteğe sağ tıklayıp “ Copy to file” diyeceğim. sqlmap bunu alacak ve enjeksiyonları arayacak:

![Pasted image 20241208044004.png](/img/user/resimler/Pasted%20image%2020241208044004.png)

Başarısız. Ama bunun enjekte edilebilir olduğunu biliyorum. Seviyeyi ve riski artırmayı deneyeceğim (ve konuları ve hızlandırmak için e-postaya odaklanmasını söyleyeceğim):

![Pasted image 20241208044031.png](/img/user/resimler/Pasted%20image%2020241208044031.png)


#### DB Enumeration

Şimdi sqlmap enjeksiyonu tanımladığına göre, onu DB'yi listelemek için kullanabilirim. Önceki komuta --dbs ekleyerek veritabanlarını listeleyerek başlayacağım:


![Pasted image 20241208044109.png](/img/user/resimler/Pasted%20image%2020241208044109.png)

information_schema ve performance_schema MySQL ile ilgilidir, usage_blog ise web sitesi ile ilgilidir. usage_blog içindeki tabloları listelemek için --dbs yerine -D usage_blog --tables kullanacağım:

![Pasted image 20241208044138.png](/img/user/resimler/Pasted%20image%2020241208044138.png)

Biraz yavaş, bu yüzden verileri seçerek dökmek isteyeceğim. admin_users tablosu ile başlayacağım, --tables yerine -T admin_users --dump kullanacağım:


`sqlmap -r reset.request --level 5 --risk 3 --threads 10 -p email --batch -D usage_blog -T admin_users --dump`

![Pasted image 20241208044215.png](/img/user/resimler/Pasted%20image%2020241208044215.png)

Bir kullanıcı var. Diğer tabloları da boşaltabilirim ama ihtiyacım olan tek şey bu.


### Crack Hash

Bu hash'i bir dosyaya kaydedeceğim ve hashcat'i rockyou.txt kelime listesiyle birlikte kullanarak kırmaya çalışacağım. Hash formatını tespit etmeye çalışmasına izin verirsem, birden fazla olasılık olduğundan şikayet edecektir:


![Pasted image 20241208044311.png](/img/user/resimler/Pasted%20image%2020241208044311.png)

Son üç durum, parolanın önce eski bir hash formatıyla ve ardından bcrypt ile hash edildiği durumlardır. Bu, kullanıcıların parolalarını değiştirmek zorunda kalmadan bir veritabanını sadece MD5 kullanmaktan BCrypt kullanmaya geçirmenin yaygın bir yoludur. Sadece her ikisini de yapacak şekilde ayarlayın ve şu anda DB'de bulunan tüm MD5'leri alın ve BCrypt ile güncelleyin.

Bu göz önüne alındığında, önce doğrudan BCrypt'i denemek mantıklıdır:

![Pasted image 20241208044340.png](/img/user/resimler/Pasted%20image%2020241208044340.png)

Benim sunucumda birkaç saniye içinde “whatever1” olarak kırılıyor.


### RCE

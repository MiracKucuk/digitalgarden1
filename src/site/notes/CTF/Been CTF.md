---
{"dg-publish":true,"permalink":"/ctf/been-ctf/"}
---

![Pasted image 20241026133856.png](/img/user/resimler/Pasted%20image%2020241026133856.png)

```powershell-session
nmap -sC -sV -p 22,25,80,110,111,143,443,745,993,995,3306,4190,4445,4559,5038,10000 10.10.10.7 

Nmap scan report for 10.10.10.7
Host is up (0.16s latency).

PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open   smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open   http       Apache httpd 2.2.3
|_http-title: Did not follow redirect to https://10.10.10.7/
|_http-server-header: Apache/2.2.3 (CentOS)
110/tcp   open   pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: EXPIRE(NEVER) TOP UIDL AUTH-RESP-CODE PIPELINING LOGIN-DELAY(0) USER STLS RESP-CODES IMPLEMENTATION(Cyrus POP3 server v2) APOP
111/tcp   open   rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            790/udp   status
|_  100024  1            793/tcp   status
143/tcp   open   imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: MULTIAPPEND OK URLAUTHA0001 X-NETSCAPE BINARY LIST-SUBSCRIBED LISTEXT RIGHTS=kxte IMAP4 CHILDREN ACL SORT STARTTLS THREAD=ORDEREDSUBJECT IDLE ANNOTATEMORE MAILBOX-REFERRALS IMAP4rev1 Completed UNSELECT NAMESPACE THREAD=REFERENCES UIDPLUS ID NO CONDSTORE RENAME CATENATE QUOTA SORT=MODSEQ ATOMIC LITERAL+
443/tcp   open   ssl/http   Apache httpd 2.2.3 ((CentOS))
|_http-server-header: Apache/2.2.3 (CentOS)
|_ssl-date: 2024-10-26T10:44:13+00:00; -1s from scanner time.
|_http-title: Elastix - Login page
| http-robots.txt: 1 disallowed entry 
|_/
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
745/tcp   closed unknown
993/tcp   open   ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open   pop3       Cyrus pop3d
3306/tcp  open   mysql      MySQL (unauthorized)
4190/tcp  open   sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open   upnotifyp?
4559/tcp  open   hylafax    HylaFAX 4.3.10
5038/tcp  open   asterisk   Asterisk Call Manager 1.1
10000/tcp open   http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix

Host script results:
|_clock-skew: -1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 395.31 seconds

```

### Öne Çıkan Servisler ve Portlar

1. **22/tcp - SSH**
    
    - **Servis:** OpenSSH 4.3 (protocol 2.0)
    - **Not:** SSH portunun açık olması uzaktan yönetim sağlıyor; ancak erişim için kullanıcı adı ve parolaya ihtiyaç var.
2. **25/tcp - SMTP (Postfix smtpd)**
    
    - **Detaylar:** Bazı SMTP komutları (PIPELINING, VRFY, vb.) destekleniyor.
    - **Not:** E-posta göndermek ve almak için kullanılan bu port, SMTP sunucusunun açık olduğunu gösteriyor ve varsa güvenlik açığı araştırılabilir.
3. **80/tcp - HTTP (Apache 2.2.3)**
    
    - **Detaylar:** Apache HTTP sunucusu çalışıyor, fakat HTTP istekleri `https://10.10.10.7/` adresine yönlendiriliyor.
    - **Not:** Web tarayıcı üzerinden hedef sisteme erişim sağlanabilir.
4. **443/tcp - HTTPS (SSL/TLS üzerinden Apache 2.2.3)**
    
    - **Detaylar:** SSL sertifikası geçersiz (2018'de sona ermiş), bu da HTTPS bağlantısında güvenlik uyarıları verebilir.
    - **Not:** `Elastix` isimli bir web uygulaması çalışıyor. Elastix, genellikle VoIP ve iletişim yönetimi için kullanılır.
5. **110/tcp ve 995/tcp - POP3 / POP3S**
    
    - **Servis:** Cyrus POP3 sunucusu çalışıyor.
    - **Not:** POP3, posta kutusuna erişim sağlıyor. `995/tcp` SSL ile güvenli bir POP3 bağlantısı sağlıyor.
6. **143/tcp ve 993/tcp - IMAP / IMAPS**
    
    - **Servis:** Cyrus IMAP sunucusu çalışıyor.
    - **Not:** IMAP, posta kutusuna uzaktan erişim sağlıyor ve `993/tcp` SSL üzerinden IMAP sağlıyor.
7. **3306/tcp - MySQL**
    
    - **Detaylar:** MySQL veritabanı çalışıyor, ancak kimlik doğrulama gerekiyor.
    - **Not:** MySQL güvenlik açıkları veya zayıf parolalarla hedef alınabilir.
8. **4190/tcp - Sieve (Cyrus timsieved)**
    
    - **Servis:** Cyrus'un Sieve sunucusu (e-posta filtreleme protokolü).
    - **Not:** E-posta yönlendirme veya filtreleme için kullanılabilir.
9. **4559/tcp - HylaFAX**
    
    - **Servis:** HylaFAX faks sunucusu (4.3.10)
    - **Not:** Faks servisi, bazı sistemlerde hassas verileri içerebilir.
10. **5038/tcp - Asterisk Call Manager**
    

- **Servis:** Asterisk 1.1 Call Manager
- **Not:** Asterisk, VoIP yönetimi için kullanılan bir platformdur ve erişim izni varsa çağrı bilgileri veya sistem yapılandırmaları yönetilebilir.

11. **10000/tcp - Webmin (MiniServ 1.570)**

- **Detaylar:** Webmin, genellikle sistem yönetimi için kullanılan bir araçtır.
- **Not:** Webmin'e erişim sağlanırsa hedef sistem yönetimi yapılabilir.

### Güvenlik İncelemesi ve Olası Güvenlik Açıkları

- **Elastix Web Arayüzü:** `443/tcp` üzerinde çalışan Elastix, kimlik doğrulama ve güvenlik açığı testleri için hedeflenebilir.
- **Webmin Yönetim Paneli:** `10000/tcp` üzerinden erişim sağlayan Webmin arayüzü, varsa varsayılan kimlik bilgileri ile denenebilir.
- **MySQL ve Posta Sunucuları:** Veritabanı ve posta sunucuları, kimlik bilgisi testleri veya açıklık taramaları için değerlendirilebilir.

Bu servislere yönelik güvenlik açıklarını analiz ederek zafiyetlerin olup olmadığını doğrulayabilirsiniz. Elastix, Webmin ve Asterisk gibi servislerde tanımlı güvenlik açıkları mevcut olabilir.


### Web - Port 80/443

Site

Port 80 sadece 443 / HTTPS portuna bir yönlendirme döndürür.

443'te bir Elastix giriş sayfası vardır:

Fakat siteye giremiyorum . Mozillada tsl ayarını düşürmemiz lazım . 

![Pasted image 20241026135606.png](/img/user/resimler/Pasted%20image%2020241026135606.png)

about:config 

![Pasted image 20241026135620.png](/img/user/resimler/Pasted%20image%2020241026135620.png)


- Arama çubuğuna `security.tls.version.min` yazın.
- Bu ayarın üzerine çift tıklayın ve değeri **1** olarak değiştirin. Bu, TLS 1.0'ı etkinleştirecektir.

![Pasted image 20241026135711.png](/img/user/resimler/Pasted%20image%2020241026135711.png)


Riski kabul edip Accept'e basalım .

![Pasted image 20241026135758.png](/img/user/resimler/Pasted%20image%2020241026135758.png)


![Pasted image 20241026135812.png](/img/user/resimler/Pasted%20image%2020241026135812.png)

Elastix, bir özel santral (PBX) yazılımıdır. PBX, bir kurumsal ağ içinde bir telefon / ses ağını kontrol eder ve onu ağın geri kalanına bağlar. Bu, kutu adı olan "Beep" temasına oldukça uygundur.


### Dizin Bruteforce


```powershell-session
gobuster dir -u https://10.10.10.7/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php -k -t 100

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.7/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 1785]
/images               (Status: 301) [Size: 310] [--> https://10.10.10.7/images/]
/help                 (Status: 301) [Size: 308] [--> https://10.10.10.7/help/]
/register.php         (Status: 200) [Size: 1785]
/themes               (Status: 301) [Size: 310] [--> https://10.10.10.7/themes/]
/modules              (Status: 301) [Size: 311] [--> https://10.10.10.7/modules/]
/mail                 (Status: 301) [Size: 308] [--> https://10.10.10.7/mail/]
/admin                (Status: 301) [Size: 309] [--> https://10.10.10.7/admin/]
/static               (Status: 301) [Size: 310] [--> https://10.10.10.7/static/]
/lang                 (Status: 301) [Size: 308] [--> https://10.10.10.7/lang/]
/config.php           (Status: 200) [Size: 1785]
/robots.txt           (Status: 200) [Size: 28]
/var                  (Status: 301) [Size: 307] [--> https://10.10.10.7/var/]
/panel                (Status: 301) [Size: 309] [--> https://10.10.10.7/panel/]
/libs                 (Status: 301) [Size: 308] [--> https://10.10.10.7/libs/]


```


#### /admin
Yönetici'ye gittiğinizde giriş yapmanızı ister:

![Pasted image 20241026141128.png](/img/user/resimler/Pasted%20image%2020241026141128.png)

Birkaç tahminden sonra iptal tuşuna basıyorum ve https://10.10.10.7/admin/config.php adresine gidiyorum, bu da yetkisiz bir mesaj yazdırıyor:

![Pasted image 20241026141208.png](/img/user/resimler/Pasted%20image%2020241026141208.png)


### LFI
Searchsploit, Elastix'te bir LFI olduğunu gösteriyor:

![Pasted image 20241026141318.png](/img/user/resimler/Pasted%20image%2020241026141318.png)

searchsploit -x exploits/php/webapps/37637.pl, /vtigercrm/graph.php sayfasında bir local file include (LFI) gösteriyor. current_language parametresi bir dosyayı işaret ediyor ve ../ filtrelemesi yok gibi görünüyor ve dizeyi kesmek için %00 geçebilirim. Bu %00, PHP'nin girdiyi aldığını ve eklemeden önce ona .php eklediğini gösteriyor. Gerçekten eski PHP örneklerinde, %00 eklemek dizeyi keser, böylece .php yok sayılır.


Code'u bulunduğumuz dizine kopyalayalım . 
![Pasted image 20241026141942.png](/img/user/resimler/Pasted%20image%2020241026141942.png)

nano ile koda baktığımızda şunlar yazıyor . 

Elastix, kullanıcı tarafından sağlanan girdiyi düzgün bir şekilde sterilize edemediği için yerel dosya ekleme güvenlik açığına eğilimlidir.

Bir saldırgan, web sunucusu işlemi bağlamında dosyaları görüntülemek ve yerel komut dosyalarını çalıştırmak için bu güvenlik açığından yararlanabilir. >

Elastix 2.2.0 savunmasızdır; diğer sürümler de etkilenmiş olabilir.

Bu açıklık, kullanıcıdan alınan `current_language` parametresindeki veriyi doğru şekilde temizlemediği için, saldırganın dosya yollarını doğrudan URL parametreleri ile manipüle etmesine olanak tanır.

Bu script, belirli bir dosya yolunu `graph.php` dosyasının URL'sine ekleyerek sunucuda `amportal.conf` gibi dosyaları okumaya çalışır.

**Scripti çalıştırma**:

- Bu Perl scriptini çalıştırmak için Perl’in yüklü olduğundan emin olun.
- Scripti bir dosyaya kaydedin, örneğin `elastix_lfi.pl` olarak kaydedebilirsiniz.


![Pasted image 20241026142905.png](/img/user/resimler/Pasted%20image%2020241026142905.png)


![Pasted image 20241026142844.png](/img/user/resimler/Pasted%20image%2020241026142844.png)

![Pasted image 20241026142943.png](/img/user/resimler/Pasted%20image%2020241026142943.png)

Diğer dosyaları da okuyabilirim, ancak POC, PBX için bir yapılandırma dosyası önerdi ve içinde alınması gereken bir dizi potansiyel parola var:  

- amp109  
- jEhdIekWmdjE  
- amp111  
- passw0rd

### Webmin - TCP 10000  

10000 portunda bir Webmin girişi var:

![Pasted image 20241026143506.png](/img/user/resimler/Pasted%20image%2020241026143506.png)

[Webmin](https://www.webmin.com/) Unix sistemlerini yönetmek için bir web arayüzüdür. Root / root gibi bazı temel şifreleri tahmin ediyorum ama başarılı olamadım. LFI'daki şifreleri daha sonraki bir bölümde deneyeceğim.


### SMTP - TCP 25  

SMTP ile kullanıcı hesaplarını kontrol edebilirim. Biraz hile yapacağım, çünkü LFI'ye sahibim ve `/etc/passwd` 'yi okuyarak kullanıcıların bir listesini oluşturabilirim:

![Pasted image 20241026143724.png](/img/user/resimler/Pasted%20image%2020241026143724.png)

![Pasted image 20241026144539.png](/img/user/resimler/Pasted%20image%2020241026144539.png)


![Pasted image 20241026144518.png](/img/user/resimler/Pasted%20image%2020241026144518.png)


![Pasted image 20241026144627.png](/img/user/resimler/Pasted%20image%2020241026144627.png)

![Pasted image 20241026144709.png](/img/user/resimler/Pasted%20image%2020241026144709.png)


https://0xdf.gitlab.io/2021/02/23/htb-beep.html

Farklı bakış açısı . 
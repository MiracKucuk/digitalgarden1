---
{"dg-publish":true,"permalink":"/ctf/certified-ctf/"}
---

Windows pentestlerinde yaygın olduğu gibi, Certified kutusunu aşağıdaki hesabın kimlik bilgileriyle başlatacaksınız: Kullanıcı adı: judith.mader Şifre: judith09

### Nmap 

```
nmap -sSCV -Pn -p- Certified.htb  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-19 04:34 EST

Nmap scan report for Certified.htb (10.10.11.41)
Host is up (0.17s latency).
rDNS record for 10.10.11.41: certified.htb
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-01-19 16:42:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-19T16:43:50+00:00; +7h00m03s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-19T16:43:49+00:00; +7h00m03s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-19T16:43:52+00:00; +7h00m03s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
|_ssl-date: 2025-01-19T16:43:49+00:00; +7h00m03s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
49713/tcp open  msrpc         Microsoft Windows RPC
49737/tcp open  msrpc         Microsoft Windows RPC
53695/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-01-19T16:43:11
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m02s, deviation: 0s, median: 7h00m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 585.78 seconds

```

---
1. **Hedef Bilgileri**:
    
    - IP: `10.10.11.41`
    - Hostname: `certified.htb`
    - rDNS kaydı: `certified.htb`

2. **Açık Portlar ve Servisler**:
    
    - **53/tcp**: `Simple DNS Plus`
    - **88/tcp**: `Kerberos`
    - **135/tcp, 139/tcp, 445/tcp**: `Microsoft Windows RPC/NetBIOS`
    - **389/tcp, 636/tcp, 3268/tcp, 3269/tcp**: `Active Directory LDAP` (bazıları SSL ile korunuyor)
    - **593/tcp, 49673/tcp**: `RPC over HTTP`
    - **5985/tcp**: `Microsoft HTTPAPI (WinRM)`
    - **9389/tcp**: `.NET Message Framing`

3. **SSL Sertifikası**:
    
    - CN: `DC01.certified.htb`
    - Geçerlilik: `2024-05-13` - `2025-05-13`

4. **Zaman Farkı**:
    
    - **Saat farkı (Clock Skew)**: 7 saat

5. **SMB Özellikleri**:
    
    - SMB protokolü zaman eşleşmesi mevcut (`smb2-time`).
    - Mesaj imzalama etkin ve gerekli (`smb2-security-mode`).

6. **İşletim Sistemi ve Host**:
    
    - OS: **Microsoft Windows**
    - Hostname: `DC01`

### Önemli Sonuçlar:

- Active Directory'ye dair LDAP ve Kerberos açıkları mevcut.
- WinRM portu (`5985`) açık, bu yönetim için kullanılabilir.
- SMB güvenlik imzalama aktif, bu da saldırı senaryolarını sınırlayabilir.

---


## GetAllUserName

![Pasted image 20250119125149.png](/img/user/resimler/Pasted%20image%2020250119125149.png)

---

- **`--rid-brute`**:  
    RID (Relative Identifier) brute-forcing işlemi yapar.
    
    - RID, bir Windows sisteminde kullanıcı veya grup kimliklerini belirlemek için kullanılan bir sayıdır.
    - Bu parametre, SMB üzerinden RID değerlerini brute-force yöntemiyle kontrol eder ve kullanıcılar ile grupları listeleyebilir.
- **`| grep SidTypeUser`**:  
    Komutun çıktısını filtrelemek için kullanılır. Yalnızca **`SidTypeUser`** içeren satırları gösterecektir.
    
    - Bu, yalnızca kullanıcı hesaplarını (SidTypeUser) listelemek için kullanılır.

----


## Bloodhound

![Pasted image 20250119125550.png](/img/user/resimler/Pasted%20image%2020250119125550.png)

Analiz için bloodhoundGUI'ye aktarıldı.

Judith kullanıcısının Management grubunun sahibini değiştirebildiği veya grubun access control list'ini (ACL) değiştirebildiği keşfedildi.

![Pasted image 20250119125712.png](/img/user/resimler/Pasted%20image%2020250119125712.png)

Ardından bloodhound yazarak GUI'yi açıyoruz. 

Analiz için bloodhoundGUI'ye aktarıldı.


![Pasted image 20250119130234.png](/img/user/resimler/Pasted%20image%2020250119130234.png)

Unmark User as Owned diyoruz çünkü bu hesabın credential'ları mevcut.

Judith kullanıcısının Management grubunun sahibini değiştirebildiği veya grubun access control list'ini (ACL) değiştirebildiği keşfedildi.

![Pasted image 20250119130448.png](/img/user/resimler/Pasted%20image%2020250119130448.png)

![Pasted image 20250119130402.png](/img/user/resimler/Pasted%20image%2020250119130402.png)

![Pasted image 20250119132426.png](/img/user/resimler/Pasted%20image%2020250119132426.png)

Management grubu SVC'lerden birine yazabilir.

![Pasted image 20250119130716.png](/img/user/resimler/Pasted%20image%2020250119130716.png)

SVC CA_OPERATOR üzerinde tam kontrole sahiptir.

![Pasted image 20250119130919.png](/img/user/resimler/Pasted%20image%2020250119130919.png)

![Pasted image 20250119130941.png](/img/user/resimler/Pasted%20image%2020250119130941.png)


## User

Management grubunun ACL'sini judith.mader kullanıcısına grubun üyelerini yönetmek için WriteMembers izni verecek şekilde değiştirin

```
impacket-dacledit  -action 'write' -rights 'WriteMembers' -target-dn "CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB" -principal "judith.mader" "certified.htb/judith.mader:judith09"
```

![Pasted image 20250119131350.png](/img/user/resimler/Pasted%20image%2020250119131350.png)

---

- **`-action 'write'`** → **Yapılacak işlemi belirtir. `'write'`: ACL üzerinde yazma işlemi yapılacağını belirtir.**
    
- **`-rights 'WriteMembers'`** → **Hangi tür hakların ekleneceğini belirtir. `'WriteMembers'`: Belirtilen kullanıcıya, hedef objenin üyelerini ekleme veya kaldırma hakkı verir.**
    
- **`-target-dn "CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB"`** → **Hedef nesnenin Distinguished Name'ini (DN) belirtir. `CN=MANAGEMENT`: **Grup Adı** (Management grubu), `CN=USERS`: Bu grup, `Users` konteyneri altında bulunur, `DC=CERTIFIED,DC=HTB`: **Domain adı (certified.htb**).
    
- **`-principal "judith.mader"`** → **ACL düzenlemesi yapılan kullanıcıyı/grubu belirtir.** **Burada:** `judith.mader` kullanıcısı, hedef nesne için hak kazanacak.
    
- **`certified.htb/judith.mader:judith09`** → **Kullanıcı kimlik doğrulama bilgileri: `certified.htb`: Domaini, `judith.mader`: Kullanıcı adı, `judith09`: Kullanıcı parolası.**

---

Judith'in kendisini Management grubuna ekleyin. 

![Pasted image 20250119141031.png](/img/user/resimler/Pasted%20image%2020250119141031.png)

**Domain Sürekliliği: Shadow Credentials – FreeBuf Ağ Güvenliği Sektör Portalı**  
**KillingTree/pywhisker: "Shadow Credentials" saldırıları için C# aracının Python versiyonu (github.com)**

Araç bir args hatası verirse, main() fonksiyonuna gidin ve args için ayrıştırma çağrısından önce globals args ekleyin.

pywhisker.py aracı bir Kerberos kimlik doğrulama baypası gerçekleştirir ve hedef nesnenin msDS-KeyCredentialLink özelliğini değiştirerek hedefle ilişkili sertifika ve anahtarı oluşturur ve saklar.

![Pasted image 20250119142003.png](/img/user/resimler/Pasted%20image%2020250119142003.png)

Sonra, management_svc için TGT (Ticket Granting Ticket) talep edin. Eğer hata alırsanız, zaman dilimini senkronize etmek için ntpdate kullanmanız gerekebilir.

PKINITtools içindeki gettgtpkinit.py script'ini kullanarak bir Kerberos TGT talep edin ve daha önce oluşturduğunuz sertifika ve anahtarları kullanın.

![Pasted image 20250119143359.png](/img/user/resimler/Pasted%20image%2020250119143359.png)

![Pasted image 20250119143458.png](/img/user/resimler/Pasted%20image%2020250119143458.png)
Daha önce elde edilmiş TGT ile management_svc hesabının NT hash'ini istemek ve kurtarmak için PKINITtools içindeki getnthash.py scriptini kullanın

![Pasted image 20250119143654.png](/img/user/resimler/Pasted%20image%2020250119143654.png)

NThash ile remote oturum açmak için Evil-winrm kullanabilirsiniz.

![Pasted image 20250119143806.png](/img/user/resimler/Pasted%20image%2020250119143806.png)


## Root

![Pasted image 20250119144023.png](/img/user/resimler/Pasted%20image%2020250119144023.png)

management_svc, ca_operator üzerinde tam kontrole sahiptir.

ca'nın parolasını değiştirmeyi deneyin

/etc/host'a DC01.certified.htb'yi ekleyin

![Pasted image 20250119144637.png](/img/user/resimler/Pasted%20image%2020250119144637.png)

Değişikliğin başarılı olup olmadığını kontrol edin

![Pasted image 20250119144744.png](/img/user/resimler/Pasted%20image%2020250119144744.png)

### No Security Extension – ESC9

- [AD CS Domain Escalation | HackTricks](https://book.hacktricks.xyz/cn/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#id-5485)

Certipy aracını kullanarak judith.mader@certified.htb kullanıcısı ile ilgili sertifika bilgilerini bulup elde etmek ve certified-DC01-CA Certificate Authority (CA) yapılandırmasını elde etmektir.

![Pasted image 20250119145113.png](/img/user/resimler/Pasted%20image%2020250119145113.png)

"NoSecurityExtension bulundu, bu nedenle ESC9 saldırısı yapılabilir.

Certipy aracını kullanarak NT hash (a091c1832bcdd4677c28b5a6a1295584) ca_operator hesabına güncelledim ve bu hesabın userPrincipalName (UPN) değerini Administrator olarak değiştirdim."

![Pasted image 20250119145220.png](/img/user/resimler/Pasted%20image%2020250119145220.png)

Bir sertifika başarıyla talep edildi ve Administrator hesabıyla ilişkilendirildi (talep ca_operator kullanıcısı için olsa bile). Bu, artık Administrator hesabını temsil edebilecek bir sertifika olduğu anlamına gelir.

![Pasted image 20250119145343.png](/img/user/resimler/Pasted%20image%2020250119145343.png)

Certipy aracını kullanarak administrator.pfx sertifika dosyası aracılığıyla administrator@certified.htb kullanıcısı olarak Kerberos kimlik doğrulaması ve başarıyla TGT ve NT hash elde etme

![Pasted image 20250119145445.png](/img/user/resimler/Pasted%20image%2020250119145445.png)

Son olarak, evil-winrm ile oturum açarak root.txt dosyasını alabilirsiniz.

![Pasted image 20250119145605.png](/img/user/resimler/Pasted%20image%2020250119145605.png)

![Pasted image 20250119145622.png](/img/user/resimler/Pasted%20image%2020250119145622.png)


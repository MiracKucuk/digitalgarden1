---
{"dg-publish":true,"permalink":"/ctf/grandpa-ctf/"}
---


Grandpa gerçekten ilk HTB makinelerinden biriydi. Bugün HTB'de görünmeyecek türden bir kutu ve açıkçası modern hedefler kadar eğlenceli değil. Yine de, OSCP'de göreceğiniz türden şeyler için harika bir proxy ve özellikle Metasploit olmadan çalışmaya çalışırsanız bazı değerli dersler öğretiyor. Metasploit ile bu kutu muhtemelen birkaç dakika içinde çözülebilir. Tipik olarak, Metasploit'ten kaçınmanın değeri, açıkları ve neler olup bittiğini gerçekten anlayabilmekten gelir. Bu durumda, daha çok dosyaları hareket ettirme, binaryleri bulma vb. ile ilgilidir.

## Box Info

|Name|[Grandpa](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fgrandpa)[![Grandpa](https://0xdf.gitlab.io/icons/box-grandpa.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fgrandpa)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fgrandpa)|
|---|---|
|Release Date|12 Apr 2017|
|Retire Date|21 Nov 2017|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Grandpa](https://0xdf.gitlab.io/img/grandpa-diff.png)|
|Radar Graph|![Radar chart for Grandpa](https://0xdf.gitlab.io/img/grandpa-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|04:25:06[![v4l3r0n](https://www.hackthebox.com/badge/image/68)](https://app.hackthebox.com/users/68)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|04:25:34[![v4l3r0n](https://www.hackthebox.com/badge/image/68)](https://app.hackthebox.com/users/68)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap


![Pasted image 20250208193947.png](/img/user/resimler/Pasted%20image%2020250208193947.png)


![Pasted image 20250208190959.png](/img/user/resimler/Pasted%20image%2020250208190959.png)

IIS sürümüne bağlı olarak, host muhtemelen Windows XP veya Server 2003 çalıştırmaktadır.


### Website - TCP 80

#### Site

Siteyi ziyaret ettiğinizde, sadece “Yapım Aşamasında” mesajı var:

![Pasted image 20250208194332.png](/img/user/resimler/Pasted%20image%2020250208194332.png)

#### Directory Brute Force

Siteye karşı gobuster'ı çalıştırıyorum ve iki dizin buluyor:

```
root@kali# gobuster dir -u http://10.10.10.14 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -o scans/gobuster-root-lowercasesmall =============================================================== Gobuster v3.0.1 by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_) =============================================================== [+] Url: http://10.10.10.14 [+] Threads: 10 [+] Wordlist: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt [+] Status codes: 200,204,301,302,307,401,403 [+] User Agent: gobuster/3.0.1 [+] Timeout: 10s =============================================================== 2020/05/27 16:01:06 Starting gobuster =============================================================== /images (Status: 301) /_private (Status: 403) =============================================================== 2020/05/27 16:03:19 Finished ===============================================================
```

Ne yazık ki, /images içinde hiçbir şey bulamıyorum ve içerikleri listelemiyor ve `/_private'den` sadece 403 alıyorum.


#### WebDAV
Bu bana [Granny](https://0xdf.gitlab.io/2019/03/06/htb-granny.html#webdav)'yi hatırlattı ve nmap çıktısındaki tüm WebDAV'ı göz önüne alarak randavtest yaptım. Granny'den farklı olarak, hiçbir şey etkin değil:

```
root@kali# davtest -url http://10.10.10.14
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.14
********************************************************
NOTE    Random string for this session: NHwzc9ANv
********************************************************
 Creating directory
MKCOL           FAIL
********************************************************
 Sending test files
PUT     html    FAIL
PUT     php     FAIL
PUT     txt     FAIL
PUT     jsp     FAIL
PUT     cfm     FAIL
PUT     shtml   FAIL
PUT     asp     FAIL
PUT     cgi     FAIL
PUT     aspx    FAIL
PUT     jhtml   FAIL
PUT     pl      FAIL

********************************************************
/usr/bin/davtest Summary:
```


#### Vulnerabilities

IIS 6.0 çok eski olduğu için güvenlik açıklarını aramaya karar verdim ve bazılarını buldum:

![Pasted image 20250208195913.png](/img/user/resimler/Pasted%20image%2020250208195913.png)

## Shell as network service

### Select Vulnerability

SearchSploit listesindeki en ilginç güvenlik açığı **WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow** oldu. Bu, **Grandpa** makinesinin sürümüyle eşleşiyor ve **uzaktan kod çalıştırma (RCE)** imkanı sağlıyor. Ayrıca, **Nmap WebDAV taraması** birçok bilgi döndürdü ve bu, WebDAV’a karşı bir exploit.

Exploit kodunu incelemek için şu komutu kullanacağım:

```
searchsploit -x windows/remote/41738.py
```

Bu exploit, **HTTP isteği şeklinde bir payload oluşturuyor** ve içinde **calc.exe çalıştıran shellcode** barındırıyor.

Google’da araştırınca, bunun **CVE-2017-7269** olduğunu gördüm ve birçok **kamuya açık exploit scripti** mevcut.


![Pasted image 20250208202040.png](/img/user/resimler/Pasted%20image%2020250208202040.png)

![Pasted image 20250208202022.png](/img/user/resimler/Pasted%20image%2020250208202022.png)


bu ada biraz karışık devamını sonra çöz şimdilik çözüm aşağıdaki gibi 

![Pasted image 20250208202436.png](/img/user/resimler/Pasted%20image%2020250208202436.png)

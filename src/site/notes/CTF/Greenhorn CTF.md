---
{"dg-publish":true,"permalink":"/ctf/greenhorn-ctf/"}
---


![Pasted image 20241028230746.png](/img/user/resimler/Pasted%20image%2020241028230746.png)



```shell-session
nmap 10.10.11.25 -sCV -p 22,80,3000                                               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-28 16:20 EDT
Nmap scan report for greenhorn.htb (10.10.11.25)
Host is up (0.11s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Welcome to GreenHorn ! - GreenHorn
|_Requested resource was http://greenhorn.htb/?file=welcome-to-greenhorn
|_http-generator: pluck 4.7.18
| http-robots.txt: 2 disallowed entries 
|_/data/ /docs/
|_http-server-header: nginx/1.18.0 (Ubuntu)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=f40efbf271cd18fe; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=v1Z-wmfAoBBqdUqaUNGlfBQeVS06MTczMDE0Njg0MTU2NTU4MTgzOA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 28 Oct 2024 20:20:41 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=98b477327eb7651d; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=Wm2EyQcFdnezTwK-yezdJnwQYfk6MTczMDE0Njg0NzMzNTA0NDk1NQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 28 Oct 2024 20:20:47 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=10/28%Time=671FF219%P=x86_64-pc-linux-gnu%
SF:r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(GetRequest,2A60,"HTTP/1\.0\x20200\x20OK\r\nCache-Cont
SF:rol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_gi
SF:tea=f40efbf271cd18fe;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Co
SF:okie:\x20_csrf=v1Z-wmfAoBBqdUqaUNGlfBQeVS06MTczMDE0Njg0MTU2NTU4MTgzOA;\
SF:x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Op
SF:tions:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2028\x20Oct\x202024\x2020:20:41\
SF:x20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"th
SF:eme-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=de
SF:vice-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\
SF:x20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoi
SF:R3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA
SF:6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbm
SF:hvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciL
SF:CJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAv
SF:YX")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\
SF:x20Request")%r(HTTPOptions,197,"HTTP/1\.0\x20405\x20Method\x20Not\x20Al
SF:lowed\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Control:\x20max-age=0
SF:,\x20private,\x20must-revalidate,\x20no-transform\r\nSet-Cookie:\x20i_l
SF:ike_gitea=98b477327eb7651d;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\n
SF:Set-Cookie:\x20_csrf=Wm2EyQcFdnezTwK-yezdJnwQYfk6MTczMDE0Njg0NzMzNTA0ND
SF:k1NQ;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Fr
SF:ame-Options:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2028\x20Oct\x202024\x2020:
SF:20:47\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,67,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset
SF:=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.91 seconds

```


- **SSH (Port 22)**:
    - Açık ve **OpenSSH 8.9p1** sürümünü çalıştırıyor. Bu sürümle birlikte SSH protokolünün 2.0 sürümü kullanılıyor ve Ubuntu üzerinde çalıştığını görüyoruz. OpenSSH’de zafiyetler için sürüm özelinde araştırma yaparak, brute force veya parola tabanlı güvenlik açıklarına dair testler yapılabilir.
- **HTTP (Port 80)**:
    
    - **nginx 1.18.0** sürümünde bir HTTP sunucusu çalışıyor ve `http://greenhorn.htb/` adresine yönlendiriyor. Bu, hedefteki web uygulamasının potansiyel açığa çıkmasına yol açabilecek HTTP güvenlik açıklarını aramak için başlangıç noktası olabilir. Ayrıca, alan adını `/etc/hosts` dosyasına eklemek ve tarayıcıda görüntülemek için kullanabiliriz.
    - nginx’de sıkça karşılaşılan güvenlik açıkları arasında güvenlik yanlış yapılandırmaları, dosya dizini sızdırma ve zayıf içerik güvenlik politikaları yer alır.
- **Port 3000**:
    
    - Çıkışta "Gitea" adıyla ilişkili bilgiler var; bu, Gitea’nın açık kaynak bir Git servis platformu olduğunu düşündürüyor. Oturum açma çerezleri (`i_like_gitea`) ve _csrf çerezleri kullanılıyor. Bu, uygulamanın kimlik doğrulama ve CSRF koruma mekanizmaları olduğunu gösteriyor, ancak bu alanlarda güvenlik açıkları olabilir.
    - Gitea veya web tabanlı diğer git servislerinde bilinen zafiyetler (örneğin, kimlik doğrulama açıkları, yetkisiz erişim veya kod depolarının sızması gibi) kontrol edilebilir.

![Pasted image 20241028233237.png](/img/user/resimler/Pasted%20image%2020241028233237.png)

![Pasted image 20241028233447.png](/img/user/resimler/Pasted%20image%2020241028233447.png)
login.php adresine gidin ve pluk'un sürümünün 4.7.18 olduğunu görün

## CVE-2023-50564

![Pasted image 20241028233557.png](/img/user/resimler/Pasted%20image%2020241028233557.png)

İlgili güvenlik açıklarını sorguladıktan sonra, RCE'nin önce bir dosyanın yüklenmesini gerektirdiği keşfedildi.

3000 numaralı porta girme

![Pasted image 20241028233846.png](/img/user/resimler/Pasted%20image%2020241028233846.png)
![Pasted image 20241028233916.png](/img/user/resimler/Pasted%20image%2020241028233916.png)

Dikkat edersek web sitesinin github'u 

pass.php dosyasını bulun ve şifresini çözmeye çalışın

![Pasted image 20241028234145.png](/img/user/resimler/Pasted%20image%2020241028234145.png)

![Pasted image 20241028234221.png](/img/user/resimler/Pasted%20image%2020241028234221.png)

![Pasted image 20241028234330.png](/img/user/resimler/Pasted%20image%2020241028234330.png)

![Pasted image 20241028234426.png](/img/user/resimler/Pasted%20image%2020241028234426.png)

![Pasted image 20241028234501.png](/img/user/resimler/Pasted%20image%2020241028234501.png)

Başarıyla giriş yapın, installmodule'e gidin

![Pasted image 20241028235130.png](/img/user/resimler/Pasted%20image%2020241028235130.png)

Burada sıçrama shell dosyası sıkıştırılır ve yüklenir


[php-reverse-shell/php-reverse-shell.php at master · pentestmonkey/php-reverse-shell (github.com)](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
![Pasted image 20241028235257.png](/img/user/resimler/Pasted%20image%2020241028235257.png)

![Pasted image 20241029002345.png](/img/user/resimler/Pasted%20image%2020241029002345.png)

shell'in ziplenmiş halini sisteme yüklüyoruz ve reverse shellimiz geliyor

İki kullanıcı var, git ve junior, junior girebiliyor ama user.txt dosyasını okuma izni yok.
![Pasted image 20241029002503.png](/img/user/resimler/Pasted%20image%2020241029002503.png)

![Pasted image 20241029002551.png](/img/user/resimler/Pasted%20image%2020241029002551.png)

![Pasted image 20241029002635.png](/img/user/resimler/Pasted%20image%2020241029002635.png)


## Privilege Escalation

Bu pdf dosyasını indirmeyi deneyin

Nc bağlantısı kullanarak aktarım

Pdf dosayasını actıgımızda sansürlü sansürü kaldırdık bir program sayesinde link aşağıda . 


....

spipm/Depix: Pikselleştirilmiş ekran görüntülerinden parolaları kurtarır (github.com)

[spipm/Depix: Recovers passwords from pixelized screenshots (github.com)](https://github.com/spipm/Depix)

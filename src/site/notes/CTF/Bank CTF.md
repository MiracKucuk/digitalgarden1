---
{"dg-publish":true,"permalink":"/ctf/bank-ctf/"}
---

Bank oldukça basit bir makineydi, ancak iki önemli adım beklenmedik alternatif yöntemlere sahipti. Öncelikle, bir **hostname** bulmak için **DNS** üzerinde **enumeration** yapacağım ve bunu bir banka web sitesine erişmek için kullanacağım. Giriş bilgilerini, bir veri dizininde bularak elde edebilir veya **HTTP 302 yönlendirmelerindeki verileri analiz ederek** kimlik doğrulamasını tamamen atlayabilirim.

Bundan sonra, **filtreleri atlatıp** bir **PHP web shell** yükleyerek sistemde bir **shell** açacağım. **Root** olmak için, yönetici tarafından bırakılmış bir **SUID olarak çalışan dash binary’sini** kullanabilir veya **/etc/passwd dosyasına yazma yetkisini** kullanarak bir istismar gerçekleştirebilirim.

**Beyond Root** aşamasında, **302 yönlendirmelerindeki kodlama hatasını inceleyerek**, ayrıca SUID binary’sinin **dash** olduğunu nasıl belirlediğimi göstereceğim.


## Box Info

|Name|[Bank](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbank)[![Bank](https://0xdf.gitlab.io/icons/box-bank.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbank)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fbank)|
|---|---|
|Release Date|16 Jun 2017|
|Retire Date|16 Sept 2017|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Bank](https://0xdf.gitlab.io/img/bank-diff.png)|
|Radar Graph|![Radar chart for Bank](https://0xdf.gitlab.io/img/bank-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:33:37[![ahmed](https://www.hackthebox.com/badge/image/285)](https://app.hackthebox.com/users/285)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:45:21[![echthros](https://www.hackthebox.com/badge/image/2846)](https://app.hackthebox.com/users/2846)|
|Creator|[![makelarisjr](https://www.hackthebox.com/badge/image/95)](https://app.hackthebox.com/users/95)|

## Recon

### nmap

nmap, SSH (22), DNS (53) ve HTTP (80) olmak üzere üç açık TCP portu ve ayrıca UDP 53 üzerinde DNS buldu:

![Pasted image 20250210015325.png](/img/user/resimler/Pasted%20image%2020250210015325.png)

```
nmap -p 22,53,80 -sCV 10.10.10.29 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-09 17:48 EST
Nmap scan report for 10.10.10.29
Host is up (0.14s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.31 seconds
```

```
root@kali# nmap -p- -sU --min-rate 10000 10.10.10.29
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-01 06:17 EDT
Warning: 10.10.10.29 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.29
Host is up (0.016s latency).
Not shown: 65456 open|filtered ports, 78 closed ports
PORT   STATE SERVICE
53/udp open  domain

Nmap done: 1 IP address (1 host up) scanned in 72.94 seconds
```

OpenSSH ve Apache sürümlerine dayanarak, host muhtemelen Ubunutu 14.04 Trusty çalıştırıyor.


### DNS - TCP/UDP 53

TCP 53'ü gördüğümde ilk kontrol ettiğim şey bir zone transferi. Host/domain adına dair herhangi bir ipucu göremiyorum, bu yüzden bank.htb olabileceğini tahmin ediyorum ve bu işe yarıyor:

![Pasted image 20250210020822.png](/img/user/resimler/Pasted%20image%2020250210020822.png)
Lokal /etc/hosts dosyama aşağıdakileri ekleyeceğim:

```
10.10.10.29 bank.htb chris.bank.htb ns.bank.htb www.bank.htb
```


### Website by IP - TCP 80

Site sadece Apache2 varsayılan sayfasıdır:

![Pasted image 20250210021838.png](/img/user/resimler/Pasted%20image%2020250210021838.png)

Siteye karşı gobuster'ı çalıştıracağım, ancak hiçbir şey bulamıyorum.

http://www.bank.htb, http://chris.bank.htb, http://ns.bank.htb adreslerini ziyaret etmek de bu siteye yönlendiriyor.


### Scan for Virtual Hosts

Wfuzz kullanarak virtual hostlar için hızlı bir tarama yaptım:

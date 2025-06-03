---
{"dg-publish":true,"permalink":"/active-directory/nmap/"}
---

TCP-SYN taraması (`-sS`) en popüler yöntemdir. SYN bayrağı gönderilir, tam bağlantı kurulmaz:

- SYN-ACK cevabı: Port açık
- RST cevabı: Port kapalı
- Hiç cevap gelmezse: Filtrelenmiş

```shell-session
[!bash!]$ sudo nmap -sS localhost

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 22:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```

# Host Discovery

```shell-session
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

Bu tarama yöntemi yalnızca hostların güvenlik duvarları buna izin veriyorsa çalışır.

| **Scanning Options** | **Description**                                          |
| -------------------- | -------------------------------------------------------- |
| `10.129.2.0/24`      | Hedef ağ aralığı.                                        |
| `-sn`                | Port taramasını devre dışı bırakır.                      |
| `-oA tnet`           | Sonuçları 'tnet' adıyla başlayan tüm formatlarda saklar. |

## Scan Multiple IPs

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

Eğer port taramasını (`-sn`) devre dışı bırakırsak, Nmap otomatik olarak `ICMP Echo Requests (-PE)` ile ping taraması yapar. Böyle bir istek gönderildikten sonra, ping atan host canlıysa genellikle bir `ICMP reply` bekleriz. Daha da ilginci, önceki taramalarımız bunu yapmıyordu çünkü Nmap bir `ICMP echo` isteği göndermeden önce, bir `ARP reply` ile sonuçlanan bir `ARP pingi` gönderiyordu. Bunu “`--packet-trace`” seçeneği ile doğrulayabiliriz. ICMP echo isteklerinin gönderildiğinden emin olmak için, bunun için (`-PE`) seçeneğini de tanımlıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

Nmap'in hedefimizi neden “LİVE” olarak işaretlediğini belirlemenin bir başka yolu da “`--reason`” seçeneğidir.

`--reason` parametresi, Nmap'in her port için neden belirli bir durumu (açık, kapalı, filtresiz vb.) tespit ettiğini göstermesini sağlar. Yani, hangi nedenlerden dolayı bir portun açık veya kapalı olduğunu belirten bilgi verir. Bu, tarama sonuçlarını daha iyi anlamanıza yardımcı olabilir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:10 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

Burada Nmap'in gerçekten de sadece ARP request ve ARP reply ile hostun canlı olup olmadığını tespit ettiğini görüyoruz. ARP isteklerini devre dışı bırakmak ve hedefimizi istediğimiz ICMP echo istekleri ile taramak için “`--disable-arp-ping`” seçeneğini ayarlayarak ARP pinglerini devre dışı bırakabiliriz. Daha sonra hedefimizi tekrar tarayıp gönderilen ve alınan paketlere bakabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

Host keşfi hakkında daha fazla stratejiyi şu adreste bulabilirsiniz:

https://nmap.org/book/host-discovery-strategies.html


Soru  : Son sonuca göre, hangi işletim sistemine ait olduğunu bulun. Sonuç olarak işletim sisteminin adını gönderin.

Cevap : Windows (ttl değeri 128 olduğu için)


# Host and Port Scanning

| **State**            | **Description**                                                                                                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **open**             | Taranan porta bağlantının kurulduğunu gösterir. Bu bağlantılar TCP bağlantıları, UDP datagramları veya SCTP birleşimleri olabilir.                                                    |
| **closed**           | Portun kapalı olarak gösterilmesi, TCP protokolünün aldığımız paketin RST bayrağı içerdiğini belirttiği anlamına gelir. Hedefin aktif olup olmadığını belirlemek için kullanılabilir. |
| **filtered**         | Nmap, taranan portun açık mı yoksa kapalı mı olduğunu doğru bir şekilde tespit edemez çünkü ya hedeften port için bir yanıt alınmaz ya da hedeften bir hata kodu alırız.              |
| **unfiltered**       | Bu port durumu yalnızca `TCP-ACK` taraması sırasında ortaya çıkar. Porta erişilebilir, ancak açık mı yoksa kapalı mı olduğunun belirlenemediği anlamına gelir.                        |
| **open\|filtered**   | Belirli bir port için yanıt alınamadığında Nmap bu durumu ayarlar. Bu, portun bir güvenlik duvarı veya paket filtresi tarafından korunduğunu gösterebilir.                            |
| **closed\|filtered** | Bu durum yalnızca `IP ID` idle taramalarında ortaya çıkar. Taranan portun kapalı mı yoksa bir güvenlik duvarı tarafından filtrelenip filtrelenmediğinin belirlenemediğini ifade eder. |


## Discovering Open TCP Ports

Varsayılan olarak, Nmap en çok kullanılan 1000 TCP portunu SYN taraması (`-sS`) ile tarar. Bu SYN taraması, yalnızca `root` kullanıcısı olarak çalıştırıldığında varsayılan olarak ayarlanır, çünkü raw TCP paketleri oluşturmak için gereken soket izinlerine ihtiyaç duyar. Aksi takdirde, varsayılan olarak TCP taraması (`-sT`) gerçekleştirilir. Bu, eğer portları ve tarama yöntemlerini tanımlamazsak, bu parametrelerin otomatik olarak ayarlandığı anlamına gelir. Portları tek tek tanımlayabiliriz (-p 22,25,80,139,445), aralık olarak belirtebiliriz (-p 22-445), Nmap veritabanında en sık kullanılan olarak işaretlenmiş üst portları tarayabiliriz (`--top-ports=10`), tüm portları tarayabiliriz (-p-) veya hızlı port taraması yapabiliriz, ki bu da en çok kullanılan 100 portu içerir (-F).

- **Root değilseniz:** Nmap varsayılan olarak **TCP Connect Scan** (-sT) yapar.
- **Root iseniz:** Nmap varsayılan olarak **SYN Scan** (-sS) yapar.


#### Scanning Top 10 TCP Ports

```shell-session
sudo nmap 10.129.2.28 --top-ports=10 
```



#### Nmap - Trace the Packets

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```


|                      |                                      |
| -------------------- | ------------------------------------ |
| `-n`                 | DNS resolution'ı devre dışı bırakır. |
| `--disable-arp-ping` | ARP ping'i devre dışı bırakır.       |

**SENT** satırından, hedefimize (10.129.2.28) SYN bayraklı (S) bir TCP paketi gönderdiğimizi (10.10.14.2) görebiliriz. Sonraki **RCVD** satırında, hedefin RST ve ACK bayrakları (RA) içeren bir TCP paketiyle yanıt verdiğini görebiliriz. RST ve ACK bayrakları, TCP paketinin alındığını onaylamak (ACK) ve TCP oturumunu sonlandırmak (RST) için kullanılır.


#### Request
| **Mesaj**                                                     | **Açıklama**                                                                             |
| ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **SENT (0.0429s)**                                            | Nmap'in SENT işlemini gösterir, hedefe bir paket gönderir.                               |
| **TCP**                                                       | Hedef portla etkileşimde kullanılan protokolü gösterir.                                  |
| **10.10.14.2:63090 >**                                        | Nmap'in paketleri göndermek için kullandığı IPv4 adresimizi ve kaynak portu temsil eder. |
| **10.129.2.28:21**                                            | Hedefin IPv4 adresini ve hedef portu gösterir.                                           |
| **S**                                                         | Gönderilen TCP paketinin SYN bayrağını belirtir.                                         |
| **ttl=56 id=57322 iplen=44 seq=1699105818 win=1024 mss 1460** | TCP başlığına ait ek parametreleri gösterir.                                             |


#### Response

| **Mesaj**                            | **Açıklama**                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------- |
| **RCVD (0.0573s)**                   | Hedeften alınan bir paketi gösterir.                                            |
| **TCP**                              | Kullanılan protokolü gösterir.                                                  |
| **10.129.2.28:21 >**                 | Hedefin IPv4 adresini ve yanıt vermek için kullanılan kaynak portu temsil eder. |
| **10.10.14.2:63090**                 | Bizim IPv4 adresimizi ve yanıtın gönderileceği portu gösterir.                  |
| **RA**                               | Gönderilen TCP paketinin RST ve ACK bayraklarını belirtir.                      |
| **ttl=64 id=0 iplen=40 seq=0 win=0** | TCP başlığına ait ek parametreleri gösterir.                                    |

### Connect Scan 

Nmap TCP Connect Scan (-sT), hedef bir sistemde belirli bir portun açık veya kapalı olup olmadığını belirlemek için TCP üçlü el sıkışmasını (three-way handshake) kullanır. Tarama, hedef porta bir SYN paketi gönderir ve bir yanıt bekler. Eğer hedef port bir SYN-ACK paketi ile yanıt verirse, port açık olarak kabul edilir. Eğer RST paketi ile yanıt verirse, port kapalıdır.

Connect Scan (tam TCP bağlantı taraması olarak da bilinir), TCP tree handshake tamamladığı için oldukça doğru sonuçlar verir ve bir portun durumunu (açık, kapalı veya filtrelenmiş) kesin olarak belirlememizi sağlar. Ancak, bu yöntem en gizli tarama tekniklerinden biri değildir. Hatta, Connect Scan en az gizli tarama yöntemlerinden biri olarak kabul edilir çünkü tam bir bağlantı kurar ve bu da çoğu sistemde log kayıtları oluşturur.

Ayrıca, hedef sistemde gelen paketleri engelleyen ancak giden paketlere izin veren kişisel bir güvenlik duvarı (firewall) varsa, Connect Scan bu firewall'ı atlayabilir ve hedef portların durumunu doğru bir şekilde belirleyebilir. Ancak, Connect Scan diğer tarama türlerine göre daha yavaştır çünkü tarayıcının, gönderdiği her paketten sonra hedeften bir yanıt beklemesi gerekir. Bu, hedef meşgul veya yanıt vermiyorsa biraz zaman alabilir.

SYN taraması (yarı açık tarama olarak da bilinir) gibi taramalar genellikle daha gizli kabul edilir çünkü full handshake tamamlamaz ve başlangıçta gönderilen SYN paketinden sonra bağlantıyı tamamlamaz.

#### Connect Scan on TCP Port 443

```shell-session
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```


#### Filtered Ports

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

|Tarama Seçenekleri|Açıklama|
|---|---|
|**10.129.2.28**|Belirtilen hedefi tarar.|
|**-p 139**|Sadece belirtilen portu tarar.|
|**--packet-trace**|Gönderilen ve alınan tüm paketleri gösterir.|
|**-n**|DNS çözümlemesini devre dışı bırakır.|
|**--disable-arp-ping**|ARP ping işlemini devre dışı bırakır.|
|**-Pn**|ICMP Echo isteklerini devre dışı bırakır.|

Son taramada Nmap'in SYN bayrağı ile iki TCP paketi gönderdiğini görüyoruz. Taramanın süresinden (2.06s), öncekilerden (~0.05s) çok daha uzun sürdüğünü anlayabiliriz. Firewall paketleri reddediyorsa durum farklıdır. Bunun için, firewall'un böyle bir kuralı tarafından uygun şekilde işlenen TCP port 445'e bakıyoruz.


```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

Bir yanıt olarak, **type 3** ve **code 3** olan bir ICMP yanıtı alıyoruz. Bu, istenen portun erişilemez olduğunu belirtir. Ancak, eğer host'un aktif olduğunu biliyorsak, bu durumda ilgili port üzerinde bir **firewall**'ın paketleri reddettiğini güçlü bir şekilde varsayabiliriz ve bu portu daha detaylı incelememiz gerekecektir.


### Discovering Open UDP Ports

Bazı sistem yöneticileri, TCP portlarını filtrelemeye ek olarak UDP portlarını filtrelemeyi bazen unutabilir. UDP, **stateless** (durumsuz) bir protokol olduğu için TCP gibi **three-way handshake** gerektirmez. Bu nedenle, herhangi bir **acknowledgment** (onay) almayız. Sonuç olarak, zaman aşımı süresi çok daha uzundur ve bu durum tüm UDP taramasını (**-sU**) TCP taramasına (**-sS**) kıyasla çok daha yavaş hale getirir.

Şimdi bir UDP taramasının (**-sU**) nasıl göründüğüne ve bize hangi sonuçları verdiğine dair bir örneğe bakalım.

#### UDP Port Scan

```shell-session
sudo nmap 10.129.2.28 -F -sU
```

|      |                      |
| ---- | -------------------- |
| `-F` | Scans top 100 ports. |

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:15 CEST
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.0031s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```


|            |                                                               |
| ---------- | ------------------------------------------------------------- |
| `--reason` | Bir portun belirli bir durumda olmasının nedenini görüntüler. |
Error code 3 (port unreachable) ile bir ICMP response alırsak, portun gerçekten kapalı olduğunu biliriz.

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 
```

Diğer tüm ICMP yanıtları için, taranan portlar ((open|filtered)) olarak işaretlenir.

**`open|filtered`**, Nmap çıktısında, bir portun açık (**open**) veya bir firewall ya da filtre tarafından engellendiği (**filtered**) durumunu ayırt edemediği anlamına gelir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:32 CEST
SENT (0.0380s) UDP 10.10.14.2:52341 > 10.129.2.28:138 ttl=50 id=65159 iplen=28
SENT (1.0392s) UDP 10.10.14.2:52342 > 10.129.2.28:138 ttl=40 id=24444 iplen=28
Nmap scan report for 10.129.2.28
Host is up, received user-set.

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

#### Version Scan

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-04 11:10 GMT
SENT (0.3426s) TCP 10.10.14.2:44641 > 10.129.2.28:445 S ttl=55 id=43401 iplen=44  seq=3589068008 win=1024 <mss 1460>
RCVD (0.3556s) TCP 10.129.2.28:445 > 10.10.14.2:44641 SA ttl=63 id=0 iplen=44  seq=2881527699 win=29200 <mss 1337>
NSOCK INFO [0.4980s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.4980s] nsock_connect_tcp(): TCP connection requested to 10.129.2.28:445 (IOD #1) EID 8
NSOCK INFO [0.5130s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.2.28:445]
Service scan sending probe NULL to 10.129.2.28:445 (tcp)
NSOCK INFO [0.5130s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.2.28:445]
Service scan sending probe SMBProgNeg to 10.129.2.28:445 (tcp)
NSOCK INFO [6.5190s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [10.129.2.28:445]
NSOCK INFO [6.5190s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.2.28:445]
NSOCK INFO [6.5320s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.2.28:445] (135 bytes)
Service scan match (Probe SMBProgNeg matched with SMBProgNeg line 13836): 10.129.2.28:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.5320s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds
```

Port tarama teknikleri hakkında daha fazla bilgiyi şu adreste bulabilirsiniz: https://nmap.org/book/man-port-scanning-techniques.html


Soru : Hedefinizdeki tüm TCP portlarını bulun. Bulunan TCP portlarının toplam sayısını cevap olarak gönderin.

Cevap : `7`

![Pasted image 20250131012651.png](/img/user/resimler/Pasted%20image%2020250131012651.png)

Soru : Hedefinizin host adını numaralandırın ve cevap olarak gönderin. (büyük/küçük harfe duyarlı)

Cevap : `nix-nmap-default`

![Pasted image 20250131012721.png](/img/user/resimler/Pasted%20image%2020250131012721.png)

![Pasted image 20250131012711.png](/img/user/resimler/Pasted%20image%2020250131012711.png)


# Saving the Results

## Different Formats

- **Normal çıktı** (`-oN`) **.nmap** uzantılı dosya olarak
- **Grep uyumlu çıktı** (`-oG`) **.gnmap** uzantılı dosya olarak
- **XML formatında çıktı** (`-oX`) **.xml** uzantılı dosya olarak

Ayrıca, `-oA` seçeneğini belirterek sonuçları tüm formatlarda kaydedebiliriz.


```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -oA target

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 12:14 CEST
Nmap scan report for 10.129.2.28
Host is up (0.0091s latency).
Not shown: 65525 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
```

`-oA target` -> Sonuçları tüm formatlarda kaydeder ve her dosyanın adını 'target' ile başlatır.

Tam yol belirtilmezse, sonuçlar o anda bulunduğumuz dizinde saklanacaktır. Daha sonra, Nmap'in bizim için oluşturduğu farklı formatlara bakacağız.

```shell-session
C4RT3L@htb[/htb]$ ls

target.gnmap target.xml  target.nmap
```


#### Normal Output

```shell-session
C4RT3L@htb[/htb]$ cat target.nmap

# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 10.129.2.28
Nmap scan report for 10.129.2.28
Host is up (0.053s latency).
Not shown: 4 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

# Nmap done at Tue Jun 16 12:15:03 2020 -- 1 IP address (1 host up) scanned in 10.22 seconds
```

#### Grepable Output

```shell-session
C4RT3L@htb[/htb]$ cat target.gnmap

# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 10.129.2.28
Host: 10.129.2.28 ()	Status: Up
Host: 10.129.2.28 ()	Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http///	Ignored State: closed (4)
# Nmap done at Tue Jun 16 12:14:53 2020 -- 1 IP address (1 host up) scanned in 10.22 seconds
```

#### XML Output

```shell-session
C4RT3L@htb[/htb]$ cat target.xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/local/bin/../share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 10.129.2.28 -->
<nmaprun scanner="nmap" args="nmap -p- -oA target 10.129.2.28" start="12145301719" startstr="Tue Jun 16 12:15:03 2020" version="7.80" xmloutputversion="1.04">
<scaninfo type="syn" protocol="tcp" numservices="65535" services="1-65535"/>
<verbose level="0"/>
<debugging level="0"/>
<host starttime="12145301719" endtime="12150323493"><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="10.129.2.28" addrtype="ipv4"/>
<address addr="DE:AD:00:00:BE:EF" addrtype="mac" vendor="Intel Corporate"/>
<hostnames>
</hostnames>
<ports><extraports state="closed" count="4">
<extrareasons reason="resets" count="4"/>
</extraports>
<port protocol="tcp" portid="22"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="ssh" method="table" conf="3"/></port>
<port protocol="tcp" portid="25"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="smtp" method="table" conf="3"/></port>
<port protocol="tcp" portid="80"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="http" method="table" conf="3"/></port>
</ports>
<times srtt="52614" rttvar="75640" to="355174"/>
</host>
<runstats><finished time="12150323493" timestr="Tue Jun 16 12:14:53 2020" elapsed="10.22" summary="Nmap done at Tue Jun 16 12:15:03 2020; 1 IP address (1 host up) scanned in 10.22 seconds" exit="success"/><hosts up="1" down="0" total="1"/>
</runstats>
</nmaprun>
```

## Style sheets

XML çıktısı ile, teknik olmayan kişiler için bile okunması kolay HTML raporlarını kolayca oluşturabiliriz. Bu daha sonra dokümantasyon için çok kullanışlıdır, çünkü sonuçlarımızı ayrıntılı ve net bir şekilde sunar. Depolanan sonuçları XML formatından HTML'e dönüştürmek için `xsltproc` aracını kullanabiliriz.

```shell-session
4RT3L@htb[/htb]$ xsltproc target.xml -o target.html
```


#### Nmap Report

![Pasted image 20250219154057.png](/img/user/resimler/Pasted%20image%2020250219154057.png)

Output formatları hakkında daha fazla bilgiyi [şu](https://nmap.org/book/output.html) adreste bulabilirsiniz: 


Soru : Hedefinizde tam bir TCP port taraması gerçekleştirin ve bir HTML raporu oluşturun. Cevap olarak en yüksek portun numarasını gönderin.

Cevap : `31337`

![Pasted image 20250219154428.png](/img/user/resimler/Pasted%20image%2020250219154428.png)


![Pasted image 20250219154509.png](/img/user/resimler/Pasted%20image%2020250219154509.png)

![Pasted image 20250219154521.png](/img/user/resimler/Pasted%20image%2020250219154521.png)



# Service Enumeration

Doğru sürüm tespiti, zafiyet taraması ve kaynak kod analizi için kritiktir. Kesin sürüm numarası, hedefe uygun **`exploit`** bulmayı kolaylaştırır.

## Service Version Detection

Hızlı bir port taraması, düşük trafikle tespit riskini azaltarak genel bir görünüm sağlar. Ardından arka planda tüm portları tarayabiliriz (`-p-`). Belirli servis ve sürümleri belirlemek için **version scan** kullanabiliriz (`-sV`). Tarama durumunu görmek için `[Space Bar]` tuşuna basabiliriz.

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:44 CEST
[Space Bar]
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

Başka bir seçenek olarak (`--stats-every=5s`), tarama durumunun hangi aralıklarla gösterileceğini belirleyebiliriz. Durumun kaç saniye (s) veya dakika (m) sonra görüntüleneceğini burada belirtebiliriz.

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV --stats-every=5s

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:46 CEST
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 13.91% done; ETC: 19:49 (0:00:31 remaining)
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 39.57% done; ETC: 19:48 (0:00:15 remaining)
```

Ayrıca verbosity seviyesini (`-v / -vv`) artırabiliriz, bu da Nmap tespit ettiğinde bize açık portları doğrudan gösterecektir.

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -v 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:03 CEST
NSE: Loaded 45 scripts for scanning.
Initiating ARP Ping Scan at 20:03
Scanning 10.129.2.28 [1 port]
Completed ARP Ping Scan at 20:03, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:03
Completed Parallel DNS resolution of 1 host. at 20:03, 0.02s elapsed
Initiating SYN Stealth Scan at 20:03
Scanning 10.129.2.28 [65535 ports]
Discovered open port 995/tcp on 10.129.2.28
Discovered open port 80/tcp on 10.129.2.28
Discovered open port 993/tcp on 10.129.2.28
Discovered open port 143/tcp on 10.129.2.28
Discovered open port 25/tcp on 10.129.2.28
Discovered open port 110/tcp on 10.129.2.28
Discovered open port 22/tcp on 10.129.2.28
```



## Banner Grabbing

Tarama tamamlandığında, sistemde etkin olan tüm TCP portlarını ilgili servis ve sürümleriyle birlikte göreceğiz.

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:00 CEST
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.73 seconds
```

Nmap, **banner** bilgilerini kontrol eder; sürüm belirlenemezse, imza eşleştirme kullanır, bu da tarama süresini uzatır. Otomatik tarama bazen bilgileri atlayabilir.

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
<SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
Service scan match (Probe NULL matched with NULL line 3104): 10.129.2.28:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```

Nmap sonuçlarında port durumu, servis adı ve hostname gibi bilgiler görünür, ancak şu satırda görülenler daha fazla bilgi sağlar:

`NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..`

Burada, SMTP sunucusu daha fazla bilgi veriyor çünkü `Ubuntu` dağıtımını gösteriyor. Bu, tree handshake sıkışmadan sonra sunucunun genellikle kimlik tespiti için bir banner gönderdiği içindir. Ancak bazı servisler hemen bilgi sağlamayabilir veya bannerları değiştirebilir. Eğer `nc` ile sunucuya bağlanıp `tcpdump` ile trafiği yakalarsak, Nmap’in gösterdiği bilgiden farklı şeyler görebiliriz.


#### Tcpdump

```shell-session
C4RT3L@htb[/htb]$ sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```


#### Nc

```shell-session
C4RT3L@htb[/htb]$  nc -nv 10.129.2.28 25

Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```


#### Tcpdump - Yakalanan Traffic

```shell-session
18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], seq 1798872233, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 331260178 ecr 0,sackOK,eol], length 0

18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], seq 1130574379, ack 1798872234, win 65160, options [mss 1460,sackOK,TS val 1800383922 ecr 331260178,nop,wscale 7], length 0

18:28:07.255281 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], ack 1, win 2058, options [nop,nop,TS val 331260304 ecr 1800383922], length 0
18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], seq 1:36, ack 1, win 510, options [nop,nop,TS val 1800383985 ecr 331260304], length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)

18:28:07.319426 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], ack 36, win 2058, options [nop,nop,TS val 331260368 ecr 1800383985], length 0
```

İlk üç satır bize three-way handshake gösterir.

```
1. [SYN] 18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], <SNIP>

2. [SYN-ACK] 18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], <SNIP>

3. [ACK] 18:28:07.255281 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>
```

Bundan sonra, hedef SMTP sunucusu bize PSH ve ACK bayraklarıyla bir TCP paketi gönderir; burada PSH, hedef sunucunun bize veri gönderdiğini belirtir ve ACK ile aynı anda gerekli tüm verilerin gönderildiğini bildirir.

```
4.	[PSH-ACK]	18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], <SNIP>
```

Gönderdiğimiz son TCP paketi, bir ACK ile verilerin alındığını onaylar.

```
5.	[ACK]	18:28:07.319426 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>
```


Soru : Tüm portları ve servislerini listeleyin. Servislerden biri, cevap olarak göndermeniz gereken bayrağı içerir.

Cevap : `HTB{pr0F7pDv3r510nb4nn3r}`

```
nmap 10.129.120.185 -sCV --min-rate 10000 -p- 

Service scan Timing: About 85.71% done; ETC: 07:37 (0:00:06 remaining)
Nmap scan report for 10.129.120.185
Host is up (0.0097s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
80/tcp    open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
110/tcp   open  pop3        Dovecot pop3d
|_pop3-capabilities: CAPA SASL AUTH-RESP-CODE UIDL PIPELINING RESP-CODES TOP
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
|_imap-capabilities: LOGIN-REFERRALS LITERAL+ more ID SASL-IR IDLE OK ENABLE IMAP4rev1 post-login listed LOGINDISABLEDA0001 capabilities Pre-login have
445/tcp   open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
31337/tcp open  ftp         ProFTPD
Service Info: Host: NIX-NMAP-DEFAULT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: NIX-NMAP-DEFAUL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -20m00s, deviation: 34m38s, median: 0s
| smb2-time: 
|   date: 2025-02-19T13:37:33
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: nix-nmap-default
|   NetBIOS computer name: NIX-NMAP-DEFAULT\x00
|   Domain name: \x00
|   FQDN: nix-nmap-default
|_  System time: 2025-02-19T14:37:33+01:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.19 seconds

```

![Pasted image 20250219163955.png](/img/user/resimler/Pasted%20image%2020250219163955.png)

# Nmap Scripting Engine

Nmap Scripting Engine (NSE), Nmap'in kullanışlı bir diğer özelliğidir. Belirli servislerle etkileşim kurmak için Lua ile scriptler oluşturma imkanı sağlar. Bu scriptler toplamda 14 kategoriye ayrılabilir:

|Kategori|Açıklama|
|---|---|
|**auth**|Kimlik doğrulama bilgilerini belirleme.|
|**broadcast**|Yayın yaparak host keşfi için kullanılan scriptlerdir; bulunan hostlar otomatik olarak taramalara eklenebilir.|
|**brute**|Belirtilen servise brute-force yöntemiyle giriş yapmayı deneyen scriptleri çalıştırır.|
|**default**|`-sC` seçeneğiyle çalıştırılan varsayılan scriptlerdir.|
|**discovery**|Erişilebilir servisleri değerlendirme.|
|**dos**|Servisleri **hizmet reddi (Denial of Service - DoS)** açıklarına karşı test eden scriptlerdir; hizmetlere zarar verebileceğinden daha az kullanılır.|
|**exploit**|Taranan porta yönelik bilinen açıklardan yararlanmaya çalışan scriptlerdir.|
|**external**|Daha fazla işlem yapmak için harici servisleri kullanan scriptlerdir.|
|**fuzzer**|Servislerin beklenmeyen paketleri nasıl ele aldığını test ederek açıklıkları belirlemeye çalışan scriptlerdir; uzun sürebilir.|
|**intrusive**|Hedef sisteme olumsuz etkisi olabilecek saldırgan scriptlerdir.|
|**malware**|Hedef sistemin kötü amaçlı yazılımlarla enfekte olup olmadığını kontrol eder.|
|**safe**|Saldırgan veya zarar verici erişimler yapmayan savunmacı scriptlerdir.|
|**version**|Servis tespiti için kullanılan scriptlerin genişletilmiş versiyonudur.|
|**vuln**|Belirli güvenlik açıklarını tespit etmeye yönelik scriptlerdir.|

#### Default Scripts

```shell-session
C4RT3L@htb[/htb]$ sudo nmap <target> -sC
```


#### Specific Scripts Category

```shell-session
C4RT3L@htb[/htb]$ sudo nmap <target> --script <category>
```


#### Defined Scripts

```shell-session
C4RT3L@htb[/htb]$ sudo nmap <target> --script <script-name>,<script-name>,...
```

Örneğin, hedef SMTP portu ile çalışmaya devam edelim ve tanımlanmış iki komut dosyası ile elde ettiğimiz sonuçları görelim.

#### Nmap - Specifying Scripts

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 23:21 CEST
Nmap scan report for 10.129.2.28
Host is up (0.050s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```

Ubuntu Linux dağıtımını **banner** scripti kullanarak tanıyabildiğimizi görüyoruz. **smtp-commands** scripti ise hedef SMTP sunucusuyla etkileşim kurarak hangi komutları kullanabileceğimizi gösterir. Bu tür bilgiler, hedefte mevcut kullanıcıları belirlememize yardımcı olabilir.

Ayrıca, Nmap hedefimizi **aggressive** modu (**`-A`**) ile tarama imkanı sunar. Bu seçenek, birden fazla özelliği aynı anda çalıştırır:

- **Servis tespiti** (**-sV**)
- **İşletim sistemi tespiti** (**-O**)
- **Traceroute** (**--traceroute**)
- **Varsayılan NSE scriptleri** (**-sC**)


#### Nmap - Aggressive Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -A
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-17 01:38 CEST
Nmap scan report for 10.129.2.28
Host is up (0.012s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), 
AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), 
Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

TRACEROUTE
HOP RTT      ADDRESS
1   11.91 ms 10.129.2.28

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.36 seconds
```

`-A`
Servis tespiti, işletim sistemi tespiti, traceroute gerçekleştirir ve hedefi taramak için varsayılan komut dosyalarını kullanır.

Kullanılan tarama seçeneği (-A) ile sistemde çalışan **web server**'ın (Apache 2.4.29), kullanılan **web application**'ın (`WordPress 5.3.4`) ve web sayfasının başlığının (`blog.inlanefreight.com`) ne olduğunu belirledik. Ayrıca, Nmap `%96` ihtimalle işletim sisteminin **`Linux`** olduğunu gösteriyor.


## Vulnerability Assessment

Şimdi HTTP port 80'e geçelim ve NSE'nin vuln kategorisini kullanarak hangi bilgileri ve güvenlik açıklarını bulabileceğimizi görelim.

#### Nmap - Vuln Category

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sV --script vuln 

Nmap scan report for 10.129.2.28
Host is up (0.036s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-wordpress-users:
| Username found: admin
|_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|     	CVE-2019-0211	7.2	https://vulners.com/cve/CVE-2019-0211
|     	CVE-2018-1312	6.8	https://vulners.com/cve/CVE-2018-1312
|     	CVE-2017-15715	6.8	https://vulners.com/cve/CVE-2017-15715
<SNIP>
```


Soru : Servislerden birinin içerdiği bayrağı bulmak için NSE ve komut dosyalarını kullanın ve cevap olarak gönderin.

Cevap : `HTB{pr0F7pDv3r510nb4nn3r}`

```
nmap 10.129.4.84 -p- -sCV --min-rate 10000
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-19 09:29 CST

Host is up (0.0091s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
80/tcp    open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
110/tcp   open  pop3        Dovecot pop3d
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
445/tcp   open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
31337/tcp open  Elite?
| fingerprint-strings: 
|   GetRequest, NULL: 
|_    220 HTB{pr0F7pDv3r510nb4nn3r}
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.94SVN%I=7%D=2/19%Time=67B5F8DE%P=x86_64-pc-linux-gnu%
SF:r(NULL,1F,"220\x20HTB{pr0F7pDv3r510nb4nn3r}\r\n")%r(GetRequest,1F,"220\
SF:x20HTB{pr0F7pDv3r510nb4nn3r}\r\n");
Service Info: Host: NIX-NMAP-DEFAULT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: NIX-NMAP-DEFAUL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-02-19T15:32:05
|_  start_date: N/A
|_clock-skew: mean: -19m59s, deviation: 34m37s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: nix-nmap-default
|   NetBIOS computer name: NIX-NMAP-DEFAULT\x00
|   Domain name: \x00
|   FQDN: nix-nmap-default
|_  System time: 2025-02-19T16:32:05+01:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 168.63 seconds

```

```
nmap --script http-robots.txt -p 80,443 hedef-site.com
```


![Pasted image 20250219184713.png](/img/user/resimler/Pasted%20image%2020250219184713.png)


# Performance

Tarama performansı, geniş ağlarda veya düşük bant genişliğinde kritiktir. Nmap'e hız (`-T <0-5>`), paralellik (`--min-parallelism <sayı>`), zaman aşımı (`--max-rtt-timeout <süre>`), paket gönderim hızı (`--min-rate <sayı>`) ve yeniden deneme sayısı (`--max-retries <sayı>`) gibi ayarları belirtebiliriz.

## Timeouts

Nmap bir paket gönderdiğinde, taranan porttan yanıt almak belirli bir süre alır (`Round-Trip-Time - RTT`). Genellikle Nmap, varsayılan olarak 100ms'lik yüksek bir zaman aşımı süresiyle (`--min-RTT-timeout`) başlar. Şimdi, 256 host içeren bir ağı tarayarak en üst 100 portu inceleyen bir örneğe bakalım.


#### Default Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

#### Optimized RTT

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms

<SNIP>
Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

| Tarama Seçenekleri           | Açıklama                                               |
| ---------------------------- | ------------------------------------------------------ |
| `10.129.2.0/24`              | Belirtilen hedef ağı tarar.                            |
| `-F`                         | En üst 100 portu tarar.                                |
| `--initial-rtt-timeout 50ms` | Başlangıç RTT zaman aşımını belirtilen değere ayarlar. |
| `--max-rtt-timeout 100ms`    | Maksimum RTT zaman aşımını belirtilen değere ayarlar.  |

İki taramayı karşılaştırdığımızda, optimize edilmiş taramada iki host daha az bulduğumuzu ancak taramanın dörtte bir sürede tamamlandığını görüyoruz. Bu da bize, başlangıç RTT zaman aşımını (`--initial-rtt-timeout`) çok kısa bir süreye ayarlamanın bazı hostları gözden kaçırmamıza neden olabileceğini gösteriyor.


## Max Retries

Tarama hızını artırmanın bir başka yolu, gönderilen paketlerin yeniden deneme oranını belirlemektir (`--max-retries`). Varsayılan değer 10'dur, ancak bunu 0'a düşürebiliriz. Bu, Nmap'in bir porttan yanıt almazsa o porta tekrar paket göndermeden geçmesini sağlar.

#### Default Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l

23
```

#### Reduced Retries

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l

21
```

|Tarama Seçenekleri|Açıklama|
|---|---|
|`10.129.2.0/24`|Belirtilen hedef ağı tarar.|
|`-F`|En üst 100 portu tarar.|
|`--max-retries 0`|Tarama sırasında gerçekleştirilecek yeniden deneme sayısını ayarlar.|
Yine, hızlandırmanın da sonuçlarımız üzerinde olumsuz bir etkisi olabileceğinin farkındayız, bu da önemli bilgileri gözden kaçırabileceğimiz anlamına geliyor.


## Rates

Bir **white-box penetration test** sırasında, yalnızca güvenlik önlemlerini test etmek değil, ağdaki sistemleri zafiyetler açısından incelemek için güvenlik sistemlerinde **whitelist** edilebiliriz. Ağ bant genişliğini biliyorsak, gönderilen paket hızını ayarlayarak Nmap taramalarını önemli ölçüde hızlandırabiliriz. **Minimum paket gönderim oranını** (`--min-rate <sayı>`) belirlediğimizde, Nmap'e belirtilen sayıda paketi **eşzamanlı olarak göndermesini** söyleriz ve bu hızı korumaya çalışır.

#### Default Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```


#### Optimized Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```

|Tarama Seçenekleri|Açıklama|
|---|---|
|`10.129.2.0/24`|Belirtilen hedef ağı tarar.|
|`-F`|En üst 100 portu tarar.|
|`-oN tnet.minrate300`|Sonuçları belirtilen dosya adına sahip normal formatta kaydeder.|
|`--min-rate 300`|Saniyede gönderilecek minimum paket sayısını ayarlar.|

#### Default Scan - Found Open Ports

  Performance

```shell-session
C4RT3L@htb[/htb]$ cat tnet.default | grep "/tcp" | wc -l

23
```

#### Optimized Scan - Found Open Ports

```shell-session
C4RT3L@htb[/htb]$ cat tnet.minrate300 | grep "/tcp" | wc -l

23
```

## Timing

Bu tür ayarlar her zaman manuel olarak optimize edilemez, örneğin bir **black-box penetration test**'te olduğu gibi, bu yüzden Nmap bize kullanmamız için altı farklı **zamanlama şablonu** (`-T <0-5>`) sunar. Bu değerler (0-5) taramalarımızın **agresifliğini** belirler. Eğer tarama çok agresifse, güvenlik sistemleri ağ trafiği nedeniyle bizi engelleyebilir. Hiçbir şey tanımlamadığımızda kullanılan varsayılan zamanlama şablonu **normal (-T 3)**'tür.

- `-T 0` / `-T paranoid`
- `-T 1` / `-T sneaky`
- `-T 2` / `-T polite`
- `-T 3` / `-T normal`
- `-T 4` / `-T aggressive`
- `-T 5` / `-T insane`


#### Default Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default 

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 32.44 seconds
```

#### Insane Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 18.07 seconds
```

#### Default Scan - Found Open Ports

```shell-session
C4RT3L@htb[/htb]$ cat tnet.default | grep "/tcp" | wc -l

23
```


#### Insane Scan - Found Open Ports

```shell-session
C4RT3L@htb[/htb]$ cat tnet.T5 | grep "/tcp" | wc -l

23
```



# Firewall and IDS/IPS Evasion

Nmap bize güvenlik duvarları kurallarını ve IDS/IPS'yi atlatmak için birçok farklı yol sunar. Bu yöntemler arasında `paketlerin parçalanması`, `decoy` kullanımı ve bu bölümde tartışacağımız diğer yöntemler yer almaktadır.


## Firewalls

Bir **firewall**, dış ağlardan gelen yetkisiz bağlantıları önleyen bir güvenlik önlemidir. Ağ trafiğini izleyerek belirlenen kurallara göre **paketleri iletir, göz ardı eder veya engeller**. Bu mekanizma, **potansiyel tehditleri engellemek** için tasarlanmıştır.

## IDS/IPS

**Firewall** gibi, **Intrusion Detection System (IDS)** ve **Intrusion Prevention System (IPS)** de yazılım tabanlı bileşenlerdir. **IDS**, ağı olası saldırılara karşı tarar, analiz eder ve tespit edilen saldırıları raporlar. **IPS** ise **IDS'yi tamamlayarak** potansiyel bir saldırı algılandığında savunma önlemleri alır. Bu analiz, **pattern matching** ve **signature** tabanlıdır. Örneğin, bir **service detection scan** tespit edilirse, **IPS** bağlantı girişimlerini engelleyebilir.


#### Firewalls ve Kurallarını Belirleme

**Filtered** olarak görünen bir portun farklı nedenleri olabilir. Çoğu durumda, **firewall** belirli bağlantıları yönetmek için kurallar uygular. **Paketler ya düşürülür (dropped) ya da reddedilir (rejected).**

- **Dropped paketler** tamamen yok sayılır ve **hedeften herhangi bir yanıt gelmez.**
- **Rejected paketler** ise **RST** bayrağı ile geri döner ve bazen **ICMP hata kodları** içerebilir:
    - **Net Unreachable**
    - **Net Prohibited**
    - **Host Unreachable**
    - **Host Prohibited**
    - **Port Unreachable**
    - **Proto Unreachable**

**Nmap'in TCP ACK taraması (-sA)**, **SYN (-sS) veya Connect (-sT) taramalarına kıyasla firewall ve IDS/IPS sistemleri tarafından filtrelenmesi daha zordur.** Çünkü sadece **ACK bayrağı taşıyan** bir TCP paketi gönderir.

- **Port açık veya kapalıysa**, hedef **RST bayrağı** ile yanıt vermek zorundadır.
- **Firewall'lar genellikle dış ağlardan gelen SYN bayraklı paketleri engeller**, ancak ACK bayraklı paketler genellikle geçer. Çünkü **firewall, bağlantının iç ağdan mı yoksa dış ağdan mı başlatıldığını belirleyemez.**

Bu taramaları incelediğimizde, **sonuçların nasıl değiştiğini görebiliriz.**


#### SYN-Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:56 CEST
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:22 S ttl=53 id=22412 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:25 S ttl=50 id=62291 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:21 S ttl=58 id=38696 iplen=44  seq=4092255222 win=1024 <mss 1460>
RCVD (0.0329s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=40884 iplen=72 ]
RCVD (0.0341s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
RCVD (1.0386s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
SENT (1.1366s) TCP 10.10.14.2:57348 > 10.129.2.28:25 S ttl=44 id=6796 iplen=44  seq=4092320759 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.0053s latency).

PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

#### ACK-Scan

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:57 CEST
SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A ttl=49 id=12381 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A ttl=41 id=5146 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A ttl=49 id=5800 iplen=40  seq=0 win=1024
RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=55628 iplen=68 ]
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R ttl=64 id=0 iplen=40  seq=1660784500 win=0
SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A ttl=59 id=21915 iplen=40  seq=0 win=1024
Nmap scan report for 10.129.2.28
Host is up (0.083s latency).

PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

Hedefimizden aldığımız **RCVD paketlerine** ve **bayraklarına** dikkat edin. **SYN taraması (-sS)** ile hedefimiz, TCP bağlantısını kurmaya çalışırken **SYN-ACK (SA)** bayrakları ile geri bir paket gönderir. **ACK taraması (-sA)** ile ise **`RST`** bayrağını alırız çünkü TCP portu 22 açıktır. Ancak, TCP portu 25 için geri paket almazsak, bu **paketlerin düşürüldüğünü** gösterir.


## Detect IDS/IPS

**Firewall'lar ve kuralları** ile karşılaştırıldığında, **IDS/IPS sistemlerini tespit etmek** çok daha zordur çünkü bunlar **pasif trafik izleme sistemleri**dir. **IDS sistemleri**, **hostlar arasındaki tüm bağlantıları** inceler. Eğer **IDS**, tanımlanan içerikleri veya spesifikasyonları içeren paketler bulursa, **yöneticiye bildirim gönderilir** ve en kötü durumda uygun önlemler alınır.

**IPS sistemleri**, **admin tarafından yapılandırılmış önlemleri** bağımsız olarak alır ve potansiyel saldırıları **otomatik olarak engellemeye çalışır**. IDS ve IPS'in farklı uygulamalar olduğunu ve **IPS'in IDS'in tamamlayıcısı** olarak servis verdiğini bilmek önemlidir.

Penetrasyon testi sırasında, hedef ağda böyle bir sistemin olup olmadığını belirlemek için **farklı IP adreslerine sahip birkaç sanal özel sunucu (VPS)** kullanılması önerilir. Eğer yönetici, hedef ağda böyle bir potansiyel saldırıyı tespit ederse, ilk adım olarak **saldırının geldiği IP adresini engellemek** olacaktır. Sonuç olarak, bu IP adresiyle ağa erişimimiz engellenir ve **İnternet Servis Sağlayıcımız (ISP)** ile iletişime geçilir, böylece **tüm internet erişimi** engellenir.

**Sadece IDS sistemleri**, genellikle yöneticilerin ağlarındaki potansiyel saldırıları tespit etmelerine yardımcı olmak için bulunur. Ardından, yöneticiler bu bağlantıları nasıl ele alacaklarına karar verirler. Örneğin, bir **portu agresif şekilde tarayarak ve servisi inceleyerek**, **yöneticinin bazı güvenlik önlemlerini tetiklemesini sağlayabiliriz**. Eğer belirli güvenlik önlemleri alındıysa, ağda **izleme uygulamaları** olup olmadığını tespit edebiliriz.

Böyle bir **IPS sisteminin hedef ağda olup olmadığını belirlemenin bir yolu**, **tek bir host (VPS) ile tarama yapmaktır**. Eğer bu host herhangi bir zamanda engellenir ve hedef ağa erişemez hale gelirse, **yöneticinin bazı güvenlik önlemleri aldığı** anlaşılır. Buna göre, **penetration testine başka bir VPS ile devam edebiliriz**.

Sonuç olarak, **taramalarımızı daha sessiz yapmamız gerektiğini** ve en iyi durumda **hedef ağ ve servisiyle tüm etkileşimleri gizlememiz gerektiğini** biliyoruz.


## Decoys

Yöneticilerin, farklı bölgelerden gelen belirli **subnetleri** prensipte engellediği durumlar vardır. Bu, hedef ağa erişimi tamamen engeller. Diğer bir örnek, IPS'in bizi engellemesi gerektiği durumlardır. Bu nedenle, **`Decoy tarama yöntemi (-D)`** doğru seçimdir. Bu yöntemle, Nmap, gönderilen paketin kaynağını gizlemek için IP başlığına rastgele çeşitli **IP adresleri** ekler. Bu yöntemle, belirli bir sayıda (örneğin: 5) rastgele (RND) IP adresi oluşturabiliriz ve bunlar arasında **iki nokta (:)** ile ayrılır. Gerçek IP adresimiz, ardından oluşturulan IP adresleri arasına rastgele yerleştirilir. Bir sonraki örnekte, gerçek IP adresimiz ikinci konumda yer alacaktır. Bir diğer kritik nokta ise **`decoy'ların hayatta olması gerektiğidir`**. Aksi takdirde, hedefteki hizmet **`SYN-flooding`** güvenlik mekanizmaları nedeniyle erişilemez hale gelebilir.

#### Scan by Using Decoys

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 16:14 CEST
SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S ttl=42 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S ttl=59 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.129.2.28:80 S ttl=37 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.129.2.28:80 S ttl=38 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.129.2.28:80 S ttl=39 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.129.2.28:80 S ttl=55 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA ttl=64 id=0 iplen=44  seq=4056111701 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.099s latency).

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```


| Tarama Seçenekleri | Açıklama                                                                 |
| ------------------ | ------------------------------------------------------------------------ |
| 10.129.2.28        | Belirtilen hedefi tarar.                                                 |
| -p 80              | Yalnızca belirtilen portları tarar.                                      |
| -sS                | Belirtilen portlarda SYN taraması yapar.                                 |
| -Pn                | ICMP Echo isteklerini devre dışı bırakır.                                |
| -n                 | DNS çözümlemeyi devre dışı bırakır.                                      |
| --disable-arp-ping | ARP ping'ini devre dışı bırakır.                                         |
| --packet-trace     | Gönderilen ve alınan tüm paketleri gösterir.                             |
| -D RND:5           | Bağlantının geldiği kaynak IP'yi gösteren beş rastgele IP adresi üretir. |

Spoofed paketler, aynı ağ aralığından geliyor olsalar bile genellikle ISP'ler ve yönlendiriciler tarafından filtrelenir. Bu nedenle, VPS sunucularımızın IP adreslerini belirtebiliriz ve hedefi taramak için IP başlıklarındaki "`IP ID`" manipülasyonu ile bunları kombin edebiliriz.

Başka bir senaryo ise yalnızca belirli **subnetlerin** sunucunun belirli hizmetlerine erişiminin olmamış olmasıdır. Bu durumda, daha iyi sonuçlar alıp almadığımızı test etmek için kaynak IP adresini (`-S`) manuel olarak belirtebiliriz. **Decoy'lar**, SYN, ACK, ICMP taramaları ve OS tespiti taramaları için kullanılabilir. Şimdi böyle bir örneğe bakalım ve hangi işletim sistemine ait olduğunu en olası şekilde belirleyelim.

#### Testing Firewall Rule

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds
```

#### Farklı Kaynak IP Kullanarak Tarama

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds
```

| **Tarama Seçenekleri** | **Açıklama**                                         |
| ---------------------- | ---------------------------------------------------- |
| **10.129.2.28**        | Belirtilen hedefi tarar.                             |
| **-n**                 | DNS çözümlemeyi devre dışı bırakır.                  |
| **-Pn**                | ICMP Echo isteklerini devre dışı bırakır.            |
| **-p 445**             | Yalnızca belirtilen portları tarar.                  |
| **-O**                 | İşletim sistemi tespiti taraması gerçekleştirir.     |
| **-S**                 | Farklı bir kaynak IP adresi kullanarak hedefi tarar. |
| **10.129.2.200**       | Kaynak IP adresini belirtir.                         |
| **-e tun0**            | Tüm istekleri belirtilen arayüz üzerinden gönderir.  |


## DNS Proxying

Varsayılan olarak, Nmap hedef hakkında daha fazla bilgi edinmek için reverse DNS çözümlemesi gerçekleştirir. Bu **DNS sorguları**, çoğu durumda başarılı olur çünkü verilen **web server**, bulunup ziyaret edilmesi gereken bir servistir. **DNS sorguları** genellikle **UDP port 53** üzerinden gerçekleştirilir. Önceden, **TCP port 53** yalnızca **"Zone transfer"** işlemleri veya **512 bayttan büyük veri transferleri** için kullanılıyordu. Ancak, **IPv6** ve **DNSSEC** genişlemeleri nedeniyle giderek daha fazla **TCP port 53** kullanımı yaygınlaşmaktadır.

Yine de, **Nmap bize kendi DNS serverlarımızı belirleme (`--dns-server <ns>,<ns>`) seçeneğini sunar**. Bu yöntem, özellikle **DMZ (Demilitarized Zone)** içinde bulunuyorsak önemli olabilir. **Şirketin DNS serverları**, genellikle internet üzerindeki DNS serverlarına kıyasla daha güvenilirdir. Bu nedenle, **iç ağdaki hostlarla etkileşim kurmak** için kullanılabilirler.

Buna ek olarak, **TCP port 53'ü kaynak portu (`--source-port`)** olarak belirleyerek taramalar gerçekleştirebiliriz. Eğer **yönetici, bu portu firewall ile kontrol ediyor ancak IDS/IPS filtrelemesi düzgün yapılandırılmamışsa**, **TCP paketlerimiz güvenilir olarak algılanabilir ve geçişine izin verilebilir**.

#### SYN-Scan of a Filtered Port

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

#### SYN-Scan From DNS Port

```shell-session
C4RT3L@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

|**Scanning Options**|**Description**|
|---|---|
|**10.129.2.28**|Belirtilen hedefi tarar.|
|**-p 50000**|Yalnızca belirtilen portları tarar.|
|**-sS**|Belirtilen portlarda **SYN scan** gerçekleştirir.|
|**-Pn**|**ICMP Echo** isteklerini devre dışı bırakır.|
|**-n**|**DNS çözümlemesini** devre dışı bırakır.|
|**--disable-arp-ping**|**ARP ping** işlemini devre dışı bırakır.|
|**--packet-trace**|Gönderilen ve alınan tüm **paketleri gösterir**.|
|**--source-port 53**|**Belirtilen kaynak portundan** tarama gerçekleştirir.|

Firewall'un TCP port 53'ü kabul ettiğini öğrendiğimize göre, IDS/IPS filtrelerinin de diğerlerinden çok daha zayıf yapılandırılmış olması muhtemeldir. Bunu Netcat kullanarak bu porta bağlanmaya çalışarak test edebiliriz.

#### Connect To The Filtered Port

```shell-session
C4RT3L@htb[/htb]$ ncat -nv --source-port 53 10.129.2.28 50000

Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

## Firewall and IDS/IPS Evasion Labs

Sonraki üç bölümde, hedefimizi taramamız gereken farklı senaryolar üzerinde pratik yapacağız. **Firewall** kuralları ve **IDS/IPS** sistemleri hedefleri koruduğundan, **firewall** kurallarını atlatmak ve bunu mümkün olduğunca sessiz yapmak için gösterilen teknikleri kullanmamız gerekecek. Aksi takdirde **IPS** tarafından engelleniriz.

Bize sadece IDS/IPS sistemleri tarafından korunan ve test edilebilen bir makine verilmektedir. Öğrenme amacıyla ve IDS/IPS'in nasıl davranabileceğine dair bir fikir edinmek için, şu adresteki bir durum web sayfasına erişimimiz vardır:

![Pasted image 20250219230239.png](/img/user/resimler/Pasted%20image%2020250219230239.png)

Bu sayfa bize uyarı sayısını gösterir. Belirli bir miktarda uyarı alırsak yasaklanacağımızı biliyoruz. Bu nedenle hedef sistemi mümkün olduğunca sessiz bir şekilde test etmeliyiz.


```
sudo nmap 10.129.143.75 -A -p 80,22 -Pn -n -S 10.10.15.9 -e tun0
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-19 14:29 CST
Nmap scan report for 10.129.143.75
Host is up (0.0086s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (95%), Linux 5.0 (95%), Linux 5.0 - 5.4 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), HP P2000 G3 NAS device (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT     ADDRESS
1   8.27 ms 10.10.14.1
2   8.72 ms 10.129.143.75

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.45 seconds
```

Soru : Müşterimiz, kendilerine sağlanan makinenin hangi işletim sistemi üzerinde çalıştığını tespit edip edemeyeceğimizi öğrenmek istiyor. Cevap olarak işletim sistemi adını gönderin.

Cevap : `Ubuntu`


# Firewall and IDS/IPS Evasion - Medium Lab
 
İlk testi gerçekleştirdikten ve sonuçları müşterimize sunduktan sonra, yöneticiler IDS/IPS ve firewall üzerinde bazı değişiklikler ve iyileştirmeler yaptı. Toplantıda yöneticilerin önceki yapılandırmalarından memnun olmadıklarını duyduk ve ağ trafiğinin daha sıkı bir şekilde filtrelenebileceğini fark ettiklerini söylediler.

Not: Alıştırmayı başarıyla çözmek için VPN üzerinde UDP protokolünü kullanmalıyız.

Soru : Yapılandırmalar sisteme aktarıldıktan sonra, müşterimiz hedefin DNS sunucu sürümünü öğrenmek istiyor. Hedefin DNS sunucu sürümünü cevap olarak gönderin.

Cevap : `HTB{GoTtgUnyze9Psw4vGjcuMpHRp}`

```
nmap -sU -p 53 -D RND:5 --disable-arp-ping -Pn --max-retries 1 --host-timeout 30s 10.129.106.110 -sV
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-19 14:42 CST
Nmap scan report for 10.129.106.110
Host is up (0.0089s latency).

PORT   STATE SERVICE VERSION
53/udp open  domain  (unknown banner: HTB{GoTtgUnyze9Psw4vGjcuMpHRp})
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-UDP:V=7.94SVN%I=7%D=2/19%Time=67B64230%P=x86_64-pc-linux-gnu%r(D
SF:NSVersionBindReq,57,"\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\x04b
SF:ind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x1f\x1eHTB{GoTtgUnyze9
...SNIP...
```


- `-sU`: UDP taraması yapar.
- `-p 53`: 53 numaralı portu tarar (DNS).
- `-D RND:5`: 5 sahte IP ile tarama yapar.
- `--disable-arp-ping`: ARP pingini devre dışı bırakır.
- `-Pn`: Ping yapmadan taramaya başlar.
- `--max-retries 1`: 1 defa yeniden dener.
- `--host-timeout 30s`: Hedef için 30 saniyelik zaman aşımı belirler.
- `10.129.106.110`: Hedef IP adresi.
- `-sV`: Servis sürümünü tespit eder.


# Firewall and IDS/IPS Evasion - Hard Lab

İkinci testimizin yardımıyla, müşterimiz yeni bilgiler edindi ve bir yöneticisini IDS/IPS sistemleri için bir eğitim kursuna gönderdi. Müşterimizin söylediğine göre, eğitim bir hafta sürecekti. Şimdi yönetici gerekli tüm önlemleri aldı ve bize tekrar test etmemizi istiyor çünkü belirli servislerin değiştirilmesi ve sağlanan yazılımın iletişiminin modifiye edilmesi gerekiyordu.


Soru : 

Cevap: `HTB{kjnsdf2n982n1827eh76238s98di1w6}`


```
sudo nmap 10.129.127.42 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

Nmap scan report for 10.129.127.42
Host is up (0.0087s latency).
Not shown: 869 closed tcp ports (reset), 128 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50000/tcp open  ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 1.70 seconds
```

```
sudo nc -nv -p 53 10.129.127.42 50000
(UNKNOWN) [10.129.127.42] 50000 (?) open
220 HTB{kjnsdf2n982n1827eh76238s98di1w6}
```
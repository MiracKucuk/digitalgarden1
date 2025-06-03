---
{"dg-publish":true,"permalink":"/ctf/permx-ctf/"}
---


```shell-session
nmap -p- --min-rate 10000 10.10.11.23                                                       
Nmap scan report for 10.10.11.23
Host is up (0.20s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 9.43 seconds
```

İki port açık 22 ve 80  . 

![Pasted image 20241026152011.png](/img/user/resimler/Pasted%20image%2020241026152011.png)

- **22/tcp (SSH)**:
    - **Açık**: Hedef makinede SSH hizmeti çalışıyor.
    - **Versiyon**: OpenSSH 8.9p1, bu da güncel bir sürüm olduğunu gösteriyor.
    - **Anahtarlar**: ECDSA ve ED25519 anahtarlarının parmak izleri verilmiş. Bu, güvenlik değerlendirmeleri için kullanılabilir.
- **80/tcp (HTTP)**:
    - **Açık**: Hedef makinede Apache HTTP sunucusu çalışıyor.
    - **Versiyon**: Apache httpd 2.4.52. Bu da güncel bir sürüm, ancak güvenlik açıkları olabilir.
    - **HTTP Başlığı**: Apache/2.4.52 (Ubuntu).
    - **HTTP Başlığı ile Yönlendirme**: Tarayıcı yönlendirmesini takip etmediğinden, muhtemelen bir HTTP 3xx durumu ile karşılaşılıyor. Bu, kullanıcıyı başka bir URL'ye yönlendiren bir uygulama olabilir. Çıktıdaki bilgiye göre, hedef `http://permx.htb` adresine yönlendiriyor. Bu yönlendirme, hedef sistemde başka bir uygulama veya hizmetin çalıştığını gösterebilir.


### Subdomain Fuzz
![Pasted image 20241026152340.png](/img/user/resimler/Pasted%20image%2020241026152340.png)

/etc/hosts dosyasına ekleyin . 

![Pasted image 20241026152754.png](/img/user/resimler/Pasted%20image%2020241026152754.png)

www ve lms olan 2 subdomain aldık . 

![Pasted image 20241026152919.png](/img/user/resimler/Pasted%20image%2020241026152919.png)


### Dirsearch
![Pasted image 20241026153117.png](/img/user/resimler/Pasted%20image%2020241026153117.png)

![Pasted image 20241026153232.png](/img/user/resimler/Pasted%20image%2020241026153232.png)

![Pasted image 20241026153251.png](/img/user/resimler/Pasted%20image%2020241026153251.png)
![Pasted image 20241026153420.png](/img/user/resimler/Pasted%20image%2020241026153420.png)
![Pasted image 20241026153437.png](/img/user/resimler/Pasted%20image%2020241026153437.png)
![Pasted image 20241026153448.png](/img/user/resimler/Pasted%20image%2020241026153448.png)


****LICENSE**** içinde ****Chamilo LMS**** sürüm bilgilerini alın

![Pasted image 20241026153640.png](/img/user/resimler/Pasted%20image%2020241026153640.png)

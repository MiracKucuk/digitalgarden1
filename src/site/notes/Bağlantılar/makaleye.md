---
{"dg-publish":true,"permalink":"/baglantilar/makaleye/"}
---

'.htpasswd' dosyası, genellikle Apache ve Nginx gibi web sunucusu yazılımları tarafından HTTP temel kimlik doğrulaması (HTTP Basic Authentication) için kullanılan, kullanıcı adları ve şifreleri depolayan bir düz metin dosyasıdır. Bu dosya, her satırda bir kullanıcı adı ve ona karşılık gelen şifreyi içerir; kullanıcı adı ve şifre iki nokta üst üste (:) ile ayrılır. Şifreler, güvenlik amacıyla açık metin yerine şifrelenmiş biçimde saklanır.


Örnek .htpasswd Dosyası İçeriği:

```
ismail:$apr1$/5EzxSg3$SXVemrqNIb/TrKvJv4Z5r0 ahmet:$apr1$cEKbD/Wa$M012WG8Txqp/dhso8.znk0 ali:$apr1$o/t1Efly$E7798GsGjMWoNUpqmG4l60
```


**.htpasswd Dosyası Nasıl Oluşturulur?**

'.htpasswd' dosyası, 'htpasswd' komutu kullanılarak oluşturulabilir ve düzenlenebilir. Aşağıdaki komut, belirtilen dosyayı oluşturur ve 'ismail' kullanıcısını ekler:

```
htpasswd -c .htpasswd ismail
```

Eğer dosyanın tam yolunu belirtmek isterseniz:

```
htpasswd -c /var/www/mysite/.htpasswd ismail
```

Alternatif olarak, 'touch' komutuyla boş bir '.htpasswd' dosyası oluşturabilirsiniz:

```
touch .htpasswd
```

Veya bir metin düzenleyici (ör. 'nano', 'vi', 'vim') kullanarak dosyayı oluşturup düzenleyebilirsiniz:

```
nano .htpasswd
```

.htpasswd Dosyasını Yapılandırma

'.htpasswd' dosyası, Apache gibi web sunucularında kullanıcı kimlik doğrulaması için kullanılır. Aşağıdaki yapılandırma, HTTP temel kimlik doğrulamasını etkinleştirir ve kullanıcı adı ile şifre doğrulaması için '.htpasswd' dosyasını belirtir:


```
AuthUserFile /var/www/mysite/.htpasswd
AuthType Basic
AuthName "Poftut HTTP Authentication"
Require valid-user
```

Bu yapılandırma, belirtilen '.htpasswd' dosyasındaki geçerli kullanıcıların kimlik doğrulamasını gerektirir.


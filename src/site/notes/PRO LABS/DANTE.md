---
{"dg-publish":true,"permalink":"/pro-labs/dante/"}
---


10.10.110.0/24 subnet'ini tarayalım. Aktif hostları taramadan önce tanımlamak için bir ping taraması başlatabiliriz.

`nmap -sn -T4 10.10.110.0/24 -oN active-hosts`

![Pasted image 20241218025438.png](/img/user/resimler/Pasted%20image%2020241218025438.png)

Nmap -sn bayrağı port taramasını devre dışı bırakır ve ICMP isteklerine göre hostları keşfeder. İki aktif host buldu, bunlardan 10.10.110.2 lab controller olduğu için göz ardı edilebilir.


### Senin için deli oluyorum.

Bu hostlarda açık olan portları keşfetmek için servis ve versiyon numaralandırma ile tam bir port SYN taraması yapalım

![Pasted image 20241218025631.png](/img/user/resimler/Pasted%20image%2020241218025631.png)

![Pasted image 20241218025725.png](/img/user/resimler/Pasted%20image%2020241218025725.png)

Nmap çıktısı, 10.10.110.100 hostunun 21 (FTP), 22 (SSH) ve 65000 (HTTP) portlarının açık olduğunu ortaya koymaktadır. Anonim FTP girişi etkinleştirilmiştir. FTP servisine anonymous / anonymous kimlik bilgileriyle giriş yapalım.

![Pasted image 20241218025854.png](/img/user/resimler/Pasted%20image%2020241218025854.png)

Transfer/Incoming klasöründe bir todo.txt dosyası var. Get komutunu kullanarak dosyayı indirelim.

![Pasted image 20241218030018.png](/img/user/resimler/Pasted%20image%2020241218030018.png)

Dosyanın içeriği:

![Pasted image 20241218030154.png](/img/user/resimler/Pasted%20image%2020241218030154.png)

- **Wordpress izin değişikliklerini tamamla** - BEKLEMEDE
- **Bağlantıları DNS adını kullanacak şekilde güncelle, port 80'e geçmeden önce** - BEKLEMEDE
- **Diğer sitedeki LFI zafiyetini kaldır** - BEKLEMEDE
- **James'in şifresini daha güvenli bir şeye sıfırla** - BEKLEMEDE
- **Junior Penetrasyon Testi değerlendirmesinden önce sistemi güçlendir** - DEVAM EDİYOR


Dosya, bir sitedeki LFI güvenlik açığından bahsetmekte ve James kullanıcısının zayıf bir parolaya sahip olduğunu ifşa etmektedir. Nmap çıktısı ayrıca Apache sunucusunda bir robots.txt dosyası olduğunu göstermektedir. Bu dosya, web tarayıcılarına (arama motorları) belirtilen sunucu yollarını ve dosyalarını taramamalarını söylemek için kullanılır. İlk flag robots.txt dosyasında bulunabilir.

![Pasted image 20241218030243.png](/img/user/resimler/Pasted%20image%2020241218030243.png)


### Bu şekilde daha kolay

Robots.txt'deki giriş, sunucuda WordPress CMS'nin yüklü olduğunu gösterir. wordpress klasörüne göz atmak bunu doğrular

![Pasted image 20241218030539.png](/img/user/resimler/Pasted%20image%2020241218030539.png)

WordPress güvenliğini wpscan kullanarak daha otomatik bir şekilde değerlendirebiliriz.

![Pasted image 20241218030644.png](/img/user/resimler/Pasted%20image%2020241218030644.png)

**`--enumerate vp`**:  
Sitedeki **"vulnerable plugins"** yani **güvenlik zafiyeti olan eklentileri** tarayıp listeler.

`vp` burada **"vulnerable plugins"**'in kısa kodudur.

![Pasted image 20241218030736.png](/img/user/resimler/Pasted%20image%2020241218030736.png)


vp bayrağı güvenlik açığı olan eklentileri tarar. wpscan sunucunun dizin listelemesinin etkin olduğunu ve WordPress sürümünün en son sürüm olmayan 5.4.1 olduğunu tespit etti. wpvulndb üzerinde bilinen güvenlik açıklarının kontrol edilmesi aşağıdaki sonuçları göstermektedir.

![Pasted image 20241218031029.png](/img/user/resimler/Pasted%20image%2020241218031029.png)

Ancak bunların hiçbiri sunucuya doğrudan erişmemize izin vermez. Kullanıcıları wpscan ile numaralandıralım.


![Pasted image 20241218031058.png](/img/user/resimler/Pasted%20image%2020241218031058.png)

![Pasted image 20241218031111.png](/img/user/resimler/Pasted%20image%2020241218031111.png)

Böylece admin ve james kullanıcıları tespit edildi. Ayrıca Meet The Team sayfasına da göz atalım

![Pasted image 20241218031212.png](/img/user/resimler/Pasted%20image%2020241218031212.png)

Çalışan adlarını names.txt dosyasına kaydedin.

![Pasted image 20241218031333.png](/img/user/resimler/Pasted%20image%2020241218031333.png)

Languages and Frameworks sayfası ekibin kullandığı programlama dilleri hakkında bazı detaylar içeriyor. CeWL kullanarak bu sayfadan bir kelime listesi oluşturalım.

![Pasted image 20241218031454.png](/img/user/resimler/Pasted%20image%2020241218031454.png)

Daha sonra, oluşturulan username ve password listeleri ile WordPress girişini wpscan ile bruteforce etmeye çalışabiliriz

![Pasted image 20241218031630.png](/img/user/resimler/Pasted%20image%2020241218031630.png)

![Pasted image 20241218031656.png](/img/user/resimler/Pasted%20image%2020241218031656.png)

Bu, james / Toyota WordPress kimlik bilgilerinin geçerli olduğunu ortaya çıkardı. Ayrıca, kazanılan kimlik bilgilerinin SSH gibi diğer hizmetlerde oturum açmak için kullanılıp kullanılamayacağını kontrol etmeye değer. Ancak, bu başarılı değildir.

![Pasted image 20241218031728.png](/img/user/resimler/Pasted%20image%2020241218031728.png)

wordpress/wp-admin/ adresine gidin ve kimlik bilgileriyle giriş yapın.

![Pasted image 20241218031751.png](/img/user/resimler/Pasted%20image%2020241218031751.png)

Kullanıcılar sayfası incelendiğinde james kullanıcısının Administrator rolünün bir üyesi olduğu görülür. Temaları özelleştirmek için WordPress, Administrator rolü üyelerinin yerleşik düzenleyicisini kullanarak dosyaları düzenlemelerine izin verir. Bu, dosyalardan birine php kodu ekleyerek sunucu üzerinde komutları yürütmek için kullanılabilir. “ Appearance” > “Theme Editor” bölümüne gidin ve Twenty Nineteen temasını seçin

![Pasted image 20241218031948.png](/img/user/resimler/Pasted%20image%2020241218031948.png)

404.php sayfa içeriğini aşağıdakilerle güncelleyin:

![Pasted image 20241218032011.png](/img/user/resimler/Pasted%20image%2020241218032011.png)

![Pasted image 20241218032023.png](/img/user/resimler/Pasted%20image%2020241218032023.png)

Ardından, 1234 numaralı portta bir listener kurun ve reverse shell'i tetiklemek için /wordpress/wpcontent/themes/twentynineteen/404.php sayfasına erişin.

![Pasted image 20241218032046.png](/img/user/resimler/Pasted%20image%2020241218032046.png)

DANTE-WEB-NIX01 üzerinde bir shell www-data olarak alınır. Şimdi /home klasörünü kontrol edelim

![Pasted image 20241218032102.png](/img/user/resimler/Pasted%20image%2020241218032102.png)

Sunucuda iki kullanıcı mevcut. Sistem hesaplarıyla tekrar kullanmış olma ihtimaline karşı WordPress şifresini kullanarak james'e geçmeyi deneyebiliriz.

![Pasted image 20241218032125.png](/img/user/resimler/Pasted%20image%2020241218032125.png)

Bu başarılı. Python3 kullanarak bir PTY shell'e geçelim.

![Pasted image 20241218032142.png](/img/user/resimler/Pasted%20image%2020241218032142.png)

Bayrak, kullanıcının home klasöründe bulunabilir.

![Pasted image 20241218032155.png](/img/user/resimler/Pasted%20image%2020241218032155.png)

---


### Bana yolu göster

.bash_history dosyasının içeriği kontrol edildiğinde balthazar kullanıcısına ait MySQL kimlik bilgileri görülür.

![Pasted image 20241218121551.png](/img/user/resimler/Pasted%20image%2020241218121551.png)

![Pasted image 20241218121916.png](/img/user/resimler/Pasted%20image%2020241218121916.png)

![Pasted image 20241218121925.png](/img/user/resimler/Pasted%20image%2020241218121925.png)

[gtfobins](https://gtfobins.github.io/gtfobins/find/#suid) referansına göre, bu komut kullanılarak sunucuda root erişimi elde edilebilir: [[Bağlantılar/SUİD\|SUİD]] 

![Pasted image 20241218122723.png](/img/user/resimler/Pasted%20image%2020241218122723.png)

[[Bağlantılar/Açıklama ..\|Açıklama ..]]

![Pasted image 20241218122839.png](/img/user/resimler/Pasted%20image%2020241218122839.png)

Bu flag root klasöründe bulunabilir

![Pasted image 20241218122859.png](/img/user/resimler/Pasted%20image%2020241218122859.png)


### Seclusion is an illusion

ifconfig çalıştırıldığında 172.16.1.0/24 subnet'inde olduğumuz görülür. Bir ping taraması kullanarak subnet üzerindeki canlı hostları belirleyelim.

![Pasted image 20241218131827.png](/img/user/resimler/Pasted%20image%2020241218131827.png)

![Pasted image 20241218131834.png](/img/user/resimler/Pasted%20image%2020241218131834.png)

Bu, 172.16.1.100 iç IP adresine sahip mevcut hostumuz NIX01 dışında toplam on bir hostu ortaya çıkarır. Public anahtarımızı /root/.ssh/authorized_keys dosyasına ekleyelim ve bir SSH shell'e yükseltelim.

Makinemizden 172.16.1.0/24 ağındaki hostları daha fazla numaralandırmak için yerel bir SOCKS proxy kurabiliriz.

![Pasted image 20241218131931.png](/img/user/resimler/Pasted%20image%2020241218131931.png)

SOCKS port 1080 üzerinden istekleri proxylemek için /etc/proxychains.conf dosyasını düzenleyin

![Pasted image 20241218131951.png](/img/user/resimler/Pasted%20image%2020241218131951.png)

ProxyChains, TCP bağlantılarını TOR, SOCKS ve HTTP/HTTPS proxy sunucuları üzerinden yeniden yönlendirebilen ve aynı zamanda birden fazla proxy sunucusunu birbirine zincirlememizi sağlayan bir araçtır. Şimdi 172.16.1.10 hostunu tarayarak başlayalım. Nmap SYN taraması proxy farkında değildir ve Nmap --proxies seçeneği için sınırlı destek vardır. Bunun yerine, trafiği proxy üzerinden yönlendiren daha üst düzey bir connect sistem çağrısı yayınlamak için işletim sistemini kullanan bir TCP Connect Scan ( -sT ) kullanabiliriz

![Pasted image 20241218132141.png](/img/user/resimler/Pasted%20image%2020241218132141.png)

![Pasted image 20241218132208.png](/img/user/resimler/Pasted%20image%2020241218132208.png)

Bu, hostun 22 (SSH), 80 (HTTP) ve 139/445 (Samba) portlarının açık olduğunu gösterir. Proxychains kullanarak bir tarayıcı açmak ve 80 numaralı porta göz atmak Dante Hosting şirketi için bir site ortaya çıkarır.

![Pasted image 20241218132314.png](/img/user/resimler/Pasted%20image%2020241218132314.png)

![Pasted image 20241218132330.png](/img/user/resimler/Pasted%20image%2020241218132330.png)

Mevcut sayfalara bir göz atalım.

![Pasted image 20241218132346.png](/img/user/resimler/Pasted%20image%2020241218132346.png)

URL'ye baktığımızda, uygulamanın bir sayfa parametresi kullanarak farklı sayfalar içerdiğini görebiliriz. Bu, olası bir local file inclusion (LFI) güvenlik açığı için iyi bir işarettir. Bunu /etc/passwd dosyasına erişmeye çalışarak test edelim.

![Pasted image 20241218132410.png](/img/user/resimler/Pasted%20image%2020241218132410.png)

![Pasted image 20241218132418.png](/img/user/resimler/Pasted%20image%2020241218132418.png)

Bu başarılı oldu ve frank ve margaret kullanıcılarını görüyoruz. Şimdi smbclient aracını kullanarak Samba servisine bağlanalım.

![Pasted image 20241218132439.png](/img/user/resimler/Pasted%20image%2020241218132439.png)

![Pasted image 20241218132447.png](/img/user/resimler/Pasted%20image%2020241218132447.png)

Root username ve boş parola ile oturum açma başarılıdır, bu da SMB NULL oturumlarına izin verildiği anlamına gelir. Bir SlackMigration paylaşımı var. Hadi smbclient kullanarak ona bağlanalım.

![Pasted image 20241218132556.png](/img/user/resimler/Pasted%20image%2020241218132556.png)

![Pasted image 20241218132602.png](/img/user/resimler/Pasted%20image%2020241218132602.png)

admintasks.txt dosyası, içeriğini görüntülemek için indirdiğimiz paylaşımda mevcuttur.

![Pasted image 20241218132626.png](/img/user/resimler/Pasted%20image%2020241218132626.png)

![Pasted image 20241218132634.png](/img/user/resimler/Pasted%20image%2020241218132634.png)

- **Wordpress kurulumunu web kök dizininden kaldır** - **BEKLEMEDE**
- **Ubuntu makinesinde Slack entegrasyonunu yeniden etkinleştir** - **BEKLEMEDE**
- **Eski çalışan hesaplarını kaldır** - **TAMAMLANDI**
- **Margaret'i yeni değişiklikler hakkında bilgilendir** - **TAMAMLANDI**
- **Margaret'in hesap kısıtlamalarını yönetici terfisinden sonra kaldır** - **BEKLEMEDE**

Bu, WordPress CMS'nin web root'unda yüklü olduğunu belirtir. Hadi ona erişmeyi deneyelim


![Pasted image 20241218132907.png](/img/user/resimler/Pasted%20image%2020241218132907.png)

WordPress CMS'nin doğrudan erişilebilir olmadığı görülüyor. WordPress CMS'nin **/var/www/html/** web root dizinine kurulu olması ve Dante barındırma uygulamasının bir alt dizinden sunulması durumu olabilir. Bunu, **LFI (Local File Inclusion)** zafiyetini kullanarak **/var/www/html/index.html** dosyasına erişerek doğrulayalım.

![Pasted image 20241218133142.png](/img/user/resimler/Pasted%20image%2020241218133142.png)

Varsayılan apache sayfasının Web root üzerinde mevcut olduğunu görebiliriz. WordPress sayfası index.php'yi yüklemeyi deneyelim.

![Pasted image 20241218133249.png](/img/user/resimler/Pasted%20image%2020241218133249.png)

![Pasted image 20241218133256.png](/img/user/resimler/Pasted%20image%2020241218133256.png)

Hata mesajı WordPress'in web root'unda kurulu olduğunu doğrulamaktadır. Genel olarak, WordPress veritabanı yapılandırmasını wp-config.php dosyasında saklar.

HP, dosyalara, protokollere veya akışlara daha kolay erişim için kullanılabilecek çeşitli wrappers sağlar. Wrappers'ın tam listesi [burada](https://www.php.net/manual/en/wrappers.php) bulunabilir. php:// wrapper varsayılan olarak etkindir ve IO akışlarıyla etkileşim için kullanılır. Örneğin: php://stdin ve php://stdout proses için girdi ve çıktı akışlarına erişmek için kullanılabilirken, php://input ve php://output istek verilerine erişmek için kullanılır. Yararlı bir wrapper, istenen çıktıyı elde etmek için birden fazla filtre ile zincirlenebilen php://filter 'dır.

Örneğin, php://filter/read=/resource=/etc/passwd filtresi /etc/passwd kaynağını okur ve içeriğini çıktı olarak verir. Okuma filtresi, girdiyi base64 kodlaması, ROT13 kodlaması vb. gibi çeşitli string işlemleri ile işlemek için kullanılabilir.

Aşağıdaki filtreler /etc/passwd içeriğini sırasıyla base64 ve ROT13'e dönüştürecektir.

![Pasted image 20241218133455.png](/img/user/resimler/Pasted%20image%2020241218133455.png)

Filtreyi kullanarak base64 kodlamayı ve wp-config.php'yi içermeyi deneyelim.

![Pasted image 20241218133522.png](/img/user/resimler/Pasted%20image%2020241218133522.png)


![Pasted image 20241218133534.png](/img/user/resimler/Pasted%20image%2020241218133534.png)

Wrapper, verilen kaynağı base64 biçiminde kodlar. İçeriğin kodunu çözelim ve bir dosyaya kaydedelim

![Pasted image 20241218133554.png](/img/user/resimler/Pasted%20image%2020241218133554.png)

Dosyanın incelenmesi, beklendiği gibi veritabanı kimlik bilgilerini ortaya çıkarır.

![Pasted image 20241218133607.png](/img/user/resimler/Pasted%20image%2020241218133607.png)

Bu kimlik bilgilerini SSH hizmetinde deneyelim.


![Pasted image 20241218133624.png](/img/user/resimler/Pasted%20image%2020241218133624.png)

![Pasted image 20241218133630.png](/img/user/resimler/Pasted%20image%2020241218133630.png)

margaret kullanıcısı olarak başarılı bir şekilde oturum açtık, ancak shell erişimimiz kısıtlandı. vim izin verilen komutlar arasında mevcut. [gtfobins](https://gtfobins.github.io/gtfobins/vim/#shell), kısıtlı shell'den kaçmak ve tam shell erişimi elde etmek için bunu kullanabileceğimizi ortaya koyuyor. vim komutunu çalıştırın ve aşağıdaki komutları uygulayın.

![Pasted image 20241218133714.png](/img/user/resimler/Pasted%20image%2020241218133714.png)

![Pasted image 20241218133721.png](/img/user/resimler/Pasted%20image%2020241218133721.png)

DANTE-NIX02 üzerinde margaret olarak tam bir shell elde edilir. Flag home dizininde bulunabilir.

![Pasted image 20241218133744.png](/img/user/resimler/Pasted%20image%2020241218133744.png)

Kullanıcının home klasörünün numaralandırılması .config içinde bir Slack alt dizini ortaya çıkarır.

![Pasted image 20241218133809.png](/img/user/resimler/Pasted%20image%2020241218133809.png)

Önceki numaralandırmada Slack entegrasyon görevinin beklemede olduğu belirtilmişti. Genel olarak Slack, her kullanıcı için dosyaları gizli `/home/<user>/.config/Slack` dizininde saklar. Bir Slack admin workspace verilerini dışa aktardığında, bunların çoğunu JSON formatında saklar. Bu dizini numaralandırdıktan sonra, chat günlükleri, channel ve workspace verileri de dahil olmak üzere bazı dışa aktarılmış verileri buluruz.

![Pasted image 20241218133923.png](/img/user/resimler/Pasted%20image%2020241218133923.png)


Secure channel sohbet kayıtları incelendiğinde Margaret ve Frank arasında geçen ve frank kullanıcısı için potansiyel kimlik bilgilerini içeren bir konuşma ortaya çıkar.

![Pasted image 20241218134139.png](/img/user/resimler/Pasted%20image%2020241218134139.png)

Elde edilen şifreyi kullanarak frank'a geçelim.

![Pasted image 20241218134152.png](/img/user/resimler/Pasted%20image%2020241218134152.png)

Home klasöründe bulunan bir apache_restart.py scripti vardır

![Pasted image 20241218134215.png](/img/user/resimler/Pasted%20image%2020241218134215.png)

İçeriğini kontrol edelim.

![Pasted image 20241218134314.png](/img/user/resimler/Pasted%20image%2020241218134314.png)


Script, localhost'a bir GET isteği göndererek apache2 sunucu hizmet durumunu kontrol eder. Yanıt kodu 200'e eşit değilse hizmeti yeniden başlatır. Scriptte kontrolü düzenli aralıklarla tekrarlayan bir while döngüsü bulunmaması dikkat çekicidir, bu da bir cronjob yapılandırılmış olabileceğine işaret eder. Bunu pspy'yi indirip çalıştırarak doğrulayabiliriz.

**pspy**, **Linux sistemlerinde çalışan süreçleri (process) ve cron görevlerini** tespit etmek için kullanılan bir **process monitoring** aracıdır. Özellikle **root yetkilerine** ihtiyaç duymadan çalışan işlemleri görmek için kullanılır, bu da onu **privilege escalation** (yetki yükseltme) senaryolarında oldukça popüler bir araç haline getirir.

![Pasted image 20241218134852.png](/img/user/resimler/Pasted%20image%2020241218134852.png)

pspy çıktısı, root'un scripti her dakika çalıştırdığını ortaya koyuyor. Script, call ve urllib modüllerini içe aktarmasına rağmen frank tarafından yazılabilir değildir. Python'un kütüphaneleri için bir arama yolları listesi vardır. Bu, aşağıdaki komutlar çalıştırılarak görüntülenebilir.

![Pasted image 20241218135413.png](/img/user/resimler/Pasted%20image%2020241218135413.png)

Python, diğer yollara bakmadan önce ilk olarak geçerli çalışma dizinindeki modülleri kontrol eder. Yasal call veya urllib modüllerinin yüklenmesini engelleyerek bunun yerine kötü niyetli modülümüzün yüklenmesini sağlayabiliriz. Şimdi /home/frank klasöründe aşağıdaki içeriğe sahip bir urllib.py dosyası oluşturalım.

![Pasted image 20241218135512.png](/img/user/resimler/Pasted%20image%2020241218135512.png)

Bir dakika sonra /tmp/sh dosyasının oluşturulduğunu görebiliriz.

![Pasted image 20241218135540.png](/img/user/resimler/Pasted%20image%2020241218135540.png)

Aşağıdaki komut çalıştırılarak bir root kabuğu elde edilebilir

![Pasted image 20241218135556.png](/img/user/resimler/Pasted%20image%2020241218135556.png)

![Pasted image 20241218135611.png](/img/user/resimler/Pasted%20image%2020241218135611.png)

Bash, çağırma sırasında -p seçeneği sağlandığında etkin kullanıcı kimliğini (yani root) korur, bu da bize root erişimi sağlar. Bu bayrağı /root klasöründe bulabilirsiniz. Daha sonra SSH public anahtarımızı authorized_keys'e ekleyelim ve suid ikilisini silelim.

![Pasted image 20241218135700.png](/img/user/resimler/Pasted%20image%2020241218135700.png)


### Şimdi 172.16.1.17 hostunu tarayalım

![Pasted image 20241218135853.png](/img/user/resimler/Pasted%20image%2020241218135853.png)

![Pasted image 20241218135859.png](/img/user/resimler/Pasted%20image%2020241218135859.png)

Nmap, hostun 80 (HTTP,) 139,445 (Samba) ve 10000 (HTTP) portlarının açık olduğunu gösterir. Şimdi 10000 numaralı porta göz atalım.

![Pasted image 20241218135930.png](/img/user/resimler/Pasted%20image%2020241218135930.png)

Webmin uygulaması 10000 numaralı portta çalışıyor. Varsayılan kimlik bilgilerini kullanarak oturum açmaya çalışmak başarılı değil. Smbclient kullanarak Samba hizmetine bağlanalım.

![Pasted image 20241218135950.png](/img/user/resimler/Pasted%20image%2020241218135950.png)

![Pasted image 20241218135956.png](/img/user/resimler/Pasted%20image%2020241218135956.png)

Burada not-default forensics paylaşımını görüyoruz.

![Pasted image 20241218140728.png](/img/user/resimler/Pasted%20image%2020241218140728.png)

![Pasted image 20241218140734.png](/img/user/resimler/Pasted%20image%2020241218140734.png)


Share monitör adlı bir dosya içeriyor. Hadi indirelim.

![Pasted image 20241218141000.png](/img/user/resimler/Pasted%20image%2020241218141000.png)

File komutu, bunun muhtemelen tcpdump veya Wireshark ile alınmış bir paket yakalama olan bir pcap olduğunu ortaya koymaktadır. Dosyayı Wireshark'ta analiz edelim.

![Pasted image 20241218141059.png](/img/user/resimler/Pasted%20image%2020241218141059.png)

![Pasted image 20241218141349.png](/img/user/resimler/Pasted%20image%2020241218141349.png)

UDP'nin yanı sıra TCP trafiği de mevcuttur. TCP altında, HTML Form URL Encoded paketlerinin mevcut olduğunu görebiliriz. Bu paketler HTTP protokolü kullanılarak iletilir. HTTP paketleri için filtreleme yapalım.

![Pasted image 20241218141436.png](/img/user/resimler/Pasted%20image%2020241218141436.png)

HTTP POST istek paketine sağ tıklayın ve “ Follow” > “HTTP Stream” seçeneğine tıklayın.

![Pasted image 20241218141504.png](/img/user/resimler/Pasted%20image%2020241218141504.png)

Paket, Webmin uygulamasına giriş yapmak için yapılan bir POST isteğini ortaya koymaktadır. Webmin uygulamasına admin / password6543 kimlik bilgilerini kullanarak giriş yapalım.

![Pasted image 20241218141530.png](/img/user/resimler/Pasted%20image%2020241218141530.png)

Bu başarısız oldu. Yakalanan dosyanın daha ayrıntılı incelenmesi, Password6543 parolasını içeren başka bir POST isteğini ortaya çıkarır.

![Pasted image 20241218141812.png](/img/user/resimler/Pasted%20image%2020241218141812.png)

Bu şifreyi kullanarak uygulamaya erişim kazanırız

![Pasted image 20241218142026.png](/img/user/resimler/Pasted%20image%2020241218142026.png)

Dashboard, Webmin sürümünün 1.900 olduğunu gösterir. Ayrıca 1.930'un altındaki sürümlerin bir remote exploit'e karşı savunmasız olduğunu bildirmektedir.

![Pasted image 20241218142444.png](/img/user/resimler/Pasted%20image%2020241218142444.png)

Bu sürüm için bilinen açıklar arandığında aşağıdaki sonuçlar elde edilir

![Pasted image 20241218142538.png](/img/user/resimler/Pasted%20image%2020241218142538.png)

[roughiz](https://github.com/roughiz/Webmin-1.910-Exploit-Script) deposundan exploit'i indirin ve ardından bir reverse shell elde etmek için aşağıdaki komutları verin.

![Pasted image 20241218142653.png](/img/user/resimler/Pasted%20image%2020241218142653.png)

![Pasted image 20241218142714.png](/img/user/resimler/Pasted%20image%2020241218142714.png)

DANTE-NIX03 üzerinde bir shell alındı. Python kullanarak shell'i bir PTY'ye yükseltelim.

![Pasted image 20241218142734.png](/img/user/resimler/Pasted%20image%2020241218142734.png)
Flag root dizininde bulunabilir


![Pasted image 20241218142924.png](/img/user/resimler/Pasted%20image%2020241218142924.png)


Next up, let's scan 172.16.1.13 .

![Pasted image 20241218142955.png](/img/user/resimler/Pasted%20image%2020241218142955.png)


![Pasted image 20241218143002.png](/img/user/resimler/Pasted%20image%2020241218143002.png)

Nmap, hostun 80 (HTTP) ve 443 (HTTPS) portlarının açık olduğunu tespit etti. 80 numaralı porta göz atıldığında bir XAMPP panosu görülür.


![Pasted image 20241218143041.png](/img/user/resimler/Pasted%20image%2020241218143041.png)

phpmyadmin sayfasına uzaktan erişilemiyor

![Pasted image 20241218143109.png](/img/user/resimler/Pasted%20image%2020241218143109.png)


Gobuster'ı kullanarak 80 numaralı portta barındırılan diğer dosya ve klasörleri listeleyelim.

![Pasted image 20241218143131.png](/img/user/resimler/Pasted%20image%2020241218143131.png)

![Pasted image 20241218143528.png](/img/user/resimler/Pasted%20image%2020241218143528.png)

Bu bir /discuss dizini tanımladı. Buna göz atalım

![Pasted image 20241218143549.png](/img/user/resimler/Pasted%20image%2020241218143549.png)

Blog başlığı buranın bir Technical Discussion Forum olduğunu söylüyor ve bu terimde bilinen açıkları aramak bizi potansiyel bir açığa götürüyor.

![Pasted image 20241218144146.png](/img/user/resimler/Pasted%20image%2020241218144146.png)


```
# Exploit Title: Online Discussion Forum Site 1.0 - Remote Code Execution # Google Dork: N/A 
# Date: 2020-05-24 
# Exploit Author: Selim Enes 'Enesdex' Karaduman 
# Vendor Homepage: https://www.sourcecodester.com/php/14233/online-discussionforum-site.html 
# Software Link: https://www.sourcecodester.com/download-code? nid=14233&title=Online+Discussion+Forum+Site 
# Version: 1.0 (REQUIRED) # Tested on: Windows 10 / Wamp Server 
# CVE : N/A
```

---

1. **Register sayfasına gidin**:

http://localhost/Online%20Discussion%20Forum%20Site/register.php

Kayıt sayfasında üyelik oluşturun.

2. **Bir PHP shell yükleyin**:

Shell dosyasını aşağıdaki içerikle hazırlayın ve yükleyin:

![Pasted image 20241218144451.png](/img/user/resimler/Pasted%20image%2020241218144451.png)

3. **Shell'e erişim sağlayın ve komut çalıştırın**:

Kayıt işlemi tamamlandıktan sonra aşağıdaki URL'ye giderek **istemiş olduğunuz komutları çalıştırabilirsiniz**:

http://localhost/Online%20Discussion%20Forum%20Site/ups/shell.php?cmd=$THECODEYOU-WANT-TO-EXECUTE

---

Register Up butonuna tıklayarak exploit'te anlatıldığı gibi yeni bir kullanıcı kaydedelim.

![Pasted image 20241218144822.png](/img/user/resimler/Pasted%20image%2020241218144822.png)

Register formunda bir resim yükleme seçeneği vardır. Aşağıdaki kodu shell.php dosyasına kaydedin ve formu kullanarak yükleyin.

![Pasted image 20241218144913.png](/img/user/resimler/Pasted%20image%2020241218144913.png)

Kaydı tamamlamak ve webshell'i yüklemek için Gönder'e tıklayın

![Pasted image 20241218144933.png](/img/user/resimler/Pasted%20image%2020241218144933.png)

Web shell'e /discussion/ups/shell.php adresinden erişilebilir.

![Pasted image 20241218145019.png](/img/user/resimler/Pasted%20image%2020241218145019.png)****

Şimdi uygun bir shell'e geçelim. İlk olarak, nc.exe'yi barındıran lokal bir web sunucusu kurun.

![Pasted image 20241218145044.png](/img/user/resimler/Pasted%20image%2020241218145044.png)

Tarayıcıda aşağıdaki komutu vererek host'a indirin

![Pasted image 20241218145101.png](/img/user/resimler/Pasted%20image%2020241218145101.png)

Ardından, 1234 numaralı portta bir Netcat listener kurun ( nc -lvnp 1234 ) ve bir reverse shell göndermek için tarayıcıda aşağıdaki komutu verin.

![Pasted image 20241218145209.png](/img/user/resimler/Pasted%20image%2020241218145209.png)


![Pasted image 20241218145215.png](/img/user/resimler/Pasted%20image%2020241218145215.png)

DANTE-WS01 üzerindeki bir shell gerald olarak alınır. Bayrak masaüstünde bulunabilir.


![Pasted image 20241218145319.png](/img/user/resimler/Pasted%20image%2020241218145319.png)


### Rakamlarımı karşılaştırın

Host üzerinde kurulu yazılım kontrol edildiğinde, varsayılan olmayan Druva inSync programının kurulu olduğu görülür


![Pasted image 20241218145358.png](/img/user/resimler/Pasted%20image%2020241218145358.png)

Licence.txt, yazılımın sürümünün 6.6.3 olduğunu ortaya koymaktadır.

![Pasted image 20241218145435.png](/img/user/resimler/Pasted%20image%2020241218145435.png)

Bunu çevrimiçi olarak araştırmak, ifşa edilen sürümün lokal bir ayrıcalık yükseltme açığına karşı savunmasız olduğunu ortaya koymaktadır. [Exploiti](https://www.exploit-db.com/exploits/48505) host'a indirelim


![Pasted image 20241218145505.png](/img/user/resimler/Pasted%20image%2020241218145505.png)

Exploit kullanım talimatlarında gösterildiği gibi, gerald kullanıcısını administrators grubuna ekleyeceğiz

![Pasted image 20241218145548.png](/img/user/resimler/Pasted%20image%2020241218145548.png)


Bu başarılıdır ve gerald kullanıcısının artık administrators grubunun bir parçası olduğunu görebiliriz.


![Pasted image 20241218145605.png](/img/user/resimler/Pasted%20image%2020241218145605.png)

Ancak, UAC (Kullanıcı Hesabı Denetimi) nedeniyle Administrator masaüstündeki bayrağı okuyamıyoruz. Şimdi nc.exe dosyasını indirelim ve bize bir reverse shell göndermesi için exploit'i çalıştıralım.

![Pasted image 20241218145646.png](/img/user/resimler/Pasted%20image%2020241218145646.png)

Bu başarılı olur ve DANTE-WS01'de nt authority\system olarak bir shell alınır. [[Komutun ayrıntısı 1 \|Komutun ayrıntısı 1 ]]


![Pasted image 20241218145703.png](/img/user/resimler/Pasted%20image%2020241218145703.png)


Artık bayrağı okuyabiliyoruz.

![Pasted image 20241218145939.png](/img/user/resimler/Pasted%20image%2020241218145939.png)

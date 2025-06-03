---
{"dg-publish":true,"permalink":"/ctf/htb-serv-mon-ctf/"}
---


### Enumeration
![Pasted image 20241017225804.png](/img/user/resimler/Pasted%20image%2020241017225804.png)
![Pasted image 20241017225817.png](/img/user/resimler/Pasted%20image%2020241017225817.png)
Nmap çıktısı, FTP ve SSH'nin varsayılan portlarında ve HTTP'de (portlar
80 ve 8443. Kutudaki FTP'nin anonim girişe izin verdiğini not ediyoruz.


### FTP
FTP'ye bağlanıyoruz ve güvenlik duvarımız etkin olduğu için pasif aktarım modunu belirliyoruz. Bir Kullanıcı
dizini Nadine ve Nathan için alt dizinler içerir ve bu alt dizinlerin her biri bir metin dosyası içerir.

![Pasted image 20241017230352.png](/img/user/resimler/Pasted%20image%2020241017230352.png)

![Pasted image 20241017230411.png](/img/user/resimler/Pasted%20image%2020241017230411.png)

Confidential.txt, Nathan'ın masaüstünde bir Passwords.txt dosyasının varlığını ortaya çıkarır

Nathan,
Parolalar.txt dosyanızı Masaüstünüzde bıraktım. Lütfen bunu bir kez kaldırın
kendiniz düzenleyin ve güvenli klasöre geri yerleştirin.
Saygılarımla
Nadine

Notes to do.txt, yüklenen görevler için tamamlanmış ve tamamlanmamış görevler hakkında bilgi içerir
izleme uygulamaları.

![Pasted image 20241017230509.png](/img/user/resimler/Pasted%20image%2020241017230509.png)


HTTP/S

Bir tarayıcıda 80 numaralı portun incelenmesi, NVMS-1000 ağ gözetim yazılımı için bir oturum açma sayfası ortaya çıkarır. Varsayılan kimlik bilgileri admin / 123456 veya diğer yaygın kimlik bilgileri bize erişim sağlamaz.

![Pasted image 20241017230604.png](/img/user/resimler/Pasted%20image%2020241017230604.png)

8443 numaralı portun incelenmesi NSClient++ için bir oturum açma ekranı gösterir. Ortak parolalarla oturum açma girişimi de başarısız olur.
![Pasted image 20241017230642.png](/img/user/resimler/Pasted%20image%2020241017230642.png)


### Foothold

NVMS
![Pasted image 20241017230900.png](/img/user/resimler/Pasted%20image%2020241017230900.png)
![Pasted image 20241017230904.png](/img/user/resimler/Pasted%20image%2020241017230904.png)

Tarayıcıyı Burp'ü proxy olarak kullanacak şekilde yapılandırın, NVMS-1000 web sayfasını yenileyin ve isteği engelleyin. İsteği Burp'ün Repeater modülüne göndermek için CTRL + R tuşlarına basın. İlk satırdaki GET isteğini aşağıdaki payload ile değiştirin. Win.ini dosyası Windows kurulumlarında bulunur ve tüm kullanıcılar tarafından okunabilir ve bu nedenle bir LFI'yi doğrulamak için iyi bir hedeftir.

![Pasted image 20241017230944.png](/img/user/resimler/Pasted%20image%2020241017230944.png)

![Pasted image 20241017230957.png](/img/user/resimler/Pasted%20image%2020241017230957.png)

Güvenlik açığını doğrulayan win.ini dosyası görüntülenir. FTP sunucusundan gelen bilgileri kullanarak C:\Users\Nathan\Desktop\Passwords.txt dosyasını açmaya çalışalım.
![Pasted image 20241017231052.png](/img/user/resimler/Pasted%20image%2020241017231052.png)

Bu çalışır ve bir parola listesi döndürülür


SSH
SSH'a karşı bir parola spreyi deneyebiliriz. Yukarıdaki listeyi passwords.txt olarak kaydedin. FTP'nin incelenmesi Nadine ve Nathan kullanıcılarını ortaya çıkardı. Bunları administrator ile birlikte bir users.txt dosyasına ekleyin.

![Pasted image 20241017231146.png](/img/user/resimler/Pasted%20image%2020241017231146.png)

![Pasted image 20241017231152.png](/img/user/resimler/Pasted%20image%2020241017231152.png)

L1k3B1gBut7s@W0rk parolasının nadine kullanıcı adı için çalıştığı tespit edildi ve bu kullanıcı olarak bir komut shell'i açıldı. Ancak whoami /priv komutu ayrıcalıksız bir kullanıcı olduğunu ortaya çıkarır.

Kullanıcı bayrağı C:\Users\Nadine\Desktop\ içinde bulunabilir.


### Privilege Escalation

Enumeration

Dosya sisteminin numaralandırılması varsayılan olmayan C:\Program Files\NSClient++\ dizinini ortaya çıkarır. NSClient için .ini dosyası içinde bulunur. Hadi okuyalım.

![Pasted image 20241017231527.png](/img/user/resimler/Pasted%20image%2020241017231527.png)

Ayrıca sürümü şu komutla da belirleyebiliriz:

![Pasted image 20241017231547.png](/img/user/resimler/Pasted%20image%2020241017231547.png)

![Pasted image 20241017231553.png](/img/user/resimler/Pasted%20image%2020241017231553.png)

Web uygulamasının parolasını elde ettik ve localhost'un beyaz listedeki tek giriş olduğunu biliyoruz. NSClient'ı çevrimiçi araştırırken, özellik istismarını içeren bu ayrıcalık yükseltme tekniğine rastladık. Bu prosedürde bahsedilen yazılım sürümü 0.5.2.35'tir. Aşağıdaki komut, aynı yazılım sürümünün kutuda yüklü olduğunu ortaya koymaktadır.

NSClient, NT AUTHORITY\SYSTEM bağlamında çalıştırılır ve başarılı bir istismarın ardından, komut yürütme bu bağlamda gerçekleştirilir. Exploit'in çalışması için bir ön koşul, hizmetin yeniden başlatılmasıdır. NSCP hizmetini yeniden başlatma iznimiz olup olmadığını görmek için bu hizmetin izinlerini kontrol edelim. Rohn Edwards tarafından yazılan bu blog yazısı, PowerShell'de hizmet izinlerini nasıl alabileceğimizi göstermektedir. Komut dosyasını bellekte indirmek ve çalıştırmak için bir Msxml2.XMLHTTP COM nesnesi indirme cradle'ı kullanabiliriz.

Ancak, Service Control Manager'a erişimimiz reddedildi, bu nedenle hizmeti yeniden başlatma izinlerini varsaymamız gerekiyor.

![Pasted image 20241017231723.png](/img/user/resimler/Pasted%20image%2020241017231723.png)

![Pasted image 20241017231729.png](/img/user/resimler/Pasted%20image%2020241017231729.png)

PowerShell kullanarak temel servis özelliklerini inceleyelim.

![Pasted image 20241017231746.png](/img/user/resimler/Pasted%20image%2020241017231746.png)

![Pasted image 20241017231754.png](/img/user/resimler/Pasted%20image%2020241017231754.png)

CanStop parametresi true olarak ayarlanmıştır, bu da hizmeti durdurabileceğimiz anlamına gelir. Normalde düşük ayrıcalıklı bir kullanıcı hizmeti başlatamaz ancak bu durumda kullanıcıya başlatma izni verilmiştir.


Exploitation

Web uygulamasına localhost port 8443'ten erişmek için bir SSH tüneli kuralım
![Pasted image 20241017231840.png](/img/user/resimler/Pasted%20image%2020241017231840.png)
https://localhost:8443 adresine gidin ve giriş yapmak için ini dosyasında bulunan parolayı kullanın.

Payload'umuzu sistem üzerinde çalıştıracak harici bir script oluşturalım. Settings > External Scripts > Scripts'e gidin ve + Add new 'e tıklayın.

![Pasted image 20241017231920.png](/img/user/resimler/Pasted%20image%2020241017231920.png)

Ardından, Bölüm alanına /settings/external scripts/scripts/shell, Anahtar alanına komut ve Değer alanına C:\Temp\pwn.bat girin. Bat dosyası, komutları sistem olarak çalıştırmak için kullanılacaktır.

![Pasted image 20241017231939.png](/img/user/resimler/Pasted%20image%2020241017231939.png)

Komut dosyasını kaydedin ve Değişiklikler'e ve ardından Yapılandırmayı Kaydet'e tıklayın.

![Pasted image 20241017231954.png](/img/user/resimler/Pasted%20image%2020241017231954.png)

Son olarak, yeni oluşturulan komut dosyası girdisini yüklemek için NSCP hizmetini yeniden başlatalım.

![Pasted image 20241017232008.png](/img/user/resimler/Pasted%20image%2020241017232008.png)

![Pasted image 20241017232016.png](/img/user/resimler/Pasted%20image%2020241017232016.png)

Bir shell elde etmek için GreatSCT ile bir meterpreter payload oluşturalım.

![Pasted image 20241017232034.png](/img/user/resimler/Pasted%20image%2020241017232034.png)

![Pasted image 20241017232038.png](/img/user/resimler/Pasted%20image%2020241017232038.png)

DLL'yi indirmek için bir Python3 HTTP Sunucusu başlatın.

![Pasted image 20241017232102.png](/img/user/resimler/Pasted%20image%2020241017232102.png)

PowerShel kullanarak DLL'yi sunucudan indirin

![Pasted image 20241017232118.png](/img/user/resimler/Pasted%20image%2020241017232118.png)

Pwn.bat dosyasını oluşturmak için yükümüzü kutuya " echo " edelim. 

![Pasted image 20241017232139.png](/img/user/resimler/Pasted%20image%2020241017232139.png)

![Pasted image 20241017232145.png](/img/user/resimler/Pasted%20image%2020241017232145.png)

msfconsole'u açın ve oluşturulan RCE dosyasını belirtin

![Pasted image 20241017232159.png](/img/user/resimler/Pasted%20image%2020241017232159.png)

Ardından, http://127.0.0.1/8443 adresindeki konsola gidin, kod adını girin ve Çalıştır'a tıklayın.
![Pasted image 20241017232214.png](/img/user/resimler/Pasted%20image%2020241017232214.png)

Bir bağlantı alınır. Bazen ilk bağlantı kesilir. Bu durumda komutu tekrar çalıştırın ve kararlı olan ikinci bir bağlantı alınacaktır

![Pasted image 20241017232232.png](/img/user/resimler/Pasted%20image%2020241017232232.png)
Kök bayrak C:\Users\Administrator\Desktop konumunda bulunur.


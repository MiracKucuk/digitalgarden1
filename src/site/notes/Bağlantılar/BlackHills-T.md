---
{"dg-publish":true,"permalink":"/baglantilar/black-hills-t/"}
---



### Password Spraying ve RPCCLIENT ile Diğer Eğlenceler

UYARI: Bu blog yazısında atıfta bulunulan teknikler ve araçlar eski olabilir ve mevcut durumlar için geçerli olmayabilir. Bununla birlikte, bu blog girdisinin öğrenme ve muhtemelen modern araç ve tekniklere güncelleme veya entegre etme fırsatı olarak kullanılması için hala potansiyel vardır.

UYARI: Bu blog yazısında atıfta bulunulan teknikler ve araçlar eski olabilir ve mevcut durumlar için geçerli olmayabilir. Bununla birlikte, bu blog girdisinin öğrenme ve muhtemelen modern araç ve tekniklere güncelleme veya entegre etme fırsatı olarak kullanılması için hala potansiyel vardır.

Sızma testi topluluğundaki birçoğumuz, bir Windows kurumsal ortamında hedefli bir phishing kampanyası gerçekleştirdiğimiz ve Windows command line networking araçları dünyasına harika bir erişim sağladığımız senaryolara alışkınız. Shell'inizi alırsınız ve bir de bakmışsınız ki en sevdiğiniz numaralandırma komutlarını çalıştırmaya hazırsınız. Bunlar şunlar gibi şeyler:

- C:\> NET VIEW /DOMAIN
- C:\> NET GROUP “Domain Administrators” /DOMAIN

...vb. Metasploit post exploitation modüllerinin tüm zenginliğine ve Veil ve PowerShell Empire gibi çeşitli PowerShell araçlarının birçok harikasına sahip olduğunuzdan bahsetmiyorum bile.

Sahip olduğunuz tek şeyin, mevcut herhangi bir Windows sistemine backdoor shell erişimi olmayan bir iç ağda bulunan bir Linux host olduğu bir dünya hayal edin. Ağın geri kalanından etkin bir şekilde ayrıldığınız ve Ettercap gibi engelleme tekniklerini kullanarak yararlı ağ trafiğini bile yakalayamadığınız bir dünya hayal edin. Yakın zamanda benim için de durum böyleydi ve tek yapabildiğim kontrol ettiğim tek bir Linux hostuna SSH ile girmekti.

Bir süredir böyle bir durumla karşılaşmadığım için, Samba'nın harika dünyasını hatırlamadan önce bir an durakladım. Samba paketinde özellikle “rpcclient” ve arkadaşı “smbclient” olmak üzere iki mükemmel ve kullanışlı program bulunmaktadır. Ayrıca, “dig” adlı favori DNS yardımcı programımızı da unutmayalım.

İlk görevim, iç domain adının ne olabileceğine dair bilinçli tahminler yapmak için mevcut keşifleri kullanmaktı. Burada üzerinde düşünülmesi gereken birkaç farklı yöntem var, ancak ilk iş DNS bilgilerini belirlemek için dig ile oynamaktı. Domain controller adreslerini belirlemek için Windows global catalog kayıtlarına ve authoritative domain server kayıtlarına bakmayı deneyebilirim.   Örnekler aşağıdaki gibidir:

![Pasted image 20250106171338.png](/img/user/resimler/Pasted%20image%2020250106171338.png)

Bu bana yalnızca doğru “domain.corp” adını tahmin ettiğim ya da belirlediğim takdirde yanıt verecektir.

Şansıma, Nessus güvenlik açığı raporu verilerine erişimim vardı ve bazı hostlarda SMB NULL oturumlarına izin verildiğini tespit etmiştim. Verileri kazı sonuçlarımla eşleştirdim ve NULL oturumların aslında domain controller adreslerine karşılık geldiğini belirledim. Bir sonraki görevim, yalnızca benim kullanabildiğim rpcclient ile domain controller'lardan kullanıcı ve grup bilgilerini listelemeye çalışmaktı. “man” sayfasını kullanarak rpcclient'ın gerçekten de aşağıdaki gibi anonim bir bağlama gerçekleştirebildiğini hızlıca belirledim.

![Pasted image 20250106171536.png](/img/user/resimler/Pasted%20image%2020250106171536.png)

...burada 10.10.10.10, anonim olarak bağlanabileceğim domain controller'ın seçilen adresiydi. Bu komut çalıştırıldıktan sonra, rpcclient size en mükemmel “rpcclient> ” komut istemini verecektir. Bu noktada, anonim oturumları kullanabiliyorsanız, araç içinde çok yararlı bazı komutlar vardır. 

1. Enumerate Domain Users

![Pasted image 20250106171649.png](/img/user/resimler/Pasted%20image%2020250106171649.png)

2. Enumerate Domain Groups

![Pasted image 20250106171737.png](/img/user/resimler/Pasted%20image%2020250106171737.png)

3. Grup Bilgilerini ve Grup Üyeliğini Sorgulama

![Pasted image 20250106171805.png](/img/user/resimler/Pasted%20image%2020250106171805.png)

4. Belirli Kullanıcı Bilgilerini (bilgisayarlar dahil) RID'ye göre sorgulayın.

![Pasted image 20250106171857.png](/img/user/resimler/Pasted%20image%2020250106171857.png)

Bu temel komutlarla çalışırken, Windows domain kullanıcı ve grup bilgilerini oldukça kapsamlı bir şekilde inceleyebildim.

Sızma testi sırasında sıklıkla kullanılan bir başka teknik de “ password spraying” olarak adlandırılır. Bu özellikle etkili bir tekniktir, domain kullanıcılarının bir listesi ve çok yaygın parola kullanımı bilgisi verildiğinde, test uzmanı listedeki her kullanıcı için bir oturum açma gerçekleştirmeye çalışır. Denenecek parola listesini kasıtlı olarak az sayıda tuttuğunuzda bu teknik çok etkilidir. Aslında, hesapları gerçekten kilitlemek istememenizin tek nedeni spreyleme denemesi başına tek bir parola tavsiye edilir.

Password spraying yapmadan önce, Windows dünyasında “NET ACCOUNTS /DOMAIN” gibi bir komut kullanarak Windows domain password policy'sini belirlemek çok faydalıdır. Ancak elimizde bir Windows shell'i olmadığı için rpcclient bize aşağıdaki seçenekleri sunar.

![Pasted image 20250106172042.png](/img/user/resimler/Pasted%20image%2020250106172042.png)

En azından parola uzunluğu ile ilgili önemli bilgileri belirleyebiliyoruz. Bunu yazdıktan sonra, muhtemelen parola özelliklerini nasıl çözeceğimi ve bunları uygun bilgilerle nasıl eşleştireceğimi çözeceğim, ancak bu görevi henüz yapmadım.

Bir password sprey saldırısı gerçekleştirmek için, bir sonraki adım yaygın bir şifre seçmek (“Autumn2015” gibi) ve rpcclient kullanarak nasıl sprey yapacağımıza dair tekniğimizi çalışmaktır. Kullanışlı bir şekilde, rpcclient komut satırında bazı komutları belirtmemize izin verir ki bu çok kullanışlıdır. Aşağıdaki iki örnek başarılı bir oturum açma ile başarısız bir oturum açmayı göstermektedir. (“bbb” parolası doğru oturum açma işlemidir).

![Pasted image 20250106172140.png](/img/user/resimler/Pasted%20image%2020250106172140.png)


Bu örneklerde, rpcclient'a özellikle iki komut çalıştırmasını söyledik, bunlar “getusername” ve ardından client'tan çıkmak için “quit” komutlarıydı. Artık bir password spraying saldırısı gerçekleştirmek için tüm malzemelere sahibiz. İhtiyacımız olan tek şey bir bourne//bash shell döngüsü ve yarışa hazırız. “enumdomusers” çıktısının 'domain-users.txt' dosyasında olduğu göz önüne alındığında, spreyleme için basit bir shell script veya komut satırı örneği aşağıdaki gibi olacaktır.

![Pasted image 20250106172249.png](/img/user/resimler/Pasted%20image%2020250106172249.png)

Çıktıda “ Authority” stringini göründüğünü gördüğünüzde başarılı olduğunuzu anlarsınız. Her kullanıcı için başarı sağlanamaması “NT_STATUS_LOGON_FAILURE” mesajı olacaktır.

“ACCOUNT_LOCKED” hatası almaya başlarsanız, spreyinizi derhal durdurmalısınız çünkü muhtemelen kısa bir süre içinde çok fazla kez spreylemişsinizdir.

Bir kimlik bilgisine erişim sağladığınızı varsayarsak, yapabileceğiniz diğer güzel şeylerden biri de smbclient programını kullanarak SYSVOL'u keşfetmektir. Sözdizimi aşağıdaki gibidir.


![Pasted image 20250106172341.png](/img/user/resimler/Pasted%20image%2020250106172341.png)


UNIX Samba paketine ve özellikle de bu araçlara aşina olmanızı şiddetle tavsiye ederim. Geçtiğimiz hafta tam anlamıyla hayatımı kurtardılar ve siz de gelecekte bu eğlenceli araçlara ihtiyaç duyabilirsiniz.




---
{"dg-publish":true,"permalink":"/active-directory/password-attacks-new/","created":"2025-06-08T17:18:59.884+03:00","updated":"2025-06-08T18:33:44.216+03:00"}
---


# Introduction

`Confidentiality`, `integrity` ve `availability` her bilgi güvenliği uygulayıcısının sorumluluklarının merkezinde yer alır. Bunlar arasında bir denge kurmadan işletmelerimizin güvenliğini sağlayamayız. Bu denge, ortamdaki her dosya, object ve host için denetim ve muhasebe yaparak; kullanıcıların bu kaynaklara erişmek için uygun permission'lara (authorization) sahip olduğunu doğrulayarak ve erişim izni vermeden önce her kullanıcının kimliğini doğrulayarak (`authentication`) korunur. Çoğu ihlal, bu üç ilkeden birinin ihlal edilmesine kadar izlenebilir. Bu modül, çeşitli işletim sistemleri, uygulamalar ve şifreleme yöntemlerinde kullanıcı parolalarını tehlikeye atarak kimlik doğrulama prensibine saldırmaya ve atlamaya odaklanmaktadır. Heyecan verici kısma, yani parolalara saldırmaya geçmeden önce, kimlik doğrulama ve bileşenlerini tartışmak için biraz zaman ayıralım.

## Authentication

Authentication, özünde, bir doğrulama mekanizmasına dört faktörün bir kombinasyonunu sunarak kimliğinizin doğrulanmasıdır. Bunlar

* `Bildiğiniz bir şey`: password, passcode, pin, vb.
* `Sahip olduğunuz bir şey`: ID Kart, güvenlik anahtarı veya diğer MFA araçları
* `Olduğunuz bir şey`: fiziksel benliğiniz, kullanıcı adınız, e-posta adresiniz veya diğer tanımlayıcılar
* `Olduğunuz bir yer`: coğrafi konum, IP adresi vb.

Proses, bu kimlik doğrulama faktörlerinden herhangi birini veya tümünü gerektirebilir. Bu yöntemler, erişilen bilgi veya sistemlerin ciddiyetine ve ne kadar korumaya ihtiyaç duyduklarına göre belirlenecektir. Örneğin, doktorların hasta verilerini giren veya depolayan terminallere erişmek için genellikle bir pin kodu veya şifre ile eşleştirilmiş bir Common Access Card (CAC) kullanmaları gerekir. Kuruluşun güvenlik duruşunun olgunluğuna bağlı olarak, her üç türe de ihtiyaç duyabilirler (Örneğin, bir kimlik doğrulama uygulamasından bir CAC, şifre ve pin).

Bunun bir başka basit örneği de e-posta adresimize erişimdir. Bu durumda bilgi kanıtı, e-posta adresinin kendisinin ve ilişkili şifrenin bilinmesi olacaktır. Örneğin, `2FA`'lı bir cep telefonu kullanılabilir. Üçüncü bir husus da rol oynayabilir: parmak izi veya yüz tanıma gibi biyometrik tanıma yoluyla kullanıcının varlığı.


## The use of passwords

En yaygın ve yaygın olarak kullanılan kimlik doğrulama yöntemi hala parola kullanımıdır. Peki ama parola nedir? Şifre ya da parola genel olarak kimlik doğrulama için bir string içinde harf, rakam ve sembollerin bir kombinasyonu olarak tanımlanabilir. Örneğin, şifrelerle çalışırsak ve yalnızca büyük harf ve rakamlardan oluşan standart 8 basamaklı bir şifre alırsak, toplam `36⁸` (`208.827.064.576`) olası şifre elde ederiz.

Gerçekçi olmak gerekirse, bunların bir kombinasyonu olması gerekmez. Bir şarkıdan veya şiirden bir söz, bir kitaptan bir satır, hatırlayabileceğiniz bir cümle veya hatta “TreeDogEvilElephant” gibi rastgele bir araya getirilmiş kelimeler olabilir. Önemli olan, kuruluşunuz tarafından uygulanan güvenlik standartlarını karşılaması veya aşmasıdır. Kimlik oluşturmak için birden fazla katman kullanmak tüm kimlik doğrulama sürecini karmaşık ve maliyetli hale getirebilir. Kimlik doğrulama sürecine karmaşıklık eklemek, bir kişinin tipik bir iş günü boyunca sahip olabileceği stres ve iş yükünü artırabilecek daha fazla çaba yaratır. Karmaşık sistemler genellikle uygunsuz manuel süreçler veya etkileşimi ve kullanıcı deneyimini (`UX`) önemli ölçüde karmaşıklaştırabilecek ek adımlar gerektirebilir. Bir çevrimiçi mağazadan alışveriş yapma sürecini düşünün. Mağazanın web sitesinde bir hesap oluşturmak, kimlik doğrulama ve ödeme süreçlerini, her satın alma işlemi yapmak istediğinizde kişisel bilgilerinizi manuel olarak girmekten çok daha hızlı hale getirebilir. Bu nedenle, bir hesabı güvence altına almak için kullanıcı adı ve şifre kullanmak, bu kolaylık ve güvenlik dengesini göz önünde bulundurarak tekrar tekrar göreceğimiz en yaygın kimlik doğrulama yöntemidir.

Google ve Harris Poll tarafından 2019 yılında yapılan bir [anket](https://web.archive.org/web/20201115110047/https://storage.googleapis.com/gweb-uniblog-publish-prod/documents/PasswordCheckup-HarrisPoll-InfographicFINAL.pdf), `Amerikalıların %24`'ünün `123456`, `qwerty` ve `password` gibi şifreler kullandığını ortaya koyuyor. O dönemde `Amerikalıların yalnızca %15`'i password manager kullanıyordu. Ayrıca `%22`'sinin kendi adını, `%33`'ünün ise evcil hayvanının ya da çocuğunun adını kullandığı belirtiliyor. Bir diğer kritik istatistik ise birden fazla hesapta şifrenin yeniden kullanım oranının `%66` olması. Bu istatistiğe göre tüm Amerikalıların `%66`'sı aynı şifreyi birden fazla platform için kullanmıştır. Dolayısıyla, bir şifreyi edindiğimizde ya da tahmin ettiğimizde, bu şifreyi kullanıcı kimliği (“username” ya da “email address”) ile başka platformlarda kimlik doğrulaması yapmak için kullanma ihtimalimiz `%66`'dır. Elbette bunun için kullanıcının kullanıcı kimliğini tahmin edebilmemiz gerekir ki çoğu durumda bunu yapmak zor değildir.

Panda Security tarafından 2025 yılında derlenen [istatistikler](https://web.archive.org/web/20250405101232/https://www.pandasecurity.com/en/mediacenter/password-statistics/), bazı iyileşme işaretleriyle birlikte bu eğilimlerin benzer kaldığını göstermektedir. Veri ihlallerinde 4,5 milyon kez görülen `123456` hala en yaygın paroladır ve en az `%23`'ü parolaları üç veya daha fazla hesapta tekrar kullanmaktadır. Bununla birlikte, Amerikalıların `%36`'sı password manager'ları benimsemiştir ki bu oran 6 yıl önceki oranın iki katından fazladır.

Google'ın anketinin anlaşılması biraz daha zor olan bir yönü, Amerikalıların yalnızca %45'inin bir veri ihlalinden sonra şifrelerini değiştirecek olmasıdır. Bu da `%55'inin şifreleri sızdırılmış` olsa bile hala şifrelerini sakladıkları anlamına geliyor. E-posta adreslerimizden birinin çeşitli veri ihlallerinden etkilenip etkilenmediğini de kontrol edebiliriz. Bunun için en iyi bilinen kaynaklardan biri HaveIBeenPwned. [HaveIBeenPwned](https://haveibeenpwned.com/) web sitesine bir e-posta adresi giriyoruz ve e-posta adresinin bildirilen herhangi bir veri ihlalinden etkilenip etkilenmediğini veritabanında kontrol ediyor. Eğer durum böyleyse, e-posta adresimizin göründüğü tüm ihlallerin bir listesini göreceğiz.

Parolanın ne olduğunu, nasıl kullanıldığını ve genel güvenlik ilkelerini tanımladığımıza göre, şimdi parolaları ve diğer kimlik bilgilerini nasıl sakladığımıza bakalım.


# Introduction to Password Cracking

Parolalar, bir saldırganın eline geçmeleri durumunda bir miktar koruma sağlamak amacıyla saklanırken genellikle hashlenir. Hashing, rastgele sayıda input baytını (tipik olarak) sabit boyutlu bir çıktıya dönüştüren matematiksel bir fonksiyondur; hash fonksiyonlarının yaygın örnekleri `MD5` ve `SHA-256`'dır.

Örneğin `Soccer06!` parolasını ele alalım. İlgili `MD5` ve `SHA-256` hash'leri aşağıdaki komutlarla oluşturulabilir:

```shell-session
bmdyy@htb:~$ echo -n Soccer06! | md5sum
40291c1d19ee11a7df8495c4cccefdfa  -

bmdyy@htb:~$ echo -n Soccer06! | sha256sum
a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -
```

Hash fonksiyonları tek yönde çalışmak üzere tasarlanmıştır. Bu, orijinal parolanın ne olduğunu yalnızca hash'e dayanarak bulmanın mümkün olmaması gerektiği anlamına gelir. Saldırganlar bunu yapmaya çalıştığında, buna password cracking denir. Yaygın teknikler rainbow tablolarını kullanmak, sözlük saldırıları gerçekleştirmek ve genellikle son çare olarak brute-force saldırıları gerçekleştirmektir.

## Rainbow tables

Rainbow tabloları, belirli bir hash fonksiyonu için input ve output değerlerinin önceden derlenmiş büyük eşlemeleridir. Bunlar, karşılık gelen hash zaten eşlenmişse şifreyi çok hızlı bir şekilde tanımlamak için kullanılabilir.

|Password|MD5 Hash|
|---|---|
|123456|e10adc3949ba59abbe56e057f20f883e|
|12345|827ccb0eea8a706c4c34a16891f84e7b|
|123456789|25f9e794323b453885f5181f1b624d0b|
|password|5f4dcc3b5aa765d61d8327deb882cf99|
|iloveyou|f25a2fc72690b780b2a14e140ef6a9e0|
|princess|8afa847f50a716e64932d995c8e7435a|
|1234567|fcea920f7412b5da7be0cf42b8c93759|
|rockyou|f806fc5a2a0d5ba2471600758452799c|
|12345678|25d55ad283aa400af464c76d713c07ad|
|abc123|e99a18c428cb38d5f260853678922e03|
|...SNIP...|...SNIP...|

Rainbow tabloları çok güçlü bir saldırı olduğu için `salt` kullanılır. Kriptografik terimlerle `salt`, bir parolaya hash edilmeden önce eklenen rastgele bir bayt dizisidir. Etkiyi en üst düzeye çıkarmak için saltlar, örneğin bir veritabanında depolanan tüm parolalar için tekrar kullanılmamalıdır. Örneğin, `Th1sIsTh3S@lt_` salt'ı aynı parolaya eklenirse, MD5 hash'i artık aşağıdaki gibi olacaktır:

```shell-session
C4RT3L@htb[/htb]$ echo -n Th1sIsTh3S@lt_Soccer06! | md5sum

90a10ba83c04e7996bc53373170b5474  -
```

Salt gizli bir değer değildir - bir sistem bir kimlik doğrulama isteğini kontrol etmeye gittiğinde, parola hash'inin eşleşip eşleşmediğini kontrol edebilmek için hangi salt'ın kullanıldığını bilmesi gerekir. Bu nedenle, salt'lar genellikle ilgili hash'lerin başına eklenir. Bu tekniğin rainbow tablolarına karşı çalışmasının nedeni, doğru parola eşlenmiş olsa bile, salt ve parola kombinasyonunun muhtemelen eşlenmemiş olmasıdır (özellikle salt yazdırılamayan karakterler içeriyorsa). Rainbow tablolarını tekrar etkili hale getirmek için, bir saldırganın olası her salt'ı hesaba katmak için eşlemelerini güncellemesi gerekir. Sadece `tek bir bayttan` oluşan bir salt, daha önceki `15` milyar entry'nin `3.84 trilyon` olması gerektiği anlamına gelir (`256` faktörü).

## Brute-force attack

Brute-force saldırısı, doğru parola bulunana kadar harf, rakam ve sembollerin olası her kombinasyonunun denenmesini içerir. Açıkçası, bu çok uzun zaman alabilir - özellikle uzun parolalar için - ancak daha kısa parolalar (<9 karakter) uygun hedeflerdir, hatta tüketici donanımlarında bile. Brute-forcing `%100 etkili` olan tek parola kırma tekniğidir - yani yeterli zaman verildiğinde bu teknikle her parola kırılabilir. Bununla birlikte, daha güçlü parolalar için ne kadar zaman aldığı nedeniyle neredeyse hiç kullanılmaz ve genellikle çok daha etkili `maskeleme` saldırıları ile değiştirilir. Bu konuyu önümüzdeki birkaç bölümde ele alacağız.

|Brute-force attempt|MD5 Hash|
|---|---|
|...SNIP...|...SNIP...|
|Sxejd|2cdc813ef26e6d20c854adb107279338|
|Sxeje|7703349a1f943f9da6d1dfcda51f3b63|
|Sxejf|db914f10854b97946046eabab2287178|
|Sxejg|c0ceb70c0e0f2c3da94e75ae946f29dc|
|Sxejh|4dca0d2b706e9344985d48f95e646ce8|
|Sxeji|66b5fa128df895d50b2d70353a7968a7|
|Sxejj|dd7097ba514c136caac321e321b1b5ca|
|Sxejk|c0eb1193e62a7a57dec2fafd4177f7d9|
|Sxejl|5ad8e1282437da255b866d22339d1b53|
|Sxejm|c4b95c1fe6d2a4f22620efd54c066664|
|...SNIP...|...SNIP...|

Not: Brute-forcing hızları büyük ölçüde hash algoritmasına ve kullanılan donanıma bağlıdır. Tipik bir şirket dizüstü bilgisayarında, hashcat gibi bir araç MD5'e saldırırken saniyede beş milyondan fazla parola tahmin edebilirken, aynı zamanda bir DCC2 hash'ini hedeflerken saniyede yalnızca on bin parolayı yönetebilir.

[[DCC2 Açıklama\|DCC2 Açıklama]]


## Dictionary attack

Wordlist saldırısı olarak da bilinen sözlük saldırısı, özellikle sızma testi uzmanlarının genellikle yaptığı gibi zaman kısıtlamaları altında çalışıldığında, parolaları kırmak için en etkili tekniklerden biridir. Olası her karakter kombinasyonunu denemek yerine, istatistiksel olarak olası şifreleri içeren bir liste kullanılır. Parola kırma için iyi bilinen kelime listeleri [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) ve [SecLists](https://github.com/danielmiessler/SecLists)'te bulunanlardır.

```shell-session
C4RT3L@htb[/htb]$ head --lines=20 /usr/share/wordlists/rockyou.txt 

123456
12345
123456789
password
iloveyou
princess
1234567
rockyou
12345678
abc123
nicole
daniel
babygirl
monkey
lovely
jessica
654321
michael
ashley
qwerty
```

Not: `rockyou.txt`, 2009 yılında `RockYou` web sitesi hacklendiğinde sızdırılan `14 milyondan` fazla gerçek şifrenin bir listesidir. Şaşırtıcı bir şekilde, şirket tüm kullanıcı şifrelerini şifrelenmemiş olarak saklama kararı almıştır!

---

Soru : `Academy#2025` için SHA1 hash nedir?

Cevap : `750fe4b402dc9f91cedf09b652543cd85406be8c`

Not : `echo` komutunda `-n` koymazsan sonunda `\n` (newline) da hash'e dahil olur → yanlış sonuç.

```
┌─[eu-academy-4]─[10.10.15.98]─[htb-ac-961525@htb-wnlntd3usm]─[~]
└──╼ [★]$ echo -n Academy#2025 | sha1sum
750fe4b402dc9f91cedf09b652543cd85406be8c  -
```

Yanlış : 

```
[★]$ echo Academy#2025 | sha1sum
5814e4e6a05e3b2ae55fa4043117cd4434c40f3c  -
```

---

# Introduction to John The Ripper

[John the Ripper](https://github.com/openwall/john) (diğer adıyla `JtR` diğer adıyla `john`), brute force ve dictionary gibi çeşitli saldırılar yoluyla şifreleri kırmak için kullanılan iyi bilinen bir penetrasyon testi aracıdır. Başlangıçta UNIX tabanlı sistemler için geliştirilmiş açık kaynaklı bir yazılımdır ve ilk olarak 1996 yılında piyasaya sürülmüştür. Çeşitli yetenekleri nedeniyle güvenlik endüstrisinin temellerinden biri haline gelmiştir. Performans optimizasyonlarına, çok dilli kelime listeleri gibi ek özelliklere ve 64 bit mimariler için desteğe sahip olduğu için bizim kullanımlarımız için “`jumbo`” varyantı önerilir. Bu sürüm şifreleri daha yüksek doğruluk ve hız ile kırabilmektedir. JtR ile birlikte, farklı dosya türlerini ve hash'leri JtR tarafından kullanılabilecek formatlara dönüştürmek için çeşitli araçlar sunulmaktadır. Ayrıca, yazılım güncel güvenlik trendlerine ve teknolojilerine ayak uydurmak için düzenli olarak güncellenmektedir.


## Cracking modes

#### Single crack mode

Single crack modu, Linux kimlik bilgilerini hedeflerken en kullanışlı olan rule-based cracking tekniğidir. Kurbanın username, home directory name ve [GECOS](https://en.wikipedia.org/wiki/Gecos_field) değerlerine ( full name, room number, phone number, etc.) dayalı parola adayları oluşturur. Bu stringler, parolalarda görülen yaygın string değişikliklerini uygulayan geniş bir kural setine karşı çalıştırılır (örneğin, gerçek adı Bob Smith olan bir kullanıcı parola olarak Smith1 kullanabilir).

[[GECOS Açıklama\|GECOS Açıklama]]

Not: Linux kimlik doğrulama süreci ve kırma kuralları daha sonraki bölümlerde ayrıntılı olarak ele alınacaktır. Aşağıdaki örnek gösterim amacıyla basitleştirilmiştir.

Saldırganlar olarak aşağıdaki içeriğe sahip passwd dosyasıyla karşılaştığımızı hayal edin:

```
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```

Dosyanın içeriğine dayanarak, kurbanın kullanıcı adının `r0lf`, gerçek adının `Rolf Sebastian` ve home dizininin `/home/r0lf` olduğu sonucuna varılabilir. Single crack modu bu bilgileri kullanarak aday şifreler oluşturacak ve bunları hash'e karşı test edecektir. Saldırıyı aşağıdaki komutla çalıştırabiliriz:

```shell-session
C4RT3L@htb[/htb]$ john --single passwd

Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[...SNIP...]        (r0lf)     
1g 0:00:00:00 DONE 1/3 (2025-04-10 07:47) 12.50g/s 5400p/s 5400c/s 5400C/s NAITSABESFL0R..rSebastiannaitsabeSr
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Bu durumda, parola hash'i başarıyla kırılmıştır.

#### Wordlist mode

Wordlist modu, parolaları bir dictionary saldırısı ile kırmak için kullanılır, yani verilen bir wordlist'teki tüm parolaları parola hash'ine karşı dener. Komutun temel sözdizimi aşağıdaki gibidir:

```shell-session
C4RT3L@htb[/htb]$ john --wordlist=<wordlist_file> <hash_file>
```

Parola hash'lerini kırmak için kullanılan wordlist dosyası (veya dosyaları), her satırda bir sözcük olacak şekilde düz metin biçiminde olmalıdır. Birden fazla kelime listesi virgülle ayrılarak belirtilebilir. Özel veya built-in kurallar `--rules` argümanı kullanılarak belirtilebilir. Bunlar, sayı ekleme, harfleri büyük harf yapma ve özel karakterler ekleme gibi dönüşümleri kullanarak aday parolalar oluşturmak için uygulanabilir.

#### Incremental mode

`Incremental mode`, istatistiksel bir modele ([Markov zincirleri](https://en.wikipedia.org/wiki/Markov_chain)) dayalı olarak aday parolalar üreten güçlü, brute-force tarzı bir password cracking modudur. Belirli bir karakter seti tarafından tanımlanan tüm karakter kombinasyonlarını test etmek ve eğitim verilerine dayanarak daha olası parolalara öncelik vermek üzere tasarlanmıştır.

Bu mod en kapsamlı olanıdır, ancak aynı zamanda en çok zaman alan moddur. Parola tahminlerini dinamik olarak üretir ve kelime listesi modunun aksine önceden tanımlanmış bir kelime listesine dayanmaz. Tamamen rastgele brute force saldırılarının aksine, Incremental modu eğitimli tahminler yapmak için istatistiksel bir model kullanır, bu da naif brute-force saldırılarından çok daha verimli bir yaklaşımla sonuçlanır.

Temel syntax şöyledir:

```shell-session
C4RT3L@htb[/htb]$ john --incremental <hash_file>
```

Varsayılan olarak JtR, yapılandırma dosyasında (`john.conf`) belirtilen, karakter kümelerini ve parola uzunluklarını tanımlayan önceden tanımlanmış incremental modları kullanır. Bunları özelleştirebilir veya özel karakterler veya belirli kalıplar kullanan parolaları hedeflemek için kendi parolanızı tanımlayabilirsiniz.

```shell-session
C4RT3L@htb[/htb]$ grep '# Incremental modes' -A 100 /etc/john/john.conf

# Incremental modes

# This is for one-off uses (make your own custom.chr).
# A charset can now also be named directly from command-line, so no config
# entry needed: --incremental=whatever.chr
[Incremental:Custom]
File = $JOHN/custom.chr
MinLen = 0

# The theoretical CharCount is 211, we've got 196.
[Incremental:UTF8]
File = $JOHN/utf8.chr
MinLen = 0
CharCount = 196

# This is CP1252, a super-set of ISO-8859-1.
# The theoretical CharCount is 219, we've got 203.
[Incremental:Latin1]
File = $JOHN/latin1.chr
MinLen = 0
CharCount = 203

[Incremental:ASCII]
File = $JOHN/ascii.chr
MinLen = 0
MaxLen = 13
CharCount = 95

...SNIP...
```

Not: Bu mod, özellikle uzun veya karmaşık parolalar için yoğun kaynak gerektirebilir ve yavaş olabilir. Karakter kümesini ve uzunluğunu özelleştirmek performansı artırabilir ve saldırıya odaklanabilir.


## Identifying hash formats

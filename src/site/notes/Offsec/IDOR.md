---
{"dg-publish":true,"permalink":"/offsec/idor/"}
---

## Insecure Direct Object Referencing (IDOR) – Güvensiz Doğrudan Obje Referansı

Bu Öğrenim Modülünde aşağıdaki Öğrenim Birimlerini inceleyeceğiz:

- IDOR'a Giriş
    
- IDOR'un Sandbox Uygulamasında Sömürülmesi
    
- Vaka Çalışması: OpenEMR
    

**Insecure Direct Object Referencing (IDOR)**, bir web uygulamasının dahili bir objeye referans veren bir kaynağı doğrudan ifşa etmesi durumunda meydana gelir. Bu genellikle, web uygulamalarının dosyalar, kullanıcılar, veritabanı bilgileri veya bazı durumlarda doğrudan yinelemeli objeler gibi veri türlerini işlemesi anlamına gelir.

Bir **IDOR bulgusu**, bir uygulamanın verilerinin gizliliğini etkiler. Böyle bir bulgunun ciddiyeti, ifşa edilen verinin hassasiyetine bağlıdır. Örneğin, tıbbi kayıtlar veya fikri mülkiyet gibi gizli bilgilerin açığa çıkmasına yol açan bir IDOR güvenlik açığı, kritik bir zafiyet olarak değerlendirilebilir.

Zafiyet tarayıcıları her zaman IDOR güvenlik açıklarını tespit edemeyebilir. Bu tarayıcılar kredi kartları gibi belirli desenlere sahip hassas verileri tanıyabilir. Ancak, tanımlı bir deseni olmayan diğer veri türleri için bir **penetrasyon test uzmanının manuel inceleme yapması** gerekir.

Sonraki Öğrenim Biriminde, modern web uygulamalarında karşılaşılabilecek farklı **IDOR türlerini** tartışacağız.

---

## IDOR'a Giriş

Bu Öğrenim Birimi aşağıdaki öğrenim hedeflerini kapsamaktadır:

- **Statik Dosya IDOR** bulgularını anlamayı öğrenmek
    
- **Veritabanı Obje Referanslaması (ID tabanlı) IDOR** hakkında bilgi edinmek
    

Bu Öğrenim Biriminde, IDOR ile ilgili çeşitli Öğrenim Modülleri ve kavramları inceleyeceğiz. Özellikle, **IDOR kaynaklı güvenlik açıklarının teorisini** öğreneceğiz.

**Statik Dosya IDOR** bulgusu, hedef web uygulamasının bir veri dosyasının objesine doğrudan bağlı **zayıf biçimlendirilmiş tanımlayıcı numaralar** kullandığı durumlarda ortaya çıkar. Bu tür bir zafiyetin başarıyla sömürülmesi, dosya içeriğinin ifşa edilmesine yol açabilir.

Şimdi IDOR Sandbox uygulamamızda bulunan şu endpoint'i inceleyelim:  
`http://idor-sandbox/docs/?f=1.txt`

![Pasted image 20250523000008.png](/img/user/resimler/Pasted%20image%2020250523000008.png)


Jason’dan Jerry’ye gönderilen bir mesaj buluyoruz; bu mesajda, dinlenme odasına yeni bir kahve makinesi gerektiği belirtiliyor. Bu mesaj, endpoint'in sonunda referans verilen metin dosyası (1.txt) sayesinde görüntüleniyor.

Bu fikri bir adım ileri taşıyarak, hedef makinada başka metin belgelerinin de mevcut olabileceğini tahmin edebiliriz.

Örneğin, `f` parametresinin değerini **"2.txt"** olarak artırarak şu endpoint'i elde ederiz:  
`http://idor-sandbox/docs/?f=2.txt`

Bu şekilde **ID'lerin artırılabilir veya azaltılabilir olması**, klasik bir **Insecure Direct Object Referencing (IDOR)** göstergesidir.

Sıradaki adımda, sahada (gerçek hayatta) karşılaşabileceğimiz başka bir **statik dosya IDOR** türünü keşfedeceğiz. Bu tür IDOR, URL içerisinde, NodeJS ile ya da "Routing" olarak bilinen parametreler aracılığıyla meydana gelebilir.



### Routing Nedir?

**Routing (Yönlendirme)**, bir URL’de hangi tür bilgilerin alınacağını belirten özel segmentlerin konumlandırılmasıdır (örneğin: dosyalar, belgeler, bilet numaraları, vb). Bir **routed parametre** URL içinde belirtildiğinde, iki nokta (`:`) gibi ayraçlarla ayrılmış segmentler arka uçta çalışan web teknolojileri tarafından işlenir. Bizim ilgilendiğimiz kısım, URL segmentlerinde bulunan ve **referans verilebilen, yinelemeye uygun veya brute-force edilebilecek** değerlerdir.

Aşağıdaki URI seti, **ExpressJS** tarafından işlenmek üzere tasarlanmış yönlendirilmiş endpoint'leri ve örnek URI'leri göstermektedir. Eğik çizgiler (`/`) statik dosya yollarını temsil ederken, iki nokta (`:`) ile başlayan her bölüm dinamik tanımlayıcı olarak adlandırılır.

```plaintext
/users/:userIdent/documents/:pdfFile  
/trains/:from-:to  
/book/:year-:author  
```

**Listing 1 - Örnek Routing**

Yukarıdaki listede çeşitli segmentler bulunmaktadır:

- İlk örnek, kullanıcı kimliğini ve o kullanıcıya ait bir belgeyi içeriyor.
    
- İkinci örnek, bir tren bileti rezervasyonunu, "nereden" ve "nereye" bilgileriyle birlikte gösteriyor.
    
- Son örnek ise bir kitabı, yıl ve yazar bilgisiyle işliyor.
    

Şimdi bu segmentlerin gerçek dünyada nasıl görünebileceğini daha açık göstermek için örnek değerler kullanalım:

```plaintext
/users/18293017/documents/file-15         (PDF Dosyası Alındı)  
/trains/LVIV-ODESSA                       (Bilet Dosyası Alındı)  
/book/1996-GeorgeRRMartin                 (Kitap Bilgisi Alındı)  
```

**Listing 2 - Routed URI Örnekleri**

Bu URI’ler, URL’yi tamamlayan parametre segmentlerini göstermektedir. Bu parametreler kullanılarak hedef web sunucusundan çeşitli bilgi türleri sızdırılabilir.



### Veritabanı Obje Referanslaması ("ID-Based" IDOR)

Sıradaki bölümde **Veritabanı Obje Referanslaması** veya **ID-tabanlı IDOR**'u ele alacağız. Bu tür IDOR, bir endpoint'in bir **veritabanından obje referanslaması yaparak** bir web sayfasında veri görüntülemesi, ancak bunu **güvensiz bir şekilde yapması** anlamına gelir.

Bu tür bulgular için **artışlı kontroller** yapmak çok önemlidir. Aksi halde, güvenli tutulması gereken bir veritabanından bilgi sızdırabilecek bir zafiyeti gözden kaçırabiliriz.

Web uygulamasının arka ucunda **SQL sorguları** çalıştırıldığında, sayısal parametre değerlerinden oluşan artımlı bir dizi görebiliriz. Bu değerleri sırasıyla değiştirerek veritabanındaki objelere ait verileri dışarı sızdırabiliriz.

Bu tür bir durumu daha iyi anlamak için aşağıdaki endpoint’i inceleyelim:

```plaintext
http://idor-sandbox:80/customerPage/?custId=1
```

Bu URL, `custId` adında sayısal bir değere sahip bir **query string** içeriyor. Burada ID-tabanlı obje referanslamasının yapıldığını anlamak için bu numarayı artırıp her bir iterasyonda dönen yanıtın boyutunu (byte cinsinden) ya da tarayıcıdaki içeriği gözlemlemeliyiz.



### Unique Identifier (UID) Kavramı

UID, sayısal veya alfasayısal olabilir. Aşağıda UID ile doğrudan çalışan bazı örnek senaryoları inceleyeceğiz.

İlk olarak şu endpoint'e bakalım:

```plaintext
http://idor-sandbox:80/user/?uid=16327
```

Buradaki `uid` parametresi, veritabanından hangi kullanıcı profili bilgisinin çekileceğini belirleyen kimlik numarasıdır.

IDOR Sandbox, kullanıcıları tanımlamak için **beş haneli bir sayı** kullanır. Gerçek dünyadaki uygulamalarda ise genellikle **özel diziler** veya **UUID**'ler gibi daha karmaşık değerler kullanılır. Örnek:

```plaintext
userProfile/a8e62d80-42cc-4ac6-bf53-d28a0ff61a82
```

Tanımlayıcılar uzadıkça ve karmaşıklaştıkça **brute-force** saldırıları daha zor hale gelir. Bu, saldırının imkansız olduğu anlamına gelmez, ancak bu kursun kapsamı dışındadır.

---

### UID ile Gerçek Dünya Senaryoları

Şimdi birkaç UID senaryosuna bakalım.  
Örneğin, büyük bir nakliye şirketi için bir güvenlik değerlendirmesi yaptığımızı varsayalım. Bu durumda, **SKU (Stock Keeping Unit)** numaralarını referanslayan endpoint'leri aramamız mantıklı olacaktır. Burada SKU bizim UID’miz olacaktır.

Hassas verileri dışarı sızdırmak için, bu SKU numaralarını **brute-force** yoluyla zorlayabiliriz. Amacımız, uygulamanın işlevselliğini kötüye kullanarak şirketin özel ürün bilgilerini ortaya çıkarmaktır.

Başka bir örnekte, bir sosyal medya platformunun değerlendirmesini yaptığımızı varsayalım. Bu platformda kullanıcılar yorum yapabiliyor. Hassas verileri dışarı sızdırmak için önce kullanıcıların UID'lerinden oluşan büyük bir liste toplarız. Ardından bu değerleri kullanarak çeşitli web parametrelerinde yineleme yapar, kullanıcılarla ilgili hassas verileri çekmeye çalışırız.

Bir sonraki Öğrenim Biriminde, **IDOR Sandbox ortamında sömürme (exploitation)** işlemini ele alacağız.


**IDOR Sandbox'ta IDOR Zafiyetini Sömürme**  
Bu Öğrenme Birimi şu Öğrenme Hedeflerini kapsar:

- Statik Dosya IDOR sömürüsünün nasıl yapılacağını anlama
    
- ID Tabanlı IDOR sömürüsünü daha yakından öğrenme
    
- Daha Karmaşık IDOR türlerinin nasıl sömürüleceğini keşfetme
    

Bu Öğrenme Biriminde, IDOR sömürüsünün temel yönlerini ele alacağız.  
IDOR Sandbox'ı kullanarak daha önce tartıştığımız IDOR bulgularına karşı savunmasız olabilecek parametreleri manuel olarak tespit etmeye çalışacağız.

İlk olarak, **statik dosya tabanlı IDOR** zafiyetinin doğrudan sömürülmesini inceleyeceğiz.

Bu Öğrenme Modülü boyunca IDOR Sandbox'ı kullanacağız.  
IDOR Sandbox, IDOR tespiti pratiği yapabilmek için bilinçli olarak zafiyetli fonksiyonlar içeren özel bir uygulamadır.

Bu uygulamaya, **Kali Linux VM'imizdeki /etc/hosts dosyasına** ekleyeceğimiz bir kayıt sayesinde erişebiliriz.

```bash
kali@kali:~$ sudo mousepad /etc/hosts
```

İçeriği kontrol etmek için:

```bash
kali@kali:~$ cat /etc/hosts
```

Çıktı şu şekilde olacaktır:

```
127.0.0.1       localhost  
127.0.1.1       kali  

# IPv6 destekli sistemler için önerilen satırlar  
::1     localhost ip6-localhost ip6-loopback  
ff02::1 ip6-allnodes  
ff02::2 ip6-allrouters  

192.168.50.100  idor-sandbox  
```

**Listing 3 - /etc/hosts girdileri**

IDOR Sandbox uygulamasını başlatmadan önce, yukarıdaki gibi **IP adresini Kali makinemizde** doğru şekilde tanımladığımızdan emin olmalıyız.

IDOR Sandbox uygulamasının açılış sayfasını ziyaret ettiğimizde, bizi ilgili Öğrenme Modülü sayfalarına yönlendiren bir dizi butonla karşılaşırız.

![Pasted image 20250523000404.png](/img/user/resimler/Pasted%20image%2020250523000404.png)

Öncelikle **statik dosya IDOR** sömürüsünün nasıl yapılacağını tartışarak başlayalım.

Statik dosya IDOR hakkında daha derin bir anlayışa sahip olduğumuza göre, şimdi IDOR Sandbox içindeki bir örneği inceleyelim.  
Bunu yapmak için, **FILE-BASED IDOR (Dosya Tabanlı IDOR)** butonuna tıklayarak başlayabiliriz.

![Pasted image 20250523000433.png](/img/user/resimler/Pasted%20image%2020250523000433.png)

Yeniden /`docs/?f=1.txt` URI'sine yönlendirileceğiz.

![Pasted image 20250523000450.png](/img/user/resimler/Pasted%20image%2020250523000450.png)

Daha sonra, bu sayfada işlenen ve görüntülenen verileri yakından inceleyelim.

![Pasted image 20250523000504.png](/img/user/resimler/Pasted%20image%2020250523000504.png)

Çıktı, hedef web sunucusundaki bir metin dosyasından sızdırılan bilgileri göstermektedir.

Dosya tabanlı IDOR hakkında öğrendiklerimize dayanarak, **/docs/?f=1.txt** değerini manipüle etmeyi ve bu değeri bir artırmayı deneyelim.

Yeni sorgu dizesini gönderdikten sonra aşağıdaki çıktıyı alacağız.

![Pasted image 20250523000539.png](/img/user/resimler/Pasted%20image%2020250523000539.png)

Bu metin dosyası nesne referansını artırarak bilgi sızdırabileceğimiz yönündeki şüphelerimiz tamamen doğrulandı.  
**f** parametresine `"2.txt"` değerini verdik ve önceki çıktılardan farklı bir çıktı ile karşılaştık.

Bu sefer, bir bireyin tam adını, nereli olduğunu ve organ bağışçısı olup olmadığını sızdırmayı başardık.

Sonraki bölümde, **ID-Based IDOR** sömürüsünü tartışacağız.

Şimdi **veritabanı nesne referanslama** yani **ID-Based IDOR** konusunu inceleyelim.  
Sandbox ana sayfasına geri dönüp **ID-BASED IDOR** butonuna tıklayacağız.

![Pasted image 20250523000609.png](/img/user/resimler/Pasted%20image%2020250523000609.png)

Bu bizi `/customerPage/?custId=1` URI'sine yönlendirir.

![Pasted image 20250523000821.png](/img/user/resimler/Pasted%20image%2020250523000821.png)

URI, sayısal bir müşteri kimliği (customer ID) içeriyor gibi görünüyor.  
Bu durum, klasik bir **IDOR** bulgusunun güçlü bir göstergesidir.

Herhangi bir artış (iterasyon) gerçekleştirmeden önce,  
**custId=1** değeriyle elde edilen çıktının web tarayıcısında nasıl göründüğünü daha yakından inceleyelim.

![Pasted image 20250523000841.png](/img/user/resimler/Pasted%20image%2020250523000841.png)

custId değerini "2" olarak artırmadan önce,  
veritabanından alınan bilgi türlerini not edelim

![Pasted image 20250523000902.png](/img/user/resimler/Pasted%20image%2020250523000902.png)

Tarayıcıda artan değerden kaynaklanan çıktıyı not ederek, bu durumun müşteri kimlik nesnesine ait klasik bir IDOR vakası olduğunu doğrulayabiliriz.

Temelde, güvenli olmayan bir uç noktadan doğrudan veritabanı tablosunda yineleme yaparak, farklı bir müşteriye ait aynı türde bilgileri alabiliyoruz. Bu örnekte, ID değeri "2" olan müşteriye ait bilgiler alınmıştır. Aldığımız bilgiler arasında Kullanıcı ID’si, Kullanıcı Adı (Handle), İsim, Konum ve müşterinin Üyelik Durumu (Member Status) yer almaktadır. Sorgu dizisindeki veritabanı değerini artırarak bu tür bilgilerin erişilebilmesi, veritabanı nesne referanslı IDOR’un varlığına işaret eder.

Tarayıcı üzerinden tamamen farklı bir kullanıcıya ait ve Müşteri ID değeri "2" olan bilgilere eriştiğimiz için, bunun web uygulamasına kayıtlı ikinci kullanıcı olduğunu tahmin edebiliriz.

Bir sonraki bölümde, UID’ler (Unique Identifiers) üzerinde daha detaylı IDOR istismarını ele alacağız.

Başlamak için, kullanıcı IDOR istismarını authenticated (kimlik doğrulamalı) perspektiften inceleyeceğiz.  
Kullanıcı "Harb" olarak ve şifre "RockFaceKingdom" ile  
[http://idor-sandbox:80/user/](http://idor-sandbox/user/) adresinde giriş yapalım.

![Pasted image 20250523001003.png](/img/user/resimler/Pasted%20image%2020250523001003.png)

Giriş yaptıktan sonra, başarılı bir giriş denemesi yapıldığına dair bir bildirim alacağız.  
Ayrıca, kullanıcı "Harb" ile ilişkili çıktıyı gösteren `/user/?uid=62718` adresine yönlendirileceğiz.

![Pasted image 20250523001037.png](/img/user/resimler/Pasted%20image%2020250523001037.png)

Çıktı, oturum açmış kullanıcımız (Harb) için UID, İsim, Kullanıcı Adı (Handle), Coğrafi Konum ve Üyelik Durumunu içermektedir. Şimdi, diğer kullanıcılar hakkında bilgi almaya çalışalım. Hatta bilinmeyen UID’lere sahip kullanıcı verilerini dışarı aktarmayı gösterebiliriz.

Mevcut UID’miz beş basamaklı ardışık karakterlerden oluşuyor ve bu da en az 99.999 olasılık olduğu anlamına gelir. Bu, kayıtlı üye için belli bir güvenlik katmanı sağlar. Ancak bu saldırının temel amacı, potansiyel UID’leri brute force yaparak programatik olarak binlerce olasılığı denemektir.

UID’leri brute force etmek için, Seclists'ten alacağımız kelime listesiyle 99.999 defa deneme yapacağız.  
Kali Linux makinemizde bu kelime listesi şu yoldadır:  
`/usr/share/seclists/Fuzzing/5-digits-00000-99999.txt`

Bu isteklerin tamamını fuzz edersek, gözden geçirmemiz gereken çok sayıda yanıt alacağız.  
Öncelikle curl kullanarak temel bir yanıt boyutu belirleyelim. Bu yanıt boyutunu, wfuzz sonuçlarımızı filtrelemek için kullanacağız.

Aşağıda, `-s` argümanı curl’un ilerleme çubuğunu göstermemesini sağlar, çünkü yalnızca yanıt boyutuyla ilgileniyoruz. Son olarak, `-w '%{size_download}'` argümanıyla curl’dan yanıt boyutunu terminal çıktısında yazdırmasını istiyoruz.

```bash
kali@kali:~$ curl -s http://idor-sandbox:80/user/?uid=62718 -w '%{size_download}'
0
```

Listing 4 - Hatalı Yanıt Boyutlarının Toplanması

Çıktımız 0 bayt olarak döndü. Bunun sebebi, curl isteğini yaparken geçerli bir oturum ID’si belirtmememizdir. Web uygulaması testlerinde, doğru sonuçlar alabilmek için hem kimlik doğrulamasız hem de kimlik doğrulamalı fuzzing yapmamız gerekebilir. Bu da wfuzz komutumuza geçerli bir oturum ID’si eklememiz gerektiği anlamına gelir.

Bu oturum ID’sini almak için Burp Suite’te Intercept’i etkinleştirip, /user/login.php giriş URI’sinden Harb kullanıcısı ile kimlik doğrulaması yapılmış bir isteği yakalamamız gerekiyor.

![Pasted image 20250523001220.png](/img/user/resimler/Pasted%20image%2020250523001220.png)

Yukarıdaki isteği yakaladığımızda bize bir cookie değeri sağlanır. Bu sefer curl’u bu session ID ile kullanarak, kimlik doğrulamalı perspektiften hatalı (erroneous) yanıt boyutunu almaya çalışabiliriz.

Lütfen unutmayın, session ID her seferinde farklı olacaktır ve dikkatle incelenmelidir.

Sonraki curl komutu öncekinden sadece biraz farklıdır.  
Kimlik doğrulamalı bir istek yapmak için `--header` argümanını ekleyeceğiz ve ayrıca bilinen bir hatalı yanıt boyutunu almak için rastgele bir UID değeri olarak “91191” kullanacağız.

```bash
kali@kali:~$ curl -s /dev/null http://idor-sandbox:80/user/?uid=91191 -w '%{size_download}' --header "Cookie: PHPSESSID=2a19139a5af3b1e99dd277cfee87bd64"
2873
```

Listing 5 - Oturum ID’si ile Hatalı Yanıt Boyutlarının Toplanması

Oturum açılmış bir kullanıcı için hatalı yanıt boyutu 2873 bayttır. Gelecek fuzzing denemelerimizde bu değeri dışlayacağız.

Fuzzing işlemine başlayalım. Wfuzz’ta `-c` argümanı renkli çıktı almak için kullanılır. `-z` argümanı ise giriş yöntemimizi belirtir; burada dosya (file) ve ardından kelime listemiz verilir.  
`--hc 404` argümanıyla 404 yanıtlarını dışlayacağız.  
`--hh 2873` ile ise hatalı yanıt boyutu olan 2873 baytı dışlayacağız.

Son olarak, daha önce yakaladığımız cookie değerini içeren header’ı `-H` ile vereceğiz. Fuzzing endpoint’imiz:  
`http://idor-sandbox:80/user/?uid=FUZZ`

```bash
kali@kali:~$ wfuzz -c -z file,/usr/share/seclists/Fuzzing/5-digits-00000-99999.txt --hc 404 --hh 2873 -H "Cookie: PHPSESSID=2a19139a5af3b1e99dd277cfee87bd64" http://idor-sandbox:80/user/?uid=FUZZ
```

```
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://idor-sandbox:80/user/?uid=FUZZ
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000011112:   200        76 L     174 W      2859 Ch     "11111"
000016328:   200        76 L     174 W      2860 Ch     "16327"
000023102:   200        76 L     174 W      2874 Ch     "23101"
000039202:   200        76 L     174 W      2867 Ch     "39201"
000041913:   200        76 L     174 W      2861 Ch     "41912"
000057192:   200        76 L     174 W      2863 Ch     "57191"
000062719:   200        76 L     174 W      2871 Ch     "62718"
000074833:   200        76 L     175 W      2868 Ch     "74832"
000083272:   200        76 L     174 W      2858 Ch     "83271"
000099181:   200        76 L     174 W      2866 Ch     "99180"

Total time: 755.6711
Processed Requests: 100000
Filtered Requests: 99990
Requests/sec.: 132.3327
```

Listing 6 - 99,999 Olası UID’nin Fuzz Edilmesi

Başka kullanıcıların UID değerlerini başarıyla elde ettik. Bu değerleri geçerli UID yerine koyarak bu kullanıcılar hakkında bilgi dışarı aktarabiliriz.

Hadi deneyelim. Tarayıcımızı şu adrese yönlendirelim:  
`http://idor-sandbox:80/user/?uid=57191`  
Fuzzing ile elde ettiğimiz UID’lerden biri.

Bu endpoint’i tarayıcımızda açtığımızda aşağıdaki veriyi alacağız.

![Pasted image 20250523001359.png](/img/user/resimler/Pasted%20image%2020250523001359.png)

Başarıyla McLovin kullanıcısının Name, Handle, Location ve Member Status bilgilerini dışarı aktarabildik.

Bir sonraki Öğrenme Ünitesi’nde, OpenEMR için bir IDOR Vaka Çalışmasını ele alacağız.

## Case Study: OpenEMR

Bu Öğrenme Ünitesi aşağıdaki Öğrenme Hedeflerini kapsamaktadır:

- IDOR’a Black Box perspektifinden yaklaşmayı öğrenmek
    
- Zafiyeti keşfetmeyi anlamak
    
- OpenEMR IDOR exploitasyonu hakkında bilgi geliştirmek
    

Bu Öğrenme Ünitesi’nde, OpenEMR adı verilen açık kaynaklı bir medikal platformdaki IDOR zafiyetinin keşfi ve exploitasyonu üzerinde duracağız. Versiyon 6.0.0.0 (CVE-2021-40352) örneği üzerinden ilerleyeceğiz.[225]  
Gelecek bölümlerde web uygulamasının manuel değerlendirmesini yapacağız.

Değerlendirmeye başlamadan önce, makineye erişim sağlamamız gerekiyor.

OpenEMR uygulamasına Kali Linux VM’imizdeki /etc/hosts dosyası üzerinden erişebiliriz.

```bash
kali@kali:~$ sudo mousepad /etc/hosts
kali@kali:~$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

# IPv6 destekli hostlar için gereken satırlar
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.50.105  openemr
```

Liste 7 - /etc/hosts girdileri

Makineyi, aşağıdaki Başlat (Start) butonuna tıklayarak başlatacağız.  
Bu, host dosyalarımızı güncellemek için kullanabileceğimiz bir IP adresi sağlayacak.

Not: Bu makine çok büyük bir medikal platformdur ve tamamen yüklenmesi birkaç dakika sürebilir.

Keşif aşamamıza, web uygulamasını gezerek ve bu Öğrenme Ünitesi’nde öğrendiklerimize dayanarak dikkat çeken parametre veya işlevleri değerlendirerek başlayacağız. Öncelikle Burp Suite’in çalıştığından ve Intercept butonunun kapalı olduğundan emin olalım.

Keşfe, web tarayıcımızı başlatıp [http://openemr:80/](http://openemr/) adresine yönlendirerek başlayacağız.

![Pasted image 20250523001535.png](/img/user/resimler/Pasted%20image%2020250523001535.png)

“Password1!” parolasını kullanarak “lowpriv” adlı düşük ayrıcalıklı bir kullanıcı olarak oturum açacağız.

![Pasted image 20250523001601.png](/img/user/resimler/Pasted%20image%2020250523001601.png)

Birkaç dakika sonra, OpenEMR için gösterge tablosu arayüzü ile karşılaşacağız.

![Pasted image 20250523001614.png](/img/user/resimler/Pasted%20image%2020250523001614.png)

Sayfanın sağ üst köşesinde kullanıcımızın adı görünmekte, bu da düşük yetkili kullanıcı Nancy Jones olarak başarılı bir şekilde giriş yaptığımızı gösteriyor.

Yeni bir web uygulamasını keşfederken, uygulamanın nasıl çalıştığını ve zafiyetleri nasıl keşfedebileceğimizi daha iyi anlamak için uygulamayı manuel olarak incelemeye zaman ayırmalıyız.

Bu büyük medikal platformda, inceleyebileceğimiz birçok farklı buton, sekme, sayfa ve form bulunmakta.

Başlamak için, giriş sayfasında doğrudan karşımıza iki ayrı sekme çıktığını göreceğiz: Takvim (Calendar) ve Mesaj Merkezi (Message Center).

![Pasted image 20250523001703.png](/img/user/resimler/Pasted%20image%2020250523001703.png)

Mesaj Merkezi sekmesine tıkladıktan sonra, bir kayıt içeren bir tablo gösterilecek. Görünüşe göre burada hastalar veya personel ile ilgili gizli bilgiler yer alıyor. Tablo, Mesajlar (Messages), Hatırlatıcılar (Reminders) ve Geri Çağırmalar (Recalls) gibi tıklanabilir birkaç buton içeriyor. Ayrıca, mevcut kullanıcı olarak mesaj ekleyebiliriz. Şimdilik, Nancy Jones tarafından Stan Chyna’ya gönderilen mesajı inceleyelim.

Önemli bir not olarak, bu uygulamadan gönderilen her isteğin tamamen sonuçlanmasına izin vermemiz gerekiyor. Uygulama, arka planda bilgi işlerken “takılabilir” (hang) potansiyeline sahip. Önce Mesaj Merkezi’ne gönderdiğimiz önceki isteğin tamamlandığını doğruladıktan sonra, devam edip Stan Chyna’ya tıklayacağız.

![Pasted image 20250523001757.png](/img/user/resimler/Pasted%20image%2020250523001757.png)

İlgilendiğimiz mesajı gösteren aşağıdaki Mesajlar, Hatırlatıcılar ve Geri Çağırmalar sayfası karşımıza çıkacak. Bu sayfa, yanıt vermemize olanak tanıyor ve diğer seçenekleri sunuyor.

![Pasted image 20250523001827.png](/img/user/resimler/Pasted%20image%2020250523001827.png)

Mesajı Tamamını Görüntülemek için Print Message (Mesajı Yazdır) seçeneğini kullanabiliriz. Bunu denemeden önce, Burp Suite’e dönelim ve Intercept’i açalım.

![Pasted image 20250523001849.png](/img/user/resimler/Pasted%20image%2020250523001849.png)

Intercept özelliği etkinleştirildiğinde, hasta mesajlarına geri dönebilir ve Print Message (Mesajı Yazdır) seçeneğine tıklayabiliriz.

![Pasted image 20250523001922.png](/img/user/resimler/Pasted%20image%2020250523001922.png)

Bu işlem, Burp tarafından hemen yakalanan bir istek oluşturacaktır. Yakalanan isteğin gövdesinde sağ tıklayarak “Send to Repeater” (Tekrar Gönder) seçeneğini seçeriz.

![Pasted image 20250523001932.png](/img/user/resimler/Pasted%20image%2020250523001932.png)

İsteği daha detaylı analiz için Repeater’a gönderdikten sonra, “Send” (Gönder) butonuna tıklayacağız ve karşımıza aşağıdaki sonuç çıkacaktır.

![Pasted image 20250523001950.png](/img/user/resimler/Pasted%20image%2020250523001950.png)

Gösterildiği gibi, isteği gönderdik ve şimdi hem istenen URI’yı hem de tam mesajı içeren yanıtı daha yakından inceleyebiliyoruz.

İstekte gönderilen ve değeri "11" olan noteid parametresinin, IDOR Sandbox’ta karşılaştığımız parametrelere çok benzediği kesinlikle görünüyor. Edindiğimiz bilgilere dayanarak burada bir IDOR açığının olabileceğini tahmin edebiliriz.

Keşif aşamamızı tamamladığımıza göre, sonraki bölümde veri sızdırmaya (exfiltration) başlayabiliriz.

Bu açığı sömürürken birincil hedefimiz mümkün olan en fazla miktarda veriyi dışarı çıkarmaktır.

Artık noteid parametresinin IDOR için potansiyel olarak savunmasız bir uç nokta olduğunu keşfettiğimize göre, noteid kullanarak veri sızdırma şüphemizi test edelim.

Makul bir şekilde, karşılaştığımız bulgunun ID tabanlı bir IDOR olduğu sonucuna varabiliriz.

Burp Suite’in Intercept özelliğini bundan sonra devre dışı bırakacağız.

Önceki bölümdeki isteği (Repeater’da bulunan) alıp, okunabilirlik açısından, uç noktayı tarayıcımıza yükleyerek çıktıyı gözlemleyelim.

![Pasted image 20250523002102.png](/img/user/resimler/Pasted%20image%2020250523002102.png)

Bir Help Desk personeli tarafından gönderilen mesajın tam çıktısını aldık; bu mesajda yaklaşan yeni hastalarla ilgili hassas bilgiler yer alıyor.

Bu bilgi, sistemin daha fazla ele geçirilmesi için doğrudan faydalı görünmüyor. Ancak, noteid değerini azaltarak (decrement) tekrar tekrar deneseydik, daha değerli bilgilere rastlayabilirdik.

![Pasted image 20250523002128.png](/img/user/resimler/Pasted%20image%2020250523002128.png)

Bu durumda biraz daha şanslıydık. Eğer bu sistemin güvenlik mekanizmalarını aşabilseydik, özel IP aralığındaki komşu makinelerin IP adreslerine ulaşmış olurduk ve bu da ağda yatay hareket (lateral movement) yapmamıza olanak tanıyabilirdi.

Zafiyetli endpoint üzerinde değerleri artırıp azaltmaya devam ederek, hedef sistemin ele geçirilmesi için çok daha değerli bilgiler elde etme şansımız olabilir.

Son olarak, bulgumuzu yakından gözden geçirelim. Düşünelim ki düşük yetkili kullanıcı olan Nancy Jones olarak hareket ediyoruz. Diyelim ki bu platforma erişimi olan bir hasta olsaydık. Bu durum son derece ciddi bir güvenlik açığını gösteriyor. Tıbbi verilerin ele geçirilmesi, doktor ile hastaları arasında kritik derecede hassas bilgilerin sızmasına sebep oluyor.

Not: Bu makine çok büyük bir tıbbi platformdur ve tamamen yüklenmesi birkaç dakika sürebilir.


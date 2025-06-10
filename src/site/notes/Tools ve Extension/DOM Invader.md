---
{"dg-publish":true,"permalink":"/tools-ve-extension/dom-invader/","created":"2025-06-10T16:29:31.544+03:00","updated":"2025-06-10T20:55:41.512+03:00"}
---

DOM Invader, hem web mesajı hem de prototype pollution vektörleri dahil olmak üzere çeşitli source ve sink'leri kullanarak DOM XSS güvenlik açıklarını test etmenize yardımcı olan tarayıcı tabanlı bir araçtır. Yalnızca Burp'ün built-in browser'ı üzerinden kullanılabilir ve burada bir extension olarak önceden yüklenmiş olarak gelir.

![Pasted image 20250610163049.png](/img/user/Pasted%20image%2020250610163049.png)

## Temel özellikler

Etkinleştirildiğinde, DOM Invader tarayıcının DevTools paneline yeni bir sekme ekler. Bu, aşağıdaki temel görevleri gerçekleştirmenizi sağlar:

* Reflected XSS gibi DOM XSS için test yapın. Genişletilmiş DOM görünümü, sayfadaki kontrol edilebilir sinkleri anında tanımlamanızı sağlayarak hem XSS context'ini hem de input'unuzun nasıl sterilize edildiğini gösterir. Daha fazla bilgi için [DOM XSS Testi](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/dom-xss) bölümüne bakın.

* `postMessage()` metodu aracılığıyla bir sayfada gönderilen web mesajlarını loglayın, değiştirin ve yeniden gönderin. Bu, web mesajları aracılığıyla DOM XSS için test yapmanızı sağlar. Ayrıca DOM Invader'ın kendi özel olarak hazırlanmış web mesajlarını göndererek sizin adınıza güvenlik açıklarını araştırmasına izin verebilirsiniz. Daha fazla bilgi için [Web mesajları kullanarak DOM XSS testi](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/web-messages) bölümüne bakın.

* Client -side prototype pollution source'larını otomatik olarak tanımlayın ve tehlikeli sink'lere aktarılan kontrol edilebilir gadget'ları tarayın. Daha fazla bilgi için [client-side prototype pollution](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution) test etme bölümüne bakın.

* [DOM clobbering](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/dom-clobbering) güvenlik açıklarını otomatik olarak belirleyin.

[DOM Invader'ın nasıl etkinleştirileceği hakkında daha fazla bilgi için bkz](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling).

DOM Invader son derece yapılandırılabilirdir, böylece davranışını farklı web sitelerine ve kullanım durumlarına uyacak şekilde ince ayar yapabilirsiniz. Daha fazla bilgi için [DOM Invader ayarları](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/settings) bölümüne bakın.


# Enabling DOM Invader (Etkinleştirme)

DOM Invader, Burp'ün tarayıcısına önceden yüklenmiştir, ancak bazı özellikleri diğer test etkinliklerinizi engelleyebileceğinden varsayılan olarak devre dışı bırakılmıştır.

DOM Invader'ı etkinleştirmek için:

1- `Proxy` > `Intercept` sekmesine gidin ve Burp'ün tarayıcısını açın.

2- Browser penceresinin sağ üst köşesinde Burp Suite logosuna tıklayın. Bu logoyu göremiyorsanız, önce yapboz simgesine tıklayın. Burp Suite Navigation Recorder ve DOM Invader ayarlar menüsü sekmelerini içeren bir panel açılır.

![Pasted image 20250610164834.png](/img/user/Pasted%20image%2020250610164834.png)

3- DOM Invader ayarlarından, switch'i DOM Invader açık olacak şekilde değiştirin.

4- Tarayıcıyı yenilemek için `Reload` (Yeniden Yükle) butonuna tıklayın. Değişikliklerinizin etkili olması için bu gereklidir.

5- Ana tarayıcı penceresinde herhangi bir yere sağ tıklayın ve tarayıcının DevTools panelini açmak için Inspect'i seçin. Bunun artık DOM Invader sekmesini içerdiğini unutmayın. En iyi deneyim için paneli tarayıcı penceresinin altına yerleştirmenizi öneririz.

![Pasted image 20250610164953.png](/img/user/Pasted%20image%2020250610164953.png)


Not : Varsayılan olarak, DOM Invader açık veya kapalı olması da içinde olmak üzere önceki ayarlarınızı hatırlar. DOM Invader hala etkin durumdayken Burp'un tarayıcısını kapatırsanız bunu aklınızda bulundurun. Bu davranışı devre dışı bırakmak için Ayarlar > Araçlar > Burp'un tarayıcısına gidin ve Kapattıktan sonra ayarları ve geçmişi sakla seçimini kaldırın.

# Testing for DOM XSS

DOM XSS testi, binlerce kod satırına kadar uzanabilen karmaşık JavaScript aracılığıyla inputunuzun akışını manuel olarak izlemeyi gerektirdiğinden sıkıcı olabilir. DOM Invader, inputunuzun içine aktığı tüm sinkleri çevredeki context ile birlikte size anında göstererek bu süreci büyük ölçüde basitleştirir.

![Pasted image 20250610165257.png](/img/user/Pasted%20image%2020250610165257.png)

İlgili özelliklerin çoğuna extension'ın DOM görünümünden erişebilirsiniz.

## Injecting a canary

DOM Invader, önceden tanımlanmış bir “canary” stringinin oluşumlarını aramak için DOM'u otomatik olarak pars ederek çalışır. Bu, hangi sinklere gittiklerini görmek için farklı source'lara enjekte edebileceğiniz keyfi ancak farklı bir alfanümerik karakter stringidir.

DOM Invader'ın izlediği geçerli canary'yi DOM görünümünün sol üst köşesinde görebilirsiniz. İsterseniz canary'yi özel bir string olarak [değiştirebileceğinizi](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/settings/canary) unutmayın.

Canary'yi bir source'a manuel olarak enjekte etmek için:

1- Tarayıcının DevTools panelindeki DOM Invader sekmesine gidin.
2- DOM görünümünde olduğunuzdan emin olun.
3- Canary'yi kopyala'ya tıklayın. DOM Invader'ın izlediği canary panonuza kopyalanır.
4- Canary'yi test etmek istediğiniz herhangi bir input'a yapıştırın. Bu, URL'deki sorgu parametreleri, form alanları vb. olabilir.

Potansiyel source'lar hakkında daha fazla bilgi için Web Security Academy'de DOM tabanlı güvenlik açıkları hakkındaki konumuza göz atın.

### Bir canary'i birden fazla source'a enjekte etme

Canary'i aynı anda birden fazla source manuel olarak yapıştırabilmenize rağmen, bunu otomatik olarak yapmak için aşağıdaki seçeneklere de sahipsiniz:

* URL parametrelerini enjekte et - Her parametre için ayrı bir sekme kullanarak canary'yi URL'deki her sorgu parametresine otomatik olarak enjekte eder.

* Formları enjekte et - Kanaryayı sayfada algılanan tüm HTML form alanlarına otomatik olarak enjekte eder. Enjeksiyonun etkili olması için formu manuel olarak göndermeniz gerektiğini unutmayın.


Not : Canary'nin tüm URL parametrelerine ve form alanlarına aynı anda enjekte edilmesi sitenin düzgün çalışmasını engelleyebilir. En iyi sonuçlar için her seferinde bir source'u test etmenizi öneririz.

## Kontrol edilebilir sinklerin belirlenmesi

Bir canary enjekte ettikten sonra DOM Invader, canary'nizin göründüğü tüm sinkleri tanımlamak için DOM'u otomatik olarak parse eder. Daha sonra bu sinkleri, ne kadar ilginç olduklarına göre sıralanmış olarak DOM görünümünde görüntüler.

## XSS context'inin belirlenmesi

Kontrol edilebilir bir sink belirledikten sonra, bir sonraki adım enjekte edilen payload'ınızın göründüğü context'i incelemektir. Bu, aşağıdaki bilgilerin belirlenmesini içerir:

* İster HTML ister JavaScript execution sink ile çalışıyor olun.

* İnputunuzun, içinden çıkmanız gereken herhangi bir özel karakterle çevrili olup olmadığı. Bunlar tırnak işaretleri, taglar, attribute'lar ve benzerlerini içerir.

* Web sitesinin sink'e ulaşmadan önce inputunuz üzerinde ne tür bir doğrulama, temizleme veya başka bir işlem gerçekleştirdiği.

Bu konuda size yardımcı olmak için DOM Invader, hem canary'iniz hem de DOM'da göründükleri gibi enjekte ettiğiniz tüm çevreleyici karakterleri dahil olmak üzere sink'in içeriğini görüntüler. Bu, escape edip etmediklerini veya encode kodlanmadıklarını kolayca görmek için canary'inize özel karakterler ekleyebileceğiniz anlamına gelir. Aşağıdaki örnekte, çeşitli yararlı karakterleri başarıyla enjekte edebildiğimizi görebilirsiniz.

![Pasted image 20250610172757.png](/img/user/Pasted%20image%2020250610172757.png)

DOM Invader'ın tanımladığı sink türüne bağlı olarak aşağıdaki ayrıntıları da görebilirsiniz:

* `Outer HTML` - Canary'nizi çevreleyen HTML element.
* `Frame path` - Canary'inizin sink'e aktarıldığı frame.
* `Event`- Canary'inizin sink'e aktarıldığında gerçekleşen JavaScript event'i.

Bu bilgiler, XSS context'ini kolayca görmenizi ve bir exploit oluşturmak için hangi karakterlere ve eventlere ihtiyacınız olduğunu test etmenizi sağlar. Aşağıdaki örnekte, XSS proof-of-concept exploit'imizi enjekte etmek için çift tırnaklı string ve çevreleyen `<span>` den başarıyla çıktık.

![Pasted image 20250610173159.png](/img/user/Pasted%20image%2020250610173159.png)

## client-side code'un incelenmesi

Farklı injection'ları denerken, inputunuzun aniden sink'e akışının bıraktığını fark edebilirsiniz. Bunun nedeni, sink'e yalnızca koşullu bir ifadenin bir dallanması gibi belirli bir kod yolu üzerinden ulaşabilmeniz olabilir.

DOM Invader, client-side'daki kodda input'unuzun sink'e aktarıldığı noktaya doğrudan atlamanızı sağlar. Daha sonra, inputunuzun sink'e ulaşması için hangi koşulları karşılaması gerektiğini belirlemek için önceki kodu inceleyebilirsiniz.

Koddaki ilgili satırı görüntülemek için:

1. Sinke ulaşacağını bildiğiniz bir payload enjekte edin.
2. DOM Invader'ın `DOM` görünümünde, `Stack Trace` kolonundaki bağlantıya tıklayın. Bu, tarayıcının konsoluna bir stack trace çıktısı verir.
3. DevTools panelinde, `console` sekmesine geçin.
4. Stack trace'de, en üstteki bağlantıya tıklayın (yalnızca bir tane olabilir). Bu, `Source`'lar sekmesinde client-side JavaScript'i açar ve inputunuzun sink'e aktarıldığı satıra odaklanır.

Daha fazla bilgi edinin
DOM Invader son derece yapılandırılabilir. DOM Invader'ın gelişmiş özellikleri ve belirli bir site için davranışlarına nasıl ince ayar yapabileceğiniz hakkında daha fazla bilgi için DOM Invader ayarları bölümüne bakın.


# Web mesajlarını kullanarak DOM XSS testi

DOM Invader, web mesajlarını kullanarak DOM XSS için test yapmanızı sağlayan bir dizi özellik sunar. Bunlar şunları içerir:

* Sayfadaki `postMessage()` methodu aracılığıyla gönderilen tüm web mesajlarını, bunlarla ilgili yararlı ayrıntılarla birlikte günlüğe kaydetme. Bu, Burp Proxy'nin HTTP isteklerinizin ve yanıtlarınızın geçmişini göstermesine benzer.

* DOM XSS güvenlik açıklarını manuel olarak araştırmak için web mesajlarını değiştirmenize ve yeniden göndermenize olanak tanır. Bu, Burp Repeater'ın değiştirilmiş HTTP isteklerini yeniden yayınlamasına benzer.

 * Sizin adınıza DOM XSS araştırması yapmak için web mesajlarını otomatik olarak değiştirir ve gönderir.

Tüm bu özelliklere DOM Invader'ın `Messages` görünümünden erişebilirsiniz.

![Pasted image 20250610173947.png](/img/user/Pasted%20image%2020250610173947.png)

Bu özellikleri kullanabilmek için öncelikle DOM Invader ayarlar menüsünden `Postmessage` **`interception`** etkinleştirmeniz gerektiğini unutmayın. Daha fazla bilgi için [Main settings](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/settings/main) bölümüne bakınız.

#### Web Security Academy

web message-based DOM XSS konusunda bilginizi tazelemeniz gerekiyorsa, pratik yapmak için kasıtlı olarak savunmasız bazı laboratuvarlar bulabileceğiniz Web Security Academy'ye göz atın.

[Web mesajı kaynağını kontrol etme](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source)


## Enabling web message interception (Etkinleştirme)

Hedef sitenizin işlevselliğine müdahale etmekten kaçınmak için DOM Invader'ın web message özellikleri varsayılan olarak devre dışıdır.

Bu özellikleri etkinleştirmek için:

1. DOM Invader ayarları menüsüne gidin.
2. Postmessage interception switch'ini seçin.
3. Tarayıcıyı yeniden yüklemek için Reload'ye tıklayın. Değişikliklerinizin etkili olması için bu gereklidir.

![Pasted image 20250610174526.png](/img/user/Pasted%20image%2020250610174526.png)

## Identifying interesting web messages

Web message yakalamayı etkinleştirdiğinizde, DOM Invader sayfada `postMessage()` methodu aracılığıyla gönderilen tüm web mesajlarını otomatik olarak log'a kaydeder. Varsayılan olarak, algıladığı tüm message event handler'lara kendi mesajlarını da oluşturur ve gönderir.

Bunları `Messages` görünümünde görebilirsiniz.

![Pasted image 20250610174719.png](/img/user/Pasted%20image%2020250610174719.png)

## Automated web message analysis

Varsayılan olarak, DOM Invader sizin adınıza ilginç message'ları belirlemeye ve işaretlemeye çalışır. Bunu, message'ları aşağıdaki şekillerde değiştirerek yapar:

* Canary'nizi mesajın `data` özelliği aracılığıyla enjekte etmek. DOM Invader, tıpkı `DOM` görünümündeki diğer soruce'larda yaptığı gibi, bu verilerin aktığı tüm sinkleri tanımlamak için bunu kullanabilir.

* Message'nin origin'ini, beklenen domain adı ile başlayan ve biten sahte bir origin ile değiştirmek. Bu, DOM Invader'ın gelen iletilerin origin'ini doğrulamak için hatalı mantığa veya düzenli ifadelere dayanan event handler'ları otomatik olarak tanımlamasını sağlar.

Not : Bu özelliklerin her ikisini de DOM Invader'ın ayarlar menüsünden devre dışı bırakabilirsiniz. Daha fazla bilgi için Web mesajı ayarları bölümüne bakın.

DOM Invader, gözlemlenen davranışa dayanarak, tahmini bir sorun önem derecesi ve güven düzeyi görüntüleyerek istismar edilebilir olduğunu düşündüğü iletileri otomatik olarak işaretler. Sayfada gönderilen tüm mesajlar, DOM Invader'ın otomatik olarak tespit edemediği güvenlik açıkları içerebileceğinden, en azından bir **`Information`** önem derecesi ile listelenir.

## Message details

Mesajın `origin`, `data` veya `source` özelliklerine client-side JavaScript tarafından erişilip erişilmediği de dahil olmak üzere, mesaj hakkında daha ayrıntılı bilgi görüntülemek için her bir mesaja tıklayabilirsiniz.

![Pasted image 20250610175424.png](/img/user/Pasted%20image%2020250610175424.png)

Bu bilgiler, mesajın kullanışlı olup olmadığını ve uygun bir istismarı nasıl yapabileceğinizi belirlemenize yardımcı olacak ipuçları sağlayabilir.

### Origin accessed

Client-side kodu mesajın `origin` özelliğine hiç erişmiyorsa, originin doğrulanmıyor olması muhtemeldir. Sonuç olarak, event handler'a rastgele bir external domain'den cross-origin mesajlar gönderebilirsiniz.

Ancak, client-side kodun origin özelliğine eriştiği mesajlar bile hala güvensiz olabilir. Hala doğrulamayı atlatabilirsiniz. Bunu yapmanın yollarını bulmanıza yardımcı olmak için, DOM Invader bir stack trace aracılığıyla koddaki ilgili satıra bir link sağlar. Daha fazla bilgi için [client-side kodunu inceleme](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/dom-xss#studying-the-client-side-code) bölümüne bakın.

#### Web Security Academy

[Bypassing flawed origin validation](https://portswigger.net/web-security/cors#errors-parsing-origin-headers)


### Data accessed

Mesajın `data` özelliği, potansiyel payloadları enjekte ettiğiniz yerdir. JavaScript bu özelliğe hiç erişmezse, bir alıcıya aktarılamaz. Bu durumda, mesaj ilgi çekici değildir.

### Source accessed

Message'in `source` özelliği, gönderildiği `window` object'e bir referanstır. Pratikte bu genellikle bir iframe'e yapılan bir referanstır. Web siteleri genellikle origin yerine `source` özelliğini doğrular çünkü bu, mesajın belirli, güvenilir bir iframe'den geldiğinden emin olmanın daha sağlam bir yoludur.

Originde olduğu gibi, bu özelliğe erişen client-side kodun source'un doğrulandığını veya bu doğrulamanın atlanamayacağını garanti etmediğini unutmayın.

## Replaying web messages

DOM Invader, Burp Repeater'daki HTTP isteklerinde olduğu gibi web mesajlarını değiştirmenize ve yeniden oynatmanıza olanak tanır. Bu, web mesajlarını source olarak kullanarak DOM XSS için araştırma yapmayı çok daha basit hale getirir.

Değiştirilmiş bir web mesajı göndermek için:

1- `Messages` görünümünden, mesaj ayrıntıları dialog kutusunu açmak için herhangi bir mesaja tıklayın.
2- `Data` alanını gerektiği gibi düzenleyin.
3- `Send`'e tıklayın.

Örneğin, event handler'ın origini doğrulamadığı ve verileri `element.innerHTML` sink'ine aktardığı bir mesaj tanımlayabilirsiniz. Bu durumda, `<`, `>` ve `“` gibi karakterlerin escaped olup olmadığını test etmek için mesajlar gönderebilir, ardından bu karakterleri kullanarak bir proof-of-concept payload oluşturup gönderebilirsiniz.

## Generating a proof of concept

Bir web mesajı kullanarak exploit edilebilir bir güvenlik açığını başarıyla tespit ettiğinizde, DOM Invader raporlarınıza dahil edebileceğiniz bir HTML proof of concept oluşturabilir.

Bir proof of concept oluşturmak için:

* Mesaj ayrıntıları iletişim kutusunu açmak için savunmasız mesajı seçin.
* Değerleri istismarınız için gerektiği gibi değiştirin.
* `Click Build PoC`'a tıklayın. HTML panonuza kaydedilir.


# Testing for client-side prototype pollution

DOM Invader, client-side prototype pollution güvenlik açıklarını test etmenize yardımcı olacak bir dizi özellik sunar. Bunlar aşağıdaki temel görevleri gerçekleştirmenizi sağlar:

* URL'deki prototype pollution ve web mesajları aracılığıyla gönderilen JSON objectleri için source'ları otomatik olarak tespit edin. Bu, aynı source'u kullanan alternatif tekniklerin tespit edilmesini de içerir.

* Keşfedilen source'ları kullanarak `Object.prototype`'ı pollution ederek bir proof of concept oluşturun. Daha sonra tarayıcı konsolu aracılığıyla güvenlik açığını manuel olarak doğrulayabilirsiniz.

*  Bir exploit oluşturmak için kullanabileceğiniz potansiyel araçları tarayın.

## Enabling prototype pollution

Hedef sitenizin işlevselliğine müdahale etmekten kaçınmak için DOM Invader'ın prototype pollution özellikleri varsayılan olarak devre dışıdır. Bu özellikleri etkinleştirmek için:

* DOM Invader ayarları menüsüne gidin.
* `Attack type` altında, `Prototype pollution` açık olacak şekilde switch'i değiştirin.
* Tarayıcıyı yenilemek için `Reload`'a tıklayın. Değişikliklerinizin etkili olması için bu gereklidir.

![Pasted image 20250610181611.png](/img/user/Pasted%20image%2020250610181611.png)

DOM Invader artık siz göz atarken prototype pollution source'ları tarıyor.

## Detecting sources for prototype pollution

Prototype pollution'ı etkinleştirdiğinizde, DOM Invader sayfayı `Object.prototype`'a rastgele özellikler eklemenizi sağlayan source'lar için otomatik olarak kontrol eder. Tanımladığı tüm source'lar, daha fazla test için bazı yararlı bilgiler ve özelliklerle birlikte DOM view'da görüntülenir.

![Pasted image 20250610181825.png](/img/user/Pasted%20image%2020250610181825.png)

Bu örnekte, DOM Invader `location.hash` source'ını kullanarak `Object.prototype`'ı polliting etmek için iki potansiyel teknik belirlemiştir.

## Manually confirming sources for prototype pollution

DOM Invader prototype pollution için potansiyel bir source belirledikten sonra, bunu manuel olarak onaylamanıza da yardımcı olur.

Bu source aracılığıyla prototype pollution'ın mümkün olup olmadığını manuel olarak test etmek için:

1. `DOM` view'dan ilgili source'un yanındaki `Test` butonuna tıklayın. DOM Invader, `Object.prototype`'a rastgele bir özellik eklemek için seçili source'u kullandığı yeni bir sekme açar.

2. Yeni sekmede tarayıcı konsoluna gidin. DOM Invader'ın otomatik olarak `Object.prototype` çıktısını verdiğine dikkat edin.

3. Bu object'in bir proof-of-concept `testproperty` içerdiğini doğrulamak için node'ları genişletin.

![Pasted image 20250610182300.png](/img/user/Pasted%20image%2020250610182300.png)

4. Konsolda yeni bir object oluşturun:

```js
let myObject = {};
```

Yeni object'inizin prototype chain aracılığıyla `testproperty`'yi miras aldığını doğrulayın:

```js
console.log(myObject.testproperty);
// Output: 'DOM_INVADER_PP_POC'
```

## Scanning for prototype pollution gadgets

Bir prototype pollution source'u, bir “gadget” özelliğine de erişiminiz olmadığı sürece hiçbir işe yaramaz. Bu, düzgün bir şekilde sterilize edilmeden bir sink'e aktarılan, kullanıcı tarafından kontrol edilebilen herhangi bir özelliktir. Böyle bir gadget'ı manuel olarak bulmak son derece sıkıcıdır, ancak DOM Invader bu işlemi otomatikleştirebilir.

Belirli bir source'u kullanan gadget'ları taramak için:

1. `DOM` view'dan, DOM Invader'ın bulduğu herhangi bir prototype pollution'ın yanındaki `Scan for gadgets` butonuna tıklayın. DOM Invader yeni bir sekme açar ve uygun gadget'ları taramaya başlar.

2. Aynı sekmede, DevTools panelindeki `DOM Invader` sekmesini açın. Tarama tamamlandığında, `DOM` view'i DOM Invader'ın tanımlanan gadget'lar aracılığıyla erişebildiği tüm sinkleri görüntüler. Aşağıdaki örnekte, `html` adlı bir gadget özelliği `innerHTML` sink'ine aktarılmıştır.

![Pasted image 20250610182751.png](/img/user/Pasted%20image%2020250610182751.png)

## Generating a proof-of-concept exploit

DOM Invader prototype pollution için bir gadget bulduğunda, XSS'yi doğrulamak için source, gadget ve sink'i birleştirerek otomatik olarak bir proof-of-concept oluşturabilir.

Keşfedilen sink'in yanındaki Exploit butonuna tıklamanız yeterlidir. DOM Invader, `alert()` fonksiyonunu başarıyla çağırdığı yeni bir pencere açar.

# Testing for DOM clobbering with DOM Invader

DOM Invader, sizin adınıza DOM clobbering güvenlik açıklarını otomatik olarak test edebilir. DOM clobbering, DOM'u sayfadaki JavaScript'in davranışını değiştirmenizi sağlayacak şekilde manipüle etmek için bir sayfaya HTML enjekte ettiğiniz bir tekniktir.

[DOM clobbering](https://portswigger.net/web-security/dom-based/dom-clobbering)

## Enabling DOM clobbering

Hedef sitenizin işlevselliğine müdahale etmekten kaçınmak için DOM clobbering varsayılan olarak devre dışıdır. Bu kontrolleri etkinleştirmek için:

1. DOM Invader ayarları menüsüne gidin.
2. `Attack types` altında, `DOM clobbering` açık olması için switch'i değiştirin.
3. Tarayıcıyı yenilemek için `Reload`'a tıklayın. Değişikliklerinizin etkili olması için bu gereklidir.

![Pasted image 20250610203834.png](/img/user/Pasted%20image%2020250610203834.png)

DOM Invader, artık siz gezinirken DOM'daki güvenlik açıklarını tarıyor.

# Main DOM Invader settings

DOM Invader ayarlar menüsüne erişmek için tarayıcının sağ üst köşesindeki Burp Suite logosuna tıklayın ve ardından DOM Invader sekmesine geçin.

![Pasted image 20250610204258.png](/img/user/Pasted%20image%2020250610204258.png)

`Main settings` bölümünden aşağıdaki seçeneklere sahipsiniz.

## Enable DOM Invader

Bu, DOM Invader'ı etkinleştirmek için kullanılan genel switch'tir. DOM Invader varsayılan olarak devre dışıdır, çünkü bazı özellikleri hedef web sitesinin işlevselliğine müdahale ederek diğer test faaliyetlerinizi etkileyebilir. Etkinleştirildiğinde, DOM Invader'a tarayıcının DevTools panelindeki sekmesinden erişebilirsiniz.

## Postmessage interception

Bu ayar etkinleştirildiğinde, sitenin web mesajlarındaki DOM XSS'yi test etmek için DevTools panelindeki `Messages` görünümünü kullanabilirsiniz. DOM Invader'ın web mesajı özellikleri hakkında daha fazla bilgi için [Web mesajlarını kullanarak DOM XSS testi bölümüne bakın](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/web-messages).

Bu davranışa ince ayar yapmanızı sağlayan birkaç alt ayar da vardır. Daha fazla bilgi için [Web message settings](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/settings/web-messages) bölümüne bakın.


## Source ve sink'leri özelleştirme

Source ve sink ayarlarını açmak için, main DOM Invader switch'inin yanındaki dişli simgesine tıklayın. Buradan, DOM Invader'ın hangi source ve sink'leri enjekte ettiğini (instrument) kontrol edebilirsiniz. Varsayılan olarak, tüm source'lar gizlidir ve yalnızca en ilgi çekici sink'ler enjekte edilir.

![Pasted image 20250610205341.png](/img/user/Pasted%20image%2020250610205341.png)

Belirli bir sink’in enjekte edilmesini (instrumentation) devre dışı bırakmak isteyebilirsiniz, özellikle de bu durum sitenin doğru çalışmasını engelliyorsa. Örneğin, `eval()` fonksiyonunun enjekte edilmesi davranışını değiştirebilir ve ilgili işlevselliğin bozulmasına neden olabilir.

# DOM Invader attack types



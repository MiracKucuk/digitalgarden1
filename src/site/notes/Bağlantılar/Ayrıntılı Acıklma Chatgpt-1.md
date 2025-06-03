---
{"dg-publish":true,"permalink":"/baglantilar/ayrintili-aciklma-chatgpt-1/"}
---

Bu kod, bir kullanıcı adı kontrolü yapmak için kullanılan bir fonksiyon. Amaç, kullanıcıya, girdiği kullanıcı adının alınmış mı yoksa kullanılabilir mi olduğunu göstermek. Kodun her bir adımını ve ne iş yaptığını tek tek açıklayacağım:

1. **Fonksiyon Tanımlaması**:

![Pasted image 20241213072100.png](/img/user/resimler/Pasted%20image%2020241213072100.png)

Bu satırda bir **fonksiyon** başlatılıyor. Fonksiyon, belirli bir işlem yapmak üzere tanımlanmış bir kod bloğudur. **checkUsername** fonksiyonu, bir kullanıcı adı kontrolü yapmak için çalışacak.


2. **XMLHttpRequest Nesnesi Oluşturma**:

![Pasted image 20241213072125.png](/img/user/resimler/Pasted%20image%2020241213072125.png)

Burada, **xhr** adında bir değişken oluşturuluyor ve ona **XMLHttpRequest** adlı bir nesne atanıyor. Bu nesne, JavaScript'in **asenkron** (yani sayfanın yeniden yüklenmesini engellemeden veri gönderebilmesini sağlayan) bir özelliğidir. Bu nesne sayesinde, sayfanın yüklenmesi durmadan, sunucuya istek gönderebiliriz.


3. **XMLHttpRequest'e Olay Dinleyici Eklenmesi**:

![Pasted image 20241213072202.png](/img/user/resimler/Pasted%20image%2020241213072202.png)

Burada, **xhr** nesnesine bir **olay dinleyici** ekleniyor. Bu, **xhr** nesnesinin durumu her değiştiğinde (yani, istek tamamlandığında) çalışacak bir fonksiyon tanımlanıyor. Bu fonksiyon, isteğin sonucuna göre işlemler yapacak.

4. **Sunucudan Gelen Yanıtın Kontrolü**:

![Pasted image 20241213072226.png](/img/user/resimler/Pasted%20image%2020241213072226.png)

Bu satır, sunucudan gelen yanıtı kontrol eder. **readyState** değeri 4'e eşit olduğunda, istek tamamlanmış demektir. **status** ise HTTP durum kodudur ve 200, işlemin başarıyla tamamlandığını gösterir.

5. **Yanıtı JSON Olarak Parse Etme**:

![Pasted image 20241213072247.png](/img/user/resimler/Pasted%20image%2020241213072247.png)

Burada, sunucudan gelen **yanıt** (**responseText**) JSON formatında olabilir. **JSON.parse** fonksiyonu, bu JSON verisini **JavaScript nesnesi** haline getirir. Bu sayede veriyi kolayca işleyebiliriz.

6. **Kullanıcı Adı Girişi Almak ve Temizlemek**:

`var username = document.getElementById("usernameInput").value;` 

`username = username.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');`

- **document.getElementById("usernameInput")**: Bu, HTML sayfasındaki **usernameInput** id'sine sahip **input** (kullanıcı adı) alanını bulur.
- **.value**: Kullanıcının bu alana yazdığı değeri alır.
- Bu satırda ayrıca, kullanıcıdan gelen veriyi **güvenli hale getirmek** için bazı karakterler **HTML özel karakterleri** ile değiştirilir. Örneğin:
    - **&** yerine **&**,
    - **<** yerine **<** gibi. Bu işlem, kullanıcıların HTML veya JavaScript kodu eklemelerini engellemeye yardımcı olur (XSS saldırılarından korur).

7. **Yanıtı Görsel Olarak Kullanıcıya Gösterme**:

![Pasted image 20241213072414.png](/img/user/resimler/Pasted%20image%2020241213072414.png)

Burada, **usernameHelp** id'sine sahip bir başka HTML öğesi bulunur. Bu öğe, kullanıcının girdiği kullanıcı adı ile ilgili bilgi gösterecek alan olacaktır (örneğin: "Kullanıcı adı kullanılabilir" veya "Kullanıcı adı alınmış").


8. **Sunucudan Gelen Duruma Göre Kullanıcıya Bilgi Verme**:

`if (json['status'] === 'available') {`

`usernameHelp.innerHTML = "<span style='color:green'>The username '" + username + "' is <b>available</b></span>";`

`} else {`

`usernameHelp.innerHTML = "<span style='color:red'>The username '" + username + "' is <b>taken</b>, please use a different one</span>";`

`}`

Burada, sunucudan dönen JSON verisi üzerinden **status** değerine bakılır.

- Eğer **status** 'available' (kullanılabilir) ise:
    - **usernameHelp** öğesinin **innerHTML** özelliği, yani içeriği, "Kullanıcı adı kullanılabilir" şeklinde yeşil renkli bir mesajla güncellenir.
- Eğer **status** 'taken' (alınmış) ise:
    - **usernameHelp** öğesinin içeriği, "Kullanıcı adı alınmış, lütfen başka bir tane kullanın" şeklinde kırmızı renkli bir mesajla güncellenir.



9. **GET İsteği Gönderme**:

`xhr.open("GET", "/api/check-username.php?u=" + document.getElementById("usernameInput").value, true);`

`xhr.send();`

- **xhr.open("GET", ...)**: Burada, **GET** türünde bir istek gönderileceği belirtiliyor. İstek, **/api/check-username.php** adresine gönderilecek ve kullanıcı tarafından girilen **username** değeri URL'ye eklenerek sunucuya iletiliyor.
- **xhr.send()**: Bu satır, isteği sunucuya gönderir.



### Özet:

Bu fonksiyon, kullanıcının girdiği kullanıcı adının kullanılabilir olup olmadığını kontrol eder. Kullanıcı adı **alınmamışsa** yeşil renkte "Kullanıcı adı kullanılabilir" mesajı, **alınmışsa** kırmızı renkte "Kullanıcı adı alınmış" mesajı gösterilir. Bu işlem, sayfa yeniden yüklenmeden asenkron olarak gerçekleşir.

Bu tür işlemler genellikle daha kullanıcı dostu deneyimler sağlamak için kullanılır, çünkü kullanıcı hemen sonuçları görür ve sayfa yeniden yüklenmesine gerek yoktur.



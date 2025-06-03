---
{"dg-publish":true,"permalink":"/baglantilar/kodun-detayi/"}
---


### **1. `const app = express();`**

- Burada **`express`** adlı bir çerçeve kullanılıyor. Bu, Node.js ile hızlı ve kolay web sunucuları oluşturmanıza olanak sağlar.
- **`express()`** ile bir **uygulama nesnesi** oluşturuluyor. Bu nesne, sunucunun "kalbi" gibidir. Tüm ayarları ve işlemleri bu nesne üzerinden tanımlarsınız.

---

### **2. `const port = parseInt("5000");`**

- **`port`**, uygulamanın çalışacağı bağlantı noktasını (port) temsil eder. Burada `"5000"` yazılmış ve bu metin (`string`) bir sayı (`integer`) haline getirilmiş.
    - **Port Numarası:** Bir sunucu, bilgisayarınızın belirli bir kapısı gibidir. Örneğin, `5000` numaralı kapıda (portta) bir hizmet çalışacak.
    - **`parseInt` Fonksiyonu:** Bir metni (örneğin `"5000"`) sayıya dönüştürür.

---

### **3. `app.use(bodyParser.json());`**

- **Body parser**, uygulamanıza gelen isteklerin (request) verilerini işlemeye yarar.
- Örneğin, bir istemci (client), sunucuya bir JSON verisi (JavaScript Object Notation) gönderirse, bu veri otomatik olarak okunabilir hale getirilir.
    - **Ne İşe Yarar?**
        
    ![Pasted image 20241209213012.png](/img/user/resimler/Pasted%20image%2020241209213012.png)
    Gibi bir veriyi anlamak ve kullanmak için bu kod gereklidir.

### **4. `app.use("/api/auth", authRoutes);`**

- Bu kod, bir **API rotası** tanımlar. Rotalar, istemcinin (örneğin tarayıcı) sunucudan hangi sayfayı veya işlemi çağıracağını belirler.
- **Detaylı Anlamı:**
    - **`/api/auth`**: Bu, URL'nin bir kısmıdır. Örneğin, bir kullanıcı şu adresi ziyaret eder:

![Pasted image 20241209213034.png](/img/user/resimler/Pasted%20image%2020241209213034.png)
- Bu durumda, burada tanımlanan **`authRoutes`** kodları çalışır.
- **`authRoutes`**: `authRoutes` adlı bir dosya veya modül içerisine yazılmış rotaları (işlemleri) kullanır. Bu dosya, genellikle kullanıcı kimlik doğrulama (login, register) gibi işlevler içerir.


### **5. `app.use("/api/service", serviceRoutes);`**

- Bu da başka bir rota tanımıdır.
- **Detaylı Anlamı:**
    - **`/api/service`**: İstemci bu URL'yi ziyaret ettiğinde, **`serviceRoutes`** adlı modül içindeki işlemler çalıştırılır.
    - **`serviceRoutes`**: Bu modül, genellikle bir hizmetle ilgili (örneğin, veritabanından bilgi almak veya bilgi göndermek) işlevler içerir.
    - 
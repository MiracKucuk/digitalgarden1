---
{"dg-publish":true,"permalink":"/baglantilar/kodun-detayi-2/"}
---

1. **`app.listen(port, callback)`**
    
    - **Ne işe yarar?** Bu, uygulamanın belirli bir port numarası üzerinden gelen HTTP isteklerini dinlemeye başlamasını sağlar.
    - **Port nedir?** Port, bilgisayarınıza gelen ağ trafiğini yönlendiren bir kapıdır. Örneğin, port `3000` üzerinden çalışan bir uygulama, tarayıcınızda `http://localhost:3000` adresinden ulaşılabilir olur.
2. **`port`**
    
    - Bu bir değişkendir ve uygulamanın hangi port numarasını dinleyeceğini belirtir.
    - Örneğin: Eğer `port = 3000` ise, tarayıcıya şu şekilde yazabilirsiniz: `http://localhost:3000`.
3. **Callback Fonksiyonu (`() => { ... }`)**
    
    - **Ne işe yarar?** `app.listen` çalıştığında, yani sunucu başarılı bir şekilde başlatıldığında, bu fonksiyon tetiklenir. Callback, bir işin tamamlandığında ne yapılacağını tanımlar.
    - **Bu örnekte ne yapılıyor?** Sunucu başlatıldığında, konsola (terminal) bazı mesajlar yazdırılıyor.
4. **`console.log(...)`**
    
    - **Ne yapar?** JavaScript'in `console.log` fonksiyonu, bir mesajı konsola yazdırır. Bu, genelde hata ayıklama (debugging) veya bilgi verme amacıyla kullanılır.
    - **Mesajlar:**
        - **`⚡ [server]: Server is running at http://localhost:${port}`**
            - Bu mesaj, sunucunun çalışmaya başladığını ve hangi adreste (URL) dinlediğini bildirir.
            - **`localhost` nedir?**
                - Bilgisayarınızda çalışan yerel bir sunucudur. Harici bir ağ bağlantısına ihtiyaç duymaz.
        - **`⚡ [api]: APIs are running at http://localhost:${port}/api`**
            - Bu mesaj, uygulamanın API'lerinin hangi URL'de olduğunu belirtir. API'ler, uygulamanın başka programlarla iletişim kurmasını sağlayan bir yapıdır.

---

### **Kodun Çalışma Süreci**

1. **Sunucu Başlatılır**:
    
    - Kod çalıştırıldığında, `app.listen` fonksiyonu belirtilen port üzerinden gelen istekleri dinler.
2. **Başlatma Başarılı Olduğunda Mesaj Gönderilir**:
    
    - Callback fonksiyonu çalışır ve konsola mesajlar yazılır
    
![Pasted image 20241209213632.png](/img/user/resimler/Pasted%20image%2020241209213632.png)

### **Özet**

Bu kod, Express.js kullanarak bir web sunucusunu başlatır. Tarayıcınızda `http://localhost:3000` adresine giderek bu sunucuyla iletişim kurabilirsiniz. `console.log` fonksiyonu ise uygulamanın çalıştığını ve nereden erişileceğini bildirir. Bu, bir Node.js uygulamasının temel çalışma prensiplerinden biridi

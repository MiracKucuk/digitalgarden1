---
{"dg-publish":true,"permalink":"/baglantilar/chatgpt-origin-aciklama/"}
---

### **Same-Origin Politikası (SOP)**

Web tarayıcılarında uygulanan bir güvenlik mekanizmasıdır ve aşağıdaki şekilde özetlenebilir:

---

#### **Amaç ve İşlev**

1. **Cross-Origin Erişimi Önler:**
    
    - Kötü niyetli sitelerin başka bir orijinden bilgi sızdırmasını engeller.
    - Farklı orijinlere yapılan istek türlerini kısıtlar.
2. **JavaScript'in Kapsamını Sınırlar:**
    
    - Bir kaynaktaki (origin) JavaScript kodu, farklı bir kaynaktan veri okuyamaz.

---

#### **Orijin (Origin) Tanımı**

Orijin, bir URL’nin şu üç bileşeninden oluşur:

- **Scheme (Protokol):** Örneğin, `http` veya `https`.
- **Host (Alan Adı):** Örneğin, `hackthebox.com`.
- **Port:** Örneğin, `80` veya `443`.

---

#### **Same-Origin Politikası Nasıl Çalışır?**

- İki URL'nin aynı orijini paylaşması için **scheme**, **host** ve **port** değerlerinin **tamamen aynı** olması gerekir.
- Bu bileşenlerden herhangi biri farklıysa, SOP uygulanır ve erişim engellenir.

---

#### **Örnekler**

1. **Farklı Orijinler:**
    
    - **[http://example.com](http://example.com)** ve **[https://example.com](https://example.com)**
        - **Scheme farklılığı** nedeniyle farklı orijin.
    - **[https://academy.hackthebox.com](https://academy.hackthebox.com)** ve **[https://hackthebox.com](https://hackthebox.com)**
        - **Host farklılığı** nedeniyle farklı orijin.
2. **Aynı Orijin:**
    
    - **[https://hackthebox.com](https://hackthebox.com)** ve **[https://hackthebox.com:443](https://hackthebox.com:443)**
        - **Scheme**, **host** ve **port** aynı olduğundan aynı orijin.
        - **443** HTTPS'nin varsayılan portu olduğu için eşleşme sağlanır.

---

#### **Bypass ve Güvenlik Riskleri**

- Web tarayıcılarının SOP'u uygularken kullandığı mekanizmalarda açıklar olabilir.
- Bu tür açıklar, SOP'un bypass edilmesine yol açarak yüksek önemde güvenlik açıkları oluşturabilir:
    - **Cross-Origin Resource Sharing (CORS) yanlış yapılandırmaları.**
    - **Web tarayıcılarındaki hatalar veya zafiyetler.**

---

#### **Önemli Notlar**

- SOP, modern web güvenliğinin temel yapı taşlarından biridir.
- Doğru yapılandırılmadığında, hassas verilerin başka bir siteye sızdırılması gibi tehlikeli sonuçlara yol açabilir.
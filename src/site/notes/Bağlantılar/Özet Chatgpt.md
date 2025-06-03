---
{"dg-publish":true,"permalink":"/baglantilar/oezet-chatgpt/"}
---

- **Domain Çözümleme Yöntemi:**
    
    - Domain adları **sağdan sola doğru** çözülür.  
        Örneğin, `www.example.com` için sıralama şu şekildedir:
        1. **Top-level domain (TLD):** `.com`
        2. **Second-level domain (SLD):** `example`
        3. **Subdomain:** `www`
- **DNS Zone Sahipliği:**
    
    - Bir **DNS zone** sahibi, kendisine ait olan DNS zone'unu ve o zone'a bağlı tüm subdomain'leri yönetme yetkisine sahiptir.  
        Örneğin, `inlanefreight.com` zone'unun sahibi, bu zone'a ait tüm subdomain'leri (örn. `www.inlanefreight.com`, `api.inlanefreight.com`) serbestçe kontrol edebilir.
- **DNS Zone Yönetim Hakları:**
    
    - Zone sahibi şu işlemleri yapabilir:
        - **Yeni subdomain'ler eklemek:** Örneğin, `app.inlanefreight.com` gibi yeni bir subdomain oluşturabilir.
        - **Mevcut subdomain'leri kaldırmak:** Örneğin, `old.inlanefreight.com` girişini silebilir.
        - **Domain IP adreslerini değiştirmek:**
            - `www.inlanefreight.com` subdomain'inin sabah **1.2.3.4** IP adresine, gece ise **5.6.7.8** IP adresine çözülmesini sağlayabilir.
- **DNS ile Load Balancing:**
    
    - Zone sahibi, domain IP adreslerini farklı zamanlarda farklı IP adreslerine çözerek **load balancing** (yük dengeleme) yapabilir.  
        Örneğin:
        - **Gündüz:** Trafiği bir sunucuya (**1.2.3.4**) yönlendirebilir.
        - **Gece:** Trafiği başka bir sunucuya (**5.6.7.8**) aktarabilir.
- **Serbest IP Atama Yetkisi:**
    
    - Zone sahibi, domain'lerini **herhangi bir IP adresine** yönlendirebilir. Bu IP adresinin:
        - Zone sahibiyle ilişkili olup olmaması,
        - Veya bu IP adresine sahip olan sistemin zone sahibiyle aynı yapıda olup olmaması fark etmez.
- **DNS Rebinding Bağlantısı:**
    
    - DNS zone'larının bu serbest yapılandırma yeteneği, **DNS Rebinding saldırıları** için önemlidir.
        - Örneğin, bir domain'in IP adresi, saldırganın belirlediği IP adresine çözümlenecek şekilde yapılandırılabilir. Bu durum, saldırganın hedef sistemdeki kısıtlamaları aşmasına yardımcı olabilir.
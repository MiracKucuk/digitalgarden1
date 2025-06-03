---
{"dg-publish":true,"permalink":"/baglantilar/nt-lan-manager/"}
---

**NTLM Protokol Genel Bakış**

- **NTLM Nedir?**  
    **NTLM (NT LAN Manager)**, uzaktaki kullanıcıları kimlik doğrulamak ve isteğe bağlı olarak oturum güvenliği sağlamak için kullanılan bir güvenlik protokolleri ailesidir.
    
- **Nasıl Çalışır?**  
    NTLM, bir **challenge-response** (meydan okuma-cevap) kimlik doğrulama protokolüdür:
    
    1. **Server**, **client**’a bir **challenge** mesajı gönderir.
    2. **Client**, bu **challenge**, kullanıcının şifresi ve diğer bilgileri kullanarak bir **response** oluşturur.
    3. **Server**, bu cevabı, kullanıcı bilgilerini doğrulamak için bir hesap veritabanına danışarak doğrular.
- **Protokol Yapısı:**  
    NTLM, bağımsız bir ağ protokolü katmanı değildir; bunun yerine bir uygulama protokolüne gömülü bir **subroutine** (alt program) olarak çalışır. NTLM mesajları, uygulamanın protokol mesajları içinde **byte array** olarak iletilir. Bu mesajların nasıl kodlandığı, çerçevelendiği ve taşındığı, çağıran uygulama protokolü tarafından belirlenir.
    
- **Uygulama:**  
    NTLM, bir **function library** (fonksiyon kütüphanesi) olarak çalışır. Çağıran uygulama protokolünden parametreler alır ve kimlik doğrulama mesajlarını döner. Bu mesajlar daha sonra uygulamanın protokol mesajlarına entegre edilir.
    
- **İki Temel Varyant:**
    
    1. **Connection-oriented:** Mesaj alışverişi için sürekli bir bağlantıya dayanır.
    2. **Connectionless (datagram):**
        - Uygulama protokolü tarafından sağlanan bir **sequence number** kullanır; dahili bir sıralama numarası tutmaz.
        - **Session key**'ler, **client initialization** sırasında oluşturulur.
        - **NEGOTIATE_MESSAGE** gönderilemez.
- **Üç Versiyon:**  
    NTLM'in üç versiyonu vardır: **LM**, **NTLMv1** ve **NTLMv2**. Farklılıklar, **response field**'lerin nasıl hesaplandığı ve hangi alanların kullanıldığı ile ilgilidir.
    
- **İsteğe Bağlı Oturum Güvenliği:**  
    NTLM, mesaj bütünlüğü (**signing**) ve gizliliği (**sealing**) sağlayarak **session security** hizmeti de sunabilir.
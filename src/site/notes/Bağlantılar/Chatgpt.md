---
{"dg-publish":true,"permalink":"/baglantilar/chatgpt/"}
---

### Parametrelerin Açıklaması:

1. **.\Rubeus.exe:**
    
    - Rubeus, Kerberos tabanlı güvenlik testleri için kullanılan bir araçtır.
    - Bu komut, mevcut dizindeki `Rubeus.exe` dosyasını çalıştırır.
2. **createnetonly:**
    
    - Windows API'sini kullanarak bir **"Net-Only" oturum** oluşturur.
    - "Net-Only" oturumu, yerel sistem üzerinde herhangi bir kullanıcı kimliği doğrulaması yapılmadan, belirli bir program için oturum başlatmanızı sağlar.
    - Bu, genellikle kimlik doğrulama bilgilerini manuel olarak eklemek için kullanılır.
3. **/program:powershell.exe:**
    
    - Hangi programın "Net-Only" modunda başlatılacağını belirtir.
    - Burada `powershell.exe` başlatılacaktır. Bu, başlatılan oturumda kimlik doğrulama bilgilerini eklemek veya test etmek için bir PowerShell oturumu sağlar.
4. **/show:**
    
    - Oturumu başlattıktan sonra, yeni oturumun ID'sini ve bağlam bilgilerini kullanıcıya gösterir.
    - Bu, oluşturulan oturumu tanımlamak ve daha fazla işlem yapmak için kullanışlıdır.

### Ne İşe Yarar?

- **"Net-Only" Oturumun Avantajı:**
    
    - Bu mod, oturumu başlatırken sistemin varsayılan kullanıcı kimlik doğrulama mekanizmalarını (örneğin, NTLM veya Kerberos) kullanmaz.
    - Bunun yerine, manuel olarak belirttiğiniz kimlik doğrulama bilgileriyle çalışır.
- **Saldırı Amaçlı Kullanımı:**
    
    - "Net-Only" modunda başlatılan bir oturum, test amaçlı olarak kimlik doğrulama bilgilerini (örneğin, Kerberos biletlerini) enjekte etmek için kullanılabilir.
    - Bu, genellikle lateral hareket (yatay ağ geçişi) veya belirli sistemlere erişim testi yapmak için kullanılır.

### Örnek Senaryo:

1. Komut çalıştırıldığında, bir **PowerShell oturumu** başlatılır. Ancak bu oturum yerel kimlik doğrulaması yapmaz.
2. Ardından, başka bir komutla bu oturuma **Kerberos biletleri** veya kimlik bilgileri enjekte edilebilir. Örneğin:

![Pasted image 20241205231600.png](/img/user/resimler/Pasted%20image%2020241205231600.png)

Bu bilet daha sonra oluşturulan "Net-Only" oturumunda kullanılabilir.


### Güvenlik Perspektifi:

- **Penetrasyon Testi ve Red Teaming:**  
    Bu komut, saldırganların bir ağda kimlik bilgisi enjekte ederek test yapmasına olanak tanır.
- **Savunma Açısından:**  
    Sistem yöneticileri, bu tür işlemleri algılayabilmek için oturum oluşturma etkinliklerini ve Kerberos bilet işlemlerini izlemelidir.
    
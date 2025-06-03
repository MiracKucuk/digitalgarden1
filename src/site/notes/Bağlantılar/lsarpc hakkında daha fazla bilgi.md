---
{"dg-publish":true,"permalink":"/baglantilar/lsarpc-hakkinda-daha-fazla-bilgi/"}
---

**LSARPC (Local Security Authority Remote Procedure Call)**, Windows işletim sistemlerinde kullanılan bir protokoldür ve **Local Security Authority Subsystem Service (LSASS)** ile client'lar arasında iletişim sağlamak için kullanılır.

### LSARPC'nin Amacı

- LSARPC, bir Windows sistemindeki güvenlik politikalarını ve kullanıcı bilgilerini yönetmek için kullanılan **LSASS** servisine erişim sağlar.
- RPC (Remote Procedure Call) mekanizmasını kullanarak, güvenlik verilerine uzak erişim sunar.

### LSARPC'nin Kullanım Alanları

1. **Kullanıcı ve Grup Bilgileri**:
    
    - Sistem üzerindeki kullanıcı hesaplarını ve grupları sorgulamak için kullanılır.
    - Örneğin, bir saldırgan bu protokolü kullanarak hedef sistemdeki kullanıcı adlarını öğrenebilir.
2. **Erişim Hakları ve Politikalar**:
    
    - Kullanıcıların sistemdeki dosyalara, dizinlere veya kaynaklara erişim haklarını öğrenmek için kullanılabilir.
    - Güvenlik politikalarını sorgulamak veya güncellemek için yönetici yetkisiyle kullanılabilir.
3. **Domain İletişimi**:
    
    - Domain ortamlarında, bir domain controller üzerindeki güvenlik bilgilerini sorgulamak veya değiştirmek için kullanılabilir.

### LSARPC'nin Güvenlik İle İlgisi

- **LSASS**, kritik güvenlik bilgilerini (şifre hash'leri, access politikaları vb.) barındırdığı için LSARPC'nin korunması çok önemlidir.
- Protokoldeki açıklar, saldırganların yetkisiz erişim sağlamasına ve hassas bilgileri ele geçirmesine yol açabilir.

### Saldırı Yöntemleri

- **LsaQueryInformationPolicy** gibi LSARPC fonksiyonları kullanarak domain adları, SID'ler (Security Identifier) ve sistem bilgileri gibi hassas veriler elde edilebilir.
- **Pass-the-Hash** veya **Pass-the-Ticket** saldırıları için ön bilgi toplamak amacıyla kullanılabilir.

### Kısaca

**LSARPC**, Windows sistemlerinde güvenlik yönetimi ve politikalarıyla ilgili işlemler yapmak için kullanılan bir protokoldür. Ancak, sistemlerinizi bu protokolden kaynaklanabilecek güvenlik tehditlerine karşı korumak için doğru yapılandırmalar ve güncellemeler yapmak önemlidir.
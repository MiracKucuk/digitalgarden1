---
{"dg-publish":true,"permalink":"/baglantilar/double-hop/"}
---

**Double-Hop Sorunu (Çift Atlama Sorunu):**

Windows kimlik doğrulama protokollerindeki bir sınırlamadır. Bu sorun, bir kullanıcı bir makineye (örneğin, **Makine A**) uzaktan bağlandığında ve bu bağlantı üzerinden başka bir makineye (**Makine B**) erişmeye çalıştığında ortaya çıkar. Kullanıcının kimlik bilgileri **Makine B’ye** geçmez, bu da yetkilendirme sorunlarına neden olur.

### Daha Basit Bir Örnek:

1. **Kullanıcı → Makine A:**  
    Kullanıcı, kimlik bilgilerini kullanarak **Makine A’ya** başarıyla bağlanır (örneğin, RDP veya WinRM ile).
    
2. **Makine A → Makine B:**  
    Kullanıcı **Makine A’dan** komut çalıştırarak veya dosya paylaşımı gibi yollarla **Makine B’ye** erişmeye çalışır. Ancak kullanıcı kimlik bilgileri **Makine B’ye** iletilmez.
    

**Sonuç:**  
Makine B, kimlik doğrulama bilgisi olmadığı için kullanıcıyı yetkilendiremez. Bu, bir **"Double-Hop Sorunu"** olarak adlandırılır.

---

### Neden Ortaya Çıkar?

Double-Hop sorunu, protokollerin kimlik bilgilerini geçirme şekliyle ilgilidir:

- **NTLM (ve NTLMv2):** Kimlik bilgileri oturuma bağlıdır ve birden fazla makine üzerinden iletilemez.
- **Kerberos:** Daha güvenli olduğu için, kimlik bilgilerini doğrudan taşımak yerine biletler kullanır. Ancak biletler genellikle belirli bir hedef için geçerlidir ve ikinci atlamayı desteklemez.

---

### Çözüm Yöntemleri:

1. **PsExec (Double-Hop Sorununu Aşar):**  
    PsExec, kimlik bilgilerini hedef makineye ilettiği için **etkileşimli bir oturum açar** ve bu sorun ortadan kalkar.
    
2. **Kerberos Delegasyonu:**  
    Kerberos’un "delegation" özelliğiyle, kullanıcının kimlik bilgileri veya biletleri başka bir makineye yetki vermek üzere iletilebilir.
    
3. **CredSSP (Credential Security Support Provider):**  
    Bu yöntemle, kullanıcı kimlik bilgileri güvenli bir şekilde uzak makineye gönderilir ve ikinci makineye erişim sağlanabilir.
    
4. **Restricted Admin Mode:**  
    **`mstsc /restrictedAdmin`** komutuyla kimlik bilgileri taşınmaz; sadece bilet kullanılarak yetkilendirme yapılır.
    

---

### PsExec Neden Double-Hop'u Çözer?

PsExec, kimlik bilgilerini doğrudan hedef makineye (**Makine B’ye**) aktarır ve bu makinede etkileşimli bir oturum açar. Bu sayede, ikinci makinede oturum kimlik bilgileriyle işlem yapmak mümkün olur.

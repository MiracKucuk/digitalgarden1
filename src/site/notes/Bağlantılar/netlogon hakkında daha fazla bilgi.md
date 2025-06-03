---
{"dg-publish":true,"permalink":"/baglantilar/netlogon-hakkinda-daha-fazla-bilgi/"}
---

**NETLOGON**, Active Directory Domain Services (AD DS) ortamında kullanılan bir sistem paylaşımıdır ve **domain tabanlı oturum açma** işlemlerinde önemli bir role sahiptir. İşte NETLOGON paylaşımının detaylı açıklaması:

---

### **NETLOGON Nedir?**

- **NETLOGON**, **Domain Controller** (DC) üzerinde oturum açma (logon) işlemlerini destekleyen bir servisdir.
- Domain'e bağlı clientların ve server'ların, oturum açma kimlik doğrulaması için Domain Controller ile iletişim kurmasını sağlar.
- **Active Directory ortamlarında** kritik bir rol oynar ve oturum açma işlemlerine yardımcı olur.

---

### **NETLOGON'un Görevleri**

1. **Oturum Açma ve Kimlik Doğrulama**:
    
    - Client sistem, domain ortamında oturum açarken NETLOGON servisini kullanır.
    - Client, Domain Controller ile iletişim kurarak kimlik bilgilerini doğrular.

2. **Domain Güvenlik Kanalı**:
    
    - Domain üyeleri ile Domain Controller arasında güvenli bir kanal kurar.
    - Bu güvenli kanal, domain'e katılmış cihazlar ve DC arasında şifreli iletişim sağlar.

3. **Logon Script Paylaşımı**:
    
    - Domain kullanıcılarına oturum açma sırasında çalıştırılacak **logon script'leri** sağlar.
    - Örnek: Yazıcı haritalama veya ağ driver ekleme gibi işlemleri gerçekleştiren script'ler.

4. **Replication Desteği**:
    
    - Active Directory'nin birden fazla Domain Controller arasında doğru şekilde çoğaltılmasını (replication) destekler.
    - **SYSVOL** diziniyle birlikte çalışarak grup politikalarının ve script'lerin dağıtımında yardımcı olur.

---

### **NETLOGON ve SMB**

- **NETLOGON**, genellikle **SMB** protokolü üzerinden erişilir.
- SMB paylaşımına şu şekilde erişilebilir:

```
\\<DC_IP>\NETLOGON
```

- Bu paylaşımda genellikle **logon script'leri** ve **grup politikası dosyaları** bulunur.

---

### **NETLOGON ve RPC İlişkisi**

NETLOGON, named pipes ve **RPC** ile de sıkı bir ilişki içindedir:

- NETLOGON servisi, aşağıdaki anamed pipes üzerinden çalışır:
    - **`\\pipe\netlogon`**
- Bu pipe, domain ortamında kimlik doğrulama ve oturum açma taleplerini işlemek için kullanılır.
- Örneğin:
    - Domain'e katılım işlemleri (domain join).
    - Güvenli kanal oluşturma.

---

### **NETLOGON ve Güvenlik**

NETLOGON, saldırganlar tarafından sıkça hedef alınan bir bileşendir. Aşağıdaki güvenlik konularına dikkat edilmelidir:

1. **ZeroLogon (CVE-2020-1472)**:
    
    - NETLOGON'daki bir güvenlik açığı, saldırganların kimlik doğrulama olmadan DC'ye erişmesine izin verir.
    - Bu güvenlik açığı, özellikle **güvenli kanal** mekanizmasını hedef alır.
    - Düzeltme: Sistemler düzenli olarak güncellenmeli ve yama uygulanmalıdır.

2. **Doğru Erişim Kontrolü**:
    
    - NETLOGON paylaşımına yalnızca yetkili kullanıcıların erişimi sağlanmalıdır.
    - Yanlış yapılandırılmış bir NETLOGON paylaşımı, sistemin yetkisiz erişimlere açık hale gelmesine neden olabilir.

### **NETLOGON'un Pratik Kullanımı**

#### Logon Script'lerini Görüntüleme:

```
dir \\<DC_IP>\NETLOGON
```

- Bu komut, DC'deki NETLOGON paylaşımında bulunan tüm logon script'lerini listeler.

#### Kimlik Doğrulama:

- NETLOGON, clientlerin domain ortamına kimlik bilgilerini doğrulamak için kullanılır. Örneğin:
    - Client sistem, Domain Controller ile iletişime geçer.
    - DC, Clientin kimlik doğrulama isteğini işleyerek yanıt verir.

---

### **Özet**

- **NETLOGON**, Active Directory ortamında oturum açma, güvenli kanal oluşturma ve logon script yönetimi gibi kritik işlevler sağlar.
- **RPC ve SMB protokolleri** üzerinden çalışarak clientlerin ile Domain Controller arasında iletişim kurar.
- Güvenlik açısından düzenli güncellemeler ve doğru yapılandırmalar yapılması büyük önem taşır.


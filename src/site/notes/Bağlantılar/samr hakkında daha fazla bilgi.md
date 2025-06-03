---
{"dg-publish":true,"permalink":"/baglantilar/samr-hakkinda-daha-fazla-bilgi/"}
---


### **SAMR (Security Account Manager Remote Protocol) Nedir?**

**SAMR** (Security Account Manager Remote Protocol), bir Windows ortamında kullanıcı hesapları, gruplar ve diğer securty nesnelerini (security objects) yönetmek için kullanılan bir protokoldür. Bu protokol, **Security Account Manager (SAM)** veritabanına erişimi sağlar ve genellikle **RPC (Remote Procedure Call)** protokolü üzerinden çalışır.

---

### **SAMR'nin Görevleri**

SAMR, aşağıdaki işlemleri gerçekleştirmek için kullanılır:

1. **Hesap Bilgilerini Sorgulama**:
    
    - Bir domain'deki veya local sistemdeki kullanıcılar, gruplar ve diğer hesapların bilgilerini sorgulamak için kullanılır.
    - Örnek: Kullanıcı adı, SID (Security Identifier), grup üyelikleri.

2. **Hesap Yönetimi**:
    
    - Kullanıcı ve grup hesapları oluşturulabilir, güncellenebilir veya silinebilir.
    - Örnek: Yeni bir kullanıcı oluşturma veya bir kullanıcının parolasını sıfırlama.

3. **Erişim Kontrolü ve Yetkilendirme**:
    
    - Bir kullanıcının veya grup üyelerinin hangi kaynaklara erişebileceğini belirlemek için SAMR kullanılır.
    - **Access Control Lists (ACLs)** ile etkileşir.

4. **Domain Bilgileri**:
    
    - Bir domain'deki genel bilgileri sorgulamak için kullanılır.
    - Örnek: Domain adı, domain SID.

---

### **SAMR ve SMB**

- SAMR genellikle **SMB** protokolüyle birlikte çalışır ve **RPC** üzerinden **named pipe** kullanır.
- Aşağıdaki named pipe kullanılarak SAMR'ye erişilebilir:

```
\\pipe\samr
```

### **SAMR (Security Account Manager Remote Protocol) Nedir?**

**SAMR** (Security Account Manager Remote Protocol), bir Windows ortamında kullanıcı hesapları, gruplar ve diğer security objects yönetmek için kullanılan bir protokoldür. Bu protokol, **Security Account Manager (SAM)** veritabanına erişimi sağlar ve genellikle **RPC (Remote Procedure Call)** protokolü üzerinden çalışır.

---



### **SAMR'nin Teknik Detayları**


1. **Portlar**:
    
    - **SAMR**, genellikle **TCP 445** (SMB) veya **TCP 135** (RPC Endpoint Mapper) portu üzerinden çalışır.
    - RPC, dinamik portlar kullanabilir; bu nedenle bağlantı sırasında belirlenen port numarası değişebilir.

2. **Veritabanı Erişimi**:
    
    - SAM protokolü, kullanıcı ve grup bilgilerini depolayan **SAM veritabanına** erişim sağlar.
    - SAM veritabanı, local güvenlik bilgilerini içeren bir yapılandırma dosyasıdır.

3. **Sorgulama Örnekleri**:
    
    - Kullanıcı hesaplarının listelenmesi.
    - Bir kullanıcının SID'inin alınması.
    - Grup üyeliklerinin sorgulanması.

### **SAMR ve Güvenlik**

1. **Yetkisiz Erişim**:
    
    - Yanlış yapılandırılmış SAMR erişimi, saldırganların bir sistemdeki kullanıcı ve grup bilgilerini listelemesine olanak tanır.
    - Örnek: **Null Session** saldırıları, SAMR'nin doğru şekilde güvenlik altına alınmadığı sistemlerde kullanılabilir.

2. **Bilgi Toplama**:
    
    - Saldırganlar, SAMR protokolünü kullanarak hedef sistem hakkında bilgi toplayabilir. Bu bilgiler daha sonra başka saldırılar için kullanılabilir:
        - **SID bilgileri**.
        - **Administrator hesabı** gibi yüksek öncelikli kullanıcıların varlığı.

3. **Örnek Saldırı**:
    
    - **LSASS (Local Security Authority Subsystem Service)** üzerinden credential dumping işlemi:
        - SAMR protokolü, kullanıcı bilgilerini toplayarak bu tür saldırılara katkıda bulunabilir.

---

### **SAMR'nin Kullanım Senaryoları**

1. **Pentest ve Güvenlik Testleri**:
    
    - SAMR, bir sistem veya domain'deki kullanıcı ve grup bilgilerini sorgulamak için pentester'lar tarafından kullanılır.
    - Örnek bir sorgu aracı: **==rpcclient==**.

2. **Bilgi Toplama**:
    
    - Kullanıcı ve grup bilgileri aşağıdaki komutla elde edilebilir


```
rpcclient -U "" <IP>
```

* Bu komutla bağlantı kurulduktan sonra **enumdomusers** gibi komutlar çalıştırılarak kullanıcı bilgileri alınabilir:

```
rpcclient> enumdomusers
```


3. **Local Yönetim**:

- Sistem yöneticileri, kullanıcı hesaplarını ve gruplarını yönetmek için SAMR protokolünü kullanır.


### **SAMR'nin Saldırı Yüzeyi**

1. **Null Session ile Sorgulama**:
    
    - Bazı durumlarda, kimlik doğrulama olmadan SAMR'ye erişim sağlanabilir. Bu saldırı, sistem hakkında bilgi toplamak için kullanılır.
    - Null Session bağlantısı:

```
rpcclient -U "" <IP>
```


2. **Hardening Önlemleri**:

- **Null Session erişimini devre dışı bırakmak**.
- SAMR sorgularını yalnızca yetkili kullanıcılarla sınırlandırmak.
- RPC ve SMB bağlantılarını şifrelemek ve güvenlik duvarıyla korumak.

### **SAMR'nin Önemi**

- SAMR, Windows sistemlerinde kullanıcı ve grup bilgilerini yönetmek için temel bir protokoldür.
- Ancak, doğru şekilde yapılandırılmadığında, saldırganlar için bilgi toplama ve sistem açıklarını istismar etme açısından cazip bir hedef olabilir.
- Güvenlik önlemleri alınarak bu tür saldırılar engellenebilir.



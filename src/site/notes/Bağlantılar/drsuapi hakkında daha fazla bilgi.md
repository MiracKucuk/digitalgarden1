---
{"dg-publish":true,"permalink":"/baglantilar/drsuapi-hakkinda-daha-fazla-bilgi/"}
---

### **DRSUAPI (Directory Replication Service Remote Protocol API) Nedir?**

**DRSUAPI**, Windows Active Directory'nin (AD) **dizin çoğaltma (replication)** işlemlerini gerçekleştirmek için kullanılan bir protokoldür. **Directory Replication Service Remote Protocol** (DRS-R) adıyla da bilinir. Temel amacı, bir **Domain Controller** (DC) üzerindeki Active Directory veritabanındaki değişiklikleri diğer DC'lere çoğaltmaktır.

Bu protokol, Active Directory'nin tutarlı ve güncel kalmasını sağlamak için kullanılır. DRSUAPI, replication işlemlerinde RPC (Remote Procedure Call) mekanizmasını kullanır ve genellikle aşağıdaki named pipe üzerinden çalışır:

```
\\pipe\drsuapi
```

### **DRSUAPI'nin Görevleri**

DRSUAPI protokolü aşağıdaki görevleri yerine getirir:

1. **Active Directory Veri Çoğaltma**:
    
    - Domain Controller'lar arasında kullanıcı hesapları, gruplar, politikalar ve diğer dizin verilerinin senkronize edilmesini sağlar.

2. **Veri Sorgulama**:
    
    - Active Directory veritabanından belirli bilgileri sorgulamak için kullanılabilir.
    - Örneğin, kullanıcı hesaplarının veya grup üyeliklerinin alınması.

3. **Parola Hash'lerinin Alınması (NTDS.DIT)**:
    
    - DRSUAPI, saldırganlar tarafından Active Directory veritabanından (NTDS.dit) kullanıcıların parola hash'lerini almak için kullanılabilir.

4. **Schema ve Konfigürasyon Bilgileri**:
    
    - Active Directory'nin şeması ve yapılandırma bilgileri gibi önemli verileri çoğaltır.

---

### **DRSUAPI'nin Çalışma Prensibi**

1. **Replication Request**:
    
    - Bir DC, başka bir DC'den veri istemek için DRSUAPI kullanarak bir replication request gönderir.

2. **RPC Bağlantısı**:
    
    - Replication işlemleri, **RPC** (Remote Procedure Call) üzerinden gerçekleştirilir.
    - Endpoint Mapper genellikle **TCP 135** portunda çalışır ve ardından dinamik portlar üzerinden iletişim kurulur.

3. **Dizin Verilerinin Gönderimi**:
    
    - Talep edilen veriler, DRSUAPI aracılığıyla istekte bulunan DC'ye iletilir.

4. **Tutarlılık Kontrolü**:
    
    - Domain Controller'lar arasında veri tutarlılığı sağlanır.

### **DRSUAPI'nin Kullanım Alanları**

1. **Domain Controller Yönetimi**:
    
    - DC'ler arasında dizin verilerinin çoğaltılmasını sağlar.
    - Örnek: Yeni bir kullanıcı hesabı oluşturulduğunda, bu bilgi tüm DC'lerde eşitlenir.

2. **Yedekleme ve Kurtarma**:
    
    - Active Directory verilerinin yedeklenmesi ve kurtarılması işlemlerinde önemli bir rol oynar.

3. **Penetrasyon Testleri ve Güvenlik Analizi**:
    
    - Saldırganlar, DRSUAPI protokolünü kullanarak NTDS.dit dosyasından parola hash'lerini ve diğer hassas bilgileri elde etmeye çalışabilir.


### **DRSUAPI'nin Güvenlik Riskleri**

1. **NTDS.DIT Dumping**:
    
    - DRSUAPI, saldırganlar tarafından Active Directory'nin NTDS.dit veritabanından parola hash'lerini çalmak için kullanılabilir. Bu işlem için genellikle aşağıdaki araçlar kullanılır:
        - **Mimikatz** (DCSync modülü)
        - **Impacket** (secretsdump.py)

2. **DCSync Saldırısı**:
    
    - **DCSync** saldırısı, saldırganların kendilerini bir DC gibi davranarak DRSUAPI protokolü üzerinden veri talep etmelerini sağlar. Bu saldırıda şunlar elde edilebilir:
        - Kullanıcı parolalarının NTLM hash'leri
        - Kerberos Ticket Granting Ticket (TGT) bilgileri
        - Domain'in SID bilgileri

3. **Yetkisiz Erişim**:
    
    - Yanlış yapılandırılmış izinler nedeniyle, düşük yetkili bir kullanıcı DRSUAPI'yi kullanarak hassas verileri okuyabilir.

---

### **DRSUAPI ile İlgili Araçlar**

1. **Impacket (secretsdump.py)**:
    
    - NTDS.dit dosyasından veri çıkarma ve parola hash'lerini alma işlemleri için kullanılır:

```
secretsdump.py administrator@<IP>
```

2. **Mimikatz (DCSync)**:

- Bir DCSync saldırısı gerçekleştirmek için kullanılan popüler bir araçtır:

```
lsadump::dcsync /domain:domain.local /user:Administrator
```


3. **Repadmin**:

- Microsoft tarafından sağlanan bir araçtır ve DRSUAPI protokolünü kullanarak DC'ler arasındaki çoğaltma işlemlerini yönetir:

```
repadmin /showrepl
```

### **DRSUAPI'nin Saldırı Yüzeyi**

1. **DCSync Saldırısı**:
    
    - Saldırgan, bir DC'den diğerine veri çoğaltan bir sistem gibi davranarak hassas bilgileri talep eder.
    - Yüksek yetkilere sahip hesaplar (ör. Domain Admins veya Enterprise Admins) bu saldırıyı gerçekleştirebilir.

2. **Pass-the-Hash Saldırıları**:
    
    - Elde edilen NTLM hash'leri, sistemlere kimlik doğrulaması yapmak için kullanılabilir.

3. **Güvenlik Zafiyetleri**:
    
    - Yanlış yapılandırılmış Active Directory güvenlik politikaları, saldırganların DRSUAPI üzerinden yetkisiz erişim sağlamasına yol açabilir.

---

### **DRSUAPI'nin Hardening Önlemleri**

1. **Yönetici Hesaplarını Koruma**:
    
    - **Domain Admins** ve **Enterprise Admins** gibi yüksek yetkili hesapların saldırıya uğramasını önlemek.

2. **İzinleri Sınırlandırma**:
    
    - DRSUAPI ve diğer Active Directory servislerine erişimi yalnızca yetkili hesaplarla sınırlandırmak.

3. **Güvenlik İzleme**:
    
    - DCSync veya diğer DRSUAPI tabanlı saldırıları tespit etmek için Active Directory günlüklerini izlemek.
        - Event ID 4662: Active Directory'deki hassas bir nesneye erişim.

4. **Güçlü Parola Politikaları**:
    
    - Hassas hesaplar için güçlü parolalar ve çok faktörlü kimlik doğrulama kullanmak.

---

**Özet:**  
DRSUAPI, Active Directory'nin temel bir protokolü olup, dizin verilerinin Domain Controller'lar arasında çoğaltılmasını  sağlar. Ancak, bu protokol saldırganlar tarafından da kullanılabilir; bu nedenle, doğru yapılandırma ve güvenlik önlemleri kritik öneme sahiptir.




---
{"dg-publish":true,"permalink":"/ctf/cicada-ctf/"}
---

![Pasted image 20241028162824.png](/img/user/resimler/Pasted%20image%2020241028162824.png)

### 1. Genel Bilgiler:

- **Hedef Sistem:** Microsoft Windows işletim sistemi
- **Hostname:** CICADA-DC
- **Domain:** cicada.htb (Active Directory için yapılandırılmış)
- **Clock Skew:** Zaman farkı yaklaşık 7 saat; bu, oturum açma ve doğrulama işlemlerinde sorunlara yol açabilir.

### 2. Portlar ve Servisler:

- **53/tcp (domain):**
    
    - DNS servisi çalışıyor. "Simple DNS Plus" isimli bir DNS yazılımı kullanılıyor. Bu, potansiyel olarak DNS yönlendirme ve keşif saldırıları için bilgi sağlayabilir.
- **88/tcp (kerberos-sec):**
    
    - Microsoft Windows Kerberos protokolü, bu sistemde kimlik doğrulama için kullanılıyor. Bu, hedefin bir Active Directory Domain Controller (DC) olduğunu gösterir.
- **135/tcp (msrpc):**
    
    - Microsoft Windows RPC (Remote Procedure Call) hizmeti çalışıyor. Bu, uzaktan prosedür çağrılarının yapılmasına olanak tanır ve çeşitli uzaktan yönetim işlemleri için kullanılabilir.
- **139/tcp (netbios-ssn):**
    
    - NetBIOS Session Service, dosya paylaşımı gibi işlemler için kullanılan SMB’nin (Server Message Block) bir parçasıdır.
- **389/tcp (ldap):**
    
    - LDAP (Lightweight Directory Access Protocol) protokolü Microsoft Active Directory ile entegre çalışıyor. Bu port üzerinden Active Directory bilgilerine erişilebilir.
    - **Domain:** cicada.htb
    - **Site:** Default-First-Site-Name
- **445/tcp (microsoft-ds):**
    
    - SMB protokolünün modern versiyonu çalışıyor. SMB hizmetine doğrudan erişim mümkündür; bu da ağ paylaşımlarına erişmek için kullanılabilir.
- **464/tcp (kpasswd5):**
    
    - Kerberos password değişiklikleri için kullanılan bir port. Kullanıcı parolası sıfırlama işlemleri yapılabilir.
- **593/tcp (ncacn_http):**
    
    - RPC over HTTP protokolü çalışıyor. Bu, HTTP üzerinden RPC çağrılarının yapılmasını sağlar ve genelde firewall bypass için kullanılabilir.
- **636/tcp (ssl/ldap):**
    
    - SSL ile güvenli LDAP bağlantısı. Active Directory'ye SSL üzerinden erişim imkanı tanır.
- **3268/tcp (ldap):**
    
    - Global Catalog (GC) portu olarak bilinir ve Active Directory'deki tüm nesnelere hızlı erişim sağlar.
- **3269/tcp (ssl/ldap):**
    
    - Güvenli Global Catalog bağlantısı için SSL kullanıyor. Active Directory üzerinde SSL korumalı GC erişimi sağlar.
- **5985/tcp (http):**
    
    - Microsoft HTTPAPI httpd 2.0 çalışıyor. Bu port, Windows Remote Management (WinRM) için kullanılır. Windows sunucularında uzaktan yönetim için kullanılır ve PowerShell komutları gibi yönetim araçlarıyla uyumludur.
- **61883/tcp (msrpc):**
    
    - RPC bağlantısı sağlıyor, ek uzaktan yönetim işlemleri yapılabilir.

### 3. SSL Sertifikaları:

- 389, 636, 3268, ve 3269 numaralı portlar için geçerli SSL sertifikaları gözlemlenmiş.
- Sertifika Detayları:
    - **Common Name (CN):** CICADA-DC.cicada.htb
    - **Geçerlilik Süresi:** 2024-08-22 ile 2025-08-22 arası

Bu sertifikalar, hedef sistemin domain controller olduğuna dair güvenilir bilgi sunuyor.


### Güvenlik Açığı Analizi ve Saldırı Vektörleri:

1. **SMB (Port 445)** ve **NetBIOS (Port 139)**:
    - SMB üzerinden bilgi sızdırma, kullanıcı adı toplama ve açık paylaşımlara erişim mümkündür. SMB ve NetBIOS’a yönelik zafiyetler sıkça kullanılır.
2. **Kerberos (Port 88 ve 464)**:
    - Kerberos kimlik doğrulama protokolüne yönelik saldırılar, özellikle Pass-the-Ticket ve Kerberoasting gibi tekniklerle gerçekleştirilebilir. Eğer kullanıcı adları bulunursa, parolaların hash değerleri çıkarılabilir.
3. **LDAP (389, 636)** ve **Global Catalog (3268, 3269)**:
    - Active Directory yapısına yönelik saldırılar düzenlenebilir. LDAP ile kullanıcı bilgileri ve grup üyelikleri elde edilebilir.
4. **RPC (Port 135 ve 593)**:
    - Uzaktan kod yürütme saldırıları için kullanılabilir. Güvenlik zafiyetleri var ise, hizmetlere erişim sağlanabilir.
5. **WinRM (Port 5985)**:
    - WinRM etkinse ve erişim sağlanabilirse, uzaktan PowerShell çalıştırma ve yönetim işlemleri yapılabilir.



smbclient kullanılarak bağlanalım . (anon)

![Pasted image 20241028163538.png](/img/user/resimler/Pasted%20image%2020241028163538.png)

Ve HR'nın dizininde bir txt dosyası buldum

![Pasted image 20241028164037.png](/img/user/resimler/Pasted%20image%2020241028164037.png)

![Pasted image 20241028164136.png](/img/user/resimler/Pasted%20image%2020241028164136.png)

Cicada$M6Corpb*@Lp#nZp!8

![Pasted image 20241028164315.png](/img/user/resimler/Pasted%20image%2020241028164315.png)


### RID BRUTE FORCE
Rid Blute ile Kullanıcıları Numaralandırma

![Pasted image 20241028164506.png](/img/user/resimler/Pasted%20image%2020241028164506.png)

- **`cicada.htb`**: Bu, hedef sistemin (domain veya IP adresi olarak) tanımlandığı bölümdür. Burada hedef sistem `cicada.htb` olarak verilmiştir.
    
- **`-u "guest"`**: Kullanıcı adı parametresidir. Bu komutta, `guest` kullanıcısı ile giriş yapmayı denemektesiniz.
    
- **`-p ''`**: Parola parametresi. `''` olarak belirtilmiş, yani parolasız giriş deneniyor (boş parola ile oturum açma).
    
- **`--rid-brute`**: RID brute-forcing işlemi başlatır. RID brute-forcing, hedef sistemdeki kullanıcı hesaplarını tespit etmek için Relative Identifier (RID) değerlerini sıralı bir şekilde tahmin eder. Bu, kullanıcı hesaplarının listelemek için kullanılan bir tekniktir.
    
- **`| grep SidTypeUser`**: Çıktıyı filtrelemek için kullanılan `grep` komutu. Burada yalnızca `SidTypeUser` içeren satırları listelemek istiyorsunuz, bu da kullanıcı hesaplarının SID türü olarak işaretlenmiş olanları gösterir.

Bu komut, `guest` kullanıcı adı ve boş parola ile hedef sistemdeki kullanıcı hesaplarını RID brute-forcing ile bulmaya çalışır ve yalnızca kullanıcı türündeki SID'leri listeler

![Pasted image 20241028164652.png](/img/user/resimler/Pasted%20image%2020241028164652.png)

Michael wrightson kullanıcısı aracılığıyla sunucuya bağlanmak ve çeşitli bilgileri ayıklamak için enum4linux-ng kullanın

![Pasted image 20241028164948.png](/img/user/resimler/Pasted%20image%2020241028164948.png)

### Parametreler ve Açıklamaları:

- **`enum4linux-ng`**: `enum4linux` aracının güncel bir versiyonudur ve Windows SMB/NetBIOS servisleri üzerinde bilgi toplamak için kullanılır. `enum4linux-ng`, hedef sistem hakkında daha ayrıntılı bilgi sağlamak için geliştirilmiştir.
    
- **`-A`**: Bu parametre, geniş kapsamlı bir bilgi toplama modunu aktif hale getirir. `-A` kullanıldığında, aşağıdaki işlemler gerçekleştirilir:
    
    - Hedef sistem hakkında temel bilgiler toplanır.
    - SMB paylaşım bilgileri, kullanıcı hesapları, grup bilgileri, politika bilgileri gibi çeşitli SMB bilgileri alınır.
    - Domain veya iş grubu bilgileri ve diğer ağ kaynakları taranır.
- **`-u 'Michael.wrightson'`**: Hedef sistemdeki **kullanıcı adı**dır. Bu parametre, bağlantının bu kullanıcı adıyla yapılacağını belirtir.
    
- **`-p 'Cicada$M6Corpb*@Lp#nZp!8'`**: **Parola** parametresidir. Hedef sistemde `Michael.wrightson` kullanıcısı için gerekli parolayı belirtir.
    
- **`10.10.11.35`**: Bu, hedef IP adresidir. `enum4linux-ng` aracı bu IP adresine bağlanarak bilgi toplamaya çalışacaktır.
    

Bu komut, `Michael.wrightson` kullanıcısı ve parolası ile hedef sisteme erişip kapsamlı bilgi toplama işlemi yapacaktır.

![Pasted image 20241028165428.png](/img/user/resimler/Pasted%20image%2020241028165428.png)

David'in şifresi bulundu: aRt$Lp#7t*VQ!3

![Pasted image 20241028165621.png](/img/user/resimler/Pasted%20image%2020241028165621.png)

![Pasted image 20241028165653.png](/img/user/resimler/Pasted%20image%2020241028165653.png)

user.txt dosyasını almak için evil-winrm oturum açma özelliğini kullanın

- **Kaynak ve Hedef Dizini**: `sourceDirectory` değişkeni yedeklenecek dosyaların bulunduğu dizini, `destinationDirectory` ise yedekleme dosyasının kaydedileceği hedef dizini tanımlar.
    
- **Kimlik Bilgileri**: Kullanıcı adı ve parola `emily.oscars` kullanıcısı için güvenli bir biçimde saklanır. Parola `ConvertTo-SecureString` ile güvenli dizeye dönüştürülür ve ardından `credentials` nesnesi olarak depolanır.
    
- **Tarih Damgası ve Dosya Adı**: `dateStamp` değişkeni, yedek dosya adı için geçerli tarih ve saati `yyyyMMdd_HHmmss` formatında alır. Bu, her yedek dosyanın benzersiz olmasını sağlar.
    
- **Dosya Sıkıştırma**: `Compress-Archive` komutu, belirtilen kaynak dizini sıkıştırır ve tarih damgası eklenmiş dosya adı ile hedef dizine `.zip` olarak kaydeder.
    
- **Tamamlanma Mesajı**: `Write-Host` komutu, yedekleme işlemi başarılı olduğunda dosyanın nereye kaydedildiğine dair bir mesaj verir.

user.txt dosyasını almak için evil-winrm oturum açma özelliğini kullanın


**Evil-WinRM**, Windows sistemlerine uzaktan erişim sağlamak için kullanılan bir araçtır. WinRM (Windows Remote Management) protokolünü kullanarak, kullanıcı adı ve parola ile bağlanabilir. Ana işlevleri şunlardır:

- **Uzaktan Komut Yürütme**: Windows komutlarını uzaktan çalıştırma imkanı.
- **Dosya Aktarımı**: Yerel dosyaları hedef sisteme yükleme veya indirme.
- **Kimlik Doğrulama**: Kerberos ve NTLM gibi yöntemlerle erişim sağlama.
- **Bilgi Toplama**: Hedef sistemdeki kullanıcılar ve gruplar hakkında bilgi edinme.

Penetrasyon testleri ve sistem yönetimi için etkili bir araçtır.

![Pasted image 20241028170343.png](/img/user/resimler/Pasted%20image%2020241028170343.png)
![Pasted image 20241028170355.png](/img/user/resimler/Pasted%20image%2020241028170355.png)

![Pasted image 20241028170436.png](/img/user/resimler/Pasted%20image%2020241028170436.png)


## Privilege Escalation
![Pasted image 20241028170553.png](/img/user/resimler/Pasted%20image%2020241028170553.png)

![Pasted image 20241028170632.png](/img/user/resimler/Pasted%20image%2020241028170632.png)

![Pasted image 20241028170647.png](/img/user/resimler/Pasted%20image%2020241028170647.png)

Bu SeBackupPrivilege hakkında: Windows Privilege Escalation: SeBackupPrivilege - Hacking Articles

[Makaledeki](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/) adımları izleyin

### . **SeBackupPrivilege (Yedekleme Yetkisi)**

- **Açıklama**: Bu yetki, kullanıcının dosyaları ve dizinleri yedeklemesine olanak tanır. Kayıt defterindeki `SAM` ve `SYSTEM` dosyalarını çıkartmak için bu yetkinin aktif olması gerekir.
- **Neden Önemli**: Bu yetki, bir kullanıcının kayıt defterinin kritik bölümlerine erişim sağlamasına ve bu bölümleri dışa aktarmasına izin verir.

### 2. **SeRestorePrivilege (Geri Yükleme Yetkisi)**

- **Açıklama**: Bu yetki, kullanıcının dosyaları ve dizinleri geri yüklemesine olanak tanır.
- **Neden Önemli**: Geri yükleme yetkisi, yedekleme işlemleri sırasında kritik dosyaların geri yüklenmesi veya dosya sisteminde değişiklik yapılması gerektiğinde faydalıdır.

### Diğer Yetkilerin Rolü

- Diğer listelenen yetkiler (örneğin, `SeShutdownPrivilege` ve `SeChangeNotifyPrivilege`) sistem yönetimi ve işlem kontrolü için önemlidir, ancak doğrudan `SAM` ve `SYSTEM` dosyalarını çıkartma işlemi için gerekli değildir.

![Pasted image 20241028171631.png](/img/user/resimler/Pasted%20image%2020241028171631.png)

Yöneticinin hash değerini almak için pypykatz kullanın

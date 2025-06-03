---
{"dg-publish":true,"permalink":"/baglantilar/chroot/"}
---

**chroot**, Unix ve Linux sistemlerinde kullanılan bir güvenlik mekanizmasıdır. Bu mekanizma, bir prosesin root dizinini (`/`) değiştirerek, o prosesin dosya sistemine erişimini sınırlar. Yani, proses artık belirlenen dizin dışındaki dosya ve dizinlere erişemez. Bu, özellikle güvenlik açısından kritik uygulamalarda (örneğin FTP sunucuları) kullanılır.

### chroot Ne İşe Yarar?

- **Güvenlik:** Bir kullanıcı veya proses, chroot ile sınırlandırılmış bir dizin içinde "hapisteymiş" gibi davranır. Bu dizin dışındaki dosya ve dizinlere erişemez.
    
- **İzolasyon:** Özellikle sunucu uygulamalarında, kullanıcıların sistemin geri kalanına erişmesini engeller.
    
- **Test Ortamı:** chroot, farklı bir root dizin altında yalıtılmış bir ortam oluşturarak yazılım testleri için de kullanılabilir.
    

### VSFTPD ve chroot

VSFTPD (Very Secure FTP Daemon), güvenli bir FTP sunucusu olarak bilinir. VSFTPD, kullanıcıların FTP üzerinden erişimini sınırlamak için **chroot** mekanizmasını kullanır. Bu sayede, kullanıcılar kendilerine ayrılan dizin dışındaki dosya ve dizinlere erişemez.

### secure_chroot_dir Nedir?

- **secure_chroot_dir**, VSFTPD'nin chroot işlemi için kullandığı boş bir dizini belirtir.
    
- Bu dizin, VSFTPD'nin güvenli bir şekilde chroot işlemi yapabilmesi için gereklidir.
    
- Örneğin, `secure_chroot_dir=/var/run/vsftpd/empty` ayarı, VSFTPD'nin chroot işlemi için `/var/run/vsftpd/empty` dizinini kullanacağını belirtir.
    
- Bu dizin genellikle boştur ve sadece VSFTPD'nin chroot işlemi için gerekli temel dosyaları içerir.
    

### chroot Örneği

Örneğin, bir FTP kullanıcısı için chroot dizini `/home/user1` olarak ayarlandıysa:

- Bu kullanıcı, FTP üzerinden sadece `/home/user1` dizini içindeki dosya ve dizinlere erişebilir.
    
- Örneğin, `/etc`, `/var` veya `/usr` gibi sistem dizinlerine erişemez.
    

### Güvenlik Açısından Önemi

- chroot, kullanıcıların sistemin geri kalanına erişimini engelleyerek, potansiyel güvenlik açıklarını azaltır.
    
- Özellikle FTP sunucularında, kullanıcıların hassas sistem dosyalarına erişmesini önlemek için kritiktir.
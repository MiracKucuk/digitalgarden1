---
{"dg-publish":true,"permalink":"/baglantilar/inetd-ve-standalone-nedir-aciklama/"}
---

### 1. **inetd (Internet Daemon):**

- **inetd**, Unix ve Linux sistemlerinde kullanılan bir "süper sunucu" (`super server`) olarak bilinir.
    
- Temel görevi, belirli ağ portlarını dinlemek ve bu portlara gelen bağlantıları ilgili servislere yönlendirmektir.
    
- Örneğin, bir FTP client'i sunucuya bağlandığında, `inetd` bu bağlantıyı alır ve `VSFTPD` gibi bir FTP sunucusunu başlatır.
    
- **Avantajı:** Sistem kaynaklarını daha verimli kullanır, çünkü servisler sadece ihtiyaç duyulduğunda çalıştırılır.
    
- **Dezavantajı:** Her bağlantıda servisin yeniden başlatılması gerektiğinden, başlatma süresi nedeniyle performans düşebilir.
    

### 2. **Bağımsız Daemon (Standalone Daemon):**

- **Bağımsız daemon**, bir servisin sürekli olarak arka planda çalıştığı ve kendi bağlantılarını yönettiği anlamına gelir.
    
- VSFTPD bu modda çalıştığında, sürekli olarak FTP portunu (genellikle 21 numaralı port) dinler ve gelen bağlantıları kendisi yönetir.
    
- **Avantajı:** Bağlantılar daha hızlı kabul edilir, çünkü servis zaten çalışıyordur.
    
- **Dezavantajı:** Sürekli çalıştığı için sistem kaynaklarını daha fazla kullanır.
    

### VSFTPD ve Çalışma Modları:

- VSFTPD, hem **inetd** üzerinden hem de **bağımsız daemon** olarak çalışabilir.
    
- **inetd** modu, daha az kaynak kullanımı gerektiren durumlar için uygundur.
    
- **Bağımsız daemon** modu ise yüksek performans ve hızlı yanıt süreleri gerektiren durumlar için tercih edilir.
    

### Hangi Modu Seçmelisiniz?

- Eğer sunucunuzda çok sayıda farklı servis çalışıyorsa ve kaynak kullanımını optimize etmek istiyorsanız, **inetd** modunu tercih edebilirsiniz.
    
- Eğer FTP sunucunuz yoğun bir şekilde kullanılıyorsa ve hızlı yanıt süreleri önemliyse, **bağımsız daemon** modu daha uygun olacaktır.
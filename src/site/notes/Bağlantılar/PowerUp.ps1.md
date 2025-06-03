---
{"dg-publish":true,"permalink":"/baglantilar/power-up-ps1/"}
---



**PowerUp.ps1**, Windows ortamlarında yerel ayrıcalık yükseltme (privilege escalation) testleri yapmak için kullanılan bir PowerShell betiğidir. Bu betik, sistem yapılandırmalarını, izinleri ve hizmetleri inceleyerek, düşük ayrıcalıklı bir kullanıcının daha yüksek izinler elde edebileceği güvenlik açıklarını tespit eder.

**PowerUp.ps1**'in sağladığı özellikler şunlardır:

1. **Yazma İzinlerini Tespit Etme**: Yönetici izinleri olmadan değiştirilebilecek hassas dosya ve klasörleri bulur.
2. **Zayıf Hizmet Yapılandırmalarını İnceleme**: Zayıf izinlere sahip hizmetleri veya güvenlik açıkları olan hizmetleri listeler.
3. **Sistem Bilgilerini Toplama**: Çeşitli sistem bilgilerini toplayarak güvenlik açığı analizine katkı sağlar.

Bu betik, sızma testlerinde veya güvenlik değerlendirmelerinde, sistem yöneticileri ve güvenlik uzmanları tarafından sıkça kullanılır.
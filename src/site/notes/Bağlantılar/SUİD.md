---
{"dg-publish":true,"permalink":"/baglantilar/suid/"}
---

`find / -perm -4000 2>/dev/null` komutunun amacı, Linux/Unix sistemlerinde **SUID** (Set User ID) bitine sahip dosyaları aramaktır. Bu tür dosyalar, dosyanın sahibi kim olursa olsun, o kullanıcı yetkileriyle çalıştırılmasına izin verir. Bu özellik, sistemde güvenlik açıkları oluşturabileceği için önemlidir.

Kısaca açıklama:

1. **`find /`**: Tüm dosya sistemini tarar ("/" kök dizinden başlayarak).
2. **`-perm -4000`**: Sadece SUID biti (4000) açık olan dosyaları bulur.
3. **`2>/dev/null`**: Hata mesajlarını bastırır (örneğin, erişim izni olmayan dizinlerden gelen hatalar).

### Önemli Noktalar:

- **SUID dosyaları**: Sistem yönetimi için faydalıdır ama potansiyel bir güvenlik riski oluşturabilir.
- **Denetleme**: Çıktıyı inceleyerek hangi dosyaların SUID özelliğine sahip olduğunu kontrol edebilirsiniz.
- **Örnek Çıktı**: `/usr/bin/passwd` gibi dosyalar genellikle SUID olarak ayarlanmıştır.

> **Not**: Bu komut, güvenlik araştırmaları veya sistem denetimi yaparken faydalıdır. Ancak, izinsiz sistemlerde bu tür işlemler etik dışıdır ve yasa dışı olabilir.


`4000` ifadesi, dosya izinlerinde **SUID (Set User ID)** bitini belirtir. Dosya izinleri, Linux ve Unix sistemlerinde 4 basamaklı oktal (sekizlik) bir sistemle ifade edilir. Burada her basamağın bir anlamı vardır:

### Detaylı Anlam:

1. **Dosya İzinleri Formatı**:
    
    - İzinler 4 basamaklıdır: `[SUID/SGID/STICKY][OWNER][GROUP][OTHERS]`.
    - İlk basamak özel bitleri (SUID, SGID, Sticky) temsil eder.
    - Diğer üç basamak ise dosya sahibinin (Owner), grup üyelerinin (Group) ve diğer kullanıcıların (Others) okuma (r), yazma (w) ve çalıştırma (x) izinlerini temsil eder.
2. **4000 (SUID - Set User ID)**:
    
    - **4**: SUID bitini temsil eder.
    - Bu bit ayarlandığında, dosyayı çalıştıran kişi, dosyanın sahibi gibi işlem yapar. Yani, kullanıcı geçici olarak dosya sahibinin izinlerini alır.
    - Örneğin: `/usr/bin/passwd` komutu.
        - Bu komut, bir kullanıcının şifresini değiştirmek için root yetkilerini gerektirir. SUID ile çalıştırıldığında, geçici olarak root yetkileri devreye girer.
3. **Diğer Özel Bitler**:
    
    - **2000**: SGID (Set Group ID) → Grup yetkilerini devreye sokar.
    - **1000**: Sticky bit → Dizinlerde dosyaları sadece sahibi silebilir.

### Örnek:

Bir dosyanın izinleri `-rwsr-xr-x` şeklinde ise:

- İlk `s`: SUID'nin aktif olduğunu gösterir.
- Bu durumda, dosya sahibinin (örneğin, root) yetkileriyle çalışır.

### Önem:

- SUID, sistem işlevselliği açısından gereklidir ama yanlış yapılandırıldığında güvenlik açıklarına yol açabilir. Bu yüzden, SUID'li dosyaları düzenli olarak denetlemek önemlidir.


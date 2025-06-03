---
{"dg-publish":true,"permalink":"/baglantilar/bitsadmin/"}
---



### Bitsadmin

- **Tanım**: Bitsadmin, Windows'un arka planda dosya indirmek ve yüklemek için kullanılan bir komut satırı aracıdır. BITS (Background Intelligent Transfer Service), dosya transferi sırasında ağ bant genişliğini optimize etmek için tasarlanmıştır.
- **Kullanım Alanları**:
    - Büyük dosyaların indirilmesi ve yüklenmesi.
    - Dosya transfer işlemlerinin zamanlaması ve durumu hakkında bilgi almak.
    - Düşük bant genişliği veya kesintili ağ bağlantıları durumunda güvenilir transfer sağlamak.

```shell-session
bitsadmin /transfer myDownloadJob /download /priority normal http://example.com/file.exe C:\path\to\save\file.exe
```

Bu komut, belirtilen URL'den dosyayı indirir ve belirtilen konuma kaydeder.
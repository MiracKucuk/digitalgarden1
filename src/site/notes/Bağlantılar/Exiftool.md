---
{"dg-publish":true,"permalink":"/baglantilar/exiftool/"}
---

**Exiftool**, resim dosyaları ve diğer medya dosyaları üzerinde **metadata** okuma, yazma ve düzenleme işlemleri yapmaya yarayan bir komut satırı aracıdır.

* [[Bağlantılar/Metadata nedir\|Metadata nedir]]


### Exiftool'un Özellikleri:

1. **Çoklu Format Desteği:** JPEG, PNG, PDF, MP3, MP4 gibi birçok dosya formatının metadata'sını işleyebilir.
2. **Metadata Düzenleme:** Metadata bilgilerini güncelleyebilir, silebilir veya yenilerini ekleyebilir.
3. **Komut Satırı İşlemleri:** Özellikle otomasyon ve toplu işlemler için kullanışlıdır.
4. **Veri Çıkarma:** Metadata bilgilerini detaylı şekilde analiz etmek için kullanılabilir.


### Exiftool ve Güvenlik:

Exiftool, genellikle zararsız bir araç olarak kabul edilir, ancak bazı eski sürümlerinde **komut enjeksiyonu** gibi güvenlik açıkları bulunmuştur. Bu tür açıklar, bir saldırganın özel hazırlanmış dosyalarla hedef sistemde kötü niyetli komutlar çalıştırmasına izin verebilir. Bu nedenle, Exiftool'un güncel sürümünü kullanmak önemlidir.

Örneğin, bir saldırgan bir resmin metadata'sına zararlı bir komut ekleyebilir. Exiftool bu dosyayı işlerken komutu çalıştırabilir ve sistemin ele geçirilmesine yol açabilir.

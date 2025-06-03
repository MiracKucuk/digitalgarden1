---
{"dg-publish":true,"permalink":"/baglantilar/regsvr32/"}
---



**Regsvr32**, Windows işletim sistemlerinde kullanılan bir komut satırı aracıdır ve özellikle **[[Bağlantılar/COM (Component Object Model\|COM (Component Object Model]])** bileşenlerini kaydetmek veya kayıttan kaldırmak için kullanılır. Regsvr32 aracı, DLL (Dynamic Link Library) dosyalarını kaydederek sistemdeki uygulamalar tarafından kullanılmasını sağlar.

### Özellikleri ve Kullanım Alanları:

- **DLL Kaydetme**: Regsvr32, bir DLL dosyasını sistem kaydına ekleyerek, o DLL'nin kullanılabilir hale gelmesini sağlar. Bu, COM bileşenleri gibi belirli türdeki yazılımların çalışması için gereklidir.
- **Kayıt Veritabanını Güncelleme**: Kaydedilen DLL dosyaları, sistemin kayıt defterinde belirli anahtarlarla ilişkilendirilir, böylece uygulamalar bu bileşenleri kullanabilir.
- **Kaydetme ve Kaldırma**: DLL dosyalarını kaydetmenin yanı sıra, aynı zamanda kayıt defterinden kaldırma işlemi de yapılabilir.


### Örnek Komutlar:

- **DLL Kaydetme**:

```shell-session
regsvr32 C:\path\to\your.dll
```
Bu komut, belirtilen DLL dosyasını kayıt defterine kaydeder.


* ***DLL Kaldırma**:
```shell-session
regsvr32 /u C:\path\to\your.dll
```

Bu komut, belirtilen DLL dosyasını kayıt defterinden kaldırır.


### Güvenlik Açısından Dikkat Edilmesi Gerekenler:

Kötü niyetli aktörler, Regsvr32 aracını kullanarak zararlı DLL dosyalarını kaydedebilir ve sistemdeki süreçlere entegre edebilirler. Bu nedenle, Regsvr32'nin kullanımı dikkatli bir şekilde izlenmelidir. Astaroth saldırısında olduğu gibi, zararlı yazılımlar, bu aracı kullanarak dosyalarını yükleyip, sistemde kalıcılık sağlamaya çalışabilir.

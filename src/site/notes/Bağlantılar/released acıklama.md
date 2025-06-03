---
{"dg-publish":true,"permalink":"/baglantilar/released-aciklama/"}
---

SQL sorgusunda **`released = 1`**, sorgunun bir koşulunu ifade eder. Bu, sorgunun yalnızca **`released`** sütununda değeri `1` olan satırları seçmesini sağlar.

### Detaylı Açıklama:

- **`released`**: Veritabanındaki bir sütun adıdır. Bu sütun, genellikle ürünlerin yayınlanmış olup olmadığını göstermek için kullanılır.
- **`= 1`**: Bu, **`released`** sütununun değerinin `1` olmasını şart koşar.
    - `1`: Genelde **"yayınlanmış"**, **"aktif"** ya da **"kullanılabilir"** anlamına gelir.
    - `0`: Yayınlanmamış veya pasif ürünler olabilir.
---
{"dg-publish":true,"permalink":"/baglantilar/sqlmap-dosyasinin-ayrintili-aciklamasi/"}
---


### SQLMap Çıktısının Ayrıntılı Analizi

**Parametre:**

- `id (POST)`  
    SQL açığının bulunduğu yer, HTTP POST isteğindeki `id` parametresidir. SQL sorgularına bu parametre üzerinden zararlı veri enjekte edilebilmektedir.

---

**Tespit Edilen SQL Açığı Türleri ve Detayları:**

1. **Tür:** `boolean-based blind` (Koşul Tabanlı Kör Enjeksiyon)
    
    - **Başlık:** `AND boolean-based blind - WHERE or HAVING clause`  
        SQL enjeksiyonu, `WHERE` veya `HAVING` koşulunda `AND` ifadesi kullanılarak tespit edilmiştir. Bu yöntem, sistemin belirli bir koşula verdiği cevaba göre çalışır.
    - **Payload (Kullanılan Zararlı Veri):**  
        `id=1 AND 4002=4002`  
        Burada `4002=4002` koşulu doğru olduğu için, sunucunun verdiği yanıt incelenerek SQL açığının varlığı tespit edilmiştir.
2. **Tür:** `error-based` (Hata Tabanlı Enjeksiyon)
    
    - **Başlık:** `MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)`  
        MySQL 5.0 veya üzeri sürümlerinde çalışan hata tabanlı enjeksiyon. Bu yöntemde, hataları tetikleyerek veri tabanı ile ilgili bilgi elde edilir.
    - **Payload:**  
        `id=1 AND (SELECT 8574 FROM(SELECT COUNT(*),CONCAT(0x7162707871,(SELECT (ELT(8574=8574,1))),0x71706a7a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)`  
        Burada hata tetiklemek için rastgele değerler kullanılarak sorgunun hatalı çalışması sağlanmıştır. Bu hatalardan sunucunun veri tabanı yapısı hakkında bilgi edinilmiştir.
3. **Tür:** `stacked queries` (Zincirlenmiş Sorgular)
    
    - **Başlık:** `MySQL >= 5.0.12 stacked queries (comment)`  
        MySQL 5.0.12 veya üzeri sürümlerde çalışan bu yöntemle, aynı sorgu içerisinde birden fazla işlem zincirlenmiştir.
    - **Payload:**  
        `id=1;SELECT SLEEP(5)#`  
        Bu payload ile `id=1` sorgusundan sonra `SELECT SLEEP(5)` sorgusu eklenmiştir. Bu, sunucunun 5 saniye beklediğini doğrulayarak enjeksiyonun çalıştığını gösterir.
4. **Tür:** `time-based blind` (Zaman Tabanlı Kör Enjeksiyon)
    
    - **Başlık:** `MySQL >= 5.0.12 AND time-based blind (query SLEEP)`  
        Sunucunun yanıt süresine dayalı olarak çalışan bu yöntem, sorgunun etkili olup olmadığını anlamak için bir zaman gecikmesi yaratır.
    - **Payload:**  
        `id=1 AND (SELECT 4719 FROM (SELECT(SLEEP(5)))uGMv)`  
        Burada sunucu sorguyu çalıştırırken 5 saniye bekler. Bu süre, SQL açığının varlığını doğrulamak için kullanılır.
5. **Tür:** `UNION query` (Birleştirme Sorgusu Enjeksiyonu)
    
    - **Başlık:** `Generic UNION query (NULL) - 9 columns`  
        Bu yöntem, UNION sorgusuyla farklı veri kümelerini birleştirerek çalışır. Enjeksiyon sırasında sorgunun kaç kolon içerdiği belirlenir.
    - **Payload:**  
        `id=1 UNION ALL SELECT NULL,CONCAT(0x7162707871,0x65646f6252695259514666796a4a6e4f7568674a51746e7a696f566d675241634e654c7661796a79,0x71706a7a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -`  
        Burada UNION sorgusuyla hedef sistemden veri çekilebilecek kolonlar ve yapılar test edilmiştir.

---

**Hedef Hakkında Bilgi:**

- **Web Sunucusu İşletim Sistemi:** `Linux Debian 10 (buster)`  
    Sunucu, Debian 10 işletim sistemi üzerinde çalışıyor.
- **Web Uygulama Teknolojisi:** `Apache 2.4.38`  
    Apache 2.4.38, web sunucusu olarak kullanılıyor.
- **Veri Tabanı Yönetim Sistemi:** `MySQL >= 5.0 (MariaDB fork)`  
    Arka uç veri tabanı, MySQL 5.0 veya üzeri bir sürüm (MariaDB çatallaması).

---

**Sonuç:**  
SQLMap, HTTP POST isteğindeki `id` parametresinin SQL enjeksiyonlarına karşı savunmasız olduğunu farklı yöntemlerle tespit etti. Test edilen tekniklerin çoğu başarılı oldu ve bu sayede hedef sistemdeki veri tabanının yapısı ve içerdiği bilgilere erişim sağlanabilir.
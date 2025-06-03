---
{"dg-publish":true,"permalink":"/baglantilar/kod-aciklmasi-php-3/"}
---


- **$link = mysqli_connect($host, $username, $password, $database, 3306);**  
    Veritabanına bağlanır. Bağlantı bilgileri, `mysqli_connect` fonksiyonu tarafından `$link` değişkeninde saklanır.
    
- **$sql = "SELECT * FROM users WHERE id = " . $_GET["id"] . " LIMIT 0, 1";**  
    Kullanıcıdan gelen `id` parametresine göre bir SQL sorgusu oluşturur. Bu sorgu, `users` tablosundaki belirtilen `id` değerine sahip ilk satırı seçer.
    
- **$result = mysqli_query($link, $sql);**  
    SQL sorgusunu çalıştırır ve sonucu `$result` değişkenine kaydeder. Eğer sorgu başarısız olursa, `$result` değeri `false` olur.
    
- **if (!$result)**  
    `$result` değeri `false` ise (yani sorgu başarısız olmuşsa), aşağıdaki hata mesajı gösterilir ve program durdurulur.
    
- **die("<b>SQL error:</b> ". mysqli_error($link) . "<br>\n");**  
    Programın çalışmasını durdurur (`die`) ve oluşan SQL hatasının ayrıntılarını ekrana yazdırır. `mysqli_error($link)` fonksiyonu, sorguda oluşan hatanın detaylarını alır.


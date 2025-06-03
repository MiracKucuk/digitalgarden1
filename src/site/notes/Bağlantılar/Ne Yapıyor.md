---
{"dg-publish":true,"permalink":"/baglantilar/ne-yapiyor/"}
---

**`SELECT * FROM INFORMATION_SCHEMA.TABLES;`**

- **Amacı:** Veritabanındaki tüm tabloların adını, şemasını ve türünü listeler.
- **Çıktı**

![Pasted image 20241213045559.png](/img/user/resimler/Pasted%20image%2020241213045559.png)

**`SELECT TOP 5 users.firstName, users.lastName, posts.title ...`**

- **Tam Sorgu:**

![Pasted image 20241213045612.png](/img/user/resimler/Pasted%20image%2020241213045612.png)

- **`SELECT TOP 5`**: İlk 5 satırı seçer.
- **`users.firstName, users.lastName, posts.title`**: `users` tablosundan `firstName` ve `lastName`, `posts` tablosundan `title` sütunlarını alır.
- **`FROM users JOIN posts`**: `users` tablosunu `posts` tablosuyla birleştirir.
- **`ON users.id = posts.authorId`**: Birleştirme için `users` tablosundaki `id` sütunu ile `posts` tablosundaki `authorId` sütunu eşleştirilir.


**Amacı:**

- `users` tablosundaki kullanıcıların isimlerini ve soyisimlerini (`firstName`, `lastName`) alır.
- `posts` tablosundaki bu kullanıcıların yazdığı gönderi başlıklarını (`title`) getirir.
- Sonuçları ilk 5 satırla sınırlar.

![Pasted image 20241213045642.png](/img/user/resimler/Pasted%20image%2020241213045642.png)

**Sonuç:**

- İlk sorgu, veritabanındaki tabloların bilgisini listeler.
- İkinci sorgu, her bir kullanıcının yazdığı gönderi başlıklarından ilk 5’ini gösterir.




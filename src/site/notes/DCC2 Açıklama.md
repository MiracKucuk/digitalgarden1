---
{"dg-publish":true,"permalink":"/dcc-2-aciklama/","created":"2025-06-08T18:04:34.803+03:00","updated":"2025-06-08T18:06:30.240+03:00"}
---

DCC2, yani **Domain Cached Credentials v2**, Windows sistemlerinde kullanılan bir **hashleme formatıdır** ve genellikle bir kullanıcı domainde oturum açtıktan sonra sistem çevrimdışıyken de tekrar oturum açabilsin diye **kimlik bilgilerini cache'e almak** için kullanılır.

### DCC Nedir?

- **DCC (Domain Cached Credentials)**: Domain kullanıcılarının şifre hash'lerini local bilgisayarda saklamak için kullanılır. Bu, kullanıcı çevrimdışıyken bile sisteme giriş yapabilmesini sağlar.

- DCC, Windows 2000 ve XP zamanlarında **MS-Cache** olarak da bilinirdi.


### DCC2 Nedir?

- **DCC2**, DCC'nin geliştirilmiş bir sürümüdür ve Windows Vista, Windows 7 ve sonraki sistemlerde kullanılır.
- DCC2, **PBKDF2 (Password-Based Key Derivation Function 2)** algoritmasını kullanarak daha güvenli bir şifreleme sağlar.

- Yani:
    
    - DCC1 (eski): `MD4(MD4(password)) + username` → sonra birleştirip SHA1 uygulanır.
    - DCC2 (yeni): `PBKDF2(HMAC-SHA1, MD4(password), username, 10240 iterations)`

### Neden Brute-force Daha Yavaş?

- DCC2, **PBKDF2 algoritması** kullandığı için binlerce kez (örneğin 10.240 defa) HMAC hesaplaması gerekir.
- Bu, özellikle brute-force saldırılarında **hesaplama maliyetini ciddi şekilde artırır**.
- Bu nedenle hashcat gibi araçlarla DCC2 brute-force yapmak, MD5'e kıyasla **çok daha yavaş** olur.

### Hashcat DCC2 Modu

Hashcat, DCC2 hash'lerini kırmak için `-m 2100` modunu kullanır.


**Özetle:**

- **DCC2**, Windows tarafından domain kullanıcılarının şifrelerini local olarak saklamak için kullanılan güçlü bir hash algoritmasıdır.
- PBKDF2 tabanlı olduğu için brute-force saldırılarına karşı **daha dayanıklıdır** ve bu nedenle saldırılar **çok daha yavaş ilerler**.


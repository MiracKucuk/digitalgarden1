---
{"dg-publish":true,"permalink":"/hacker-one-rapor/idor-1/"}
---


### Sorunun Tanımı:

Bir web sitesinde (ismi gizli tutulmuş) kimlik doğrulaması yapılmış bir kullanıcının, diğer kullanıcıların hesaplarındaki hayvanları silebilmesine olanak tanıyan bir güvenlik açığı tespit edilmiştir. Bu güvenlik açığı, **IDOR (Insecure Direct Object Reference - Güvensiz Doğrudan Nesne Referansı)** sorunundan kaynaklanmaktadır. Yetkilendirme mekanizması, kullanıcıların kimliklerini doğru bir şekilde doğrulayıp kaynaklara erişimlerini kısıtlamadığı için, kötü niyetli bir kullanıcı, bir istekteki **id_mascota** (hayvan ID'si) parametresini değiştirerek başkalarına ait hayvanları silebilmektedir. Bu durum, yetkisiz veri silinmesine yol açmaktadır.

---

### Saldırının İşleyişi (Adımlar):

1. Kayıt sayfasına giderek bir hesap oluşturun.
2. Hesabı doğrulayıp giriş yapın.
3. Burp Suite veya benzeri bir araç kullanarak bir hayvanı silme isteğini yakalayın.
4. Saldırgan, bu istekteki **id_mascota** (hayvan ID'si) parametresini, kurbanın hayvan ID'si ile değiştirir.
5. İstek gönderildiğinde, saldırgan, kurbanın hayvanını başarıyla siler.


![Pasted image 20241208211126.png](/img/user/resimler/Pasted%20image%2020241208211126.png)

Bu istekteki **id_mascota=144094** parametresi, saldırgan tarafından kurbanın hayvan ID'si ile değiştirilerek hayvan silinir.

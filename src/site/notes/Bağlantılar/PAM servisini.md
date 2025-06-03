---
{"dg-publish":true,"permalink":"/baglantilar/pam-servisini/"}
---

**PAM (Pluggable Authentication Modules)**, Linux ve Unix tabanlı sistemlerde kullanılan bir kimlik doğrulama (authentication) mekanizmasıdır. PAM, uygulamaların kullanıcı kimlik doğrulama işlemlerini merkezi ve esnek bir şekilde yönetmesini sağlar. Bu sayede, farklı uygulamalar için farklı kimlik doğrulama yöntemleri kolayca yapılandırılabilir.

### PAM Nasıl Çalışır?

- PAM, bir uygulama (örneğin VSFTPD) ile sistemdeki kimlik doğrulama mekanizmaları (örneğin şifre doğrulama, parmak izi, akıllı kart, LDAP, vs.) arasında bir köprü görevi görür.
    
- Uygulama, kullanıcının kimliğini doğrulamak istediğinde PAM'a başvurur.
    
- PAM, yapılandırma dosyalarına (`/etc/pam.d/` dizini altında) göre hangi modüllerin kullanılacağını belirler ve kimlik doğrulama işlemini gerçekleştirir.
    

### PAM Servisleri

- Her uygulama, PAM için bir "servis adı" (service name) kullanır. Bu servis adı, `/etc/pam.d/` dizini altında bir yapılandırma dosyasına karşılık gelir.
    
- Örneğin, VSFTPD için `pam_service_name=vsftpd` ayarı yapıldığında, PAM `/etc/pam.d/vsftpd` dosyasını kullanarak kimlik doğrulama işlemlerini gerçekleştirir.
    

### VSFTPD ve PAM

- VSFTPD, kullanıcıların kimlik doğrulamasını yapmak için PAM'ı kullanır.
    
- Örneğin, bir kullanıcı FTP sunucusuna bağlanmaya çalıştığında, VSFTPD PAM'a başvurur ve kullanıcının kimlik bilgilerini (kullanıcı adı ve şifre) doğrular.
    
- PAM, bu bilgileri sistemdeki kullanıcı veritabanına (örneğin `/etc/passwd` ve `/etc/shadow`) veya diğer kimlik doğrulama yöntemlerine (LDAP, Active Directory, vs.) göre kontrol eder.
    

### Örnek PAM Yapılandırması

VSFTPD için PAM yapılandırması genellikle `/etc/pam.d/vsftpd` dosyasında bulunur. Örnek bir yapılandırma şöyle olabilir:

```
# /etc/pam.d/vsftpd
auth    required    pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
auth    required    pam_unix.so
account required    pam_unix.so
```

- **auth**: Kimlik doğrulama işlemini yönetir. Örneğin, kullanıcı adı ve şifre kontrolü.
    
- **account**: Hesap yönetimi ile ilgili kontrolleri yapar. Örneğin, kullanıcının hesabının süresinin dolup dolmadığını kontrol eder.
    
- **pam_unix.so**: Geleneksel Unix kimlik doğrulama yöntemlerini kullanır (`/etc/passwd` ve `/etc/shadow` dosyalarını okur).
    
- **pam_listfile.so**: Belirli bir dosyadaki kullanıcıları engellemek için kullanılır (örneğin, `/etc/vsftpd/ftpusers` dosyasındaki kullanıcıların FTP erişimini engeller).
    

### PAM'ın Avantajları

- **Esneklik:** Farklı kimlik doğrulama yöntemleri (şifre, LDAP, kerberos, vs.) kolayca entegre edilebilir.
    
- **Merkezi Yönetim:** Tüm uygulamalar için kimlik doğrulama politikaları tek bir yerden yönetilebilir.
    
- **Güvenlik:** Karmaşık kimlik doğrulama süreçleri, uygulamalardan bağımsız olarak yönetilebilir.
    

### Özet

- **PAM**, Linux sistemlerinde kimlik doğrulama işlemlerini yöneten bir mekanizmadır.
    
- **pam_service_name=vsftpd** ayarı, VSFTPD'nin `/etc/pam.d/vsftpd` dosyasını kullanarak kimlik doğrulama yapacağını belirtir.
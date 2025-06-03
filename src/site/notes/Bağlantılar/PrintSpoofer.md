---
{"dg-publish":true,"permalink":"/baglantilar/print-spoofer/"}
---



**PrintSpoofer**, Windows sistemlerinde local ayrıcalık yükseltme (privilege escalation) yapmaya yarayan bir araçtır. Özellikle **SeImpersonatePrivilege** veya **SeAssignPrimaryTokenPrivilege** yetkilerine sahip kullanıcılar için tasarlanmıştır ve Print Spooler servisinin güvenlik açığından yararlanır. Print Spooler, yazıcı işlerini yönetir, ancak bu hizmetteki güvenlik zafiyetleri kullanılarak daha yüksek ayrıcalıklı bir kimliğe bürünmek mümkündür.

PrintSpoofer aracı, Print Spooler servisinin kimlik taklit yeteneklerini manipüle ederek düşük yetkili bir kullanıcının **NT AUTHORITY\SYSTEM** haklarını almasını sağlar.
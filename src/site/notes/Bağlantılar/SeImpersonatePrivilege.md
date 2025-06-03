---
{"dg-publish":true,"permalink":"/baglantilar/se-impersonate-privilege/"}
---



**SeImpersonatePrivilege** yetkisi, bir Windows kullanıcısının veya process'in başka bir kullanıcının kimliğine bürünmesine izin verir. Bu yetki, bir process'in başka bir process'in yetkilerini geçici olarak üstlenmesini sağlar, bu da özellikle ayrıcalık yükseltme saldırılarında kullanılabilecek önemli bir yetkidir.

Bu yetkiye sahip bir kullanıcı, **"Token Impersonation"** adı verilen tekniklerle sistemdeki daha yüksek yetkili kullanıcıların kimlik bilgilerini geçici olarak kullanabilir. Örneğin, SeImpersonatePrivilege yetkisi, JuicyPotato ve RottenPotato gibi "token stealing" (token çalma) saldırılarında kullanılarak daha yüksek ayrıcalıklara erişim sağlamak için tercih edilir.

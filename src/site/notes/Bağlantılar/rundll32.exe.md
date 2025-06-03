---
{"dg-publish":true,"permalink":"/baglantilar/rundll32-exe/"}
---


`rundll32.exe`, Windows işletim sisteminde yerleşik bir programdır ve DLL (Dynamic Link Library) dosyalarını çalıştırmak için kullanılır. Özellikle belirli fonksiyonları ve uygulamaları çağırmak amacıyla DLL'leri hafızada çalıştırmak gerektiğinde `rundll32.exe` kullanılır. Bu araç, bir DLL dosyasında yer alan fonksiyonları doğrudan çalıştırmayı sağlar.

### Neden `lsass.exe`'i Dump Ederken Kullanılır?

`lsass.exe` (Local Security Authority Subsystem Service), Windows'ta kimlik doğrulama işlemlerini yöneten bir sistem sürecidir ve kullanıcı oturum açma bilgilerini saklar. `lsass.exe`'in belleğini dump ederek (hafıza dökümü alarak), içinde saklanan parolalar, hash'ler ve kimlik doğrulama token'ları gibi hassas bilgileri analiz edebiliriz.

`rundll32.exe`, bu dump işlemi sırasında kullanışlıdır çünkü belirli bir DLL dosyasını çalıştırarak hafıza dökümünü gerçekleştirebiliriz.
---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/1-bash-home/","created":"2025-06-13T02:37:01.095+03:00","updated":"2025-06-13T02:45:58.263+03:00"}
---


## Learn Bash

* Bash, Linux sistemlerinde komut dosyaları yazmak ve komutları çalıştırmak için kullanılır.
* Görevleri otomatikleştirmeye, sistem işlemlerini yönetmeye ve üretkenliği artırmaya yardımcı olur.

## Understanding Shells

Shell, bilgisayarınızla konuşmanızı sağlayan metin tabanlı bir arayüzdür.

Farklı shell türleri vardır, ancak Bash (Bourne Again SHell) güçlü ve kullanımı kolay olduğu için en popüler olanıdır.

Shell Çeşitleri:

* `Bourne Shell (sh)`: Stephen Bourne tarafından geliştirilen orijinal Unix shell.
* `C Shell (csh)`: C benzeri sözdizimi ile bilinir, etkileşimli kullanım için popülerdir.
* `Korn Shell (ksh)`: sh ve csh özelliklerini birleştirerek gelişmiş script yetenekleri sunar.
* `Bash (Bourne Again SHell)`: command history ve tab completion gibi ek özelliklerle sh'nin geliştirilmiş bir versiyonu.

**Why Use Bash?**

* Unix/Linux sistemlerinde yaygın olarak kullanılabilir ve scriptleri taşınabilir hale getirir.
* Döngüler, koşullular ve fonksiyonlar dahil olmak üzere güçlü script özelliklerini destekler
* Kullanım kolaylığı için command history ve tab completion sağlar.
* Otomasyon için diğer Unix/Linux araçlarıyla entegre edilebilir.

## Learning by Examples

Bu eğitimde, size bunun gibi Bash komutlarını göstereceğiz:

```shell-session
bash --version
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

Yeni kullanıcılar için terminal görünümünü kullanmak biraz karmaşık görünebilir.

Endişelenmeyin! Bunu gerçekten basit tutacağız ve bu şekilde öğrenmek size Bash'in nasıl çalıştığını iyi bir şekilde kavratacaktır.

Yukarıdaki kodda komutları (input) ve output'u görebilirsiniz.

Bunun gibi satırlar girdiğimiz komutlardır:

```
bash --version
```

Bunun gibi satırlar komutlarımızın output/response:

```
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

Genel olarak, önünde `$` olan satırlar input'tur. Bunlar terminalinizde kopyalayıp çalıştırabileceğiniz komutlardır.

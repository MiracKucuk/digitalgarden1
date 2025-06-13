---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/8-bash-echo-command/","created":"2025-06-13T15:18:47.349+03:00","updated":"2025-06-13T16:02:51.404+03:00"}
---

## Using the `echo` Command

`echo` komutu, bir metin satırını veya bir değişkenin değerini terminalde göstermek için kullanılır.

## Basic Usage

Basit bir mesaj görüntülemek için `echo “message”` kullanın:

```shell
echo "Hello, World!"

Hello, World!
```

## Options Overview

`echo` komutunun çıktısını özelleştirmek için çeşitli seçenekleri vardır:

* `-n` - Sonuna yeni bir satır eklemeyin
* `-e` - Yeni satırlar için \n gibi özel karakterlere izin ver
* `-E` - Özel karakterlere izin verme (varsayılan)

## `-n` Option: No Trailing Newline

 `-n` seçeneği `echo`'nun çıktının sonuna yeni bir satır eklemesini engeller.

 Bu, çıktıya aynı satırda devam etmek istediğinizde kullanışlıdır.

```shell
echo -n "Hello,";echo " World!"
Hello, World!
```

## `-e` Option: Enable Backslash Escapes

`-e` seçeneği, yeni satırlar için `\n`, sekmeler için `\t` gibi backslash escapes kullanımını etkinleştirir.

Bu, daha formatlı çıktı elde edilmesini sağlar.

```shell
echo -e "Hello\nWorld!"
Hello
World!
```

## `-E` Option: Disable Backslash Escapes

`-E` seçeneği, varsayılan davranış olan backslash kaçışlarının kullanımını devre dışı bırakır.

Bu, metnin tam olarak yazıldığı gibi çıktılanmasını sağlar.

```shell
echo -E "Hello\nWorld!"
Hello\nWorld!
```

## Using `echo` in Scripts

`echo` komutu genellikle debugging veya loglama bilgileri için scriptlerde kullanılır. Mesajları terminale yazdırarak script'inizde neler olduğunu görmenize yardımcı olur.

```bash
#!/bin/bash
echo "Starting the script..."
# Your script commands here
echo "Script finished."
```
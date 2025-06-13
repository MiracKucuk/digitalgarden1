---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/3-bash-get-started/","created":"2025-06-13T12:20:55.360+03:00","updated":"2025-06-13T14:56:47.032+03:00"}
---


## Setting Up Bash

Çoğu Unix/Linux sistemi Bash önceden yüklenmiş olarak gelir.

Bash'in kurulu olup olmadığını kontrol etmek için bir terminal açın ve şunu yazın:

```
bash --version
```

Bash yüklü değilse, sisteminizin paket yöneticisini kullanarak yükleyebilirsiniz.

Örneğin, Ubuntu/Debian'da şunu yazın:

```
sudo apt-get install bash
```

macOS'te Bash'i Homebrew aracılığıyla yükleyebilirsiniz:

```
brew install bash
```

Windows'ta, Linux çalıştırmak için WSL (Linux için Windows Subsystem) kullanabilir veya sadece Windows için Git Bash kullanabilirsiniz.

## Running Bash Commands

```
bash --version
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

Bu komut sisteminizde yüklü olan Bash sürümünü gösterir.

Ayrıca `.sh` uzantılı bir dosyaya scriptler (komutların bir listesi) yazabilirsiniz:

**Simple Script Example:**

```bash
 #!/bin/bash
echo "Hello, Bash!"
```

Bunu `hello.sh` adlı bir dosyaya kaydedin ve şu şekilde çalıştırın:

```
bash hello.sh
```

```Output
Hello, Bash!
```


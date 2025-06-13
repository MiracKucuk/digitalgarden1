---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/bash-cd-change-directory/","created":"2025-06-13T15:01:33.198+03:00","updated":"2025-06-13T15:08:10.792+03:00"}
---


## Using the `cd` Command

* `cd` komutu, terminaldeki geçerli çalışma dizinini değiştirmek için kullanılır.

## Basic Usage

Belirli bir dizine geçmek için `cd dizin_adı` seçeneğini kullanın:

```shell
cd my_directory
```

## Options Overview

 `cd` komutu, dizinlerde gezinmek için çeşitli yararlı seçenekleri destekler:
* `cd ..`: Bir dizin seviyesi yukarı git
* `cd ~`: Home dizinine geç
* `cd -`: Önceki dizine geçin
* `cd /`: Root dizinine geç

## `cd ..` Option: Move Up One Directory Level

* `cd ..` komutu, mevcut klasörünüzün bir üstündeki klasöre gitmenizi sağlar.
* Bir üst klasöre gitmeniz gerektiğinde kullanışlıdır.

```
cd ..
```

## `cd ~` Option: Change to Home Directory

`cd ~` komutu sizi kullanıcı hesabınız için varsayılan dizin olan home dizininize götürür.

Bu seçenek, çeşitli dizinler arasında gezindikten sonra başlangıç noktanıza dönmeniz gerektiğinde kullanışlıdır.

```shell
cd ~
```

## `cd -` Option: Switch to Previous Directory

`cd -` komutu çalışma dizininizi bulunduğunuz bir önceki dizine geçirir.

Bu seçenek, tam yollarını tekrar tekrar yazmaya gerek kalmadan iki dizin arasında geçiş yapmak için kullanışlıdır.

```
cd -
```

## `cd /` Option: Change to Root Directory

`cd /` komutu sizi dosya sisteminin root dizinine götürür.

Bu seçenek, sistem genelindeki dosyalara veya dizinlere erişmeniz gerektiğinde kullanışlıdır.

```shell
cd /
```

## Seçeneklerin Birleştirilmesi

`cd` komutunun kendisi seçenekleri birleştirmeyi desteklemez, ancak gezinme verimliliğini artırmak için diğer komutlarla birlikte kullanabilirsiniz.

Örneğin, `cd`'yi `ls` ile birlikte kullanmak, gittiğiniz bir dizinin içeriğini size hızlı bir şekilde gösterebilir:

```shell
cd my_directory && ls

copy_of_my_file.txt  Cosmere_RPG_Beta_Rules_Preview.pdf
images/  my_file.txt  myfolder/  report.csv  voiceover.wav
```
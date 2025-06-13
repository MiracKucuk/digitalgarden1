---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/10-bash-cp-copy-files-ve-directories/","created":"2025-06-13T16:29:43.633+03:00","updated":"2025-06-13T16:40:49.356+03:00"}
---


## Using the `cp` Command

`cp` komutu, dosya ve dizinleri bir konumdan diğerine kopyalamak için kullanılır

Bu, dosyanızın veya klasörünüzün bir kopyasını oluşturmak gibidir.

## Basic Usage

Bir dosyayı kopyalamak için `cp source_file destination_file` kullanın:

```
cp my_file.txt copy_of_my_file.txt
```

## Options

`cp` komutunun çalışma şeklini değiştirmek için seçenekleri vardır:

`-r` - Bir dizin içindeki tüm dosya ve klasörleri kopyalar
`-i` - Dosyaları değiştirmeden önce sor
`-u` - Yalnızca kaynak daha yeniyse kopyala
`-v` - Verbose modu, kopyalanan dosyaları gösterir


## `-v` Option: Verbose Mode

`-v` seçeneği, kopyalanmakta olan dosyaları terminalde görüntüleyen ayrıntılı modu etkinleştirir.

Bu, özellikle çok sayıda dosyayla uğraşırken kopyalama işlemini izlemek için kullanışlıdır.

```shell
cp -v my_file.txt copy_of_my_file.txt
'my_file.txt' -> 'copy_of_my_file.txt'
```

## Copy Directories Recursively

`-r` seçeneği, tüm dosyalar ve alt dizinler dahil olmak üzere tüm dizinleri kopyalamanızı sağlar.

```
cp -r images images2
```

## Üzerine Yazmadan Önce Sor

`-i` seçeneği dosyaların üzerine yazmadan önce sizi uyararak yanlışlıkla yapılan değişiklikleri önlemenize yardımcı olur.

```shell
cp -i my_file.txt copy_of_my_file.txt

cp: overwrite copy_of_my_file.txt?
```

## Yalnızca Yeni Dosyaları Kopyalama

`-u` seçeneği dosyaları yalnızca kaynak dosya hedef dosyadan daha yeniyse kopyalar.

```shell
cp -u new_file.txt existing_file.txt
```

## Using `cp` with Wildcards

Joker karakterler aynı anda birden fazla dosyayı kopyalamanıza olanak tanır. Örneğin, `cp *.txt /destination/` tüm text dosyalarını hedef klasöre kopyalayacaktır.

```shell
cp *.txt /destination/
```

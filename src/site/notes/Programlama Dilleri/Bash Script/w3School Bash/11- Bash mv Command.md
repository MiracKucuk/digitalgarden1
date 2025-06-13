---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/11-bash-mv-command/","created":"2025-06-13T17:45:53.614+03:00","updated":"2025-06-13T17:54:48.801+03:00"}
---


# Move or Rename Files

## Using the `mv` Command

`mv` komutu dosya ve dizinleri taşımak veya yeniden adlandırmak için kullanılır.

Bu, bir dosyanın nerede olduğunu ya da adının ne olduğunu değiştirmek gibidir.

## Basic Usage

Bir dosyayı taşımak için `mv source_file destination_directory` kullanın:

```shell
mv my_file.txt /path/to/destination/
```


## Renaming Files

Bir dosyayı yeniden adlandırmak için `mv eski_ad yeni_ad` seçeneğini kullanın:

```
mv old_name.txt new_name.txt
```

## Options Overview

`mv` komutu, davranışını özelleştirmek için çeşitli seçeneklere sahiptir:

* `-i` - Dosyaları değiştirmeden önce sor
* `-u` - Yalnızca kaynak daha yeniyse taşı
* `-v` - Verbose modu, taşınan dosyaları gösterir


## `-i` Option: Prompt Before Overwrite

`-i` seçeneği dosyaların üzerine yazmadan önce sizi uyararak yanlışlıkla yapılan değişiklikleri önlemenize yardımcı olur.

```
mv -i my_file.txt myfolder/
mv: overwrite 'myfolder/my_file.txt'?
```

## `-u` Option: Move Only Newer Files

`-u` seçeneği dosyaları yalnızca kaynak dosya hedef dosyadan daha yeni ise taşır.

```
mv -u new_file.txt /path/to/destination/
```


## `-v` Option: Verbose Mode

`-v` seçeneği, taşınan dosyaları terminalde görüntüleyen ayrıntılı modu etkinleştirir.

Bu, özellikle çok sayıda dosyayla uğraşırken taşıma işlemini izlemek için kullanışlıdır.

```
mv -v my_file.txt myfolder/new_directory/  
renamed 'my_file.txt' -> 'myfolder/new_directory/my_file.txt'
```

## Using `mv` with Wildcards

Wildcard'lar birden fazla dosyayı aynı anda taşımanıza olanak tanır. Örneğin, `mv *.txt /destination/` tüm metin dosyalarını hedef klasöre taşıyacaktır.

```
mv *.txt /destination/
```


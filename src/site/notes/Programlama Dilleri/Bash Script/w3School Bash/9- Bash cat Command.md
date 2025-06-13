---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/9-bash-cat-command/","created":"2025-06-13T16:03:26.667+03:00","updated":"2025-06-13T16:25:20.715+03:00"}
---


# Dosyaları Birleştirme ve Görüntüleme

## Using the `cat` Command

`cat` komutu terminaldeki dosyaların içeriğini göstermek için kullanılır.

Birden fazla dosyayı tek bir dosyada birleştirmek için de kullanabilirsiniz.

## Basic Usage

Bir dosyanın içeriğini görüntülemek için `cat filename` kullanın:

```shell-session
cat my_file.txt

Understanding Shells
A shell is a text-based interface that lets you talk to your computer.

There are different types of shells. Bash (Bourne Again SHell)
is popular because it's powerful and easy to use.
```

## Options

`cat` komutunun metni gösterme şeklini değiştirmek için seçenekleri vardır:

`-n` - Her satıra sayı ekler
`-b` - Yalnızca metin içeren satırlara sayı ekleme
`-s` - Fazladan boş satırları kaldırın
`-v` - Yazdırılmayan karakterleri göster (sekmeler ve satır sonu hariç)

## `-n` Option: Number All Lines

`-n` seçeneği çıktının her satırına sayı ekler.

```shell
cat -n my_file.txt
    1  Understanding Shells
    2  A shell is a text-based interface that lets you talk to your computer.
    3
    4  There are different types of shells. Bash (Bourne Again SHell)
    5  is popular because its powerful and easy to use.
```

## `-b` Option: Number Non-Blank Lines

`-b` seçeneği, boş satırları göz ardı ederek yalnızca metin içeren satırlara sayı ekler.

```shell
cat -b my_file.txt
    1  Understanding Shells
    2  A shell is a text-based interface that lets you talk to your computer.

    3  There are different types of shells. Bash (Bourne Again SHell)
    4  is popular because its powerful and easy to use.
```

## `-s` Option: Suppress Repeated Empty Lines

`-s` seçeneği, çıktıdaki fazladan boş satırları kaldırır ve birden fazla boş satırın bulunduğu yerde yalnızca tek bir boş satır bırakır.

```shell
cat -s my_file.txt
Understanding Shells
A shell is a text-based interface that lets you talk to your computer.
There are different types of shells. Bash (Bourne Again SHell)
is popular because its powerful and easy to use.
```

## `-v` Option: Show Non-Printing Characters

`-v` seçeneği, sekmeler ve satır sonu karakterleri hariç, yazdırılmayan karakterleri görünür hale getirir.

Bu, gizli karakterler içeren dosyalarda debug yapmak için kullanışlıdır.

```
cat -v my_file.txt
```

## İki Dosyayı Birleştirme

`cat` komutu birden fazla dosyayı tek bir dosyada birleştirmek için kullanılabilir.

Bu, dosyaları birleştirmek veya mevcut bir dosyaya içerik eklemek için kullanışlıdır.

```shell
cat file1.txt file2.txt > combined.txt
```

Bu komut `file1.txt` ve `file2.txt` dosyalarının içeriğini alır ve bunları `combined.txt` dosyasına yazar.

## Using `cat` with Piping

`cat` komutu genellikle dosyaların içeriğini diğer komutlara göndermek için piping ile birlikte kullanılır.

Bu, metin verilerini işlemek için kullanışlıdır.

```shell
cat my_file.txt | grep "shells"
Understanding Shells
There are different types of shells. Bash (Bourne Again SHell)
```
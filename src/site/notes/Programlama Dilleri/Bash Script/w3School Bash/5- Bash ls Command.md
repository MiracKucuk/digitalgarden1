---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/5-bash-ls-command/","created":"2025-06-13T12:37:25.201+03:00","updated":"2025-06-13T14:56:07.182+03:00"}
---


## Using the `ls` Command

* `ls` komutu bir dizinin içeriğini listelemek için kullanılır.
* `ls` komutu dosyaları, dizinleri ve bunlarla ilgili bilgileri görüntüleyebilir.

## Basic Usage

Geçerli klasörde ne olduğunu görmek için `ls` kullanın:

```shell
ls

Cosmere_RPG_Beta_Rules_Preview.pdf  images/
my_file.txt  report.csv  voiceover.wav
```

## Options Overview

`ls` komutu, çıktısını özelleştirmek için çeşitli seçeneklere sahiptir:

- **`-l`** – Uzun liste formatı
- **`-a`** – Gizli dosyaları da dahil et
- **`-h`** – Boyutları insan tarafından okunabilir biçimde göster (örn. KB, MB)
- **`-t`** – Dosyaları değiştirilme zamanına göre sırala
- **`-r`** – Sıralamayı tersine çevir (azdan çoğa yerine çoktan aza gibi)
- **`-R`** – Alt dizinleri de yinelemeli (recursive) olarak listele
- **`-S`** – Dosyaları boyutlarına göre sırala
- **`-1`** – Her dosyayı bir satıra gelecek şekilde listele
- **`-d`** – Dizinlerin içeriğini değil, sadece kendilerini listele
- **`-F`** – Girdilere gösterge ekle (örneğin: `*` yürütülebilir, `/` dizin, `=` soket vb.)

## Long Listing Format

`-l` seçeneği size dosya ve klasörler hakkında ayrıntılı bilgi verir.

Aşağıdaki gibi bilgileri görüntüler:

- **file permissions** – dosya izinleri
- **number of links** – bağlantı (link) sayısı
- **owner name** – dosya sahibinin adı
- **owner group** – dosya sahibinin grubu
- **file size** – dosya boyutu
- **time of last modification** – son değiştirilme zamanı
- **file or directory name** – dosya veya dizin adı

Bu format, dosya attribute'lerine kapsamlı bir genel bakış elde etmek için kullanışlıdır.

```shell
ls -l

total 24232
-rw-r--r-- 1 user 197609 23777028 Jan 15 20:38 Cosmere_RPG_Beta_Rules_Preview.pdf
drwxr-xr-x 1 user 197609        0 Apr  9 07:46 images/
-rw-r--r-- 1 user 197609      890 Apr  9 07:48 my_file.txt
-rw-r--r-- 1 user 197609   305366 Apr  9 07:48 report.csv
-rw-r--r-- 1 user 197609   720974 Apr  9 07:47 voiceover.wav
```

* `-a` seçeneği gizli dosyaları da listeye dahil eder.
* Unix/Linux sistemlerindeki gizli dosyalar nokta ile başlar (örneğin, .bashrc).
* Bu seçenek, varsayılan olarak görünmeyen yapılandırma dosyalarını görüntülemeniz veya yönetmeniz gerektiğinde yardımcı olur.

```shell
ls -a

./  ../  .my_secret_file  Cosmere_RPG_Beta_Rules_Preview.pdf
images/  my_file.txt  report.csv  voiceover.wav
```

## Human-Readable Sizes

* `-h` seçeneği, bayt sayılarını kilobayt (K), megabayt (M), gigabayt (G) vb. dönüştürerek dosya boyutlarının okunmasını kolaylaştırır.
* Bu seçenek özellikle baytları elle dönüştürmeden dosya ve dizinlerin boyutunu hızlı bir şekilde değerlendirmek istediğinizde kullanışlıdır.

```shell
ls -lh

total 24M
-rw-r--r-- 1 user 197609  23M Jan 15 20:38 Cosmere_RPG_Beta_Rules_Preview.pdf
drwxr-xr-x 1 user 197609    0 Apr  9 07:51 images/
-rw-r--r-- 1 user 197609  890 Apr  9 07:48 my_file.txt
-rw-r--r-- 1 user 197609 299K Apr  9 07:48 report.csv
-rw-r--r-- 1 user 197609 705K Apr  9 07:47 voiceover.wav
```

## Sorting by Time

* `-t` seçeneği, dosyaları ve dizinleri değişiklik zamanına göre sıralar ve en son değiştirilen dosyalar önce gelir.
* Bu seçenek, en son güncellenen dosyaları ilk olarak görmek istediğinizde kullanışlıdır.

```shell
ls -t

images/  my_file.txt  report.csv  voiceover.wav
Cosmere_RPG_Beta_Rules_Preview.pdf
```

## Reverse Order

* `-r` seçeneği sıralama düzenini tersine çevirir.
* `-t` gibi diğer seçeneklerle birlikte kullanıldığında, en eski dosyaları önce görüntüleyebilir.
* Bu seçenek, özel ihtiyaçları karşılamak için varsayılan sıralama davranışını tersine çevirmek için kullanışlıdır.

```shell
ls -r

voiceover.wav  report.csv  my_file.txt  images/
Cosmere_RPG_Beta_Rules_Preview.pdf
```

## Recursive Listing

* `-R` seçeneği dizinleri ve içeriklerini recursive (özyinelemeli) olarak listeler.
* Bu, tüm dizin ağacını görüntülemek için kullanışlıdır.

```shell
ls -R

.:
copy_of_my_file.txt  Cosmere_RPG_Beta_Rules_Preview.pdf
images/  my_file.txt  myfolder/  report.csv  voiceover.wav

./images:
1.png  2.png  3.png  4.png

./myfolder:
my_file.txt  new_directory/

./myfolder/new_directory:
```

## Sort by Size

* `-S` seçeneği dosyaları boyutlarına göre sıralar, en büyük dosyalar önce gelir.
* Bu, bir dizindeki büyük dosyaları hızlı bir şekilde tanımlamak için yararlıdır.

```shell
ls -S

Cosmere_RPG_Beta_Rules_Preview.pdf  voiceover.wav  report.csv
copy_of_my_file.txt  my_file.txt  images/  myfolder/
```

## One File per Line

* `-1` seçeneği her satırda bir dosya listeler, bu da scriptler için veya çıktıyı diğer komutlara aktarırken kullanışlıdır.

```shell
ls -1

Cosmere_RPG_Beta_Rules_Preview.pdf
images/
my_file.txt
report.csv
voiceover.wav
```

## Directories Only

* `-d` seçeneği dizinlerin içeriğini değil kendilerini listeler.
* Bu, içindekiler olmadan dizin adlarını görmek için kullanışlıdır.

```shell
ls -d */

images//  myfolder//
```

## Append Indicator

`-F` seçeneği entry'lere bir indicator karakteri ekler (örneğin, dizinler için `/`, çalıştırılabilir dosyalar için `*`).

```shell
ls -F

Cosmere_RPG_Beta_Rules_Preview.pdf  images/
my_file.txt  report.csv  voiceover.wav
```

## Birden Fazla Seçenek Kullanma

* Daha karmaşık komutlar oluşturmak için birden fazla seçeneği birleştirebilirsiniz.
* Örneğin, `ls -l -a` gizli dosyalar da dahil olmak üzere tüm dosya ve dizinlerin ayrıntılı bir listesini görüntüler.

```shell
ls -l -a

total 24248
drwxr-xr-x 1 user 197609        0 Apr  9 07:53 ./
drwxr-xr-x 1 user 197609        0 Apr  9 07:42 ../
-rw-r--r-- 1 user 197609      890 Apr  9 07:48 .my_secret_file
-rw-r--r-- 1 user 197609 23777028 Jan 15 20:38 Cosmere_RPG_Beta_Rules_Preview.pdf
drwxr-xr-x 1 user 197609        0 Apr  9 07:51 images/
-rw-r--r-- 1 user 197609      890 Apr  9 07:48 my_file.txt
-rw-r--r-- 1 user 197609   305366 Apr  9 07:48 report.csv
-rw-r--r-- 1 user 197609   720974 Apr  9 07:47 voiceover.wav
```

Ayrıca birden fazla seçeneği aralarında boşluk bırakmadan birleştirebilirsiniz. Örneğin, `ls -la`:

```shell
ls -la

total 24248
drwxr-xr-x 1 user 197609        0 Apr  9 07:53 ./
drwxr-xr-x 1 user 197609        0 Apr  9 07:42 ../
-rw-r--r-- 1 user 197609      890 Apr  9 07:48 .my_secret_file
-rw-r--r-- 1 user 197609 23777028 Jan 15 20:38 Cosmere_RPG_Beta_Rules_Preview.pdf
drwxr-xr-x 1 user 197609        0 Apr  9 07:51 images/
-rw-r--r-- 1 user 197609      890 Apr  9 07:48 my_file.txt
-rw-r--r-- 1 user 197609   305366 Apr  9 07:48 report.csv
-rw-r--r-- 1 user 197609   720974 Apr  9 07:47 voiceover.wav
```


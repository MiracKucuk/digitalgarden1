---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/w3-school-bash/7-bash-pwd-command/","created":"2025-06-13T15:12:08.261+03:00","updated":"2025-06-13T15:18:23.306+03:00"}
---

# Print Working Directory

## Using the `pwd` Command

`pwd` komutu size o anda içinde bulunduğunuz klasörün tam yolunu gösterir.

## Basic Usage

Geçerli klasörün tam yolunu görmek için `pwd` yazmanız yeterlidir:

```
pwd
/home/user/my_directory
```

## Options Overview

`pwd` komutu, çıktısını özelleştirmek için birkaç seçeneği destekler:

* `-L`: Logical geçerli çalışma dizinini görüntüleme
* `-P`: Fiziksel geçerli çalışma dizinini görüntüler (sembolik bağlantılar olmadan)

## `-L` Option: Logical Path

`-L` seçeneği, sembolik bağlantılar da dahil olmak üzere mantıksal yolu gösterir.

Bu, pwd'nin varsayılan davranışıdır.

```shell
pwd -L

/home/user/my_directory
```

## `-P` Option: Physical Path

`-P` seçeneği, sembolik bağlantıları gerçek konumlarına çözümleyerek fiziksel yolu gösterir.

Bu, tam fiziksel dizin yapısını bilmeniz gerektiğinde kullanışlıdır.

```shell
pwd -P

/
```

## Mevcut Dizininizi Bilmek Neden Önemlidir?

Dosya sisteminde dolaşırken hangi klasörde olduğunuzu bilmek önemlidir. Göreceli yollar kullanan komutları çalıştırırken doğru yerde olduğunuzdan emin olmanıza yardımcı olur.

## `pwd` için Yaygın Kullanım Durumları

* Scriptleri Çalıştırma: Scriptinizin doğru dosya ve klasörleri kullandığından emin olun.
* Dosyaları Yönetme: Hatalardan kaçınmak için dosyaları kopyalamadan veya taşımadan önce nerede olduğunuzu kontrol edin.
* Sorunları Düzeltme: Mevcut klasörünüzü bilmek, yolla ilgili sorunları çözmenize yardımcı olabilir.
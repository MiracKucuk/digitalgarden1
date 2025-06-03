---
{"dg-publish":true,"permalink":"/baglantilar/bahsedilen-8-yoentemin-detayli-aciklamasi/"}
---


## 1. **OpenRead**

### Detaylı Açıklama:

- **Ne yapar?**: OpenRead, belirtilen URL’den veriyi **stream (akış)** olarak alır. Tüm veriyi belleğe yüklemek yerine, ihtiyaç duydukça parça parça okumanıza olanak tanır.
- **Ne zaman kullanılır?**:
    - Dosyayı sabit diske kaydetmek yerine doğrudan işlemek istediğinizde.
    - Büyük boyutlu verilerle çalışırken bellekteki yükü azaltmak için.
- **Kısıtlamaları**:
    - Veriyi doğrudan sabit diske yazmaz. Akışı manuel olarak işlemeniz gerekir.

```
# Akıştan veriyi okuyarak işleme

$webClient = New-Object System.Net.WebClient

$stream = $webClient.OpenRead("https://www.example.com/largefile.txt")

$streamReader = New-Object System.IO.StreamReader($stream)

while (-not $streamReader.EndOfStream) {

$line = $streamReader.ReadLine()

Write-Output $line # Her satırı işleyebilirsiniz

}

$stream.Close()

$streamReader.Close()
```


## 2. **OpenReadAsync**

### Detaylı Açıklama:

- **Ne yapar?**: OpenRead ile aynı işlemi yapar, ancak asenkron olarak çalışır. Uygulamanızın diğer işlemleri duraksamadan çalışmaya devam eder.
- **Ne zaman kullanılır?**:
    - Büyük bir dosya akışı indirirken, uygulamanızın kullanıcı arayüzünün donmasını önlemek istediğinizde.
    - Veriyi arka planda indirip işleyeceğiniz zaman.

```
$webClient = New-Object System.Net.WebClient

$webClient.OpenReadCompleted += {

param($sender, $eventArgs)

$reader = New-Object System.IO.StreamReader($eventArgs.Result)

while (-not $reader.EndOfStream) {

$line = $reader.ReadLine()

Write-Output $line

}

$reader.Close()

}

$webClient.OpenReadAsync("https://www.example.com/largefile.txt")
```


## 3. **DownloadData**

### Detaylı Açıklama:

- **Ne yapar?**: Belirtilen URL’den veriyi indirir ve **byte dizisi** olarak belleğe yükler.
- **Ne zaman kullanılır?**:
    - Veriyi bellekte işlemek istediğinizde.
    - Örneğin, bir resim dosyasını veya ikili (binary) veriyi belleğe almak istediğinizde.


```
$webClient = New-Object System.Net.WebClient

$data = $webClient.DownloadData("https://www.example.com/file.zip")

Write-Output "Veri uzunluğu: $($data.Length)"
```


## 4. **DownloadDataAsync**

### Detaylı Açıklama:

- **Ne yapar?**: DownloadData’nın asenkron versiyonudur. Veriyi indirirken uygulamanızın diğer işlemleri devam eder.
- **Ne zaman kullanılır?**:
    - Büyük verilerle çalışıyorsanız ve uygulamanızın diğer bölümlerinin çalışmaya devam etmesini istiyorsanız.
- **Kısıtlamaları**:
    - Sonuç veriyi işlemek için bir olay (event) mekanizması kullanmanız gerekir.

```
$webClient = New-Object System.Net.WebClient

$webClient.DownloadDataCompleted += {

param($sender, $eventArgs)

$data = $eventArgs.Result

Write-Output "İndirilen veri uzunluğu: $($data.Length)"

}

$webClient.DownloadDataAsync("https://www.example.com/file.zip")
```


## 5. **DownloadFile**

### Detaylı Açıklama:

- **Ne yapar?**: URL’deki veriyi indirir ve belirtilen bir dosya adıyla yerel diskte kaydeder.
- **Ne zaman kullanılır?**:
    - Dosyayı doğrudan sabit diskinize indirmek istediğinizde.
    - Örneğin, bir PDF dosyasını belirli bir konuma kaydetmek için.
- **Kısıtlamaları**:
    - Dosya indirildiğinde uygulama duraklayabilir (bloklama yapar).

```
$webClient = New-Object System.Net.WebClient

$webClient.DownloadFile("https://www.example.com/file.pdf", "C:\path\to\file.pdf")

Write-Output "Dosya başarıyla indirildi!"
```


## 6. **DownloadFileAsync**

### Detaylı Açıklama:

- **Ne yapar?**: DownloadFile’ın asenkron versiyonudur. Dosya indirildiği sırada uygulamanız çalışmaya devam eder.
- **Ne zaman kullanılır?**:
    - Kullanıcı arayüzü donmadan bir dosyayı indirmek istediğinizde.

```
$webClient = New-Object System.Net.WebClient

$webClient.DownloadFileCompleted += {

param($sender, $eventArgs)

Write-Output "Dosya başarıyla indirildi!"

}

$webClient.DownloadFileAsync("https://www.example.com/file.pdf", "C:\path\to\file.pdf")
```


## 7. **DownloadString**

### Detaylı Açıklama:

- **Ne yapar?**: Belirtilen URL’nin içeriğini bir metin dizesi (string) olarak indirir.
- **Ne zaman kullanılır?**:
    - Web sayfası içeriğini veya API’den JSON/HTML almak istediğinizde.
    - Örneğin, bir JSON API yanıtını okumak için.


```
$webClient = New-Object System.Net.WebClient

$content = $webClient.DownloadString("https://www.example.com/api/data")

Write-Output "İndirilen içerik: $content"
```


## 8. **DownloadStringAsync**

### Detaylı Açıklama:

- **Ne yapar?**: DownloadString’in asenkron versiyonudur. Metin indirildiği sırada uygulamanız çalışmaya devam eder.
- **Ne zaman kullanılır?**:
    - API’den gelen büyük metin verilerini indirirken uygulamanızın donmasını istemediğinizde.

```
$webClient = New-Object System.Net.WebClient

$webClient.DownloadStringCompleted += {

param($sender, $eventArgs)

$content = $eventArgs.Result

Write-Output "Asenkron indirilen içerik: $content"

}

$webClient.DownloadStringAsync("https://www.example.com/api/data")
```




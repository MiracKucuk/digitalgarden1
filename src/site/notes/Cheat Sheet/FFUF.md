---
{"dg-publish":true,"permalink":"/cheat-sheet/ffuf/"}
---


### 1- Klasik Fuzz

```
ffuf -w /worldlist:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

### 2-Extension Fuzzing

```
ffuf -w /worldlist:FUZZ -u http://SERVER_IP:PORT/indexFUZZ
```

Not: Seçtiğimiz kelime listesi  bir nokta (.) içeriyor ise , fuzzing işlemimizde "index" kelimesinden sonra nokta eklememiz gerekmeyecek.

### 3-Page Fuzzing

```
ffuf -w /worldlist:FUZZ -u http://SERVER_IP:PORT/FUZZ.php
```


### 4-Recursive Fuzzing + Extension Fuzzing

```
ffuf -w /worldlist:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```


### 5-Sub-domain Fuzzing

```
ffuf -w /worldlist:FUZZ -u https://FUZZ.example.com/
```

### 6-Vhost Fuzzing

```
ffuf -w /worldlist:FUZZ -u http://example.com:PORT/ -H 'Host: FUZZ.example.com'
```

#### 6.1 Filtering Results
```
ffuf -w /worldlist:FUZZ -u http://example.com:PORT/ -H 'Host: FUZZ.example.com' -fs 900
```


### 7-Parameter Fuzzing - GET

```
ffuf -w /worldlist:FUZZ -u http://blog.example.com:PORT/admin/admin.php?FUZZ=key -fs xxx
```


### 8-Parameter Fuzzing - POST (PHP)

```
ffuf -w /worldlist:FUZZ -u http://admin.example.com:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

#### 8.1-Parameter Fuzzing - POST (PHP) Doğrulama Curl

```
curl http://admin.example.com:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

#### 8.2-Value Fuzzing (Bulunan Parametreye)

```shell-session
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

```
ffuf -w ids.txt:FUZZ -u http://admin.example.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

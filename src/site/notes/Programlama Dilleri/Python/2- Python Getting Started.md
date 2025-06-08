---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/2-python-getting-started/","created":"2025-06-07T23:49:44.408+03:00","updated":"2025-06-08T00:39:28.652+03:00"}
---


## Python Install

Birçok PC ve Mac'te python zaten yüklü olacaktır.

Bir Windows PC'de python kurulu olup olmadığını kontrol etmek için başlat çubuğunda Python'u arayın veya Command Line'da (`cmd.exe`) aşağıdakileri çalıştırın:

```
C:\Users\_Your Name_>python --version
```

Linux veya Mac'te python kurulu olup olmadığını kontrol etmek için, Linux'ta komut satırını açın veya Mac'te Terminal'i açın ve şunu yazın:

```
python --version
```

Eğer bilgisayarınızda Python yüklü değilse, aşağıdaki web sitesinden ücretsiz olarak indirebilirsiniz: https://www.python.org/


## Python Quickstart
  
Python yorumlanmış bir programlama dilidir, bu da bir geliştirici olarak Python (.py) dosyalarını bir metin düzenleyicide yazdığınız ve daha sonra bu dosyaları çalıştırılmak üzere python yorumlayıcısına koyduğunuz anlamına gelir.

Herhangi bir metin düzenleyicide yapılabilecek hello.py adlı ilk Python dosyamızı yazalım:

`hello.py`:

```python
print("Hello, World!")
```

Bu kadar basit. Dosyanızı kaydedin. Komut satırınızı açın, dosyanızı kaydettiğiniz dizine gidin ve çalıştırın:

Çıktı şu şekilde olmalıdır:

```
Hello, World!
```

## Python Version

Düzenleyicinin Python sürümünü kontrol etmek için, `sys` modülünü import ederek bulabilirsiniz:

```python
import sys  
  
print(sys.version)
```

```output
python3 1.py
3.11.9 (main, Apr 10 2024, 13:16:36) [GCC 13.2.0]
```

Python Modülleri bölümümüzde modülleri import hakkında daha fazla bilgi edineceksiniz.

## Python Command Line

Python'da kısa bir miktarda kodu test etmek için bazen kodu bir dosyaya yazmamak en hızlı ve en kolay yoldur. Python komut satırı olarak çalıştırılabildiği için bu mümkündür.

Windows, Mac veya Linux komut satırına aşağıdakileri yazın:

```
C:\Users\_Your Name_>python
```

Ya da “`python`” komutu işe yaramadıysa, “`py`” komutunu deneyebilirsiniz:

```
C:\Users\_Your Name_>py
```

Buradan, eğitimin başındaki hello world örneğimiz de dahil olmak üzere herhangi bir python yazabilirsiniz:

```
─(root㉿kali)-[~/Desktop/python]
└─# python      
Python 3.11.9 (main, Apr 10 2024, 13:16:36) [GCC 13.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello python3")
Hello python3
```

Bu da komut satırına “`Hello python3`” yazacaktır:

Python komut satırında işiniz bittiğinde, python komut satırı arayüzünden çıkmak için aşağıdakileri yazabilirsiniz:

```
exit()
```



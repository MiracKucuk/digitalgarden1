---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/3-python-syntax/"}
---


## Execute Python Syntax

Python syntax'ı doğrudan Command Line'a yazılarak çalıştırılabilir:

```
>>> print("Hello, World!")  
Hello, World!
```

Ya da server üzerinde bir python dosyası oluşturarak, .py dosya uzantısını kullanarak ve Command Line'da çalıştırarak:

```
C:\Users\_Your Name_>python myfile.py
```


## Python Indentation (Girinti)

* Girinti, bir kod satırının başındaki boşlukları ifade eder.
* Diğer programlama dillerinde koddaki girinti sadece okunabilirlik içinken, Python'da girinti çok önemlidir.
* Python bir kod bloğunu belirtmek için girinti kullanır.

```python
if 5 > 2:  
  print("Five is greater than two!")
```

Eğer girintilemeyi atlarsanız Python size hata verecektir:

Syntax Error:

```python
if 5 > 2:  
print("Five is greater than two!")
```

Boşluk sayısı programcı olarak size bağlıdır, en yaygın kullanım dörttür, ancak en az bir olmak zorundadır.

```python
if 5 > 2:  
 print("Five is greater than two!")   
if 5 > 2:  
        print("Five is greater than two!")
```

Aynı kod bloğunda aynı sayıda boşluk kullanmanız gerekir, aksi takdirde Python size hata verecektir:

Syntax Error:

```python
if 5 > 2:  
 print("Five is greater than two!")  
        print("Five is greater than two!")
```


## Python Variables

Python'da değişkenler, siz ona bir değer atadığınızda oluşturulur:

```python
x = 5 
y = "Hello, World!"
```

Python'da değişken bildirmek için bir komut yoktur.

Python Değişkenleri bölümünde değişkenler hakkında daha fazla bilgi edineceksiniz.


## Comments

Python, kod içi dökümantasyon amacıyla yorumlama özelliğine sahiptir.

Yorumlar `#` ile başlar ve Python satırın geri kalanını bir yorum olarak işler:

```python
#This is a comment.
print("Hello World")
```


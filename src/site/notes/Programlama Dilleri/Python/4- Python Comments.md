---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/4-python-comments/","created":"2025-06-08T00:34:30.998+03:00","updated":"2025-06-08T00:39:36.900+03:00"}
---


# Python Comments

* Yorumlar Python kodunu açıklamak için kullanılabilir.
* Yorumlar kodu daha okunabilir hale getirmek için kullanılabilir.
* Yorumlar, kodu test ederken yürütmeyi önlemek için kullanılabilir.

## Creating a Comment

Yorumlar `#` ile başlar ve Python bunları görmezden gelir:

```python
#This is a comment  
print("Hello, World!")
```

Yorumlar bir satırın sonuna yerleştirilebilir ve Python satırın geri kalanını görmezden gelir:

```python
print("Hello, World!") #This is a comment
```

Yorum, kodu açıklayan bir metin olmak zorunda değildir, Python'un kodu çalıştırmasını önlemek için de kullanılabilir:

```python
#print("Hello, World!")  
print("Cheers, Mate!")
```


## Multiline Comments

Python çok satırlı yorumlar için bir sözdizimine sahip değildir.

Çok satırlı bir yorum eklemek için her satıra bir `#` ekleyebilirsiniz:

```python
#This is a comment  
#written in  
#more than just one line  
print("Hello, World!")
```

Ya da, tam olarak amaçlandığı gibi değil, çok satırlı bir string kullanabilirsiniz.

Python bir değişkene atanmamış string değerlerini göz ardı edeceğinden, kodunuza çok satırlı bir string (üç tırnak) ekleyebilir ve yorumunuzu bunun içine yerleştirebilirsiniz:

```python
"""  
This is a comment  
written in  
more than just one line  
"""  
print("Hello, World!")
```

String bir değişkene atanmadığı sürece, Python kodu okuyacak, ancak daha sonra görmezden gelecek ve çok satırlı bir yorum yapmış olacaksınız.



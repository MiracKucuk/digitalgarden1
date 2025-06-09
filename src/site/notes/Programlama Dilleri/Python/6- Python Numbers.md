---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/6-python-numbers/","created":"2025-06-09T14:35:58.392+03:00","updated":"2025-06-09T15:14:26.371+03:00"}
---

Python'da üç numerik tip vardır:

- `int`
- `float`
- `complex`

```python
x = 1    # int  
y = 2.8  # float  
z = 1j   # complex
```

Sayısal türdeki değişkenler, onlara bir değer atadığınızda oluşturulur:

Python'da herhangi bir objenin tipini doğrulamak için `type()` fonksiyonunu kullanın:

```python
print(type(x))
print(type(y))
print(type(z))
```

## Int

Int veya tamsayı, pozitif veya negatif, ondalıksız, sınırsız uzunlukta bir tam sayıdır.

Integers:

```python
x = 1
y = 35656222554887711
z = -3255522

print(type(x))
print(type(y))
print(type(z))
```

```output
<class 'int'>  
<class 'int'>  
<class 'int'>
```

## Float

Float veya “floating point number”, bir veya daha fazla ondalık içeren pozitif veya negatif bir sayıdır.

Floats:

```python
x = 1.10  
y = 1.0  
z = -35.59  
  
print(type(x))  
print(type(y))  
print(type(z))
```

Float, 10'un kuvvetini belirtmek için “`e`” ile bilimsel sayılar da olabilir.

```python
x = 35e3  
y = 12E4  
z = -87.7e100  
  
print(type(x))  
print(type(y))  
print(type(z))
```

```
<class 'float'>  
<class 'float'>  
<class 'float'>
```

## Complex

Karmaşık sayılar, imajiner kısım olarak bir “j” ile yazılır:

```python
x = 3+5j  
y = 5j  
z = -5j  
  
print(type(x))  
print(type(y))  
print(type(z))
```

```output
<class 'complex'>  
<class 'complex'>  
<class 'complex'>
```


## Type Conversion (Dönüşüm)

`int()`, `float()` ve `complex()` metotlarıyla bir type'dan diğerine dönüştürebilirsiniz:

Bir type'dan diğerine dönüştürün:

```python

x = 1   #int
y = 2.8 #float
z = 1j  #complex


#convert from int to float:
a = float(x)

#convert from float to int:
b = int(y)

#convert from int to complex:
c = complex(x)

print(a)
print(b)
print(c)

print(type(a))
print(type(b))
print(type(c))
```

```output
1.0  
2  
(1+0j)  
<class 'float'>  
<class 'int'>  
<class 'complex'>
```

Not: Karmaşık sayıları başka bir sayı türüne dönüştüremezsiniz.

## Random Number

Python'da rastgele bir sayı oluşturmak için `random()` fonksiyonu yoktur, ancak Python'da rastgele sayılar oluşturmak için kullanılabilecek `random` adında built-in bir modül vardır:

Random modülünü import edin ve 1'den 9'a kadar rastgele bir sayı görüntüleyin:

```python
import random

print(random.randrange(1, 10))
```

Random Modülünde Random modülü hakkında daha fazla bilgi edineceğiz.








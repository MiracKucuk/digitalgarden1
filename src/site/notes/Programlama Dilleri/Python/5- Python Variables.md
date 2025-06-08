---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/5-python-variables/","created":"2025-06-08T00:40:04.253+03:00","updated":"2025-06-08T17:09:39.840+03:00"}
---

## Variables

Değişkenler, veri değerlerini saklamak için kullanılan kaplardır.

## Creating Variables

Python'da değişken bildirmek için bir komut yoktur.

Bir değişken, ona ilk kez bir değer atadığınız anda oluşturulur.

```python
x = 5 
y = "John"
print(x)
print(y)
```

Değişkenlerin belirli bir türle bildirilmesine gerek yoktur ve hatta ayarlandıktan sonra tür değiştirebilirler.

```python
x = 4 # x is of type int
x = "Sally" # x is now of type str
print(x)
```

## Casting

Bir değişkenin data type'ını belirtmek isterseniz, bu casting ile yapılabilir.

```python
x = str(3) # x will be '3'
y = int(3) # y will be 3
z = float(3) # z will be 3.0
```

## Get the Type

Bir değişkenin veri türünü `type()` fonksiyonuyla elde edebilirsini

```python
x = 5 
y = "John"
print(type(x))
print(type(y))
```

```output
<class 'int'>  
<class 'str'>
```

Bu eğitimin ilerleyen bölümlerinde data types ve casting hakkında daha fazla bilgi edineceksiniz.

## Single or Double Quotes?


String değişkenleri tek ya da çift tırnak kullanılarak bildirilebilir:

```python
x = "John"
# is the same as
x = 'John'
```


## Case-Sensitive

Değişken adları büyük/küçük harfe duyarlıdır.

Bu, iki değişken oluşturacaktır:

```python
a = 4 
A = "Sally"
#A will not overwrite a
```


## Variable Names

Bir değişkenin kısa bir adı (x ve y gibi) veya daha açıklayıcı bir adı ( age, carname, total_volume) olabilir.

Python değişkenleri için kurallar:

* Bir değişken adı bir harf veya alt çizgi karakteri ile başlamalıdır
* Bir değişken adı bir sayı ile başlayamaz
* Bir değişken adı yalnızca alfa-sayısal karakterler ve alt çizgiler (A-z, 0-9 ve _ ) içerebilir
* Değişken adları büyük/küçük harfe duyarlıdır (age, Age ve AGE üç farklı değişkendir)
* Bir değişken adı Python keyword'lerden herhangi biri olamaz.

Legal variable names:

```python
myvar = "John"
my_var = "John"
_my_var = "John"
myVar = "John"
MYVAR = "John"
myvar2 = "John"
```

Illegal variable names:

```python
2myvar = "John"  
my-var = "John"  
my var = "John"
```

Değişken adlarının büyük/küçük harfe duyarlı olduğunu unutmayın

## Multi Words Variable Names

Birden fazla kelime içeren değişken adlarının okunması zor olabilir.

Bunları daha okunabilir hale getirmek için kullanabileceğiniz birkaç teknik vardır:

## Camel Case

İlki hariç her kelime büyük harfle başlar:

```python
myVariableName = "John"
```

## Pascal Case

Her kelime büyük harfle başlar:

```python
MyVariableName = "John"
```

## Snake Case

Her kelime bir alt çizgi karakteri ile ayrılır:

```python
my_variable_name = "John"
```

## Many Values to Multiple Variables

Python tek bir satırda birden fazla değişkene değer atamanıza izin verir:

```python
x, y, z = "Orange", "Apple", "Cherry"
print(x)
print(y)
print(z)
```

Not: Değişken sayısının değer sayısıyla eşleştiğinden emin olun, aksi takdirde hata alırsınız.

## One Value to Multiple Variables

Ve aynı değeri tek bir satırda birden fazla değişkene atayabilirsiniz:

```python
x = y = z = "Orange"
print(x)
print(y)
print(z)
```


## Unpack a Collection

Bir liste, tuple vb. içinde bir değerler koleksiyonunuz varsa. Python, değerleri değişkenlere çıkarmanıza izin verir. Buna `unpack` (paket açma) denir.

Unpack a list:

```python
fruits = ["Orange", "Apple", "Chery"]
x, y, z = fruits
print(x)
print(y)
print(z)
```

```output
apple  
banana  
cherry
```

Unpack Tuples Bölümümüzde paketten çıkarma hakkında daha fazla bilgi edineceğiz.

## Output Variables

Python `print()` fonksiyonu genellikle değişkenlerin çıktısını almak için kullanılır.

```python
x = "Python harika"
print(x)
```

`print()` fonksiyonunda, virgülle ayrılmış birden fazla değişkenin çıktısını alırsınız:

```python
x = "Python "
y = "Çok "
z = "Harika"
print(x, y, z)
```

```output
Python Çok Harika
```

Birden fazla değişkenin çıktısını almak için `+` operatörünü de kullanabilirsiniz:

```python
x = "Python "
y = "Çok "
z = "Harika"
print (x + y + z)
```

“`Python` ” ve “`Çok` ” ifadelerinden sonraki boşluk karakterine dikkat edin, bunlar olmasaydı sonuç “`Pythonisawesome`” olurdu.

Sayılar için `+` karakteri matematiksel bir operatör olarak çalışır:

```python
x = 5 
y = 10
print(x + y)
```

```Output
15
```

`print()` fonksiyonunda, bir string ve bir sayıyı `+` operatörü ile birleştirmeye çalıştığınızda, Python size bir hata verecektir:

```python
x = 5 
y = "John"
print(x + y)
```

`print()` fonksiyonunda birden fazla değişkenin çıktısını almanın en iyi yolu, bunları farklı veri türlerini de destekleyen virgüllerle ayırmaktır:

```python
x = 5 
y = "John"
print(x, y)
```

```output
5 John
```


## Global Variables

Bir fonksiyonun dışında oluşturulan değişkenler (önceki sayfalardaki tüm örneklerde olduğu gibi) global değişkenler olarak bilinir.

Global değişkenler hem fonksiyonların içinde hem de dışında herkes tarafından kullanılabilir.

Bir fonksiyonun dışında değişken oluşturma ve bunu fonksiyonun içinde kullanma: 

```python
x = "awasome"

def my_func():
    print("Python is " + x)

my_func()
```

```output
Python is awasome
```

Bir fonksiyon içinde aynı isimde bir değişken oluşturursanız, bu değişken local olur ve sadece fonksiyon içinde kullanılabilir. Aynı ada sahip global değişken olduğu gibi, global ve orijinal değeriyle kalacaktır.

Bir fonksiyonun içinde, global değişkenle aynı ada sahip bir değişken oluşturun

```python
x = "Python"

def my_func():
    x = "Java"
    print("En iyi programlama dili: " + x )

my_func()

print("En iyi programlama dili:" + x) 
```

```output
En iyi programlama dili Java
En iyi programlama dili Python
```

## The global Keyword

Normalde, bir fonksiyon içinde bir değişken oluşturduğunuzda, bu değişken lokaldir ve yalnızca o fonksiyon içinde kullanılabilir.

Bir fonksiyon içinde global bir değişken oluşturmak için `global` keyword'ünü kullanabilirsiniz.

`Global` keyword'ünü kullanırsanız, değişken global scope'a ait olur:

```python
def my_func():
    global x 
    x = python

my_func()

print("En iyi programlama dili: " + x)
```

```output
En iyi programlama dili python
```

Ayrıca, bir fonksiyon içinde `global` bir değişkeni değiştirmek istiyorsanız global keyword'ünü kullanın.

Bir fonksiyon içinde global bir değişkenin değerini değiştirmek için, global keyword'ünü kullanarak değişkene referans verin:


```python
x = "Java"

def my_func():
    global x 
    x = "Python"

my_func()

print("En iyi programlama dili: " + x)
```

```output
En iyi programlama dili Python
```


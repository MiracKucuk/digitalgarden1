---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/7-python-casting/","created":"2025-06-09T15:01:15.075+03:00","updated":"2025-06-09T15:14:20.677+03:00"}
---


## Değişken Type'ı Belirtme

Bir değişkene bir tür belirtmek istediğiniz zamanlar olabilir. Bu, casting ile yapılabilir. Python object-orientated bir dildir ve bu nedenle primitive type'ları da dahil olmak üzere data type'larını tanımlamak için class'ları kullanır.

Bu nedenle python'da casting işlemi constructor fonksiyonları kullanılarak yapılır:

* `int()` - bir integer literalden, bir float literalden (tüm ondalıkları kaldırarak) veya bir string literalden (stringin bir tam sayıyı temsil etmesi koşuluyla) bir tam sayı oluşturur

* `float()` - bir integer literal, bir float literal veya bir string literalden bir float sayı oluşturur (stringin bir float veya integer temsil etmesi şartıyla)

* `str()` - stringler, integer literaller ve float literaller dahil olmak üzere çok çeşitli veri tiplerinden bir string oluşturur

Integers:

```python
x = int(1)    # x 1 olacak
y = int(2.8)  # x 2 olacak
z = int("3")  # x 3 olacak
```

Floats:

```python
x = float(2)   # x 2.0 olacak
y = float(2.8) # x 2.8 olacak
z = float("3") # x 3.0 olacak
w = float("3.4")  # x 3.4 olacak
```


Strings:

```python
x = str("s1")  # x 's1' olacak
y = str(2)     # x '2' olacak
z = str(3.0)   # x '3.0' olacak
```
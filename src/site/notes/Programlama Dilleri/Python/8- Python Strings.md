---
{"dg-publish":true,"permalink":"/programlama-dilleri/python/8-python-strings/","created":"2025-06-09T15:14:12.706+03:00","updated":"2025-06-09T18:51:17.833+03:00"}
---

## Strings

Python'da stringler ya tek tırnak işareti ya da çift tırnak işareti ile çevrelenir.

'hello', “hello” ile aynıdır.

`print()` fonksiyonu ile bir string literali görüntüleyebilirsiniz:

```python
print("Hello")
print('Hello')
```

## Quotes Inside Quotes

String'i çevreleyen tırnak işaretleriyle eşleşmedikleri sürece, bir string içinde tırnak işaretleri kullanabilirsiniz:

```python
print("It's alright")  
print("He is called 'Johnny'")  
print('He is called "Johnny"')
```

## Bir Değişkene String Atama

Bir değişkene string atamak, değişken adının ardından bir eşittir işareti ve string ile yapılır:

```python
a = "Hello"
print(a)
```


## Çok Satırlı Stringler

Üç tırnak işareti kullanarak bir değişkene çok satırlı bir string atayabilirsiniz:

```python
a = """Lorem ipsum dolor sit amet,  
consectetur adipiscing elit,  
sed do eiusmod tempor incididunt  
ut labore et dolore magna aliqua."""  
print(a)
```

Ya da üç tek tırnak:

```python
a = '''Lorem ipsum dolor sit amet,  
consectetur adipiscing elit,  
sed do eiusmod tempor incididunt  
ut labore et dolore magna aliqua.'''  
print(a)
```

```output
Lorem ipsum dolor sit amet,  
consectetur adipiscing elit,  
sed do eiusmod tempor incididunt  
ut labore et dolore magna aliqua.
```

Not: sonuçta, satır sonları koddaki ile aynı konuma eklenir.


## Strings are Arrays

Diğer birçok popüler programlama dili gibi, Python'daki stringler de unicode karakterleri temsil eden bayt dizileridir.

Ancak Python'da karakter veri tipi yoktur, tek bir karakter sadece 1 uzunluğunda bir stringtir.

Karakter dizisinin elemanlarına erişmek için köşeli parantezler kullanılabilir.

Pozisyon 1'deki karakteri alın (ilk karakterin pozisyon 0 olduğunu unutmayın):

```python
a = "Hello World!"
print(a[1])
```

```output
e
```


## Bir String İçinde Döngü

Stringler array olduğu için, bir `for` döngüsü ile bir string içindeki karakterler arasında döngü yapabiliriz.

“banana" kelimesindeki harfler arasında dolaşın:

```python
for x in "banana"
    print(x)
```

```output
b  
a  
n  
a  
n  
a
```

Python For Loops bölümümüzde For Loops hakkında daha fazla bilgi edineceğiz.


## String Length

Bir stringin uzunluğunu elde etmek için `len()` fonksiyonunu kullanın.

```python
a = "Hello World!"
print(len(a))
```

```output
13
```


## Check String

Bir string içinde belirli bir ifadenin veya karakterin bulunup bulunmadığını kontrol etmek için “`in`” keyword'ünü kullanabiliriz.

Aşağıdaki metinde “free” olup olmadığını kontrol edin:

```python
txt = "The best things in life are free!"
print("free" in txt)
```

```output
True
```

Bunu bir `if` deyimi içinde kullanın:

```python
txt = "The best things in life are free!"
if "free" in txt
    print("Yes, 'free' is present.")
```

```output
Yes, 'free' is present.
```

Python If...Else bölümümüzde If deyimleri hakkında daha fazla bilgi edinineceğiz.

## Check if NOT

Bir string içinde belirli bir ifadenin veya karakterin bulunup bulunmadığını kontrol etmek için `not in` keyword'ünü kullanabiliriz.

```python
txt = "The best things in life are free!"
print("expensive" not in txt)
```

```output
True
```

Bunu bir if deyimi içinde kullanın:

```python
txt = "The best things in life are free!"
if "expensive" not in txt:
    print("No, 'expensive' is NOT present.")
```


## Slicing

Slice syntax kullanarak bir karakter aralığı döndürebilirsiniz.

String'in bir bölümünü döndürmek için başlangıç indeksini ve bitiş indeksini iki nokta üst üste ile ayırarak belirtin.

Konum 2'den konum 5'e kadar olan karakterleri alın (dahil değildir):

```python
b = "Hello World!"
print(b[2:5])
```

```output
llo
```


## Slice From the Start

Başlangıç indeksini dışarıda bıraktığınızda, aralık ilk karakterden başlayacaktır:

Başlangıçtan 5. konuma kadar olan karakterleri alın (dahil değildir):

```python
b= "Hello World!"
print(b[:5])
```

```Output
Hello
```


## Slice To the End

Bitiş indeksini dışarıda bıraktığınızda, aralık sonuna kadar gidecektir:

Karakterleri 2. konumdan sonuna kadar alın:

```python
b = "Hello World!"
print(b[2:])
```

```output
llo World!
```


## Negative Indexing

Slice'ı string'in sonundan başlatmak için negatif indeksler kullanın:

Karakterleri al:

Kimden: “ World!” içindeki “`o`” (konum -5)

Şuna, ama dahil değil: “ World!”deki “`d`” (pozisyon -2):

```python
a = "Hello World!"
print(a[-5:-2])
```

```output
orl
```


# Modify Strings

Python, stringler üzerinde kullanabileceğiniz bir dizi built-in metoda sahiptir.

## Upper Case

`upper()` metodu stringi büyük harf olarak döndürür:

```python
a = "Hello World!"
print(a.upper())
```

```
HELLO WORLD!
```


## Lower Case

`lower()` metodu stringi küçük harf olarak döndürür:

```python
a = "Hello World!"
print(a.lower())
```

```output
hello world!
```


## Remove Whitespace

Boşluk (whitespace), genellikle metnin önünde ve/veya arkasında bulunan boş alanlardır ve çoğu zaman bu boşlukları kaldırmak istersiniz.

`strip()` metodu, baştaki veya sondaki tüm boşlukları kaldırır:

```python
a = " Hello World! "
print(a.strip()) #returns "Hello, World!"
```

## Replace String

`replace()` metodu bir stringi başka bir string ile değiştirir:

```python
a = "Hello World!"
print(a.replace("H","J"))
```

```output
Jello World!
```


## Split String

`split()` metodu, belirtilen ayırıcı arasındaki metnin liste öğeleri haline geldiği bir liste döndürür.

`split()` metodu, ayırıcının örneklerini bulursa stringi substring'lere böler:

```python
a = "Hello, World!"
print(a.split(","))
```

```output
['Hello', ' World!']
```


# String Concatenation

İki string'i birleştirmek veya bir araya getirmek için `+` operatörünü kullanabilirsiniz.

a değişkenini b değişkeniyle c değişkeninde birleştirin:

```python
a = "Hello"
b = "World"
c = a + b 
print(c)
```

Aralarına boşluk eklemek için bir `“ ”` ekleyin:

```python
a = Hello 
b = World
c = a + " " + b 
print(c)
```

# Format - Strings

Python Değişkenleri bölümünde öğrendiğimiz gibi, stringleri ve sayıları bu şekilde birleştiremeyiz:

```python
a = 36
Text = "Hello world" + a
print(Text)
```

Ancak `f-strings` veya `format()` metodunu kullanarak string ve sayıları birleştirebiliriz!

## F-Strings

F-String Python 3.6'da tanıtıldı ve artık stringleri formatlamak için tercih edilen yoldur.

Bir stringi f-string olarak belirtmek için, string literalinin önüne bir `f` koymanız ve değişkenler ve diğer işlemler için placeholder olarak küme parantezleri `{}` eklemeniz yeterlidir.

Bir f stringi oluşturun:

```python
age = 36
txt = f"My name is John, I am {age}"
print(txt)
```

```output
My name is John, I am 36
```


## Placeholders and Modifiers

Bir placeholder, değeri formatlamak için değişkenler, operasyonlar, fonksiyonlar ve modifier'lar içerebilir.

`price` değişkeni için bir placeholder ekleyin:

```python
price = 50
txt = f"The price is {price} dollars"
print(txt)
```

Bir placeholder, değeri formatlamak için bir modifier içerebilir.

Modifier, iki nokta üst üste `:` ve ardından 2 ondalıklı fixed point sayı anlamına gelen `.2f` gibi legal bir formatlama türü eklenerek dahil edilir:

```python
price = 59
txt = f"The price is {price:.2f} dolars"
print(txt)
```

```output
The price is 59.00 dollars
```

Bir placeholder, matematik işlemleri gibi Python kodu içerebilir:

```python
txt  = f"The price is {59 * 20} dollars."
print(txt)
```

# Escape Characters

Bir stringe kurallara aykırı karakterler eklemek için bir escape karakteri kullanın.

Escape karakteri, bir ters eğik çizgi `\` ve ardından eklemek istediğiniz karakterdir.

Kural dışı karaktere örnek olarak, çift tırnakla çevrili bir string içindeki çift tırnak verilebilir:

Çift tırnakla çevrili bir stringin içinde çift tırnak kullanırsanız hata alırsınız:

```python
txt = "We are the so-called "Vikings" from the north."  # Hata Verir
```

Bu sorunu çözmek için `\”` escape karakterini kullanın:

```output
txt = "We are the so-called \"Vikings\" from the north."
```

## Escape Characters

Python'da kullanılan diğer escape karakterleri:

| Code | Result          |
| ---- | --------------- |
| \'   | Single Quote    |
| \\   | Backslash       |
| \n   | New Line        |
| \r   | Carriage Return |
| \t   | Tab             |
| \b   | Backspace       |
| \f   | Form Feed       |
| \ooo | Octal value     |
| \xhh | Hex value       |

```python
txt = "This will insert one \\ (backslash)."
print(txt) 
```

```output
Bu, bir \ (ters eğik çizgi) ekleyecektir.
```

```python
txt = "Hello\nWorld!"
print(txt) 
```

```output
Hello  
World!
```

```python
txt = "Hello\rWorld!"
print(txt) 
```

```output
World!
```

```python
txt = "Hello\tWorld!"
print(txt) 
```

```
Hello   World!
```

```python
#Bu örnek bir karakteri siler (backspace):
txt = "Hello \bWorld!"
print(txt) 
```

```output
HelloWorld!
```

```python
#Bir ters eğik çizgi (backslash) ve ardından gelen üç sayı, sekizlik (octal) bir değeri temsil eder:
txt = "\110\145\154\154\157"
print(txt) 
```

```output
Hello
```

```python
#Bir ters eğik çizgi (backslash), ardından 'x' ve bir onaltılık (hex) sayı gelirse, bu bir onaltılık değeri temsil eder:
txt = "\x48\x65\x6c\x6c\x6f"
print(txt) 
```

```output
Hello
```


# String Methods

Python, stringler üzerinde kullanabileceğiniz bir dizi built-in metoda sahiptir.

Not: Tüm string metotları yeni değerler döndürür. Orijinal stringi değiştirmezler.

|Yöntem|Açıklama|
|---|---|
|[capitalize()](https://www.w3schools.com/python/ref_string_capitalize.asp)|İlk karakteri büyük harfe dönüştürür|
|[casefold()](https://www.w3schools.com/python/ref_string_casefold.asp)|String'i küçük harfe dönüştürür|
|[center()](https://www.w3schools.com/python/ref_string_center.asp)|Ortalanmış bir string döndürür|
|[count()](https://www.w3schools.com/python/ref_string_count.asp)|Belirtilen bir değerin bir string içinde kaç kez geçtiğini döndürür|
|[encode()](https://www.w3schools.com/python/ref_string_encode.asp)|String'in encode edilmiş bir halini döndürür|
|[endswith()](https://www.w3schools.com/python/ref_string_endswith.asp)|String belirtilen değerle bitiyorsa True döndürür|
|[expandtabs()](https://www.w3schools.com/python/ref_string_expandtabs.asp)|String'in sekme (tab) karakterinin genişliğini ayarlar|
|[find()](https://www.w3schools.com/python/ref_string_find.asp)|Belirtilen bir değer için string’i arar ve bulunduğu konumu döndürür|
|[format()](https://www.w3schools.com/python/ref_string_format.asp)|Belirtilen değerleri bir string içinde biçimlendirir|
|`format_map()`|Belirtilen değerleri bir string içinde biçimlendirir|
|[index()](https://www.w3schools.com/python/ref_string_index.asp)|Belirtilen bir değerin konumunu döndürür (bulunamazsa hata verir)|
|[isalnum()](https://www.w3schools.com/python/ref_string_isalnum.asp)|Tüm karakterler alfanümerikse True döndürür|
|[isalpha()](https://www.w3schools.com/python/ref_string_isalpha.asp)|Tüm karakterler harfse True döndürür|
|[isascii()](https://www.w3schools.com/python/ref_string_isascii.asp)|Tüm karakterler ASCII karakterleriyse True döndürür|
|[isdecimal()](https://www.w3schools.com/python/ref_string_isdecimal.asp)|Tüm karakterler ondalık sayıysa True döndürür|
|[isdigit()](https://www.w3schools.com/python/ref_string_isdigit.asp)|Tüm karakterler rakamsa True döndürür|
|[isidentifier()](https://www.w3schools.com/python/ref_string_isidentifier.asp)|Geçerli bir tanımlayıcı (identifier) ise True döndürür|
|[islower()](https://www.w3schools.com/python/ref_string_islower.asp)|Tüm karakterler küçük harfse True döndürür|
|[isnumeric()](https://www.w3schools.com/python/ref_string_isnumeric.asp)|Tüm karakterler sayısalsa True döndürür|
|[isprintable()](https://www.w3schools.com/python/ref_string_isprintable.asp)|Tüm karakterler yazdırılabilirse True döndürür|
|[isspace()](https://www.w3schools.com/python/ref_string_isspace.asp)|Tüm karakterler boşluk karakteriyse True döndürür|
|[istitle()](https://www.w3schools.com/python/ref_string_istitle.asp)|Her kelimenin ilk harfi büyükse True döndürür|
|[isupper()](https://www.w3schools.com/python/ref_string_isupper.asp)|Tüm karakterler büyük harfse True döndürür|
|[join()](https://www.w3schools.com/python/ref_string_join.asp)|Bir iterable’daki elemanları birleştirip string oluşturur|
|[ljust()](https://www.w3schools.com/python/ref_string_ljust.asp)|String’in sola hizalanmış bir versiyonunu döndürür|
|[lower()](https://www.w3schools.com/python/ref_string_lower.asp)|String’i küçük harfe dönüştürür|
|[lstrip()](https://www.w3schools.com/python/ref_string_lstrip.asp)|String’in baştaki boşluklarını temizler|
|[maketrans()](https://www.w3schools.com/python/ref_string_maketrans.asp)|Çeviri işlemlerinde kullanılacak bir çeviri tablosu döndürür|
|[partition()](https://www.w3schools.com/python/ref_string_partition.asp)|String’i üç parçaya bölen bir tuple döndürür|
|[replace()](https://www.w3schools.com/python/ref_string_replace.asp)|Belirtilen bir değeri başka bir değerle değiştirip yeni string döndürür|
|[rfind()](https://www.w3schools.com/python/ref_string_rfind.asp)|Belirtilen değerin string içindeki son konumunu döndürür|
|[rindex()](https://www.w3schools.com/python/ref_string_rindex.asp)|Belirtilen değerin string içindeki son konumunu döndürür (bulunamazsa hata verir)|
|[rjust()](https://www.w3schools.com/python/ref_string_rjust.asp)|String’in sağa hizalanmış bir versiyonunu döndürür|
|[rpartition()](https://www.w3schools.com/python/ref_string_rpartition.asp)|String’i sağdan başlayarak üç parçaya bölen bir tuple döndürür|
|[rsplit()](https://www.w3schools.com/python/ref_string_rsplit.asp)|String’i sağdan bölüp liste olarak döndürür|
|[rstrip()](https://www.w3schools.com/python/ref_string_rstrip.asp)|String’in sonundaki boşlukları temizler|
|[split()](https://www.w3schools.com/python/ref_string_split.asp)|String’i belirtilen ayırıcıya göre böler ve liste döndürür|
|[splitlines()](https://www.w3schools.com/python/ref_string_splitlines.asp)|Satır sonlarına göre bölerek liste döndürür|
|[startswith()](https://www.w3schools.com/python/ref_string_startswith.asp)|String belirtilen değerle başlıyorsa True döndürür|
|[strip()](https://www.w3schools.com/python/ref_string_strip.asp)|Baş ve sondaki boşlukları temizleyen versiyonu döndürür|
|[swapcase()](https://www.w3schools.com/python/ref_string_swapcase.asp)|Küçük harfleri büyük, büyük harfleri küçük yaparak yeni bir string döndürür|
|[title()](https://www.w3schools.com/python/ref_string_title.asp)|Her kelimenin ilk harfini büyük yapar|
|[translate()](https://www.w3schools.com/python/ref_string_translate.asp)|Çeviri tablosuna göre dönüştürülmüş yeni bir string döndürür|
|[upper()](https://www.w3schools.com/python/ref_string_upper.asp)|Tüm karakterleri büyük harfe dönüştürür|
|[zfill()](https://www.w3schools.com/python/ref_string_zfill.asp)|String’in başına belirtilen sayıda 0 ekler|


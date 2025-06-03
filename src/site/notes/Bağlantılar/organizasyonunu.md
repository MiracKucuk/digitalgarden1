---
{"dg-publish":true,"permalink":"/baglantilar/organizasyonunu/"}
---

### Genel Yapı

1. **`.vscode/` Klasörü**:
    
    - **İçinde ne var?** Bu klasör, Visual Studio Code'un proje bazlı ayarlarını içerir.
    - **`launch.json` Dosyası:** Bu dosya, VS Code'da hata ayıklama (debugging) işlemleri için gerekli ayarları içerir. Örneğin, uygulamayı nasıl çalıştıracağınızı tanımlayabilirsiniz.
2. **`src/` Klasörü**:
    
    - Bu, genelde "source" (kaynak) kodlarının bulunduğu ana klasördür. Projeye dair işlevsel tüm kodlar burada yer alır.

### `src` Klasörünün Alt Yapısı

1. **`controllers/` Klasörü**:
    
    - **Ne işe yarar?** "Controller" dosyaları, uygulamanın iş mantığını (business logic) içerir. Bir API çağrısı yapıldığında veya bir rota tetiklendiğinde, bu işlemleri yöneten kodlar buradadır.
    - **Dosyalar:**
        - `auth-controllers.js`: Kullanıcı kimlik doğrulama (authentication) işlemlerini yöneten kodlar burada bulunur.
        - `service-controllers.js`: Diğer hizmetlere ait iş mantığını yöneten kodlar (örneğin, bir ürün listeleme veya bir hesap güncelleme) buradadır.
2. **`routes/` Klasörü**:
    
    - **Ne işe yarar?** "Routes" dosyaları, kullanıcıdan gelen HTTP isteklerini doğru "controller" dosyalarına yönlendirmek için kullanılır. Örneğin, bir kullanıcı `/login` URL'sine istek gönderdiğinde, hangi controller'ın bu isteği işleyeceğini belirler.
    - **Dosyalar:**
        - `auth-routes.js`: Kimlik doğrulama ile ilgili tüm yolları (routes) içerir. Örneğin:
            - POST `/login`
            - POST `/register`
        - `service-routes.js`: Diğer hizmetlerle ilgili yolları içerir. Örneğin:
            - GET `/products`
            - POST `/update-profile`
3. **`app.js`**:
    
    - **Ne işe yarar?** Bu dosya, genelde uygulamanın giriş noktasıdır. Sunucu burada başlatılır ve tüm temel ayarlar yapılır. Örneğin:
        - Middleware (ara yazılım) tanımlamaları,
        - Routes (yollar) bağlamaları,
        - Port numarası ayarları gibi işlemler bu dosyada yapılır.
4. **`package.json`**:
    
    - **Ne işe yarar?** Bu dosya, Node.js projesinin temel bilgilerini ve bağımlılıklarını (dependencies) tanımlar. Örneğin:
        - Projenin adı, sürümü,
        - Kullanılan kütüphaneler (ör. Express, Mongoose),
        - Çalıştırma komutları (ör. `npm start`) burada tanımlanır.

### Özetle:

- **`controllers/`:** İş mantığını içeren dosyalar.
- **`routes/`:** HTTP isteklerini doğru controller'a yönlendiren dosyalar.
- **`app.js`:** Uygulamanın başlangıç dosyası.
- **`package.json`:** Proje bilgilerini ve bağımlılıklarını tanımlayan dosya.

Eğer bir web uygulaması oluşturuluyorsa, bu yapı modülerlik sağlar ve her parçayı ayrı ayrı düzenli bir şekilde geliştirme imkânı sunar. Bu, projeyi büyüdükçe yönetilebilir kılar.
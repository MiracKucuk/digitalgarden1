---
{"dg-publish":true,"permalink":"/baglantilar/com-component-object-model/"}
---



**COM (Component Object Model)**, Microsoft tarafından geliştirilmiş bir yazılım bileşenleri mimarisidir. Bu mimari, farklı yazılım bileşenlerinin birbiriyle etkileşime geçmesini sağlayarak, yazılım geliştirmede yeniden kullanılabilirliği ve modülerliği artırır. İşte COM'un ana özellikleri ve işlevleri:

### Temel Özellikleri:

1. **Yeniden Kullanılabilirlik**: COM bileşenleri, farklı uygulamalarda tekrar kullanılabilir. Bu sayede, yazılımlar daha hızlı ve verimli bir şekilde geliştirilir.
    
2. **Dilin Bağımsızlığı**: COM, farklı programlama dilleri (C++, C#, Visual Basic, vb.) ile uyumlu çalışabilir. Bu, geliştiricilerin farklı dillerde yazılmış bileşenleri kullanabilmesini sağlar.
    
3. **Dağıtılmış Uygulamalar**: COM, bileşenlerin farklı makinelerde çalışmasını destekler. Bu, dağıtık uygulamaların geliştirilmesine olanak tanır.
    
4. **Yüksek Performans**: COM, doğrudan bellek erişimi kullanarak performansı artırır ve bileşenler arasında hızlı iletişim sağlar.
    

### İşleyiş Prensibi:

- **Bileşenler ve Nesneler**: COM, yazılım bileşenlerini nesne tabanlı bir şekilde tanımlar. Her bir bileşen, belirli bir işlevi yerine getiren bir nesne olarak düşünülebilir.
    
- **Interface (Arayüz)**: COM bileşenleri, belirli işlevlere erişim sağlamak için arayüzler tanımlar. Bu arayüzler, bir bileşenin sunduğu işlevlerin diğer bileşenler tarafından nasıl kullanılacağını belirler.
    
- **Kayıt ve Kullanım**: COM bileşenleri, sistemin kayıt defterine kaydedilir. Bu kayıt, bileşenin sistemde nasıl kullanılacağını tanımlar. Geliştiriciler, COM bileşenlerine erişmek için belirli bir arayüzü kullanarak nesne örnekleri oluşturabilir.
    

### Kullanım Alanları:

- **Windows Uygulamaları**: COM, Windows uygulamalarında sıkça kullanılır ve Microsoft Office gibi uygulamalar bu mimariyi destekler.
- **ActiveX**: COM bileşenleri, web tarayıcılarında çalışan ActiveX denetimleri gibi bileşenler oluşturmak için de kullanılır.
- **OLE (Object Linking and Embedding)**: OLE, COM tabanlı bir başka teknolojidir ve farklı uygulamalar arasında nesne paylaşımını sağlar.

COM, yazılım geliştirme sürecini kolaylaştıran ve bileşen tabanlı mimarilerin temelini oluşturan önemli bir teknolojidir. Ancak, güvenlik açısından dikkatli kullanılmalıdır; çünkü kötü niyetli aktörler COM bileşenlerini kullanarak sistem üzerinde zararlı işlemler gerçekleştirebilir.
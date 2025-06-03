---
{"dg-publish":true,"permalink":"/baglantilar/sanal-ortamada-kurulabilir-ayrinti-icin-tikla/"}
---

Python ortamı kurarken, öncelikle bir sanal ortam (virtual environment) oluşturmanızı öneririm. Bu, bağımlılıkların ve paketlerin yönetimini kolaylaştırır ve proje içinde her bir uygulamanın bağımsız çalışmasını sağlar. Aşağıda, Python'da bir sanal ortam oluşturmak ve bir proje için gerekli bağımlılıkları yüklemek için adım adım bir rehber sundum:

### 1. **Python'da Sanal Ortam Oluşturma**

Python'da bir sanal ortam oluşturmak için `venv` modülünü kullanabilirsiniz. Aşağıdaki adımları takip edebilirsiniz:

```
# Yeni bir sanal ortam oluşturmak için aşağıdaki komutu kullanın:

python -m venv myenv

# Oluşturduğunuz sanal ortama geçiş yapmak için:

cd myenv

# Ortamı etkinleştirin. Windows'da bu komutu kullanın:

.\Scripts\activate

# Ortamı etkinleştirdikten sonra, etkin ortamın adı parenthesis içinde görünür:

(myenv) C:\path\to\your\project>
```

Not: Eğer sisteminizde `venv` modülü yüklü değilse, Python 3.3 ve üzeri bir sürüm kullanıyorsanız, varsayılan olarak geliyor. Eğer daha eski bir Python sürümünüz varsa veya `venv` modülünü kullanamazsanız, bir sanal ortam oluşturmak için `virtualenv` kullanabilirsiniz:

```
# virtualenv kurmak için:

pip install virtualenv

# virtualenv kullanarak bir sanal ortam oluşturun:

virtualenv myenv

# Oluşturulan sanal ortama geçiş yapmak ve etkinleştirmek için:

cd myenv

.\Scripts\activate
```

### 2. **Bağımlılıkları Yükleme**

Şimdi, projeniz için gerekli olan Python paketlerini yükleyebilirsiniz. Genellikle, `requirements.txt` dosyası kullanılarak ihtiyaç duyulan paketler belirtilebilir. Eğer böyle bir dosya yoksa veya belirli bir Python paketini yüklemek isterseniz, `pip` komutunu kullanabilirsiniz:

```
# Bir requirements.txt dosyasını kullanarak bağımlılıkları yüklemek için:

pip install -r requirements.txt

# Özel bir Python paketini yüklemek için:

pip install paket_ismi
```


### 3. **Çalışma Ortamınızı Yönetme**

Sanal ortamı kapatmak istediğinizde, komut satırını kapatabilir veya ortamı kapatmak için şu komutu kullanabilirsiniz:

```
# Ortamı kapatmak için: deactivate
```

Bu adımlar, Python projeniz için etkili bir şekilde yapılandırılmış bir çalışma ortamına sahip olmanıza olanak tanır ve sanal ortam, bağımlılıkların bağımsız olarak yönetilmesini sağlar.

Bu, Python'da otomatik keşif (reconnaissance) yaparken veya genel olarak geliştirici işlemlerinde iyi bir uygulama olacaktır.


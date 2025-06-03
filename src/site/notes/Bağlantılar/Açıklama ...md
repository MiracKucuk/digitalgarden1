---
{"dg-publish":true,"permalink":"/baglantilar/aciklama/"}
---

`find . -exec /bin/bash -p \; -quit` komutunu parçalara ayırarak açıklayalım:

### Komutun Anlamı:

1. **`find .`**:
    
    - Şu anki dizinde (`.`) ve onun altındaki tüm dosya/dizinlerde arama yapar.
2. **`-exec`**:
    
    - `find` komutu ile bulunan her bir dosya/dizin üzerinde belirli bir komutu çalıştırmak için kullanılır.
    - Burada, her bir dosya/dizin için `/bin/bash -p` komutu çalıştırılır.
3. **`/bin/bash -p`**:
    
    - **`/bin/bash`**: Bash kabuğunu çalıştırır.
    - **`-p` (privileged mode)**: Yetkili modda bash çalıştırır.
        - Bu modda bash, SUID/SGID (örneğin root yetkileri) varsa bunu devralır ve ortam değişkenlerini güvenlik için sıfırlamaz.
        - Bu, normalde güvenlik önlemi olarak devre dışı bırakılan özellikleri aktif hale getirir.
4. **`\;`**:
    
    - `-exec` parametresinde, belirtilen komutun sonlandığını belirtir.
    - `\;` ifadesindeki `\` karakteri, `;` işaretinin shell tarafından özel bir karakter olarak yorumlanmasını engeller.
5. **`-quit`**:
    
    - `find` komutuna, ilk eşleşen dosya/dizini bulduktan sonra çalışmayı durdurmasını söyler.
    - Böylece tüm dizini taramak yerine, yalnızca ilk uygun dosyada komut çalıştırılır ve işlem hızlanır.

---

### Özetle Ne Yapar?

1. Şu anki dizinde (`.`) ve alt dizinlerde dosya arar.
2. İlk bulunan dosya/dizin üzerinde `/bin/bash -p` komutunu çalıştırır (yetkili bash kabuğu açar).
3. İlk eşleşme bulunduğunda işlemi durdurur.

---

### **Neden Kullanılır?**

- SUID/SGID yetkilerini devralıp bash kabuğunu çalıştırmak için kullanılır.
- Genellikle güvenlik testlerinde ve privilege escalation (yetki yükseltme) senaryolarında, bir dosyanın yetkili modda çalıştırılabilirliğini test etmek için uygulanır.


---
{"dg-publish":true,"permalink":"/baglantilar/smbserver-py/"}
---



`smberver.py`, Impacket adlı araç setinin bir parçası olan bir Python betiğidir ve SMB (Server Message Block) protokolünü kullanarak bir klasörü ağ üzerinden paylaşıma açmanıza olanak tanır. Bu araç sayesinde kendi bilgisayarınızı veya sanal makinenizi (örneğin Pwnbox gibi bir ortamda) geçici bir SMB sunucusuna dönüştürebilir ve başka sistemlerin bu sunucuya bağlanarak dosya indirmesini veya yüklemesini sağlayabilirsiniz.

### Kullanım Alanı

Özellikle sızma testlerinde ve güvenlik araştırmalarında faydalıdır. `smbserver.py`, hedef makineye dosya aktarımı yaparken veya hedeften dosya çekerken yaygın olarak kullanılır. Bağlantı, Windows ve Linux sistemlerinde `copy`, `move`, `PowerShell Copy-Item` gibi komutlar veya diğer SMB uyumlu araçlarla yapılabilir.

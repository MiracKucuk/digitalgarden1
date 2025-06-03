---
{"dg-publish":true,"permalink":"/baglantilar/impacket-smbserver/"}
---



**Impacket smbserver**, Python ile yazılmış ve Windows'un SMB (Server Message Block) protokolünü kullanarak bir SMB paylaşımı oluşturmaya yarayan bir araçtır. SMB protokolü, ağ üzerinden dosya ve kaynak paylaşımı sağlamak için kullanılır ve bu araç, Linux gibi diğer işletim sistemlerinden Windows makinelerine dosya paylaşımı yapılmasını kolaylaştırır.

**smbserver** komutuyla, bir klasörü ağ paylaşımı olarak sunabilirsiniz, böylece hedef makine bu paylaşımı kullanarak dosyaları indirme veya yükleme işlemi yapabilir. Özellikle sızma testlerinde, güvenlik araştırmalarında ve ayrıcalık yükseltme çalışmalarında, hedef makineye dosya veya zararlı bir binary aktarmak için sıkça kullanılan bir yöntemdir. SMB protokolü üzerinden dosya transferine izin verilen ortamlarda **Impacket smbserver** etkili bir dosya aktarım aracı olarak öne çıkar.
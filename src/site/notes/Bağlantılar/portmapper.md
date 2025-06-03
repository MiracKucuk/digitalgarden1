---
{"dg-publish":true,"permalink":"/baglantilar/portmapper/"}
---

**Portmapper**, **Remote Procedure Call (RPC)** protokollerinde kullanılan bir hizmettir. RPC ile çalışan uygulamaların ağ üzerindeki hizmetlere bağlanabilmesi için hangi portun kullanıldığını öğrenmelerine olanak sağlar.

### Portmapper'ın İşlevi

- RPC kullanan bir istemci, önce **portmapper** hizmetine bağlanır.
- Portmapper, hangi hizmetin hangi TCP veya UDP portunda çalıştığını belirler ve istemciye bu bilgiyi iletir.
- Bu sayede istemci, doğrudan hedef hizmetle iletişime geçebilir.

### Portmapper ve Güvenlik

Portmapper kullanımı **NFSv2** ve **NFSv3** gibi eski protokol sürümlerinde zorunludur. Ancak, bu durum bazı güvenlik riskleri oluşturur:

1. **Açık Portlar:** Portmapper dinamik portlar kullandığı için, sistemde birçok açık port bırakır ve bu, saldırganlar için bir hedef oluşturabilir.
2. **Firewall Uyumluluğu:** Dinamik portlar, **firewall** yapılandırmasını zorlaştırır ve güvenliği azaltabilir.

### NFSv4 ile Değişiklikler

**NFSv4**, portmapper gereksinimini ortadan kaldırır. Bunun yerine, varsayılan olarak **2049** numaralı sabit portu kullanır.

- Bu, güvenlik duvarlarıyla uyumluluğu artırır.
- Ağ üzerinde NFS hizmetlerinin daha güvenli ve kolay bir şekilde çalışmasını sağlar.

Kısaca, **portmapper**, eski protokollerin ağ iletişimini sağlamak için kullandığı bir mekanizmadır ve modern protokollerle bu ihtiyaç ortadan kaldırılmıştır.
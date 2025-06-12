---
{"dg-publish":true,"permalink":"/programlama-dilleri/bash-script/htb-bash-script/","created":"2025-06-13T02:00:27.858+03:00","updated":"2025-06-13T02:36:03.302+03:00"}
---


# Bourne Again Shell

[Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)), Unix tabanlı işletim sistemi ile iletişim kurmak ve sisteme komutlar vermek için kullandığımız script dilidir. Mayıs 2019'dan bu yana Windows, Linux için Bash'i Windows ortamında kullanmamızı sağlayan bir [Windows Subsystem](https://docs.microsoft.com/en-us/windows/wsl/about) sağlamaktadır. Bu dille verimli bir şekilde çalışmak için dile hakim olmak çok önemlidir. Scripting ve programlama dilleri arasındaki temel fark, programlama dillerinin aksine scripting dilini çalıştırmak için kodu derlememize gerek olmamasıdır.

Penetrasyon test uzmanları olarak, ister Windows ister Unix tabanlı olsun, her türlü işletim sistemiyle çalışabilmeliyiz. Verimlilik, özellikle privilege escalation alanında olmak üzere, temel olarak sistemler hakkındaki bilgiye bağlıdır. Unix tabanlı sistemlerde terminalin nasıl kullanılacağını, verilerin nasıl filtreleneceğini ve bu işlemlerin nasıl otomatikleştirileceğini öğrenmek çok önemlidir. Özellikle Unix tabanlı büyük kurumsal ağlarda, büyük miktarda veri ile uğraşmak zorunda kalacağız. Potansiyel boşlukları ve bilgileri mümkün olduğunca hızlı bir şekilde belirlemek için buna göre sıralamak ve filtrelemek zorundayız.

Ayrıca birkaç komutun nasıl birleştirileceğini ve tek tek sonuçlarla nasıl çalışılacağını öğrenmek de önemlidir. İşte bu noktada script devreye girerek hızımızı ve verimliliğimizi artırır. Bir programlama dili gibi, bir script dili de hemen hemen aynı yapıya sahiptir ve bu da ikiye ayrılabilir:

- `Input` & `Output`
- `Arguments`, `Variables` & `Arrays`
- `Conditional execution`
- `Arithmetic`
- `Loops`
- `Comparison operators`
- `Functions`

Bazı süreçleri her zaman tekrarlamamak veya büyük miktarda bilgiyi işlemek ve filtrelemek için otomatikleştirmek genellikle yaygındır. Genel olarak, bir script bir süreç oluşturmaz, ancak script'i çalıştıran yorumlayıcı, bu durumda `Bash` tarafından çalıştırılır. Bir script'i çalıştırmak için, yorumlayıcıyı belirtmemiz ve hangi script'i işlemesi gerektiğini söylememiz gerekir. Böyle bir çağrı şuna benzer:

#### Script Execution - Examples

```shell-session
C4RT3L@htb[/htb]$ bash script.sh <optional arguments>
```

```shell-session
C4RT3L@htb[/htb]$ sh script.sh <optional arguments>
```

```shell-session
C4RT3L@htb[/htb]$ ./script.sh <optional arguments>
```

Böyle bir scripte bakalım ve belirli sonuçlar elde etmek için nasıl oluşturulabileceklerini görelim. Bu scripti çalıştırır ve bir domain belirtirsek, bu scriptin hangi bilgileri sağladığını görürüz.

#### CIDR.sh

```shell-session
C4RT3L@htb[/htb]$ ./CIDR.sh inlanefreight.com

Discovered IP address(es):
165.22.119.202

Additional options available:
	1) Identify the corresponding network range of target domain.
	2) Ping discovered hosts.
	3) All checks.
	*) Exit.

Select your option: 3

NetRange for 165.22.119.202:
NetRange:       165.22.0.0 - 165.22.255.255
CIDR:           165.22.0.0/16

Pinging host(s):
165.22.119.202 is up.

1 out of 1 hosts are up.
```

Şimdi bu script'e detaylı bir şekilde bakalım ve mümkün olan en iyi şekilde satır satır okuyalım. İlerleyen bölümlerde bu scriptin tüm bölümlerine bakacak ve analiz edeceğiz.

#### CIDR.sh

```bash
#!/bin/bash

# Check for given arguments
if [ $# -eq 0 ]
then
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
else
	domain=$1
fi

# Identify Network range for the specified IP address(es)
function network_range {
	for ip in $ipaddr
	do
		netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
		cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
		cidr_ips=$(prips $cidr)
		echo -e "\nNetRange for $ip:"
		echo -e "$netrange"
	done
}

# Ping discovered IP address(es)
function ping_host {
	hosts_up=0
	hosts_total=0
	
	echo -e "\nPinging host(s):"
	for host in $cidr_ips
	do
		stat=1
		while [ $stat -eq 1 ]
		do
			ping -c 2 $host > /dev/null 2>&1
			if [ $? -eq 0 ]
			then
				echo "$host is up."
				((stat--))
				((hosts_up++))
				((hosts_total++))
			else
				echo "$host is down."
				((stat--))
				((hosts_total++))
			fi
		done
	done
	
	echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

echo -e "Discovered IP address:\n$hosts\n"
ipaddr=$(host $domain | grep "has address" | cut -d" " -f4 | tr "\n" " ")

# Available options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
	"1") network_range ;;
	"2") ping_host ;;
	"3") network_range && ping_host ;;
	"*") exit 0 ;;
esac
```

Gördüğümüz gibi, burada script'i bölebileceğimiz birkaç parçayı yorumladık.

1. Verilen bağımsız değişkenleri kontrol edin
2. Belirtilen IP adres(ler)i için ağ aralığını belirleyin
3. Bulunan IP adres(ler)ine ping atın
4. Belirtilen domainin IP adres(ler)ini belirleyin
5. Kullanılabilir seçenekler


#### 1. Verilen bağımsız değişkenleri kontrol edin

Script'in ilk bölümünde, hedef şirketi temsil eden bir domain belirtip belirtmediğimizi kontrol eden bir if-else deyimimiz var.

#### 2. Belirtilen IP adres(ler)i için ağ aralığını belirleyin

Her IP adresi için bir “whois” sorgusu yapan ve ayrılmış ağ aralığı için satırı görüntüleyen ve bunu `CIDR.txt` dosyasında saklayan bir fonksiyon oluşturduk.

#### 3. Bulunan IP adres(ler)ine ping atın

Bu ek fonksiyon, bulunan hostların ilgili IP adresleri ile erişilebilir olup olmadığını kontrol etmek için kullanılır. For-Loop ile ağ aralığındaki her IP adresine `ping` atıyoruz ve sonuçları sayıyoruz.

#### 4. Belirtilen domainin IP adres(ler)ini belirleyin

Bu scriptin ilk adımı olarak, bize döndürülen domainin IPv4 adresini tespit ediyoruz.

#### 5. Kullanılabilir seçenekler

Ardından, altyapı hakkında daha fazla bilgi edinmek için hangi fonksiyonları kullanmak istediğimize karar veririz.


# Conditional Execution (Koşullu Yürütme)

Koşullu yürütme, farklı koşullara ulaşarak kodumuzun akışını kontrol etmemizi sağlar. Bu fonksiyon temel bileşenlerden biridir. Aksi takdirde, yalnızca bir komutu diğerinin ardından yürütebiliriz.

Çeşitli koşulları tanımlarken, belirli bir değer için hangi fonksiyonların veya kod bölümlerinin çalıştırılması gerektiğini belirtiriz. Belirli bir koşula ulaşırsak, yalnızca o koşul için kod çalıştırılır ve diğerleri atlanır. Kod bölümü tamamlanır tamamlanmaz, aşağıdaki komutlar koşullu yürütme dışında yürütülecektir. Kodun ilk bölümüne tekrar bakalım ve analiz edelim.

#### Script.sh

```bash
#!/bin/bash

# Verilen argümanı kontrol edin
if [ $# -eq 0 ]
then
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
else
	domain=$1
fi

<SNIP>
```

Özet olarak, bu kod bölümü aşağıdaki bileşenlerle çalışır:

* `#!/bin/bash` - Shebang (Başlık).
* `if-else-fi` - Koşullu yürütme.
* `echo` - Belirli çıktıları yazdırır.
* `$# / $0 / $1` - Özel değişkenler.
* `domain` - Variables.

Koşullu yürütmelerin koşulları, sonraki örneklerde göreceğimiz gibi değişkenler (`$#`, `$0`, `$1`, `domain`), değerler (`0`) ve stringler kullanılarak tanımlanabilir. Bu değerler bir sonraki bölümde inceleyeceğimiz `karşılaştırma operatörleri` (`-eq`) ile karşılaştırılır.


## Shebang

Shebang satırı her scriptin başında yer alır ve her zaman “`#!`” ile başlar. Bu satır, scriptin çalıştırılacağı belirtilen yorumlayıcının (`/bin/bash`) yolunu içerir. Shebang'i Python, Perl ve diğerleri gibi diğer yorumlayıcıları tanımlamak için de kullanabiliriz.

```python
#!/usr/bin/env python
```

```perl
#!/usr/bin/env perl
```

## If-Else-Fi

En temel programlama görevlerinden biri, bunlarla başa çıkmak için farklı koşulları kontrol etmektir. Koşulların kontrol edilmesinin programlama ve script dillerinde genellikle iki farklı şekli vardır: `if-else koşulu` ve `case deyimleri`. Sözde kodda if koşulu şu anlama gelir:

#### Pseudo-Code

```bash
if [ verilen argümanların sayısı eşittir 0 ]
then
	Print: "Hedef domain'i belirtmeniz gerekir."
	Print: "<empty line>"
	Print: "Usage:"
	Print: "   <script'in adı> <domain>"
	Exit the script with an error
else
	"domain" değişkeni, verilen argüman için takma ad görevi görür 
if koşulunu tamamlayın
```

Varsayılan olarak, bir `If-Else` koşulu, bir sonraki örnekte gösterildiği gibi yalnızca tek bir “`If`” içerebilir.

#### If-Only.sh

```bash
#!/bin/bash

value=$1

if [ $value -gt "10" ]
then
        echo "Given argument is greater than 10."
fi
```

#### If-Only.sh - Execution

```shell-session
C4RT3L@htb[/htb]$ bash if-only.sh 5
```

```shell-session
C4RT3L@htb[/htb]$ bash if-only.sh 12

Given argument is greater than 10.
```

`Elif` veya `Else` eklerken, belirli değerleri veya durumları ele almak için alternatifler ekleriz. Belirli bir değer ilk durum için geçerli değilse, diğerleri tarafından algılanacaktır.

#### If-Elif-Else.sh

```bash
#!/bin/bash

value=$1

if [ $value -gt "10" ]
then
	echo "Given argument is greater than 10."
elif [ $value -lt "10" ]
then
	echo "Given argument is less than 10."
else
	echo "Given argument is not a number."
fi
```

#### If-Elif-Else.sh - Execution

```shell-session
C4RT3L@htb[/htb]$ bash if-elif-else.sh 5

Given argument is less than 10.
```

```shell-session
C4RT3L@htb[/htb]$ bash if-elif-else.sh 12

Given argument is greater than 10.
```

```shell-session
C4RT3L@htb[/htb]$ bash if-elif-else.sh HTB

if-elif-else.sh: line 5: [: HTB: integer expression expected
if-elif-else.sh: line 8: [: HTB: integer expression expected
Given argument is not a number.
```

Kodumuzu genişletebilir ve birkaç koşul belirleyebiliriz. Bu şuna benzer bir şey olabilir:

#### Çeşitli Koşullar - Script.sh

```bash
#!/bin/bash

# Check for given argument
if [ $# -eq 0 ]
then
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
elif [ $# -eq 1 ]
then
	domain=$1
else
	echo -e "Too many arguments given."
	exit 1
fi

<SNIP>
```

Burada, bize birden fazla argüman verdiğimizi söyleyen bir satır yazdıran (`echo -e “...”`) ve programdan bir hata ile çıkan (`exit 1`) başka bir koşul tanımlıyoruz (`elif [<condition>];then`).

## Exercise Script

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -c

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
        var=$(echo $var | base64)
done
```

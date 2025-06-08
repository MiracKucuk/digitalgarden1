---
{"dg-publish":true,"permalink":"/cheat-sheet/curl-cheat-sheet/","created":"2025-02-07T01:07:16.869+03:00","updated":"2025-05-27T23:53:46.744+03:00"}
---


```
curl domain.com
```


### Dosya İndirme 

```
curl -O domain.com/index.html
```



### Sessiz Modda Dosya indirme 

```
curl -s -O domain.com/index.html
```



### SSL Doğrulamasını Atlayarak İstek Gönderme

```shell-session
curl -k https://domain.com
```


### HTTP Request ve Response Görüntüleme    ( Daha fazla için -vvv)

```shell-session
curl https://domain.com -v
```


### Sadece HTTP Headerlarını Getirme

```shell-session
curl -I https://domain.com 
```


### HTTP Headerlarını ve İçeriği Getirme 

```shell-session
curl -i https://domain.com 
```


### User-Agent Belirtme 

```
curl https://www.domain.com -A 'Mozilla/5.0'
```



### Basic Authentication ile Request Gönderme

```shell-session
curl -u username:password http://<SERVER_IP>:<PORT>/
```

```
curl http://admin:admin@<SERVER_IP>:<PORT>/
```



### Custom HTTP  Header Belirtme

```
curl -H "User-Agent: Mozilla/5.0" https://www.example.com
```

```
curl -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"1234"}' https://api.example.com/login
```

```
curl -H "Authorization: Bearer TOKEN123" https://api.example.com/data
```



### cURL ile POST Request'i Gönderme

```
curl -X POST -d "param1=deger1&param2=deger2" domain.com
```


### `-L` Bayrağı (Redirect Takip Etme)

```
curl -L domain.com
```


### `-b` Bayrağı (Cookie Gönderme)

```
curl -b "cookie_adı=cookie_değeri" domain.com
```

veya

```bash
curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/
```


### JSON Verisi Gönderme (POST İsteği)

```
curl -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"1234"}' https://api.example.com/login
```


## CRUD API 

### Read (GET)

```
curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq
```

### Create (POST)

```
curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ -d '{"city_name":"HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```


### Update (PUT)

```
curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london -d '{"city_name":"New_HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```


### Delete (DELETE)

```
curl -X DELETE http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City
```



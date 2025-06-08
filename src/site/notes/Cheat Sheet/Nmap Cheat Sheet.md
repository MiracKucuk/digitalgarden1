---
{"dg-publish":true,"permalink":"/cheat-sheet/nmap-cheat-sheet/","created":"2025-02-21T23:49:49.229+03:00","updated":"2025-02-22T00:42:30.432+03:00"}
---

### TCP SYN Scan

```
nmap -sS 10.10.10.10.
```


### Ping Scan (Host Discovery)

```
nmap 10.10.10.10/24 -sn
```


### Scan a Range of IPs

```
nmap 10.10.10.10-20
```


### Scan Multiple IPs

```
nmap 10.10.10.18 10.10.10.19 10.10.10.20
```

### ICMP Echo Request (Ping) Scan

```
nmap -PE 10.10.10.10
```

### Packet Trace

```
sudo nmap 10.10.10.10 --packet-trace
```

### Port Durumu Analizi

```
nmap --reason
```

### ARP Ping Devre Dışı Bırak

```
nmap `--disable-arp-ping`
```

### Top 100 Portu Tarama

```
nmap -F
```

### Scanning Top 10 TCP Ports

```
nmap 10.10.10.10 --top-ports=10 
```


### Ping Atla (Host Discovery'yi Devre Dışı Bırak)

```
nmap -Pn 10.10.10.10
```


### DNS Çözümlemesini Devre Dışı Bırak

```
nmap -n 10.10.10.10
```


### TCP Connect Scan

```
nmap -sT 10.10.10.10
```

### UDP Port Scan

```
nmap -sU 10.10.10.10
```

### Version Scan 

```
nmap -sV 10.10.10.10
```


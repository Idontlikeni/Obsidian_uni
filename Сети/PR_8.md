1) файл webhook_receiver.py
2) Запуск контейнера с приемником![[Pasted image 20261109105522.png]]
3) blackbox.yml
   ![[Pasted image 20261110191006.png]]
4) 

blackbox.yml:
```
modules:
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ipv4"
  tcp_connect:
    prober: tcp
    timeout: 5s
```
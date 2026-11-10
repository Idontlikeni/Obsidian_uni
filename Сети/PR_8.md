1) файл webhook_receiver.py
2) Запуск контейнера с приемником![[Pasted image 20261109105522.png]]
3) blackbox.yml
   ![[Pasted image 20261110191006.png]]
4) проверка работы![[Pasted image 20261110202116.png]]
5) 

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


============================================================================
Вот список команд и что они выводят:
```
curl -i 'http://127.0.0.1:9115/probe?target=127.0.0.1:6653&module=tcp_connect'
- 
curl: (7) Failed to connect to 127.0.0.1 port 9115 after 0 ms: Couldn't connect to server
```

```
curl -s http://127.0.0.1:9115/config - ничего не выводит
```

файл примонтирован правильно


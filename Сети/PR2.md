Запуск топологии после установки docker контейнера:
```
sudo mn --topo single,4 --controller=remote,ip=127.0.0.1,port=6653 \
--switch ovsk,protocols=OpenFlow13
```

# Выполнение пр
1) Установка и запуск mininet![[Pasted image 20260926195216.png]]
2) проверка работы![[Pasted image 20260926195315.png]]![[Pasted image 20260926195249.png]]
3) Логи до![[Pasted image 20260926195649.png]]
4) Пинг проходит![[Pasted image 20260926211801.png]]
5) Поднимаем слушающий порт на h3, а затем пытаемся подключиться к нему (правило ACL режет подключение)![[Pasted image 20260926212421.png]]
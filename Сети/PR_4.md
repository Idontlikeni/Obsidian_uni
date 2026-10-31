Создание veth-пар и включение узлов в топологию
![[Pasted image 20261031222002.png]]

Успешный пинг L3 на underlay
![[Pasted image 20261031223728.png]]

Построение vxlan-туннеля
![[Pasted image 20261031223759.png]]


ping overlay
![[Pasted image 20261029095018.png]]

Проверка связаности
![[Pasted image 20261031224315.png]]
FDB
![[Pasted image 20261031224348.png]]

# доп. задание
Меняем порт на площадке B (ломаем)

![[Pasted image 20261031224904.png]]

Потеря связяности на overlay, underlay исправлен
![[Pasted image 20261031225200.png]]

Восстановление согласованного порта
![[Pasted image 20261031225641.png]]

Снова 0% pocket loss:
![[Pasted image 20261031225723.png]]
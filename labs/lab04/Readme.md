### Underlay BGP

### Цели:
- 1: Настроить BGP для Underlay сети

### Собрана топология:
![img_1.png](main_topology_lab04.png)
Spine и Leaf - Huawei CE12800, Clients - Linux<br>

### Особенности настройки:
В качестве протокола динамической маршрутизации для Undelay выбран EBGP<br>
В качестве протокола сетевого уровня выбран IPv6<br>
На всех маршрутизаторах настроена аутентификация MD5 для BGP пиров .<br>
Уменьшены таймеры BGP - timer keepalive 3 hold 9.<br>
route-update-interval установлен =0 (по дефолту для EBGP было 30сек.) 

### IP план:

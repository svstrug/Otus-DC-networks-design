### VxLAN. Аналоги VPC.

### Цели:
- 1: Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.
- 2: Протестировать отказоустойчивость - убедиться, что связность не теряется при отключении одного из линков



### Собрана топология:
![image](main_topology_lab07.png)

### Особенности настройки:
Настройки underlay и overlay для L2 и L3 VNI унаследованы с предыдущей лабы №6.<br>
На Leaf-1 и Leaf-2 настроен ESI LAG, 1-й портченнел в сторону Client-1 в режиме trunk для двух vlan 10 и 11, 2-й портченнел в сторону Client-2 в режиме access vlan 11.<br>
Client-3 и Client-4 имеют single homed подключения в vlan 12 и vlan 13 соответственно.<br>
Spine/Leaf - Arista, Clients - Cumulus linux.

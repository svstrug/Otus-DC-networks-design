### Underlay OSPF

### Цели:
- 1: Настроить OSPF для Underlay сети

### Собрана топология:
![img_1.png](main_topology2.png)

### Особенности настройки:
Изменены стандартные таймеры OSPF на интерфейсах  Timers: Hello 3, Dead 12.<br>
На Leaf'ах коммутаторах включен silent-interface all (passive interface default), который отключен только на интерфейсах в сторону Spine'ов.

#### Конфигурация на оборудовании Huawei
<details>
<summary> Spine-1 </summary>
#<br>
sysname Spine-1<br>
#<br>
interface GE1/0/1<br>
 undo portswitch<br>
 description to Leaf-1<br>
 undo shutdown<br>
 ip address 10.4.1.0 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/2<br>
 undo portswitch<br>
 description to Leaf-2<br>
 undo shutdown<br>
 ip address 10.4.1.2 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/3<br>
 undo portswitch<br>
 description to Leaf-3<br>
 undo shutdown<br>
 ip address 10.4.1.4 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface LoopBack1<br>
 description underlay<br>
 ip address 10.0.1.0 255.255.255.255<br>
#<br>
interface LoopBack2<br>
 description overlay<br>
 ip address 10.2.1.0 255.255.255.255<br>
#<br>
ospf 555 router-id 10.0.1.0<br>
 area 0.0.0.0<br>
  network 10.0.1.0 0.0.0.0 description underlay Loopback1<br>
  network 10.2.1.0 0.0.0.0 description overlay Loopback2<br>
  network 10.4.1.0 0.0.0.1 description PtP to Leaf-1<br>
  network 10.4.1.2 0.0.0.1 description PtP to Leaf-2<br>
  network 10.4.1.4 0.0.0.1 description PtP to Leaf-3<br>
#<br>
</details>

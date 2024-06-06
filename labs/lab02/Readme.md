### Underlay OSPF

### Цели:
- 1: Настроить OSPF для Underlay сети

### Собрана топология:
![img_1.png](main_topology2.png)

### Особенности настройки:
Изменены стандартные таймеры OSPF на интерфейсах  Timers: Hello 3, Dead 12 с целью улучшения сходимости протокола.<br>
На Leaf'ах включен silent-interface all (passive interface default), который отключен только на интерфейсах в сторону Spine'ов, чтобы не отправлять hello в клиентские порты.<br>
Применена команда ospf network-type p2p для оптимизации работы протокола.

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
<details>
<summary> Spine-2 </summary>
#<br>
sysname Spine-2<br>
#<br>
interface GE1/0/1<br>
 undo portswitch<br>
 description to Leaf-1<br>
 undo shutdown<br>
 ip address 10.4.2.0 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/2<br>
 undo portswitch<br>
 description to Leaf-2<br>
 undo shutdown<br>
 ip address 10.4.2.2 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/3<br>
 undo portswitch<br>
 description to Leaf-3<br>
 undo shutdown<br>
 ip address 10.4.2.4 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface LoopBack1<br>
 description underlay<br>
 ip address 10.0.2.0 255.255.255.255<br>
#<br>
interface LoopBack2<br>
 description overlay<br>
 ip address 10.2.2.0 255.255.255.255<br>
#<br>
ospf 777 router-id 10.0.2.0<br>
 area 0.0.0.0<br>
  network 10.0.2.0 0.0.0.0 description underlay Loopback1<br>
  network 10.2.2.0 0.0.0.0 description overlay Loopback2<br>
  network 10.4.2.0 0.0.0.1 description PtP to Leaf-1<br>
  network 10.4.2.2 0.0.0.1 description PtP to Leaf-2<br>
  network 10.4.2.4 0.0.0.1 description PtP to Leaf-3<br>
#<br>
</details>
<details>
<summary> Leaf-1 </summary>
#<br>
sysname Leaf-1<br>
#<br>
interface GE1/0/1<br>
 undo portswitch<br>
 description to Spine-1<br>
 undo shutdown<br>
 ip address 10.4.1.1 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/2<br>
 undo portswitch<br>
 description to Spine-2<br>
 undo shutdown<br>
 ip address 10.4.2.1 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/9<br>
 undo portswitch<br>
 description to Client-1<br>
 undo shutdown<br>
 ip address 10.8.0.1 255.255.255.240<br>
#<br>
interface LoopBack1<br>
 description underlay<br>
 ip address 10.0.0.1 255.255.255.255<br>
#<br>
interface LoopBack2<br>
 description overlay<br>
 ip address 10.2.0.1 255.255.255.255<br>
#<br>
ospf 333 router-id 10.0.0.1<br>
 silent-interface all<br>
 undo silent-interface GE1/0/1<br>
 undo silent-interface GE1/0/2<br>
 area 0.0.0.0<br>
  network 10.0.0.1 0.0.0.0 description underlay LoopBack1<br>
  network 10.2.0.1 0.0.0.0 description overlay LoopBack2<br>
  network 10.4.1.0 0.0.0.1 description PtP to Spine-1<br>
  network 10.4.2.0 0.0.0.1 description PtP to Spine-2<br>
  network 10.8.0.0 0.0.0.15 description Client-1 network<br>
#<br>
</details>
<details>
<summary> Leaf-2 </summary>
#<br>
sysname Leaf-2<br>
#<br>
interface GE1/0/1<br>
 undo portswitch<br>
 description to Spine-1<br>
 undo shutdown<br>
 ip address 10.4.1.3 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/2<br>
 undo portswitch<br>
 description to Spine-2<br>
 undo shutdown<br>
 ip address 10.4.2.3 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/9<br>
 undo portswitch<br>
 description to Client-2<br>
 undo shutdown<br>
 ip address 10.8.0.17 255.255.255.240<br>
#<br>
interface LoopBack1<br>
 description underlay<br>
 ip address 10.0.0.2 255.255.255.255<br>
#<br>
interface LoopBack2<br>
 description overlay<br>
 ip address 10.2.0.2 255.255.255.255<br>
#<br>
ospf 200 router-id 10.0.0.2<br>
 silent-interface all<br>
 undo silent-interface GE1/0/1<br>
 undo silent-interface GE1/0/2<br>
 area 0.0.0.0<br>
  network 10.0.0.2 0.0.0.0 description underlay LoopBack1<br>
  network 10.2.0.2 0.0.0.0 description overlay LoopBack2<br>
  network 10.4.1.2 0.0.0.1 description PtP to Spine-1<br>
  network 10.4.2.2 0.0.0.1 description PtP to Spine-2<br>
  network 10.8.0.16 0.0.0.15 description Client-2 network<br>
#<br>
</details>
<details>
<summary> Leaf-3 </summary>
#<br>
 sysname Leaf-3<br>
#<br>
interface GE1/0/1<br>
 undo portswitch<br>
 description to Spine-1<br>
 undo shutdown<br>
 ip address 10.4.1.5 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/2<br>
 undo portswitch<br>
 description to Spine-2<br>
 undo shutdown<br>
 ip address 10.4.2.5 255.255.255.254<br>
 ospf network-type p2p<br>
 ospf timer hello 3<br>
#<br>
interface GE1/0/8<br>
 undo portswitch<br>
 description to Client-4<br>
 undo shutdown<br>
 ip address 10.8.0.41 255.255.255.248<br>
#<br>
interface GE1/0/9<br>
 undo portswitch<br>
 description to Client-3<br>
 undo shutdown<br>
 ip address 10.8.0.33 255.255.255.248<br>
#<br>
interface LoopBack1<br>
 description underlay<br>
 ip address 10.0.0.3 255.255.255.255<br>
#<br>
interface LoopBack2<br>
 description overlay<br>
 ip address 10.2.0.3 255.255.255.255<br>
#<br>
ospf 999 router-id 10.0.0.3<br>
 silent-interface all<br>
 undo silent-interface GE1/0/1<br>
 undo silent-interface GE1/0/2<br>
 area 0.0.0.0<br>
  description overlay Loopback2<br>
  network 10.0.0.3 0.0.0.0 description underlay Loopback1<br>
  network 10.2.0.3 0.0.0.0 description overlay Loopback2<br>
  network 10.4.1.4 0.0.0.1 description PtP to Spine-1<br>
  network 10.4.2.4 0.0.0.1 description PtP to Spine-2<br>
  network 10.8.0.32 0.0.0.7 description Client-3 network<br>
  network 10.8.0.40 0.0.0.7 description Client-4 network<br>
#<br>
</details>

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
  network 10.0.0.3 0.0.0.0 description underlay Loopback1<br>
  network 10.2.0.3 0.0.0.0 description overlay Loopback2<br>
  network 10.4.1.4 0.0.0.1 description PtP to Spine-1<br>
  network 10.4.2.4 0.0.0.1 description PtP to Spine-2<br>
  network 10.8.0.32 0.0.0.7 description Client-3 network<br>
  network 10.8.0.40 0.0.0.7 description Client-4 network<br>
#<br>
</details>

#### Диагностика

<details>
<summary> Spine-1 diag </summary>
 
 ```
<Spine-1>display ip routing-table
Proto: Protocol        Pre: Preference
Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
------------------------------------------------------------------------------
Routing Table : _public_
         Destinations : 27       Routes : 31

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

       10.0.0.1/32  OSPF    10   1             D   10.4.1.1        GE1/0/1
       10.0.0.2/32  OSPF    10   1             D   10.4.1.3        GE1/0/2
       10.0.0.3/32  OSPF    10   1             D   10.4.1.5        GE1/0/3
       10.0.1.0/32  Direct  0    0             D   127.0.0.1       LoopBack1
       10.0.2.0/32  OSPF    10   2             D   10.4.1.5        GE1/0/3
                    OSPF    10   2             D   10.4.1.3        GE1/0/2
                    OSPF    10   2             D   10.4.1.1        GE1/0/1
       10.2.0.1/32  OSPF    10   1             D   10.4.1.1        GE1/0/1
       10.2.0.2/32  OSPF    10   1             D   10.4.1.3        GE1/0/2
       10.2.0.3/32  OSPF    10   1             D   10.4.1.5        GE1/0/3
       10.2.1.0/32  Direct  0    0             D   127.0.0.1       LoopBack2
       10.2.2.0/32  OSPF    10   2             D   10.4.1.5        GE1/0/3
                    OSPF    10   2             D   10.4.1.3        GE1/0/2
                    OSPF    10   2             D   10.4.1.1        GE1/0/1
       10.4.1.0/31  Direct  0    0             D   10.4.1.0        GE1/0/1
       10.4.1.0/32  Direct  0    0             D   127.0.0.1       GE1/0/1
       10.4.1.2/31  Direct  0    0             D   10.4.1.2        GE1/0/2
       10.4.1.2/32  Direct  0    0             D   127.0.0.1       GE1/0/2
       10.4.1.4/31  Direct  0    0             D   10.4.1.4        GE1/0/3
       10.4.1.4/32  Direct  0    0             D   127.0.0.1       GE1/0/3
       10.4.2.0/31  OSPF    10   2             D   10.4.1.1        GE1/0/1
       10.4.2.2/31  OSPF    10   2             D   10.4.1.3        GE1/0/2
       10.4.2.4/31  OSPF    10   2             D   10.4.1.5        GE1/0/3
       10.8.0.0/28  OSPF    10   2             D   10.4.1.1        GE1/0/1
      10.8.0.16/28  OSPF    10   2             D   10.4.1.3        GE1/0/2
      10.8.0.32/29  OSPF    10   2             D   10.4.1.5        GE1/0/3
      10.8.0.40/29  OSPF    10   2             D   10.4.1.5        GE1/0/3
      127.0.0.0/8   Direct  0    0             D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0             D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0

<Spine-1>display ospf peer
OSPF Process 555 with Router ID 10.0.1.0
 Area 0.0.0.0 interface 10.4.1.0 (GE1/0/1)'s neighbors
  Router ID: 10.0.0.1           Address : 10.4.1.1
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 34h42m04s
  Neighbor up time stamp      : 2024-06-05 05:06:35
  Authentication Sequence     : 0

 Area 0.0.0.0 interface 10.4.1.2 (GE1/0/2)'s neighbors
  Router ID: 10.0.0.2           Address : 10.4.1.3
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 34h21m04s
  Neighbor up time stamp      : 2024-06-05 05:27:35
  Authentication Sequence     : 0

 Area 0.0.0.0 interface 10.4.1.4 (GE1/0/3)'s neighbors
  Router ID: 10.0.0.3           Address : 10.4.1.5
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 02h32m53s
  Neighbor up time stamp      : 2024-06-06 13:15:47
  Authentication Sequence     : 0
```
</details>
<details>
<summary> Spine-2 diag </summary>
 
 ```
<Spine-2>display ip routing-table
Proto: Protocol        Pre: Preference
Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
------------------------------------------------------------------------------
Routing Table : _public_
         Destinations : 27       Routes : 31        

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

       10.0.0.1/32  OSPF    10   1             D   10.4.2.1        GE1/0/1
       10.0.0.2/32  OSPF    10   1             D   10.4.2.3        GE1/0/2
       10.0.0.3/32  OSPF    10   1             D   10.4.2.5        GE1/0/3
       10.0.1.0/32  OSPF    10   2             D   10.4.2.5        GE1/0/3
                    OSPF    10   2             D   10.4.2.3        GE1/0/2
                    OSPF    10   2             D   10.4.2.1        GE1/0/1
       10.0.2.0/32  Direct  0    0             D   127.0.0.1       LoopBack1
       10.2.0.1/32  OSPF    10   1             D   10.4.2.1        GE1/0/1
       10.2.0.2/32  OSPF    10   1             D   10.4.2.3        GE1/0/2
       10.2.0.3/32  OSPF    10   1             D   10.4.2.5        GE1/0/3
       10.2.1.0/32  OSPF    10   2             D   10.4.2.5        GE1/0/3
                    OSPF    10   2             D   10.4.2.3        GE1/0/2
                    OSPF    10   2             D   10.4.2.1        GE1/0/1
       10.2.2.0/32  Direct  0    0             D   127.0.0.1       LoopBack2
       10.4.1.0/31  OSPF    10   2             D   10.4.2.1        GE1/0/1
       10.4.1.2/31  OSPF    10   2             D   10.4.2.3        GE1/0/2
       10.4.1.4/31  OSPF    10   2             D   10.4.2.5        GE1/0/3
       10.4.2.0/31  Direct  0    0             D   10.4.2.0        GE1/0/1
       10.4.2.0/32  Direct  0    0             D   127.0.0.1       GE1/0/1
       10.4.2.2/31  Direct  0    0             D   10.4.2.2        GE1/0/2
       10.4.2.2/32  Direct  0    0             D   127.0.0.1       GE1/0/2
       10.4.2.4/31  Direct  0    0             D   10.4.2.4        GE1/0/3
       10.4.2.4/32  Direct  0    0             D   127.0.0.1       GE1/0/3
       10.8.0.0/28  OSPF    10   2             D   10.4.2.1        GE1/0/1
      10.8.0.16/28  OSPF    10   2             D   10.4.2.3        GE1/0/2
      10.8.0.32/29  OSPF    10   2             D   10.4.2.5        GE1/0/3
      10.8.0.40/29  OSPF    10   2             D   10.4.2.5        GE1/0/3
      127.0.0.0/8   Direct  0    0             D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0             D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0

<Spine-2>display ospf peer 
OSPF Process 777 with Router ID 10.0.2.0
 Area 0.0.0.0 interface 10.4.2.0 (GE1/0/1)'s neighbors
  Router ID: 10.0.0.1           Address : 10.4.2.1         
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 03h26m21s
  Neighbor up time stamp      : 2024-06-06 13:23:46
  Authentication Sequence     : 0 

 Area 0.0.0.0 interface 10.4.2.2 (GE1/0/2)'s neighbors
  Router ID: 10.0.0.2           Address : 10.4.2.3         
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 12
  Retrans timer interval      : 5
  Neighbor up time            : 04h24m30s
  Neighbor up time stamp      : 2024-06-06 12:25:36
  Authentication Sequence     : 0 

 Area 0.0.0.0 interface 10.4.2.4 (GE1/0/3)'s neighbors
  Router ID: 10.0.0.3           Address : 10.4.2.5         
  State    : Full               Mode    : Nbr is Slave       Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 03h34m20s
  Neighbor up time stamp      : 2024-06-06 13:15:46
  Authentication Sequence     : 0 
```
</details>
<details>
<summary> Leaf-1 diag </summary>
 
 ```
<Leaf-1>display ip routing-table 
Proto: Protocol        Pre: Preference
Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
------------------------------------------------------------------------------
Routing Table : _public_
         Destinations : 28       Routes : 35        

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

       10.0.0.1/32  Direct  0    0             D   127.0.0.1       LoopBack1
       10.0.0.2/32  OSPF    10   2             D   10.4.2.0        GE1/0/2
                    OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.0.0.3/32  OSPF    10   2             D   10.4.2.0        GE1/0/2
                    OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.0.1.0/32  OSPF    10   1             D   10.4.1.0        GE1/0/1
       10.0.2.0/32  OSPF    10   1             D   10.4.2.0        GE1/0/2
       10.2.0.1/32  Direct  0    0             D   127.0.0.1       LoopBack2
       10.2.0.2/32  OSPF    10   2             D   10.4.2.0        GE1/0/2
                    OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.2.0.3/32  OSPF    10   2             D   10.4.2.0        GE1/0/2
                    OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.2.1.0/32  OSPF    10   1             D   10.4.1.0        GE1/0/1
       10.2.2.0/32  OSPF    10   1             D   10.4.2.0        GE1/0/2
       10.4.1.0/31  Direct  0    0             D   10.4.1.1        GE1/0/1
       10.4.1.1/32  Direct  0    0             D   127.0.0.1       GE1/0/1
       10.4.1.2/31  OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.4.1.4/31  OSPF    10   2             D   10.4.1.0        GE1/0/1
       10.4.2.0/31  Direct  0    0             D   10.4.2.1        GE1/0/2
       10.4.2.1/32  Direct  0    0             D   127.0.0.1       GE1/0/2
       10.4.2.2/31  OSPF    10   2             D   10.4.2.0        GE1/0/2
       10.4.2.4/31  OSPF    10   2             D   10.4.2.0        GE1/0/2
       10.8.0.0/28  Direct  0    0             D   10.8.0.1        GE1/0/9
       10.8.0.1/32  Direct  0    0             D   127.0.0.1       GE1/0/9
      10.8.0.15/32  Direct  0    0             D   127.0.0.1       GE1/0/9
      10.8.0.16/28  OSPF    10   3             D   10.4.2.0        GE1/0/2
                    OSPF    10   3             D   10.4.1.0        GE1/0/1
      10.8.0.32/29  OSPF    10   3             D   10.4.2.0        GE1/0/2
                    OSPF    10   3             D   10.4.1.0        GE1/0/1
      10.8.0.40/29  OSPF    10   3             D   10.4.2.0        GE1/0/2
                    OSPF    10   3             D   10.4.1.0        GE1/0/1
      127.0.0.0/8   Direct  0    0             D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0             D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0

<Leaf-1> display ospf peer
OSPF Process 333 with Router ID 10.0.0.1
 Area 0.0.0.0 interface 10.4.1.1 (GE1/0/1)'s neighbors
  Router ID: 10.0.1.0           Address : 10.4.1.0         
  State    : Full               Mode    : Nbr is Master      Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 11
  Retrans timer interval      : 5
  Neighbor up time            : 35h46m39s
  Neighbor up time stamp      : 2024-06-05 05:06:35
  Authentication Sequence     : 0 

 Area 0.0.0.0 interface 10.4.2.1 (GE1/0/2)'s neighbors
  Router ID: 10.0.2.0           Address : 10.4.2.0         
  State    : Full               Mode    : Nbr is Master      Priority: 1
  DR       : None               BDR     : None               MTU     : 0
  Dead timer due (in seconds) : 10
  Retrans timer interval      : 5
  Neighbor up time            : 03h29m29s
  Neighbor up time stamp      : 2024-06-06 13:23:47
  Authentication Sequence     : 0 
```
</details>

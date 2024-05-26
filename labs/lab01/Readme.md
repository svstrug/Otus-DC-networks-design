### Проектирование адресного пространства

### Цели:
- 1: Собрать схему CLOS;
- 2: Распределить адресное пространство;

### Собрана топология:
![img_1.png](main_topology.png)

### IP адресация:
Принцип распределения IP адресов - немного изменил пример с занятия в сторону увеличения количества адресов на каждый DC /12, всего до 16 DC.

DC1<br> 
0 – Lo1 (/15) (16*(N-1))<br> 
10.0.0.0/15<br> 
2 – Lo2 (/15) (16*(N-1) + 2)<br> 
10.2.0.0/15<br> 
Суммарный для Lo1 и Lo2 – 10.0.0.0/14<br> 

4 – p2p links (/15) (16*(N-1) + 4)<br> 
10.4.0.0/15<br> 
6 – резерв (/15) (16*(N-1) + 6)<br> 
10.6.0.0/15<br> 
Суммарный для p2p и резерва – 10.4.0.0/14<br> 

[8 .. 15] – services (/16) (16*(N-1) + [8 .. 15])<br> 
10.8.0.0/16<br> 
10.9.0.0/16<br> 
10.10.0.0/16<br> 
10.11.0.0/16<br> 
10.12.0.0/16<br> 
10.13.0.0/16<br> 
10.14.0.0/16<br> 
10.15.0.0/16<br> 
Суммарный – 10.8.0.0/13<br> 

DC1 – 10.0.0.0/12<br> 
DC2 – 10.16.0.0/12<br> 
…<br> 
DCN – 10.16*(N-1).0.0/12<br> 
N = [1 .. 16]<br> 

### IP план:
Device|Interface|IP Address|Subnet Mask
---|---|---|---
Spine-1|Lo1|10.0.1.0|255.255.255.255
-|Lo2|10.2.1.0|255.255.255.255
-|GE1/0/1|10.4.1.0|255.255.255.254
-|GE1/0/2|10.4.1.2|255.255.255.254
-|GE1/0/3|10.4.1.4|255.255.255.254
Spine-2|Lo1|10.0.2.0|255.255.255.255
-|Lo2|10.2.2.0|255.255.255.255
-|GE1/0/1|10.4.2.0|255.255.255.254
-|GE1/0/2|10.4.2.2|255.255.255.254
-|GE1/0/3|10.4.2.4|255.255.255.254
Leaf-1|Lo1|10.0.0.1|255.255.255.255
-|Lo2|10.2.0.1|255.255.255.255
-|GE1/0/1|10.4.1.1|255.255.255.254
-|GE1/0/2|10.4.2.1|255.255.255.254
-|GE1/0/9|10.8.0.1|255.255.255.240
Leaf-2|Lo1|10.0.0.2|255.255.255.255
-|Lo2|10.2.0.2|255.255.255.255
-|GE1/0/1|10.4.1.3|255.255.255.254
-|GE1/0/2|10.4.2.3|255.255.255.254
-|GE1/0/9|10.8.0.17|255.255.255.240
Leaf-3|Lo1|10.0.0.3|255.255.255.255
-|Lo2|10.2.0.3|255.255.255.255
-|GE1/0/1|10.4.1.5|255.255.255.254
-|GE1/0/2|10.4.2.5|255.255.255.254
-|GE1/0/9|10.8.0.33|255.255.255.248
-|GE1/0/8|10.8.0.41|255.255.255.248
Client-1|eth0|10.8.0.2|255.255.255.240
Client-2|eth0|10.8.0.18|255.255.255.240
Client-3|eth0|10.8.0.34|255.255.255.248
Client-4|eth0|10.8.0.42|255.255.255.248

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
#<br> 
interface GE1/0/2<br> 
 undo portswitch<br> 
 description to Leaf-2<br> 
 undo shutdown<br> 
 ip address 10.4.1.2 255.255.255.254<br> 
#<br> 
interface GE1/0/3<br> 
 undo portswitch<br> 
 description to Leaf-3<br> 
 undo shutdown<br> 
 ip address 10.4.1.4 255.255.255.254<br> 
#<br> 
interface LoopBack1<br> 
 description underlay<br> 
 ip address 10.0.1.0 255.255.255.255<br> 
#<br> 
interface LoopBack2<br> 
 description overlay<br> 
 ip address 10.2.1.0 255.255.255.255<br> 
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
#<br> 
interface GE1/0/2<br> 
 undo portswitch<br> 
 description to Leaf-2<br> 
 undo shutdown<br> 
 ip address 10.4.2.2 255.255.255.254<br> 
#<br> 
interface GE1/0/3<br> 
 undo portswitch<br> 
 description to Leaf-3<br> 
 undo shutdown<br> 
 ip address 10.4.2.4 255.255.255.254<br> 
#<br> 
interface LoopBack1<br> 
 description underlay<br> 
 ip address 10.0.2.0 255.255.255.255<br> 
#<br> 
interface LoopBack2<br> 
 description overlay<br> 
 ip address 10.2.2.0 255.255.255.255<br> 
#<br> 
</details>
<details>
<summary> Leaf-1 </summary>
</details>
<details>
<summary> Leaf-2 </summary>
</details>
<details>
<summary> Leaf-3 </summary>
</details>
<details>
<summary> Clients 1-4 </summary>
</details>

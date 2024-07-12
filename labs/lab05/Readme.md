### VxLAN. L2 VNI.

### Цели:
- 1: Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

### Собрана топология:
![image](main_topology_lab05.png)

### Особенности настройки:
Uderlay OSPF.<br>
Overlay eBGP.<br>

### IP план:
Device|Interface|IP Address|Subnet Mask
---|---|---|---
Spine-1|Lo1|10.0.1.0|255.255.255.255
-|Lo2|10.2.1.0|255.255.255.255
-|Eth1|10.4.1.0|255.255.255.254
-|Eth2|10.4.1.2|255.255.255.254
-|Eth3|10.4.1.4|255.255.255.254
Spine-2|Lo1|10.0.2.0|255.255.255.255
-|Lo2|10.2.2.0|255.255.255.255
-|Eth1|10.4.2.0|255.255.255.254
-|Eth2|10.4.2.2|255.255.255.254
-|Eth3|10.4.2.4|255.255.255.254
Leaf-1|Lo1|10.0.0.1|255.255.255.255
-|Lo2|10.2.0.1|255.255.255.255
-|Eth1|10.4.1.1|255.255.255.254
-|Eth2|10.4.2.1|255.255.255.254
Leaf-2|Lo1|10.0.0.2|255.255.255.255
-|Lo2|10.2.0.2|255.255.255.255
-|Eth1|10.4.1.3|255.255.255.254
-|Eth2|10.4.2.3|255.255.255.254
Leaf-3|Lo1|10.0.0.3|255.255.255.255
-|Lo2|10.2.0.3|255.255.255.255
-|Eth1|10.4.1.5|255.255.255.254
-|Eth2|10.4.2.5|255.255.255.254
Client-1|eth0|192.168.1.1|255.255.255.0
Client-2|eth0|192.168.1.2|255.255.255.0
Client-3|eth0|192.168.1.3|255.255.255.0
Client-4|eth0|192.168.1.4|255.255.255.0

#### Конфигурация на оборудовании Arista.
<details>
<summary> Spine-1 </summary>
#<br>
Spine-1#sh run<br>
! Command: show running-config<br>
! device: Spine-1 (vEOS-lab, EOS-4.29.2F)<br>
!<br>
! boot system flash:/vEOS-lab.swi<br>
!<br>
no aaa root<br>
!<br>
transceiver qsfp default-mode 4x10G<br>
!<br>
service routing protocols model multi-agent<br>
!<br>
hostname Spine-1<br>
!<br>
spanning-tree mode mstp<br>
!<br>
interface Ethernet1<br>
   description Leaf-1 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.0/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet2<br>
   description Leaf-2 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.2/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet3<br>
   description Leaf-3 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.4/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback1<br>
   description Underlay<br>
   ip address 10.0.1.0/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback2<br>
   description Overlay<br>
   ip address 10.2.1.0/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
ip routing<br>
!<br>
router bgp 65000<br>
   neighbor evpn peer group<br>
   neighbor evpn next-hop-unchanged<br>
   neighbor evpn update-source Loopback2<br>
   neighbor evpn ebgp-multihop 3<br>
   neighbor evpn send-community extended<br>
   neighbor 10.2.0.1 peer group evpn<br>
   neighbor 10.2.0.1 remote-as 65001<br>
   neighbor 10.2.0.2 peer group evpn<br>
   neighbor 10.2.0.2 remote-as 65002<br>
   neighbor 10.2.0.3 peer group evpn<br>
   neighbor 10.2.0.3 remote-as 65003<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
!<br>
router ospf 1<br>
   router-id 10.0.1.0<br>
   auto-cost reference-bandwidth 10000<br>
   passive-interface default<br>
   no passive-interface Ethernet1<br>
   no passive-interface Ethernet2<br>
   no passive-interface Ethernet3<br>
   network 0.0.0.0/0 area 0.0.0.0<br>
   max-lsa 12000<br>
</details>
<details>
<summary> Spine-2 </summary>
#<br>
Spine-2#sh run<br>
! Command: show running-config<br>
! device: Spine-2 (vEOS-lab, EOS-4.29.2F)<br>
!<br>
! boot system flash:/vEOS-lab.swi<br>
!<br>
no aaa root<br>
!<br>
transceiver qsfp default-mode 4x10G<br>
!<br>
service routing protocols model multi-agent<br>
!<br>
hostname Spine-2<br>
!<br>
spanning-tree mode mstp<br>
!<br>
interface Ethernet1<br>
   description Leaf-1 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.0/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet2<br>
   description Leaf-2 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.2/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet3<br>
   description Leaf-3 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.4/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback1<br>
   description Underlay<br>
   ip address 10.0.2.0/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback2<br>
   description Overlay<br>
   ip address 10.2.2.0/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
ip routing<br>
!<br>
router bgp 65000<br>
   neighbor evpn peer group<br>
   neighbor evpn next-hop-unchanged<br>
   neighbor evpn update-source Loopback2<br>
   neighbor evpn ebgp-multihop 3<br>
   neighbor evpn send-community extended<br>
   neighbor 10.2.0.1 peer group evpn<br>
   neighbor 10.2.0.1 remote-as 65001<br>
   neighbor 10.2.0.2 peer group evpn<br>
   neighbor 10.2.0.2 remote-as 65002<br>
   neighbor 10.2.0.3 peer group evpn<br>
   neighbor 10.2.0.3 remote-as 65003<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
!<br>
router ospf 1<br>
   router-id 10.0.2.0<br>
   auto-cost reference-bandwidth 10000<br>
   passive-interface default<br>
   no passive-interface Ethernet1<br>
   no passive-interface Ethernet2<br>
   no passive-interface Ethernet3<br>
   network 0.0.0.0/0 area 0.0.0.0<br>
   max-lsa 12000<br>
</details>

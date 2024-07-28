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

### IP план:
Device|Interface|IP Address|Subnet Mask|Default GW
---|---|---|---|---
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
-|vlan10|192.168.10.254|255.255.255.0
-|vlan11|192.168.11.254|255.255.255.0
Leaf-2|Lo1|10.0.0.2|255.255.255.255
-|Lo2|10.2.0.2|255.255.255.255
-|Eth1|10.4.1.3|255.255.255.254
-|Eth2|10.4.2.3|255.255.255.254
-|vlan10|192.168.10.254|255.255.255.0
-|vlan11|192.168.11.254|255.255.255.0
Leaf-3|Lo1|10.0.0.3|255.255.255.255
-|Lo2|10.2.0.3|255.255.255.255
-|Eth1|10.4.1.5|255.255.255.254
-|Eth2|10.4.2.5|255.255.255.254
-|vlan12|192.168.12.254|255.255.255.0
-|vlan13|192.168.13.254|255.255.255.0
Client-1|bond1.10|192.168.10.1|255.255.255.0|192.168.10.254
-|bond1.11|192.168.11.1|255.255.255.0|192.168.11.254
Client-2|bond1|192.168.11.2|255.255.255.0|192.168.11.254
Client-3|swp1|192.168.12.1|255.255.255.0|192.168.12.254
Client-4|swp1|192.168.13.1|255.255.255.0|192.168.13.254

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
<details>
<summary> Leaf-1 </summary>
<br>
Leaf-1#show running-config <br>
! Command: show running-config<br>
! device: Leaf-1 (vEOS-lab, EOS-4.29.2F)<br>
!<br>
! boot system flash:/vEOS-lab.swi<br>
!<br>
no aaa root<br>
!<br>
transceiver qsfp default-mode 4x10G<br>
!<br>
service routing protocols model multi-agent<br>
!<br>
hostname Leaf-1<br>
!<br>
spanning-tree mode mstp<br>
!<br>
vlan 10<br>
   name data1<br>
!<br>
vlan 11<br>
   name data2<br>
!<br>
vrf instance vrf-vxlan<br>
!<br>
interface Port-Channel1<br>
   switchport trunk allowed vlan 10-11<br>
   switchport mode trunk<br>
   !<br>
   evpn ethernet-segment<br>
      identifier 00cc:cccc:cccc:cccc:cccc<br>
      route-target import cc:cc:cc:cc:cc:cc<br>
   lacp system-id 1111.1111.1111<br>
!<br>
interface Port-Channel2<br>
   switchport access vlan 11<br>
   !<br>
   evpn ethernet-segment<br>
      identifier 00dd:dddd:dddd:dddd:dddd<br>
      route-target import dd:dd:dd:dd:dd:dd<br>
   lacp system-id 1111.1111.1111<br>
!<br>
interface Ethernet1<br>
   description Spine-1 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.1/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet2<br>
   description Spine-2 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.1/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet3<br>
   channel-group 1 mode active<br>
!<br>
interface Ethernet4<br>
   channel-group 2 mode active<br>
!<br>
interface Loopback1<br>
   description Underlay<br>
   ip address 10.0.0.1/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback2<br>
   description Overlay<br>
   ip address 10.2.0.1/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Vlan10<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.10.254/24<br>
!<br>
interface Vlan11<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.11.254/24<br>
!<br>
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 10 vni 1010<br>
   vxlan vlan 11 vni 1011<br>
   vxlan vrf vrf-vxlan vni 50000<br>
   vxlan learn-restrict any<br>
!<br>
ip virtual-router mac-address 00:00:11:22:33:44<br>
!<br>
ip routing<br>
ip routing vrf vrf-vxlan<br>
!<br>
router bgp 65001<br>
   neighbor evpn peer group<br>
   neighbor evpn remote-as 65000<br>
   neighbor evpn update-source Loopback2<br>
   neighbor evpn ebgp-multihop 3<br>
   neighbor evpn send-community extended<br>
   neighbor 10.2.1.0 peer group evpn<br>
   neighbor 10.2.2.0 peer group evpn<br>
   !<br>
   vlan 10<br>
      rd 65001:1010<br>
      route-target both 10:1010<br>
      redistribute learned<br>
   !<br>
   vlan 11<br>
      rd 65001:1011<br>
      route-target both 11:1011<br>
      redistribute learned<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   vrf vrf-vxlan<br>
      rd 10.2.0.1:1<br>
      route-target import evpn 1:50000<br>
      route-target export evpn 1:50000<br>
      redistribute connected<br>
!<br>
router ospf 1<br>
   router-id 10.0.0.1<br>
   auto-cost reference-bandwidth 10000<br>
   passive-interface default<br>
   no passive-interface Ethernet1<br>
   no passive-interface Ethernet2<br>
   network 0.0.0.0/0 area 0.0.0.0<br>
   max-lsa 12000<br>
</details>
<details>
<summary> Leaf-2 </summary>
<br>
Leaf-2#show run<br>
! Command: show running-config<br>
! device: Leaf-2 (vEOS-lab, EOS-4.29.2F)<br>
!<br>
! boot system flash:/vEOS-lab.swi<br>
!<br>
no aaa root<br>
!<br>
transceiver qsfp default-mode 4x10G<br>
!<br>
service routing protocols model multi-agent<br>
!<br>
hostname Leaf-2<br>
!<br>
spanning-tree mode mstp<br>
!<br>
vlan 10<br>
   name data1<br>
!<br>
vlan 11<br>
   name data2<br>
!<br>
vrf instance vrf-vxlan<br>
!<br>
interface Port-Channel1<br>
   switchport trunk allowed vlan 10-11<br>
   switchport mode trunk<br>
   !<br>
   evpn ethernet-segment<br>
      identifier 00cc:cccc:cccc:cccc:cccc<br>
      route-target import cc:cc:cc:cc:cc:cc<br>
   lacp system-id 1111.1111.1111<br>
!<br>
interface Port-Channel2<br>
   switchport access vlan 11<br>
   !<br>
   evpn ethernet-segment<br>
      identifier 00dd:dddd:dddd:dddd:dddd<br>
      route-target import dd:dd:dd:dd:dd:dd<br>
   lacp system-id 1111.1111.1111<br>
!<br>
interface Ethernet1<br>
   description Spine-1 | Eth2<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.3/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet2<br>
   description Spine-2 | Eth2<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.3/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet3<br>
   channel-group 1 mode active<br>
!<br>
interface Ethernet4<br>
   channel-group 2 mode active<br>
!<br>
interface Ethernet5<br>
!<br>
interface Ethernet6<br>
!<br>
interface Ethernet7<br>
!<br>
interface Ethernet8<br>
!<br>
interface Loopback1<br>
   description Underlay<br>
   ip address 10.0.0.2/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback2<br>
   description Overlay<br>
   ip address 10.2.0.2/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Management1<br>
!<br>
interface Vlan10<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.10.254/24<br>
!<br>
interface Vlan11<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.11.254/24<br>
!<br>
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 10 vni 1010<br>
   vxlan vlan 11 vni 1011<br>
   vxlan vrf vrf-vxlan vni 50000<br>
   vxlan learn-restrict any<br>
!<br>
ip virtual-router mac-address 00:00:11:22:33:44<br>
!<br>
ip routing<br>
ip routing vrf vrf-vxlan<br>
!<br>
router bgp 65002<br>
   neighbor evpn peer group<br>
   neighbor evpn remote-as 65000<br>
   neighbor evpn update-source Loopback2<br>
   neighbor evpn ebgp-multihop 3<br>
   neighbor evpn send-community extended<br>
   neighbor 10.2.1.0 peer group evpn<br>
   neighbor 10.2.2.0 peer group evpn<br>
   !<br>
   vlan 10<br>
      rd 65002:1010<br>
      route-target both 10:1010<br>
      redistribute learned<br>
   !<br>
   vlan 11<br>
      rd 65002:1011<br>
      route-target both 11:1011<br>
      redistribute learned<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   vrf vrf-vxlan<br>
      rd 10.2.0.2:1<br>
      route-target import evpn 1:50000<br>
      route-target export evpn 1:50000<br>
      redistribute connected<br>
!<br>
router ospf 1<br>
   router-id 10.0.0.2<br>
   auto-cost reference-bandwidth 10000<br>
   passive-interface default<br>
   no passive-interface Ethernet1<br>
   no passive-interface Ethernet2<br>
   network 0.0.0.0/0 area 0.0.0.0<br>
   max-lsa 12000<br>
</details>
<details>
<summary> Leaf-3 </summary>
<br>
Leaf-3#show running-config <br>
! Command: show running-config<br>
! device: Leaf-3 (vEOS-lab, EOS-4.29.2F)<br>
!<br>
! boot system flash:/vEOS-lab.swi<br>
!<br>
no aaa root<br>
!<br>
transceiver qsfp default-mode 4x10G<br>
!<br>
service routing protocols model multi-agent<br>
!<br>
hostname Leaf-3<br>
!<br>
spanning-tree mode mstp<br>
!<br>
vlan 12<br>
   name Client-3<br>
!<br>
vlan 13<br>
   name Client-4<br>
!<br>
vrf instance vrf-vxlan<br>
!<br>
interface Ethernet1<br>
   description Spine-1 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.1.5/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet2<br>
   description Spine-2 | Eth1<br>
   mtu 9214<br>
   no switchport<br>
   ip address 10.4.2.5/31<br>
   ip ospf network point-to-point<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Ethernet3<br>
   switchport access vlan 12<br>
!<br>
interface Ethernet4<br>
   switchport access vlan 13<br>
!<br>
interface Loopback1<br>
   description Underlay<br>
   ip address 10.0.0.3/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Loopback2<br>
   description Overlay<br>
   ip address 10.2.0.3/32<br>
   ip ospf area 0.0.0.0<br>
!<br>
interface Vlan12<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.12.254/24<br>
!<br>
interface Vlan13<br>
   vrf vrf-vxlan<br>
   ip address virtual 192.168.13.254/24<br>
!<br>
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 12 vni 1012<br>
   vxlan vlan 13 vni 1013<br>
   vxlan vrf vrf-vxlan vni 50000<br>
   vxlan learn-restrict any<br>
!<br>
ip virtual-router mac-address 00:00:11:22:33:44<br>
!<br>
ip routing<br>
ip routing vrf vrf-vxlan<br>
!<br>
router bgp 65003<br>
   neighbor evpn peer group<br>
   neighbor evpn remote-as 65000<br>
   neighbor evpn update-source Loopback2<br>
   neighbor evpn ebgp-multihop 3<br>
   neighbor evpn send-community extended<br>
   neighbor 10.2.1.0 peer group evpn<br>
   neighbor 10.2.2.0 peer group evpn<br>
   !<br>
   vlan 12<br>
      rd 65003:1012<br>
      route-target both 12:1012<br>
      redistribute learned<br>
   !<br>
   vlan 13<br>
      rd 65003:1013<br>
      route-target both 13:1013<br>
      redistribute learned<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   vrf vrf-vxlan<br>
      rd 10.2.0.3:1<br>
      route-target import evpn 1:50000<br>
      route-target export evpn 1:50000<br>
      redistribute connected<br>
!<br>
router ospf 1<br>
   router-id 10.0.0.3<br>
   auto-cost reference-bandwidth 10000<br>
   passive-interface default<br>
   no passive-interface Ethernet1<br>
   no passive-interface Ethernet2<br>
   network 0.0.0.0/0 area 0.0.0.0<br>
   max-lsa 12000<br>
</details>

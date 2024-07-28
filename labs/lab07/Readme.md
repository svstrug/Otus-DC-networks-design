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

#### Диагностика c Leaf все dual homed линки Client-1 и Client-2 в работе 

<details>
<summary> Leaf-1 diag </summary>
 
 ```
Leaf-1#show bgp evpn instance vlan 10
EVPN instance: VLAN 10
  Route distinguisher: 65001:1010
  Route target import: Route-Target-AS:10:1010
  Route target export: Route-Target-AS:10:1010
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.1
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.1
      Non-Designated forwarder: 10.2.0.2
	  
Leaf-1#show bgp evpn instance vlan 11
EVPN instance: VLAN 11
  Route distinguisher: 65001:1011
  Route target import: Route-Target-AS:11:1011
  Route target export: Route-Target-AS:11:1011
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.1
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00dd:dddd:dddd:dddd:dddd
      Interface: Port-Channel2
      Mode: all-active
      State: up
      ES-Import RT: dd:dd:dd:dd:dd:dd
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.2
      Non-Designated forwarder: 10.2.0.1
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.2
      Non-Designated forwarder: 10.2.0.1
	  
Leaf-1#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >      RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >Ec    RD: 65002:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 65002:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 65001:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i
 * >Ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 10.2.0.1:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i

Leaf-1#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 10.2.0.1:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
```
</details>
<details>
<summary> Leaf-2 diag </summary>
 
 ```
Leaf-2#show bgp evpn instance vlan 10
EVPN instance: VLAN 10
  Route distinguisher: 65002:1010
  Route target import: Route-Target-AS:10:1010
  Route target export: Route-Target-AS:10:1010
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.2
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.1
      Non-Designated forwarder: 10.2.0.2
	  
Leaf-2#show bgp evpn instance vlan 11
EVPN instance: VLAN 11
  Route distinguisher: 65002:1011
  Route target import: Route-Target-AS:11:1011
  Route target export: Route-Target-AS:11:1011
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.2
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00dd:dddd:dddd:dddd:dddd
      Interface: Port-Channel2
      Mode: all-active
      State: up
      ES-Import RT: dd:dd:dd:dd:dd:dd
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.2
      Non-Designated forwarder: 10.2.0.1
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election algorithm: modulus
      Designated forwarder: 10.2.0.2
      Non-Designated forwarder: 10.2.0.1
	  
Leaf-2#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 65002:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >      RD: 65002:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 10.2.0.2:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >Ec    RD: 65001:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i

Leaf-2#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 10.2.0.2:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 -                     -       -       0       i
```
</details>
<details>
<summary> Leaf-3 diag </summary>
 
 ```
Leaf-3#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65002:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 65002:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 65001:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i

Leaf-3#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
```
</details>

#### Проверка наличия IP связности у клиентов:

<details>
<summary> Client-1 diag </summary>
 
 ```
cumulus@client-1:~$ ping -c 5 192.168.11.2
PING 192.168.11.2 (192.168.11.2) 56(84) bytes of data.
64 bytes from 192.168.11.2: icmp_seq=1 ttl=64 time=5.88 ms
64 bytes from 192.168.11.2: icmp_seq=2 ttl=64 time=5.39 ms
64 bytes from 192.168.11.2: icmp_seq=3 ttl=64 time=5.39 ms
64 bytes from 192.168.11.2: icmp_seq=4 ttl=64 time=6.08 ms
64 bytes from 192.168.11.2: icmp_seq=5 ttl=64 time=5.80 ms

--- 192.168.11.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 5.392/5.712/6.087/0.276 ms

cumulus@client-1:~$ ping -c 5 192.168.12.1
PING 192.168.12.1 (192.168.12.1) 56(84) bytes of data.
64 bytes from 192.168.12.1: icmp_seq=1 ttl=62 time=26.8 ms
64 bytes from 192.168.12.1: icmp_seq=2 ttl=62 time=37.6 ms
64 bytes from 192.168.12.1: icmp_seq=3 ttl=62 time=19.4 ms
64 bytes from 192.168.12.1: icmp_seq=4 ttl=62 time=20.0 ms
64 bytes from 192.168.12.1: icmp_seq=5 ttl=62 time=38.4 ms

--- 192.168.12.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 19.485/28.501/38.493/8.245 ms

cumulus@client-1:~$ ping -c 5 192.168.13.1
PING 192.168.13.1 (192.168.13.1) 56(84) bytes of data.
64 bytes from 192.168.13.1: icmp_seq=1 ttl=62 time=22.0 ms
64 bytes from 192.168.13.1: icmp_seq=2 ttl=62 time=22.2 ms
64 bytes from 192.168.13.1: icmp_seq=3 ttl=62 time=19.1 ms
64 bytes from 192.168.13.1: icmp_seq=4 ttl=62 time=20.8 ms
64 bytes from 192.168.13.1: icmp_seq=5 ttl=62 time=21.1 ms

--- 192.168.13.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 19.199/21.095/22.230/1.078 ms
```
</details>
<details>
<summary> Client-2 diag </summary>
 
 ```
cumulus@client-2:~$ ping -c 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=8.97 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=6.81 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=64 time=6.99 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=64 time=8.18 ms
64 bytes from 192.168.10.1: icmp_seq=5 ttl=64 time=8.20 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 6.812/7.833/8.976/0.817 ms

cumulus@client-2:~$ ping -c 5 192.168.11.1
PING 192.168.11.1 (192.168.11.1) 56(84) bytes of data.
64 bytes from 192.168.11.1: icmp_seq=1 ttl=64 time=5.59 ms
64 bytes from 192.168.11.1: icmp_seq=2 ttl=64 time=5.53 ms
64 bytes from 192.168.11.1: icmp_seq=3 ttl=64 time=6.23 ms
64 bytes from 192.168.11.1: icmp_seq=4 ttl=64 time=5.70 ms
64 bytes from 192.168.11.1: icmp_seq=5 ttl=64 time=4.88 ms

--- 192.168.11.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 4.880/5.592/6.239/0.433 ms

cumulus@client-2:~$ ping -c 5 192.168.12.1
PING 192.168.12.1 (192.168.12.1) 56(84) bytes of data.
64 bytes from 192.168.12.1: icmp_seq=1 ttl=62 time=24.6 ms
64 bytes from 192.168.12.1: icmp_seq=2 ttl=62 time=19.2 ms
64 bytes from 192.168.12.1: icmp_seq=3 ttl=62 time=19.0 ms
64 bytes from 192.168.12.1: icmp_seq=4 ttl=62 time=24.2 ms
64 bytes from 192.168.12.1: icmp_seq=5 ttl=62 time=19.1 ms

--- 192.168.12.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 19.025/21.251/24.645/2.608 ms

cumulus@client-2:~$ ping -c 5 192.168.13.1
PING 192.168.13.1 (192.168.13.1) 56(84) bytes of data.
64 bytes from 192.168.13.1: icmp_seq=1 ttl=62 time=21.2 ms
64 bytes from 192.168.13.1: icmp_seq=2 ttl=62 time=29.5 ms
64 bytes from 192.168.13.1: icmp_seq=3 ttl=62 time=21.6 ms
64 bytes from 192.168.13.1: icmp_seq=4 ttl=62 time=21.8 ms
64 bytes from 192.168.13.1: icmp_seq=5 ttl=62 time=21.6 ms

--- 192.168.13.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 21.271/23.212/29.556/3.182 ms
```
</details>

#### Диагностика c Leaf резервные линки Client-1 и Client-2 выключены в целях имитации отказа<br>
Leaf-1 interface eth4 shutdown<br> 
Leaf-2 interface eth3 shutdown<br> 

<details>
<summary> Leaf-1 diag </summary>
 
 ```
Leaf-1#show bgp evpn instance vlan 10
EVPN instance: VLAN 10
  Route distinguisher: 65001:1010
  Route target import: Route-Target-AS:10:1010
  Route target export: Route-Target-AS:10:1010
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.1
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      Designated forwarder: 10.2.0.1
	  
Leaf-1#show bgp evpn instance vlan 11
EVPN instance: VLAN 11
  Route distinguisher: 65001:1011
  Route target import: Route-Target-AS:11:1011
  Route target export: Route-Target-AS:11:1011
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.1
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00dd:dddd:dddd:dddd:dddd
      Interface: Port-Channel2
      Mode: all-active
      State: down
      ES-Import RT: dd:dd:dd:dd:dd:dd
      DF election state: pending
      Designated forwarder: 
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: cc:cc:cc:cc:cc:cc
      Designated forwarder: 10.2.0.1
	  
Leaf-1#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >      RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >      RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 -                     -       -       0       i
 * >Ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i

Leaf-1#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
```
</details>
<details>
<summary> Leaf-2 diag </summary>
 
 ```
Leaf-2#show bgp evpn instance vlan 10
EVPN instance: VLAN 10
  Route distinguisher: 65002:1010
  Route target import: Route-Target-AS:10:1010
  Route target export: Route-Target-AS:10:1010
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.2
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: down
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election state: pending
      Designated forwarder: 
	  
Leaf-2#show bgp evpn instance vlan 11
EVPN instance: VLAN 11
  Route distinguisher: 65002:1011
  Route target import: Route-Target-AS:11:1011
  Route target export: Route-Target-AS:11:1011
  Service interface: VLAN-based
  Local VXLAN IP address: 10.2.0.2
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 00dd:dddd:dddd:dddd:dddd
      Interface: Port-Channel2
      Mode: all-active
      State: up
      ES-Import RT: dd:dd:dd:dd:dd:dd
      Designated forwarder: 10.2.0.2
    ESI: 00cc:cccc:cccc:cccc:cccc
      Interface: Port-Channel1
      Mode: all-active
      State: down
      ES-Import RT: cc:cc:cc:cc:cc:cc
      DF election state: pending
      Designated forwarder: 

Leaf-2#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i
 * >      RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 -                     -       -       0       i

Leaf-2#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 -                     -       -       0       i
```
</details>
<details>
<summary> Leaf-3 diag </summary>
 
 ```
Leaf-3#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1011 auto-discovery 0 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 auto-discovery 00cc:cccc:cccc:cccc:cccc
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1011 auto-discovery 0 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 auto-discovery 00dd:dddd:dddd:dddd:dddd
                                 10.2.0.2              -       100     0       65000 65002 i

Leaf-3#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 10.2.0.1:1 ethernet-segment 00cc:cccc:cccc:cccc:cccc 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 10.2.0.2:1 ethernet-segment 00dd:dddd:dddd:dddd:dddd 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
```
</details>

#### Проверка наличия IP связности у клиентов:

<details>
<summary> Client-1 diag </summary>
 
 ```
cumulus@client-1:~$ ping -c 5 192.168.11.2
PING 192.168.11.2 (192.168.11.2) 56(84) bytes of data.
64 bytes from 192.168.11.2: icmp_seq=1 ttl=64 time=20.2 ms
64 bytes from 192.168.11.2: icmp_seq=2 ttl=64 time=17.3 ms
64 bytes from 192.168.11.2: icmp_seq=3 ttl=64 time=22.5 ms
64 bytes from 192.168.11.2: icmp_seq=4 ttl=64 time=20.1 ms
64 bytes from 192.168.11.2: icmp_seq=5 ttl=64 time=19.6 ms

--- 192.168.11.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 17.316/19.961/22.510/1.664 ms

cumulus@client-1:~$ ping -c 5 192.168.12.1
PING 192.168.12.1 (192.168.12.1) 56(84) bytes of data.
64 bytes from 192.168.12.1: icmp_seq=1 ttl=62 time=24.2 ms
64 bytes from 192.168.12.1: icmp_seq=2 ttl=62 time=20.2 ms
64 bytes from 192.168.12.1: icmp_seq=3 ttl=62 time=21.8 ms
64 bytes from 192.168.12.1: icmp_seq=4 ttl=62 time=20.3 ms
64 bytes from 192.168.12.1: icmp_seq=5 ttl=62 time=23.4 ms

--- 192.168.12.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 20.227/22.022/24.286/1.629 ms

cumulus@client-1:~$ ping -c 5 192.168.13.1
PING 192.168.13.1 (192.168.13.1) 56(84) bytes of data.
64 bytes from 192.168.13.1: icmp_seq=1 ttl=62 time=22.7 ms
64 bytes from 192.168.13.1: icmp_seq=2 ttl=62 time=18.0 ms
64 bytes from 192.168.13.1: icmp_seq=3 ttl=62 time=37.7 ms
64 bytes from 192.168.13.1: icmp_seq=4 ttl=62 time=22.6 ms
64 bytes from 192.168.13.1: icmp_seq=5 ttl=62 time=22.6 ms

--- 192.168.13.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 18.082/24.771/37.765/6.736 ms
```
</details>
<details>
<summary> Client-2 diag </summary>
 
 ```
cumulus@client-2:~$ ping -c 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=21.7 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=18.7 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=64 time=22.7 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=64 time=17.8 ms
64 bytes from 192.168.10.1: icmp_seq=5 ttl=64 time=18.6 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 17.885/19.953/22.735/1.902 ms

cumulus@client-2:~$ ping -c 5 192.168.11.1
PING 192.168.11.1 (192.168.11.1) 56(84) bytes of data.
64 bytes from 192.168.11.1: icmp_seq=1 ttl=64 time=17.9 ms
64 bytes from 192.168.11.1: icmp_seq=2 ttl=64 time=22.2 ms
64 bytes from 192.168.11.1: icmp_seq=3 ttl=64 time=21.1 ms
64 bytes from 192.168.11.1: icmp_seq=4 ttl=64 time=23.9 ms
64 bytes from 192.168.11.1: icmp_seq=5 ttl=64 time=22.5 ms

--- 192.168.11.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 17.969/21.573/23.957/2.013 ms

cumulus@client-2:~$ ping -c 5 192.168.12.1
PING 192.168.12.1 (192.168.12.1) 56(84) bytes of data.
64 bytes from 192.168.12.1: icmp_seq=1 ttl=62 time=21.8 ms
64 bytes from 192.168.12.1: icmp_seq=2 ttl=62 time=20.6 ms
64 bytes from 192.168.12.1: icmp_seq=3 ttl=62 time=20.3 ms
64 bytes from 192.168.12.1: icmp_seq=4 ttl=62 time=19.6 ms
64 bytes from 192.168.12.1: icmp_seq=5 ttl=62 time=23.7 ms

--- 192.168.12.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 19.604/21.219/23.733/1.453 ms

cumulus@client-2:~$ ping -c 5 192.168.13.1
PING 192.168.13.1 (192.168.13.1) 56(84) bytes of data.
64 bytes from 192.168.13.1: icmp_seq=1 ttl=62 time=21.6 ms
64 bytes from 192.168.13.1: icmp_seq=2 ttl=62 time=19.8 ms
64 bytes from 192.168.13.1: icmp_seq=3 ttl=62 time=22.2 ms
64 bytes from 192.168.13.1: icmp_seq=4 ttl=62 time=20.6 ms
64 bytes from 192.168.13.1: icmp_seq=5 ttl=62 time=17.5 ms

--- 192.168.13.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 17.548/20.375/22.234/1.638 ms
```
</details>

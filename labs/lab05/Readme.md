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
<details>
<summary> Leaf-1 </summary>
<br>
Leaf-1#sh run<br>
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
   name Test<br>
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
   switchport access vlan 10<br>
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
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 10 vni 1010<br>
   vxlan learn-restrict any<br>
!<br>
ip routing<br>
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
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   address-family ipv4<br>
      network 10.2.0.1/32<br>
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
Leaf-2#sh run<br>
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
   name Test<br>
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
   switchport access vlan 10<br>
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
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 10 vni 1010<br>
   vxlan learn-restrict any<br>
!<br>
ip routing<br>
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
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   address-family ipv4<br>
      network 10.2.0.2/32<br>
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
Leaf-3#sh run<br>
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
vlan 10<br>
   name Test<br>
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
interface Vxlan1<br>
   vxlan source-interface Loopback2<br>
   vxlan udp-port 4789<br>
   vxlan vlan 10 vni 1010<br>
   vxlan learn-restrict any<br>
!<br>
ip routing<br>
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
   vlan 10<br>
      rd 65003:1010<br>
      route-target both 10:1010<br>
      redistribute learned<br>
   !<br>
   address-family evpn<br>
      neighbor evpn activate<br>
   !<br>
   address-family ipv4<br>
      network 10.2.0.3/32<br>
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
</details>
#### Диагностика Spine/Leaf

<details>
<summary> Spine-1 diag </summary>
 
 ```
Spine-1#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.2.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.0.1 4 65001             42        43    0    0 00:23:47 Estab   2      2
  10.2.0.2 4 65002             40        39    0    0 00:19:48 Estab   2      2
  10.2.0.3 4 65003             40        36    0    0 00:19:37 Estab   3      3

Spine-1#show bgp evpn 
BGP routing table information for VRF default
Router identifier 10.2.1.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65001 i
 * >      RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65002 i
 * >      RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65003 i
 * >      RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65003 i
 * >      RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65001 i
 * >      RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65002 i
 * >      RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65003 i
```
</details>
<details>
<summary> Spine-2 diag </summary>

 ```
Spine-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.2.2.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.0.1 4 65001             50        47    0    0 00:26:07 Estab   2      2
  10.2.0.2 4 65002             45        43    0    0 00:22:07 Estab   2      2
  10.2.0.3 4 65003             44        39    0    0 00:22:04 Estab   3      3

Spine-2#show bgp evpn 
BGP routing table information for VRF default
Router identifier 10.2.2.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65001 i
 * >      RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65002 i
 * >      RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65003 i
 * >      RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65003 i
 * >      RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65001 i
 * >      RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65002 i
 * >      RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65003 i
```
</details>
<details>
<summary> Leaf-1 diag </summary>

 ```
Leaf-1# show bgp evpn summary 
BGP summary information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65000             48        47    0    0 00:28:03 Estab   5      5
  10.2.2.0 4 65000             48        53    0    0 00:28:03 Estab   5      5
  
Leaf-1# show bgp evpn 
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:1010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec    RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65000 65003 i
 * >Ec    RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65000 65003 i
 * >      RD: 65001:1010 imet 10.2.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 * >Ec    RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65000 65003 i  
  
Leaf-1# show interfaces vxlan 1 
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 10.2.0.1
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is 
    [10, 1010]       
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.2.0.3        10.2.0.2       
  Shared Router MAC is 0000.0000.0000

Leaf-1#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Et3        1       0:00:22 ago
  10    0050.7966.6807    DYNAMIC     Vx1        1       0:00:07 ago
  10    0050.7966.6808    DYNAMIC     Vx1        1       0:00:14 ago
  10    0050.7966.6809    DYNAMIC     Vx1        1       0:00:22 ago
Total Mac Addresses for this criterion: 4
```
</details>
<summary> Leaf-2 diag </summary>

 ```
Leaf-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65000             58        60    0    0 00:30:46 Estab   5      5
  10.2.2.0 4 65000             58        61    0    0 00:30:46 Estab   5      5

Leaf-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 65002:1010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >Ec    RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 mac-ip 0050.7966.6808
                                 10.2.0.3              -       100     0       65000 65003 i
 * >Ec    RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 mac-ip 0050.7966.6809
                                 10.2.0.3              -       100     0       65000 65003 i
 * >Ec    RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >      RD: 65002:1010 imet 10.2.0.2
                                 -                     -       -       0       i
 * >Ec    RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65000 65003 i
 *  ec    RD: 65003:1010 imet 10.2.0.3
                                 10.2.0.3              -       100     0       65000 65003 i

Leaf-2#show interfaces vxlan 1 
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 10.2.0.2
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is 
    [10, 1010]       
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.2.0.1        10.2.0.3       
  Shared Router MAC is 0000.0000.0000

Leaf-2#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Vx1        1       0:01:50 ago
  10    0050.7966.6807    DYNAMIC     Et3        1       0:01:35 ago
  10    0050.7966.6808    DYNAMIC     Vx1        1       0:01:42 ago
  10    0050.7966.6809    DYNAMIC     Vx1        1       0:01:50 ago
Total Mac Addresses for this criterion: 4
```
</details>
<summary> Leaf-3 diag </summary>

 ```
Leaf-3# show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65000             54        62    0    0 00:32:43 Estab   4      4
  10.2.2.0 4 65000             54        63    0    0 00:32:51 Estab   4      4

Leaf-3#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 mac-ip 0050.7966.6806
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 mac-ip 0050.7966.6807
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 65003:1010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65003:1010 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >Ec    RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 *  ec    RD: 65001:1010 imet 10.2.0.1
                                 10.2.0.1              -       100     0       65000 65001 i
 * >Ec    RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 *  ec    RD: 65002:1010 imet 10.2.0.2
                                 10.2.0.2              -       100     0       65000 65002 i
 * >      RD: 65003:1010 imet 10.2.0.3
                                 -                     -       -       0       i

Leaf-3#show interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 10.2.0.3
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is 
    [10, 1010]       
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.2.0.1        10.2.0.2       
  Shared Router MAC is 0000.0000.0000

Leaf-3#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Vx1        1       0:04:15 ago
  10    0050.7966.6807    DYNAMIC     Vx1        1       0:03:59 ago
  10    0050.7966.6808    DYNAMIC     Et3        1       0:04:07 ago
  10    0050.7966.6809    DYNAMIC     Et4        1       0:04:15 ago
Total Mac Addresses for this criterion: 4
```
</details>

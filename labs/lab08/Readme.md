### VxLAN. Routing.

### Цели:
- 1: Реализовать передачу суммарных префиксов через EVPN route-type 5.



### Собрана топология:
![image](main_topology_lab08.png)

### Особенности настройки:
Настройки underlay OSPF и overlay eBGP.<br>
На Leaf-4 настроены BGP сессии в address-family ipv4 в vrf vrf-blue и vrf-red с Router-1.<br>
Настроена опция allowas-in 2 на Leaf-4, allowas-in 1 на Spine-1, Spine-2.<br>
На Router-1 настроены суммарные маршруты и анонс только суммарных префиксов:<br>
aggregate-address 192.168.30.0/23 summary-only<br>
aggregate-address 192.168.50.0/23 summary-only<br>


### IP план:

<details>

Device|Interface|IP Address|Subnet Mask|Default GW
---|---|---|---|---
Spine-1|Lo1|10.0.1.0|255.255.255.255
-|Lo2|10.2.1.0|255.255.255.255
-|Eth1|10.4.1.0|255.255.255.254
-|Eth2|10.4.1.2|255.255.255.254
-|Eth3|10.4.1.4|255.255.255.254
-|Eth4|10.4.1.6|255.255.255.254
Spine-2|Lo1|10.0.2.0|255.255.255.255
-|Lo2|10.2.2.0|255.255.255.255
-|Eth1|10.4.2.0|255.255.255.254
-|Eth2|10.4.2.2|255.255.255.254
-|Eth3|10.4.2.4|255.255.255.254
-|Eth4|10.4.2.6|255.255.255.254
Leaf-1|Lo1|10.0.0.1|255.255.255.255
-|Lo2|10.2.0.1|255.255.255.255
-|Eth1|10.4.1.1|255.255.255.254
-|Eth2|10.4.2.1|255.255.255.254
-|vlan10|192.168.30.254|255.255.255.0
Leaf-2|Lo1|10.0.0.2|255.255.255.255
-|Lo2|10.2.0.2|255.255.255.255
-|Eth1|10.4.1.3|255.255.255.254
-|Eth2|10.4.2.3|255.255.255.254
-|vlan11|192.168.50.254|255.255.255.0
Leaf-3|Lo1|10.0.0.3|255.255.255.255
-|Lo2|10.2.0.3|255.255.255.255
-|Eth1|10.4.1.5|255.255.255.254
-|Eth2|10.4.2.5|255.255.255.254
-|vlan100|192.168.31.254|255.255.255.0
Leaf-4|Lo1|10.0.0.4|255.255.255.255
-|Lo2|10.2.0.4|255.255.255.255
-|Eth1|10.4.1.7|255.255.255.254
-|Eth2|10.4.2.7|255.255.255.254
-|Eth3.1000|10.4.3.1|255.255.255.254
-|Eth3.2000|10.4.3.3|255.255.255.254
-|vlan101|192.168.51.254|255.255.255.0
VPC1-Blue|Eth0|192.168.30.1|255.255.255.0|192.168.30.254
VPC2-Red|Eth0|192.168.50.1|255.255.255.0|192.168.50.254
VPC3-Blue|Eth0|192.168.31.1|255.255.255.0|192.168.31.254
VPC4-Red|Eth0|192.168.51.1|255.255.255.0|192.168.51.254

</details>

#### Конфигурация на оборудовании Arista.

<details>
<summary> Spine-1 </summary>
 
 ```
Spine-1#show running-config 
! Command: show running-config
! device: Spine-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description Leaf-1 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Leaf-2 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Leaf-3 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   description Leaf-4 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.6/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   description Underlay
   ip address 10.0.1.0/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.1.0/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router bgp 65000
   neighbor evpn peer group
   neighbor evpn next-hop-unchanged
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.0.1 peer group evpn
   neighbor 10.2.0.1 remote-as 65001
   neighbor 10.2.0.2 peer group evpn
   neighbor 10.2.0.2 remote-as 65002
   neighbor 10.2.0.3 peer group evpn
   neighbor 10.2.0.3 remote-as 65003
   neighbor 10.2.0.4 peer group evpn
   neighbor 10.2.0.4 remote-as 65004
   neighbor 10.2.0.4 allowas-in 1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
!
router ospf 1
   router-id 10.0.1.0
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Spine-2 </summary>
 
 ```
Spine-2#show running-config 
! Command: show running-config
! device: Spine-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description Leaf-1 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Leaf-2 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Leaf-3 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   description Leaf-4 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.6/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   description Underlay
   ip address 10.0.2.0/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.2.0/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router bgp 65000
   neighbor evpn peer group
   neighbor evpn next-hop-unchanged
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.0.1 peer group evpn
   neighbor 10.2.0.1 remote-as 65001
   neighbor 10.2.0.2 peer group evpn
   neighbor 10.2.0.2 remote-as 65002
   neighbor 10.2.0.3 peer group evpn
   neighbor 10.2.0.3 remote-as 65003
   neighbor 10.2.0.4 peer group evpn
   neighbor 10.2.0.4 remote-as 65004
   neighbor 10.2.0.4 allowas-in 1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
!
router ospf 1
   router-id 10.0.2.0
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Leaf-1 </summary>
 
 ```
Leaf-1#show running-config 
! Command: show running-config
! device: Leaf-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf-1
!
spanning-tree mode mstp
!
vlan 10
   name data1
!
vrf instance vrf-blue
!
interface Ethernet1
   description Spine-1 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Spine-2 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description VPC1-Blue
   switchport access vlan 10
!
interface Loopback1
   description Underlay
   ip address 10.0.0.1/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.0.1/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   vrf vrf-blue
   ip address 192.168.30.254/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vrf vrf-blue vni 50000
   vxlan learn-restrict any
!
ip routing
ip routing vrf vrf-blue
!
router bgp 65001
   neighbor evpn peer group
   neighbor evpn remote-as 65000
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.2.0 peer group evpn
   !
   vlan 10
      rd 65001:1010
      route-target both 10:1010
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
   !
   vrf vrf-blue
      rd 10.2.0.1:1
      route-target import evpn 1:50000
      route-target export evpn 1:50000
      redistribute connected
!
router ospf 1
   router-id 10.0.0.1
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Leaf-2 </summary>
 
 ```
Leaf-2#show running-config 
! Command: show running-config
! device: Leaf-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf-2
!
spanning-tree mode mstp
!
vlan 11
   name data2
!
vrf instance vrf-red
!
interface Ethernet1
   description Spine-1 | Eth2
   mtu 9214
   no switchport
   ip address 10.4.1.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Spine-2 | Eth2
   mtu 9214
   no switchport
   ip address 10.4.2.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description VPC2-Red
   switchport access vlan 11
!
interface Loopback1
   description Underlay
   ip address 10.0.0.2/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.0.2/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan11
   vrf vrf-red
   ip address 192.168.50.254/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 11 vni 1011
   vxlan vrf vrf-red vni 60000
   vxlan learn-restrict any
!
ip routing
ip routing vrf vrf-red
!
router bgp 65002
   neighbor evpn peer group
   neighbor evpn remote-as 65000
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.2.0 peer group evpn
   !
   vlan 11
      rd 65002:1011
      route-target both 11:1011
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
   !
   vrf vrf-red
      rd 10.2.0.2:1
      route-target import evpn 1:60000
      route-target export evpn 1:60000
      redistribute connected
!
router ospf 1
   router-id 10.0.0.2
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Leaf-3 </summary>
 
 ```
Leaf-3#show running-config 
! Command: show running-config
! device: Leaf-3 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf-3
!
spanning-tree mode mstp
!
vlan 100
   name data3
!
vrf instance vrf-blue
!
interface Ethernet1
   description Spine-1 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Spine-2 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description VPC3-Blue
   switchport access vlan 100
!
interface Loopback1
   description Underlay
   ip address 10.0.0.3/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.0.3/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan100
   vrf vrf-blue
   ip address 192.168.31.254/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 10100
   vxlan vrf vrf-blue vni 50000
   vxlan learn-restrict any
!
ip routing
ip routing vrf vrf-blue
!
router bgp 65003
   neighbor evpn peer group
   neighbor evpn remote-as 65000
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.2.0 peer group evpn
   !
   vlan 100
      rd 65003:10100
      route-target both 100:10100
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
   !
   vrf vrf-blue
      rd 10.2.0.3:1
      route-target import evpn 1:50000
      route-target export evpn 1:50000
      redistribute connected
!
router ospf 1
   router-id 10.0.0.3
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Leaf-4 </summary>
 
 ```
Leaf-4#show running-config 
! Command: show running-config
! device: Leaf-4 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf-4
!
spanning-tree mode mstp
!
vlan 101
   name data4
!
vrf instance vrf-blue
!
vrf instance vrf-red
!
interface Ethernet1
   description Spine-1 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.1.7/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Spine-2 | Eth1
   mtu 9214
   no switchport
   ip address 10.4.2.7/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Router-1 | Eth1
   mtu 9214
   no switchport
!
interface Ethernet3.1000
   encapsulation dot1q vlan 1000
   vrf vrf-red
   ip address 10.4.3.1/31
!
interface Ethernet3.2000
   encapsulation dot1q vlan 2000
   vrf vrf-blue
   ip address 10.4.3.3/31
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description VPC4-Red
   switchport access vlan 101
!
interface Loopback1
   description Underlay
   ip address 10.0.0.4/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   description Overlay
   ip address 10.2.0.4/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan101
   vrf vrf-red
   ip address 192.168.51.254/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 101 vni 10101
   vxlan vrf vrf-blue vni 50000
   vxlan vrf vrf-red vni 60000
   vxlan learn-restrict any
!
ip routing
ip routing vrf vrf-blue
ip routing vrf vrf-red
!
router bgp 65004
   neighbor evpn peer group
   neighbor evpn remote-as 65000
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.2.0 peer group evpn
   !
   vlan 101
      rd 65004:10101
      route-target both 101:10101
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor 10.4.3.0 activate
      neighbor 10.4.3.2 activate
   !
   vrf vrf-blue
      rd 10.2.0.4:2
      route-target import evpn 1:50000
      route-target export evpn 1:50000
      neighbor 10.4.3.2 remote-as 65100
      neighbor 10.4.3.2 allowas-in 2
      redistribute connected
   !
   vrf vrf-red
      rd 10.2.0.4:1
      route-target import evpn 1:60000
      route-target export evpn 1:60000
      neighbor 10.4.3.0 remote-as 65100
      neighbor 10.4.3.0 allowas-in 2
      redistribute connected
!
router ospf 1
   router-id 10.0.0.4
   auto-cost reference-bandwidth 10000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end
```
</details>

<details>
<summary> Router-1 </summary>
 
 ```
Router-1#sh run
! Command: show running-config
! device: Router-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Router-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description Leaf-4 | Eth3
   mtu 9214
   no switchport
!
interface Ethernet1.1000
   encapsulation dot1q vlan 1000
   ip address 10.4.3.0/31
!
interface Ethernet1.2000
   encapsulation dot1q vlan 2000
   ip address 10.4.3.2/31
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback2
   ip address 10.2.0.254/32
!
interface Management1
!
ip routing
!
router bgp 65100
   router-id 10.2.0.254
   neighbor fabric peer group
   neighbor fabric remote-as 65004
   neighbor 10.4.3.1 peer group fabric
   neighbor 10.4.3.3 peer group fabric
   aggregate-address 192.168.30.0/23 summary-only
   aggregate-address 192.168.50.0/23 summary-only
   !
   address-family ipv4
      neighbor fabric activate
!
end
Router-1#show running-config 
! Command: show running-config
! device: Router-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Router-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description Leaf-4 | Eth3
   mtu 9214
   no switchport
!
interface Ethernet1.1000
   encapsulation dot1q vlan 1000
   ip address 10.4.3.0/31
!
interface Ethernet1.2000
   encapsulation dot1q vlan 2000
   ip address 10.4.3.2/31
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback2
   ip address 10.2.0.254/32
!
interface Management1
!
ip routing
!
router bgp 65100
   router-id 10.2.0.254
   neighbor fabric peer group
   neighbor fabric remote-as 65004
   neighbor 10.4.3.1 peer group fabric
   neighbor 10.4.3.3 peer group fabric
   aggregate-address 192.168.30.0/23 summary-only
   aggregate-address 192.168.50.0/23 summary-only
   !
   address-family ipv4
      neighbor fabric activate
!
end
```
</details>

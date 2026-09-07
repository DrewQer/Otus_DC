# Lab02.Построение Underlay сети (OSPF)
> «Выбирая кратчайший путь, убедись, что все участники движения знают карту дорог.»

## 1. Цель работы
1. Настроить OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами;
2. Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств;
3. Убедить в наличии IP связанности между устройствами в OSFP домене.

## 2. Топология
![Scheme_C1](C1.png)

| Device     | Hostname  | Platform        | Interface Loopback | IP network /32 |
|------------|-----------|-----------------| -------------------|----------------|
| Spine1     | S1        | Arista vEOS-lab | Loopback 0         | 192.168.0.1    |
| Spine1     | S1        | Arista vEOS-lab | Loopback 1         | 192.168.100.1  |
| Spine2     | S2        | Arista vEOS-lab | Loopback 0         | 192.168.0.2    |
| Spine2     | S2        | Arista vEOS-lab | Loopback 1         | 192.168.100.2  |
| Leaf1      | L1        | Arista vEOS-lab | Loopback 0         | 192.168.0.11   |
| Leaf1      | L1        | Arista vEOS-lab | Loopback 1         | 192.168.100.11 |
| Leaf2      | L2        | Arista vEOS-lab | Loopback 0         | 192.168.0.12   |
| Leaf2      | L2        | Arista vEOS-lab | Loopback 1         | 192.168.100.12 |
| Leaf3      | L3        | Arista vEOS-lab | Loopback 0         | 192.168.0.13   |
| Leaf3      | L3        | Arista vEOS-lab | Loopback 1         | 192.168.100.13 |

## 3. Адресный план (Underlay)
![Scheme_C2](C2.png)

| Device1    | Interface1   | Device1 IP | Network /30 | Device2 IP | Interface2   | Device2    |
|------------|--------------|------------|-------------|------------|--------------|------------|
| Leaf1      | Ethernet1    |  10.1.1.2  |  10.1.1.0   |  10.1.1.1  | Ethernet1    | Spine1     |
| Leaf1      | Ethernet2    |  10.1.4.2  |  10.1.4.0   |  10.1.4.1  | Ethernet1    | Spine2     |
| Leaf2      | Ethernet1    |  10.1.2.2  |  10.1.2.0   |  10.1.2.1  | Ethernet2    | Spine1     |
| Leaf2      | Ethernet2    |  10.1.5.2  |  10.1.5.0   |  10.1.5.1  | Ethernet2    | Spine2     |
| Leaf3      | Ethernet1    |  10.1.3.2  |  10.1.3.0   |  10.1.3.1  | Ethernet3    | Spine1     |
| Leaf3      | Ethernet2    |  10.1.6.2  |  10.1.6.0   |  10.1.6.1  | Ethernet3    | Spine2     |

## 4. Конфигурация 

<details>
<summary> Spine1
</summary>

```
#configure terminal

hostname S1
!
interface Loopback0
   ip address 192.168.0.1/32
   ip ospf area 0.0.0.0
   exit
!
interface Ethernet1
   description P2P_to_Leaf1_Eth1
   no switchport
   mtu 9214
   ip address 10.1.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description P2P_to_Leaf2_Eth1
   no switchport
   mtu 9214
   ip address 10.1.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description P2P_to_Leaf3_Eth1
   no switchport
   mtu 9214
   ip address 10.1.3.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
router ospf 1
   router-id 192.168.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
   bfd all-interfaces
   log-adjacency-changes detail
   maximum-paths 4
   exit
```
</details>

<details>
<summary> Spine2
</summary>

```
#configure terminal

hostname S2
!
interface Loopback0
   ip address 192.168.0.2/32
   ip ospf area 0.0.0.0
   exit
!
interface Ethernet1
   description P2P_to_Leaf1_Eth2
   no switchport
   mtu 9214
   ip address 10.1.4.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description P2P_to_Leaf2_Eth2
   no switchport
   mtu 9214
   ip address 10.1.5.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description P2P_to_Leaf3_Eth2
   no switchport
   mtu 9214
   ip address 10.1.6.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
router ospf 1
   router-id 192.168.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
   bfd all-interfaces
   log-adjacency-changes detail
   maximum-paths 4
   exit
```
</details>

<details>
<summary> Leaf1
</summary>

```
#configure terminal

hostname L1
!
interface Loopback0
   ip address 192.168.0.11/32
   ip ospf area 0.0.0.0
   exit
!
interface Ethernet1
   description P2P_to_Spine1_Eth1
   no switchport
   mtu 9214
   ip address 10.1.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description P2P_to_Spine2_Eth1
   no switchport
   mtu 9214
   ip address 10.1.4.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet8
   description To_Client1
   mtu 9214
   exit
!
router ospf 1
   router-id 192.168.0.11
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
   bfd all-interfaces
   log-adjacency-changes detail
   maximum-paths 4
   exit
```
</details>

<details>
<summary> Leaf2
</summary>

```
#configure terminal

hostname L12
!
interface Loopback0
   ip address 192.168.0.12/32
   ip ospf area 0.0.0.0
   exit
!
interface Ethernet1
   description P2P_to_Spine1_Eth2
   no switchport
   mtu 9214
   ip address 10.1.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description P2P_to_Spine2_Eth2
   no switchport
   mtu 9214
   ip address 10.1.5.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet8
   description To_Client2
   mtu 9214
   exit
!
router ospf 1
   router-id 192.168.0.12
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
   bfd all-interfaces
   log-adjacency-changes detail
   maximum-paths 4
   exit
```
</details>

<details>
<summary> Leaf3
</summary>

```
#configure terminal

hostname L3
!
interface Loopback0
   ip address 192.168.0.13/32
   ip ospf area 0.0.0.0
   exit
!
interface Ethernet1
   description P2P_to_Spine1_Eth3
   no switchport
   mtu 9214
   ip address 10.1.3.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description P2P_to_Spine2_Eth3
   no switchport
   mtu 9214
   ip address 10.1.6.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 OTUS
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet7
   description To_Client3.1
   mtu 9214
   exit
!
interface Ethernet8
   description To_Client3.2
   mtu 9214
   exit
!
router ospf 1
   router-id 192.168.0.13
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
   bfd all-interfaces
   log-adjacency-changes detail
   maximum-paths 4
   exit
```
</details>

<details>
<summary> Monitoring/Debug
</summary>

```
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip ospf database router
show bfd peers

debug ip ospf events
debug ip ospf adjacency
```
</details>

## 5. Проверка работоспособности топологии

<details>
<summary> Проверка работы протоколов
</summary>

```

```
</details>


<details>
<summary> Проверка доступности L1 <--> L2
</summary>

```

```
</details>

<details>
<summary> Проверка доступности L1 <--> L3
</summary>

```

```
</details>

<details>
<summary> Проверка доступности L2 <--> L3
</summary>

```

```
</details>

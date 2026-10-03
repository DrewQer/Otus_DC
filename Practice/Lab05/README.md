# Lab03.Построение Underlay сети (eBGP)
> «eBGP — единственный протокол, где фраза “Я тебе не верю, покажи документы” является базовой настройкой по умолчанию».»

## 1. Цель работы
1. Настроите BGP в Underlay сети (eBGP), для IP связанности между всеми сетевыми устройствами;
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств;
3. Убедитесь в наличии IP связанности между устройствами в BGP домене.

## 2. Топология
![Scheme_C1](C1.png)

### IPv4
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

### IPv6
| Device     | Hostname  | Platform        | Interface Loopback | IP network /128       |
|------------|-----------|-----------------| -------------------|-----------------------|
| Spine1     | S1        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::1/128   |
| Spine1     | S1        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::1/128   |
| Spine2     | S2        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::2/128   |
| Spine2     | S2        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::2/128   |
| Leaf1      | L1        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::11/128  |
| Leaf1      | L1        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::11/128  |
| Leaf2      | L2        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::12/128  |
| Leaf2      | L2        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::12/128  |
| Leaf3      | L3        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::13/128  |
| Leaf3      | L3        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::13/128  |

## 3. Адресный план (Underlay)
![Scheme_C2](C2_eBGP.png)

### IPv4 
| Device1    | Interface1   | Device1 IP | Network /30 | Device2 IP | Interface2   | Device2    |
|------------|--------------|------------|-------------|------------|--------------|------------|
| Leaf1      | Ethernet1    |  10.1.1.2  |  10.1.1.0   |  10.1.1.1  | Ethernet1    | Spine1     |
| Leaf1      | Ethernet2    |  10.1.4.2  |  10.1.4.0   |  10.1.4.1  | Ethernet1    | Spine2     |
| Leaf2      | Ethernet1    |  10.1.2.2  |  10.1.2.0   |  10.1.2.1  | Ethernet2    | Spine1     |
| Leaf2      | Ethernet2    |  10.1.5.2  |  10.1.5.0   |  10.1.5.1  | Ethernet2    | Spine2     |
| Leaf3      | Ethernet1    |  10.1.3.2  |  10.1.3.0   |  10.1.3.1  | Ethernet3    | Spine1     |
| Leaf3      | Ethernet2    |  10.1.6.2  |  10.1.6.0   |  10.1.6.1  | Ethernet3    | Spine2     |

### IPv6
| Device1    | Interface1   | Device1 IP      | Network /127   | Device2 IP      | Interface2   | Device2    |
|------------|--------------|-----------------|--------------- |-----------------|--------------|------------|
| Leaf1      | Ethernet1    | 2001:db8:1:1::2 | 2001:db8:1:1:: | 2001:db8:1:1::1 | Ethernet1    | Spine1     |
| Leaf1      | Ethernet2    | 2001:db8:1:4::2 | 2001:db8:1:4:: | 2001:db8:1:4::1 | Ethernet1    | Spine2     |
| Leaf2      | Ethernet1    | 2001:db8:1:2::2 | 2001:db8:1:2:: | 2001:db8:1:2::1 | Ethernet2    | Spine1     |
| Leaf2      | Ethernet2    | 2001:db8:1:5::2 | 2001:db8:1:5:: | 2001:db8:1:5::1 | Ethernet2    | Spine2     |
| Leaf3      | Ethernet1    | 2001:db8:1:3::2 | 2001:db8:1:3:: | 2001:db8:1:3::1 | Ethernet3    | Spine1     |
| Leaf3      | Ethernet2    | 2001:db8:1:6::2 | 2001:db8:1:6:: | 2001:db8:1:6::1 | Ethernet3    | Spine2     |

## 4. Конфигурация 

| Device1    | ASN       |
|------------|-----------|
| Spine1     | 65001     |
| Spine2     | 65002     |
| Leaf1      | 65011     |
| Leaf2      | 65012     |
| Leaf3      | 65013     |

<details>
<summary> Spine1
</summary>

```
!
service routing protocols model multi-agent
!
ipv6 unicast-routing
!
hostname Spine1
!
interface Loopback0
   ip address 192.168.0.1/32
   ipv6 address 2001:db8:100::1/128
   exit
!
interface Loopback1
   ip address 192.168.100.1/32
   ipv6 address 2001:db8:200::1/128
   exit
!
interface Ethernet1
   description eBGP_to_Leaf1_Eth1
   no switchport
   mtu 9000
   ip address 10.1.1.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:1::1/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description eBGP_to_Leaf2_Eth1
   no switchport
   mtu 9000
   ip address 10.1.2.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:2::1/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description eBGP_to_Leaf3_Eth1
   no switchport
   mtu 9000
   ip address 10.1.3.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:3::1/126
   bfd interval 300 min-rx 300 multiplier 3

   exit
!
ip prefix-list PL-LOOPBACKS-V4 seq 10 permit 192.168.0.0/24 le 32
ip prefix-list PL-LOOPBACKS-V4 seq 20 permit 192.168.100.0/24 le 32
!
ipv6 prefix-list PL-LOOPBACKS-V6
   seq 10 permit 2001:db8:100::/48 le 128
   seq 20 permit 2001:db8:200::/48 le 128
   exit
!
route-map UNDERLAY-IN permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-OUT permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-IN6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
route-map UNDERLAY-OUT6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
peer-filter LEAF-AS-RANGE
   10 match as-range 65011-65013 result accept
   20 match as-range 1-4294967295 result reject
   exit
!
router bgp 65001
   router-id 192.168.0.1
   no bgp default ipv4-unicast
   bgp bestpath as-path multipath-relax
   maximum-paths 4 ecmp 4
   distance bgp 20 20 20
   !
   bgp listen range 10.1.0.0/16 peer-group UNDERLAY-V4 peer-filter LEAF-AS-RANGE
   bgp listen range 2001:db8:1::/48 peer-group UNDERLAY-V6 peer-filter LEAF-AS-RANGE
   !
   neighbor UNDERLAY-V4 peer group
   neighbor UNDERLAY-V4 bfd
   neighbor UNDERLAY-V4 password OTUS
   neighbor UNDERLAY-V4 ttl maximum-hops 1
   neighbor UNDERLAY-V4 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V4 route-map UNDERLAY-IN in
   neighbor UNDERLAY-V4 route-map UNDERLAY-OUT out
   !
   neighbor UNDERLAY-V6 peer group
   neighbor UNDERLAY-V6 bfd
   neighbor UNDERLAY-V6 password OTUS
   neighbor UNDERLAY-V6 ttl maximum-hops 1
   neighbor UNDERLAY-V6 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V6 route-map UNDERLAY-IN6 in
   neighbor UNDERLAY-V6 route-map UNDERLAY-OUT6 out
   !
   address-family ipv4
      neighbor UNDERLAY-V4 activate
      network 192.168.0.1/32
      network 192.168.100.1/32
      exit
   !
   address-family ipv6
      neighbor UNDERLAY-V6 activate
      network 2001:db8:100::1/128
      network 2001:db8:200::1/128
      exit
    exit
```

</details>

[Spine1 Running-conifg ](_Spine1_running-config.txt)

<details>
<summary> Spine2
</summary>

```
!
service routing protocols model multi-agent
!
ipv6 unicast-routing
!
hostname Spine2
!
interface Loopback0
   ip address 192.168.0.2/32
   ipv6 address 2001:db8:100::2/128
   exit
!
interface Loopback1
   ip address 192.168.100.2/32
   ipv6 address 2001:db8:200::2/128
   exit
!
interface Ethernet1
   description eBGP_to_Leaf1_Eth2
   no switchport
   mtu 9000
   ip address 10.1.4.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:4::1/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description eBGP_to_Leaf2_Eth2
   no switchport
   mtu 9000
   ip address 10.1.5.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:5::1/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description eBGP_to_Leaf3_Eth2
   no switchport
   mtu 9000
   ip address 10.1.6.1/30
   ipv6 enable
   ipv6 address 2001:db8:1:6::1/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
ip prefix-list PL-LOOPBACKS-V4 seq 10 permit 192.168.0.0/24 le 32
ip prefix-list PL-LOOPBACKS-V4 seq 20 permit 192.168.100.0/24 le 32
!
ipv6 prefix-list PL-LOOPBACKS-V6
   seq 10 permit 2001:db8:100::/48 le 128
   seq 20 permit 2001:db8:200::/48 le 128
   exit
!
route-map UNDERLAY-IN permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-OUT permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
!
route-map UNDERLAY-IN6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
route-map UNDERLAY-OUT6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
peer-filter LEAF-AS-RANGE
   10 match as-range 65011-65013 result accept
   20 match as-range 1-4294967295 result reject
   exit
!
router bgp 65002
   router-id 192.168.0.2
   no bgp default ipv4-unicast
   bgp bestpath as-path multipath-relax
   maximum-paths 4 ecmp 4
   distance bgp 20 20 20
   !
   bgp listen range 10.1.0.0/16 peer-group UNDERLAY-V4 peer-filter LEAF-AS-RANGE
   bgp listen range 2001:db8:1::/48 peer-group UNDERLAY-V6 peer-filter LEAF-AS-RANGE
   !
   neighbor UNDERLAY-V4 peer group
   neighbor UNDERLAY-V4 bfd
   neighbor UNDERLAY-V4 password OTUS
   neighbor UNDERLAY-V4 ttl maximum-hops 1
   neighbor UNDERLAY-V4 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V4 route-map UNDERLAY-IN in
   neighbor UNDERLAY-V4 route-map UNDERLAY-OUT out
   !
   neighbor UNDERLAY-V6 peer group
   neighbor UNDERLAY-V6 bfd
   neighbor UNDERLAY-V6 password OTUS
   neighbor UNDERLAY-V6 ttl maximum-hops 1
   neighbor UNDERLAY-V6 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V6 route-map UNDERLAY-IN6 in
   neighbor UNDERLAY-V6 route-map UNDERLAY-OUT6 out
   !
   address-family ipv4
      neighbor UNDERLAY-V4 activate
      network 192.168.0.2/32
      network 192.168.100.2/32
      exit
   !
   address-family ipv6
      neighbor UNDERLAY-V6 activate
      network 2001:db8:100::2/128
      network 2001:db8:200::2/128
      exit
    exit
```

</details>

[Spine2 Running-conifg ](_Spine2_running-config.txt)

<details>
<summary> Leaf1
</summary>

```
!
service routing protocols model multi-agent
!
ipv6 unicast-routing
!
hostname Leaf1
!
interface Loopback0
   ip address 192.168.0.11/32
   ipv6 address 2001:db8:100::11/128
   exit
!
interface Loopback1
   ip address 192.168.100.11/32
   ipv6 address 2001:db8:200::11/128
   exit
!
interface Ethernet1
   description eBGP_to_Spine1_Eth1
   no switchport
   mtu 9000
   ip address 10.1.1.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:1::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description eBGP_to_Spine2_Eth1
   no switchport
   mtu 9000
   ip address 10.1.4.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:4::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description To_Client1
   no switchport
   exit
!
ip prefix-list PL-LOOPBACKS-V4 seq 10 permit 192.168.0.0/24 le 32
ip prefix-list PL-LOOPBACKS-V4 seq 20 permit 192.168.100.0/24 le 32
!
ipv6 prefix-list PL-LOOPBACKS-V6
   seq 10 permit 2001:db8:100::/48 le 128
   seq 20 permit 2001:db8:200::/48 le 128
   exit
!
route-map UNDERLAY-IN permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-OUT permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-IN6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
route-map UNDERLAY-OUT6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
router bgp 65011
   router-id 192.168.0.11
   no bgp default ipv4-unicast
   bgp bestpath as-path multipath-relax
   maximum-paths 4 ecmp 4
   distance bgp 20 20 20
   !
   neighbor UNDERLAY-V4 peer-group
   neighbor UNDERLAY-V4 bfd
   neighbor UNDERLAY-V4 password OTUS
   neighbor UNDERLAY-V4 ttl maximum-hops 1
   neighbor UNDERLAY-V4 smaximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V4 route-map UNDERLAY-IN in
   neighbor UNDERLAY-V4 route-map UNDERLAY-OUT out
   !
   neighbor UNDERLAY-V6 peer-group
   neighbor UNDERLAY-V6 bfd
   neighbor UNDERLAY-V6 password OTUS
   neighbor UNDERLAY-V6 ttl maximum-hops 1
   neighbor UNDERLAY-V6 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V6 route-map UNDERLAY-IN6 in
   neighbor UNDERLAY-V6 route-map UNDERLAY-OUT6 out
   !
   neighbor 10.1.1.1 peer group UNDERLAY-V4
   neighbor 10.1.1.1 remote-as 65001
   neighbor 10.1.4.1 peer group UNDERLAY-V4
   neighbor 10.1.4.1 remote-as 65002
   !
   neighbor 2001:db8:1:1::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:1::1 remote-as 65001
   neighbor 2001:db8:1:4::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:4::1 remote-as 65002
   !
   address-family ipv4
      neighbor UNDERLAY-V4 activate
      network 192.168.0.11/32
      network 192.168.100.11/32
      exit
   !
   address-family ipv6
      neighbor UNDERLAY-V6 activate
      network 2001:db8:100::11/128
      network 2001:db8:200::11/128
      exit
    exit
```

</details>

[Leaf1 Running-conifg ](_Leaf1_running-config.txt)

<details>
<summary> Leaf2
</summary>

```
!
service routing protocols model multi-agent
!
ipv6 unicast-routing
!
hostname Leaf2
!
interface Loopback0
   ip address 192.168.0.12/32
   ipv6 address 2001:db8:100::12/128
   exit
!
interface Loopback1
   ip address 192.168.100.12/32
   ipv6 address 2001:db8:200::12/128
   exit
!
interface Ethernet1
   description eBGP_to_Spine1_Eth2
   no switchport
   mtu 9000
   ip address 10.1.2.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:2::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description eBGP_to_Spine2_Eth2
   no switchport
   mtu 9000
   ip address 10.1.5.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:5::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet3
   description To_Client2
   no switchport
   exit
!
ip prefix-list PL-LOOPBACKS-V4 seq 10 permit 192.168.0.0/24 le 32
ip prefix-list PL-LOOPBACKS-V4 seq 20 permit 192.168.100.0/24 le 32
!
ipv6 prefix-list PL-LOOPBACKS-V6
   seq 10 permit 2001:db8:100::/48 le 128
   seq 20 permit 2001:db8:200::/48 le 128
   exit
!
route-map UNDERLAY-IN permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-OUT permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
   exit
!
route-map UNDERLAY-IN6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
route-map UNDERLAY-OUT6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
router bgp 65012
   router-id 192.168.0.12
   no bgp default ipv4-unicast
   bgp bestpath as-path multipath-relax
   maximum-paths 4 ecmp 4
   distance bgp 20 20 20
   !
   neighbor UNDERLAY-V4 peer group
   neighbor UNDERLAY-V4 bfd
   neighbor UNDERLAY-V4 password OTUS
   neighbor UNDERLAY-V4 ttl maximum-hops 1
   neighbor UNDERLAY-V4 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V4 route-map UNDERLAY-IN in
   neighbor UNDERLAY-V4 route-map UNDERLAY-OUT out
   !
   neighbor UNDERLAY-V6 peer group
   neighbor UNDERLAY-V6 bfd
   neighbor UNDERLAY-V6 password OTUS
   neighbor UNDERLAY-V6 ttl maximum-hops 1
   neighbor UNDERLAY-V6 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V6 route-map UNDERLAY-IN6 in
   neighbor UNDERLAY-V6 route-map UNDERLAY-OUT6 out
   !
   neighbor 10.1.2.1 peer group UNDERLAY-V4
   neighbor 10.1.2.1 remote-as 65001
   neighbor 10.1.5.1 peer group UNDERLAY-V4
   neighbor 10.1.5.1 remote-as 65002
   !
   neighbor 2001:db8:1:2::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:2::1 remote-as 65001
   neighbor 2001:db8:1:5::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:5::1 remote-as 65002
   !
   address-family ipv4
      neighbor UNDERLAY-V4 activate
      network 192.168.0.12/32
      network 192.168.100.12/32
      exit
   !
   address-family ipv6
      neighbor UNDERLAY-V6 activate
      network 2001:db8:100::12/128
      network 2001:db8:200::12/128
      exit
    exit
```

</details>

[Leaf2 Running-conifg ](_Leaf2_running-config.txt)

<details>
<summary> Leaf3
</summary>

```
!
service routing protocols model multi-agent
!
ipv6 unicast-routing
!
hostname Leaf3
!
interface Loopback0
   ip address 192.168.0.13/32
   ipv6 address 2001:db8:100::13/128
   exit
!
interface Loopback1
   ip address 192.168.100.13/32
   ipv6 address 2001:db8:200::13/128
   exit
!
interface Ethernet1
   description eBGP_to_Spine1_Eth3
   no switchport
   mtu 9000
   ip address 10.1.3.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:3::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet2
   description eBGP_to_Spine2_Eth3
   no switchport
   mtu 9000
   ip address 10.1.6.2/30
   ipv6 enable
   ipv6 address 2001:db8:1:6::2/126
   bfd interval 300 min-rx 300 multiplier 3
   exit
!
interface Ethernet7
   description To_Client3.1
   exit
!
interface Ethernet8
   description To_Client3.2
   exit
!
ip prefix-list PL-LOOPBACKS-V4 seq 10 permit 192.168.0.0/24 le 32
ip prefix-list PL-LOOPBACKS-V4 seq 20 permit 192.168.100.0/24 le 32
!
ipv6 prefix-list PL-LOOPBACKS-V6 
   seq 10 permit 2001:db8:100::/48 le 128
   seq 20 permit 2001:db8:200::/48 le 128
   exit
!
route-map UNDERLAY-IN permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
!
route-map UNDERLAY-OUT permit 10
   match ip address prefix-list PL-LOOPBACKS-V4
!
route-map UNDERLAY-IN6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
!
route-map UNDERLAY-OUT6 permit 10
   match ipv6 address prefix-list PL-LOOPBACKS-V6
   exit
!
router bgp 65013
   router-id 192.168.0.13
   no bgp default ipv4-unicast
   bgp bestpath as-path multipath-relax
   maximum-paths 4 ecmp 4
   distance bgp 20 20 20
   !
   neighbor UNDERLAY-V4 peer group
   neighbor UNDERLAY-V4 bfd
   neighbor UNDERLAY-V4 password OTUS
   neighbor UNDERLAY-V4 ttl maximum-hops 1
   neighbor UNDERLAY-V4 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V4 route-map UNDERLAY-IN in
   neighbor UNDERLAY-V4 route-map UNDERLAY-OUT out
   !
   neighbor UNDERLAY-V6 peer group
   neighbor UNDERLAY-V6 bfd
   neighbor UNDERLAY-V6 password OTUS
   neighbor UNDERLAY-V6 ttl maximum-hops 1
   neighbor UNDERLAY-V6 maximum-routes 100 warning-limit 75
   neighbor UNDERLAY-V6 route-map UNDERLAY-IN6 in
   neighbor UNDERLAY-V6 route-map UNDERLAY-OUT6 out
   !
   neighbor 10.1.3.1 peer group UNDERLAY-V4
   neighbor 10.1.3.1 remote-as 65001
   neighbor 10.1.6.1 peer group UNDERLAY-V4
   neighbor 10.1.6.1 remote-as 65002
   !
   neighbor 2001:db8:1:3::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:3::1 remote-as 65001
   neighbor 2001:db8:1:6::1 peer group UNDERLAY-V6
   neighbor 2001:db8:1:6::1 remote-as 65002
   !
   address-family ipv4
      neighbor UNDERLAY-V4 activate
      network 192.168.0.13/32
      network 192.168.100.13/32
      exit
   !
   address-family ipv6
      neighbor UNDERLAY-V6 activate
      network 2001:db8:100::13/128
      network 2001:db8:200::13/128
      exit
    exit
```

</details>

[Leaf3 Running-conifg ](_Leaf3_running-config.txt)

<details>
<summary> Monitoring&Debag
</summary>

```
show bgp summary
show bgp ipv4 unicast summary
show bgp ipv6 unicast summary
show ip route bgp
show ipv6 route bgp
show bfd peers

debug bgp updates
debug bgp events
```

</details>



## 5. Проверка работоспособности топологии

<details>
<summary> Проверка работы протоколов на S1
</summary>

```
Spine1#show bgp summary

BGP summary information for VRF default
Router identifier 192.168.0.1, local AS number 65001
Neighbor                 AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
--------------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.2              65011 Established   IPv4 Unicast            Negotiated              4          4
10.1.2.2              65012 Established   IPv4 Unicast            Negotiated              4          4
10.1.3.2              65013 Established   IPv4 Unicast            Negotiated              4          4
2001:db8:1:1::2       65011 Established   IPv6 Unicast            Negotiated              6          6
2001:db8:1:2::2       65012 Established   IPv6 Unicast            Negotiated              6          6
2001:db8:1:3::2       65013 Established   IPv6 Unicast            Negotiated              8          8

Spine1#show bgp ipv4 unicast summary

BGP summary information for VRF default
Router identifier 192.168.0.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.2 4 65011             59        60    0    0 00:44:39 Estab   4      4
  10.1.2.2 4 65012             53        62    0    0 00:47:56 Estab   4      4
  10.1.3.2 4 65013             10        12    0    0 00:04:17 Estab   4      4

Spine1#show bgp ipv6 unicast summary

BGP summary information for VRF default
Router identifier 192.168.0.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor        V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  2001:db8:1:1::2 4 65011             16        17    0    0 00:09:35 Estab   6      6
  2001:db8:1:2::2 4 65012             14        16    0    0 00:07:39 Estab   6      6
  2001:db8:1:3::2 4 65013             14        12    0    0 00:04:13 Estab   8      8

Spine1#show ip route bgp

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 B E      192.168.0.2/32 [20/0] via 10.1.1.2, Ethernet1
                                via 10.1.2.2, Ethernet2
                                via 10.1.3.2, Ethernet3
 B E      192.168.0.11/32 [20/0] via 10.1.1.2, Ethernet1
 B E      192.168.0.12/32 [20/0] via 10.1.2.2, Ethernet2
 B E      192.168.0.13/32 [20/0] via 10.1.3.2, Ethernet3
 B E      192.168.100.2/32 [20/0] via 10.1.1.2, Ethernet1
                                  via 10.1.2.2, Ethernet2
                                  via 10.1.3.2, Ethernet3
 B E      192.168.100.11/32 [20/0] via 10.1.1.2, Ethernet1
 B E      192.168.100.12/32 [20/0] via 10.1.2.2, Ethernet2
 B E      192.168.100.13/32 [20/0] via 10.1.3.2, Ethernet3

Spine1#show ipv6 route bgp

VRF: default
Displaying 8 of 19 IPv6 routing table entries
Codes: C - connected, S - static, K - kernel, O3 - OSPFv3,
       B - Other BGP Routes, A B - BGP Aggregate, R - RIP,
       I L1 - IS-IS level 1, I L2 - IS-IS level 2, DH - DHCP,
       NG - Nexthop Group Static Route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       RC - Route Cache Route

 B E      2001:db8:100::2/128 [20/0]
           via 2001:db8:1:1::2, Ethernet1
           via 2001:db8:1:2::2, Ethernet2
           via 2001:db8:1:3::2, Ethernet3
 B E      2001:db8:100::11/128 [20/0]
           via 2001:db8:1:1::2, Ethernet1
 B E      2001:db8:100::12/128 [20/0]
           via 2001:db8:1:2::2, Ethernet2
 B E      2001:db8:100::13/128 [20/0]
           via 2001:db8:1:3::2, Ethernet3
 B E      2001:db8:200::2/128 [20/0]
           via 2001:db8:1:1::2, Ethernet1
           via 2001:db8:1:2::2, Ethernet2
           via 2001:db8:1:3::2, Ethernet3
 B E      2001:db8:200::11/128 [20/0]
           via 2001:db8:1:1::2, Ethernet1
 B E      2001:db8:200::12/128 [20/0]
           via 2001:db8:1:2::2, Ethernet2
 B E      2001:db8:200::13/128 [20/0]
           via 2001:db8:1:3::2, Ethernet3

Spine1#show bfd peers

VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp 
--------- ----------- ----------- -------------------- ------- ----------------
10.1.1.2  1688852484  2286652582        Ethernet1(15)  normal   10/03/26 12:44 
10.1.2.2  2217213932  3240150844        Ethernet2(16)  normal   10/03/26 12:41 
10.1.3.2  4206834522  3543922918        Ethernet3(17)  normal   10/03/26 13:25 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

DstAddr             MyDisc   YourDisc Interface/Transport   Type         LastUp
--------------- ---------- ---------- ------------------- ------ --------------
2001:db8:1:1::2 1442475337 2543641865       Ethernet1(15) normal 10/03/26 13:19
2001:db8:1:2::2 2208687174 2849300274       Ethernet2(16) normal 10/03/26 13:21
2001:db8:1:3::2 2983575910 4214021374       Ethernet3(17) normal 10/03/26 13:25

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```

</details>

<details>
<summary> Проверка работы протоколов на S2
</summary>

```
Spine2#show bgp summary

BGP summary information for VRF default
Router identifier 192.168.0.2, local AS number 65002
Neighbor                 AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
--------------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.4.2              65011 Established   IPv4 Unicast            Negotiated              8          8
10.1.5.2              65012 Established   IPv4 Unicast            Negotiated              8          8
10.1.6.2              65013 Established   IPv4 Unicast            Negotiated              8          8
2001:db8:1:4::2       65011 Established   IPv6 Unicast            Negotiated              6          6
2001:db8:1:5::2       65012 Established   IPv6 Unicast            Negotiated              6          6
2001:db8:1:6::2       65013 Established   IPv6 Unicast            Negotiated              4          4

Spine2#show bgp ipv4 unicast summary

BGP summary information for VRF default
Router identifier 192.168.0.2, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.4.2 4 65011             64        65    0    0 00:46:16 Estab   8      8
  10.1.5.2 4 65012             57        63    0    0 00:49:31 Estab   8      8
  10.1.6.2 4 65013             15        15    0    0 00:05:52 Estab   8      8

Spine2#show bgp ipv6 unicast summary

BGP summary information for VRF default
Router identifier 192.168.0.2, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor        V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  2001:db8:1:4::2 4 65011             18        18    0    0 00:10:53 Estab   6      6
  2001:db8:1:5::2 4 65012             18        19    0    0 00:09:10 Estab   6      6
  2001:db8:1:6::2 4 65013             11        14    0    0 00:05:54 Estab   4      4

Spine2#show ip route bgp

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 B E      192.168.0.1/32 [20/0] via 10.1.4.2, Ethernet1
                                via 10.1.5.2, Ethernet2
                                via 10.1.6.2, Ethernet3
 B E      192.168.0.11/32 [20/0] via 10.1.4.2, Ethernet1
 B E      192.168.0.12/32 [20/0] via 10.1.5.2, Ethernet2
 B E      192.168.0.13/32 [20/0] via 10.1.6.2, Ethernet3
 B E      192.168.100.1/32 [20/0] via 10.1.4.2, Ethernet1
                                  via 10.1.5.2, Ethernet2
                                  via 10.1.6.2, Ethernet3
 B E      192.168.100.11/32 [20/0] via 10.1.4.2, Ethernet1
 B E      192.168.100.12/32 [20/0] via 10.1.5.2, Ethernet2
 B E      192.168.100.13/32 [20/0] via 10.1.6.2, Ethernet3

Spine2#show ipv6 route bgp

VRF: default
Displaying 8 of 19 IPv6 routing table entries
Codes: C - connected, S - static, K - kernel, O3 - OSPFv3,
       B - Other BGP Routes, A B - BGP Aggregate, R - RIP,
       I L1 - IS-IS level 1, I L2 - IS-IS level 2, DH - DHCP,
       NG - Nexthop Group Static Route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       RC - Route Cache Route

 B E      2001:db8:100::1/128 [20/0]
           via 2001:db8:1:4::2, Ethernet1
           via 2001:db8:1:5::2, Ethernet2
           via 2001:db8:1:6::2, Ethernet3
 B E      2001:db8:100::11/128 [20/0]
           via 2001:db8:1:4::2, Ethernet1
 B E      2001:db8:100::12/128 [20/0]
           via 2001:db8:1:5::2, Ethernet2
 B E      2001:db8:100::13/128 [20/0]
           via 2001:db8:1:6::2, Ethernet3
 B E      2001:db8:200::1/128 [20/0]
           via 2001:db8:1:4::2, Ethernet1
           via 2001:db8:1:5::2, Ethernet2
           via 2001:db8:1:6::2, Ethernet3
 B E      2001:db8:200::11/128 [20/0]
           via 2001:db8:1:4::2, Ethernet1
 B E      2001:db8:200::12/128 [20/0]
           via 2001:db8:1:5::2, Ethernet2
 B E      2001:db8:200::13/128 [20/0]
           via 2001:db8:1:6::2, Ethernet3

Spine2#show bfd peers

VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp 
--------- ----------- ----------- -------------------- ------- ----------------
10.1.4.2   711099829  1044834938        Ethernet1(15)  normal   10/03/26 12:44 
10.1.5.2  4117455362  2421675363        Ethernet2(16)  normal   10/03/26 12:41 
10.1.6.2  4064491212  1667660416        Ethernet3(17)  normal   10/03/26 13:25 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

DstAddr             MyDisc   YourDisc Interface/Transport   Type         LastUp
--------------- ---------- ---------- ------------------- ------ --------------
2001:db8:1:4::2 2629926883 4167069723       Ethernet1(15) normal 10/03/26 13:20
2001:db8:1:5::2 2092555985  576490031       Ethernet2(16) normal 10/03/26 13:21
2001:db8:1:6::2 2503799457 2286974775       Ethernet3(17) normal 10/03/26 13:25

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```

</details>

<details>
<summary> Проверка доступности L1 <--> L2
</summary>

```
Leaf1(config)#ping 192.168.100.12 source 192.168.0.11 rep 1
PING 192.168.100.12 (192.168.100.12) from 192.168.0.11 : 72(100) bytes of data.
80 bytes from 192.168.100.12: icmp_seq=1 ttl=63 time=40.8 ms

--- 192.168.100.12 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 40.860/40.860/40.860/0.000 ms

Leaf1(config)#ping 2001:db8:200::12 source 2001:db8:100::11 rep 1
PING 2001:db8:200::12(2001:db8:200::12) from 2001:db8:100::11 : 52 data bytes
60 bytes from 2001:db8:200::12: icmp_seq=1 ttl=63 time=22.7 ms

--- 2001:db8:200::12 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 22.714/22.714/22.714/0.000 ms
```

</details>

<details>
<summary> Проверка доступности L1 <--> L3
</summary>

```
Leaf1(config)#ping 192.168.100.13 source 192.168.0.11 rep 1
PING 192.168.100.13 (192.168.100.13) from 192.168.0.11 : 72(100) bytes of data.
80 bytes from 192.168.100.13: icmp_seq=1 ttl=63 time=28.0 ms

--- 192.168.100.13 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 28.018/28.018/28.018/0.000 ms

Leaf1(config)#ping 2001:db8:200::13 source 2001:db8:100::11 rep 1
PING 2001:db8:200::13(2001:db8:200::13) from 2001:db8:100::11 : 52 data bytes
60 bytes from 2001:db8:200::13: icmp_seq=1 ttl=63 time=23.7 ms

--- 2001:db8:200::13 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 23.757/23.757/23.757/0.000 ms
```

</details>

<details>
<summary> Проверка доступности L2 <--> L3
</summary>

```
Leaf2#ping 192.168.100.13 source 192.168.0.12 rep 1
PING 192.168.100.13 (192.168.100.13) from 192.168.0.12 : 72(100) bytes of data.
80 bytes from 192.168.100.13: icmp_seq=1 ttl=63 time=35.1 ms

--- 192.168.100.13 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 35.159/35.159/35.159/0.000 ms

Leaf2#ping 2001:db8:200::13 source 2001:db8:100::12 rep 1
PING 2001:db8:200::13(2001:db8:200::13) from 2001:db8:100::12 : 52 data bytes
60 bytes from 2001:db8:200::13: icmp_seq=1 ttl=63 time=23.8 ms

--- 2001:db8:200::13 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 23.805/23.805/23.805/0.000 ms
```

</details>
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
| Device     | Hostname  | Platform        | Interface Loopback | IP network /128      |
|------------|-----------|-----------------| -------------------|----------------------|
| Spine1     | S1        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::1/128  |
| Spine1     | S1        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::1/128  |
| Spine2     | S2        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::2/128  |
| Spine2     | S2        | Arista vEOS-lab | Loopback 1         | 2001:db8:200::2/128  |
| Leaf1      | L1        | Arista vEOS-lab | Loopback 0         | 2001:db8:100::11/128 |
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

```

</details>

[Spine1 Running-conifg ](_Spine1_running-config.txt)

<details>
<summary> Spine2
</summary>

```

```

</details>

[Spine2 Running-conifg ](_Spine2_running-config.txt)

<details>
<summary> Leaf1
</summary>

```

```

</details>

[Leaf1 Running-conifg ](_Leaf1_running-config.txt)

<details>
<summary> Leaf2
</summary>

```

```

</details>

[Leaf2 Running-conifg ](_Leaf2_running-config.txt)

<details>
<summary> Leaf3
</summary>

```

```

</details>

[Leaf3 Running-conifg ](_Leaf3_running-config.txt)

<details>
<summary> Monitoring/Debug
</summary>

```
show bgp summary
show bgp ipv6 unicast summary
show ip route bgp
show ipv6 route bgp
show bgp neighbors 10.1.1.2
show bgp ipv4 unicast paths
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

```

</details>

<details>
<summary> Проверка работы протоколов на S2
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
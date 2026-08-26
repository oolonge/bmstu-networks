# Сбор конфигурации сети - Лабораторная работа 5 (NAT)

## Логическая структура сети

- **Провайдер**: Router1 (центральный), Router2
- **Клиенты провайдера**:
  - Router3 — Dynamic NAT
  - Router4 — Static NAT
  - Router5 — PAT

---

## Router1 (Центральный роутер провайдера)

### Базовая конфигурация
```
show running-config
```

**Вывод:**
```
Router1#show running-config 
Building configuration...

Current configuration : 865 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router1
!
!
!
!
!
!
!
!
no ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2811/K9 sn FTX1017BINB-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface FastEthernet0/0
 ip address 172.17.200.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 172.17.201.1 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address 172.17.202.1 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet1/1
 ip address 172.17.203.1 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 0.0.0.0 0.0.0.0 172.17.200.1 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### Таблица маршрутизации
```
show ip route
```

**Вывод:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.200.1 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 8 subnets, 3 masks
C       172.17.200.0/30 is directly connected, FastEthernet0/0
L       172.17.200.2/32 is directly connected, FastEthernet0/0
C       172.17.201.0/24 is directly connected, FastEthernet0/1
L       172.17.201.1/32 is directly connected, FastEthernet0/1
C       172.17.202.0/24 is directly connected, FastEthernet1/0
L       172.17.202.1/32 is directly connected, FastEthernet1/0
C       172.17.203.0/30 is directly connected, FastEthernet1/1
L       172.17.203.1/32 is directly connected, FastEthernet1/1
S*   0.0.0.0/0 [1/0] via 172.17.200.1
```

### Состояние интерфейсов
```
show ip interface brief
```

**Вывод:**
```
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/0        172.17.200.2    YES manual up                    up 
FastEthernet0/1        172.17.201.1    YES manual up                    up 
FastEthernet1/0        172.17.202.1    YES manual up                    up 
FastEthernet1/1        172.17.203.1    YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

---

## Router2 (Роутер провайдера, подключён к серверам)

### Базовая конфигурация
```
show running-config
```

**Вывод:**
```
Building configuration...

Current configuration : 839 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router2
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2811/K9 sn FTX1017I009-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface FastEthernet0/0
 ip address 172.17.100.1 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 172.17.200.1 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 172.17.201.0 255.255.255.252 172.17.200.2 
ip route 172.17.202.0 255.255.255.0 172.17.200.2 
ip route 172.17.201.0 255.255.255.0 172.17.200.2 
ip route 172.17.203.0 255.255.255.0 172.17.200.2 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### Таблица маршрутизации
```
show ip route
```

**Вывод:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     172.17.0.0/16 is variably subnetted, 8 subnets, 3 masks
C       172.17.100.0/24 is directly connected, FastEthernet0/0
L       172.17.100.1/32 is directly connected, FastEthernet0/0
C       172.17.200.0/30 is directly connected, FastEthernet0/1
L       172.17.200.1/32 is directly connected, FastEthernet0/1
S       172.17.201.0/24 [1/0] via 172.17.200.2
S       172.17.201.0/30 [1/0] via 172.17.200.2
S       172.17.202.0/24 [1/0] via 172.17.200.2
S       172.17.203.0/24 [1/0] via 172.17.200.2
```

### Состояние интерфейсов
```
show ip interface brief
```

**Вывод:**
```
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/0        172.17.100.1    YES manual up                    up 
FastEthernet0/1        172.17.200.1    YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

---

## Router3 (Dynamic NAT)

### Базовая конфигурация
```
show running-config
```

**Вывод:**
```
Building configuration...

Current configuration : 860 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router3
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2811/K9 sn FTX1017IXIB-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface FastEthernet0/0
 ip address 172.17.202.2 255.255.255.0
 ip nat outside
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 172.17.20.1 255.255.255.0
 ip nat inside
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip nat pool POOL_202 172.17.202.5 172.17.202.6 netmask 255.255.255.248
ip nat inside source list 1 pool POOL_202
ip classless
ip route 0.0.0.0 0.0.0.0 172.17.202.1 
!
ip flow-export version 9
!
!
access-list 1 permit 172.17.20.0 0.0.0.255
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### Таблица маршрутизации
```
show ip route
```

**Вывод:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.202.1 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 4 subnets, 2 masks
C       172.17.20.0/24 is directly connected, FastEthernet0/1
L       172.17.20.1/32 is directly connected, FastEthernet0/1
C       172.17.202.0/24 is directly connected, FastEthernet0/0
L       172.17.202.2/32 is directly connected, FastEthernet0/0
S*   0.0.0.0/0 [1/0] via 172.17.202.1
```

### Состояние интерфейсов
```
show ip interface brief
```

**Вывод:**
```
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/0        172.17.202.2    YES manual up                    up 
FastEthernet0/1        172.17.20.1     YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

### NAT-трансляции
```
show ip nat translations
```

**Вывод:**
```
Pro  Inside global     Inside local       Outside local      Outside global
icmp 172.17.202.5:1    172.17.20.2:1      172.17.30.10:1     172.17.30.10:1
icmp 172.17.202.5:2    172.17.20.2:2      172.17.30.10:2     172.17.30.10:2
icmp 172.17.202.5:3    172.17.20.2:3      172.17.10.2:3      172.17.10.2:3
icmp 172.17.202.5:4    172.17.20.2:4      172.17.30.11:4     172.17.30.11:4
```

### NAT-статистика
```
show ip nat statistics
```

**Вывод:**
```
Total translations: 4 (0 static, 4 dynamic, 4 extended)
Outside Interfaces: FastEthernet0/0
Inside Interfaces: FastEthernet0/1
Hits: 1  Misses: 4
Expired translations: 0
Dynamic mappings:
-- Inside Source
access-list 1 pool POOL_202 refCount 4
 pool POOL_202: netmask 255.255.255.248
       start 172.17.202.5 end 172.17.202.6
       type generic, total addresses 2 , allocated 1 (50%), misses 0
```

---

## Router4 (Static NAT)

### Базовая конфигурация
```
show running-config
```

**Вывод:**
```
Building configuration...

Current configuration : 758 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router4
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2811/K9 sn FTX1017T9Z9-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface FastEthernet0/0
 ip address 172.17.201.2 255.255.255.0
 ip nat outside
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 172.17.10.1 255.255.255.0
 ip nat inside
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip nat inside source static 172.17.10.2 172.17.201.5 
ip classless
ip route 0.0.0.0 0.0.0.0 172.17.201.1 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### Таблица маршрутизации
```
show ip route
```

**Вывод:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.201.1 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 4 subnets, 2 masks
C       172.17.10.0/24 is directly connected, FastEthernet0/1
L       172.17.10.1/32 is directly connected, FastEthernet0/1
C       172.17.201.0/24 is directly connected, FastEthernet0/0
L       172.17.201.2/32 is directly connected, FastEthernet0/0
S*   0.0.0.0/0 [1/0] via 172.17.201.1
```

### Состояние интерфейсов
```
show ip interface brief
```

**Вывод:**
```
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/0        172.17.201.2    YES manual up                    up 
FastEthernet0/1        172.17.10.1     YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

### NAT-трансляции
```
show ip nat translations
```

**Вывод:**
```
Pro  Inside global     Inside local       Outside local      Outside global
icmp 172.17.201.5:1    172.17.10.2:1      172.17.20.2:1      172.17.20.2:1
---  172.17.201.5      172.17.10.2        ---                ---

```

### NAT-статистика
```
show ip nat statistics
```

**Вывод:**
```
Total translations: 2 (1 static, 1 dynamic, 1 extended)
Outside Interfaces: FastEthernet0/0
Inside Interfaces: FastEthernet0/1
Hits: 0  Misses: 1
Expired translations: 0
Dynamic mappings:
```

---

## Router5 (PAT)

### Базовая конфигурация
```
show running-config
```

**Вывод:**
```
Building configuration...

Current configuration : 1004 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router5
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2811/K9 sn FTX1017E059-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface FastEthernet0/0
 ip address 172.17.203.2 255.255.255.252
 ip nat outside
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 172.17.30.1 255.255.255.0
 ip helper-address 172.17.31.2
 ip nat inside
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address 172.17.31.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/1
 no ip address
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip nat inside source list 1 interface FastEthernet0/0 overload
ip classless
ip route 0.0.0.0 0.0.0.0 172.17.203.1 
!
ip flow-export version 9
!
!
access-list 1 permit 172.17.30.0 0.0.0.255
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end

```

### Таблица маршрутизации
```
show ip route
```

**Вывод:**
```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.203.1 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 6 subnets, 3 masks
C       172.17.30.0/24 is directly connected, FastEthernet0/1
L       172.17.30.1/32 is directly connected, FastEthernet0/1
C       172.17.31.0/30 is directly connected, FastEthernet1/0
L       172.17.31.1/32 is directly connected, FastEthernet1/0
C       172.17.203.0/30 is directly connected, FastEthernet0/0
L       172.17.203.2/32 is directly connected, FastEthernet0/0
S*   0.0.0.0/0 [1/0] via 172.17.203.1
```

### Состояние интерфейсов
```
show ip interface brief
```

**Вывод:**
```
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/0        172.17.203.2    YES NVRAM  up                    up 
FastEthernet0/1        172.17.30.1     YES NVRAM  up                    up 
FastEthernet1/0        172.17.31.1     YES manual up                    up 
FastEthernet1/1        unassigned      YES unset  down                  down 
Vlan1                  unassigned      YES unset  administratively down down
```

### NAT-трансляции
```
show ip nat translations
```

**Вывод:**
```
Pro  Inside global     Inside local       Outside local      Outside global
icmp 172.17.203.2:1024 172.17.30.11:1     172.17.10.2:1      172.17.10.2:1024
icmp 172.17.203.2:1    172.17.30.10:1     172.17.10.2:1      172.17.10.2:1
```

### NAT-статистика
```
show ip nat statistics
```

**Вывод:**
```
Total translations: 2 (0 static, 2 dynamic, 2 extended)
Outside Interfaces: FastEthernet0/0
Inside Interfaces: FastEthernet0/1
Hits: 1  Misses: 2
Expired translations: 0
Dynamic mappings:
```

---

## Серверы

### FTPServer

**IP-конфигурация** (из GUI или ipconfig):
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::260:2FFF:FEB7:2A53
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.100.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.100.1
```

**DNS-имя (если настроено)**:
```

```

### DNSServer

**IP-конфигурация**:
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::240:BFF:FEE7:59AB
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.100.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.100.1
```

**DNS-записи** (из GUI сервера, вкладка DNS):
```
DNS Service: ON

No.  Name            Type      Detail
0    ftp.com         A Record  172.17.100.3
1    ftp.files.com   A Record  172.17.100.3
```

### DHCPServer

**IP-конфигурация**:
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::260:47FF:FE91:9A19
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.31.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.31.1
```

**DHCP-пулы** (из GUI сервера, вкладка DHCP):
```
DHCP Service: ON

Pool Name    Default Gateway  DNS Server    Start IP       Subnet Mask     Max User
Local30      172.17.30.1      172.17.100.2  172.17.30.10   255.255.255.0   246
serverPool   0.0.0.0          0.0.0.0       172.17.31.0    255.255.255.0   512
```

---

## Клиентские ПК

### PC0 (подключён через Switch1 к Router4 - Static NAT)

```
ipconfig /all
```

**Вывод:**
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0030.A391.484B
   Link-local IPv6 Address.........: FE80::230:A3FF:FE91:484B
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.10.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.10.1
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-3D-25-AD-89-00-30-A3-91-48-4B
   DNS Servers.....................: ::
                                     172.17.100.2

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0002.16C2.EBBD
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-3D-25-AD-89-00-30-A3-91-48-4B
   DNS Servers.....................: ::
                                     172.17.100.2
```

### PC1 (подключён через Switch2 к Router3 - Dynamic NAT)

```
ipconfig /all
```

**Вывод:**
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.4789.13A8
   Link-local IPv6 Address.........: FE80::260:47FF:FE89:13A8
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.20.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.20.1
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-45-6C-DE-1B-00-60-47-89-13-A8
   DNS Servers.....................: ::
                                     172.17.100.2

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0005.5E3B.84D2
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-45-6C-DE-1B-00-60-47-89-13-A8
   DNS Servers.....................: ::
                                     172.17.100.2
```

### PC2 (подключён через Switch3 к Router5 - PAT)

```
ipconfig /all
```

**Вывод:**
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.C9AB.285D
   Link-local IPv6 Address.........: FE80::201:C9FF:FEAB:285D
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.30.10
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.30.1
   DHCP Servers....................: 172.17.31.2
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-8B-7B-B0-9D-00-01-C9-AB-28-5D
   DNS Servers.....................: ::
                                     172.17.100.2

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0030.A359.026E
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-8B-7B-B0-9D-00-01-C9-AB-28-5D
   DNS Servers.....................: ::
                                     172.17.100.2
```

### PC3 (подключён через Switch3 к Router5 - PAT)

```
ipconfig /all
```

**Вывод:**
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.B046.196E
   Link-local IPv6 Address.........: FE80::2E0:B0FF:FE46:196E
   IPv6 Address....................: ::
   IPv4 Address....................: 172.17.30.11
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.17.30.1
   DHCP Servers....................: 172.17.31.2
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-6D-0C-00-70-00-E0-B0-46-19-6E
   DNS Servers.....................: ::
                                     172.17.100.2

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 00D0.97D8.638A
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-6D-0C-00-70-00-E0-B0-46-19-6E
   DNS Servers.....................: ::
                                     172.17.100.2
```

---

## Проверка связности (опционально)

### С PC0 на FTPServer
```
ping <IP_FTPServer>
```

**Вывод:**
```
C:\>ping 172.17.100.3

Pinging 172.17.100.3 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125

Ping statistics for 172.17.100.3:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 172.17.100.3

Pinging 172.17.100.3 with 32 bytes of data:

Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time=1ms TTL=125
Reply from 172.17.100.3: bytes=32 time=1ms TTL=125

Ping statistics for 172.17.100.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 172.17.100.3

Pinging 172.17.100.3 with 32 bytes of data:

Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time=1ms TTL=125
Reply from 172.17.100.3: bytes=32 time=34ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125

Ping statistics for 172.17.100.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 34ms, Average = 8ms
```

пинговал несколько раз подряд, после того, как видел lost. После первого lost всё стало норм

### С PC0 на FTPServer по DNS-имени
```
ping <dns_имя_ftp>
```

**Вывод:**
```
C:\>ping ftp.com

Pinging 172.17.100.3 with 32 bytes of data:

Reply from 172.17.100.3: bytes=32 time=13ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125
Reply from 172.17.100.3: bytes=32 time<1ms TTL=125

Ping statistics for 172.17.100.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 13ms, Average = 3ms
```

---

## Примечания

Заполни пустые блоки кода выводом соответствующих команд. После заполнения я проанализирую конфигурацию и отвечу на твои вопросы.

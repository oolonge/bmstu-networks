# Полное руководство по настройке сетевой топологии

## Обзор топологии

Топология состоит из двух автономных систем (AS), связанных через BGP:
- **AS 65001** (OSPF area 0): сеть 172.21.16.0/16 с маршрутизаторами R1, R2, R3, R4
- **AS 65002**: сеть 172.22.0.0/16 с маршрутизатором R5

## Таблица устройств и интерфейсов

### Маршрутизаторы (Cisco 2811)

| Устройство | Интерфейс | IP-адрес | Назначение |
|------------|-----------|----------|------------|
| **R1** | Fa0/0 | 172.21.1.1/24 | К R2 |
| | Fa0/1 | 172.21.2.1/24 | К R3 |
| | Fa1/1 | 172.21.100.1/30 | К R5 (BGP) |
| **R2** | Fa0/0 | 172.21.1.2/24 | К R1 |
| | Fa0/1 | 172.21.3.2/24 | К R4 |
| | Fa1/0 | 172.21.10.1/24 | К PC0 |
| **R3** | Fa0/0 | 172.21.2.3/24 | К R1 |
| | Fa0/1 | 172.21.4.3/24 | К R4 |
| **R4** | Fa0/1 | 172.21.3.4/24 | К R2 |
| | Fa1/0 | 172.21.4.4/24 | К R3 |
| | Fa1/1 | 172.21.20.1/24 | К PC3 |
| | Fa0/0 | 172.21.21.1/24 | К PC4 |
| **R5** | Fa1/1 | 172.21.100.2/30 | К R1 (BGP) |
| | Fa0/1 | 172.22.10.1/24 | К PC1 |
| | Fa1/0 | 172.22.11.1/24 | К PC2 |

### Компьютеры

| Устройство | IP-адрес | Маска подсети | Шлюз по умолчанию |
|------------|----------|---------------|-------------------|
| **PC0** | 172.21.10.10 | 255.255.255.0 | 172.21.10.1 |
| **PC1** | 172.22.10.10 | 255.255.255.0 | 172.22.10.1 |
| **PC2** | 172.22.11.10 | 255.255.255.0 | 172.22.11.1 |
| **PC3** | 172.21.20.10 | 255.255.255.0 | 172.21.20.1 |
| **PC4** | 172.21.21.10 | 255.255.255.0 | 172.21.21.1 |

---

## Часть 1: Создание топологии в Cisco Packet Tracer

### Шаг 1: Добавление устройств

1. Откройте Cisco Packet Tracer
2. Добавьте следующие устройства:
   - 5x Router 2811 (R1, R2, R3, R4, R5)
   - 5x PC-PT (PC0, PC1, PC2, PC3, PC4)

### Шаг 2: Подключение устройств

Соедините устройства медными прямыми кабелями (Copper Straight-Through):

**AS 65001 (внутренние связи):**
- R1 Fa0/0 ↔ R2 Fa0/0
- R1 Fa0/1 ↔ R3 Fa0/0
- R2 Fa0/1 ↔ R4 Fa0/1
- R3 Fa0/1 ↔ R4 Fa1/0

**Связь между AS (BGP):**
- R1 Fa1/1 ↔ R5 Fa1/1

**Подключение конечных устройств:**
- PC0 Fa0 ↔ R2 Fa1/0
- PC1 Fa0 ↔ R5 Fa0/1
- PC2 Fa0 ↔ R5 Fa1/0
- PC3 Fa0 ↔ R4 Fa1/1
- PC4 Fa0 ↔ R4 Fa0/0

---

## Часть 2: Базовая настройка маршрутизаторов

### R1 - Граничный маршрутизатор AS 65001

```cisco
enable
configure terminal

! Установка hostname
hostname R1

! Настройка интерфейсов
interface FastEthernet0/0
 description Link to R2
 ip address 172.21.1.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description Link to R3
 ip address 172.21.2.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet1/1
 description BGP Link to R5 (AS 65002)
 ip address 172.21.100.1 255.255.255.252
 no shutdown
 exit

! Сохранение конфигурации
end
write memory
```

**Назначение команд:**
- `hostname R1` - устанавливает имя маршрутизатора для удобства идентификации
- `ip address [IP] [MASK]` - назначает IP-адрес интерфейсу
- `no shutdown` - активирует интерфейс (по умолчанию они выключены)
- `description` - добавляет описание интерфейса для документации
- `write memory` - сохраняет конфигурацию в NVRAM (не потеряется при перезагрузке)

### R2 - Внутренний маршрутизатор с доступом к PC0

```cisco
enable
configure terminal

hostname R2

interface FastEthernet0/0
 description Link to R1
 ip address 172.21.1.2 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description Link to R4
 ip address 172.21.3.2 255.255.255.0
 no shutdown
 exit

interface FastEthernet1/0
 description Link to PC0
 ip address 172.21.10.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

### R3 - Внутренний маршрутизатор

```cisco
enable
configure terminal

hostname R3

interface FastEthernet0/0
 description Link to R1
 ip address 172.21.2.3 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description Link to R4
 ip address 172.21.4.3 255.255.255.0
 no shutdown
 exit

end
write memory
```

### R4 - Внутренний маршрутизатор с доступом к PC3 и PC4

```cisco
enable
configure terminal

hostname R4

interface FastEthernet0/1
 description Link to R2
 ip address 172.21.3.4 255.255.255.0
 no shutdown
 exit

interface FastEthernet1/0
 description Link to R3
 ip address 172.21.4.4 255.255.255.0
 no shutdown
 exit

interface FastEthernet1/1
 description Link to PC3
 ip address 172.21.20.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/0
 description Link to PC4
 ip address 172.21.21.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

### R5 - Маршрутизатор AS 65002

```cisco
enable
configure terminal

hostname R5

interface FastEthernet1/1
 description BGP Link to R1 (AS 65001)
 ip address 172.21.100.2 255.255.255.252
 no shutdown
 exit

interface FastEthernet0/1
 description Link to PC1
 ip address 172.22.10.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet1/0
 description Link to PC2
 ip address 172.22.11.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

---

## Часть 3: Настройка OSPF в AS 65001

OSPF (Open Shortest Path First) - это протокол динамической маршрутизации, который автоматически обменивается информацией о маршрутах между маршрутизаторами в одной автономной системе.

### R1 - OSPF

```cisco
enable
configure terminal

router ospf 1
 router-id 1.1.1.1
 network 172.21.1.0 0.0.0.255 area 0
 network 172.21.2.0 0.0.0.255 area 0
 passive-interface FastEthernet1/1
 exit

end
write memory
```

**Назначение команд OSPF:**
- `router ospf 1` - запускает процесс OSPF с ID 1
- `router-id 1.1.1.1` - уникальный идентификатор маршрутизатора в OSPF
- `network [IP] [wildcard] area 0` - объявляет сети, участвующие в OSPF
- `wildcard mask` - инверсия маски подсети (255.255.255.0 → 0.0.0.255)
- `passive-interface` - отключает отправку OSPF hello-пакетов на интерфейс (для интерфейсов, подключенных к другим AS или конечным устройствам)

### R2 - OSPF

```cisco
enable
configure terminal

router ospf 1
 router-id 2.2.2.2
 network 172.21.1.0 0.0.0.255 area 0
 network 172.21.3.0 0.0.0.255 area 0
 network 172.21.10.0 0.0.0.255 area 0
 passive-interface FastEthernet1/0
 exit

end
write memory
```

### R3 - OSPF

```cisco
enable
configure terminal

router ospf 1
 router-id 3.3.3.3
 network 172.21.2.0 0.0.0.255 area 0
 network 172.21.4.0 0.0.0.255 area 0
 exit

end
write memory
```

### R4 - OSPF

```cisco
enable
configure terminal

router ospf 1
 router-id 4.4.4.4
 network 172.21.3.0 0.0.0.255 area 0
 network 172.21.4.0 0.0.0.255 area 0
 network 172.21.20.0 0.0.0.255 area 0
 network 172.21.21.0 0.0.0.255 area 0
 passive-interface FastEthernet1/1
 passive-interface FastEthernet0/0
 exit

end
write memory
```

---

## Часть 4: Настройка BGP между AS

BGP (Border Gateway Protocol) - это протокол маршрутизации, используемый для обмена информацией между различными автономными системами в Интернете.

### R1 - BGP (AS 65001)

```cisco
enable
configure terminal

router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 172.21.100.2 remote-as 65002
 network 172.21.0.0 mask 255.255.0.0
 exit

end
write memory
```

**Назначение команд BGP:**
- `router bgp 65001` - запускает процесс BGP с номером AS 65001
- `bgp router-id` - идентификатор маршрутизатора в BGP
- `neighbor [IP] remote-as [AS]` - устанавливает BGP-соседство с маршрутизатором из другой AS
- `network [IP] mask [MASK]` - объявляет сети, которые будут анонсироваться в BGP

### R5 - BGP (AS 65002)

```cisco
enable
configure terminal

router bgp 65002
 bgp router-id 5.5.5.5
 neighbor 172.21.100.1 remote-as 65001
 network 172.22.0.0 mask 255.255.0.0
 exit

end
write memory
```

### R5 - Статические маршруты для локальных сетей

Поскольку R5 не использует динамическую маршрутизацию внутри AS 65002, необходимо добавить статические маршруты для локальных сетей:

```cisco
enable
configure terminal

ip route 172.22.10.0 255.255.255.0 FastEthernet0/1
ip route 172.22.11.0 255.255.255.0 FastEthernet1/0

end
write memory
```

**Назначение:**
- `ip route [network] [mask] [interface/next-hop]` - создает статический маршрут
- Эти маршруты необходимы для агрегации в сеть 172.22.0.0/16, которая анонсируется через BGP

---

## Часть 5: Настройка компьютеров

### PC0 (подключен к R2)

1. Перейдите в Desktop → IP Configuration
2. Установите:
   - IP Address: `172.21.10.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `172.21.10.1`

### PC1 (подключен к R5)

1. Desktop → IP Configuration
2. Установите:
   - IP Address: `172.22.10.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `172.22.10.1`

### PC2 (подключен к R5)

1. Desktop → IP Configuration
2. Установите:
   - IP Address: `172.22.11.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `172.22.11.1`

### PC3 (подключен к R4)

1. Desktop → IP Configuration
2. Установите:
   - IP Address: `172.21.20.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `172.21.20.1`

### PC4 (подключен к R4)

1. Desktop → IP Configuration
2. Установите:
   - IP Address: `172.21.21.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `172.21.21.1`

---

## Часть 6: Проверка работоспособности

### Проверка на маршрутизаторах

#### Проверка интерфейсов
```cisco
show ip interface brief
```
**Что проверяет:** статус всех интерфейсов и их IP-адреса. Status и Protocol должны быть "up".

#### Проверка OSPF соседей
```cisco
show ip ospf neighbor
```
**Что проверяет:** список соседей OSPF, их состояние должно быть FULL.

#### Проверка OSPF маршрутов
```cisco
show ip route ospf
```
**Что проверяет:** маршруты, полученные через OSPF (помечены как "O").

#### Проверка BGP соседей
```cisco
show ip bgp summary
```
**Что проверяет:** состояние BGP-соседства. State/PfxRcd должен показывать число принятых префиксов.

#### Проверка BGP маршрутов
```cisco
show ip bgp
```
**Что проверяет:** таблицу BGP с маршрутами от соседей.

#### Проверка всей таблицы маршрутизации
```cisco
show ip route
```
**Что проверяет:** полную таблицу маршрутизации с обозначениями:
- C - directly connected
- O - OSPF
- B - BGP
- S - static

### Проверка связности с компьютеров

#### Тест 1: Проверка связи внутри AS 65001
С PC0 выполните:
```
ping 172.21.20.10  (PC3)
ping 172.21.21.10  (PC4)
```
**Ожидаемый результат:** успешный ответ (Reply from...), пакеты передаются через OSPF.

#### Тест 2: Проверка связи между AS
С PC0 выполните:
```
ping 172.22.10.10  (PC1 в AS 65002)
ping 172.22.11.10  (PC2 в AS 65002)
```
**Ожидаемый результат:** успешный ответ, пакеты передаются через BGP между AS.

#### Тест 3: Обратная проверка
С PC1 выполните:
```
ping 172.21.10.10  (PC0 в AS 65001)
```

#### Traceroute
С PC0 выполните:
```
tracert 172.22.10.10
```
**Что показывает:** путь пакетов через маршрутизаторы (должен проходить через R2 → R1 → R5 → PC1).

---

## Часть 7: Настройка ACL (Access Control Lists)

ACL используются для фильтрации трафика и повышения безопасности сети.

### Пример 1: Запретить ICMP (ping) от PC0 к PC1

На маршрутизаторе R1:

```cisco
enable
configure terminal

! Создание ACL
access-list 100 deny icmp host 172.21.10.10 host 172.22.10.10
access-list 100 permit ip any any

! Применение ACL на интерфейс
interface FastEthernet1/1
 ip access-group 100 out
 exit

end
write memory
```

**Назначение:**
- `access-list 100` - создает расширенный ACL с номером 100
- `deny icmp` - блокирует ICMP-пакеты (ping)
- `host` - указывает на конкретный хост
- `permit ip any any` - разрешает весь остальной трафик (обязательно, иначе весь остальной трафик будет заблокирован)
- `ip access-group 100 out` - применяет ACL на исходящий трафик интерфейса

**Проверка:** ping от PC0 к PC1 должен быть заблокирован, но другой трафик (например, TCP) будет работать.

### Пример 2: Разрешить только HTTP (порт 80) от PC3 к PC1

На R1:

```cisco
enable
configure terminal

access-list 101 permit tcp host 172.21.20.10 host 172.22.10.10 eq 80
access-list 101 deny ip host 172.21.20.10 host 172.22.10.10
access-list 101 permit ip any any

interface FastEthernet1/1
 ip access-group 101 out
 exit

end
write memory
```

### Удаление ACL

```cisco
enable
configure terminal

interface FastEthernet1/1
 no ip access-group 100 out
 exit

no access-list 100

end
write memory
```

---

## Часть 8: Настройка NAT (Network Address Translation)

NAT позволяет преобразовывать внутренние IP-адреса во внешние при выходе в другую сеть.

### Настройка NAT на R1 (для AS 65001)

```cisco
enable
configure terminal

! Определение внутренних и внешних интерфейсов
interface FastEthernet0/0
 ip nat inside
 exit

interface FastEthernet0/1
 ip nat inside
 exit

interface FastEthernet1/1
 ip nat outside
 exit

! Создание ACL для NAT
access-list 1 permit 172.21.0.0 0.0.255.255

! Настройка NAT overload (PAT)
ip nat inside source list 1 interface FastEthernet1/1 overload

end
write memory
```

**Назначение:**
- `ip nat inside` - помечает интерфейс как внутренний (со стороны локальной сети)
- `ip nat outside` - помечает интерфейс как внешний (со стороны Интернета/другой AS)
- `ip nat inside source list 1 interface Fa1/1 overload` - преобразует адреса из ACL 1 в адрес интерфейса Fa1/1 с использованием PAT (множество внутренних адресов → один внешний с разными портами)

### Проверка NAT

```cisco
show ip nat translations
show ip nat statistics
```

---

## Дополнительные настройки безопасности

### Установка паролей

```cisco
enable
configure terminal

! Пароль на привилегированный режим
enable secret cisco123

! Пароль на консоль
line console 0
 password console123
 login
 exit

! Пароль на VTY (Telnet/SSH)
line vty 0 4
 password vty123
 login
 exit

! Шифрование паролей в конфигурации
service password-encryption

end
write memory
```

### Настройка баннера

```cisco
enable
configure terminal

banner motd #
*************************************
  Unauthorized access is prohibited!
  Company Network - AS 65001
*************************************
#

end
write memory
```

---

## Полезные команды для диагностики

### Основные команды show

```cisco
show running-config          ! Текущая конфигурация
show startup-config          ! Сохраненная конфигурация
show ip interface brief      ! Краткая информация об интерфейсах
show ip route                ! Таблица маршрутизации
show ip protocols            ! Активные протоколы маршрутизации
show ip ospf neighbor        ! OSPF соседи
show ip bgp summary          ! BGP соседи
show ip bgp                  ! BGP таблица
show ip nat translations     ! NAT трансляции
show access-lists            ! ACL списки
show version                 ! Информация об устройстве
show interfaces              ! Детальная информация об интерфейсах
show cdp neighbors           ! CDP соседи (Cisco Discovery Protocol)
```

### Команды debug (использовать осторожно!)

```cisco
debug ip ospf events         ! Отладка OSPF событий
debug ip bgp                 ! Отладка BGP
undebug all                  ! Отключить всю отладку
```

### Очистка и перезагрузка

```cisco
clear ip route *             ! Очистить таблицу маршрутизации
clear ip bgp *               ! Перезапустить BGP сессии
reload                       ! Перезагрузка маршрутизатора
```

---

## Типичные проблемы и решения

### Проблема 1: Интерфейс показывает "administratively down"
**Решение:** Выполните `no shutdown` на интерфейсе

### Проблема 2: OSPF соседи не устанавливаются
**Возможные причины:**
- Разные area ID
- Разные hello/dead интервалы
- Проблемы с сетевой связностью
- Несовпадение масок подсети

**Проверка:**
```cisco
show ip ospf interface
show ip ospf neighbor
debug ip ospf hello
```

### Проблема 3: BGP соседство не устанавливается
**Возможные причины:**
- Неправильный IP-адрес соседа
- Неправильный номер AS
- Проблемы с маршрутизацией до BGP-соседа
- Firewall/ACL блокирует TCP порт 179

**Проверка:**
```cisco
show ip bgp summary
show ip bgp neighbors
debug ip bgp
```

### Проблема 4: Нет связи между AS
**Проверка:**
- BGP соседство установлено: `show ip bgp summary`
- Маршруты анонсируются: `show ip bgp`
- Маршруты установлены: `show ip route bgp`
- ACL не блокирует трафик: `show access-lists`

### Проблема 5: Ping не работает, но маршруты есть
**Возможные причины:**
- ACL блокирует ICMP
- Обратный маршрут отсутствует
- NAT неправильно настроен

---

## Схема последовательности настройки

1. **Физическое подключение** - создать топологию, соединить устройства
2. **Базовая настройка** - hostname, IP-адреса на интерфейсах
3. **Проверка связности** - ping между напрямую подключенными устройствами
4. **OSPF** - настроить внутри AS 65001
5. **Проверка OSPF** - проверить соседство и маршруты
6. **BGP** - настроить между R1 и R5
7. **Проверка BGP** - проверить соседство и анонсы
8. **Настройка PC** - назначить IP-адреса
9. **Тестирование end-to-end** - ping между всеми устройствами
10. **ACL (опционально)** - настроить фильтрацию трафика
11. **NAT (опционально)** - настроить трансляцию адресов
12. **Безопасность** - пароли, баннеры

---

## Следующие шаги

После настройки базовой топологии можно изучить:
- **VLAN** - разделение сетей на виртуальные LAN
- **VTP** - автоматическая синхронизация VLAN
- **STP** - предотвращение петель в сети
- **EIGRP** - альтернативный протокол динамической маршрутизации
- **IPv6** - настройка современного протокола IP
- **QoS** - приоритизация трафика
- **VPN** - безопасные туннели между сетями

---

## Файлы проекта

- `lab_01_final.pkt` - готовая лабораторная работа №1
- `lab_02.pkt` - лабораторная работа №2
- `topologies/cisco_topology.png` - схема топологии
- `tasks/` - задания по различным темам (VLAN, STP, Routing, ACL, NAT)

---

**Автор:** Claude
**Дата создания:** 2025-11-23
**Версия:** 1.0

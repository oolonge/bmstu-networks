# Лабораторная 5: NAT (Network Address Translation)

## Топология

- **Провайдер**: Router1, Router2
- **Клиенты**: Router3 (Dynamic NAT), Router4 (Static NAT), Router5 (PAT)
- **Серверы**: FTPServer (172.17.100.3), DNSServer (172.17.100.2), DHCPServer (172.17.31.2)

---

## Терминология NAT

| Термин | Описание |
|--------|----------|
| Inside local | Реальный IP хоста внутри сети |
| Inside global | IP хоста, как его видит внешний мир |
| Outside local/global | IP внешнего хоста (обычно совпадают) |

---

## Типы NAT

| Тип | Соотношение | Ключевая команда |
|-----|-------------|------------------|
| Static | 1:1 фиксированное | `ip nat inside source static <local> <global>` |
| Dynamic | N:M из пула | `ip nat inside source list <ACL> pool <POOL>` |
| PAT | N:1 через порты | `ip nat inside source list <ACL> interface <IF> overload` |

---

## Static NAT (Router4)

```cisco
interface Fa0/1
 ip nat inside
interface Fa0/0
 ip nat outside

ip nat inside source static 172.17.10.2 172.17.201.5
```

PC0 (172.17.10.2) всегда виден как 172.17.201.5.

---

## Dynamic NAT (Router3)

```cisco
interface Fa0/1
 ip nat inside
interface Fa0/0
 ip nat outside

access-list 1 permit 172.17.20.0 0.0.0.255
ip nat pool POOL_202 172.17.202.5 172.17.202.6 netmask 255.255.255.248
ip nat inside source list 1 pool POOL_202
```

Хосты из 172.17.20.0/24 получают адреса из пула динамически. Пул из 2 адресов = макс. 2 одновременных соединения.

---

## PAT (Router5)

```cisco
interface Fa0/1
 ip nat inside
interface Fa0/0
 ip nat outside

access-list 1 permit 172.17.30.0 0.0.0.255
ip nat inside source list 1 interface Fa0/0 overload
```

Все хосты используют один IP (172.17.203.2), различаются по портам. **overload** — ключевое слово для PAT.

---

## DHCP Relay

DHCPServer в другой подсети → нужен relay на Router5:

```cisco
interface Fa0/1
 ip helper-address 172.17.31.2
```

Роутер конвертирует broadcast DHCP → unicast на сервер.

---

## DNS

DNSServer (172.17.100.2) содержит записи:
- ftp.com → 172.17.100.3
- ftp.files.com → 172.17.100.3

Клиенты получают адрес DNS вручную или через DHCP.

---

## Полезные команды

```cisco
show ip nat translations    ! Текущие трансляции
show ip nat statistics      ! Статистика и пулы
show ip route               ! Таблица маршрутизации
```

# Packet Tracer — практика по сетям

Лабы по сетям, выполненные в Cisco Packet Tracer.

## Список лаб

| №   | Тема                      | Файл                               | Что проверяется            |
| --- | ------------------------- | ---------------------------------- | -------------------------- |
| 01  | 2 ПК + коммутатор         | [01-pc-switch.md](01-pc-switch.md) | L2-связность, ARP          |
| 02  | VLAN                      | 02-vlan.md                         | Сегментация L2             |
| 03  | Router-on-a-stick         | 03-router-on-a-stick.md            | Маршрутизация между VLAN   |
| 04  | Trunk между коммутаторами | 04-trunk-between-switches.md       | VLAN через trunk           |
| 05  | Статическая маршрутизация | 05-static-routing.md               | Маршруты между сетями      |
| 06  | NAT (PAT)                 | 06-nat-pat.md                      | Трансляция адресов         |
| 07  | Extended ACL              | 07-extended-acl.md                 | Фильтрация трафика         |
| 08  | DHCP                      | 08-dhcp.md                         | Автоматическая выдача IP   |
| 09  | OSPF                      | 09-ospf.md                         | Динамическая маршрутизация |

## Как запускать

1. Установить Cisco Packet Tracer.
2. Открыть `.pkt`-файл в Packet Tracer.
3. Следовать шагам в `.md`-файле.

## Что запомнить

- **Коммутатор — L2**, работает с MAC.
- **Роутер — L3**, работает с IP.
- **Access-порт** — одна VLAN, **trunk** — несколько.
- **TTL уменьшается на 1 на каждом роутере.**
- **NAT ≠ маршрутизация.** Нужен default route.
- **ACL** — правила permit/deny, сверху вниз.
- **DHCP** — DORA: Discover, Offer, Request, Acknowledge.
- **OSPF** — динамическая маршрутизация, area 0.
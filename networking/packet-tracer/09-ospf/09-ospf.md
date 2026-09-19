# 09. OSPF — динамическая маршрутизация

## Задача

Собрать сеть из 3 роутеров, настроить OSPF, чтобы PC0 пинговал PC1.
Убедиться, что роутеры сами обменялись маршрутами (без статических).

## Что понадобилось

- **Cisco Packet Tracer**
- **Устройства:** 2 PC, 2 Switch (2960), 3 Router (2911)
- **Время:** ~35 минут

## Топология
PC0 ── Switch0 ── Router0 ── Router1 ── Router2 ── Switch1 ── PC1
сеть A G0/1 G0/0 G0/0 G0/1 G0/0 G0/1 сеть B
192.168.1.1 10.0.0.1 10.0.0.2 10.0.1.1 10.0.1.2 192.168.3.1


**Сети:**
- **Сеть A** (за Router0): `192.168.1.0/24`, шлюз `192.168.1.1`.
- **Link1** (Router0 ↔ Router1): `10.0.0.0/30`.
- **Link2** (Router1 ↔ Router2): `10.0.1.0/30`.
- **Сеть B** (за Router2): `192.168.3.0/24`, шлюз `192.168.3.1`.

## Настройки

| Устройство | Интерфейс | IP           | Маска | Шлюз        |
| ---------- | --------- | ------------ | ----- | ----------- |
| PC0        | NIC       | 192.168.1.10 | /24   | 192.168.1.1 |
| PC1        | NIC       | 192.168.3.10 | /24   | 192.168.3.1 |
| Router0    | G0/1      | 192.168.1.1  | /24   | —           |
| Router0    | G0/0      | 10.0.0.1     | /30   | —           |
| Router1    | G0/0      | 10.0.0.2     | /30   | —           |
| Router1    | G0/1      | 10.0.1.1     | /30   | —           |
| Router2    | G0/0      | 10.0.1.2     | /30   | —           |
| Router2    | G0/1      | 192.168.3.1  | /24   | —           |

## Ход работы

1. Собрал топологию: 3 роутера, 2 коммутатора, 2 ПК.
2. Настроил IP на всех интерфейсах.
3. Поднял интерфейсы (`no shutdown`).
4. Проверил `show ip route` — только connected маршруты.
5. Включил OSPF на всех трёх роутерах (`router ospf 1`).
6. Анонсировал сети в OSPF (`network ... area 0`).
7. Проверил `show ip ospf neighbor` — соседи установлены (State FULL).
8. Проверил `show ip route` — появились OSPF-маршруты (`O`).
9. Проверил ping PC0 → PC1 — работает, TTL=125.

## Команды / настройки

### IP на роутерах

**Router0:**
```
interface Gig0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
interface Gig0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit
```

**Router1:**
```
interface Gig0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit
interface Gig0/1
ip address 10.0.1.1 255.255.255.252
no shutdown
exit
```

**Router2:**
```
interface Gig0/0
ip address 10.0.1.2 255.255.255.252
no shutdown
exit
interface Gig0/1
ip address 192.168.3.1 255.255.255.0
no shutdown
exit
```


**Проверка:** `show ip route` — только `C` (connected). **Статических маршрутов нет.**

### OSPF на роутерах

**Router0:**
```
configure terminal

router ospf 1
network 192.168.1.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0
exit

exit
write memory
```


**Router1:**
```
configure terminal

router ospf 1
network 10.0.0.0 0.0.0.3 area 0
network 10.0.1.0 0.0.0.3 area 0
exit

exit
write memory
```


**Router2:**
```
configure terminal

router ospf 1
network 10.0.1.0 0.0.0.3 area 0
network 192.168.3.0 0.0.0.255 area 0
exit

exit
write memory
```


**Разбор команд:**

- **`router ospf 1`** — включаем OSPF, процесс №1 (номер произвольный).
- **`network <сеть> <wildcard> area 0`** — какие сети **анонсировать** в OSPF.
  - **Wildcard** — **инверсия маски** (`255.255.255.0` → `0.0.0.255`, `255.255.255.252` → `0.0.0.3`).
  - **`area 0`** — все роутеры в одной зоне (backbone). В простых сетях — все в `area 0`.

### Проверка соседей OSPF

`show ip ospf neighbor`


Ожидаемый вывод (на Router1):
```
Neighbor ID Pri State Dead Time Address Interface
192.168.1.1 1 FULL/DR 00:00:38 10.0.0.1 GigabitEthernet0/0
192.168.3.1 1 FULL/BDR 00:00:30 10.0.1.2 GigabitEthernet0/1
```


- **State: FULL** — соседство установлено.
- **DR/BDR** — Designated Router / Backup DR.

### Проверка таблицы маршрутизации

`show ip route`


Ожидаемый вывод (на Router1):
```
C 10.0.0.0/30 is directly connected, GigabitEthernet0/0
C 10.0.1.0/30 is directly connected, GigabitEthernet0/1
O 10.0.1.0/30 [110/2] via 10.0.0.2
C 192.168.1.0/24 is directly connected, GigabitEthernet0/1
O 192.168.3.0/24 [110/3] via 10.0.0.2
```


**`O`** — маршрут, полученный по OSPF. **`[110/3]`** — `[AD/метрика]`:
- **110** — административная дистанция OSPF.
- **3** — метрика (cost) до сети B.

### Проверка с PC0

**PC0 → PC1:**

`ping 192.168.3.10`

Результат: **0% потерь, TTL=125** ✅ (3 роутера)

**tracert:**

`tracert 192.168.3.10`

Результат:

1. 192.168.1.1 (Router0)

2. 10.0.0.2 (Router1)

3. 10.0.1.2 (Router2)

4. 192.168.3.10 (PC1)
Trace complete.


## Результат

| Проверка                   | Результат                       |
| -------------------------- | ------------------------------- |
| `show ip ospf neighbor`    | 2 соседа на Router1, State FULL |
| `show ip route` на Router1 | O 192.168.3.0/24 [110/3]        |
| ping PC0 → PC1             | OK, TTL=125                     |
| tracert                    | Путь через 3 роутера            |

**OSPF работает** — роутеры сами обменялись маршрутами. Без статических.

## Грабли

- **Забыл `no shutdown`** — интерфейсы в статусе `administratively down`.
- **Неверный wildcard** — OSPF не анонсирует сеть.
- **Разные area** — соседи не устанавливаются.
- **`network` с обычной маской** вместо wildcard — ошибка.
- **TTL=125** — пакет прошёл через 3 роутера.

## Что запомнить

- **OSPF** — динамическая маршрутизация, роутеры сами обмениваются маршрутами.
- **`router ospf 1`** — включение.
- **`network <сеть> <wildcard> area 0`** — анонс сети.
- **Wildcard** = инверсия маски.
- **`area 0`** — backbone-зона.
- **`show ip ospf neighbor`** — соседи.
- **`show ip route`** — маршруты `O` (OSPF).
- **`[110/3]`** — `[AD/метрика]`. AD OSPF = 110, метрика = cost.
- **TTL уменьшается на 1** на каждом роутере.
- **`tracert`** — показывает путь через роутеры.
- **OSPF vs Static:** OSPF масштабируется, static — нет.
- **OSPF vs RIP:** OSPF быстрее, не ограничен 15 хопами.

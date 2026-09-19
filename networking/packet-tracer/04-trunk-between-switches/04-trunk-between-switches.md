# 04. Trunk между коммутаторами

## Задача

Соединить 2 коммутатора trunk-портом, протянуть VLAN 10 и 20.
Проверить L2-связность внутри одной VLAN между разными коммутаторами.

## Что понадобилось

- **Cisco Packet Tracer**
- **Устройства:** 4 PC, 2 Switch (2960-24TT)
- **Время:** ~25 минут

## Топология
Switch2 ──trunk── Switch3
│ │
PC4 (VLAN 10) PC6 (VLAN 10)
PC5 (VLAN 20) PC7 (VLAN 20)


**Схема портов:**
- Switch2 `Gig0/1` ═══ `Gig0/1` Switch3 (trunk)
- Switch2 `Fa0/1` → PC4 (VLAN 10)
- Switch2 `Fa0/2` → PC5 (VLAN 20)
- Switch3 `Fa0/1` → PC6 (VLAN 10)
- Switch3 `Fa0/2` → PC7 (VLAN 20)

## Настройки ПК

| ПК  | IP            | Маска         | VLAN | Коммутатор |
| --- | ------------- | ------------- | ---- | ---------- |
| PC4 | 192.168.10.10 | 255.255.255.0 | 10   | Switch2    |
| PC5 | 192.168.20.10 | 255.255.255.0 | 20   | Switch2    |
| PC6 | 192.168.10.20 | 255.255.255.0 | 10   | Switch3    |
| PC7 | 192.168.20.20 | 255.255.255.0 | 20   | Switch3    |

**Шлюз не нужен** — проверяем L2-связность внутри VLAN.

## Ход работы

1. Собрал топологию: 2 коммутатора + 4 ПК.
2. Соединил коммутаторы кабелем (Gig0/1 ↔ Gig0/1).
3. Создал VLAN 10 и VLAN 20 на **обоих** коммутаторах.
4. Назначил access-порты для ПК на каждом коммутаторе.
5. Настроил trunk-порт на обоих коммутаторах.
6. Проверил `show interfaces trunk` — trunk работает.
7. Проверил ping PC4 → PC6 (VLAN 10, разные коммутаторы) — работает.
8. Проверил ping PC5 → PC7 (VLAN 20, разные коммутаторы) — работает.
9. Проверил ping PC4 → PC5 (разные VLAN) — timeout.

## Команды / настройки

### На Switch2
```
enable
configure terminal

! Создаём VLAN
vlan 10
name Sales
exit
vlan 20
name Accounting
exit

! Access-порты для ПК
interface fa0/1
switchport mode access
switchport access vlan 10
exit

interface fa0/2
switchport mode access
switchport access vlan 20
exit

! Trunk-порт к Switch3
interface Gig0/1
switchport mode trunk
exit

exit
write memory
```


### На Switch3
```
enable
configure terminal

! Создаём VLAN (те же, что на Switch2)
vlan 10
name Sales
exit
vlan 20
name Accounting
exit

! Access-порты
interface fa0/1
switchport mode access
switchport access vlan 10
exit

interface fa0/2
switchport mode access
switchport access vlan 20
exit

! Trunk-порт к Switch2
interface Gig0/1
switchport mode trunk
exit

exit
write memory
```

**Важно:** VLAN нужно создавать на **каждом** коммутаторе. Иначе trunk не пропустит трафик этой VLAN.

### Проверка на коммутаторах

`show interfaces trunk`

Ожидаемый вывод:
```
Port Mode Encapsulation Status Native vlan
Gig0/1 on 802.1q trunking 1

Vlans allowed and active: 1,10,20
```


### Проверка с ПК

**PC4 → PC6 (VLAN 10, разные коммутаторы):**
`ping 192.168.10.20`

Результат: **0% потерь, TTL=128** ✅

**PC5 → PC7 (VLAN 20, разные коммутаторы):**

`ping 192.168.20.20`

Результат: **0% потерь, TTL=128** ✅

**PC4 → PC5 (разные VLAN):**

`ping 192.168.20.10`

Результат: **Timeout, 100% потерь** ❌

## Результат

| Проверка                                | Результат                            |
| --------------------------------------- | ------------------------------------ |
| PC4 → PC6 (VLAN 10, разные коммутаторы) | Ping OK, TTL=128                     |
| PC5 → PC7 (VLAN 20, разные коммутаторы) | Ping OK, TTL=128                     |
| PC4 → PC5 (разные VLAN)                 | Timeout                              |
| `show interfaces trunk`                 | Gig0/1 — trunk, 802.1q, VLAN 1,10,20 |

**TTL=128** — трафик **не проходил через роутер**. Только через коммутаторы (L2). TTL не меняется на L2.

## Грабли

- **VLAN нужно создавать на каждом коммутаторе.** Если на Switch3 нет VLAN 10 — trunk её не пропустит.
- **Trunk-порт должен быть на обоих концах.** Если один конец access, другой trunk — не заработает.
- **Native VLAN** должна совпадать на обоих концах (по умолчанию VLAN 1).
- **TTL=128** — не меняется, потому что нет роутера.

## Что запомнить

- **Trunk-порт** — пропускает **несколько VLAN** по одному кабелю.
- **Access-порт** — одна VLAN, для конечного устройства.
- **802.1Q** — стандарт тегирования VLAN (тег в кадре).
- **Одна VLAN на разных коммутаторах** — одна логическая сеть (L2-связность есть).
- **VLAN нужно создавать на каждом коммутаторе.**
- **Разные VLAN** — изолированы, нужен роутер (L3).
- **TTL не меняется на L2** — только на роутерах.

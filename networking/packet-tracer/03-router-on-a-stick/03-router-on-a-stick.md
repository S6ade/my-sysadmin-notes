# 03. Router-on-a-Stick: маршрутизация между VLAN

## Задача

Добавить роутер в топологию VLAN, настроить маршрутизацию между
VLAN 10 и VLAN 20. Проверить, что PC0 (VLAN 10) пингует PC2 (VLAN 20).

## Что понадобилось

- **Cisco Packet Tracer**
- **Устройства:** 4 PC, 1 Switch (2960), 1 Router (2911)
- **Время:** ~30 минут

## Топология
Router0 (2911)
│ trunk
Switch0 (2960)
/ | |
PC0 PC1 PC2 PC3
(VLAN 10) (VLAN 20)


**Схема портов:**
- Router0 `G0/0` → Switch0 `Gig0/1` (trunk)
- Switch0 `Fa0/1` → PC0 (VLAN 10)
- Switch0 `Fa0/2` → PC1 (VLAN 10)
- Switch0 `Fa0/3` → PC2 (VLAN 20)
- Switch0 `Fa0/4` → PC3 (VLAN 20)

## Настройки ПК

| ПК  | IP            | Маска         | Шлюз         | VLAN |
| --- | ------------- | ------------- | ------------ | ---- |
| PC0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 10   |
| PC1 | 192.168.10.20 | 255.255.255.0 | 192.168.10.1 | 10   |
| PC2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 20   |
| PC3 | 192.168.20.20 | 255.255.255.0 | 192.168.20.1 | 20   |

**Шлюз** — это IP подынтерфейса роутера в этой VLAN.

## Ход работы

1. Добавил Router0 (2911) в топологию.
2. Соединил Router0 `G0/0` с Switch0 `Gig0/1`.
3. Настроил IP и шлюзы на всех ПК.
4. Настроил порт коммутатора к роутеру как **trunk**.
5. Настроил на роутере **sub-interfaces** для VLAN 10 и 20.
6. Проверил `show ip route` на роутере — два connected маршрута.
7. Проверил `show interfaces trunk` на коммутаторе — trunk работает.
8. Проверил ping PC0 → PC2 — работает (TTL=127).

## Команды / настройки

### На коммутаторе (Switch0)
```
enable
configure terminal

! Порт к роутеру — trunk
interface Gig0/1
switchport mode trunk
exit

exit
write memory
```

**Порты к ПК** — настроены как access в лабе 02 (VLAN 10 и 20).

### На роутере (Router0)
```
enable
configure terminal

! Основной интерфейс
interface Gig0/0
no shutdown
exit

! Подынтерфейс для VLAN 10
interface Gig0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
exit

! Подынтерфейс для VLAN 20
interface Gig0/0.20
encapsulation dot1q 20
ip address 192.168.20.1 255.255.255.0
exit

exit
write memory
```

**Разбор:**
- **`interface G0/0.10`** — создаём подынтерфейс `.10`.
- **`encapsulation dot1q 10`** — этот подынтерфейс обрабатывает трафик VLAN 10 (тег 802.1Q = 10).
- **`ip address 192.168.10.1 255.255.255.0`** — IP шлюза для VLAN 10.

### Проверка на роутере
`show ip route`


Ожидаемый вывод:
```
C 192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L 192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
C 192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L 192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

### Проверка на коммутаторе

`show interfaces trunk`


Ожидаемый вывод:

```
Port Mode Encapsulation Status Native vlan
Gig0/1 on 802.1q trunking 1

Vlans allowed and active: 1,10,20
```

### Проверка с ПК

**PC0 → PC1 (одна VLAN):**
`ping 192.168.10.20`

Результат: **0% потерь** ✅

**PC0 → PC2 (разные VLAN):**
`ping 192.168.20.10`

Результат: **0% потерь, TTL=127** ✅ — теперь работает!

## Результат

| Проверка                 | Результат              |
| ------------------------ | ---------------------- |
| PC0 → PC1 (VLAN 10)      | Ping OK                |
| PC2 → PC3 (VLAN 20)      | Ping OK                |
| PC0 → PC2 (VLAN 10 → 20) | Ping OK, **TTL=127**   |
| `show ip route`          | Два connected маршрута |
| `show interfaces trunk`  | Gig0/1 — trunk, 802.1q |

**TTL=127** — доказательство, что пакет прошёл через **роутер** (TTL уменьшился на 1). Это L3-маршрутизация.

## Грабли

- **Забыл `no shutdown` на `G0/0`** — подынтерфейсы не поднимутся.
- **Забыл `encapsulation dot1q`** — роутер не поймёт, какой трафик к какой VLAN.
- **Порт коммутатора к роутеру не trunk** — трафик VLAN не пройдёт.
- **Неверный шлюз на ПК** — ПК не знает, куда отправлять пакеты за пределы подсети.
- **TTL=127, а не 128** — пакет прошёл через роутер.

## Что запомнить

- **Router-on-a-stick** — маршрутизация между VLAN через **один физический интерфейс** роутера.
- **Sub-interface** — виртуальный интерфейс, по одному на VLAN.
- **`encapsulation dot1q N`** — тег VLAN на подынтерфейсе.
- **Trunk-порт** между коммутатором и роутером — пропускает все VLAN.
- **Шлюз ПК = IP подынтерфейса** роутера в этой VLAN.
- **TTL уменьшается на 1** на каждом роутере — доказательство L3.
- **Альтернатива** — L3-коммутатор (SVI). В современных сетях чаще используется.

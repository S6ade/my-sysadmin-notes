# 06. NAT (PAT)

## Задача

Настроить PAT на Router0, чтобы ПК внутренней сети (192.168.1.0/24)
выходили в «интернет» через один публичный IP (200.0.0.1).

## Что понадобилось

- **Cisco Packet Tracer**
- **Устройства:** 2 PC, 1 Switch (2960), 1 Router (2911), 1 Server (Server0)
- **Время:** ~30 минут

## Топология

PC0 ─┐
├── Switch0 ── Router0 ── Server0
PC1 ─┘ G0/1 G0/0
192.168.1.1 200.0.0.1
Server0: 200.0.0.2


**Сети:**
- **Внутренняя** (за Router0 `G0/1`): `192.168.1.0/24`, шлюз `192.168.1.1`.
- **Внешняя** (link между Router0 и Server0): `200.0.0.0/24`.

## Настройки

| Устройство | Интерфейс | IP           | Маска | Шлюз        |
| ---------- | --------- | ------------ | ----- | ----------- |
| PC0        | NIC       | 192.168.1.10 | /24   | 192.168.1.1 |
| PC1        | NIC       | 192.168.1.20 | /24   | 192.168.1.1 |
| Router0    | G0/1      | 192.168.1.1  | /24   | —           |
| Router0    | G0/0      | 200.0.0.1    | /24   | —           |
| Server0    | NIC       | 200.0.0.2    | /24   | 200.0.0.1   |

## Ход работы

1. Собрал топологию: PC0, PC1, Switch0, Router0, Server0.
2. Настроил IP на всех устройствах.
3. На Router0 настроил `ip nat inside` на `G0/1`, `ip nat outside` на `G0/0`.
4. Создал ACL для внутренней сети.
5. Настроил PAT через `ip nat inside source list 1 interface G0/0 overload`.
6. Добавил default route к Server0.
7. Проверил ping PC0 → Server0 — работает.
8. Проверил `show ip nat translations` — таблица трансляций.

## Команды / настройки

### На Router0
```
enable
configure terminal

! Внутренний интерфейс
interface Gig0/1
ip address 192.168.1.1 255.255.255.0
ip nat inside
no shutdown
exit

! Внешний интерфейс (публичный)
interface Gig0/0
ip address 200.0.0.1 255.255.255.0
ip nat outside
no shutdown
exit

! ACL — какие адреса транслировать
access-list 1 permit 192.168.1.0 0.0.0.255

! PAT (NAT Overload)
ip nat inside source list 1 interface Gig0/0 overload

! Default route — куда отправлять всё остальное
ip route 0.0.0.0 0.0.0.0 200.0.0.2

exit
write memory
```


**Разбор команд:**

- **`ip nat inside`** — интерфейс, смотрящий во внутреннюю сеть.
- **`ip nat outside`** — интерфейс, смотрящий в интернет.
- **`access-list 1 permit 192.168.1.0 0.0.0.255`** — ACL, разрешающий трафик из внутренней сети.
- **`ip nat inside source list 1 interface G0/0 overload`** — главная команда NAT:
  - `source list 1` — какие адреса транслировать (из ACL 1).
  - `interface G0/0` — через какой интерфейс (публичный IP).
  - `overload` — **PAT**, много адресов через один IP.
- **`ip route 0.0.0.0 0.0.0.0 200.0.0.2`** — default route к Server0.

### Проверка на Router0
`show ip nat translations`


Ожидаемый вывод:
```
Pro Inside global Inside local Outside local Outside global
icmp 200.0.0.1:17 192.168.1.10:17 200.0.0.2:17 200.0.0.2:17
icmp 200.0.0.1:18 192.168.1.10:18 200.0.0.2:18 200.0.0.2:18
...
```


**Что здесь происходит:**
- **Inside local** — внутренний IP и порт PC0 (`192.168.1.10:17`).
- **Inside global** — публичный IP и порт, подменённый Router0 (`200.0.0.1:17`).
- **Outside global** — реальный IP и порт Server0 (`200.0.0.2:17`).

`show ip nat statistics`


Покажет:
- Total active translations
- Hits
- Misses

### Проверка с PC0

**PC0 → Server0:**

`ping 200.0.0.2`

Результат: **0% потерь** ✅

**Server0 видит пакет от `200.0.0.1`, а не от `192.168.1.10`.** Это и есть NAT.

## Результат

| Проверка                   | Результат                        |
| -------------------------- | -------------------------------- |
| PC0 → Server0 (200.0.0.2)  | Ping OK, 0% потерь               |
| PC1 → Server0              | Ping OK                          |
| `show ip nat translations` | Таблица трансляций заполнена     |
| `show ip nat statistics`   | Total active translations > 0    |
| Server0 видит пакет        | От 200.0.0.1, не от 192.168.1.10 |

**PAT работает** — много внутренних IP через один публичный, по портам.

## Грабли

- **Забыл default route** — Router0 не знает, куда отправлять пакеты к Server0. Ошибка `Destination host unreachable`.
- **Маска `/16` на Server0** — шлюз оказался вне подсети. Исправил на `/24`.
- **Забыл `ip nat inside` / `ip nat outside`** — NAT не работает.
- **Забыл ACL** — нечего транслировать.
- **Server0 не в сети `200.0.0.0/24`** — если Server0 `8.8.8.8`, нужен второй роутер (ISP).

## Что запомнить

- **NAT** — трансляция IP-адресов.
- **PAT (Overload)** — много внутренних → один публичный, по портам.
- **`ip nat inside` / `ip nat outside`** — какие интерфейсы внутренний/внешний.
- **ACL** — какие адреса транслировать.
- **`ip nat inside source list <ACL> interface <iface> overload`** — команда PAT.
- **`show ip nat translations`** — таблица трансляций.
- **`show ip nat statistics`** — статистика NAT.
- **NAT ≠ маршрутизация.** Нужен default route.
- **Внутренние адреса скрыты** от внешнего мира — плюс безопасности.
- **NAT — временное решение** для нехватки IPv4. IPv6 решает нативно.

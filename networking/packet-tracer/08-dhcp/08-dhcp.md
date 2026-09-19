# 08. DHCP на роутере

## Задача

Настроить DHCP-сервер на Router0, чтобы ПК получали IP автоматически.
Проверить через `ipconfig /renew` на ПК и `show ip dhcp binding` на роутере.

## Что понадобилось

- **Cisco Packet Tracer**
- **Устройства:** 2 PC, 1 Switch (2960), 1 Router (2911)
- **Время:** ~25 минут

## Топология

PC0 ─┐
├── Switch0 ── Router0
PC1 ─┘ G0/1
192.168.1.1
DHCP-сервер


**Сеть:** `192.168.1.0/24`, шлюз `192.168.1.1` (Router0).

## Настройки

| Устройство | Интерфейс | IP          | Маска |
| ---------- | --------- | ----------- | ----- |
| PC0        | NIC       | DHCP        | —     |
| PC1        | NIC       | DHCP        | —     |
| Router0    | G0/1      | 192.168.1.1 | /24   |

**ПК настраиваются на DHCP** — IP получат автоматически.

## Ход работы

1. Собрал топологию: PC0, PC1, Switch0, Router0.
2. Настроил IP на `G0/1` роутера: `192.168.1.1/24`.
3. Исключил адреса `.1`–`.10` из пула DHCP.
4. Создал DHCP-пул `LAN` с сетью `192.168.1.0/24`.
5. Указал шлюз и DNS для клиентов.
6. Переключил ПК на DHCP.
7. Проверил `ipconfig /all` на PC0 — получил IP.
8. Проверил `show ip dhcp binding` на Router0 — таблица выданных адресов.

## Команды / настройки

### На Router0
```
enable
configure terminal

! Интерфейс — шлюз для сети
interface Gig0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

! Исключаем адреса, которые НЕ выдаём
ip dhcp excluded-address 192.168.1.1 192.168.1.10

! Создаём пул DHCP
ip dhcp pool LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
exit

exit
write memory
```


**Разбор команд:**

- **`ip dhcp excluded-address 192.168.1.1 192.168.1.10`** — адреса, которые **не выдаются**:
  - `.1` — сам роутер (шлюз).
  - `.2`–`.10` — зарезервированы под серверы, принтеры, статические устройства.
- **`ip dhcp pool LAN`** — имя пула (любое).
- **`network 192.168.1.0 255.255.255.0`** — какая сеть.
- **`default-router 192.168.1.1`** — шлюз, который выдаётся клиентам.
- **`dns-server 8.8.8.8`** — DNS-сервер (можно указать локальный).

**Примечание:** команда `lease` в Packet Tracer **не поддерживается**. В реальном IOS:

`lease 0 1`

(0 дней, 1 час). Без неё время аренды — по умолчанию 1 день.

### На ПК

**На PC0 и PC1:**
- **Desktop → IP Configuration**
- Выбрать **DHCP** (вместо Static).
- Через пару секунд ПК получит IP.

### Проверка на Router0

`show ip dhcp binding`

Ожидаемый вывод:
```
IP address Client-ID/Hardware address Lease expiration Type
192.168.1.11 0001.43B2.1234 ... Automatic
192.168.1.12 0002.16A1.5678 ... Automatic
```


`show ip dhcp pool`
Ожидаемый вывод:
```
Pool LAN :
Utilization mark (high/low) : 100 / 0
Subnet size (first/next) : 0 / 0
Total addresses : 254
Leased addresses : 2
Excluded addresses : 10
Pending event : none

1 subnet is currently in the pool
Current index IP address range Leased/Excluded/Total
192.168.1.1 192.168.1.1 - 192.168.1.254 2 / 10 / 254
```

`show ip dhcp conflict`


Покажет конфликты (если есть).

### Проверка с PC0

`ipconfig /all`

Ожидаемый вывод:
```
IPv4 Address : 192.168.1.11
Subnet Mask : 255.255.255.0
Default Gateway : 192.168.1.1
DHCP Servers : 192.168.1.1
DNS Servers : 8.8.8.8
```

`ipconfig /renew`


ПК **перезапросит** IP у DHCP. В Simulation Mode видно DORA.

## Результат

| Проверка               | Результат                          |
| ---------------------- | ---------------------------------- |
| PC0 `ipconfig /all`    | IP 192.168.1.11, DHCP Enabled: Yes |
| PC1 `ipconfig /all`    | IP 192.168.1.12                    |
| `show ip dhcp binding` | Записи для PC0 и PC1               |
| `show ip dhcp pool`    | Leased addresses: 2                |
| ping PC0 → 192.168.1.1 | Ping OK, TTL=255                   |

**DHCP работает** — ПК получают IP автоматически из пула.

## Грабли

- **Команда `lease 0 0 30` не работает в Packet Tracer** — не поддерживается.
  Пропусти `lease` — DHCP заработает без неё, время аренды 1 день.
- **PC не получает IP, если оставлен Static** — переключи на DHCP.
- **IP `0.0.0.0`** в `ipconfig /all` — DHCP не сработал (или не включён на ПК).
- **`excluded-address` слишком большой** — все адреса исключены, DHCP не может выдать.
- **Пул с неправильной сетью** — не совпадает с интерфейсом роутера.

## Что запомнить

- **DHCP** — автоматическая выдача IP, маски, шлюза, DNS.
- **DORA:** Discover (broadcast) → Offer → Request (broadcast) → Acknowledge.
- **`ip dhcp excluded-address`** — адреса, которые не выдаются (шлюз, серверы).
- **`ip dhcp pool`** — пул адресов.
- **`default-router`** — шлюз, который выдаётся клиентам.
- **`dns-server`** — DNS, который выдаётся клиентам.
- **`show ip dhcp binding`** — таблица выданных адресов.
- **`show ip dhcp pool`** — статистика пула.
- **`ipconfig /renew`** — перезапросить IP.
- **DHCP Relay (`ip helper-address`)** — для нескольких подсетей через один сервер.
- **В домене AD DHCP должен выдавать IP контроллера домена как DNS.**
- **DHCP авторизуется в AD** — защита от поддельных DHCP-серверов.

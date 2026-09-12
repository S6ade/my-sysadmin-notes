# 01. Установка контроллера домена

## Задача
Поднять DC с ролью AD DS + DNS в VirtualBox, создать лес company.local.

## Что понадобилось
- VirtualBox 7.x + ISO Windows Server 2025 Evaluation (Microsoft Eval Center)
- VM: 2 vCPU, 4 ГБ RAM, 50 ГБ диск
- Сеть: NAT Network `ad-lab-net`, CIDR 10.10.10.0/24

## Ход работы
1. Создал VM в VirtualBox, отключил Unattended Installation
2. Установил Windows Server 2025 Standard Evaluation (Desktop Experience)
3. Переименовал сервер в DC01
4. Задал статический IP: 10.10.10.10/24, шлюз 10.10.10.1, DNS 10.10.10.10
5. Установил роли AD DS + DNS через Server Manager
6. Повысил сервер до DC, создал новый лес company.local
7. Задал DSRM-пароль, дождался перезагрузки

## Команды / настройки
- `ipconfig /all` — проверка IP и DNS
- `echo %USERDOMAIN%` → COMPANY
- `hostname` → DC01
- `nslookup -type=SRV _ldap._tcp.dc._msdcs.company.local`
  → priority 0, weight 100, port 389, host dc01.company.local
- DNS-зона company.local с папками _msdcs, _sites, _tcp, _udp

## Результат
- Домен company.local создан
- AD Users and Computers открывается, домен виден
- DNS Manager показывает зону company.local
- SRV-запись _ldap корректна — клиенты смогут найти DC

## Грабли
- Ctrl+Alt+Del в VirtualBox перехватывается хостом — использовать
  Input → Keyboard → Insert Ctrl+Alt+Del или Host+Del
- При выборе редакции важно было взять Desktop Experience, а не Core
- Unattended Installation в VirtualBox пропускает ручную настройку —
  отключил для полноценного прохождения установки
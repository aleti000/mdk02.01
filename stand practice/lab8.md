# Лабораторное занятие №8: Настройка rsyslog

## Тема
Настройка rsyslog как центрального сервера сбора логов в ALT Linux

## Цель
Освоить настройку rsyslog-сервера на srv2; отправку warning+ с gateway/SRV1/SRV2, error+ с CLI-home/office.

## Схема сети

![alt text](pic/network02.jpg)

## Теоретические основы

### rsyslog в ALT Linux

rsyslog — демон системного логирования. Конфиг /etc/rsyslog.conf, /etc/rsyslog.d/.

- **Сервер:** imudp/imtcp для приема UDP/TCP 514.
- **Клиент:** omrelp/omfwd для отправки.
- **Правила:** facility.priority e.g. *.warning @@IP:514

| Facility | Описание |
|----------|----------|
| auth | Аутентификация |
| cron | Планировщик |
| kern | Ядро |
| user | Пользовательские процессы |
| mail | Почта |

| Priority | Уровень |
|----------|---------|
| err | Ошибки |
| warning | Предупреждения |
| info | Инфо |
| debug | Отладка |

### Методические указания

### Подготовка
1. Сеть lab1.
2. `apt-get update && apt-get install rsyslog` на всех.

### Рекомендации
- TCP > UDP.
- Firewall: iptables -A INPUT -p udp --dport 514 -j ACCEPT

### Ход работы

### srv2 (сервер)
```bash
# Config /etc/rsyslog.conf добавить:
$ModLoad imudp
$UDPServerRun 514
$ModLoad imtcp
$InputTCPServerRun 514
$template RemoteLogs,"/var/log/remote/%HOSTNAME%-%PROGRAMNAME%.log"
local0.* ?RemoteLogs
*.* /var/log/remote/%HOSTNAME%.log
systemctl restart rsyslog
iptables -A INPUT -p udp --dport 514 -j ACCEPT
iptables -A INPUT -p tcp --dport 514 -j ACCEPT
```

### Clients (gateway/SRV1 det):
```bash
# /etc/rsyslog.conf добавить в конец:
*.warning @@172.16.0.3:514
systemctl restart rsyslog
logger -p user.warning "Test warning"
```

### CLI-home/office (само):
*.err @@172.16.0.3:514

## Задание
1. srv2: rsyslog server с remote logs.
2. gateway/SRV1/SRV2: отправлять только логи уровня warning+ to srv2.
3. CLI-home/office: отправлять только логи уровня error+ to srv2.
Критерии: tail /var/log/remote/gateway.log shows warnings.

## Полезные команды
```bash
systemctl status rsyslog
tail -f /var/log/remote/*
logger -p user.err "Test"
iptables -L | grep 514

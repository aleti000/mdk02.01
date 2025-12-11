# Лабораторное занятие №8: Настройка rsyslog

## Тема
Настройка rsyslog как центрального сервера сбора логов в ALT Linux

## Цель
Освоить настройку rsyslog-сервера на srv2; отправку warning+ с gateway/SRV1/SRV2, error+ с CLI-home/office.

## Схема сети

![alt text](https://github.com/aleti000/mdk02.01/blob/main/pic/network02.jpg)

## Теоретические основы
# Ознакомится со статьей по адресу: https://www.altlinux.org/Journald
### rsyslog в ALT Linux

rsyslog — современный демон системного логирования в ALT Linux (замена sysklogd).

**История развития:**
- **syslog (BSD, 1980):** Базовый UDP-only протокол.
- **sysklogd (Linux, 1990s):** Стандарт Linux до 2008, ограничен (UDP, простые правила).
- **syslog-ng (2000, BalaBit):** Расширения (TCP, фильтры, DB).
- **rsyslog (2004, Rainer Gerhards):** Форк syslog-ng, высокопроизводительный, scripting (RELP), Elasticsearch output, ALT default.

**Преимущества rsyslog:**
- По сравнению с sysklogd: TCP/RELp, сложные фильтры, templates, высокая нагрузка.
- По сравнению с syslog-ng: Простой синтаксис, быстрая обработка, встроенные драйверы (omkafka, omelasticsearch).

**Конфигурация в ALT Linux:**
- Основной файл: /etc/rsyslog.conf
- Дополнительные: /etc/rsyslog.d/*.conf
- Служба: systemctl rsyslog
- Порты: UDP/TCP 514

- **Сервер:** $ModLoad imudp; $UDPServerRun 514
- **Клиент:** *.warning @@server:514
- **Правила:** facility.priority (auth.err, *.warning); templates %HOSTNAME%-%$YEAR%.log

| Facility | Описание |
|----------|----------|
| auth | События аутентификации (login, su) |
| authpriv | Приватные события безопасности (пароли, PAM) |
| cron | События планировщика cron/at |
| daemon | Действия системных демонов (sshd, apache) |
| kern | Сообщения ядра (dmesg, драйверы) |
| lpr | Печать (CUPS, LPD) |
| mail | Почтовые события (Postfix, Sendmail) |
| mark | Маркеры времени (rsyslog internal) |
| news | News-серверы (NNTP) |
| security | Безопасность (alias для auth в старых sysklogd) |
| user | Пользовательские процессы (logger) |
| uucp | UUCP протокол (устаревший) |
| local0-local7 | Локальные приложения (8 пользовательских каналов) |

| Код | Priority | Уровень |
|-----|----------|---------|
| 0 | emerg | Системная авария |
| 1 | alert | Требует немедленного внимания |
| 2 | crit | Критические условия |
| 3 | err | Ошибки |
| 4 | warning | Предупреждения |
| 5 | notice | Важные условия |
| 6 | info | Информационные сообщения |
| 7 | debug | Отладочная информация |

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
2. SRV1: отправлять только логи уровня warning+ to srv2.
3. CLI-home/office: отправлять только логи уровня error+ to srv2.
Критерии: tail /var/log/remote/gateway.log shows warnings.

## Полезные команды
```bash
systemctl status rsyslog
tail -f /var/log/remote/*
logger -p user.err "Test"
iptables -L | grep 514

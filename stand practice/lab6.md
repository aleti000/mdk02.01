# Лабораторное занятие №6: Фильтрация пакетов с помощью iptables

## Тема
Настройка правил фильтрации сетевых пакетов

## Цель
Освоить настройку iptables для обеспечения безопасности сети

## Схема сети

См. лабораторную работу №1.

## Теоретические основы

### Основные понятия iptables

В Linux для фильтрации пакетов используется netfilter, управляемый через iptables.

- **Цепочки**: INPUT (входящие пакеты), OUTPUT (исходящие), FORWARD (транзитные)

- **Таблицы**: filter (фильтрация), nat (трансляция адресов), mangle (изменение пакетов)

- **Команды**: -A (добавить), -D (удалить), -I (вставить), -L (показать), -F (очистить), -P (политика)

- **Критерии отбора**: -p (протокол), -s (источник), -d (назначение), -i (входной интерфейс), -o (выходной), --sport/--dport (порты), -m state (состояние соединения)

- **Действия**: ACCEPT (принять), DROP (сбросить), REJECT (отклонить с ответом), LOG (записать в лог), RETURN (вернуться)

- **NAT преобразования**: SNAT (изменение источника), DNAT (изменение назначения), MASQUERADE (маскарадинг)

### Лабораторные работы из Block 6

- **Лабораторная работа А**: Использование критерия limit для ограничения количества пакетов

- **Лабораторная работа Б**: Использование действия LOG для логирования пакетов

- **Лабораторная работа В**: Настройка NAT преобразований

## Методические указания

- Выполняйте настройку на устройстве gateway
- Тестируйте правила с других устройств сети
- Сохраняйте правила для автоматической загрузки
- Используйте команды диагностики для проверки

## Ход работы

### 1. Настройка базового firewall на gateway

1. **Очистите существующие правила:**
   ```bash
   iptables -F
   iptables -t nat -F
   iptables -t mangle -F
   ```

2. **Установите политику по умолчанию:**
   ```bash
   iptables -P INPUT DROP
   iptables -P FORWARD DROP
   iptables -P OUTPUT ACCEPT
   ```

3. **Разрешите трафик на loopback интерфейсе:**
   ```bash
   iptables -A INPUT -i lo -j ACCEPT
   ```

4. **Разрешите установленные соединения:**
   ```bash
   iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
   ```

5. **Запретите недействительные пакеты:**
   ```bash
   iptables -A INPUT -m state --state INVALID -j DROP
   iptables -A FORWARD -m state --state INVALID -j DROP
   ```

6. **Проверьте правила:**
   ```bash
   iptables -L -v
   ```

### 2. Настройка NAT для выхода в интернет

1. **Включите IP forwarding:**
   ```bash
   echo 1 > /etc/net/sysctl.conf
   ```

2. **Настройте MASQUERADE:**
   ```bash
   iptables -t nat -A POSTROUTING -o **выходной интерфейс** -j MASQUERADE
   ```

3. **Проверьте NAT правила:**
   ```bash
   iptables -t nat -L -v
   ```

### 3. Разрешение необходимого трафика

1. **SSH доступ:**
   ```bash
   iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   ```

2. **DNS запросы:**
   ```bash
   iptables -A INPUT -p udp --dport 53 -j ACCEPT
   iptables -A INPUT -p tcp --dport 53 -j ACCEPT
   iptables -A FORWARD -p udp --dport 53 -j ACCEPT
   iptables -A FORWARD -p tcp --dport 53 -j ACCEPT
   ```

3. **ICMP пакеты:**
   ```bash
   iptables -A INPUT -p icmp -j ACCEPT
   iptables -A FORWARD -p icmp -j ACCEPT
   ```

4. **Разрешите трафик между сетями:**
   ```bash
   iptables -A FORWARD -i ens19 -o ens21 -j ACCEPT
   iptables -A FORWARD -i ens21 -o ens19 -j ACCEPT
   ```

### 4. Лабораторная работа А: Ограничение ICMP пакетов

1. **Добавьте правило с ограничением:**
   ```bash
   iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s --limit-burst 1 -j ACCEPT
   iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
   ```

2. **Тестируйте с другой машины:**
   ```bash
   ping -f gateway_ip -c 10
   ```

3. **Проверьте счетчики:**
   ```bash
   iptables -L -v -n
   ```

### 5. Лабораторная работа Б: Логирование пакетов

1. **Добавьте правило логирования:**
   ```bash
   iptables -A INPUT -p tcp --dport 23 -m limit --limit 1/m --limit-burst 1 -j LOG --log-prefix "Telnet attempt: "
   iptables -A INPUT -p tcp --dport 23 -j DROP
   ```

2. **Тестируйте попытку подключения:**
   ```bash
   telnet gateway_ip 23
   ```

3. **Проверьте логи:**
   ```bash
   dmesg | grep "Telnet attempt"
   ```

### 6. Сохранение и восстановление правил

1. **Сохраните правила:**
   ```bash
   iptables-save -f /etc/sysconfig/iptables
   ```

2. **Добавьте в автозагрузку:**
   ```bash
   systemctl enable iptables
   ```

## Задание

Настроить firewall на устройстве gateway для обеспечения безопасности сети согласно задачам лабораторной работы.

## Критерии выполнения работы

Работа считается выполненной если:
- Правила iptables настроены правильно
- Трафик между сетями разрешен/запрещен согласно правилам
- NAT работает для выхода в интернет
- ICMP пакеты ограничены
- Попытки доступа к запрещенным портам логируются
- Правила сохраняются и восстанавливаются

## Полезные команды для диагностики

```bash
# Просмотр правил
iptables -L -v -n
iptables -t nat -L -v -n

# Просмотр логов
dmesg | grep iptables

# Проверка соединений
netstat -tuln

# Тестирование
ping -c 4 8.8.8.8
curl google.com

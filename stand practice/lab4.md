# Лабораторное занятие №4: Установка домена samba-dc

## Тема
Установка домена samba-dc

## Цель
Освоить процесс развертывания контроллера домена Active Directory на базе Samba 4 в операционной системе ALT Linux, настройку доменной инфраструктуры, создание пользователей и групп, а также присоединение клиентских машин к домену

## Схема сети

![alt text](https://github.com/aleti000/mdk02.01/blob/main/pic/network02.jpg)

## Теоретические основы

### Samba Active Directory Domain Controller

Samba AD DC — реализация контроллера домена Microsoft Active Directory на базе открытого программного обеспечения Samba. Позволяет управлять пользователями, группами, политиками, аутентификацией и DNS в гетерогенной среде.

#### Основные компоненты Samba AD:
- **samba-tool** — консольная утилита для администрирования домена
- **Kerberos** — система аутентификации для доменных пользователей
- **LDAP** — хранилище объектов домена (пользователи, группы, компьютеры)
- **DNS** — интегрированная служба разрешения имен (SAMBA_INTERNAL)

#### Доменные понятия:
- **Realm** — область Kerberos (обычно совпадает с DNS-доменом в верхнем регистре)
- **Domain** — доменное имя (NetBIOS-имя)
- **Administrator** — учетная запись администратора домена

### Управление пользователями и группами

- **Пользователи домена** — учетные записи для аутентификации в домене
- **Группы домена** — коллекции пользователей для удобного управления правами
- **Организационные единицы (OU)** — контейнеры для группировки объектов

### Присоединение к домену

Для присоединения клиентских машин используются протоколы:
- **SSSD (System Security Services Daemon)** — для Linux-клиентов
- **Winbind** — альтернативный метод для Samba
- **Realmd** — утилита для автоматического присоединения

## Методические указания

### Подготовка к выполнению работы

1. **Изучите теоретический материал** о Samba Active Directory
2. **Проверьте сетевые настройки** согласно лабораторной работе №1
3. **Определите параметры домена** (имя, realm, пароль администратора)

### Рекомендации по выполнению

#### При настройке Samba DC:
- Используйте статические IP-адреса для DC
- Настройте время синхронизации (NTP)
- Создавайте резервные копии конфигурационных файлов
- Проверяйте синтаксис команд перед выполнением

#### При присоединении клиентов:
- Убедитесь в доступности DC по сети
- Проверьте правильность DNS-записей
- После присоединения перезагрузите клиентскую машину

#### Управление пользователями:
- Используйте сложные пароли для учетных записей
- Группируйте пользователей по ролям
- Документируйте созданные объекты

### Возможные проблемы и решения

| Проблема | Возможное решение |
|----------|-------------------|
| Ошибка провизии домена | Проверить параметры сети, очистить старые конфигурации |
| DNS не работает | Перезагрузить службы Samba, проверить файлы зон |
| Клиент не присоединяется | Проверить DNS-записи SRV, доступность портов Kerberos/LDAP |
| Пользователи не аутентифицируются | Проверить членство в группах, статус учетных записей |

## Ход работы

### Часть 1: Подготовка и установка контроллера домена на SRV1

#### Шаг 1. Настройка имени хоста и параметров сети

**На SRV1:**
```bash
# Установить имя хоста
hostnamectl set-hostname srv1.test.sa
exec bash

# Проверить IP-адрес (должен быть 172.16.0.2/24)
ip addr show

# Обновить систему
apt-get update
```

#### Шаг 2. Установка пакета Samba DC

**На SRV1:**
```bash
# Установить Samba DC
apt-get install -y task-samba-dc krb5-workstation

# Остановить и отключить конфликтующие службы
systemctl stop smb nmb winbind
systemctl disable smb nmb winbind

# Удалить старые конфигурации (если есть)
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba/*
rm -rf /var/cache/samba/*
```

#### Шаг 3. Провизия домена Samba с внутренним DNS

**На SRV1:**
```bash
# Выполнить провизию домена с внутренним DNS
samba-tool domain provision --realm=test.sa --domain=TEST --server-role=dc --dns-backend=SAMBA_INTERNAL --adminpass='P@ssw0rd'

# Параметры:
# --realm: Указывает Kerberos realm (TEST.SA)
# --domain: Указывает NetBIOS доменное имя (TEST)
# --server-role: dc (контроллер домена)
# --dns-backend: SAMBA_INTERNAL (встроенный DNS Samba)
# --adminpass: Пароль администратора

# Альтернативно, интерактивная провизия:
# samba-tool domain provision --use-rfc2307 --interactive
```

#### Шаг 4. Настройка локального резолвера

**На SRV1:**
```bash
# Настроить локальный DNS
echo "nameserver 127.0.0.1" > /etc/resolv.conf
echo "search test.sa" >> /etc/resolv.conf

# Копировать Kerberos конфигурацию домена
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

#### Шаг 5. Запуск и проверка доменных служб

**На SRV1:**
```bash
# Запустить службы Samba
systemctl enable samba
systemctl start samba

# Проверить статус
systemctl status samba

# Проверить домен
samba-tool domain info

# Проверить DNS-записи
host -t A test.sa
host -t SRV _kerberos._udp.test.sa
host -t SRV _ldap._tcp.test.sa

# Проверить зоны DNS
samba-tool dns zonelist srv1
samba-tool dns zoneinfo test.sa
```

### Часть 2: Присоединение CLI-office к домену

#### Шаг 6. Подготовка клиента CLI-office

**На CLI-office:**
```bash
# Обновить систему и установить пакеты
apt-get update
apt-get install -y task-auth-ad-sssd realmd krb5-workstation samba-client

# Настроить DNS для разрешения домена
echo "nameserver 172.16.0.2" > /etc/resolv.conf
echo "search test.sa" >> /etc/resolv.conf
```

#### Шаг 7. Присоединение к домену через sssd

**На CLI-office:**
```bash
# Обнаружить домен
realm discover test.sa

# Присоединиться к домену
realm join --user=administrator test.sa

# Ввести пароль администратора: P@ssw0rd
```

#### Шаг 8. Проверка присоединения

**На CLI-office:**
```bash
# Проверить статус домена
realm list

# Перезагрузить систему для применения настроек
reboot
```

**После перезагрузки на CLI-office:**
```bash
# Проверить членство в домене
realm list

# Проверить доступ к доменным ресурсам
getent passwd administrator@test.sa

# Тестировать аутентификацию Kerberos
kinit administrator@test.sa
klist

# Протестировать вход под доменным пользователем
su - administrator@test.sa
```

### Часть 3: Создание доменных пользователей и групп

#### Шаг 9. Создание пользователей

**На SRV1:**
```bash
# Создать нескольких доменных пользователей
samba-tool user create user1 'Password1!'
samba-tool user create user2 'Password2!'
samba-tool user create user3 'Password3!'

# Включить учетные записи (по умолчанию могут быть отключены)
samba-tool user enable user1
samba-tool user enable user2
samba-tool user enable user3

# Проверить создание пользователей
samba-tool user list
samba-tool user show user1
```

#### Шаг 10. Создание доменной группы

**На SRV1:**
```bash
# Создать группу домена
samba-tool group create "DomainUsers"

# Проверить создание группы
samba-tool group list
samba-tool group show "DomainUsers"
```

#### Шаг 11. Добавление пользователей в группу

**На SRV1:**
```bash
# Добавить пользователей в группу
samba-tool group addmembers "DomainUsers" user1,user2,user3

# Проверить членство в группе
samba-tool group listmembers "DomainUsers"
```

#### Шаг 12. Тестирование пользователей на клиенте

**На CLI-office:**
```bash
# Проверить возможность получения билета Kerberos
kinit user1@test.sa
# Ввести пароль: Password1!

# Проверить билет
klist

# Проверить группы пользователя
id user1@test.sa

# Попробовать войти под пользователем
su - user1@test.sa

# Проверить доступ к ресурсам Samba (если настроено)
smbclient -L //srv1.test.sa -U user1@test.sa
```

## Задание

**Настройка Samba Active Directory домена test.sa:**

1. **SRV1** (172.16.0.2) - установить контроллер домена Samba с внутренним DNS
2. **CLI-office** - присоединить клиентскую машину к домену test.sa
3. **Создать 3 доменных пользователя** - user1, user2, user3
4. **Создать доменную группу** - "DomainUsers"
5. **Добавить всех пользователей в группу "DomainUsers"**

**Критерии выполнения работы:**
Работа считается выполненной если:
- Samba DC на SRV1 работает и обслуживает DNS-запросы
- CLI-office успешно присоединен к домену test.sa
- Доменные пользователи могут аутентифицироваться на клиенте
- Пользователи принадлежат созданной группе
- DNS-записи домена корректно разрешаются

## Критерии выполнения работы

Работа считается выполненной если:

1. **Samba DC настроен** на SRV1 с внутренним DNS и функционирует корректно
2. **DNS-сервер работает** и разрешает доменные имена
3. **CLI-office присоединен** к домену test.sa через SSSD
4. **Доменные пользователи созданы** (user1, user2, user3) и могут входить в систему
5. **Доменная группа создана** ("DomainUsers") и содержит всех пользователей
6. **Аутентификация работает** на клиентской машине

## Возможные проблемы и решения

| Проблема | Возможное решение |
|----------|-------------------|
| Ошибка провизии домена | Проверить параметры, очистить /var/lib/samba |
| DNS не разрешает имена | Проверить samba-tool dns, перезагрузить samba |
| Клиент не присоединяется | Проверить DNS SRV-записи, доступность портов 88,389 |
| Пользователи отключены | Использовать samba-tool user enable |
| SSSD не работает | systemctl restart sssd, проверить логи |
| Kerberos не аутентифицирует | Проверить время на сервере и клиенте |

## Полезные команды для диагностики

```bash
# Проверка статуса служб
systemctl status samba sssd

# Диагностика домена
samba-tool domain info
samba-tool domain level

# Управление пользователями и группами
samba-tool user list
samba-tool user show <user>
samba-tool user enable <user>
samba-tool group list
samba-tool group listmembers "<group>"

# DNS-диагностика
host test.sa
host -t SRV _kerberos._udp.test.sa
samba-tool dns zonelist srv1
samba-tool dns query srv1 test.sa srv1 A

# Kerberos-диагностика
kinit administrator@TEST.SA
klist
kdestroy

# Клиентская диагностика
realm list
realm discover test.sa
getent passwd | grep test.sa
id <username>@test.sa
sssd --version-info

# Логи служб
journalctl -u samba -f
journalctl -u sssd -f

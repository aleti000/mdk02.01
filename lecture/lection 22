# Лекция 22: Настройка Samba DC

## Цели и задачи
- Изучить основные принципы работы Samba как контроллера домена (Samba DC).
- Разобраться с ролью Samba в интеграции Linux и Windows-систем.
- Научиться устанавливать и настраивать Samba DC для управления доменом Active Directory.
- Понять процесс добавления пользователей, групп и компьютеров в домен.
- Изучить диагностику и устранение неисправностей в работе Samba DC.
- Научиться подключать ALT Workstation к домену Samba DC.

---

## Основной материал

### 1. Введение в Samba и Samba DC

#### Что такое Samba?
Samba — это программное обеспечение с открытым исходным кодом, которое позволяет Linux-системам взаимодействовать с Windows-системами через протокол SMB/CIFS. Это особенно полезно для совместного использования файлов, принтеров и управления доменами.

#### Что такое Samba DC?
Samba DC (Domain Controller) — это реализация контроллера домена Active Directory (AD) на базе Samba. Он позволяет:
- Управлять пользователями, группами и политиками.
- Аутентифицировать клиентов Windows в домене.
- Интегрировать Linux-серверы в Windows-инфраструктуру.

#### Зачем нужен Samba DC?
- Создание централизованного управления доменом.
- Обеспечение совместимости между Linux и Windows.
- Снижение зависимости от Windows Server.

---

### 2. Установка Samba DC

#### Используемые пакеты
Для установки Samba DC используется пакетный менеджер. Например, в Ubuntu:

1. Обновите список пакетов:
   apt update
2. Установите Samba и необходимые компоненты:
   apt install samba krb5-user winbind

#### Настройка Kerberos
Kerberos — это система аутентификации, используемая в Active Directory. Настройте файл /etc/krb5.conf:
[libdefaults]
    default_realm = EXAMPLE.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true

#### Подготовка сервера
1. Убедитесь, что имя хоста настроено правильно:
   hostnamectl set-hostname dc.example.com
2. Добавьте запись в /etc/hosts:
   127.0.0.1 dc.example.com dc

#### Удаление конфликтующих служб
Убедитесь, что другие службы (например, smbd, nmbd) остановлены:
systemctl stop smbd nmbd winbind
systemctl disable smbd nmbd winbind

---

### 3. Настройка Samba DC

#### Создание домена
Инициализируйте новый домен Active Directory:
samba-tool domain provision --use-rfc2307 --interactive

При выполнении команды потребуется ввести:
- Realm (например, EXAMPLE.COM).
- Domain (например, EXAMPLE).
- Пароль администратора.

#### Запуск службы
Запустите службу Samba:
systemctl start samba-ad-dc
systemctl enable samba-ad-dc

#### Проверка работы
Проверьте статус службы:
systemctl status samba-ad-dc

Проверьте работу DNS:
host -t SRV _ldap._tcp.example.com

---

### 4. Управление доменом

#### Добавление пользователей
Создайте нового пользователя:
samba-tool user create username

Измените пароль пользователя:
samba-tool user setpassword username

#### Добавление групп
Создайте новую группу:
samba-tool group add groupname

Добавьте пользователя в группу:
samba-tool group addmembers groupname username

#### Управление компьютерами
Добавьте компьютер в домен:
samba-tool computer add computername

---

### 5. Настройка клиентов Windows

#### Присоединение к домену
1. Откройте "Свойства системы" на клиентской машине.
2. В разделе "Имя компьютера" выберите "Изменить".
3. Укажите имя домена (например, EXAMPLE.COM).
4. Введите учетные данные администратора домена.

#### Проверка подключения
Проверьте подключение к домену:
ping dc.example.com

---

### 6. Включение ALT Workstation в домен

#### Установка необходимых пакетов
На ALT Workstation установите пакеты для работы с доменом:
apt-get install realmd sssd adcli krb5-user samba-common-bin

#### Настройка Kerberos
Настройте файл /etc/krb5.conf для работы с доменом:
[libdefaults]
    default_realm = EXAMPLE.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true

#### Поиск домена
Проверьте доступность домена:
realm discover example.com

#### Присоединение к домену
Присоедините ALT Workstation к домену:
realm join example.com --user=Administrator

#### Проверка подключения
Проверьте успешность присоединения:
realm list

#### Настройка авторизации
Настройте SSSD для работы с доменом:
1. Отредактируйте файл /etc/sssd/sssd.conf:
   [sssd]
   services = nss, pam
   domains = example.com

   [domain/example.com]
   ad_domain = example.com
   krb5_realm = EXAMPLE.COM
   realmd_tags = manages-system joined-with-samba
   cache_credentials = True
   id_provider = ad
   access_provider = ad
2. Перезапустите службу SSSD:
   systemctl restart sssd

#### Проверка авторизации
Проверьте, что пользователи домена доступны:
id username@example.com

Войдите в систему под учетной записью домена.

---

### 7. Диагностика и устранение неисправностей

#### Просмотр логов
Логи Samba находятся в каталоге /var/log/samba/:
tail -f /var/log/samba/log.samba

#### Проверка работы DNS
Проверьте работу DNS:
host -t SRV _kerberos._tcp.example.com

#### Распространенные проблемы:
- Неправильная настройка имени хоста.
- Конфликты с другими службами.
- Ошибки Kerberos.

---

### 8. Безопасность Samba DC

#### Резервное копирование
Регулярно создавайте резервные копии базы данных Samba:
samba-tool domain backup online --targetdir=/backup

#### Мониторинг
Настройте мониторинг состояния Samba:
samba-tool dbcheck

---

### 9. Пример настройки Samba DC

#### Шаги:
1. Установите Samba:
   apt install samba krb5-user winbind
2. Настройте Kerberos:
   /etc/krb5.conf
3. Создайте домен:
   samba-tool domain provision --use-rfc2307 --interactive
4. Запустите службу:
   systemctl start samba-ad-dc
5. Добавьте пользователя:
   samba-tool user create adminuser
6. Присоедините ALT Workstation к домену:
   realm join example.com --user=Administrator

---

## Домашнее задание
1. Установите и настройте Samba DC на виртуальной машине.
2. Создайте домен Active Directory.
3. Добавьте пользователя и группу в домен.
4. Присоедините ALT Workstation к домену.
5. Изучите логи и найдите записи о работе Samba DC.

---

## Вопросы для самопроверки
1. Что такое Samba и для чего он используется?
2. Как установить и настроить Samba DC?
3. Как добавить пользователей и группы в домен?
4. Как присоединить ALT Workstation к домену?
5. Какие меры безопасности можно применить для защиты Samba DC?

# Лекция 23: Настройка Nagios

## Цели и задачи
- Изучить основные принципы работы системы мониторинга Nagios.
- Разобраться с архитектурой Nagios и его компонентами.
- Научиться устанавливать и настраивать Nagios для мониторинга серверов и сетевых устройств.
- Понять процесс создания пользовательских проверок и уведомлений.
- Изучить диагностику и устранение неисправностей в работе Nagios.

---

## Основной материал

### 1. Введение в Nagios

#### Что такое Nagios?
Nagios — это система мониторинга, которая позволяет отслеживать состояние серверов, сетевых устройств, приложений и сервисов. Она предоставляет:
- Мониторинг состояния хостов и сервисов.
- Уведомления о проблемах (по email, SMS и т.д.).
- Графический интерфейс для анализа данных.

#### Зачем нужен Nagios?
- Обеспечение непрерывности работы IT-инфраструктуры.
- Выявление проблем до их критического проявления.
- Автоматизация уведомлений и реагирования на инциденты.

#### Основные компоненты:
1. Nagios Core: Основная система мониторинга.
2. Nagios Plugins: Набор скриптов для проверки различных параметров.
3. NRPE (Nagios Remote Plugin Executor): Инструмент для удаленного мониторинга.
4. Web-интерфейс: Веб-панель для управления и анализа данных.

---

### 2. Установка Nagios

#### Используемые пакеты
Для установки Nagios используется пакетный менеджер. Например, в Ubuntu:

1. Обновите список пакетов:
   apt update
2. Установите необходимые зависимости:
   apt install autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php7.4 libgd-dev
3. Скачайте и распакуйте Nagios Core:
   wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz
   tar -xzf nagios-4.4.6.tar.gz
4. Перейдите в каталог:
   cd nagios-4.4.6

#### Компиляция и установка
Скомпилируйте и установите Nagios:
./configure --with-httpd-conf=/etc/apache2/sites-enabled
make all
make install-groups-users
usermod -a -G nagios www-data
make install
make install-daemoninit
make install-commandmode
make install-config
make install-webconf

#### Установка плагинов
Скачайте и установите плагины:
wget https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
tar -xzf nagios-plugins-2.3.3.tar.gz
cd nagios-plugins-2.3.3
./configure
make
make install

---

### 3. Настройка Nagios

#### Настройка Apache
Включите модуль CGI для Apache:
a2enmod cgi
systemctl restart apache2

Создайте пользователя для доступа к веб-интерфейсу:
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

#### Настройка конфигурации
Основной файл конфигурации:
/usr/local/nagios/etc/nagios.cfg

Пример базовой конфигурации:
log_file=/usr/local/nagios/var/nagios.log
cfg_dir=/usr/local/nagios/etc/servers

Добавьте конфигурацию для хостов:
define host {
    use                     linux-server
    host_name               server1
    alias                   Server 1
    address                 192.168.1.10
}

#### Проверка конфигурации
Проверьте корректность конфигурации:
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

#### Запуск службы
Запустите службу Nagios:
systemctl start nagios
systemctl enable nagios

---

### 4. Мониторинг хостов и сервисов

#### Добавление хостов
Создайте файл конфигурации для нового хоста:
/usr/local/nagios/etc/servers/server1.cfg

Пример конфигурации:
define host {
    use                     linux-server
    host_name               server1
    alias                   Server 1
    address                 192.168.1.10
}

define service {
    use                     generic-service
    host_name               server1
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

#### Настройка NRPE
На удаленном хосте установите NRPE:
apt install nagios-nrpe-server nagios-plugins

Настройте файл конфигурации NRPE:
/etc/nagios/nrpe.cfg

Пример команды для проверки дискового пространства:
command[check_disk]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /

Перезапустите службу NRPE:
systemctl restart nagios-nrpe-server

---

### 5. Уведомления

#### Настройка email-уведомлений
Отредактируйте файл контактов:
/usr/local/nagios/etc/objects/contacts.cfg

Пример конфигурации:
define contact {
    contact_name            admin
    use                     generic-contact
    alias                   Admin User
    email                   admin@example.com
}

#### Настройка команд
Добавьте команду для отправки уведомлений:
/usr/local/nagios/etc/objects/commands.cfg

Пример команды:
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "Host Alert: $HOSTNAME$ is $HOSTSTATE$\n\nDate/Time: $LONGDATETIME$" | mail -s "Host Alert: $HOSTNAME$" $CONTACTEMAIL$
}

---

### 6. Диагностика и устранение неисправностей

#### Просмотр логов
Логи Nagios находятся в каталоге:
/usr/local/nagios/var/nagios.log

#### Распространенные проблемы:
- Неправильная конфигурация файлов.
- Отсутствие доступа к удаленным хостам.
- Проблемы с правами доступа.

---

### 7. Безопасность Nagios

#### Ограничение доступа
Ограничьте доступ к веб-интерфейсу через Apache:
<Directory "/usr/local/nagios/sbin">
    Options ExecCGI
    AllowOverride None
    Order allow,deny
    Allow from 192.168.1.0/24
</Directory>

#### Регулярное обновление
Регулярно обновляйте Nagios и плагины для защиты от уязвимостей.

---

### 8. Пример настройки Nagios

#### Шаги:
1. Установите Nagios:
   apt install autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php7.4 libgd-dev
2. Скомпилируйте и установите Nagios Core.
3. Установите плагины.
4. Настройте Apache и создайте пользователя:
   htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
5. Добавьте хосты и сервисы:
   /usr/local/nagios/etc/servers/server1.cfg
6. Настройте NRPE для удаленного мониторинга.

---

## Домашнее задание
1. Установите и настройте Nagios на виртуальной машине.
2. Добавьте два хоста для мониторинга.
3. Настройте проверку дискового пространства и нагрузки CPU.
4. Настройте email-уведомления.
5. Изучите логи и найдите записи о работе Nagios.

---

## Вопросы для самопроверки
1. Что такое Nagios и для чего он используется?
2. Как установить и настроить Nagios Core?
3. Как добавить хосты и сервисы для мониторинга?
4. Как настроить NRPE для удаленного мониторинга?
5. Какие меры безопасности можно применить для защиты Nagios?

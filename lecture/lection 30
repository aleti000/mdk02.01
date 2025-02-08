# Лекция 30: Настройка Zabbix

## Цели и задачи
- Изучить основные принципы работы Zabbix как системы мониторинга.
- Разобраться с архитектурой Zabbix и её компонентами (Server, Agent, Proxy, Frontend).
- Научиться устанавливать и настраивать Zabbix для мониторинга серверов и приложений.
- Понять процесс создания шаблонов, триггеров и оповещений.
- Изучить диагностику и устранение неисправностей в работе Zabbix.

---

## Основной материал

### 1. Введение в Zabbix

#### Что такое Zabbix?
Zabbix — это система мониторинга с открытым исходным кодом, которая позволяет собирать метрики с серверов, сетевых устройств и приложений. Она предоставляет:
- Сбор данных о производительности и доступности.
- Создание графиков и дашбордов.
- Настройку оповещений о проблемах.
- Автоматическое обнаружение устройств.

#### Зачем нужен Zabbix?
- Мониторинг состояния инфраструктуры.
- Выявление проблем до их критического проявления.
- Автоматизация оповещений и реагирования на инциденты.

#### Основные компоненты:
1. Zabbix Server: Основной компонент, который собирает и обрабатывает данные.
2. Zabbix Agent: Программа, установленная на целевых системах для сбора метрик.
3. Zabbix Proxy: Промежуточный сервер для распределенного мониторинга.
4. Zabbix Frontend: Веб-интерфейс для управления системой.

---

### 2. Установка Zabbix

#### Используемые пакеты
Для установки Zabbix используется пакетный менеджер. Например, в Ubuntu:

1. Добавьте репозиторий Zabbix:
   wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-3+ubuntu20.04_all.deb
   dpkg -i zabbix-release_6.0-3+ubuntu20.04_all.deb
   apt update
2. Установите Zabbix Server, Agent и Frontend:
   apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent

#### Настройка базы данных
1. Установите MySQL:
   apt install mysql-server
2. Создайте базу данных для Zabbix:
   mysql -uroot -p
   create database zabbix character set utf8mb4 collate utf8mb4_bin;
   create user zabbix@localhost identified by 'password';
   grant all privileges on zabbix.* to zabbix@localhost;
   exit;
3. Импортируйте схему базы данных:
   zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

#### Настройка конфигурации
1. Откройте файл конфигурации:
   /etc/zabbix/zabbix_server.conf
2. Укажите параметры базы данных:
   DBHost=localhost
   DBName=zabbix
   DBUser=zabbix
   DBPassword=password

#### Запуск службы
Запустите Zabbix Server и Agent:
systemctl start zabbix-server zabbix-agent
systemctl enable zabbix-server zabbix-agent

---

### 3. Настройка веб-интерфейса

#### Доступ к интерфейсу
Откройте веб-интерфейс Zabbix:
http://localhost/zabbix

#### Шаги настройки:
1. Выберите язык и нажмите Next.
2. Укажите параметры подключения к базе данных:
   Database host: localhost
   Database name: zabbix
   User: zabbix
   Password: password
3. Укажите адрес Zabbix Server:
   Host: localhost
   Port: 10051
4. Завершите настройку и войдите в систему:
   Username: Admin
   Password: zabbix

---

### 4. Настройка мониторинга

#### Добавление хоста
1. Перейдите в раздел Configuration -> Hosts.
2. Нажмите Create host.
3. Укажите имя хоста и IP-адрес:
   Host name: webserver
   Groups: Linux servers
   Interfaces: IP address: 192.168.1.10

#### Привязка шаблона
1. Выберите шаблон Template OS Linux.
2. Нажмите Add и сохраните изменения.

---

### 5. Создание триггеров

#### Что такое триггер?
Триггер — это условие, которое определяет проблему. Например, высокая загрузка CPU.

#### Пример создания триггера:
1. Перейдите в раздел Configuration -> Templates.
2. Выберите шаблон Template OS Linux.
3. Перейдите в раздел Triggers и нажмите Create trigger.
4. Укажите условия:
   Name: High CPU load
   Expression: {Template OS Linux:system.cpu.load[percpu,avg1].last()} > 5

---

### 6. Настройка оповещений

#### Создание медиа-типа
1. Перейдите в раздел Administration -> Media types.
2. Нажмите Create media type.
3. Выберите тип Email и укажите параметры:
   SMTP server: smtp.example.com
   SMTP email: zabbix@example.com

#### Настройка пользователя
1. Перейдите в раздел Administration -> Users.
2. Выберите пользователя Admin.
3. Перейдите в раздел Media и добавьте новый медиа-тип:
   Type: Email
   Send to: admin@example.com

---

### 7. Диагностика и устранение неисправностей

#### Просмотр логов
Логи Zabbix находятся в каталоге:
/var/log/zabbix/

#### Распространенные проблемы:
- Неправильная конфигурация базы данных.
- Отсутствие доступа к целевым системам.
- Проблемы с правами доступа.

---

### 8. Безопасность Zabbix

#### Ограничение доступа
Ограничьте доступ к Zabbix через файрвол:
ufw allow from 192.168.1.0/24 to any port 10051

#### Настройка HTTPS
Настройте SSL/TLS для защиты данных:
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /path/to/cert.pem
    SSLCertificateKeyFile /path/to/key.pem
</VirtualHost>

---

### 9. Пример настройки Zabbix

#### Шаги:
1. Установите Zabbix:
   apt install zabbix-server-mysql zabbix-frontend-php zabbix-agent
2. Настройте базу данных:
   mysql -uroot -p
   create database zabbix;
3. Запустите службы:
   systemctl start zabbix-server zabbix-agent
4. Добавьте хост:
   Configuration -> Hosts -> Create host
5. Настройте оповещения:
   Administration -> Media types -> Email

---

## Практические задания

1. Установите Zabbix на виртуальной машине.
2. Настройте мониторинг для двух серверов.
3. Создайте триггер для высокой загрузки CPU.
4. Настройте оповещения по email.
5. Настройте HTTPS для веб-интерфейса.

---

## Домашнее задание

1. Установите и настройте Zabbix на виртуальной машине.
2. Добавьте два хоста для мониторинга.
3. Создайте триггер для высокой загрузки RAM.
4. Настройте оповещения по email.
5. Настройте HTTPS для защиты данных.

---

## Вопросы для самопроверки

1. Что такое Zabbix и для чего он используется?
2. Как установить и настроить Zabbix?
3. Как добавить хост для мониторинга?
4. Как создать триггер для определения проблем?
5. Какие меры безопасности можно применить для защиты Zabbix?

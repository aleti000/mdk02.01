# Лекция 29: Настройка HAProxy

## Цели и задачи
- Изучить основные принципы работы HAProxy как балансировщика нагрузки.
- Разобраться с архитектурой HAProxy и её компонентами (frontend, backend, server).
- Научиться устанавливать и настраивать HAProxy для распределения нагрузки между серверами.
- Понять процесс создания правил маршрутизации трафика и настройки высокой доступности.
- Изучить диагностику и устранение неисправностей в работе HAProxy.

---

## Основной материал

### 1. Введение в HAProxy

#### Что такое HAProxy?
HAProxy (High Availability Proxy) — это высокопроизводительный балансировщик нагрузки и прокси-сервер, который используется для распределения входящего трафика между несколькими серверами. Он поддерживает протоколы TCP и HTTP и обеспечивает высокую доступность и отказоустойчивость.

#### Зачем нужен HAProxy?
- Распределение нагрузки между серверами для повышения производительности.
- Обеспечение отказоустойчивости за счет перенаправления трафика на рабочие серверы.
- Мониторинг состояния серверов и автоматическое исключение неисправных.

#### Основные компоненты:
1. Frontend: Определяет правила приема входящего трафика (например, порт и IP-адрес).
2. Backend: Группа серверов, на которые распределяется трафик.
3. Server: Отдельный сервер в группе backend.
4. ACL (Access Control List): Правила для фильтрации и маршрутизации трафика.
5. Health Check: Проверка работоспособности серверов.

---

### 2. Установка HAProxy

#### Используемые пакеты
Для установки HAProxy используется пакетный менеджер. Например, в Ubuntu:

1. Обновите список пакетов:
   apt update
2. Установите HAProxy:
   apt install haproxy

#### Проверка установки
Проверьте версию HAProxy:
haproxy -v

---

### 3. Настройка конфигурации

#### Файл конфигурации
Основной файл конфигурации:
/etc/haproxy/haproxy.cfg

Пример базовой конфигурации:
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check

#### Объяснение параметров:
1. global: Глобальные настройки (логирование, права доступа).
2. defaults: Параметры по умолчанию для всех frontend и backend.
3. frontend: Определяет правила приема трафика.
4. backend: Группа серверов для обработки трафика.
5. balance: Алгоритм балансировки (например, roundrobin, leastconn).

---

### 4. Алгоритмы балансировки

HAProxy поддерживает несколько алгоритмов балансировки:
1. roundrobin: Поочередное распределение запросов.
2. leastconn: Перенаправление на сервер с наименьшим количеством активных соединений.
3. source: Распределение на основе IP-адреса клиента.

Пример настройки алгоритма:
backend http_back
    balance leastconn
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check

---

### 5. Настройка Health Check

#### Что такое Health Check?
Health Check — это проверка работоспособности серверов. Если сервер не отвечает, HAProxy временно исключает его из пула.

#### Пример настройки:
backend http_back
    balance roundrobin
    server web1 192.168.1.10:80 check inter 2000 rise 2 fall 3
    server web2 192.168.1.11:80 check inter 2000 rise 2 fall 3

#### Объяснение параметров:
1. check: Включение проверки.
2. inter: Интервал проверки (в миллисекундах).
3. rise: Количество успешных проверок для включения сервера.
4. fall: Количество неудачных проверок для исключения сервера.

---

### 6. Настройка ACL

#### Что такое ACL?
ACL (Access Control List) — это правила для фильтрации и маршрутизации трафика. Они позволяют направлять запросы на разные backend в зависимости от условий.

#### Пример настройки:
frontend http_front
    bind *:80
    acl is_web hdr(host) -i example.com
    use_backend web_back if is_web
    default_backend default_back

backend web_back
    server web1 192.168.1.10:80 check

backend default_back
    server default 192.168.1.12:80 check

#### Объяснение параметров:
1. acl: Создание правила (например, проверка заголовка host).
2. use_backend: Направление трафика на указанный backend.
3. default_backend: Резервный backend.

---

### 7. Диагностика и устранение неисправностей

#### Просмотр логов
Логи HAProxy находятся в каталоге:
/var/log/haproxy/

#### Распространенные проблемы:
- Неправильная конфигурация файла haproxy.cfg.
- Отсутствие доступа к серверам backend.
- Проблемы с правами доступа.

---

### 8. Безопасность HAProxy

#### Ограничение доступа
Ограничьте доступ к HAProxy через файрвол:
ufw allow from 192.168.1.0/24 to any port 80

#### Настройка HTTPS
Настройте SSL/TLS для защиты данных:
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/example.com.pem
    default_backend https_back

---

### 9. Пример настройки HAProxy

#### Шаги:
1. Установите HAProxy:
   apt install haproxy
2. Настройте конфигурацию:
   /etc/haproxy/haproxy.cfg
3. Добавьте frontend и backend:
   frontend http_front
       bind *:80
       default_backend http_back

   backend http_back
       balance roundrobin
       server web1 192.168.1.10:80 check
       server web2 192.168.1.11:80 check
4. Перезапустите службу:
   systemctl restart haproxy

---

## Практические задания

1. Установите HAProxy на виртуальной машине.
2. Настройте балансировку нагрузки между двумя веб-серверами.
3. Добавьте правило ACL для маршрутизации трафика.
4. Настройте Health Check для мониторинга серверов.
5. Настройте HTTPS для защиты данных.

---

## Домашнее задание

1. Установите и настройте HAProxy на виртуальной машине.
2. Создайте два backend для балансировки нагрузки.
3. Настройте ACL для маршрутизации трафика.
4. Проверьте работу Health Check.
5. Настройте HTTPS и протестируйте безопасное соединение.

---

## Вопросы для самопроверки

1. Что такое HAProxy и для чего он используется?
2. Как установить и настроить HAProxy?
3. Какие алгоритмы балансировки поддерживаются в HAProxy?
4. Как настроить Health Check для мониторинга серверов?
5. Как использовать ACL для маршрутизации трафика?
6. Какие меры безопасности можно применить для защиты HAProxy?

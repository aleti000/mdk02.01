# Лекция 27: Настройка Prometheus

## Цели и задачи
- Изучить основные принципы работы системы мониторинга Prometheus.
- Разобраться с архитектурой Prometheus и её компонентами (Exporter, Alertmanager, Grafana).
- Научиться устанавливать и настраивать Prometheus для мониторинга серверов и приложений.
- Понять процесс создания правил сбора метрик и настройки оповещений.
- Изучить диагностику и устранение неисправностей в работе Prometheus.

---

## Основной материал

### 1. Введение в Prometheus

#### Что такое Prometheus?
Prometheus — это система мониторинга с открытым исходным кодом, которая собирает метрики с целевых систем (targets) через HTTP-запросы. Она предоставляет:
- Сбор метрик в реальном времени.
- Мощный язык запросов PromQL для анализа данных.
- Интеграцию с Alertmanager для отправки уведомлений.

#### Зачем нужен Prometheus?
- Мониторинг состояния серверов, приложений и сервисов.
- Выявление проблем до их критического проявления.
- Автоматизация оповещений и реагирования на инциденты.

#### Основные компоненты:
1. Prometheus Server: Основной компонент, который собирает и хранит метрики.
2. Exporter: Программы, которые предоставляют метрики для сбора (например, Node Exporter, MySQL Exporter).
3. Alertmanager: Управляет оповещениями и отправляет уведомления.
4. Grafana: Визуализация метрик в виде графиков и дашбордов.

---

### 2. Установка Prometheus

#### Используемые пакеты
Для установки Prometheus используется пакетный менеджер или скачивание бинарных файлов. Например:

1. Скачайте архив с официального сайта:
   wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
2. Распакуйте архив:
   tar -xzf prometheus-2.45.0.linux-amd64.tar.gz
3. Перейдите в каталог:
   cd prometheus-2.45.0.linux-amd64

#### Запуск Prometheus
Запустите Prometheus:
./prometheus --config.file=prometheus.yml

---

### 3. Настройка конфигурации

#### Файл конфигурации
Основной файл конфигурации:
prometheus.yml

Пример базовой конфигурации:
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['192.168.1.10:9100']

#### Объяснение параметров:
1. scrape_interval: Интервал сбора метрик.
2. job_name: Имя задачи для сбора метрик.
3. targets: Адреса целевых систем.

---

### 4. Работа с Exporter

#### Node Exporter
Node Exporter собирает метрики с серверов (CPU, RAM, дисковое пространство и т.д.).

1. Скачайте и запустите Node Exporter:
   wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
   tar -xzf node_exporter-1.6.1.linux-amd64.tar.gz
   cd node_exporter-1.6.1.linux-amd64
   ./node_exporter

2. Добавьте Node Exporter в конфигурацию Prometheus:
   - job_name: 'node'
     static_configs:
       - targets: ['192.168.1.10:9100']

---

### 5. Настройка Alertmanager

#### Установка Alertmanager
Скачайте и запустите Alertmanager:
wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
tar -xzf alertmanager-0.26.0.linux-amd64.tar.gz
cd alertmanager-0.26.0.linux-amd64
./alertmanager --config.file=alertmanager.yml

#### Файл конфигурации
Пример файла alertmanager.yml:
route:
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'admin@example.com'
        from: 'prometheus@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'user'
        auth_password: 'password'

---

### 6. Создание правил оповещений

#### Файл правил
Создайте файл rules.yml для определения правил:
groups:
- name: example
  rules:
  - alert: HighCpuLoad
    expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High CPU load on {{ $labels.instance }}"
      description: "CPU load is above 80% for more than 5 minutes."

#### Подключение правил
Добавьте файл правил в prometheus.yml:
rule_files:
  - "rules.yml"

---

### 7. Интеграция с Grafana

#### Установка Grafana
Установите Grafana через пакетный менеджер:
apt install grafana

Запустите службу:
systemctl start grafana-server
systemctl enable grafana-server

#### Настройка источника данных
1. Откройте веб-интерфейс Grafana:
   http://localhost:3000
2. Добавьте Prometheus как источник данных:
   URL: http://localhost:9090

---

### 8. Диагностика и устранение неисправностей

#### Просмотр логов
Логи Prometheus находятся в каталоге:
/var/log/prometheus/

#### Распространенные проблемы:
- Неправильная конфигурация файлов.
- Отсутствие доступа к целевым системам.
- Проблемы с правами доступа.

---

### 9. Безопасность Prometheus

#### Ограничение доступа
Ограничьте доступ к Prometheus через файрвол:
ufw allow from 192.168.1.0/24 to any port 9090

#### Шифрование данных
Используйте HTTPS для защиты данных.

---

### 10. Пример настройки Prometheus

#### Шаги:
1. Скачайте и запустите Prometheus:
   wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
   tar -xzf prometheus-2.45.0.linux-amd64.tar.gz
   ./prometheus --config.file=prometheus.yml
2. Установите Node Exporter:
   wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
   tar -xzf node_exporter-1.6.1.linux-amd64.tar.gz
   ./node_exporter
3. Настройте Alertmanager:
   ./alertmanager --config.file=alertmanager.yml
4. Создайте правила оповещений:
   rules.yml
5. Интегрируйте Grafana:
   apt install grafana

---

## Домашнее задание
1. Установите и настройте Prometheus на виртуальной машине.
2. Добавьте Node Exporter для мониторинга сервера.
3. Настройте Alertmanager для отправки уведомлений.
4. Создайте правило оповещения о высокой нагрузке CPU.
5. Интегрируйте Grafana для визуализации метрик.

---

## Вопросы для самопроверки
1. Что такое Prometheus и для чего он используется?
2. Как установить и настроить Prometheus?
3. Как добавить Exporter для сбора метрик?
4. Как настроить Alertmanager для отправки уведомлений?
5. Какие меры безопасности можно применить для защиты Prometheus?

# Лекция 13.2: Docker Compose

## Цели и задачи
- Изучить основные принципы работы с Docker Compose.
- Разобраться с ролью Docker Compose в управлении многоконтейнерными приложениями.
- Научиться создавать и использовать файл docker-compose.yml.
- Понять процесс запуска, остановки и управления контейнерами через Docker Compose.
- Изучить диагностику и устранение неисправностей в работе Docker Compose.

---

## Основной материал

### 1. Введение в Docker Compose

#### Что такое Docker Compose?
Docker Compose — это инструмент для определения и запуска многоконтейнерных приложений. Он позволяет описать все сервисы, сети и тома (volumes) в одном файле (docker-compose.yml) и управлять ими одной командой.

#### Зачем нужен Docker Compose?
- Упрощение управления сложными приложениями, состоящими из нескольких контейнеров.
- Централизованное описание конфигурации приложения.
- Возможность воспроизводить одинаковую среду на разных машинах.

#### Преимущества Docker Compose:
- Единая команда для запуска всех контейнеров.
- Поддержка сетей и томов для взаимодействия между контейнерами.
- Легкость масштабирования и тестирования.

---

### 2. Установка Docker Compose

#### Установка через пакетный менеджер
Для установки Docker Compose в Linux используется пакетный менеджер. Например, в Ubuntu:

1. Скачайте бинарный файл:
   curl -L "https://github.com/docker/compose/releases/download/v2.x.x/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
2. Добавьте права на выполнение:
   chmod +x /usr/local/bin/docker-compose
3. Проверьте версию:
   docker-compose --version

---

### 3. Файл docker-compose.yml

#### Структура файла
Файл docker-compose.yml состоит из трех основных секций:
1. version: Версия формата файла.
2. services: Описание контейнеров (сервисов).
3. networks и volumes: Опциональные секции для настройки сетей и томов.

##### Пример файла docker-compose.yml:
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: testdb
    volumes:
      - db_data:/var/lib/mysql
volumes:
  db_data:

- version: Версия формата файла (например, 3.8).
- services: Описание сервисов (контейнеров).
- web: Имя сервиса (веб-сервер Nginx).
- db: Имя сервиса (база данных MySQL).
- ports: Проброс портов хоста в контейнер.
- volumes: Монтирование директорий или томов.
- environment: Переменные окружения.

---

### 4. Основные команды Docker Compose

#### Запуск контейнеров
Запустите все контейнеры, описанные в docker-compose.yml:
docker-compose up -d

#### Просмотр запущенных контейнеров
Просмотрите список запущенных контейнеров:
docker-compose ps

#### Остановка контейнеров
Остановите все контейнеры:
docker-compose down

#### Просмотр логов
Просмотрите логи всех контейнеров:
docker-compose logs

#### Масштабирование сервисов
Масштабируйте сервис (например, запустите 3 экземпляра):
docker-compose up --scale web=3

---

### 5. Пример использования Docker Compose

#### Сценарий: Веб-приложение с базой данных
Создадим простое веб-приложение, состоящее из:
1. Веб-сервера Nginx.
2. Базы данных MySQL.

##### Шаги:
1. Создайте файл docker-compose.yml:
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: testdb
    volumes:
      - db_data:/var/lib/mysql
volumes:
  db_data:

2. Создайте директорию html и добавьте файл index.html:
<!DOCTYPE html>
<html>
<head>
    <title>Test Page</title>
</head>
<body>
    <h1>Hello from Docker Compose!</h1>
</body>
</html>

3. Запустите контейнеры:
   docker-compose up -d

4. Откройте браузер и перейдите по адресу:
   http://localhost:8080

---

### 6. Диагностика и устранение неисправностей

#### Проверка статуса контейнеров
Проверьте статус всех контейнеров:
docker-compose ps

#### Просмотр логов
Просмотрите логи конкретного сервиса:
docker-compose logs web

#### Распространенные проблемы:
- Ошибки в файле docker-compose.yml.
- Конфликты портов на хосте.
- Недостаточно ресурсов для запуска контейнеров.

---

### 7. Безопасность Docker Compose

#### Защита контейнеров
1. Используйте переменные окружения для хранения секретов:
environment:
  MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
2. Ограничьте доступ к сетям:
networks:
  app_network:
    internal: true

#### Мониторинг контейнеров
Используйте инструменты мониторинга для отслеживания работы контейнеров:
docker-compose events

---

## Домашнее задание
1. Установите Docker Compose на виртуальной машине.
2. Создайте файл docker-compose.yml для приложения, состоящего из веб-сервера и базы данных.
3. Запустите контейнеры и проверьте их работу.
4. Изучите логи контейнеров и найдите записи о запросах.
5. Остановите контейнеры и удалите их.

---

## Вопросы для самопроверки
1. Что такое Docker Compose и для чего он используется?
2. Как установить Docker Compose в Linux?
3. Как создать и использовать файл docker-compose.yml?
4. Как запустить, остановить и масштабировать контейнеры через Docker Compose?
5. Какие меры безопасности можно применить для защиты контейнеров?



# Лабораторная работа Введение в Docker

## Цель работы:
Ознакомиться с основами Docker, научиться создавать контейнеры, образы, работать с Docker Compose и настраивать сетевые соединения между контейнерами.

---

## Оборудование и ПО:
- Операционная система: Linux/macOS/Windows (рекомендуется Linux или Docker Desktop для Windows/macOS)
- Docker Engine (https://docs.docker.com/engine/install/)
- Docker Compose (входит в Docker Desktop или устанавливается отдельно)

---

## Теоретическая часть

### 1. Основные понятия Docker:
- **Контейнер** — изолированная среда, запущенная из образа.
- **Изображение (Image)** — шаблон для создания контейнеров.
- **Dockerfile** — текстовый файл с инструкциями для сборки образа.
- **Docker Hub** — центральный репозиторий образов.
- **Docker Compose** — инструмент для запуска мультиконтейнерных приложений.

### 2. Ключевые команды Docker:
| Команда | Описание |
|---------|----------|
| `docker run` | Запуск контейнера |
| `docker build` | Сборка образа из Dockerfile |
| `docker ps` | Список запущенных контейнеров |
| `docker images` | Список локальных образов |
| `docker exec` | Выполнение команд внутри контейнера |
| `docker rm` | Удаление контейнера |
| `docker rmi` | Удаление образа |

---

## Практическая часть

### Задание 1. Установка Docker и запуск первого контейнера
1. Установите Docker (следуйте официальной документации):
   ```bash
   sudo apt-get update && sudo apt-get install docker.io
   ```

2. Проверьте установку:
   ```bash
   docker --version
   docker run hello-world
   ```

3. Остановите контейнер:
   ```bash
   docker stop [ID_контейнера]
   ```

---

### Задание 2. Создание собственного образа
1. Создайте файл `Dockerfile` в новой директории:
   ```dockerfile
   # Используем базовый образ Ubuntu
   FROM ubuntu:latest

   # Устанавливаем Apache
   RUN apt-get update && apt-get install -y apache2

   # Копируем файл index.html в директорию веб-сервера
   COPY index.html /var/www/html/

   # Определяем порт для открытия
   EXPOSE 80

   # Запускаем Apache
   CMD ["apache2ctl", "-D", "FOREGROUND"]
   ```

2. Создайте файл `index.html`:
   ```html
   <h1>Привет из Docker!</h1>
   ```

3. Соберите образ:
   ```bash
   docker build -t my-apache-app .
   ```

4. Запустите контейнер:
   ```bash
   docker run -d -p 8080:80 --name my-apache-container my-apache-app
   ```

5. Проверьте работу через браузер: `http://localhost:8080`

---

### Задание 3. Работа с Docker Compose
1. Создайте файл `docker-compose.yml`:
   ```yaml
   version: '3'
   services:
     web:
       build: .
       ports:
         - "8080:80"
     db:
       image: mysql:5.7
       environment:
         MYSQL_ROOT_PASSWORD: example
   ```

2. Запустите стек:
   ```bash
   docker-compose up -d
   ```

3. Проверьте запуск:
   ```bash
   docker-compose ps
   ```

4. Удалите стек:
   ```bash
   docker-compose down
   ```

---

### Задание 4. Настройка сети между контейнерами
1. Измените `docker-compose.yml`:
   ```yaml
   version: '3'
   services:
     web:
       build: .
       ports:
         - "8080:80"
       environment:
         DB_HOST: db
     db:
       image: mysql:5.7
       environment:
         MYSQL_ROOT_PASSWORD: example
       networks:
         - my-network
   networks:
     my-network:
       driver: bridge
   ```

2. Перезапустите стек:
   ```bash
   docker-compose up -d
   ```

3. Проверьте доступность базы данных из веб-контейнера:
   ```bash
   docker exec -it my-web-container bash
   ping db
   ```

---

## Контрольные вопросы
1. В чем разница между Docker-образом и контейнером?
2. Какие преимущества дает использование Docker Compose?
3. Как настроить подключение к базе данных из другого контейнера?
4. Что такое Dockerfile и какие основные директивы в нем используются?

---

## Ожидаемые результаты:
1. Успешный запуск образа Apache через Docker.
2. Запуск мультиконтейнерного приложения через Docker Compose.
3. Установление связи между веб-сервером и базой данных.

---

## Дополнительные задания (опционально):
1. Создайте Dockerfile для Node.js приложения.
2. Настройте автоматическое пересоздание образа при изменении кода.
3. Используйте Docker Hub для публикации своего образа.

---

## Вывод:
В ходе работы вы освоили основные инструменты Docker, научились создавать образы, работать с контейнерами и настраивать сетевые соединения. Это позволяет управлять средами приложений в изолированных окружениях.

---

**Примечание:** Все команды и конфигурации должны быть проверены в рабочей среде. Для углубленного изучения рекомендуется ознакомиться с официальной документацией Docker.

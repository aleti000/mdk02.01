# Лекция 26: Настройка Ansible

## Цели и задачи
- Изучить основные принципы работы Ansible как инструмента автоматизации.
- Разобраться с архитектурой Ansible и её компонентами (Control Node, Managed Nodes, Playbooks).
- Научиться устанавливать и настраивать Ansible для управления серверами.
- Понять процесс создания Playbook'ов для автоматизации задач.
- Изучить диагностику и устранение неисправностей в работе Ansible.
- Узнать о принципах работы ansible-playbook, утилите ansible-doc и расширенных возможностях Inventory.

---

## Основной материал

### 1. Введение в Ansible

#### Что такое Ansible?
Ansible — это инструмент автоматизации, который позволяет управлять конфигурацией серверов, развертывать приложения и выполнять задачи на удаленных хостах. Он работает по протоколу SSH и не требует установки агентов на управляемые узлы.

#### Зачем нужен Ansible?
- Упрощение управления IT-инфраструктурой.
- Автоматизация рутинных задач (например, установка пакетов, настройка сервисов).
- Централизованное управление конфигурацией.

#### Основные компоненты:
1. Control Node: Управляющий узел, на котором установлен Ansible.
2. Managed Nodes: Управляемые узлы (серверы), на которых выполняются задачи.
3. Inventory: Файл, содержащий список управляемых узлов.
4. Playbook: Файл в формате YAML, описывающий задачи для выполнения.
5. Module: Отдельный блок кода, выполняющий конкретную задачу (например, установка пакета).

---

### 2. Принцип и синтаксис Ansible Playbook

#### Что такое Playbook?
Playbook — это файл в формате YAML, который описывает последовательность задач для выполнения на управляемых узлах. Playbook состоит из одного или нескольких "plays", каждый из которых определяет группу хостов и задачи для них.

#### Структура Playbook
Пример структуры Playbook:
---
- name: Install and configure Apache
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present

    - name: Ensure Apache is running
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Copy configuration file
      copy:
        src: /local/path/httpd.conf
        dest: /etc/apache2/httpd.conf
        owner: root
        group: root
        mode: '0644'

#### Ключевые элементы:
1. hosts: Группа или список хостов, на которых выполняется Playbook.
2. tasks: Список задач для выполнения.
3. vars: Переменные, используемые в Playbook.
4. handlers: Специальные задачи, которые запускаются только при изменении состояния.
5. become: Используется для повышения привилегий (например, sudo).

#### Выполнение Playbook
Запустите Playbook:
ansible-playbook playbook.yml

Проверьте синтаксис перед выполнением:
ansible-playbook --syntax-check playbook.yml

---

### 3. Утилита ansible-doc

#### Что такое ansible-doc?
ansible-doc — это утилита для просмотра документации по модулям Ansible. Она помогает узнать доступные параметры и примеры использования модулей.

#### Примеры использования:
1. Просмотр документации по модулю:
   ansible-doc apt

2. Список всех доступных модулей:
   ansible-doc -l

3. Примеры использования модуля:
   ansible-doc -s apt

#### Преимущества:
- Быстрый доступ к документации без интернета.
- Подробное описание параметров и их значений.

---

### 4. Расширенное описание Inventory

#### Что такое Inventory?
Inventory — это файл, содержащий список управляемых узлов и их групп. Он может быть написан в формате INI или YAML.

#### Пример Inventory в формате INI:
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com

#### Пример Inventory в формате YAML:
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:

#### Дополнительные возможности:
1. Переменные для хостов:
   [webservers]
   web1.example.com http_port=80 max_requests=500
   web2.example.com http_port=8080 max_requests=1000

2. Групповые переменные:
   [webservers:vars]
   http_port=80
   max_requests=500

3. Динамический Inventory:
   Использование скриптов для генерации Inventory из внешних источников (например, AWS, OpenStack).

#### Настройка подключения:
1. Укажите пользователь для подключения:
   [webservers]
   web1.example.com ansible_user=admin

2. Укажите порт SSH:
   [webservers]
   web1.example.com ansible_port=2222

3. Использование приватного ключа:
   [webservers]
   web1.example.com ansible_ssh_private_key_file=/path/to/key

---

### 5. Установка Ansible

#### Используемые пакеты
Для установки Ansible используется пакетный менеджер. Например, в Ubuntu:

1. Обновите список пакетов:
   apt update
2. Установите Ansible:
   apt install ansible

#### Проверка установки
Проверьте версию Ansible:
ansible --version

---

### 6. Работа с модулями

#### Использование модулей
Ansible предоставляет множество встроенных модулей. Примеры:

1. Установка пакета:
   - name: Install Nginx
     apt:
       name: nginx
       state: present

2. Копирование файла:
   - name: Copy index.html
     copy:
       src: /local/path/index.html
       dest: /var/www/html/index.html

3. Выполнение команды:
   - name: Run a shell command
     shell: echo "Hello, World!" > /tmp/hello.txt

---

### 7. Диагностика и устранение неисправностей

#### Просмотр логов
Включите логирование для отладки:
[defaults]
log_path = /var/log/ansible.log

#### Распространенные проблемы:
- Отсутствие доступа по SSH.
- Неправильная конфигурация Inventory.
- Ошибки в синтаксисе Playbook.

---

### 8. Безопасность Ansible

#### Ограничение доступа
Ограничьте доступ к Control Node через файрвол:
ufw allow from 192.168.1.0/24 to any port 22

#### Шифрование данных
Используйте Ansible Vault для шифрования чувствительных данных:
ansible-vault create secrets.yml

Расшифруйте данные при выполнении Playbook:
ansible-playbook playbook.yml --ask-vault-pass

---

### 9. Пример настройки Ansible

#### Шаги:
1. Установите Ansible:
   apt install ansible
2. Создайте Inventory:
   /etc/ansible/hosts
3. Настройте SSH-ключи:
   ssh-copy-id user@web1.example.com
4. Создайте Playbook:
   playbook.yml
5. Выполните Playbook:
   ansible-playbook playbook.yml

---

## Домашнее задание
1. Установите и настройте Ansible на виртуальной машине.
2. Создайте Inventory с двумя группами хостов.
3. Напишите Playbook для установки и запуска Nginx.
4. Зашифруйте чувствительные данные с помощью Ansible Vault.
5. Изучите логи и найдите записи о работе Ansible.

---

## Вопросы для самопроверки
1. Что такое Ansible и для чего он используется?
2. Как установить и настроить Ansible?
3. Как создать Playbook для автоматизации задач?
4. Как использовать Ansible Vault для защиты данных?
5. Какие меры безопасности можно применить для защиты Ansible?
6. Что такое ansible-doc и как её использовать?
7. Какие форматы поддерживаются для Inventory?
8. Как настроить динамический Inventory?

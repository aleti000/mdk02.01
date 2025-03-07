

# Лабораторная работа Введение в Ansible

---

## Цель работы:
Ознакомиться с основами Ansible, научиться создавать playbooks, управлять удалёнными хостами, использовать модули и шаблоны для автоматизации задач.

---

## Оборудование и ПО:
- Операционная система: Linux (например, Ubuntu или CentOS) на узле Ansible (контроллер) и удалённых хостах.
- Ansible (установлен на контроллере).
- SSH-доступ к удалённым хостам (ключевой файл или пароль).

---

## Теоретическая часть

### 1. Основные понятия Ansible:
- **Ansible** — инструмент для автоматизации ИТ-процессов (развертывание, настройка, конфигурация).
- **Playbook** — файл в формате YAML, описывающий последовательность задач (tasks).
- **Модуль** — инструмент Ansible для выполнения конкретной операции (установка пакетов, создание файлов и др.).
- **Инвентори (inventory)** — список управляемых хостов и групп.
- **Task** — отдельная операция в playbook (например, установка пакета).
- **Handler** — действие, которое выполняется при изменении состояния (например, перезапуск службы).
- **Template** — шаблонный файл для генерации конфигураций.

### 2. Основные модули Ansible:
| Модуль | Описание |
|--------|----------|
| `command` | Выполняет команду на хосте |
| `shell` | Выполняет команду через оболочку (с поддержкой перенаправления) |
| `file` | Управляет файлами/директориями (создание, удаление, права доступа) |
| `copy` | Копирует файлы на хост |
| `yum`/`apt` | Устанавливает пакеты |
| `service` | Управляет службами (запуск, остановка, перезапуск) |
| `template` | Рендерит шаблонные файлы |

### 3. Структура playbook:
```yaml
---
- name: Название play
  hosts: группа_хостов
  become: yes (если нужно root-права)
  tasks:
    - name: Описание задачи
      модуль: параметры
      notify:
        - имя_handlerа
  handlers:
    - name: Описание handlerа
      модуль: параметры
```

---

## Практическая часть

### Задание 1. Установка и настройка Ansible
1. Установите Ansible на контроллере:
   ```bash
   sudo apt-get update && sudo apt-get install ansible  # для Ubuntu
   sudo yum install epel-release && sudo yum install ansible  # для CentOS
   ```

2. Настройте SSH-подключение к удалённым хостам:
   - Скопируйте SSH-ключ:
     ```bash
     ssh-copy-id user@remote_host
     ```
   - Проверьте подключение:
     ```bash
     ansible all -m ping -i "remote_host," --user user
     ```

---

### Задание 2. Создание простого playbook
**Цель:** Установить веб-сервер Apache на удалённом хосте.

1. Создайте директорию для Ansible:
   ```bash
   mkdir ansible_lab && cd ansible_lab
   ```

2. Настройте инвентори (`inventory.ini`):
   ```ini
   [webservers]
   web01 ansible_host=192.168.1.10 ansible_user=user
   ```

3. Создайте playbook `apache.yml`:
   ```yaml
   ---
   - name: Установка Apache
     hosts: webservers
     become: yes
     tasks:
       - name: Установить Apache
         apt:
           name: apache2
           state: present
       - name: Запустить Apache
         service:
           name: apache2
           state: started
           enabled: yes
   ```

4. Запустите playbook:
   ```bash
   ansible-playbook -i inventory.ini apache.yml
   ```

5. Проверьте работу:
   ```bash
   curl http://web01
   ```

---

### Задание 3. Работа с модулем `file`
**Цель:** Создать директорию и файл с заданными правами.

1. Добавьте в `apache.yml` следующие задачи:
   ```yaml
   - name: Создать директорию /var/www/myapp
     file:
       path: /var/www/myapp
       state: directory
       mode: '0755'
       owner: www-data
       group: www-data

   - name: Создать файл .htaccess
     copy:
       content: |
         Order Deny,Allow
         Deny from all
       dest: /var/www/myapp/.htaccess
       mode: '0644'
   ```

2. Перезапустите playbook:
   ```bash
   ansible-playbook -i inventory.ini apache.yml
   ```

---

### Задание 4. Использование шаблонов (templates)
**Цель:** Настроить конфигурацию Apache через шаблон.

1. Создайте директорию для шаблонов:
   ```bash
   mkdir templates
   ```

2. Создайте шаблон `templates/apache.conf.j2`:
   ```nginx
   <VirtualHost *:80>
       ServerAdmin admin@example.com
       DocumentRoot /var/www/myapp
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. Добавьте в playbook задачу для копирования шаблона:
   ```yaml
   - name: Настроить конфиг Apache
     template:
       src: templates/apache.conf.j2
       dest: /etc/apache2/sites-available/myapp.conf
     notify:
       - Перезапустить Apache

   - name: Активировать конфиг
     file:
       src: /etc/apache2/sites-available/myapp.conf
       dest: /etc/apache2/sites-enabled/myapp.conf
       state: link

   handlers:
     - name: Перезапустить Apache
       service:
         name: apache2
         state: restarted
   ```

4. Запустите playbook:
   ```bash
   ansible-playbook -i inventory.ini apache.yml
   ```

---

### Задание 5. Переменные и условия
**Цель:** Использовать переменные и условия в playbook.

1. Создайте файл `vars/main.yml`:
   ```yaml
   app_user: www-data
   app_group: www-data
   app_dir: /var/www/myapp
   ```

2. Добавьте в playbook секцию `vars`:
   ```yaml
   vars_files:
     - vars/main.yml
   ```

3. Добавьте условие для установки пакета:
   ```yaml
   - name: Установить Apache только если он не установлен
     apt:
       name: apache2
       state: present
     when: ansible_os_family == "Debian"
   ```

---

## Контрольные вопросы
1. В чем принципиальное отличие Ansible от других систем управления конфигурациями (например, Puppet или Chef)?
2. Какие модули Ansible используются для управления пакетами на Debian и Red Hat системах?
3. Что такое handler в Ansible и для чего он используется?
4. Как создать шаблонный файл в Ansible?
5. Какие способы передачи переменных поддерживаются Ansible?

---

## Ожидаемые результаты:
1. Успешный запуск playbook для установки и настройки Apache.
2. Создание файлов и директорий с заданными правами.
3. Использование шаблонов и переменных в playbook.

---

## Дополнительные задания (опционально):
1. Создайте playbook для развертывания веб-приложения на нескольких хостах.
2. Используйте модуль `yum` для установки пакетов на CentOS.
3. Настройте группу хостов в инвентори и примените playbook к ней.
4. Добавьте в playbook проверку версии установленного пакета.

---

## Вывод:
В ходе работы вы освоили основы Ansible, научились создавать playbooks, использовать модули и шаблоны для автоматизации задач. Ansible позволяет упростить управление инфраструктурой, минимизируя рутинные операции.

---

**Примечание:**  
- Все команды выполняйте от имени пользователя с правами sudo.  
- Убедитесь, что удалённые хосты имеют открытый порт SSH (22) и корректные настройки SELinux/firewalld (если необходимо).  
- Для углубленного изучения рекомендуется ознакомиться с официальной документацией Ansible.

# Лекция 24: Настройка Ceph

## Цели и задачи
- Изучить основные принципы работы распределенной файловой системы Ceph.
- Разобраться с архитектурой Ceph и её компонентами (MON, OSD, MDS).
- Научиться устанавливать и настраивать Ceph для создания отказоустойчивого хранилища.
- Понять процесс добавления узлов, создания пулов и управления данными.
- Изучить диагностику и устранение неисправностей в работе Ceph.

---

## Основной материал

### 1. Введение в Ceph

#### Что такое Ceph?
Ceph — это распределенная файловая система, которая предоставляет масштабируемое и отказоустойчивое хранение данных. Она поддерживает:
- Блоковое хранилище (RADOS Block Device, RBD).
- Объектное хранилище (RADOS Gateway, RGW).
- Файловое хранилище (Ceph File System, CephFS).

#### Зачем нужен Ceph?
- Высокая отказоустойчивость за счет репликации данных.
- Масштабируемость: возможность добавлять новые узлы без простоя.
- Поддержка различных типов хранилищ (блоки, объекты, файлы).

#### Основные компоненты:
1. MON (Monitor): Отслеживает состояние кластера и хранит карту данных.
2. OSD (Object Storage Daemon): Хранит данные и управляет дисками.
3. MDS (Metadata Server): Управляет метаданными для CephFS.
4. RADOS (Reliable Autonomic Distributed Object Store): Ядро Ceph, обеспечивающее распределенное хранение.

---

### 2. Установка Ceph

#### Используемые пакеты
Для установки Ceph используется пакетный менеджер. Например, в Ubuntu:

1. Добавьте репозиторий Ceph:
   wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
   echo deb https://download.ceph.com/debian-{version}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
2. Обновите список пакетов:
   apt update
3. Установите Ceph:
   apt install ceph-deploy

#### Создание кластера
Инициализируйте новый кластер:
ceph-deploy new node1

Настройте файл конфигурации:
/etc/ceph/ceph.conf

Пример базовой конфигурации:
[global]
fsid = <generated-fsid>
mon_initial_members = node1
mon_host = 192.168.1.10
public_network = 192.168.1.0/24

Установите Ceph на узлах:
ceph-deploy install node1 node2 node3

#### Запуск MON
Запустите монитор на первом узле:
ceph-deploy mon create-initial

---

### 3. Добавление OSD

#### Подготовка дисков
Подготовьте диски для использования в качестве OSD:
ceph-deploy osd create --data /dev/sdb node1
ceph-deploy osd create --data /dev/sdc node2

#### Проверка состояния
Проверьте состояние кластера:
ceph -s

Пример вывода:
cluster:
    id:     <fsid>
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum node1
    mgr: node1(active)
    osd: 2 osds: 2 up, 2 in

---

### 4. Создание пулов

#### Добавление пула
Создайте пул для хранения данных:
ceph osd pool create mypool 128 128

#### Настройка пула
Настройте параметры пула:
ceph osd pool set mypool size 3
ceph osd pool set mypool min_size 2

---

### 5. Работа с RADOS Gateway

#### Установка RGW
Установите RADOS Gateway для объектного хранилища:
ceph-deploy rgw create node1

#### Настройка RGW
Настройте порт для доступа:
[client.rgw.node1]
rgw_frontends = "civetweb port=8080"

Проверьте работу RGW:
curl http://node1:8080

---

### 6. Работа с CephFS

#### Создание MDS
Создайте Metadata Server для CephFS:
ceph-deploy mds create node1

#### Создание файловой системы
Создайте файловую систему:
ceph fs volume create myfs

Монтируйте CephFS:
mount -t ceph node1:/ /mnt/cephfs -o name=admin,secret=<key>

---

### 7. Диагностика и устранение неисправностей

#### Просмотр логов
Логи Ceph находятся в каталоге:
/var/log/ceph/

#### Распространенные проблемы:
- Недостаточное количество OSD для достижения кворума.
- Проблемы с сетевым подключением между узлами.
- Ошибки конфигурации.

---

### 8. Безопасность Ceph

#### Ограничение доступа
Ограничьте доступ к Ceph через файрвол:
ufw allow from 192.168.1.0/24 to any port 6789 proto tcp

#### Регулярное обновление
Регулярно обновляйте Ceph для защиты от уязвимостей.

---

### 9. Пример настройки Ceph

#### Шаги:
1. Добавьте репозиторий Ceph:
   wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
   echo deb https://download.ceph.com/debian-{version}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
2. Установите Ceph:
   apt install ceph-deploy
3. Создайте кластер:
   ceph-deploy new node1
4. Установите Ceph на узлах:
   ceph-deploy install node1 node2 node3
5. Запустите MON:
   ceph-deploy mon create-initial
6. Добавьте OSD:
   ceph-deploy osd create --data /dev/sdb node1

---

## Домашнее задание
1. Установите и настройте Ceph на трех виртуальных машинах.
2. Создайте пул и проверьте его работу.
3. Настройте RADOS Gateway и проверьте доступ к объектному хранилищу.
4. Создайте файловую систему CephFS и смонтируйте её.
5. Изучите логи и найдите записи о работе Ceph.

---

## Вопросы для самопроверки
1. Что такое Ceph и для чего он используется?
2. Как установить и настроить Ceph?
3. Как добавить узлы и создать пулы в Ceph?
4. Как настроить RADOS Gateway для объектного хранилища?
5. Какие меры безопасности можно применить для защиты Ceph?

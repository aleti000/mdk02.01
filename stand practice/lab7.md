# Лабораторное занятие №7: Использование LVM

## Тема
Использование Logical Volume Manager (LVM) для гибкого управления дисковым пространством

## Цель
Освоить создание и управление физическими томами (PV), группами томов (VG), логическими томами (LV); динамическое изменение размеров разделов без простоя системы. Все работы выполняются на сервере srv1 в среде Proxmox VE (PVE), где студент самостоятельно создает дополнительные диски.

## Схема сети
См. лабораторную работу №1.

## Схема дисков srv1

```
srv1 (172.16.0.2):
├── sda — системный диск
├── sdb — 1 ГБ (новый диск, добавлен в PVE)
├── sdc — 1 ГБ (новый диск, добавлен в PVE)
└── sdd — 1 ГБ (новый диск для расширения VG, добавлен в PVE)
```

## Теоретические основы

### Logical Volume Manager (LVM)

LVM — это система управления логическими томами в Linux, позволяющая гибко управлять дисковым пространством: объединять несколько физических дисков в пулы, создавать/расширять/уменьшать разделы онлайн без размонтирования.

#### Компоненты LVM:
- **PV (Physical Volume)** — физический диск или раздел, подготовленный для LVM (`pvcreate`).
- **VG (Volume Group)** — группа PV, пул свободного места (`vgcreate`, `vgextend`).
- **LV (Logical Volume)** — логический раздел из VG (`lvcreate`, `lvextend`).

#### Основные понятия:
- **PE (Physical Extent)** — минимальный блок на PV (~4MB).
- **LE (Logical Extent)** — блок на LV.

#### Преимущества LVM:
- Онлайн-ресайз (lvextend/lvresize).
- Snapshots (`lvcreate --snapshot`).
- Thin provisioning.

### Основные команды LVM

| Компонент | Описание | Основные команды |
|-----------|----------|------------------|
| PV | Физический том (подготовка диска для LVM) | `pvcreate [-f] /dev/sdX` — инициализация; `pvs` — список; `pvdisplay` — детали; `pvremove` — удаление |
| VG | Группа томов (пул из PV) | `vgcreate vg_name /dev/sdX /dev/sdY` — создание; `vgs` — список; `vgextend vg_name /dev/sdZ` — расширение; `vgreduce` — уменьшение; `vgremove` — удаление |
| LV | Логический том (раздел из VG) | `lvcreate -L 1G -n lv_name vg_name` — создание; `lvs` — список; `lvextend -L +500M vg/lv` — расширение; `lvresize` — изменение размера; `lvremove vg/lv` — удаление |

#### Подробное описание ключевых команд

**PV команды:**
- `pvcreate /dev/sdb` — Инициализирует диск/раздел как PV (добавляет LVM метаданные). Опция `-f` для force (игнор ошибок).
- `pvs` — Краткий список всех PV (размер, свободно).
- `pvdisplay` — Полная информация по PV (PE count, VG affiliation).

**VG команды:**
- `vgcreate demo_vg /dev/sdb /dev/sdc` — Создает VG из PV (суммирует их размер минус overhead).
- `vgextend demo_vg /dev/sdd` — Добавляет PV в VG (онлайн, увеличивает Free PE).
- `vgs` — Статус VG (VG Size, Free PE/Size).

**LV команды:**
- `lvcreate -L 800M -n lv_home demo_vg` — Создает LV фиксированного размера (`-l 10%VG` для %VG).
- `lvextend -L +500M /dev/demo_vg/lv_data` — Расширяет LV (онлайн для большинства ФС).
- `lvs` — Список LV (Path, Size, VG).

**Дополнительно для ФС после lvextend:**
- `resize2fs /dev/vg/lv` — Расширяет ext4 (онлайн если mounted).

### Диагностика дисков
```bash
lsblk          # Дерево блок-устройств
blkid          # Метки файловых систем
fdisk -l       # Таблицы разделов
df -hT         # Монтированные ФС с типами
```

## Методические указания

### Подготовка к выполнению работы

1. **Изучите теоретический материал** о LVM в Linux.
2. **Создайте 3 новых диска в Proxmox VE для VM srv1:**
   - Войдите в веб-интерфейс PVE: `https://<IP-PVE>:8006`.
   - Выберите VM `srv1` > **Hardware** > **Add** > **Hard Disk**.
   - Параметры для каждого диска (повторить 3 раза):
     - **Bus/Device:** VirtIO Block
     - **Storage:** local-lvm (или ваш storage)
     - **Size:** 1.00 GiB
     - **Cache:** None
     - **Discard:** on (опционально)
   - После добавления: **Start** или **Reboot** VM srv1.
3. **В srv1 проверьте новые диски:**
   ```bash
   lsblk
   # Должны появиться /dev/sdb, /dev/sdc, /dev/sdd по 1G
   partprobe    # Обновить таблицу разделов
   ```
4. **Убедитесь в работоспособности сети** из лабораторной №1.

### Рекомендации по выполнению

- **Очистка дисков перед использованием:** `wipefs -a /dev/sdX`
- **Резервные копии:** Перед изменениями `vgcfgbackup`.
- **Файловые системы:** ext4 поддерживает онлайн-расширение; для XFS — `xfs_growfs`.
- **fstab:** Добавляйте LV в `/etc/fstab` для автомонтирования: `UUID=<blkid> /mountpoint ext4 defaults 0 2`.
- **Проверка:** Всегда `lvs`, `vgs`, `pvs` после операций.

### Возможные проблемы и решения

| Проблема | Возможное решение |
|----------|-------------------|
| Диски не видны в lsblk | `partprobe`, `echo 1 > /sys/block/sdX/device/rescan` или reboot VM |
| pvcreate: device in use | `wipefs -a /dev/sdX`, `dd if=/dev/zero of=/dev/sdX bs=1M count=100` |
| Нет свободного места в VG | `vgextend vg_name /dev/new_pv` |
| lvextend не работает | Убедитесь LV mounted (ext4 ok online), `resize2fs /dev/vg/lv` |
| fstab не монтирует | Проверьте UUID (`blkid`), пробелы в пути |

## Ход работы

### Часть 1: Подготовка физических томов (PV) на SRV1

**На SRV1:**
```bash
# Проверить диски
lsblk

# Очистить новые диски
wipefs -a /dev/sdb
wipefs -a /dev/sdc
wipefs -a /dev/sdd

# Создать PV
pvcreate /dev/sdb
pvcreate /dev/sdc
pvcreate /dev/sdd

# Проверить PV
pvs
pvdisplay
```

### Часть 2: Создание группы томов (VG) и логических томов (LV)

**На SRV1:**
```bash
# Создать VG из sdb и sdc (2 ГБ всего)
vgcreate demo_vg /dev/sdb /dev/sdc

# Создать несколько LV:
lvcreate -L 800M -n lv_home demo_vg    # /home_new
lvcreate -L 800M -n lv_data demo_vg    # /data
lvcreate -L 400M -n lv_tmp  demo_vg    # /tmp_new

# Проверить
vgs
lvs
lvdisplay

# Форматировать
mkfs.ext4 /dev/demo_vg/lv_home
mkfs.ext4 /dev/demo_vg/lv_data
mkfs.ext4 /dev/demo_vg/lv_tmp

# Создать точки монтирования
mkdir -p /home_new /data /tmp_new

# Монтировать
mount /dev/demo_vg/lv_home /home_new
mount /dev/demo_vg/lv_data /data
mount /dev/demo_vg/lv_tmp  /tmp_new

# Добавить в fstab (используйте UUID из blkid)
blkid /dev/demo_vg/lv_home
# Пример: UUID=xxxx-xxxx /home_new ext4 defaults 0 2
echo 'UUID=<UUID_lv_home> /home_new ext4 defaults 0 2' >> /etc/fstab
# Аналогично для lv_data и lv_tmp

# Проверить
df -hT
```

### Часть 3: Расширение VG и LV

**На SRV1:**
```bash
# Добавить sdd в VG
vgextend demo_vg /dev/sdd

# Расширить lv_data на +500M
lvextend -L +500M /dev/demo_vg/lv_data

# Расширить ФС онлайн
resize2fs /dev/demo_vg/lv_data

# Проверить
lvs
vgs
df -h /data
```

## Задание

**На сервере srv1 самостоятельно:**

1. **Создайте 3 диска по 1 ГБ** в PVE для VM srv1 (sdb, sdc, sdd).
2. **Подготовьте PV:** Очистите и `pvcreate` на sdb, sdc, sdd.
3. **Создайте VG `myvg`** из sdb и sdc.
4. **Создайте несколько LV:**
   - `lv_data` 1 ГБ ext4 для /data
   - `lv_home` 800 МБ ext4 для /home_new
   - `lv_swap` 512 МБ swap
   - `lv_backup` остаток (~688 МБ) ext4 для /backup
5. **Отформатируйте, создайте директории, монтируйте** (`mount`, `swapon`), добавьте в `/etc/fstab`.
6. **Расширьте:** `vgextend myvg /dev/sdd`, `lvextend -L +500M lv_data`, `resize2fs`.

**Критерии выполнения работы:**
Работа считается выполненной если:
- VG `myvg` содержит 3 PV (~3 ГБ), LV: lv_data ~1.5 ГБ, lv_home 800M, lv_swap 512M, lv_backup ~688M (`lvs; vgs`).
- `df -hT` показывает /data, /home_new, /backup ext4 mounted.
- `swapon -s` показывает lv_swap.
- Расширение lv_data успешно (`df -h /data` >1ГБ).
- Система перезагружается без ошибок fstab.

## Возможные проблемы и решения

| Проблема | Возможное решение |
|----------|-------------------|
| lvcreate: not enough free space | Проверьте `vgs` free PE, уменьшите размер LV |
| mount: wrong fs type | `mkfs.ext4`, `blkid` проверить тип |
| swap не активируется | `mkswap /dev/myvg/lv_swap`, `swapon` |
| fstab boot error | `fsck`, отмонтировать и проверить UUID |
| PVE диск не добавлен | Проверить Hardware в PVE, rescan в VM |

## Полезные команды для диагностики

```bash
# Диски и LVM
lsblk
pvs; pvdisplay
vgs; vgdisplay
lvs; lvdisplay

# ФС и монтирование
blkid
df -hT
mount | grep lvm
swapon -s
cat /etc/fstab | grep lvm

# Операции (осторожно!)
lvextend -L +100M /dev/vg/lv
resize2fs /dev/vg/lv
lvremove vg/lv; vgreduce vg /dev/pv

# Резерв/восстановление
vgcfgbackup myvg

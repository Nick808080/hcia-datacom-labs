# Настройка IPv4-адресации и статической маршрутизации в Huawei VRP

## Описание проекта
Лабораторная работа по настройке межузловой IPv4-связности в топологии из трёх маршрутизаторов Huawei (`R1`, `R2`, `R3`) в эмуляторе eNSP (уровень HCIA-Datacom).

В рамках работы выполнена базовая инициализация устройств, распределение IPv4-подсетей, настройка виртуальных интерфейсов `LoopBack0`, конфигурирование статических, резервных (плавающих) и маршрутов по умолчанию с проверкой таблиц маршрутизации и отказоустойчивости.

## Топология и таблица адресации
Маршрутизаторы соединены в треугольник (`R1 — R2 — R3 — R1`).

![Топология сети](images/topology.png)

| Устройство | Интерфейс | IP-адрес / Маска | Подключение / Назначение |
| :--- | :--- | :--- | :--- |
| **R1** | `GigabitEthernet0/0/0` | `10.0.12.1/24` | Линк к R2 |
| **R1** | `GigabitEthernet0/0/2` | `10.0.13.1/24` | Линк к R3 |
| **R1** | `LoopBack0` | `10.0.1.1/32` | Идентификация / тест связности |
| **R2** | `GigabitEthernet0/0/0` | `10.0.12.2/24` | Линк к R1 |
| **R2** | `GigabitEthernet0/0/1` | `10.0.23.1/24` | Линк к R3 |
| **R2** | `LoopBack0` | `10.0.1.2/32` | Идентификация / тест связности |
| **R3** | `GigabitEthernet0/0/1` | `10.0.23.2/24` | Линк к R2 |
| **R3** | `GigabitEthernet0/0/2` | `10.0.13.2/24` | Линк к R1 |
| **R3** | `LoopBack0` | `10.0.1.3/32` | Идентификация / тест связности |

## Выполненные задачи
1. **Инициализация устройств:** Назначение имён (`R1`, `R2`, `R3`) в `System View`.
2. **Настройка IPv4-адресации:** Конфигурирование физических интерфейсов `GigabitEthernet` и виртуальных интерфейсов `LoopBack0` (`/32`), проверка перехода интерфейсов в состояние `UP`.
3. **Анализ таблицы маршрутизации:** Проверка появления `Direct`-маршрутов после настройки интерфейсов и `Static`-маршрутов после их добавления (`display ip routing-table`).
4. **Статическая и резервная маршрутизация (Floating Static Route):** Настройка основных маршрутов (`Preference = 60`), резервных маршрутов через транзитный узел `R2` (`Preference = 70`) и маршрутов по умолчанию (`0.0.0.0/0`).

## Ход работы (Консольный лог)

### 1. Настройка интерфейсов и LoopBack0

**Маршрутизатор R1:**
```text
<Huawei> system-view
[Huawei] sysname R1
[R1] interface GigabitEthernet 0/0/0
[R1-GigabitEthernet0/0/0] ip address 10.0.12.1 24
[R1-GigabitEthernet0/0/0] quit
[R1] interface GigabitEthernet 0/0/2
[R1-GigabitEthernet0/0/2] ip address 10.0.13.1 24
[R1-GigabitEthernet0/0/2] quit
[R1] interface loopback0
[R1-LoopBack0] ip address 10.0.1.1 32
[R1-LoopBack0] quit
```

**Маршрутизатор R2:**
```text
<Huawei> system-view
[Huawei] sysname R2
[R2] interface GigabitEthernet 0/0/0
[R2-GigabitEthernet0/0/0] ip address 10.0.12.2 24
[R2-GigabitEthernet0/0/0] quit
[R2] interface GigabitEthernet 0/0/1
[R2-GigabitEthernet0/0/1] ip address 10.0.23.1 24
[R2-GigabitEthernet0/0/1] quit
[R2] interface loopback0
[R2-LoopBack0] ip address 10.0.1.2 32
[R2-LoopBack0] quit
```

**Маршрутизатор R3:**
```text
<Huawei> system-view
[Huawei] sysname R3
[R3] interface GigabitEthernet 0/0/1
[R3-GigabitEthernet0/0/1] ip address 10.0.23.2 24
[R3-GigabitEthernet0/0/1] quit
[R3] interface GigabitEthernet 0/0/2
[R3-GigabitEthernet0/0/2] ip address 10.0.13.2 24
[R3-GigabitEthernet0/0/2] quit
[R3] interface loopback0
[R3-LoopBack0] ip address 10.0.1.3 32
[R3-LoopBack0] quit
```

### 2. Настройка статических маршрутов и проверка таблиц маршрутизации

Добавление статических маршрутов к `LoopBack0` удалённых маршрутизаторов:

```text
# На R1
[R1] ip route-static 10.0.1.2 32 10.0.12.2
[R1] ip route-static 10.0.1.3 32 10.0.13.2

# На R2
[R2] ip route-static 10.0.1.1 32 10.0.12.1
[R2] ip route-static 10.0.1.3 32 10.0.23.2

# На R3
[R3] ip route-static 10.0.1.2 32 10.0.23.1
[R3] ip route-static 10.0.1.1 32 10.0.13.1
```

Проверка таблицы маршрутизации на **R1** после настройки (`Destinations: 13`, добавились два `Static`-маршрута с флагами `RD`):

```text
[R1] display ip routing-table
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 13       Routes : 13

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       10.0.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
       10.0.1.2/32  Static  60   0          RD   10.0.12.2       GigabitEthernet0/0/0
       10.0.1.3/32  Static  60   0          RD   10.0.13.2       GigabitEthernet0/0/2
      10.0.12.0/24  Direct  0    0           D   10.0.12.1       GigabitEthernet0/0/0
      10.0.13.0/24  Direct  0    0           D   10.0.13.1       GigabitEthernet0/0/2
```

Проверка таблицы маршрутизации на **R2** (`10.0.1.1/32` доступен через `10.0.12.1`, `10.0.1.3/32` — через `10.0.23.2`):

```text
[R2] display ip routing-table
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 13       Routes : 13

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       10.0.1.1/32  Static  60   0          RD   10.0.12.1       GigabitEthernet0/0/0
       10.0.1.2/32  Direct  0    0           D   127.0.0.1       LoopBack0
       10.0.1.3/32  Static  60   0          RD   10.0.23.2       GigabitEthernet0/0/1
      10.0.12.0/24  Direct  0    0           D   10.0.12.2       GigabitEthernet0/0/0
      10.0.23.0/24  Direct  0    0           D   10.0.23.1       GigabitEthernet0/0/1
```

### 3. Резервные маршруты (Floating Static Route) и Default Route

```text
# Резервные маршруты между R1 и R3 через R2 (Preference 70)
[R1] ip route-static 10.0.1.3 32 10.0.12.2 preference 70
[R3] ip route-static 10.0.1.1 32 10.0.23.1 preference 70

# Маршруты по умолчанию
[R1] ip route-static 0.0.0.0 0 10.0.12.2
[R3] ip route-static 0.0.0.0 0 10.0.23.1
```

## Разбор контрольных вопросов
1. **Когда статический маршрут попадает в таблицу IP-маршрутизации и будет ли он добавлен при недоступном next-hop?**
   Статический маршрут инсталлируется в таблицу маршрутизации только при условии, что его `NextHop` (или исходящий интерфейс) достижим и находится в состоянии `UP`. Если следующий переход недоступен, маршрут остаётся в конфигурации, но не попадает в активную FIB.
2. **Какой IP-адрес источника будет у ICMP-пакета, если при пинге Loopback-интерфейса не указать параметр `-a`?**
   По умолчанию маршрутизатор использует в качестве Source IP адрес исходящего физического интерфейса, через который пакет покидает устройство (например, `10.0.12.1` или `10.0.13.1` вместо `10.0.1.1`). Если на удалённом узле нет обратного маршрута к транзитной подсети, `Echo Reply` не вернётся.
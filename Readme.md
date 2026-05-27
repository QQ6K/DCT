# 🐶 Smart Collar MVP (Умный ошейник — учебный проект)

## 📌 Описание проекта

Это учебный MVP-проект, моделирующий систему «умного ошейника» для собаки.

Система позволяет отслеживать базовое состояние питомца:
- сытость
- вода
- прогулка
- отдых

И фиксировать действия пользователей, которые влияют на эти показатели.

Проект сделан как симуляция без реального устройства, датчиков или IoT-интеграций. Все действия вводятся вручную через консоль.

Цель проекта — показать понимание:
- базовой архитектуры приложения
- работы с состоянием
- работы с базой данных
- бизнес-логики
- событийной модели

---

## 👤 Как это выглядит для пользователя

Пользователь взаимодействует с системой через простые команды:

- `feed` — покормить собаку  
- `water` — дать воду  
- `walk` — прогулка  
- `rest` — отдых  
- `status` — текущее состояние  
- `logs` — история действий  
- `tick` — имитация времени (1 час)

### 📊 Система состояния

У собаки есть набор показателей:

- hunger (сытость)
- water (вода)
- walk (прогулка)
- rest (отдых)

Каждый показатель:
- имеет максимум
- постепенно уменьшается со временем
- увеличивается от действий пользователя

---

## ⏱ Механика времени (`tick`)

Так как проект учебный и не использует фоновые процессы, время моделируется вручную.

Команда `feed` означает:
> прошёл 1 час

После выполнения:
- все показатели уменьшаются
- система имитирует естественные потребности собаки

Это упрощение сделано специально для MVP, чтобы не использовать:
- cron
- background workers
- scheduler’ы/планировщики

---

## ⚠️ Важные ограничения MVP

Проект сознательно упрощён.

Здесь нет:
- авторизации
- пользователей и ролей безопасности
- IoT-устройств
- фоновых сервисов
- сложной распределённой архитектуры

Хотя в реальности с собакой могут взаимодействовать разные люди, система не занимается безопасностью и разграничением доступа.

Любой пользователь считается равноправным участником системы.

---

## 🧱 Архитектура (слои системы)

Проект разделён на 3 слоя:

### 1. Presentation Layer
Слой взаимодействия с пользователем.

Отвечает за:
- ввод команд
- вывод состояния
- отображение логов

Не содержит бизнес-логики.

---

### 2. Business Logic Layer
Основная логика системы.

Отвечает за:
- изменение состояния собаки
- обработку команд
- уменьшение показателей через `tick`
- ограничения (0 ≤ score ≤ max)
- формирование логов

Это ядро проекта.

---

### 3. Data Layer
Слой работы с базой данных.

Отвечает за:
- сохранение состояния
- чтение состояния
- запись логов
- получение справочных данных

Не содержит бизнес-логики.

---

## 🗄 Структура базы данных

### event_types

Справочник типов событий.

Определяет:
- название события
- какой показатель изменяется
- величину изменения

Примеры:
- feed → hunger +3
- walk → walk +4
- water → water +2
- rest → rest +2

---

### dog_status

Текущее состояние собаки.

Хранит:
- hunger_score
- water_score
- walk_score
- rest_score

Особенности:
- в MVP только одна собака
- всегда одна запись

---

### dog_logs

Журнал всех действий.

Хранит:
- событие
- изменение состояния
- время выполнения
- (опционально) инициатор действия

Используется для:
- истории
- отладки
- анализа поведения системы

---

## 🔗 Связи между таблицами

Структура связей простая:

- `event_types` → используется в `dog_logs` как справочник типов событий
- `dog_logs` фиксирует каждое действие пользователя и его влияние на состояние
- `dog_status` хранит текущее состояние и не участвует в сложных связях

Итог:
- `event_types` задаёт правила
- `dog_status` хранит текущее состояние
- `dog_logs` хранит историю

---

## 📈 Возможные расширения проекта

Проект можно развивать без изменения базовой архитектуры:

- добавление нескольких собак
- полноценные пользователи
- web-интерфейс или мобильное приложение
- автоматическое время без `tick`
- уведомления о состоянии собаки
- графики активности
- GPS-трекинг прогулок
- интеграция с реальными IoT-устройствами
- аналитика поведения питомца

---

## 🧩 ER-диаграмма (Mermaid)

Диаграмма отражает структуру данных и связи между сущностями.

```mermaid
erDiagram

    Dog {
        int id
        int hunger_score
        int water_score
        int walk_score
        int rest_score
    }

    Event {
        int id
        string name
        int score_delta
    }

    DogLog {
        int id
        string event_name
        int delta
    }

   Dog ||--o{ DogLog : has
    Event ||--o{ DogLog : triggers



# 🍖 Команда FEED — полный путь выполнения (пример с псевдокодом)

---

## 👤 Ввод пользователя (Console)

Пользователь вводит команду:

feed

---

## 🧩 Presentation Layer (UI)

function main():

    input = read_console()

    command = parse_command(input)

    if command.type == "FEED":
        result = service.feed()
        print(result)

---

## 🧠 Service Layer (Business Logic)

function feed():

    status = repository.get_status()
    event = repository.get_event("feed")

    old_hunger = status.hunger

    status.hunger = status.hunger + event.delta

    if status.hunger > MAX_HUNGER:
        status.hunger = MAX_HUNGER

    repository.save_status(status)

    repository.log_event({
        event_name: "feed",
        old_value: old_hunger,
        new_value: status.hunger,
        delta: event.delta
    })

    return status

---

## 🗄 Data Layer (Repository)

function get_status():
    return DB.SELECT_ONE("SELECT * FROM dog_status")

function get_event(name):
    return DB.SELECT_ONE("SELECT * FROM event_types WHERE name = name")

function save_status(status):
    DB.EXECUTE("UPDATE dog_status SET hunger=?, water=?, walk=?, rest=?, updated_at=NOW()")

function log_event(log):
    DB.EXECUTE("INSERT INTO dog_logs(event_name, old_value, new_value, delta, created_at) VALUES (...)")

---

## 🗄 Database (изменение состояния)

BEFORE:
hunger = 5

EVENT:
feed (+3)

AFTER:
hunger = 8

dog_logs:
feed | 5 → 8 | +3

---

## 🔁 Общий поток выполнения

Console → UI → Service → Data → DB → Service → UI

---

## 🧠 Смысл

UI слой:
- принимает ввод
- вызывает сервис
- выводит результат

Service слой:
- применяет бизнес-логику
- изменяет состояние

Data слой:
- читает и записывает данные

Database:
- хранит состояние и историю


# ⏱ Команда TICK — полный путь выполнения (пример с псевдокодом)

---

## 👤 Ввод пользователя (Console)

Пользователь вводит команду:

tick

---

## 🧩 Presentation Layer (UI)

function main():

    input = read_console()

    command = parse_command(input)

    if command.type == "TICK":
        result = service.tick()
        print(result)

---

## 🧠 Service Layer (Business Logic)

function tick():

    status = repository.get_status()

    status.hunger = status.hunger - 1
    status.water = status.water - 1
    status.walk = status.walk - 1
    status.rest = status.rest - 1

    if status.hunger < 0:
        status.hunger = 0

    if status.water < 0:
        status.water = 0

    if status.walk < 0:
        status.walk = 0

    if status.rest < 0:
        status.rest = 0

    repository.save_status(status)

    repository.log_event({
        event_name: "tick",
        old_value: "multiple",
        new_value: "multiple",
        delta: -1
    })

    return status

---

## 🗄 Data Layer (Repository)

function get_status():
    return DB.SELECT_ONE("SELECT * FROM dog_status")

function save_status(status):
    DB.EXECUTE("UPDATE dog_status SET hunger=?, water=?, walk=?, rest=?, updated_at=NOW()")

function log_event(log):
    DB.EXECUTE("INSERT INTO dog_logs(event_name, old_value, new_value, delta, created_at) VALUES (...)")

---

## 🗄 Database (изменение состояния)

BEFORE:

hunger = 5  
water = 6  
walk = 7  
rest = 4  

EVENT:

tick (-1 all)

AFTER:

hunger = 4  
water = 5  
walk = 6  
rest = 3  

dog_logs:
tick | state decreased | -1

---

## 🔁 Общий поток выполнения

Console → UI → Service → Data → DB → Service → UI

---

## 🧠 Смысл

UI слой:
- принимает команду
- передаёт в сервис
- выводит результат

Service слой:
- уменьшает показатели со временем
- применяет ограничения (не ниже 0)

Data слой:
- сохраняет изменения
- пишет лог

Database:
- хранит текущее состояние и историю

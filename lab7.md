# Лабораторная работа №7 – Проектирование логической модели БД

## Выделяем основные сущности

На основе анализа предметной области и функциональных требований выделяем следующие **хранимые сущности**:

| Сущность | Почему она нужна | Ссылка на требования |
|----------|-----------------|---------------------|
| **Tourist** (Турист) | Хранит анкетные данные, категорию и историю поездок; основа для виз, груза, экскурсий | FR-TUR-01, FR-TUR-03 |
| **TouristGroup** (Группа) | Объединяет туристов в одну поездку, привязывает к рейсу и датам пребывания | FR-TUR-02, FR-FLT-03 |
| **Flight** (Рейс) | Учитывает расписание, тип ВС и лимиты загрузки для расчётов | FR-FLT-01, FR-FLT-02 |
| **Hotel** (Гостиница) | Справочник партнёрских отелей для нормализации данных о размещении | FR-HTL-01, FR-HTL-03 |
| **Room** (Номер) | Конкретные номера с типом и вместимостью для точного бронирования | FR-HTL-01, FR-HTL-02 |
| **Booking** (Бронирование) | Факт расселения группы в номер с датами заезда/выезда | FR-HTL-02 |
| **Visa** (Виза) | Статус визы для туриста в рамках конкретной поездки | FR-VIS-01, FR-VIS-02 |
| **ExcursionAgency** (Агентство) | Справочник принимающих агентств с контактами и рейтингом | FR-EXC-01, FR-EXC-04 |
| **Excursion** (Экскурсия) | Каталог маршрутов с датами, ценами и ограничениями | FR-EXC-01, FR-EXC-02 |
| **ExcursionBooking** (Запись на экскурсию) | Связывает туриста с экскурсией, учитывает оплату и статус | FR-EXC-02, FR-EXC-03 |
| **Cargo** (Груз) | Весовая ведомость туриста: места, вес, упаковка, страховка | FR-CRG-01, FR-CRG-02 |
| **FinancialTransaction** (Финансовая операция) | Учёт доходов/расходов по статьям для отчётности | FR-FIN-01, FR-FIN-02 |
| **SystemUser** (Пользователь системы) | Учётные записи сотрудников для аутентификации и аудита | NFR-SEC-01, NFR-SEC-02 |

### Промежуточные (связующие) сущности:

| Сущность | Зачем нужна | Тип связи |
|----------|-------------|-----------|
| **GroupTourist** | Реализует M:N между туристом и группой: один турист может летать в разных группах в разные сезоны | M:N + атрибут role_in_group |
| **ParentChildLink** | Явная связь "родитель - ребёнок" для контроля сопровождения (дети не ходят одни) | 1:N (само-ссылка на Tourist, вынесена для наглядности) |

---

## Атрибуты сущностей

### Tourist
```
tourist_id       : PK, INT
passport_number  : VARCHAR(20), NOT NULL, UNIQUE
first_name       : VARCHAR(50), NOT NULL
last_name        : VARCHAR(50), NOT NULL
birth_date       : DATE, NOT NULL
gender           : ENUM('M','F'), NOT NULL
category         : ENUM('leisure','cargo','child'), NOT NULL
parent_id        : FK -> Tourist.tourist_id, NULLABLE (только для category='child')
hotel_preference : VARCHAR(100), NULLABLE
created_at       : DATETIME, DEFAULT NOW()
```

### TouristGroup
```
group_id         : PK, INT
flight_id        : FK -> Flight.flight_id, NOT NULL
start_date       : DATE, NOT NULL
end_date         : DATE, NOT NULL
status           : ENUM('forming','in_progress','completed','cancelled'), DEFAULT 'forming'
is_deleted       : BOOLEAN, DEFAULT FALSE (soft delete)
```

### Flight
```
flight_id        : PK, INT
flight_number    : VARCHAR(20), NOT NULL
departure_date   : DATETIME, NOT NULL
aircraft_type    : ENUM('passenger','cargo','combi'), NOT NULL
max_passengers   : INT, NOT NULL, CHECK (max_passengers > 0)
max_cargo_weight : DECIMAL(10,2), NOT NULL, CHECK (max_cargo_weight >= 0)
max_volume_weight: DECIMAL(10,2), NOT NULL, CHECK (max_volume_weight >= 0)
```

### Hotel
```
hotel_id         : PK, INT
name             : VARCHAR(100), NOT NULL, UNIQUE
address          : TEXT, NOT NULL
phone            : VARCHAR(30), NULLABLE
star_rating      : INT, CHECK (star_rating BETWEEN 1 AND 5)
```

### Room
```
room_id          : PK, INT
hotel_id         : FK -> Hotel.hotel_id, NOT NULL
room_number      : VARCHAR(20), NOT NULL
room_type        : ENUM('single','double','suite','family'), NOT NULL
capacity         : INT, NOT NULL, CHECK (capacity > 0)
```

### Booking
```
booking_id       : PK, INT
group_id         : FK -> TouristGroup.group_id, NOT NULL
room_id          : FK -> Room.room_id, NOT NULL
check_in         : DATE, NOT NULL
check_out        : DATE, NOT NULL
status           : ENUM('confirmed','pending','cancelled'), DEFAULT 'pending'
```

### Visa
```
visa_id          : PK, INT
tourist_id       : FK -> Tourist.tourist_id, NOT NULL
group_id         : FK -> TouristGroup.group_id, NOT NULL
status           : ENUM('pending','issued','rejected','expired'), NOT NULL
issue_date       : DATE, NULLABLE
expiry_date      : DATE, NULLABLE
rejection_comment: TEXT, NULLABLE
```

### ExcursionAgency
```
agency_id        : PK, INT
name             : VARCHAR(100), NOT NULL, UNIQUE
contact_email    : VARCHAR(100), NULLABLE
phone            : VARCHAR(30), NULLABLE
rating           : DECIMAL(3,2), CHECK (rating BETWEEN 0 AND 5)
```

### Excursion
```
excursion_id     : PK, INT
agency_id        : FK -> ExcursionAgency.agency_id, NOT NULL
title            : VARCHAR(100), NOT NULL
description      : TEXT, NULLABLE
excursion_date   : DATETIME, NOT NULL
price            : DECIMAL(10,2), NOT NULL, CHECK (price >= 0)
min_age          : INT, DEFAULT 0
max_capacity     : INT, NOT NULL, CHECK (max_capacity > 0)
```

### ExcursionBooking
```
exc_booking_id   : PK, INT
tourist_id       : FK -> Tourist.tourist_id, NOT NULL
excursion_id     : FK -> Excursion.excursion_id, NOT NULL
status           : ENUM('registered','paid','attended','cancelled'), DEFAULT 'registered'
paid_amount      : DECIMAL(10,2), DEFAULT 0
```

### Cargo
```
cargo_id         : PK, INT
tourist_id       : FK -> Tourist.tourist_id, NOT NULL
group_id         : FK -> TouristGroup.group_id, NOT NULL
places_count     : INT, NOT NULL, CHECK (places_count > 0)
weight_kg        : DECIMAL(10,2), NOT NULL, CHECK (weight_kg > 0)
packaging_cost   : DECIMAL(10,2), DEFAULT 0
insurance_cost   : DECIMAL(10,2), DEFAULT 0
total_cost       : DECIMAL(10,2), NOT NULL
label_number     : VARCHAR(50), UNIQUE, NOT NULL
status           : ENUM('received','in_warehouse','shipped'), DEFAULT 'received'
```

### FinancialTransaction
```
transaction_id   : PK, INT
group_id         : FK -> TouristGroup.group_id, NULLABLE
tourist_id       : FK -> Tourist.tourist_id, NULLABLE
type             : ENUM('income','expense'), NOT NULL
category         : ENUM('hotel','flight','excursion','visa','cargo','airport','other'), NOT NULL
amount           : DECIMAL(12,2), NOT NULL
currency         : VARCHAR(3), DEFAULT 'USD'
transaction_date : DATE, NOT NULL
comment          : TEXT, NULLABLE
```

### SystemUser
```
user_id          : PK, INT
login            : VARCHAR(50), NOT NULL, UNIQUE
password_hash    : VARCHAR(255), NOT NULL
role             : ENUM('manager','accountant','admin'), NOT NULL
is_active        : BOOLEAN, DEFAULT TRUE
last_login       : DATETIME, NULLABLE
```

### GroupTourist (связующая)
```
id               : PK, INT (суррогатный ключ для упрощения индексации)
group_id         : FK -> TouristGroup.group_id, NOT NULL
tourist_id       : FK -> Tourist.tourist_id, NOT NULL
role_in_group    : ENUM('tourist','child','leader'), DEFAULT 'tourist'
# Уникальность пары:
UNIQUE KEY uk_group_tourist (group_id, tourist_id)
```

---

## Связи и ограничения целостности

### Основные связи
- `Flight` -1:N-> `TouristGroup`: одна группа летит одним рейсом
- `Tourist` -M:N-> `TouristGroup` через `GroupTourist`: турист может участвовать в разных группах
- `Tourist` -1:N-> `Tourist` (self): ребёнок привязан к родителю (проверка: если category='child', то parent_id NOT NULL)
- `Hotel` -1:N-> `Room`: в отеле много номеров
- `TouristGroup` -1:N-> `Booking`: группа может занимать несколько номеров
- `Booking` -N:1-> `Room`: номер резервируется под одну группу в период
- `Tourist` -1:N-> `Visa`: виза оформляется на туриста для конкретной поездки
- `ExcursionAgency` -1:N-> `Excursion`: агентство проводит много экскурсий
- `Tourist` -M:N-> `Excursion` через `ExcursionBooking`: запись на экскурсии
- `Tourist` -1:N-> `Cargo`: турист сдаёт один или несколько грузов
- `TouristGroup` / `Tourist` -1:N-> `FinancialTransaction`: операции привязываются к группе или конкретному туристу

### Ограничения (конкретика для СУБД)
```
# Первичные ключи
ALTER TABLE <each_table> ADD PRIMARY KEY (<*_id>);

# Внешние ключи с защитой от "сирот"
ALTER TABLE TouristGroup 
  ADD CONSTRAINT fk_group_flight 
  FOREIGN KEY (flight_id) REFERENCES Flight(flight_id)
  ON DELETE RESTRICT ON UPDATE CASCADE;

# Уникальные поля
ALTER TABLE Tourist ADD UNIQUE (passport_number);
ALTER TABLE Hotel ADD UNIQUE (name);
ALTER TABLE Cargo ADD UNIQUE (label_number);
ALTER TABLE SystemUser ADD UNIQUE (login);

# Обязательные поля (пример)
ALTER TABLE Tourist 
  MODIFY passport_number VARCHAR(20) NOT NULL,
  MODIFY first_name VARCHAR(50) NOT NULL,
  MODIFY category ENUM(...) NOT NULL;

# Проверочные ограничения
ALTER TABLE Flight 
  ADD CONSTRAINT chk_passengers CHECK (max_passengers > 0),
  ADD CONSTRAINT chk_cargo CHECK (max_cargo_weight >= 0);

ALTER TABLE Tourist 
  ADD CONSTRAINT chk_child_parent 
  CHECK (category != 'child' OR parent_id IS NOT NULL);

# Значения по умолчанию
ALTER TABLE Tourist 
  MODIFY created_at DATETIME DEFAULT CURRENT_TIMESTAMP;
ALTER TABLE FinancialTransaction 
  MODIFY currency VARCHAR(3) DEFAULT 'USD';
```

### Обоснование проектных решений
- Суррогатные ключи (`*_id`) выбраны вместо составных для упрощения индексации и обратной разработки в MySQL Workbench - стандартная практика для учебных проектов.
- Тип `DECIMAL` для денег и веса предотвращает ошибки округления (требование финансовой точности, FR-FIN-01).
- Поле `is_deleted` реализует мягкое удаление, что соответствует требованию сохранения истории операций (NFR-REL-04).
- Связь "родитель-ребёнок" через самореференцию закрывает бизнес-правило: дети не перемещаются без сопровождения (предметная область, раздел "Категории туристов").
- Все ограничения `FOREIGN KEY ... ON DELETE RESTRICT` защищают от случайного удаления записей с активными связями (визы, бронирования, груз) - прямое соответствие NFR-REL-04.
- CHECK-ограничения на категории, веса, рейтинги обеспечивают валидацию на уровне БД, а не только в интерфейсе (поддержка NFR-USE-03).

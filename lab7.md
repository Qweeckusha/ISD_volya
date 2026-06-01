# Лабораторная работа №7 – Проектирование логической модели БД (ERD)

## Выделение основных хранимых сущностей
На основе анализа предметной области, результатов [lab1](lab1.md) и функциональных требований [lab5](lab5.md) выделены следующие сущности:

| Сущность | Обоснование выделения |
|---|---|
| `Tourist` | Хранит анкетные данные. Прямо следует из FR-TUR и описания категорий туристов (`lab1`, `lab5`). |
| `Flight` | Учитывает рейсы, типы самолётов и лимиты загрузки. Необходимо для запросов 8 и 15 (`1.txt`, `lab5`). |
| `TouristGroup` | Объединяет туристов в одну поездку. Позволяет отслеживать статус группы и привязывать финансовые/визовые операции к конкретному вылету (`lab1`, `lab3`). |
| `GroupTourist` (связующая) | Реализует отношение M:N между `Tourist` и `TouristGroup`. Позволяет одному туристу летать в составе разных групп в разные периоды (`lab1` – история посещений). |
| `Hotel` | Справочник партнёрских гостиниц. Нормализовано: вынесено отдельно от номеров (`lab1`). |
| `Room` | Конкретные номера в гостиницах. Хранит тип и вместимость. Необходимо для точного бронирования (`lab5`, FR-HTL). |
| `Booking` | Факт расселения группы в номер. Связывает `TouristGroup`, `Room` и даты проживания (`lab5`, FR-HTL-02). |
| `Visa` | Учитывает статус визы для каждого туриста в рамках конкретной поездки. Отдельная сущность, т.к. визы меняются от рейса к рейсу (`lab5`, FR-VIS). |
| `ExcursionAgency` | Справочник принимающих агентств. Вынесен отдельно для нормализации и рейтинга качества (`lab5`, FR-EXC-04). |
| `Excursion` | Каталог маршрутов, дат, цен и ограничений. Привязан к агентству (`lab5`). |
| `ExcursionBooking` | Связующая таблица записи туристов на экскурсии. Учитывает статус оплаты и участия (`lab5`, FR-EXC-02). |
| `Cargo` | Грузовые ведомости по каждому туристу. Хранит вес, места, страховку, маркировку (`lab1`, `lab5` FR-CRG). |
| `FinancialTransaction` | Учёт доходов/расходов. Привязывается к группе, категории операции и дате (`lab5`, FR-FIN). |
| `SystemUser` | Учётные записи сотрудников представительства. Требуется для аутентификации и ролевой модели (`lab6`, NFR-SEC). |

**Обоснование:** Список сущностей полностью покрывает модули из `lab5.md` и закрывает все 15 типовых запросов из `1.txt`. Разделение на справочники (`Hotel`, `ExcursionAgency`) и операционные таблицы (`Booking`, `FinancialTransaction`) соответствует принципам реляционного проектирования и устраняет избыточность, выявленную в `lab3.md` (проблема "фрагментированного учёта" и "дублирования ввода").

---

## Шаг 2. Атрибуты, ключи и абстрактные типы данных
Для каждой сущности определены атрибуты. Типы данных указаны абстрактно, готовые к маппингу в MySQL (`INT`, `VARCHAR`, `DATE`, `DECIMAL`, `ENUM` и т.д.).

| Сущность | Атрибуты (PK подчёркнуты, FK помечены) | Абстрактные типы |
|---|---|---|
| `Tourist` | **`tourist_id`**, `passport_number` (U), `first_name`, `last_name`, `birth_date`, `gender`, `category` (leisure/cargo/child), `parent_tourist_id` (FK → Tourist), `created_at` | INT, VARCHAR, DATE, ENUM, FK, DATETIME |
| `Flight` | **`flight_id`**, `flight_number`, `departure_date`, `aircraft_type`, `max_passengers`, `max_cargo_weight`, `max_volume_weight` | INT, VARCHAR, DATE, VARCHAR, INT, DECIMAL, DECIMAL |
| `TouristGroup` | **`group_id`**, `flight_id` (FK → Flight), `start_date`, `end_date`, `status`, `is_deleted` | INT, FK, DATE, DATE, ENUM, BOOLEAN |
| `GroupTourist` | `group_id` (FK → TouristGroup), `tourist_id` (FK → Tourist), **`id`** (PK-суррогат), `role_in_group` | FK, FK, INT, VARCHAR |
| `Hotel` | **`hotel_id`**, `name` (U), `address`, `phone`, `star_rating` | INT, VARCHAR, VARCHAR, VARCHAR, INT |
| `Room` | **`room_id`**, `hotel_id` (FK → Hotel), `room_number`, `room_type`, `capacity` | INT, FK, VARCHAR, VARCHAR, INT |
| `Booking` | **`booking_id`**, `group_id` (FK → TouristGroup), `room_id` (FK → Room), `check_in`, `check_out`, `status` | INT, FK, FK, DATE, DATE, ENUM |
| `Visa` | **`visa_id`**, `tourist_id` (FK → Tourist), `group_id` (FK → TouristGroup), `status`, `issue_date`, `expiry_date`, `rejection_comment` | INT, FK, FK, ENUM, DATE, DATE, TEXT |
| `ExcursionAgency` | **`agency_id`**, `name` (U), `contact_email`, `phone`, `rating` | INT, VARCHAR, VARCHAR, VARCHAR, DECIMAL |
| `Excursion` | **`excursion_id`**, `agency_id` (FK → ExcursionAgency), `title`, `description`, `excursion_date`, `price`, `min_age`, `max_capacity` | INT, FK, VARCHAR, TEXT, DATE, DECIMAL, INT, INT |
| `ExcursionBooking` | **`exc_booking_id`**, `tourist_id` (FK → Tourist), `excursion_id` (FK → Excursion), `status`, `paid_amount` | INT, FK, FK, ENUM, DECIMAL |
| `Cargo` | **`cargo_id`**, `tourist_id` (FK → Tourist), `group_id` (FK → TouristGroup), `places_count`, `weight_kg`, `packaging_cost`, `insurance_cost`, `total_cost`, `label_number` (U), `status` | INT, FK, FK, INT, DECIMAL, DECIMAL, DECIMAL, DECIMAL, VARCHAR, ENUM |
| `FinancialTransaction` | **`transaction_id`**, `group_id` (FK → TouristGroup, NULL), `tourist_id` (FK → Tourist, NULL), `type` (income/expense), `category`, `amount`, `currency`, `transaction_date`, `comment` | INT, FK, FK, ENUM, VARCHAR, DECIMAL, VARCHAR, DATE, TEXT |
| `SystemUser` | **`user_id`**, `login` (U), `password_hash`, `role`, `is_active`, `last_login` | INT, VARCHAR, VARCHAR, ENUM, BOOLEAN, DATETIME |

**Обоснование:** 
- Атрибуты взяты напрямую из `lab1.md` (основные хранимые сущности) и адаптированы под `lab5.md` (FR-TUR-03, FR-CRG-02, FR-FIN-01).
- Поля `is_deleted` добавлены для реализации мягкого удаления (soft delete), что соответствует требованию `NFR-REL-04` из `lab6.md` (запрет на удаление записей с активными связями).
- Суррогатные ключи (`*_id`) выбраны вместо составных для упрощения индексации и обратной разработки в MySQL Workbench, что является стандартной практикой для учебных проектов 3 курса.
- Типы `DECIMAL` использованы для денег и веса, чтобы избежать ошибок округления (требование финансовой точности из `lab3.md`).

---

## Шаг 3. Связи между сущностями и промежуточные таблицы
Связи спроектированы с учётом бизнес-процессов AS-IS/TO-BE (`lab3.md`) и функциональных модулей (`lab5.md`).

| Связь | Тип | Описание | Обоснование |
|---|---|---|---|
| `Flight` → `TouristGroup` | 1:N | Один рейс может обслуживать несколько групп (или одна группа = один рейс). В модели принято: группа привязана к одному рейсу. | Запросы 8, 15 (`1.txt`). Учёт загрузки самолёта. |
| `Tourist` ↔ `TouristGroup` | M:N | Реализовано через `GroupTourist`. Один турист может быть в разных группах в разные сезоны. | `lab1` (история посещений), `lab5` (FR-TUR-03). |
| `Tourist` → `Tourist` (self) | 1:N | `parent_tourist_id`. Ребёнок привязан к родителю. | `lab3` (проблема "отсутствие контроля за детьми"), правило из `1.txt`. |
| `Hotel` → `Room` | 1:N | В гостинице много номеров. | Нормализация 3НФ. Устраняет повторение адреса гостиницы в каждом бронировании. |
| `TouristGroup` → `Booking` | 1:N | Группа может бронировать несколько номеров. Номер резервируется под одну группу в конкретный период. | `lab5` (FR-HTL-01). Учёт загрузки отелей (запрос 5). |
| `Booking` → `Room` | N:1 | Бронь привязана к конкретному номеру. | Логика расселения. |
| `TouristGroup` → `Visa` | 1:N | Виза оформляется на туриста для конкретной поездки. | `lab5` (FR-VIS-01). Позволяет хранить историю отказов/выдач. |
| `ExcursionAgency` → `Excursion` | 1:N | Агентство организует множество экскурсий. | `lab5` (FR-EXC-04). Позволяет ранжировать агентства по среднему рейтингу. |
| `Tourist` ↔ `Excursion` | M:N | Реализовано через `ExcursionBooking`. | `lab5` (FR-EXC-02). Запрос 6, 7 (`1.txt`). |
| `Tourist` → `Cargo` | 1:N | Турист может сдавать несколько грузовых мест в рамках одной поездки. | `lab5` (FR-CRG-01). Запрос 9, 12. |
| `TouristGroup` → `FinancialTransaction` | 1:N | Операции учитываются по группе. Опционально привязка к `Tourist`. | `lab5` (FR-FIN-01). Запрос 10, 11, 13. |
| `SystemUser` | Нет прямых связей | Отдельный контур безопасности. В будущем можно добавить `created_by`/`updated_by` в операционные таблицы для аудита. | `lab6` (NFR-SEC-01, 02, 04). |

**Обоснование:** Промежуточные таблицы (`GroupTourist`, `ExcursionBooking`) введены строго для разрешения отношений M:N, что является обязательным условием 3НФ. Связь "Турист-Ребёнок" реализована через самореференцию, что закрывает бизнес-правило из `1.txt` и проблему из `lab3.md` без создания отдельной сущности.

---

## Шаг 4. Ограничения и целостность данных
На этом этапе формализуются правила, гарантирующие корректность данных в БД.

| Тип ограничения | Применение в модели | Обоснование (ссылка на прошлые работы) |
|---|---|---|
| **PRIMARY KEY** | На всех таблицах (`*_id`) | Гарантирует уникальную идентификацию записей. Стандарт реляционных БД. |
| **FOREIGN KEY** | Все связи из Шага 3. Правила: `ON DELETE RESTRICT`, `ON UPDATE CASCADE` | Предотвращает появление "сиротских" записей. Соответствует `NFR-REL-04` (`lab6`) – запрет удаления активных записей. |
| **UNIQUE** | `Tourist.passport_number`, `Hotel.name`, `ExcursionAgency.name`, `Cargo.label_number`, `SystemUser.login` | Исключает дублирование критических идентификаторов. Паспорт и логин должны быть уникальными по определению. |
| **NOT NULL** | `passport_number`, `first_name`, `last_name`, `birth_date`, `category`, `flight_number`, `check_in`, `issue_date`, `password_hash`, `amount` | Обязательные поля для функционирования системы. Заполнение регламентировано `lab5` (FR-TUR-01, FR-VIS-01 и др.). |
| **CHECK** | `category IN ('leisure', 'cargo', 'child')` | Категоризация строго из ТЗ (`1.txt`). |
| **CHECK** | `weight_kg > 0`, `max_passengers > 0` | Физическая непротиворечивость данных. |
| **CHECK / Логическое** | `IF category = 'child' THEN parent_tourist_id IS NOT NULL` | Реализует правило сопровождения детей (`1.txt`, `lab3` проблема 7). |
| **DEFAULT** | `status = 'pending'`, `is_deleted = FALSE`, `created_at = NOW()` | Упрощает внесение данных, автоматизирует метки времени и статусы по умолчанию. |
| **Аудит** | `created_by`, `updated_at` (рекомендуется добавить ко всем операционным таблицам) | Реализует `NFR-SEC-04` (`lab6`) – ведение журнала действий пользователей. |

**Обоснование:** Ограничения напрямую вытекают из нефункциональных требований безопасности и надёжности (`lab6.md`). `CHECK` и `NOT NULL` предотвращают ошибки ввода на уровне СУБД, а не только в UI (`NFR-USE-03`). `RESTRICT` на удаление защищает от случайной потери данных, что критично для финансовой отчётности и визовых пакетов.

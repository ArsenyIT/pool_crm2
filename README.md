# Система учёта посетителей бассейна
## 1. Запуск приложения
### Для запуска приложения введите в окне терминала команду: 
### uvicorn main:app --reload
### База данных создастся автоматически
## 2. Библиотеки, необходимые для работы приложения:
- `fastapi` 
- `uvicorn`
- `jinja2`
- `python-multipart`
- `sqlalchemy`
- `vk_api`
- `bcrypt`

# Database (Удалов Арсений)
## init_db.py
## Используется для создания таблиц и заполнения их тестовыми данными
### 1. Библиотеки, которые мы импортируем:
* import sqlite3 
* import os 
* from datetime import datetime, time, date, timedelta
* from database import db_instance, get_db_cursor, DATABASE_PATH (файл database)
### 2. Функции:
* table_exists() - Проверяет, существует ли таблица в базе данных SQLite
* create_tables() - Создаёт все таблицы:
* ensure_tables_exist() - Проверяет существование таблиц и создаёт их при необходимости
* seed_test_data() - Заполнение тестовыми данными для разработки
* drop_all_tables() - Удаляет ВСЕ таблицы из базы данных (без пересоздания)
* reset_database() - Удаляет все таблицы и создаёт их заново с тестовыми данными
* recreate_database() - Полностью пересоздаёт базу данных
* reset_database_safe() - Безопасная версия сброса с подтверждением
* show_tables() - Показать список всех таблиц в базе данных
* get_database_info() - Получить подробную информацию о базе данных
* create_full_database() - Создаёт полную базу данных с таблицами и тестовыми данными
### 3. Таблицы, которые мы создаём (в правильном порядке):
* Таблица родителей (parents)
* Таблица детей (children)
* Таблица тренеров (trainers)
* Таблица групп (groups)
* Таблица зачислений (enrollments)
* Таблица расписания (schedule)
* Таблица посещаемости(attendance)
* Таблица заявок (applications)
* Таблица логов администратора (admin_logs)
* Таблица уведомлений (notifications)
### 4. Самостоятельный запуск init_db
1) if len(sys.argv) < 2: Если аргументов меньше 2 (то есть нет команды)
2) Выводим справку (help)
3) sys.exit(0) Выходим из программы с кодом 0 (успешное завершение)
4) command = sys.argv[1] Берём первый аргумент (команду)
5) if command == "create": create_tables() Создание таблиц 
6) elif command == "seed": seed_test_data() Заполнение тестовыми данными 
7) elif command == "reset": reset_database() Полный сброс (удаление + создание + заполнение)
8) elif command == "drop": drop_all_tables() Удаление всех таблиц 
9) elif command == "recreate": recreate_database() Пересоздание с удалением файла 
10) elif command == "show": show_tables() Показать список таблиц 
11) elif command == "info": get_database_info() Показать подробную информацию о БД 
12) elif command == "full": create_full_database() Создать полную БД (таблицы + данные)
13) else: Если команда не распознана 
14) print(f"❌ Unknown command: {command}")
15) print("Use: create, seed, reset, drop, recreate, show, info, full")
### 5. Индексы для производительности:
* Индекс на внешний ключ родителя в таблице children
* Индекс на ID ребёнка в таблице зачислений 
* Индекс на ID группы в таблице зачислений 
* Индекс на ID зачисления в таблице посещаемости 
* Индекс на дату в таблице посещаемости 
* Индекс на статус в таблице заявок 
* Индекс на телефон в таблице заявок 
* Индекс на группу в таблице расписания 
* Индекс на статус в таблице уведомлений 
* Индекс на тренера в таблице групп
## database.py
## Управляет подключением к базе данных
### 1. Библиотеки, которые мы импортируем:
* import sqlite3 
* from contextlib import contextmanager 
* from fastapi import FastAPI, Depends, HTTPException
### 2. Класс Database 
1) def __init__(): Класс для управления подключением к SQLite
2) get_connection(): Создаёт соединение с SQLite при первом запросе
3) close(): Закрыть соединение с БД
### 3. Функции:
* get_db_cursor() - Автоматически управляет транзакциями
* get_db() - Используется в эндпоинтах для получения курсора
* init_database() - Функция для инициализации БД (будет вызвана при старте)  

# BACKEND (Смирнов Александр)

## main.py — основное приложение FastAPI

### Используемые библиотеки
- `fastapi` — веб-фреймворк  
- `uvicorn` — ASGI‑сервер  
- `jinja2` — шаблонизатор  
- `python-multipart` — работа с формами
- `sqlite3` — работа с БД  
- `datetime`, `secrets`, `typing` — стандартные библиотеки  

### Основные функции и модули

#### 1. Управление сессиями и аутентификация
- `generate_session_token()` — создаёт уникальный токен для сессии  
- `active_sessions` — словарь {token: {user_id, user_type, login}}  
- `get_current_user()` — извлекает пользователя из cookie и БД  
- Функции проверки ролей:  
  - `require_parent()`  
  - `require_trainer()`  
  - `require_admin()`  
  - `require_trainer_or_admin()`

#### 2. Публичные маршруты (без авторизации)
| Эндпоинт    | Описание                       |
|-------------|--------------------------------|
| `/`         | Главная страница               |
| `/login`    | Вход (админ, тренер, родитель) |
| `/logout`   | Выход из системы               |
| `/apply`    | Подача заявки на зачисление    |
| `/gallery`  | Страница галереи               |

#### 3. Родитель (личный кабинет)
| Эндпоинт                   | Описание                                                     |
|----------------------------|--------------------------------------------------------------|
| `/parent/profile`          | Список всех детей родителя                                   |
| `/parent/child/{child_id}` | Детальная карточка ребёнка: группа, расписание, посещаемость |

#### 4. Тренер
| Эндпоинт                                              | Описание                                             |
|-------------------------------------------------------|------------------------------------------------------|
| `/trainer/dashboard`                                  | Список групп (свои или все, если админ)              |
| `/trainer/group/{group_id}`                           | Просмотр группы: ученики, расписание                 |
| `/trainer/group/{group_id}/add_student`               | Добавление ученика в группу                          |
| `/trainer/group/{group_id}/remove_student/{child_id}` | Отчисление ученика                                   |
| `/trainer/student/{child_id}/edit`                    | Редактирование педагогических данных ученика         |
| `/trainer/group/{group_id}/attendance`                | Журнал посещаемости (отметка на выбранную дату)      |
| `/trainer/group/{group_id}/schedule/edit`             | Управление расписанием (добавление/удаление занятий) |

#### 5. Администратор
| Эндпоинт                              | Описание                                                  |
|---------------------------------------|-----------------------------------------------------------|
| `/admin/applications`                 | Список заявок с фильтрацией (статус, возраст, смена)      |
| `/admin/application/{app_id}`         | Карточка заявки                                           |
| `/admin/application/{app_id}/approve` | Одобрение заявки (автоматический или ручной выбор группы) |
| `/admin/application/{app_id}/reject`  | Отклонение заявки с указанием причины                     |
| `/admin/trainers`                     | Список тренеров                                           |
| `/admin/trainer/create`               | Форма создания тренера                                    |
| `/admin/trainer/{trainer_id}/edit`    | Редактирование тренера                                    |
| `/admin/trainer/{trainer_id}/delete`  | Удаление тренера (обнуляет привязку групп)                |
| `/admin/groups`                       | Список групп                                              |
| `/admin/group/create`                 | Форма создания группы                                     |
| `/admin/group/{group_id}/edit`        | Редактирование группы                                     |
| `/admin/group/{group_id}/delete`      | Удаление группы (с переводом учеников или отчислением)    |
| `/admin/api/stats`                    | JSON‑статистика для дашборда админа                       |
| `/admin/api/recent_applications`      | Последние 10 заявок (JSON)                                |

#### 7. Особенности реализации
- **Сессии** — cookie с токеном, хранилище в памяти (при перезапуске сервера сессии сбрасываются).   
- **Права доступа** — роли `parent`, `trainer`, `admin`.  
- **Даты** — все поля `datetime` преобразуются в строки перед передачей в шаблоны (чтобы избежать ошибок `datetime` не subscriptable).  
- **Вспомогательные функции**:
  - `can_manage_group()` — проверка доступа тренера к группе  
  - `auto_select_group()` — подбор группы по возрасту, году обучения, смене

# FRONTEND (Яблоков Тимур)

## Обзор интерфейса

Swim CRM - это веб-приложение для управления занятиями в бассейне. Интерфейс построен на Bootstrap 5 с кастомными анимациями и адаптивным дизайном.

## Структура страниц

### Публичные страницы

| Страница        | Путь                   | Описание                                                  |
|-----------------|------------------------|-----------------------------------------------------------|
| Главная         | '/'                    | Landing page с анимированными  фоном (вода, дельфины, кит |
| Вход            | '/login'               | Форма для родителей, тренеров и администраторов           |
| Запись          | '/apply'               | Форма подачи заявки на зачисление                         |
| Успех           | '/application_success' | Подтверждение отправки заявки                             |
| Галерея         | '/gallery'             | Карусель фотографий с эффектом размытия и колёсиком мыши  |
| Доступ запрещён | '/access_denied'       | Страница ошибки доступа                                   |

### Родительская зона

| Страница         | Путь                 | Описание                                                         |
|------------------|----------------------|------------------------------------------------------------------|
| Мои дети         | '/parent/profile'    | Список всех детей с карточками и статусом зачисления             |
| Карточка ребёнка | '/parent/child/{id}' | Детальная информация: группа, расписание, посещаемость (30 дней) |

### Тренерская зона

| Страница               | Путь                                | Описание                                                    |
|------------------------|-------------------------------------|-------------------------------------------------------------|
| Дашборд                | '/trainer/dashboard'                | Список групп (тренер видит только свои, админ - все)        |
| Группа                 | '/trainer/group/{id}'               | Информация о группе, список учеников, расписание            |
| Добавление ученика     | '/trainer/group/{id}/add_student'   | Выбор не зачисленного ученика                               |
| Редактирование ученика | '/trainer/group/{id}/edit'          | Изменение данных ученика (ФИО, возраст, школа, стаж, смена) |
| Посещаемость           | '/trainer/group/{id}/attendance'    | Журнал с отметками (present/absent/sick/excused)            |
| Расписание             | '/trainer/group/{id}/schedule/edit' | Добавление/удаление занятий в расписании                    |

### Административная зона

| Страница              | Путь                      | Описание                                           |
|-----------------------|---------------------------|----------------------------------------------------|
| Панель администратора | '/dashboard'              | Статистика и быстрые действия                      |
| Заявки                | '/admin/applications'     | Список заявок с фильтрацией                        |
| Карточка заявки       | '/admin/application/{id}' | Просмотр и обработка заявки (одобрение/отклонение) |
| Тренеры               | '/admin/trainers'         | CRUD тренеров                                      |
| Группы                | '/admin/groups'           | CRUD групп                                         |

## Компоненты интерфейса

### Навигация (Navbar)
- Логотип
- Роли: Администратор/Тренер/Родитель
- Кнопка выхода (для авторизованных)
- Кнопка записи (для гостей)

### Футер
- Копирайт
- Бегущий краб (анимированный, кликабельный)

# Взаимодействие Database с BACKEND и FRONTEND (Удалов Арсений)

В таблице описаны ключевые сценарии работы CRM‑системы бассейна.  
Для каждого сценария указаны:

- **Frontend** – HTML‑шаблон (или API‑запрос) и отправляемые данные.
- **Backend** – FastAPI эндпоинт, метод HTTP и логика обработки.
- **Database** – таблицы SQLite и основные SQL‑операции.

## Публичные страницы (без авторизации)

| Сценарий | Frontend | Backend | Database |
|----------|----------|---------|----------|
| **Подача заявки на зачисление** | Форма `application_form.html` → `POST /apply` с полями: `parent_full_name`, `parent_phone`, `child_full_name`, `child_age`, `school_name`, `shift`, `swimming_years`, `desired_lessons_per_week` и др. | Функция `submit_application`. Валидация возраста, смены, кол-ва занятий. Вставка записи в таблицу `applications`. | `INSERT INTO applications (...) VALUES (...)` <br>Таблица: `applications` |
| **Страница входа** | Форма `login.html` → `POST /login` с полями `username`, `password`. | Функция `login`. Проверка по таблицам `admins`, `trainers`, `parents` с верификацией пароля (через `verify_password`). При успехе – создание сессии (`active_sessions`), установка cookie `session_token`. | `SELECT ... FROM admins WHERE login = ?`<br>`SELECT ... FROM trainers WHERE login = ?`<br>`SELECT ... FROM parents WHERE phone = ?` |
| **Главная / Галерея** | `GET /` → `index.html`<br>`GET /gallery` → `gallery.html`<br>`GET /api/gallery/images` → список изображений. | Отдача статического шаблона. API читает папку `static/images/gallery`. | (нет) |

## Личный кабинет родителя (роль `parent`)

| Сценарий | Frontend | Backend | Database |
|----------|----------|---------|----------|
| **Просмотр списка детей** | `GET /parent/profile` → `parent_profile.html` | `parent_profile`. Проверка авторизации (`require_parent`). Выборка детей текущего родителя и их активных групп. | `SELECT ... FROM children WHERE parent_id = ?`<br>`SELECT g.id, g.name, ... FROM enrollments e JOIN groups g ... WHERE e.child_id = ? AND e.is_active = 1` |
| **Детальная карточка ребёнка** | `GET /parent/child/{child_id}` → `child_details.html` | `child_details`. Проверка принадлежности ребёнка родителю. Выборка информации о ребёнке, его группы, расписания (из `schedule`) и последних записей посещаемости (из `attendance`). | `SELECT * FROM children WHERE id = ? AND parent_id = ?`<br>`SELECT g.*, t.full_name FROM enrollments e JOIN groups g ...`<br>`SELECT * FROM schedule WHERE group_id = ?`<br>`SELECT date, status FROM attendance WHERE enrollment_id = ?` |

## Работа тренера / администратора (роли `trainer`, `admin`)

| Сценарий | Frontend | Backend | Database |
|----------|----------|---------|----------|
| **Просмотр групп (дашборд тренера)** | `GET /trainer/dashboard` → `trainer_dashboard.html` | `trainer_dashboard`. Для тренера – только его группы, для админа – все. Подсчёт количества зачисленных учеников. | `SELECT g.*, t.full_name, COUNT(e.id) as enrolled FROM groups g LEFT JOIN trainers t ... LEFT JOIN enrollments e ... WHERE (условие по роли) GROUP BY g.id` |
| **Управление учениками в группе** | `GET /trainer/group/{group_id}/add_student` → форма выбора ребёнка.<br>`POST /trainer/group/{group_id}/add_student` с `child_id`. | Проверка прав (`can_manage_group`). Проверка свободных мест, существования ребёнка. Вставка в `enrollments`. Отправка уведомления родителю (запись в `notifications`). | `SELECT max_students FROM groups WHERE id = ?`<br>`SELECT COUNT(*) FROM enrollments WHERE group_id = ? AND is_active = 1`<br>`INSERT INTO enrollments (child_id, group_id, enrolled_at) VALUES (...)`<br>`INSERT INTO notifications (user_id, user_type, ...) VALUES (...)` |
| **Отметка посещаемости** | `GET /trainer/group/{group_id}/attendance?date=...` → форма `attendance_mark.html`.<br>`POST /trainer/group/{group_id}/attendance` с полями `status_{child_id}`. | `attendance_mark_form` – выборка учеников группы и существующих отметок.<br>`attendance_mark_save` – обновление или вставка записей в `attendance`. | `SELECT c.id, c.full_name FROM enrollments e JOIN children c ...`<br>`SELECT status FROM attendance WHERE enrollment_id = ? AND date = ?`<br>`INSERT OR UPDATE attendance (enrollment_id, date, status, mark_time)` |
| **Редактирование данных ученика** | `GET /trainer/student/{child_id}/edit` → форма `edit_student.html`<br>`POST /trainer/student/{child_id}/edit` с полями `full_name`, `age`, `school_name` и др. | `edit_student_form` – проверка прав (тренер ученика или админ).<br>`edit_student_submit` – валидация и обновление `children`. | `UPDATE children SET full_name = ?, age = ?, class_number = ?, school_name = ?, swimming_years = ?, shift = ?, desired_lessons_per_week = ? WHERE id = ?` |
| **Управление расписанием группы** | `GET /trainer/group/{group_id}/schedule/edit` → `edit_schedule.html`<br>`POST /trainer/group/{group_id}/schedule/add` с полями `weekday, start_time, end_time, location, is_recurring, single_date`<br>`POST /trainer/schedule/{schedule_id}/delete` | Выборка текущего расписания. Добавление новой записи в `schedule`. Удаление записи (с проверкой принадлежности группе). | `SELECT * FROM schedule WHERE group_id = ?`<br>`INSERT INTO schedule (group_id, weekday, start_time, end_time, ...) VALUES (...)`<br>`DELETE FROM schedule WHERE id = ?` |

## Администрирование (роль `admin`)

| Сценарий | Frontend | Backend | Database |
|----------|----------|---------|----------|
| **Обработка заявок** | `GET /admin/applications` → `admin_applications.html` (с фильтрами).<br>`POST /admin/application/{app_id}/approve` (с опциональным `group_id`).<br>`POST /admin/application/{app_id}/reject` (с `reason`). | `admin_applications` – выборка заявок с фильтрацией.<br>`approve_application` – автоматический или ручной подбор группы, создание родителя (если новый), создание ребёнка, зачисление в `enrollments`, обновление статуса заявки, уведомление родителя.<br>`reject_application` – обновление статуса на `rejected`, сохранение причины. | `SELECT * FROM applications WHERE ...`<br>`INSERT OR SELECT parents ...`<br>`INSERT INTO children ...`<br>`INSERT INTO enrollments ...`<br>`UPDATE applications SET status = 'approved'/'rejected', processed_at = ..., processed_by = ...`<br>`INSERT INTO notifications ...` |
| **Управление тренерами** | `GET /admin/trainers` → `admin_trainers.html`<br>`GET /admin/trainer/create` → форма<br>`POST /admin/trainer/create`<br>`GET /admin/trainer/{id}/edit` → форма<br>`POST /admin/trainer/{id}/edit`<br>`POST /admin/trainer/{id}/delete` | CRUD‑операции над таблицей `trainers`. Проверка уникальности логина. При удалении – сброс `trainer_id` в группах на `NULL`. | `INSERT/UPDATE/DELETE FROM trainers`<br>`UPDATE groups SET trainer_id = NULL WHERE trainer_id = ?` |
| **Управление группами** | `GET /admin/groups` → `admin_groups.html`<br>`GET /admin/group/create` → форма<br>`POST /admin/group/create`<br>`GET /admin/group/{id}/edit` → форма<br>`POST /admin/group/{id}/edit`<br>`POST /admin/group/{id}/delete` (с опциональным `transfer_group_id`) | CRUD групп. При удалении непустой группы требуется перевод учеников в другую группу или отчисление. | `INSERT/UPDATE/DELETE FROM groups`<br>`UPDATE enrollments SET group_id = ? WHERE group_id = ?` или `UPDATE enrollments SET is_active = 0 WHERE group_id = ?` |
| **Дашборд админа (статистика)** | `GET /dashboard` → `admin_dashboard.html`<br>AJAX‑запросы `GET /admin/api/stats` и `GET /admin/api/recent_applications` | `admin_api_stats` – возвращает счётчики (`new_applications`, `active_groups`, `total_students`, `active_trainers`).<br>`admin_api_recent_applications` – последние 10 заявок. | `SELECT COUNT(*) FROM applications WHERE status = 'new'`<br>`SELECT COUNT(*) FROM groups WHERE is_active = 1`<br>`SELECT COUNT(DISTINCT child_id) FROM enrollments WHERE is_active = 1`<br>`SELECT COUNT(*) FROM trainers WHERE is_active = 1`<br>`SELECT id, parent_full_name, child_full_name, ... FROM applications ORDER BY created_at DESC LIMIT 10` |

## Сессии и аутентификация

| Компонент | Описание | Взаимодействие с БД |
|-----------|----------|----------------------|
| **Управление сессиями** | Словарь `active_sessions` в памяти сервера хранит токен → `{user_id, user_type, login}`. Cookie `session_token`. | При каждом запросе middleware / зависимость `get_current_user` читает токен, затем выполняет `SELECT` из соответствующей таблицы (`admins`, `trainers`, `parents`) для получения имени и проверки существования пользователя. |
| **Выход** | `GET /logout` → удаление cookie. | (нет) |

## Общие вспомогательные функции

| Функция | Назначение | Работа с БД |
|---------|------------|-------------|
| `get_db_cursor` / `get_db` | Предоставление курсора SQLite с автоматическим commit/rollback. | Используется во всех эндпоинтах для выполнения запросов. |
| `send_notification_to_parent` | Создание записи в таблице `notifications` (системное уведомление). | `INSERT INTO notifications (user_id, user_type, type, title, message, status) VALUES (...)` |
| `auto_select_group` | Подбор подходящей группы по возрасту, году обучения и смене. | `SELECT ... FROM groups WHERE is_active = 1 AND ? BETWEEN min_age AND max_age AND swimming_year = ? AND shift = ? AND (число учеников) < max_students ORDER BY ... LIMIT 1` |
| `can_manage_group` | Проверка, что тренер (или админ) имеет доступ к группе. | `SELECT trainer_id FROM groups WHERE id = ?` |

> **Примечание:** Все эндпоинты, требующие авторизации, используют зависимость `get_current_user` и соответствующие проверки (`require_parent`, `require_trainer`, `require_admin`).  
> **База данных** – SQLite с файлом `swim_crm.db`. Схема включает 11 таблиц, описанных в `init_db.py`.  
> **Frontend** – шаблоны Jinja2 в папке `templates/`, статика в `static/`. Формы передают данные через `POST` (обычные HTML‑формы) или через AJAX (для API‑вызовов админ‑дашборда).

# Шифрование паролей и VK бот (Виктор)

## 📋 Содержание
1. [Введение](#введение)
2. [Шифрование паролей (bcrypt)](#шифрование-паролей-bcrypt)
3. [Интеграция с VK ботом](#интеграция-с-vk-ботом)
4. [Установка и запуск](#установка-и-запуск)
5. [Список библиотек](#список-библиотек)
6. [Основные функции](#основные-функции)
7. [Команды VK бота](#команды-vk-бота)

---

## Введение

В данном проекте выполнены два ключевых улучшения для CRM системы бассейна:

1. **Шифрование паролей** — все пароли пользователей хешируются с помощью bcrypt
2. **Интеграция с VK ботом** — родители могут получать информацию о детях через ВКонтакте

---

## Шифрование паролей (bcrypt)

### Проблема
Изначально пароли хранились в базе данных в открытом виде, что создавало серьёзную уязвимость.

### Решение
Внедрено хеширование паролей с использованием библиотеки `bcrypt` с солью и 12 раундами шифрования.

### Реализация

#### Функции хеширования (`init_db.py`)

```python
import bcrypt

def hash_password(password: str) -> str:
    """Хеширует пароль с помощью bcrypt"""
    if not password:
        password = "default123"
    salt = bcrypt.gensalt(rounds=12)
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    return hashed.decode('utf-8')

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Проверяет соответствие пароля хешу"""
    if not plain_password or not hashed_password:
        return False
    try:
        return bcrypt.checkpw(
            plain_password.encode('utf-8'),
            hashed_password.encode('utf-8')
        )
    except Exception:
        return False
```

#### Создание пользователей с хешированным паролем

```python
# Создание родителя (пароль = телефон)
hashed_password = hash_password(parent_phone)
cursor.execute(
    "INSERT INTO parents (full_name, phone, password_hash) VALUES (?, ?, ?)",
    (full_name, phone, hashed_password)
)

# Создание тренера
hashed_password = hash_password(password)
cursor.execute(
    "INSERT INTO trainers (full_name, login, password_hash) VALUES (?, ?, ?)",
    (full_name, login, hashed_password)
)
```

#### Проверка пароля при входе

```python
from init_db import verify_password

# В эндпоинте /login
if admin and verify_password(password, admin["password_hash"]):
    # Успешный вход
```

---

## Интеграция с VK ботом

### Функциональность

VK бот позволяет родителям:
- 🔗 Привязать VK аккаунт к личному кабинету
- 👶 Просматривать список своих детей и их группы
- 📊 Проверять статус заявки на зачисление
- 👤 Получать информацию о профиле

### Структура бота

#### 1. Конфигурация (`.env` файл)

```env
VK_GROUP_TOKEN=vk1.a.ваш_токен
VK_GROUP_ID=239120595
VK_BOT_ENABLED=True
```

#### 2. Отдельное соединение с БД (`bot_db.py`)

Поскольку бот работает в отдельном потоке, ему требуется собственное соединение с базой данных:

```python
class BotDatabase:
    def __init__(self):
        self.connection = sqlite3.connect(
            DATABASE_PATH,
            check_same_thread=False,  # Важно для многопоточности
            detect_types=sqlite3.PARSE_DECLTYPES | sqlite3.PARSE_COLNAMES
        )
        self.connection.row_factory = sqlite3.Row

    def get_cursor(self):
        return self.connection.cursor()
```

#### 3. Основной класс бота (`vk_bot.py`)

```python
class VKBot:
    def __init__(self, group_token, group_id):
        self.group_token = group_token
        self.group_id = group_id
        self.vk_session = None
        self.vk = None
        self.longpoll = None

    def init_api(self):
        """Инициализирует VK API и LongPoll"""
        self.vk_session = vk_api.VkApi(token=self.group_token)
        self.vk = self.vk_session.get_api()
        self.longpoll = VkBotLongPoll(self.vk_session, self.group_id)

    def send_message(self, user_id, message):
        """Отправляет сообщение пользователю"""
        self.vk.messages.send(user_id=user_id, message=message, random_id=0)

    def link_vk_account(self, vk_id, phone, password):
        """Привязывает VK аккаунт к родителю"""
        # Поиск родителя по телефону
        # Проверка пароля через verify_password()
        # Сохранение vk_id в БД

    def handle_message(self, user_id, message_text):
        """Обработчик команд"""
        if message_text == "/my_children":
            return self.format_children_list(children)
        elif message_text.startswith("/link"):
            return self.link_vk_account(...)
        # ... обработка других команд
```

#### 4. Запуск бота в фоновом потоке (`main.py`)

```python
from vk_bot import start_vk_bot, stop_vk_bot

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Запуск при старте
    start_vk_bot()
    yield
    # Остановка при завершении
    stop_vk_bot()
```

---

## Список библиотек

| Библиотека | Версия | Назначение |
|------------|--------|------------|
| **fastapi** | 0.104.1 | Веб-фреймворк для API |
| **uvicorn** | 0.24.0 | ASGI сервер для запуска приложения |
| **jinja2** | 3.1.2 | Шаблонизатор для HTML страниц |
| **python-multipart** | 0.0.6 | Обработка form-data запросов |
| **bcrypt** | 4.0.1 | Хеширование паролей |
| **vk-api** | 11.9.9 | Работа с API ВКонтакте |
| **python-dotenv** | 1.0.0 | Загрузка переменных из .env файла |

### Установка одной командой

```bash
pip install fastapi uvicorn jinja2 python-multipart bcrypt vk-api python-dotenv
```

---

## Основные функции

### Шифрование (init_db.py)

| Функция | Параметры | Возвращает | Описание |
|---------|-----------|------------|----------|
| `hash_password(password)` | `password: str` | `str` (хеш) | Хеширует пароль с помощью bcrypt |
| `verify_password(plain, hashed)` | `plain: str, hashed: str` | `bool` | Проверяет соответствие пароля хешу |

### VK бот (vk_bot.py)

| Функция | Параметры | Описание |
|---------|-----------|----------|
| `__init__(group_token, group_id)` | токен, ID группы | Инициализация бота |
| `init_api()` | - | Подключение к VK API |
| `send_message(user_id, message)` | ID пользователя, текст | Отправка сообщения |
| `link_vk_account(vk_id, phone, password)` | VK ID, телефон, пароль | Привязка аккаунта |
| `get_parent_by_vk(vk_id)` | VK ID | Поиск родителя по VK ID |
| `get_children_by_parent(parent_id)` | ID родителя | Получение списка детей |
| `handle_message(user_id, text)` | ID пользователя, текст | Обработчик команд |
| `run()` | - | Запуск LongPoll прослушивания |

### Запуск бота (vk_bot.py)

| Функция | Описание |
|---------|----------|
| `start_vk_bot()` | Запускает бота в фоновом потоке |
| `stop_vk_bot()` | Останавливает бота |

---

## Команды VK бота

| Команда | Формат | Описание |
|---------|--------|----------|
| `/help` | `/help` | Показать список команд |
| `/link` | `/link [телефон] [пароль]` | Привязать VK к аккаунту |
| `/my_children` | `/my_children` | Список детей и их групп |
| `/status` | `/status [номер_заявки]` | Статус заявки на зачисление |
| `/profile` | `/profile` | Информация о профиле |
| `/apply` | `/apply` | Инструкция по подаче заявки |

### Примеры использования

```
/link +79123456789 password123
✅ Аккаунт привязан! Добро пожаловать, Сергей Петров

/my_children
👶 Ваши дети:
   👤 Алексей Петров
      Возраст: 7 лет
      Группа: Рыбки
      ✅ Зачислен

/status 5
📋 Заявка #5
Ребёнок: Иван Иванов
Статус: 🟢 Одобрена
```

---

## Требования к группе ВК

Для работы бота необходимо:

1. **Создать группу** ВКонтакте (или использовать существующую)
2. **Включить сообщения сообщества:**
   - Управление → Сообщения → Включить сообщения сообщества
3. **Создать ключ доступа с правами:**
   - ✅ Управление сообществом
   - ✅ Доступ к сообщениям сообщества
4. **Добавить бота в администраторы** (опционально, но рекомендуется)

---

## Возможные ошибки и их решение

### 1. VK Bot: "Cannot operate on a closed cursor"

**Причина:** Бот использует курсор из основного приложения, который закрывается после HTTP запроса.

**Решение:** Создано отдельное соединение в `bot_db.py`.

### 2. VK Bot: "Access denied: no access to call this method"

**Причина:** Недостаточно прав у токена.

**Решение:** При создании токена выберите права:
- Управление сообществом
- Доступ к сообщениям сообщества
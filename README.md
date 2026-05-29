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
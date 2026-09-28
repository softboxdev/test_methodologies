# Практическая работа: Изучение JSON на Python 3.8 в Linux Rosa. Часть 4

**Тема:** Формат JSON — чтение, запись, изменение данных.

**Цель:** Научиться работать с JSON в Python: читать из файла, сохранять, изменять, проверять структуру. Всё — офлайн, встроенными средствами Python 3.8.

**Продолжительность:** 90 минут.

**Оборудование:** Компьютер с ОС РОСА Линукс, терминал, встроенный Python 3.8.

---

## Теоретическая справка

### Что такое JSON

**JSON** (JavaScript Object Notation) — текстовый формат для хранения и передачи данных. Появился в JavaScript, но используется везде: в API, конфигах, базах, логах.

**Главная идея:** данные представлены в виде **пар «ключ → значение»** и **списков**. Формат читается и человеком, и программой.

### Типы данных в JSON

| Тип JSON | Пример | В Python |
|---|---|---|
| Объект (`{}`) | `{"name": "Иван", "age": 25}` | `dict` |
| Массив (`[]`) | `[1, 2, 3]` | `list` |
| Строка | `"привет"` | `str` |
| Число | `42`, `3.14` | `int`, `float` |
| Логическое | `true`, `false` | `True`, `False` |
| Ничего | `null` | `None` |

> ⚠️ **Важно:** в JSON строки всегда в **двойных** кавычках. Одинарные (`'Иван'`) — недопустимы. Логические значения — строчными буквами (`true`, `false`), а в Python — с большой (`True`, `False`).

### Модуль `json` в Python

Встроенный, ничего ставить не нужно. Четыре главные функции:

| Функция | Что делает |
|---|---|
| `json.load(file)` | Читает JSON **из файла** → Python-объект |
| `json.loads(string)` | Читает JSON **из строки** → Python-объект |
| `json.dump(obj, file)` | Пишет Python-объект **в файл** в формате JSON |
| `json.dumps(obj)` | Превращает Python-объект **в строку** JSON |

**Мнемоника:** буква `s` в конце = **string** (строка). Без `s` = работа с **файлом**.

### Зачем это нужно в тестировании ИС

- API отвечает в JSON — нужно распарсить и проверить поля.
- Конфиги приложений часто в JSON — нужно проверить настройки.
- Логи структурированы в JSON — нужно найти ошибки.
- Результаты тестов сохраняются в JSON — нужно сформировать отчёт.

---

## Часть 1. Первое знакомство (10 минут)

### Шаг 1. Рабочая папка

```bash
mkdir -p ~/json_urok
cd ~/json_urok
```

### Шаг 2. Интерактивный режим

Запустите Python:

```bash
python3
```

Попробуйте по одной строке:

```python
import json

# Python-объект (словарь)
data = {"name": "Иван", "age": 25, "city": "Москва"}

# Превращаем в строку JSON
text = json.dumps(data)
print(text)
```

**Результат:**
```
{"name": "\u0418\u0432\u0430\u043d", "age": 25, "city": "\u041c\u043e\u0441\u043a\u0432\u0430"}
```

Русские буквы превратились в `\uXXXX` — это нормально для JSON, но неудобно читать. Чтобы сохранить русские буквы, добавьте `ensure_ascii=False`:

```python
text = json.dumps(data, ensure_ascii=False)
print(text)
```

**Результат:**
```
{"name": "Иван", "age": 25, "city": "Москва"}
```

Выход из Python: `exit()`.

**Задание 1.** В интерактивном режиме:
1. Создайте словарь с вашим именем, возрастом и городом.
2. Преобразуйте его в JSON-строку с `ensure_ascii=False`.
3. Выведите результат.

---

## Часть 2. Запись JSON в файл (15 минут)

### Шаг 1. Создайте файл `01_write.py`

```bash
nano 01_write.py
```

```python
#!/usr/bin/env python3
# Запись данных в JSON-файл

import json

# Python-словарь (это НЕ JSON, это объект Python)
user = {
    "id": 1,
    "name": "Иван",
    "email": "ivan@mail.ru",
    "age": 25,
    "active": True,
    "roles": ["admin", "user"]
}

# Открываем файл на запись ('w' = write)
# encoding='utf-8' — обязательно, чтобы русские буквы записались корректно
with open("user.json", "w", encoding="utf-8") as f:
    # json.dump пишет объект в файл
    # indent=2 — отступы для красоты (2 пробела)
    # ensure_ascii=False — не превращать русские буквы в \uXXXX
    json.dump(user, f, indent=2, ensure_ascii=False)

print("Файл user.json создан")
```

### Шаг 2. Запустите

```bash
python3 01_write.py
```

### Шаг 3. Посмотрите содержимое файла

```bash
cat user.json
```

**Ожидаемый результат:**
```json
{
  "id": 1,
  "name": "Иван",
  "email": "ivan@mail.ru",
  "age": 25,
  "active": true,
  "roles": [
    "admin",
    "user"
  ]
}
```

**Обратите внимание:**
- Python `True` записался как JSON `true`.
- Список `roles` превратился в JSON-массив.
- Отступы делают файл читаемым.

**Задание 2.** Создайте файл `book.json` с описанием книги: автор, название, год, жанры (список), прочитана (True/False).

---

## Часть 3. Чтение JSON из файла (15 минут)

### Шаг 1. Создайте файл `02_read.py`

```bash
nano 02_read.py
```

```python
#!/usr/bin/env python3
# Чтение JSON из файла

import json

# Открываем файл на чтение ('r' = read)
with open("user.json", "r", encoding="utf-8") as f:
    # json.load читает файл и возвращает Python-объект (словарь)
    user = json.load(f)

# Теперь user — это обычный словарь Python
print("Тип данных:", type(user))
print()

# Доступ по ключу
print("Имя:", user["name"])
print("Email:", user["email"])
print("Возраст:", user["age"])
print()

# Список внутри словаря
print("Роли:", user["roles"])
print("Первая роль:", user["roles"][0])
print()

# Проверка наличия ключа
if "phone" in user:
    print("Телефон:", user["phone"])
else:
    print("Телефон не указан")
```

### Шаг 2. Запустите

```bash
python3 02_read.py
```

**Ожидаемый результат:**
```
Тип данных: <class 'dict'>

Имя: Иван
Email: ivan@mail.ru
Возраст: 25

Роли: ['admin', 'user']
Первая роль: admin

Телефон не указан
```

**Задание 3.** Дополните `02_read.py`, чтобы он выводил:
- сколько всего ролей у пользователя;
- активен ли пользователь (`user["active"]`);
- сколько ключей в словаре (`len(user)`).

---

## Часть 4. Изменение и сохранение (15 минут)

### Шаг 1. Создайте файл `03_edit.py`

```bash
nano 03_edit.py
```

```python
#!/usr/bin/env python3
# Изменение JSON-данных

import json

# Читаем
with open("user.json", "r", encoding="utf-8") as f:
    user = json.load(f)

print("Было:", user)

# Изменяем существующие поля
user["age"] = 26
user["city"] = "Санкт-Петербург"    # добавляем новый ключ

# Добавляем роль
user["roles"].append("moderator")

# Удаляем ключ
user.pop("active")                  # pop удаляет ключ и возвращает значение

print("Стало:", user)

# Сохраняем обратно
with open("user.json", "w", encoding="utf-8") as f:
    json.dump(user, f, indent=2, ensure_ascii=False)

print("Файл обновлён")
```

### Шаг 2. Запустите и проверьте

```bash
python3 03_edit.py
cat user.json
```

**Ожидаемый результат:** в файле появились `city`, `age` стал `26`, `roles` пополнился, `active` исчез.

**Задание 4.** Измените программу так, чтобы:
- email стал `ivan.new@mail.ru`;
- добавилось поле `phone` со значением `+7-900-123-45-67`;
- удалилось поле `roles`.

---

## Часть 5. Форматирование и настройки `dump` (10 минут)

### Шаг 1. Создайте файл `04_format.py`

```bash
nano 04_format.py
```

```python
#!/usr/bin/env python3
# Разные варианты форматирования

import json

data = {
    "users": [
        {"id": 1, "name": "Иван"},
        {"id": 2, "name": "Мария"}
    ],
    "total": 2
}

print("=== 1. Компактно (по умолчанию) ===")
print(json.dumps(data, ensure_ascii=False))
print()

print("=== 2. С отступами indent=2 ===")
print(json.dumps(data, indent=2, ensure_ascii=False))
print()

print("=== 3. С отступами indent=4 ===")
print(json.dumps(data, indent=4, ensure_ascii=False))
print()

print("=== 4. Сортировка ключей sort_keys=True ===")
print(json.dumps(data, indent=2, sort_keys=True, ensure_ascii=False))
print()

print("=== 5. С русскими буквами как есть ===")
print(json.dumps(data, indent=2, ensure_ascii=True))     # плохо
print()
print(json.dumps(data, indent=2, ensure_ascii=False))    # хорошо
```

### Шаг 2. Запустите

```bash
python3 04_format.py
```

**Что изучите:**
- `indent=2` — читаемый вывод, используется в конфигах.
- Без `indent` — компактно, используется для API.
- `sort_keys=True` — ключи по алфавиту, удобно для сравнения файлов.
- `ensure_ascii=False` — русские буквы не превращаются в `\uXXXX`.

---

## Часть 6. Простейшие тесты JSON (15 минут)

### Тест 1. Проверка существования файла

Создайте `test_01_exists.py`:

```python
#!/usr/bin/env python3
# Проверка: файл существует

import os

print("=== Тест: файл user.json существует ===")

if os.path.exists("user.json"):
    print("PASS")
else:
    print("FAIL")
```

**Запуск:**
```bash
python3 test_01_exists.py
```

---

### Тест 2. Проверка, что файл — валидный JSON

Создайте `test_02_valid.py`:

```python
#!/usr/bin/env python3
# Проверка: файл корректно парсится

import json

print("=== Тест: user.json — валидный JSON ===")

try:
    with open("user.json", "r", encoding="utf-8") as f:
        data = json.load(f)
    print("PASS")
except json.JSONDecodeError as e:
    print("FAIL: ошибка JSON —", e)
except FileNotFoundError:
    print("FAIL: файл не найден")
```

**Что проверяет:** `JSONDecodeError` возникает, если в файле опечатка (лишняя запятая, незакрытая кавычка и т.д.).

---

### Тест 3. Проверка обязательных полей

Создайте `test_03_fields.py`:

```python
#!/usr/bin/env python3
# Проверка: в JSON есть нужные поля

import json

print("=== Тест: обязательные поля ===")

with open("user.json", "r", encoding="utf-8") as f:
    user = json.load(f)

required = ["id", "name", "email"]
for field in required:
    if field in user:
        print(f"PASS: поле '{field}' есть")
    else:
        print(f"FAIL: поле '{field}' отсутствует")
```

**Что проверяет:** наличие обязательных ключей в объекте.

---

### Тест 4. Проверка типа данных

Создайте `test_04_types.py`:

```python
#!/usr/bin/env python3
# Проверка типов полей

import json

print("=== Тест: типы данных ===")

with open("user.json", "r", encoding="utf-8") as f:
    user = json.load(f)

# Каждое поле должно быть определённого типа
checks = [
    ("id", int),
    ("name", str),
    ("email", str),
    ("roles", list),
]

for field, expected_type in checks:
    if field not in user:
        print(f"FAIL: поля '{field}' нет")
        continue
    if isinstance(user[field], expected_type):
        print(f"PASS: '{field}' — {expected_type.__name__}")
    else:
        print(f"FAIL: '{field}' имеет тип {type(user[field]).__name__}, ожидался {expected_type.__name__}")
```

**Что проверяет:** `isinstance(obj, type)` — правильный тип данных.

---

### Тест 5. Проверка значений

Создайте `test_05_values.py`:

```python
#!/usr/bin/env python3
# Проверка конкретных значений

import json

print("=== Тест: значения полей ===")

with open("user.json", "r", encoding="utf-8") as f:
    user = json.load(f)

if user.get("id") == 1:
    print("PASS: id = 1")
else:
    print(f"FAIL: id = {user.get('id')}, ожидалось 1")

if user.get("email", "").endswith("@mail.ru"):
    print("PASS: email на @mail.ru")
else:
    print("FAIL: email не на @mail.ru")

if "admin" in user.get("roles", []):
    print("PASS: у пользователя есть роль admin")
else:
    print("FAIL: роли admin нет")
```

**Что проверяет:** `dict.get(key, default)` — безопасный доступ; проверку конкретных бизнес-правил.

---

### Запуск всех тестов

Создайте `run_tests.sh`:

```bash
#!/bin/bash
# Запуск всех тестов JSON по очереди

for t in test_*.py; do
    echo "--- $t ---"
    python3 "$t"
    echo
done
```

```bash
chmod +x run_tests.sh
./run_tests.sh
```

**Ожидаемый результат:** все тесты печатают PASS.

**Задание 5.** Запустите все тесты. Затем откройте `user.json` в редакторе и **специально сломайте** его: уберите кавычку у `name`. Запустите `test_02_valid.py` — должен быть FAIL. Верните кавычку обратно.

---

## Часть 7. Практика: мини-проект «База пользователей» (10 минут)

Создайте файл `users.json` вручную:

```json
[
  {"id": 1, "name": "Иван", "age": 25, "active": true},
  {"id": 2, "name": "Мария", "age": 30, "active": true},
  {"id": 3, "name": "Пётр", "age": 17, "active": false}
]
```

Создайте `users_report.py`:

```python
#!/usr/bin/env python3
# Отчёт по базе пользователей

import json

with open("users.json", "r", encoding="utf-8") as f:
    users = json.load(f)

print(f"Всего пользователей: {len(users)}")
print()

# Активные пользователи
active = [u for u in users if u["active"]]
print(f"Активных: {len(active)}")
for u in active:
    print(f"  - {u['name']} ({u['age']} лет)")
print()

# Средний возраст
avg = sum(u["age"] for u in users) / len(users)
print(f"Средний возраст: {avg:.1f}")
print()

# Совершеннолетние
adults = [u["name"] for u in users if u["age"] >= 18]
print("Совершеннолетние:", ", ".join(adults))
```

**Запуск:**
```bash
python3 users_report.py
```

**Ожидаемый вывод:**
```
Всего пользователей: 3

Активных: 2
  - Иван (25 лет)
  - Мария (30 лет)

Средний возраст: 24.0

Совершеннолетние: Иван, Мария
```

**Задание 6.** Допишите программу, чтобы она:
1. Находила самого старшего пользователя.
2. Считала, сколько пользователей младше 18.
3. Сохраняла отчёт в `users_report.json` в формате:
   ```json
   {
     "total": 3,
     "active": 2,
     "average_age": 24.0,
     "adults": ["Иван", "Мария"]
   }
   ```

---

## Часть 8. Самостоятельная работа (10 минут)

**Задание 7.** Создайте `products.json` со списком товаров:

```json
[
  {"id": 1, "name": "Хлеб", "price": 40.5, "quantity": 10},
  {"id": 2, "name": "Молоко", "price": 80.0, "quantity": 5},
  {"id": 3, "name": "Сыр", "price": 350.0, "quantity": 2}
]
```

Напишите программу, которая:
- считает общую стоимость склада (price × quantity);
- находит самый дорогой товар;
- сохраняет отчёт в `warehouse_report.json`.

**Задание 8.** Напишите тест `test_products.py`, который проверяет:
- все товары имеют поля `id`, `name`, `price`, `quantity`;
- все `price` > 0;
- все `quantity` ≥ 0;
- `id` уникальны.

**Подсказка для проверки уникальности:**
```python
ids = [p["id"] for p in products]
if len(ids) == len(set(ids)):
    print("PASS: id уникальны")
else:
    print("FAIL: есть дубликаты id")
```

---

## Сводная таблица: что изучили

| Действие | Функция | Пример |
|---|---|---|
| Python → строка JSON | `json.dumps(obj)` | `json.dumps(data)` |
| Строка JSON → Python | `json.loads(text)` | `json.loads('{"a":1}')` |
| Python → файл | `json.dump(obj, f)` | `json.dump(data, f, indent=2)` |
| Файл → Python | `json.load(f)` | `data = json.load(f)` |
| Русские буквы | `ensure_ascii=False` | `json.dump(..., ensure_ascii=False)` |
| Красивый вывод | `indent=2` | `json.dump(..., indent=2)` |
| Сортировка ключей | `sort_keys=True` | `json.dump(..., sort_keys=True)` |
| Безопасный доступ | `dict.get()` | `user.get("phone", "не указан")` |
| Проверка типа | `isinstance()` | `isinstance(x, list)` |
| Обработка ошибок | `try/except` | `except json.JSONDecodeError` |

---

## Контрольные вопросы

1. Чем JSON отличается от Python-словаря? Что общего?
2. Почему `True` в Python превращается в `true` в JSON?
3. Что делает `ensure_ascii=False` и зачем это нужно?
4. Чем `json.dump` отличается от `json.dumps`? А `load` от `loads`?
5. Что произойдёт, если в JSON-файле поставить лишнюю запятую?
6. Зачем нужен `indent=2`? Влияет ли он на данные?
7. Какая ошибка возникнет, если попытаться прочитать несуществующий JSON-файл? Как её обработать?
8. Можно ли в JSON использовать комментарии (`//` или `#`)? Почему?

---

## Отчёт должен содержать

- Листинги всех созданных файлов: `01_write.py`, `02_read.py`, `03_edit.py`, `04_format.py`, `users_report.py`, `test_*.py`, `run_tests.sh`.
- Содержимое созданных JSON-файлов: `user.json`, `users.json`, `products.json`.
- Скриншоты запуска всех тестов (PASS) и одного сбоя (FAIL).
- Ответы на контрольные вопросы.
- Результаты самостоятельной работы.

---

## Домашнее задание

1. **Создайте JSON-файл** `config.json` с настройками приложения:
   ```json
   {
     "server": {"host": "127.0.0.1", "port": 8080, "timeout": 30},
     "database": {"host": "localhost", "port": 5432, "name": "shop"},
     "logging": {"level": "INFO", "file": "/var/log/app.log"}
   }
   ```

2. **Напишите тесты** `test_config.py`, которые проверяют:
   - все три секции (`server`, `database`, `logging`) есть;
   - `server.port` — целое число от 1 до 65535;
   - `logging.level` — одно из `DEBUG`, `INFO`, `WARNING`, `ERROR`;
   - `database.name` не пустое.

3. **Напишите утилиту** `json_pretty.py`, которая принимает имя файла в аргументе и переформатирует его с `indent=2`:
   ```bash
   python3 json_pretty.py user.json
   ```
   Подсказка: используйте `sys.argv[1]`.

4. **Сравните два JSON-файла** — напишите скрипт `json_diff.py`, который выводит, какие ключи есть в первом, но нет во втором, и наоборот.

---

## Частые проблемы

| Проблема | Причина | Решение |
|---|---|---|
| `JSONDecodeError: Expecting ',' delimiter` | Лишняя или пропущенная запятая | Проверьте файл: `python3 -m json.tool file.json` |
| `JSONDecodeError: Expecting property name` | Ключ без кавычек | Ключи в JSON всегда в двойных кавычках |
| `UnicodeDecodeError` | Файл не в UTF-8 | Откройте с `encoding="utf-8"` |
| Русские буквы как `\u0418` | Не указан `ensure_ascii=False` | Добавьте параметр |
| `KeyError` | Обращение к несуществующему ключу | Используйте `.get("key", default)` |
| Файл не создаётся | Папка не существует или нет прав | Проверьте `ls` и права |

---

## Полезный приём: проверка JSON из командной строки

Python умеет проверять JSON-файлы без написания скриптов:

```bash
# Проверить, что файл — валидный JSON
python3 -m json.tool user.json

# С подсветкой и отступами
python3 -m json.tool user.json --indent 2
```

Если файл сломан — Python покажет строку и позицию ошибки. Это самый быстрый способ проверить конфиг или ответ API.

**Задание 9.** Испортите `user.json` (уберите закрывающую `}`) и запустите `python3 -m json.tool user.json`. Запишите в тетрадь, что показал Python.

---

**Совет:** начните с Части 2 и 3 — записи и чтения. Это самые полезные навыки. Тесты (Часть 6) — это уже реальная работа тестировщика: проверка структуры, типов, значений. Формат JSON используется **везде** в современной ИТ, поэтому практика окупится многократно.

# Практическая работа: Тестирование информационной системы на Linux Rosa. Проверка JSON-файлов в Linux Rosa (Python 3.8). Часть 3

**Тема:** Тестирование корректности JSON-файлов — структура, обязательные поля, типы значений.

**Цель:** Научиться проверять JSON-файлы информационной системы с помощью встроенного Python 3.8 и коротких bash-скриптов. Всё работает офлайн.

**Продолжительность:** 90 минут.

**Оборудование:** Компьютер с ОС РОСА Линукс, терминал, Python 3.8 (встроен).

---

## Теоретическая справка

### Что такое JSON

**JSON** (JavaScript Object Notation) — текстовый формат обмена данными. Используется везде: API, конфиги, ответы серверов, обмен между системами.

Пример:

```json
{
  "id": 1,
  "name": "Иван",
  "email": "ivan@mail.ru",
  "active": true,
  "roles": ["admin", "user"],
  "profile": {
    "age": 25,
    "city": "Москва"
  }
}
```

**Типы данных в JSON:**

| Тип | Пример | Аналог в Python |
|---|---|---|
| Объект | `{"key": "value"}` | `dict` |
| Массив | `[1, 2, 3]` | `list` |
| Строка | `"текст"` | `str` |
| Число | `42`, `3.14` | `int`, `float` |
| Логическое | `true`, `false` | `bool` |
| Пусто | `null` | `None` |

**Правила JSON:**
- Ключи — **всегда** в двойных кавычках.
- Строки — **только** в двойных кавычках (`'text'` — ошибка).
- Нет запятых после последнего элемента.
- Нет комментариев.
- Кодировка — UTF-8.

### Почему JSON нужно проверять

Ошибки в JSON ломают системы:

- Пропущена запятая → приложение не запускается.
- Не то поле → данные не сохраняются.
- Не тот тип (строка вместо числа) → ошибка при обработке.
- Дубликат ключа → второй ключ «затирает» первый.

### Инструменты проверки

**1. Python 3.8** — встроенный модуль `json`:

```python
import json

with open("file.json") as f:
    data = json.load(f)      # загрузить и проверить синтаксис
```

Если JSON сломан — `json.load` выбросит `json.JSONDecodeError`.

**2. Утилита `jq`** — внешний инструмент для работы с JSON из командной строки. Ставится отдельно:

```bash
sudo dnf install jq
```

Мы будем использовать **Python** (он гарантированно есть) и опционально **jq**.

### Bash + Python

Быстрая проверка из bash без написания файла:

```bash
python3 -c "import json; json.load(open('file.json'))"
echo $?          # 0 — валиден, 1 — ошибка
```

Это основа всех тестов.

---

## Часть 1. Подготовка среды (10 минут)

### Шаг 1. Создать рабочую папку

```bash
mkdir -p ~/json_test/data
cd ~/json_test
```

### Шаг 2. Проверить Python

```bash
python3 --version
```

Ожидаемый вывод: `Python 3.8.x`.

### Шаг 3. Создать тестовые JSON-файлы

**Файл 1 — корректный.** Создайте `data/users.json`:

```bash
cat > ~/json_test/data/users.json <<'EOF'
{
  "users": [
    {"id": 1, "name": "Иван", "email": "ivan@mail.ru", "active": true},
    {"id": 2, "name": "Мария", "email": "maria@mail.ru", "active": false},
    {"id": 3, "name": "Пётр", "email": "petr@mail.ru", "active": true}
  ]
}
EOF
```

**Файл 2 — с ошибкой.** Создайте `data/broken.json`:

```bash
cat > ~/json_test/data/broken.json <<'EOF'
{
  "name": "Тест",
  "value": 42,
}
EOF
```

Здесь лишняя запятая после `42` — это **невалидный** JSON.

**Файл 3 — конфиг.** Создайте `data/config.json`:

```bash
cat > ~/json_test/data/config.json <<'EOF'
{
  "server": {
    "host": "127.0.0.1",
    "port": 8080,
    "timeout": 30
  },
  "database": {
    "name": "shop",
    "user": "admin"
  }
}
EOF
```

### Шаг 4. Проверить вручную

```bash
python3 -c "import json; json.load(open('/home/user/json_test/data/users.json'))"
echo "users.json код: $?"

python3 -c "import json; json.load(open('/home/user/json_test/data/broken.json'))"
echo "broken.json код: $?"
```

**Ожидаемый результат:**
- `users.json код: 0` — валиден.
- `broken.json` — ошибка `JSONDecodeError`, код ≠ 0.

---

## Часть 2. Ручные проверки через Python (15 минут)

Прежде чем писать тесты, попробуйте инструменты руками.

### 2.1. Проверка синтаксиса

```bash
python3 -c "import json; json.load(open('/home/user/json_test/data/users.json'))" && echo "OK" || echo "ERROR"
```

### 2.2. Красивый вывод

```bash
python3 -m json.tool ~/json_test/data/users.json
```

`python3 -m json.tool` форматирует JSON с отступами — удобно читать.

### 2.3. Получить конкретное значение

```bash
python3 -c "import json; data=json.load(open('/home/user/json_test/data/config.json')); print(data['server']['port'])"
```

**Ожидаемый вывод:** `8080`.

### 2.4. Проверить наличие ключа

```bash
python3 -c "import json; data=json.load(open('/home/user/json_test/data/config.json')); print('port' in data['server'])"
```

**Ожидаемый вывод:** `True`.

### 2.5. Посмотреть тип значения

```bash
python3 -c "import json; data=json.load(open('/home/user/json_test/data/config.json')); print(type(data['server']['port']).__name__)"
```

**Ожидаемый вывод:** `int`.

**Задание 1.** Выполните все команды из раздела 2. Запишите в тетрадь:
- что вернул `python3 -m json.tool` для `broken.json`;
- какие типы у полей `host`, `port`, `timeout` в `config.json`.

---

## Часть 3. Функция проверки и первые тесты (20 минут)

### Шаг 1. Создать `check.sh`

```bash
nano ~/json_test/check.sh
```

```bash
#!/bin/bash
# Универсальная функция проверки. $1 — имя теста, дальше — команда.

PASS=0
FAIL=0

check() {
    local name="$1"           # запоминаем название теста
    shift                     # убираем название, остаётся команда
    if "$@" >/dev/null 2>&1; then
        echo "PASS: $name"
        PASS=$((PASS + 1))
    else
        echo "FAIL: $name"
        FAIL=$((FAIL + 1))
    fi
}
```

**Как это работает:**
- `local name="$1"` — имя теста только внутри функции.
- `shift` — сдвигаем: теперь `"$@"` — команда для выполнения.
- `"$@" >/dev/null 2>&1` — выполняем команду, глушим и stdout, и stderr.
- `if ...; then` — проверяем код возврата: 0 = PASS, иначе FAIL.

### Шаг 2. Тест 1: файл существует

Создайте `test_json_exists.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "Файл users.json существует и не пуст" \
    test -s "$DATA/users.json"

check "Файл config.json существует и не пуст" \
    test -s "$DATA/config.json"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Запуск:**
```bash
chmod +x test_json_exists.sh
./test_json_exists.sh
```

### Шаг 3. Тест 2: синтаксис валиден

Создайте `test_json_valid.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

# json.load() читает файл и выбросит ошибку, если JSON сломан
check "users.json — валидный JSON" \
    python3 -c "import json; json.load(open('$DATA/users.json'))"

check "config.json — валидный JSON" \
    python3 -c "import json; json.load(open('$DATA/config.json'))"

# broken.json должен ПРОВАЛИТЬСЯ — используем !
check "broken.json — НЕвалидный JSON (ожидаемо)" \
    bash -c "! python3 -c \"import json; json.load(open('$DATA/broken.json'))\""

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Разбор:**
- Если `json.load` не выбросил исключение — код 0 → PASS.
- `!` перед командой инвертирует код: `broken.json` **должен** провалиться, и это ожидаемо.

**Запуск:**
```bash
chmod +x test_json_valid.sh
./test_json_valid.sh
```

**Ожидаемый вывод:**
```
PASS: users.json — валидный JSON
PASS: config.json — валидный JSON
PASS: broken.json — НЕвалидный JSON (ожидаемо)
Пройдено: 3, Провалено: 0
```

### Шаг 4. Тест 3: обязательные поля

Создайте `test_json_fields.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

# Проверяем, что в config.json есть обязательные секции
check "config.json содержит 'server'" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert 'server' in d"

check "config.json содержит 'database'" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert 'database' in d"

check "server содержит 'host'" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert 'host' in d['server']"

check "server содержит 'port'" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert 'port' in d['server']"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Разбор:**
- `assert условие` — если условие ложно, Python завершится с кодом 1 → FAIL.
- Если условие истинно — программа завершится с кодом 0 → PASS.

**Запуск:**
```bash
chmod +x test_json_fields.sh
./test_json_fields.sh
```

**Задание 2.** Запустите все три теста. Убедитесь, что все PASS.

---

## Часть 4. Расширенные тесты (20 минут)

### Тест 4: типы значений

Создайте `test_json_types.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "server.host — строка" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert isinstance(d['server']['host'], str)"

check "server.port — число" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert isinstance(d['server']['port'], int)"

check "server.timeout — число" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert isinstance(d['server']['timeout'], int)"

check "database.name — строка" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert isinstance(d['database']['name'], str)"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Что проверяет:** что поля имеют правильный тип. Например, `port` — число, а не строка `"8080"`.

---

### Тест 5: массив users и его элементы

Создайте `test_json_users.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "users — это массив" \
    python3 -c "import json; d=json.load(open('$DATA/users.json')); assert isinstance(d['users'], list)"

check "В массиве ровно 3 пользователя" \
    python3 -c "import json; d=json.load(open('$DATA/users.json')); assert len(d['users']) == 3"

check "У каждого пользователя есть id, name, email" \
    python3 -c "import json; d=json.load(open('$DATA/users.json')); [__import__('sys').exit(1) for u in d['users'] if not all(k in u for k in ('id','name','email'))]"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Разбор третьего теста:**
- Проходим по всем пользователям.
- Если хотя бы у одного нет одного из полей — выходим с кодом 1.
- Если все в порядке — код 0.

Проще: можно использовать такую конструкцию:

```bash
python3 -c "
import json, sys
d = json.load(open('$DATA/users.json'))
for u in d['users']:
    if not all(k in u for k in ('id', 'name', 'email')):
        sys.exit(1)
"
```

Оба варианта работают одинаково.

---

### Тест 6: уникальность ID

Создайте `test_json_unique_ids.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

# Собираем все id в список, проверяем что длина списка = длина множества
check "Все id пользователей уникальны" \
    python3 -c "
import json, sys
d = json.load(open('$DATA/users.json'))
ids = [u['id'] for u in d['users']]
sys.exit(0 if len(ids) == len(set(ids)) else 1)
"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Разбор:**
- `set(ids)` — множество, где каждый элемент встречается один раз.
- Если длины равны — дубликатов нет.

---

### Тест 7: формат email

Создайте `test_json_email_format.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "Все email содержат @" \
    python3 -c "
import json, sys
d = json.load(open('$DATA/users.json'))
for u in d['users']:
    if '@' not in u['email']:
        sys.exit(1)
"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

---

### Тест 8: значения в допустимом диапазоне

Создайте `test_json_ranges.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "server.port в диапазоне 1–65535" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert 1 <= d['server']['port'] <= 65535"

check "server.timeout больше 0" \
    python3 -c "import json; d=json.load(open('$DATA/config.json')); assert d['server']['timeout'] > 0"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Что проверяет:** бизнес-правила. Например, порт не может быть отрицательным.

---

### Тест 9: jq (если установлен)

Проверьте наличие `jq`:

```bash
which jq || sudo dnf install jq
```

Создайте `test_json_jq.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

check "jq: users.json валиден" \
    jq empty "$DATA/users.json"

check "jq: в users 3 элемента" \
    bash -c "[ \$(jq '.users | length' $DATA/users.json) -eq 3 ]"

check "jq: server.port = 8080" \
    bash -c "[ \$(jq '.server.port' $DATA/config.json) -eq 8080 ]"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Разбор:**
- `jq empty` — проверяет синтаксис, ничего не выводит.
- `jq '.users | length'` — длина массива.
- `jq '.server.port'` — значение поля.

---

### Тест 10: сравнение структуры с эталоном

Создайте `test_json_schema.sh`:

```bash
#!/bin/bash
source ~/json_test/check.sh
DATA=~/json_test/data

# Проверяем, что в config.json нет ЛИШНИХ полей в server
check "server содержит ТОЛЬКО host, port, timeout" \
    python3 -c "
import json, sys
d = json.load(open('$DATA/config.json'))
expected = {'host', 'port', 'timeout'}
actual = set(d['server'].keys())
sys.exit(0 if actual == expected else 1)
"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Что проверяет:** что структура точно соответствует ожиданиям. Полезно ловить «лишние» поля, добавленные по ошибке.

---

## Часть 5. Запуск всех тестов (5 минут)

Создайте `run_all.sh`:

```bash
#!/bin/bash
# Запускает все тесты JSON по очереди

echo "===== Тестирование JSON-файлов ====="
echo

for test in ~/json_test/test_*.sh; do
    echo "--- $(basename $test) ---"
    bash "$test"
    echo
done

echo "===== Тестирование завершено ====="
```

**Запуск:**

```bash
chmod +x ~/json_test/run_all.sh
~/json_test/run_all.sh
```

**Ожидаемый вывод:** все тесты PASS, счётчики в конце каждого блока.

---

## Часть 6. Проверка обнаружения сбоя (15 минут)

Тесты бесполезны, если всегда возвращают PASS. Проверим, что они ловят проблемы.

### Сценарий 1. Сломать синтаксис

```bash
echo ',' >> ~/json_test/data/users.json     # добавляем лишнюю запятую
~/json_test/run_all.sh
```

**Ожидаемый результат:** тесты `test_json_valid.sh`, `test_json_users.sh` **провалятся**.

Верните файл:

```bash
# Пересоздать
cat > ~/json_test/data/users.json <<'EOF'
{
  "users": [
    {"id": 1, "name": "Иван", "email": "ivan@mail.ru", "active": true},
    {"id": 2, "name": "Мария", "email": "maria@mail.ru", "active": false},
    {"id": 3, "name": "Пётр", "email": "petr@mail.ru", "active": true}
  ]
}
EOF
```

### Сценарий 2. Удалить обязательное поле

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/config.json'))
del d['server']['port']
json.dump(d, open('/home/user/json_test/data/config.json', 'w'), indent=2)
EOF
~/json_test/run_all.sh
```

**Ожидаемый результат:** `test_json_fields.sh` и `test_json_types.sh` **провалятся**.

Верните:

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/config.json'))
d['server']['port'] = 8080
json.dump(d, open('/home/user/json_test/data/config.json', 'w'), indent=2)
EOF
```

### Сценарий 3. Изменить тип

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/config.json'))
d['server']['port'] = "8080"       # строка вместо числа
json.dump(d, open('/home/user/json_test/data/config.json', 'w'), indent=2)
EOF
~/json_test/run_all.sh
```

**Ожидаемый результат:** тест `server.port — число` **провалится**.

Верните:

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/config.json'))
d['server']['port'] = 8080
json.dump(d, open('/home/user/json_test/data/config.json', 'w'), indent=2)
EOF
```

### Сценарий 4. Дубликат ID

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/users.json'))
d['users'][2]['id'] = 1              # делаем ID 3 → 1, теперь два id=1
json.dump(d, open('/home/user/json_test/data/users.json', 'w'), indent=2, ensure_ascii=False)
EOF
~/json_test/run_all.sh
```

**Ожидаемый результат:** `test_json_unique_ids.sh` **провалится**.

Верните:

```bash
python3 - <<'EOF'
import json
d = json.load(open('/home/user/json_test/data/users.json'))
d['users'][2]['id'] = 3
json.dump(d, open('/home/user/json_test/data/users.json', 'w'), indent=2, ensure_ascii=False)
EOF
```

### Финальная проверка

```bash
~/json_test/run_all.sh
```

Все тесты должны снова пройти.

---

## Часть 7. Самостоятельная работа (10 минут)

**Задание 3.** Создайте файл `data/orders.json`:

```json
{
  "orders": [
    {"id": 101, "user_id": 1, "amount": 1500.0, "status": "paid"},
    {"id": 102, "user_id": 2, "amount": 2300.5, "status": "pending"},
    {"id": 103, "user_id": 1, "amount": 800.0, "status": "paid"}
  ]
}
```

Напишите тест `test_json_orders.sh`, который проверяет:
1. Файл валиден.
2. Есть массив `orders` из 3 элементов.
3. У каждого заказа есть поля `id`, `user_id`, `amount`, `status`.
4. Все `amount` — положительные числа.
5. Все `status` из списка `["paid", "pending", "cancelled"]`.

**Задание 4.** Напишите тест `test_json_no_extra_fields.sh`, который проверяет, что в каждом объекте пользователя **нет лишних полей** кроме `id`, `name`, `email`, `active`.

**Подсказка:**

```python
expected = {'id', 'name', 'email', 'active'}
actual = set(user.keys())
if actual - expected:          # если есть «лишние»
    sys.exit(1)
```

**Задание 5.** Напишите тест `test_json_encoding.sh`, который проверяет, что файлы сохранены в UTF-8.

**Подсказка:**

```bash
file ~/json_test/data/users.json | grep -q "UTF-8"
```

---

## Часть 8. Сводная таблица: что изучили

| Действие | Команда | Пример |
|---|---|---|
| Проверить синтаксис | `json.load()` | `python3 -c "import json; json.load(open('f.json'))"` |
| Красивый вывод | `python3 -m json.tool` | `python3 -m json.tool f.json` |
| Получить поле | `d['key']` | `d['server']['port']` |
| Проверить наличие | `'key' in d` | `'port' in d['server']` |
| Проверить тип | `isinstance()` | `isinstance(d['port'], int)` |
| Длина массива | `len()` | `len(d['users'])` |
| Уникальность | `set()` | `len(ids) == len(set(ids))` |
| Условие в Python | `assert` | `assert x > 0` |
| Выход с кодом | `sys.exit()` | `sys.exit(1)` |
| jq: синтаксис | `jq empty f.json` | `jq empty users.json` |
| jq: значение | `jq '.key'` | `jq '.server.port'` |
| jq: длина | `jq '.arr \| length'` | `jq '.users \| length'` |

---

## Контрольные вопросы

1. Что такое JSON? Чем он отличается от обычного текстового файла?
2. Почему в JSON ключи должны быть в двойных кавычках, а не в одинарных?
3. Что произойдёт, если в файле есть лишняя запятая? Как это обнаружит Python?
4. Чем `json.load()` отличается от `json.loads()`?
5. Зачем проверять **тип** значения, если JSON уже валиден?
6. Почему для проверки уникальности ID удобно использовать `set`?
7. В чём разница между `assert` и `sys.exit(1)` в тестах?
8. Зачем нужен `jq`, если есть Python?

---

## Отчёт должен содержать

- Листинги всех скриптов: `check.sh`, `test_json_exists.sh`, `test_json_valid.sh`, `test_json_fields.sh`, `test_json_types.sh`, `test_json_users.sh`, `test_json_unique_ids.sh`, `test_json_email_format.sh`, `test_json_ranges.sh`, `test_json_jq.sh`, `test_json_schema.sh`, `run_all.sh`.
- Скриншоты:
  - запуск `run_all.sh` при корректных файлах (все PASS);
  - один из сценариев сбоя (например, лишняя запятая) с FAIL;
  - вывод `python3 -m json.tool` для одного из файлов.
- Ответы на контрольные вопросы.
- Результаты самостоятельной работы (задания 3–5).

---

## Домашнее задание

1. **Создайте файл `data/products.json`** с массивом из 5 товаров:

   ```json
   {
     "products": [
       {"id": 1, "name": "Товар 1", "price": 100.0, "in_stock": true},
       ...
     ]
   }
   ```

   Напишите тест, который проверяет: все `price` > 0, все `id` уникальны, все `in_stock` — логического типа.

2. **Сравните** два подхода к проверке JSON:

   ```bash
   time python3 -c "import json; json.load(open('users.json'))"
   time jq empty users.json
   ```

   Какой быстрее? В каких случаях удобнее `jq`?

3. **Настройте запуск** `run_all.sh` через `cron` каждые 15 минут:

   ```bash
   crontab -e
   # строка:
   */15 * * * * /home/user/json_test/run_all.sh >> /home/user/json_test/json.log 2>&1
   ```

4. **Дополнительно:** если у вас есть какой-нибудь реальный JSON-конфиг (например, `package.json` от Node.js или `settings.json` от VS Code) — напишите для него 3–5 своих тестов.

---

## Частые проблемы

| Проблема | Причина | Решение |
|---|---|---|
| `JSONDecodeError` | Синтаксис сломан | Проверьте запятые, кавычки, скобки |
| `KeyError` | Нет нужного ключа | Проверьте `'key' in d` перед доступом |
| `TypeError` | Неверный тип | `isinstance(x, int)` |
| Кириллица отображается как `\u...` | Python по умолчанию экранирует | `json.dump(..., ensure_ascii=False)` |
| `ModuleNotFoundError: No module named 'json'` | Неправильный Python | Используйте `python3`, не `python` |
| `jq: command not found` | Пакет не установлен | `sudo dnf install jq` |
| Разные результаты `getent` и Python | Кеш / nsswitch | Перезапустите приложение |

---

**Совет:** JSON-тесты — это самая частая задача при работе с API. Начните с `test_json_valid.sh` (синтаксис) и `test_json_fields.sh` (обязательные поля) — эти два теста ловят 80% реальных проблем. Остальные добавляйте по мере усложнения данных.

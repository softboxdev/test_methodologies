# Практическая работа: Тестирование информационной системы на Linux Rosa. Часть 2

Ниже — продолжение набора. Задания той же сложности.
---

## 🧪 Практика 7. Проверка логов на ошибки

**Тема:** Поиск проблем в логах информационной системы.

**Инструменты:** `grep`, `awk`, `sort`, `uniq` (есть в системе).

**Идея:** В логах ИС всегда есть записи. Нужно автоматически находить ошибки и считать их по типам.

### Подготовка

Создайте «лог» тестовой системы:

```bash
mkdir -p ~/logs_test
cd ~/logs_test

cat > app.log <<'EOF'
2024-09-21 10:00:01 INFO  Сервер запущен
2024-09-21 10:00:05 INFO  Подключение к БД
2024-09-21 10:01:12 ERROR Не найден файл config.ini
2024-09-21 10:02:30 WARN  Медленный запрос: 2.5s
2024-09-21 10:03:45 ERROR Таймаут БД
2024-09-21 10:04:00 INFO  Запрос обработан
2024-09-21 10:05:20 WARN  Высокая нагрузка
2024-09-21 10:06:10 ERROR Неверный пароль пользователя
EOF
```

### Тесты

Создайте `check.sh` (если ещё нет):

```bash
#!/bin/bash
PASS=0; FAIL=0
check() {
    local name="$1"; shift
    if "$@" >/dev/null 2>&1; then
        echo "PASS: $name"; PASS=$((PASS + 1))
    else
        echo "FAIL: $name"; FAIL=$((FAIL + 1))
    fi
}
```

Создайте `test_logs.sh`:

```bash
#!/bin/bash
source ~/logs_test/check.sh
LOG=~/logs_test/app.log

check "Лог-файл существует"       test -s "$LOG"
check "В логе есть записи ERROR"  grep -q "ERROR" "$LOG"
check "Ошибок не больше 5"        bash -c "[ \$(grep -c 'ERROR' $LOG) -le 5 ]"
check "Есть INFO о запуске"       grep -q "Сервер запущен" "$LOG"
check "Нет слова FATAL"           bash -c "! grep -q 'FATAL' $LOG"
check "Записи содержат дату"      grep -qE "^[0-9]{4}-[0-9]{2}-[0-9]{2}" "$LOG"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Что изучите:** `grep -c` (подсчёт), `grep -q` (тихий поиск), регулярные выражения в `grep -E`.

---

## 📦 Практика 8. Проверка архива с данными

**Тема:** Целостность резервной копии.

**Инструменты:** `tar`, `gzip`, `sha256sum` (есть в системе).

**Идея:** Резервные копии нужно проверять: открывается ли архив, есть ли внутри нужные файлы, не повреждён ли.

### Подготовка

```bash
mkdir -p ~/backup_test/source
cd ~/backup_test/source

echo "user1" > users.txt
echo "order1" > orders.txt
echo "product1" > products.txt

# Создаём архив
cd ~/backup_test
tar -czf backup.tar.gz source/
sha256sum backup.tar.gz > backup.sha256
```

### Тесты

Создайте `test_backup.sh`:

```bash
#!/bin/bash
source ~/logs_test/check.sh
cd ~/backup_test

check "Архив существует"          test -s backup.tar.gz
check "Архив не повреждён"        gzip -t backup.tar.gz
check "Хеш совпадает"             sha256sum -c backup.sha256
check "Внутри есть users.txt"     bash -c "tar -tzf backup.tar.gz | grep -q 'users.txt'"
check "Внутри есть orders.txt"    bash -c "tar -tzf backup.tar.gz | grep -q 'orders.txt'"
check "Файлов в архиве 3"         bash -c "[ \$(tar -tzf backup.tar.gz | grep -c '\.txt$') -eq 3 ]"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Проверка обнаружения сбоя:**

```bash
echo "мусор" >> ~/backup_test/backup.tar.gz
~/backup_test/test_backup.sh
# FAIL: Архив не повреждён, Хеш совпадает
```

**Что изучите:** `tar -tzf` (список содержимого), `gzip -t` (тест целостности), сверку хешей.

---

## 🌐 Практика 9. Проверка конфигурационных файлов

**Тема:** Валидация настроек ИС.

**Инструменты:** `grep`, `awk`, `test` (есть в системе).

**Идея:** Конфиг с ошибкой ломает систему при запуске. Проверим его заранее.

### Подготовка

```bash
mkdir -p ~/config_test
cd ~/config_test

cat > app.conf <<'EOF'
[server]
host=127.0.0.1
port=8080
timeout=30

[database]
host=localhost
port=5432
name=shop
user=admin

[logging]
level=INFO
file=/var/log/app.log
EOF
```

### Тесты

Создайте `test_config.sh`:

```bash
#!/bin/bash
source ~/logs_test/check.sh
CONF=~/config_test/app.conf

check "Конфиг существует"          test -s "$CONF"
check "Секция [server] есть"       grep -q "^\[server\]" "$CONF"
check "Секция [database] есть"     grep -q "^\[database\]" "$CONF"
check "Порт сервера = 8080"        grep -q "^port=8080" "$CONF"
check "Уровень логирования INFO"   grep -q "^level=INFO" "$CONF"
check "Путь к логу абсолютный"     grep -q "^file=/var/log" "$CONF"
check "Нет пустых значений"        bash -c "! grep -qE '^[a-z]+=$' $CONF"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Проверка обнаружения сбоя:**

```bash
echo "user=" >> ~/config_test/app.conf
~/config_test/test_config.sh
# FAIL: Нет пустых значений
```

**Что изучите:** структуру INI-файлов, `grep` с якорями `^` и `$`, `grep -E` для расширенных шаблонов.

---

## 📊 Практика 10. Проверка CSV-данных на корректность

**Тема:** Валидация выгрузок и отчётов.

**Инструменты:** `awk`, `cut`, `sort`, `uniq` (есть в системе).

**Идея:** CSV-файлы — частый формат обмена между системами. Ошибки в них ломают импорт.

### Подготовка

```bash
mkdir -p ~/csv_test
cd ~/csv_test

cat > users.csv <<'EOF'
id,name,email,age
1,Иван,ivan@mail.ru,25
2,Мария,maria@mail.ru,30
3,Пётр,petr@mail.ru,17
4,Анна,anna@mail.ru,22
EOF
```

### Тесты

Создайте `test_csv.sh`:

```bash
#!/bin/bash
source ~/logs_test/check.sh
CSV=~/csv_test/users.csv

check "Файл существует"              test -s "$CSV"
check "Заголовок на месте"           grep -q "^id,name,email,age$" "$CSV"
check "Записей ровно 4"              bash -c "[ \$(tail -n +2 $CSV | wc -l) -eq 4 ]"
check "У всех 4 поля"                bash -c "! awk -F',' 'NF!=4' $CSV"
check "ID уникальны"                 bash -c "test -z \"\$(tail -n +2 $CSV | cut -d',' -f1 | sort | uniq -d)\""
check "Email уникальны"              bash -c "test -z \"\$(tail -n +2 $CSV | cut -d',' -f3 | sort | uniq -d)\""
check "Все email с @"                bash -c "! tail -n +2 $CSV | cut -d',' -f3 | grep -qv '@'"
check "Возраст — числа"              bash -c "! tail -n +2 $CSV | cut -d',' -f4 | grep -qvE '^[0-9]+$'"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Что изучите:** `awk -F','` (разделитель), `cut -d',' -f N` (поле), `uniq -d` (дубликаты), проверку формата.

---

## 🔌 Практика 11. Проверка сетевых портов локально

**Тема:** Какие сервисы слушают порты на машине.

**Инструменты:** `ss`, `netstat`, `/dev/tcp`, `nc` (есть в системе).

**Идея:** Если лишний порт открыт — это угроза безопасности. Если нужный закрыт — сервис не работает.

### Подготовка

```bash
# Запускаем два «сервиса»
python3 -m http.server 8080 &
echo $! > /tmp/svc1.pid

python3 -m http.server 8081 &
echo $! > /tmp/svc2.pid

sleep 1   # даём время на запуск
```

### Тесты

Создайте `test_ports.sh`:

```bash
#!/bin/bash
source ~/logs_test/check.sh

check "Порт 8080 слушается"        bash -c "</dev/tcp/localhost/8080"
check "Порт 8081 слушается"        bash -c "</dev/tcp/localhost/8081"
check "Порт 9999 закрыт"           bash -c "! </dev/tcp/localhost/9999"
check "Сервис 8080 отвечает"       curl -s -o /dev/null --max-time 2 http://localhost:8080/
check "В списке портов есть 8080"  bash -c "ss -tln | grep -q ':8080'"
check "В списке портов есть 8081"  bash -c "ss -tln | grep -q ':8081'"

echo "Пройдено: $PASS, Провалено: $FAIL"
```

**Остановить сервисы:**

```bash
kill $(cat /tmp/svc1.pid) $(cat /tmp/svc2.pid)
```

**Что изучите:** `/dev/tcp` для проверки порта, `ss -tln` (список слушающих TCP-портов), разницу между «порт открыт» и «сервис отвечает».

---

## 🔗 Как связать с предыдущими практиками

Все практики используют **одну функцию `check`** из `~/logs_test/check.sh` — её можно переиспользовать везде. Меняйте только строку `source`.

Итоговый отчёт (`run_report.sh`) можно расширить:

```bash
echo "--- Логи ---"        >> "$REPORT"
bash ~/logs_test/test_logs.sh >> "$REPORT"

echo "--- Архивы ---"      >> "$REPORT"
bash ~/backup_test/test_backup.sh >> "$REPORT"

echo "--- Конфиги ---"     >> "$REPORT"
bash ~/config_test/test_config.sh >> "$REPORT"

echo "--- Расписание ---"  >> "$REPORT"
bash ~/cron_test/test_cron.sh >> "$REPORT"

echo "--- CSV ---"         >> "$REPORT"
bash ~/csv_test/test_csv.sh >> "$REPORT"

echo "--- Порты ---"       >> "$REPORT"
bash ~/test_ports.sh >> "$REPORT"
```


---


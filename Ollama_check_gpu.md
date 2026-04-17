# Инструкция по установке и настройке локального ИИ (Ollama + CodeLlama:13B)


Всего 3 шага: примерно 20 минут вместе с ожиданием скачивания и установкой

---

## 1. Скачать и установить Ollama

### Windows:
1. Перейди на сайт: https://ollama.com/download/OllamaSetup.exe
2. Скачай и запусти установщик
3. Следуй инструкциям установщика
4. После установки Ollama автоматически запустится в фоне (иконка в трее)

Если сервер не запустился автоматически, то:
```powershell
# Проверить, запущен ли сервер
curl http://localhost:11434/api/tags

# Если ошибка, запустить вручную:
ollama serve

# Или через меню Пуск: найти "Ollama" и запустить
```

### Как открыть PowerShell:
- **Способ 1 (быстрый):** Нажми `Win + R`, введи `powershell`, нажми Enter
- **Способ 2 (администратор):** Нажми правой кнопкой по кнопке "Пуск" → "Windows PowerShell (Администратор)"
- **Способ 3 (из папки):** В проводнике нажми `Файл` → `Открыть Windows PowerShell`

---

## 2. Скачать модель CodeLlama

В PowerShell выполни:
```powershell
ollama pull codellama:13b-instruct-q4_K_M
```

**Время загрузки:** 5-15 минут в зависимости от скорости интернета.  
**Размер модели:** ~7.5 GB для 13B версии.

Удалять модель так:
```powershell
ollama rm codellama:13b-instruct-q4_K_M
```

Задаем конфигурацию (одинаково для CPU и GPU):

- создай файл ollama-model-codellama13dev (без расширения) заполни его такими данными:
```powershell
# Modelfile — llama13dev
FROM codellama:13b-instruct-q4_K_M

# Параметры для разработки и анализа
PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER num_ctx 32768

# Ollama сам загрузит максимум слоёв на GPU
```
- выполни в каталоге с этим файлом:
```powershell
ollama create llama13dev -f ollama-model-codellama13dev
```
- теперь можно протестировать:
```powershell
ollama run llama13dev "Сколько будет 1+2?"
```

---

## 3. Проверка установки и скорости

В PowerShell выполни:
```powershell
ollama run codellama:13b-instruct-q4_K_M --verbose "Напиши на python async оптимизатор SQL запросов: переписывание (predicate pushdown, join reorder), выбор плана на основе статистик, cost-based optimization. Верни только код." 2>&1 | Select-String "token|duration|rate|eval"
```

Результат без видеокарты CPU AMD Ryzen 7 PRO 8845HS RAM64gb:
```
total duration:       1m59.9275495s
load duration:        3.171319s
prompt eval count:    73 token(s)
prompt eval duration: 3.6189982s
prompt eval rate:     20.17 tokens/s
eval count:           760 token(s)
eval duration:        1m52.8346494s
eval rate:            6.74 tokens/s
```

Результат с видеокартой 4070super 12gb:
```
total duration:       20.2032412s
load duration:        10.6491ms
prompt eval count:    73 token(s)
prompt eval duration: 43.6858ms
prompt eval rate:     1671.02 tokens/s
eval count:           436 token(s)
eval duration:        16.0624219s
eval rate:            27.14 tokens/s
```

Результат с видеокартой 5070ti 16gb:
```
total duration:        11.6355929s
load duration:         21.662ms
prompt eval count:     73 token(s)
prompt eval duration:  28.141ms
prompt eval rate:      2594.08 tokens/s
eval count:            993 token(s)
eval duration:         11.1216699s
eval rate:             89.29 tokens/s
```

---
---

# Какой результат у тебя? Какая видеокарта? Какой проц, сколько RAM?


---
---

### Что означают параметры фильтрации

`Select-String -Pattern "token|duration|rate|eval"` — это фильтр, который оставляет только строки, содержащие эти слова.

| Ключевое слово | Что ищет |
|----------------|----------|
| `token` | строки со словом "token" (количество токенов) |
| `duration` | строки со словом "duration" (время выполнения) |
| `rate` | строки со словом "rate" (скорость) |
| `eval` | строки со словом "eval" (evaluation — генерация) |

---

### Расшифровка каждой метрики

| Метрика | Что означает | На что влияет |
|---------|--------------|---------------|
| **total duration** | Общее время выполнения запроса | Включает загрузку модели + генерацию |
| **load duration** | Время загрузки модели в память | Если модель уже в памяти, будет близко к 0 |
| **prompt eval count** | Количество токенов в вашем запросе | Чем больше промпт, тем дольше обработка |
| **prompt eval duration** | Время обработки вашего запроса | Зависит от длины промпта |
| **prompt eval rate** | Скорость обработки промпта | Обычно выше 300 ток/сек (быстро) |
| **eval count** | Количество сгенерированных токенов | Главный показатель — сколько кода выдала модель |
| **eval duration** | Время генерации ответа | Основное время работы |
| **eval rate** | **СКОРОСТЬ ГЕНЕРАЦИИ** | Самый важный показатель! |

---

### Как интерпретировать eval rate

| Скорость (ток/сек) | Оценка | Комфортность работы |
|--------------------|--------|---------------------|
| **< 20** | ⚠️ Медленно | Ждать ответа 30-60 секунд |
| **20-35** | 👍 Нормально | Терпимо для 500-1000 токенов |
| **35-50** | 🚀 Хорошо | Быстрая генерация кода |
| **> 50** | ⚡ Отлично | Мгновенные ответы |

---

### Как определить, что GPU работает

Если вы видите:
- `eval rate` **выше 30** — GPU работает
- `eval rate` **ниже 15** — вероятно, только CPU

---

### Упрощенный просмотр метрик (без кода)

```powershell
ollama run codellama:13b-instruct-q4_K_M --verbose "print(1)" 2>&1 | Select-String "eval rate|total duration|eval count"
```

Вывод будет:
```
total duration:       0.85s
eval count:           3 token(s)
eval rate:            15.38 tokens/s
```

---

### Что значит "токен"?

**Токен** — это минимальная единица текста, которую модель обрабатывает:

| Тип текста | Примерно |
|------------|----------|
| 1 английское слово | 1-2 токена |
| 1 русское слово | 2-3 токена |
| 1 символ кода | 0.5-1 токена |
| 100 строк кода | ~500-1000 токенов |

---

### Практический пример

Если вы видите:
```
eval count:    850 token(s)
eval rate:     35 tokens/s
eval duration: 24.3s
```

Это значит:
- Модель сгенерировала **850 токенов** (примерно 200-300 строк кода)
- Скорость **35 ток/сек** (хорошо)
- Ждали **24 секунды**

---

### Быстрая проверка без сложного парсинга

```powershell
ollama run codellama:13b-instruct-q4_K_M --verbose "print(1)" 2>&1 | findstr "eval rate"
```

Просто покажет скорость генерации:
```
eval rate:            57.14 tokens/s
```
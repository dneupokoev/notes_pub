# Инструкция по установке и настройке локального ИИ (Ollama + CodeLlama:13B)

---

## 1. Скачать и установить Ollama

### Windows:
1. Перейди на сайт: https://ollama.com/download/OllamaSetup.exe
2. Скачай и запусти установщик
3. Следуй инструкциям установщика
4. После установки Ollama автоматически запустится в фоне (иконка в трее)

### Как открыть PowerShell:
- **Способ 1 (быстрый):** Нажми `Win + R`, введи `powershell`, нажми Enter
- **Способ 2 (администратор):** Нажми правой кнопкой по кнопке "Пуск" → "Windows PowerShell (Администратор)"
- **Способ 3 (из папки):** В проводнике нажми `Файл` → `Открыть Windows PowerShell`

---

## 2. Скачать модель CodeLlama

```powershell
# CodeLlama 13B
ollama pull codellama:13b-instruct-q4_K_M
```

**Время загрузки:** 5-15 минут в зависимости от скорости интернета.  
**Размер модели:** ~7.5 GB для 13B версии.

---

## 3. Проверка установки и скорости

```powershell
ollama run codellama:13b-instruct-q4_K_M --verbose "Напиши на python async оптимизатор SQL запросов: переписывание (predicate pushdown, join reorder), выбор плана на основе статистик, cost-based optimization. Верни только код." 2>&1 | Select-String "token|duration|rate|eval"
```

Результат у меня без видеокарты:

---

## 4. Запуск сервера (если не запустился автоматически)

```powershell
# Проверить, запущен ли сервер
curl http://localhost:11434/api/tags

# Если ошибка, запустить вручную:
ollama serve

# Или через меню Пуск: найти "Ollama" и запустить
```

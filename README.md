# Сервис работы с переводами

## Стек технологий

- Build Tool: Vite
- Package Manager: Yarn
- Backend: Node.JS, TypeScript, Vitest (?)
- Frontend: Commander.js
- Archive Processor: libarchive.js
- Deployment: Docker
- Code Style: prettier, eslint

## Структура проекта

```
/i18n-tools
│
├── package.json      
├── .eslintrc
├── .prettierrc
│
├── apps/
│   ├── core/              
│   │   ├── package.json
│   │   └── src/
│   │       ├── archive/
│   │       ├── parser/
│   │       ├── normalizer/
│   │       ├── validator/
│   │       └── output/
│   │   
│   │
│   └── cli/              
│       ├── package.json
│       └── src/
│           ├── commands/
│           └── config/
│
└── docker/
    └── Dockerfile
```

## ТЗ

Диаграмма показывает структуру CLI-сервиса как набор взаимосвязанных частей (модулей), 
каждая из которых выполняет свою функцию. 
Сверху располагается основной вход — точка запуска команды сервиса (CLI Entry Point), 
которая управляет конфигурацией и ведет журнал работы (Logging Service).

Далее идет цепочка обработки данных — Processing Pipeline, включающая:
- Archive Processor (распаковка ZIP-архива)
- Language File Parser (чтение текстовых файлов с переводами)
- Data Normalizer (очистка и приведение данных к единому виду)

У каждого из этих модулей есть своя обработка ошибок — Error Handler, 
чтобы ловить и фиксировать проблемы на каждом шаге.

После этого данные идут в блок Validation & Aggregation, где происходит:
- Tag Validator (проверка корректности тегов), 
- Key Aggregator (объединение переводов по ключам), 
- Output Generator (генерация итогового файла в YAML/JSON).
Здесь тоже есть свои обработчики ошибок.

На выходе мы получаем итоговый файл с переводами.

Диаграмма иллюстрирует, как система разбита на четко разделённые этапы, 
где каждый модуль отвечает за конкретную задачу и может быть независимым. 
Это упрощает поддержку, расширение и замену частей сервиса без влияния на остальные.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CLI Entry Point                                    │
│                    (Main Command Handler)                                   │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                    ┌───────────┴────────────┐
                    ▼                        ▼
        ┌──────────────────────┐  ┌──────────────────────┐
        │ Configuration        │  │ Logging Service      │
        │ Manager              │  │ (Logger Interface)   │
        └──────────┬───────────┘  └──────────┬───────────┘
                   │                         │
        ┌──────────┴─────────────────────────┴──────────┐
        ▼                                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Processing Pipeline                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │   Archive    │  │  Language    │  │  Data        │               │
│  │  Processor   │→ │ File Parser  │→ │ Normalizer   │               │
│  │ (ZIP Handler)│  │ (YAML/JSON)  │  │ (Cleaner)    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│         ↓                 ↓                   ↓                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │    Error     │  │    Error     │  │    Error     │               │
│  │   Handler    │  │   Handler    │  │   Handler    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Validation & Aggregation                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │  Tag         │  │  Key         │  │ Output       │               │
│  │ Validator    │→ │ Aggregator   │→ │ Generator    │               │
│  │ (Tag Format) │  │ (Merger)     │  │ (YAML/JSON)  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│         ↓                 ↓                ↓                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │    Error     │  │    Error     │  │    Error     │               │
│  │   Handler    │  │   Handler    │  │   Handler    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                                ▼
                    ┌──────────────────────┐
                    │  Output File         │
                    │  (translations.yaml) │
                    └──────────────────────┘
```
```
User          CLI Entry      Archive      Language    Data           Key           Output
               Point        Processor    File Parser  Normalizer    Aggregator    Generator
 │               │              │           │          │              │             │
 │─ upload.zip──→│              │           │          │              │             │
 │               │              │           │          │              │             │
 │               │─ validate ───→│          │          │              │             │
 │               │              │─ ACK ────→           │              │             │
 │               │              │          │          │               │             │
 │               │─ extract ────→│          │         │              │             │
 │               │              │─ files ──→          │              │             │
 │               │              │          │          │               │             │
 │               │                         │─ parse ──→               │             │
 │               │                         │          │              │             │
 │               │                         │          │─ normalize ──→              │
 │               │                         │          │              │             │
 │               │                         │          │              │─ aggregate ─→
 │               │                         │          │              │              │
 │               │                         │          │              │              │─ generate
 │               │                         │          │              │              │  YAML file
 │               │                         │          │              │              │
 │               │◄─ result ───────────────────────────────────────────────────────│
 │◄── success ───│              │           │          │              │             │
 │  (YAML file)  │              │           │          │              │             │
```
---
```
INPUT: ZIP Archive
         ↓
    ┌────────────────────────────────────────┐
    │ Archive Processor                      │
    │ • Валидация ZIP                        │
    │ • Извлечение файлов                    │
    │ • Проверка имён файлов                 │
    │ Выход: { langCode → RawContent }       │
    └────────────────────────────────────────┘
         ↓
    ┌────────────────────────────────────────┐
    │ Language File Parser                   │
    │ • Парсинг YAML/JSON                    │
    │ • Извлечение пар ключ-перевод          │
    │ Выход: { langCode → { key → value } }  │
    └────────────────────────────────────────┘
         ↓
    ┌────────────────────────────────────────┐
    │ Data Normalizer                        │
    │ • Удаление лишних пробелов             │
    │ • Нормализация символов                │
    │ • Приведение к стандартному формату    │
    │ Выход: { langCode → { key → clean } }  │
    └────────────────────────────────────────┘
         ↓
    ┌────────────────────────────────────────┐
    │ Tag Validator                          │
    │ • Проверка тегов                       │
    │ • Валидация строк                      │
    │ • Проверка синтаксиса                  │
    │ Выход: ValidationReport                │
    └────────────────────────────────────────┘
         ↓
    ┌────────────────────────────────────────┐
    │ Key Aggregator                         │
    │ • Объединение по ключам                │
    │ • Обнаружение дубликатов               │
    │ • Выявление недостающих переводов      │
    │ Выход: { key → { lang → translation }} │
    └────────────────────────────────────────┘
         ↓
    ┌────────────────────────────────────────┐
    │ Output Generator                       │
    │ • Сериализация в YAML/JSON             │
    │ • Форматирование                       │
    │ • Запись в файл                        │
    │ Выход: translations.yaml               │
    └────────────────────────────────────────┘
```

---

## Описание ключевых модулей

### Archive Processor (Обработчик архивов)

**Назначение**: Загрузка и извлечение содержимого ZIP-архива с валидацией

**Входные данные**:
- Путь к ZIP-файлу (string)
- Конфигурация: максимальный размер, расширения

**Выходные данные**:
```
{
    "status": "success" | "error",
    "files": {
        "en.txt": "<raw file content>",
        "ru.txt": "<raw file content>",
        "zh.txt": "<raw file content>"
    },
    "metadata": {
        "archive_size": 1024,
        "files_count": 3,
        "languages": ["en", "ru", "zh"]
    },
    "errors": [
        {
            "type": "InvalidFileName",
            "file": "invalid_name.txt",
            "message": "Invalid language code format"
        }
    ]
}

```
---
### Парсер языковых файлов

**Назначение**: Парсинг файлов с переводами

**Входные данные**:
```
{
    "lang_code": "en",
    "content": "key1: value1\nkey2: value2",
    "format": "yml"
}
```
*Ключ можно попробовать генерировать нейронкой*

**Выходные данные**:
```
{
    "status": "success" | "partial" | "error",
    "language": "en",
    "translations": {
        "messages.greeting": "Hello",
        "messages.farewell": "Goodbye",
        "dialogs.start_chat": "Start a {{bold}}Full-Private Chat{{/bold}}..."
    },
    "statistics": {
        "total_keys": 100,
        "successfully_parsed": 98,
        "malformed_lines": 2
    },
    "warnings": [
        {
            "line": 15,
            "type": "MalformedLine",
            "content": "key without value"
        }
    ]
}
```

---

### Data Normalizer

**Назначение**: Очистка и нормализация текстовых данных

**Входные данные**:
```
{
    "language": "en",
    "key": "messages.greeting",
    "value": "  Hello   \n World  ",
    "tags": ["bold", "wrapper"]
}
```

**Выходные данные**:
```
{
    "language": "en",
    "key": "messages.greeting",
    "normalized_value": "Hello World",
    "normalized_tags": [
        {
            "name": "bold",
            "balanced": true
        },
        {
            "name": "wrapper",
            "balanced": true
        }
    ],
    "changes_applied": [
        "whitespace_trimmed",
        "control_chars_removed",
        "tag_normalized"
    ]
}
```

**Правила нормализации**:
- Удаление лишних пробелов
- Замена множественных пробелов на одинарный
- Удаление недопустимых управляющих символов
- Нормализация переносов строк
- Унификация кавычек

---

### Tag Validator (Валидатор тегов)

**Назначение**: Проверка корректности форматных тегов

**Входные данные**:
```
{
    "language": "en",
    "key": "messages.start_chat",
    "value": "Start a {{bold}}Full-Private Chat{{/bold}} with {{wrapper}}...",
}
```

**Выходные данные**:
```
{
    "is_valid": true,
    "errors": [],
    "warnings": [
        {
            "type": "UnclosedTag",
            "tag": "wrapper",
            "position": 42
        }
    ],
    "tag_tree": [
        {
            "name": "bold",
            "start": 10,
            "end": 32,
            "content": "Full-Private Chat",
            "nested": []
        }
    ]
}
```

**Поддерживаемые теги**:
- `{{tag}}...{{/tag}}` — парные теги
- `{{tag/}}` — самозакрывающиеся теги
- `%placeholder%` — плейсхолдеры

**Правила валидации**:
- Все открывающие теги должны быть закрыты
- Теги не должны перекрываться
- Плейсхолдеры должны быть уникальны в рамках строки

---

### Key Aggregator (Агрегатор ключей)

**Назначение**: Объединение переводов по ключам из разных языков

**Входные данные**:
```
{
    "en": {
        "messages.greeting": "Hello",
        "messages.farewell": "Goodbye"
    },
    "ru": {
        "messages.greeting": "Привет",
        "messages.farewell": "До свидания"
    },
    "zh": {
        "messages.greeting": "你好"
    }
}
```

**Выходные данные**:
```
{
    "status": "success",
    "aggregated": {
        "messages.greeting": {
            "en": "Hello",
            "ru": "Привет",
            "zh": "你好"
        },
        "messages.farewell": {
            "en": "Goodbye",
            "ru": "До свидания"
        }
    },
    "coverage": {
        "messages.greeting": {
            "total_languages": 3,
            "available_languages": 3,
            "coverage_percent": 100
        },
        "messages.farewell": {
            "total_languages": 3,
            "available_languages": 2,
            "coverage_percent": 66.67
        }
    },
    "duplicates": [
        {
            "language": "en",
            "key": "messages.greeting",
            "occurrences": 2,
            "action": "first_kept"
        }
    ],
    "missing_keys": {
        "zh": ["messages.farewell"]
    }
}
```

**Правила агрегации**:
- Объединение по ключу
- Обнаружение и обработка дубликатов
- Генерация отчёта о полноте перевода
- Выявление отсутствующих ключей

---

### Output Generator (Генератор выходных данных)

**Назначение**: Сериализация агрегированных данных в YAML/JSON

**Входные данные**:
```
{
    "aggregated": {
        "messages.greeting": {
            "en": "Hello",
            "ru": "Привет",
            "zh": "你好"
        }
    },
    "metadata": {
        "languages": ["en", "ru", "zh"],
        "total_keys": 1,
        "generation_date": "2025-11-27"
    },
    "format": "yaml",
    "output_path": "./translations.yaml"
}
```

**Выходные данные**: Файл translations.yaml

```yaml
metadata:
  languages:
    - en
    - ru
    - zh
  total_keys: 1
  generation_date: "2025-11-27"
  coverage:
    messages.greeting: 100%
    messages.farewell: 66.67%

translations:
  messages:
    greeting:
      en: "Hello"
      ru: "Привет"
      zh: "你好"
    farewell:
      en: "Goodbye"
      ru: "До свидания"
```

**Поддерживаемые форматы**: YAML, JSON, XML

---

### Error Handler (Обработчик ошибок)

**Назначение**: Централизованная обработка, логирование ошибок

**Входные данные**: Exception/Error объект

**Выходные данные**:
```
{
    "error_id": "ERR_001",
    "type": "ValidationError",
    "severity": "critical" | "warning",
    "message": "Tag mismatch in file ru.txt",
    "context": {
        "file": "ru.txt",
        "language": "ru",
        "line": 42
    },
    "action": "continue" | "retry" | "abort",
    "logged": true
}
```

**Типы ошибок**:
- **Critical**: архив не открывается, невозможно скопировать выходной файл
- **Warning**: парсинг ошибок, проблемы с тегами, отсутствующие ключи, низкое покрытие переводов

---

### Configuration Manager (Менеджер конфигурации)

**Назначение**: Управление настройками и параметрами сервиса

**Входные данные**: Файл конфигурации (config.yaml) или CLI аргументы

**Структура конфигурации**:
```yaml
archive:
  max_size_mb: 100
  allowed_extensions: [".txt", ".yaml", ".json"]
  allowed_lang_codes: ["en", "ru", "zh", "es", "fr"]

parsing:
  format: "yaml"
  strict_mode: false

normalization:
  remove_whitespace: true
  normalize_unicode: true

validation:
  check_tag_balance: true
  check_placeholders: true

aggregation:
  duplicate_handling: "first"  # first, last, error
  allow_partial_coverage: true

output:
  format: "yaml"
  include_metadata: true
  include_coverage_report: true
  pretty_print: true

logging:
  level: "INFO"
  output: "console,file"
  file_path: "./logs/translation-aggregator.log"
```

---

### Logging Service (Служба логирования)

**Назначение**: Структурированное логирование событий и ошибок
---

### Уровни ошибок

```
┌────────────────────────────────────────────────────┐
│                    ERROR LEVELS                    │
├────────────────────────────────────────────────────┤
│                                                    │
│ CRITICAL (Abort)                                   │
│ ├─ Invalid ZIP file                                │
│ ├─ No language files found                         │
│ ├─ Output directory not writable                   │
│ └─ Memory limit exceeded                           │
│                                                    │
│ WARNING (Continue with report)                     │
│ ├─ Malformed lines in file                         │
│ ├─ Unbalanced tags detected                        │
│ ├─ Duplicate keys in language file                 │
│ └─ Encoding detection issues                       │
│                                                    │
│ INFO (Log only)                                    │
│ ├─ Missing translation for key                     │
│ ├─ Low coverage for language                       │
│ ├─ Normalization applied                           │
│ └─ Processing statistics                           │
└────────────────────────────────────────────────────┘
```

### Обработка ошибок

```
Error Detected
       ↓
  Error Handler
       ↓
  ┌──────────────────────────┐
  │ Determine Severity Level │
  └────────┬─────────────────┘
           ↓
    ┌──────────────────┐
    │ CRITICAL?        │
    └────┬─────────┬───┘
         │ YES     │ NO
         ↓         ↓
      ABORT   ┌──────────────┐
              │ WARNING?     │
              └────┬────┬────┘
                   │ Y  │ N
                   ↓    ↓
              REPORT  LOG
                  ↓
             CONTINUE
```

### Recovery Points (Точки восстановления)

- **Phase 1 (Archive)**: Если ошибка — выход с ошибкой
- **Phase 2 (Parsing)**: Если ошибка в одном файле — пропуск, продолжение
- **Phase 3 (Normalization)**: Если ошибка — применение значения по умолчанию
- **Phase 4 (Validation)**: Если ошибка — логирование, продолжение
- **Phase 5 (Aggregation)**: Если ошибка — использование доступных данных
- **Phase 6 (Output)**: Если ошибка — перейти в файл с отчётом об ошибке

---

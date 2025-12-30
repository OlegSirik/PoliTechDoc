# Print Subsystem Documentation

## Purpose

Подсистема печати предназначена для генерации PDF-документов
(печатных форм) страховых договоров.

Подсистема обеспечивает:
- воспроизводимость печати
- поддержку версионирования
- независимость от бизнес-логики расчёта

---

## High-Level Flow

Print Request
→ Resolve Template
→ Load Policy Snapshot
→ Build ContextVar
→ Resolve Print Fields
→ Render PDF


---

## Input Parameters

### Print Request

```text
policyNumber
printFormCode
```

## Step-by-Step Process
### 1. Policy Lookup

По policyNumber определяется:
версия договора
productCode
productVersion
JSON-снапшот договора

### 2. Resolve Print Template

Печатный шаблон выбирается по ключу:
(productCode, productVersion, printFormCode)

Результат:
printTemplateId
версия шаблона
формат шаблона (PDF / HTML / etc.)

### 3. Load Policy Snapshot

Из БД загружается:
JSON договора
метаданные версии
Используется только snapshot, без обращения к live-данным.

### 4. Rebuild ContextVar

Процесс:
JSON Snapshot
 → PolicyDTO
   → ContextVar

ContextVar формируется по тем же правилам,
что и при расчёте договора.

### 5. PrintContext Resolution

На основе ContextVar создаётся PrintContext, который:
предоставляет доступ по именам полей
форматирует значения
обрабатывает null-значения

Пример доступа:
policy.number
insured.fullName
premium.total
product.version

### 6. Template Rendering

Печатный шаблон:
содержит список именованных полей
не знает структуру договора
не содержит бизнес-логики
Значения подставляются через PrintContext.

### 7. PDF Generation

Результат:
PDF-файл
полностью воспроизводимый
соответствующий версии договора

### Print Template Storage

Печатные шаблоны хранятся в БД и версионируются вместе с продуктом.

Атрибуты шаблона
templateId
productCode
productVersion
printFormCode
templateVersion
templateBody

### Versioning Rules

Один продукт → несколько версий печатных шаблонов
Старая версия шаблона не изменяется
Печать старого договора использует старый шаблон

### Rules & Restrictions

❌ Печатная форма не выполняет расчётов
❌ Печатная форма не изменяет договор
❌ Печать не использует live-сервисы

✔ Только snapshot
✔ Только ContextVar
✔ Только именованные поля

### Benefits

Регуляторная готовность
Полный аудит
Простота поддержки
Единый контракт данных

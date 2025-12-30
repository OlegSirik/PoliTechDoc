# ADR-004: Print Rendering Architecture

## Status
Accepted

## Date
2025-XX-XX

## Context

Система должна поддерживать генерацию печатных форм (PDF) для договоров страхования
с учётом:

- версии продукта
- версии договора
- регуляторных требований
- воспроизводимости печати во времени

Печатная форма должна:
- соответствовать состоянию договора на момент его версии
- быть независимой от runtime-состояния системы
- поддерживать версионирование и аудит

---

## Decision

Принято решение:

1. Хранить **печатные шаблоны в БД**.
2. Версионировать печатные шаблоны **вместе с версией продукта**.
3. Генерировать печать **на основе JSON-снапшота договора**, сохранённого в БД.
4. Использовать `ContextVar` как единый источник данных для печати.
5. Разрешать шаблонам доступ **только к именованным полям контекста**.

---

## Architecture Overview

~~~
Print Request
(policyNumber, printFormCode)
│
▼
Policy Lookup
│
▼
Resolve Print Template ID
(by productCode + productVersion + printFormCode)
│
▼
Load Policy JSON Snapshot
│
▼
Rebuild ContextVar
│
▼
PrintContext Resolver
│
▼
Template Rendering Engine
│
▼
PDF Output
~~~


## Consequences

### Positive
- Полная воспроизводимость печати
- Поддержка аудита и регуляторных проверок
- Независимость от изменений бизнес-логики
- Возможность печати старых договоров

### Negative
- Дополнительная логика разрешения шаблонов
- Необходимость поддержки версий шаблонов

---

## Constraints

- Печатный шаблон не выполняет расчётов
- Печатный шаблон не изменяет договор
- Печать не зависит от live-данных
- ContextVar используется в read-only режиме

---

## Alternatives Considered

### Генерация печати из Entity
❌ Отказано — Entity может не отражать историческое состояние договора

### Генерация печати напрямую из DTO
❌ Отказано — DTO не гарантирует воспроизводимость

### Хранение шаблонов вне БД
❌ Отказано — сложнее обеспечить версионирование и аудит

---

## Related Documents

- ADR-001-policy-calculation-architecture
- ADR-002-contextvar-anti-corruption-layer
- policy-calculation-process.md
- print-subsystem.md












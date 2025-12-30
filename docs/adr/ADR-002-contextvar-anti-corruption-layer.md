# ADR-002: ContextVar as Anti-Corruption Layer for Calculation

## Status
Accepted

## Date
2025-XX-XX

## Context

Система расчёта страховой премии должна:
- быть изолированной от структуры договора
- не зависеть от API / UI / Persistence моделей
- оставаться стабильной при изменении доменной модели

Прямая передача `PolicyDTO` или доменных сущностей в калькулятор
приводит к высокой связности и сложности сопровождения.

---

## Decision

Принято решение ввести отдельную модель **ContextVar**, которая:

- используется **исключительно** для расчёта
- является плоской структурой данных
- служит анти-коррупционным слоем между договором и калькулятором

Калькулятор:
- принимает `ContextVar` на вход
- возвращает `ContextVar` (или `CalculationResult`) на выход

---

## Definition of ContextVar

`ContextVar` — это:
- набор примитивных и value-типов
- без ссылок на доменные объекты
- без бизнес-поведения

Допустимые типы:
- `BigDecimal`
- `Long`
- `Integer`
- `Boolean`
- `String`
- `LocalDate`
- `Enum`

Недопустимые:
- `Policy`
- `Coverage`
- `Entity`
- `DTO`
- `Repository`
- `Service`

---

## Responsibilities

ContextVar отвечает за:
- передачу входных параметров в калькулятор
- получение результатов расчёта
- обеспечение стабильного контракта калькулятора

ContextVar НЕ отвечает за:
- валидацию API
- хранение данных
- бизнес-оркестрацию

---

## Consequences

### Positive
- калькулятор полностью изолирован
- упрощён рефакторинг доменной модели
- возможна смена API без изменения расчёта
- поддержка explainable pricing

### Negative
- дополнительный слой маппинга
- необходимость дисциплины в развитии модели

---

## Alternatives Considered

### Passing PolicyDTO directly
❌ Tight coupling with API model

### Passing Domain Entities
❌ Strong coupling, persistence leakage

---

## Related Documents

- ADR-001-policy-calculation-architecture
- policy-calculation-process.md

# Insurance Premium Calculator

## Purpose

Calculator — это изолированный модуль для расчёта страховой премии.

Он:
- не знает о договоре
- не знает об API
- не знает о БД
- работает только с ContextVar

---

## Design Principles

- Stateless
- Deterministic
- Side-effect free
- Pure function

---

## Input

Калькулятор принимает объект `ContextVar`, содержащий:
- нормализованные данные
- значения, подготовленные оркестратором

Пример входных данных:
```text
insuredAge
termInMonths
sumInsured
riskClass
productVersion
~~~


Output

Результат расчёта включает:

premium

coefficients

tariffVersion

calculationTrace (optional)

What Calculator MUST NOT do

❌ Access database
❌ Use PolicyDTO or Entity
❌ Perform product enrichment
❌ Handle JSON
❌ Perform orchestration

Versioning

Каждый калькулятор имеет версию:

calcVersion = MAJOR.MINOR.PATCH


Версия сохраняется вместе с договором.

Testing Strategy

Unit tests for each rule

Golden test cases

Regression test sets

No Spring context required

Example Usage
ContextVar input = contextMapper.fromPolicy(policyDto);
ContextVar result = calculator.calculate(input);

Ownership

Calculator module is owned by pricing / actuarial team

Changes must be backward compatible or versioned

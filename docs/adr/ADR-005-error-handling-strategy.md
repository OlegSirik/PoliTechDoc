# ADR-005: Error Handling Strategy

## Status
Accepted

## Date
2025-XX-XX

## Context

Система создания и обработки страховых договоров включает
многоэтапный процесс:

- валидация входных данных
- обогащение по метаданным продукта
- расчёт
- сохранение
- печать

Необходимо единообразно и предсказуемо обрабатывать ошибки,
чтобы:
- API было стабильным
- frontend мог корректно реагировать
- ошибки были аудируемыми

---

## Decision

Принято решение:

1. Использовать HTTP-коды в зависимости от ответственности за ошибку.
2. Чётко разделять:
   - ошибки клиента (4xx)
   - ошибки системы (5xx)
3. Использовать единый Error Contract для всех API.
4. Не пробрасывать технические исключения напрямую клиенту.

---

## Error Classification

### Client Errors (4xx)

| Code | Meaning |
|---|---|
| 400 Bad Request | Некорректный формат или структура запроса |
| 422 Unprocessable Entity | Ошибка бизнес-валидации |

### Server Errors (5xx)

| Code | Meaning |
|---|---|
| 500 Internal Server Error | Ошибка обработки корректного запроса |
| 503 Service Unavailable | Временная недоступность зависимости |

---

## Key Principles

- HTTP-код отражает **вину**, а не этап процесса
- Бизнес-валидации → `422`
- Неожиданные исключения → `500`
- Валидация запроса → `400`

---
## Error Response Model

Для всех API используется единый формат ошибок `ErrorModel`.

```json
{
  "code": 422,
  "message": "Policy validation failed",
  "errors": [
    {
      "domain": "POLICY",
      "reason": "REQUIRED",
      "message": "Birth date is required",
      "field": "insured.birthDate"
    }
  ]
}
```

## Consequences

### Positive
- Предсказуемое API
- Простота интеграции frontend
- Чёткая граница ответственности

### Negative
- Требуется дисциплина в реализации
- Необходимо поддерживать error-коды

---

## Related Documents

- policy-api-errors.md
- error-contract.md
- ADR-004-print-rendering-architecture.md

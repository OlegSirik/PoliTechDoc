# Error Contract

## Purpose

Единый контракт ошибок для всех API,
используемый frontend и внешними интеграциями.

---

## Error Response Format

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Policy data is invalid",
  "details": [
    {
      "field": "insured.birthDate",
      "code": "REQUIRED",
      "message": "Birth date is required"
    }
  ],
  "traceId": "abc-123"
}
```

| Field   | Description                      |
| ------- | -------------------------------- |
| error   | Машинно-читаемый тип ошибки      |
| message | Краткое описание                 |
| details | Детализация ошибок (опционально) |
| traceId | Идентификатор запроса для логов  |


| error                  | HTTP |
| ---------------------- | ---- |
| BAD_REQUEST            | 400  |
| VALIDATION_ERROR       | 422  |
| CALCULATION_ERROR      | 422  |
| PRODUCT_METADATA_ERROR | 422  |
| INTERNAL_ERROR         | 500  |
| SERVICE_UNAVAILABLE    | 503  |


{
  "field": "coverage.limit",
  "code": "EXCEEDS_MAX",
  "message": "Limit exceeds product maximum"
}

| Code           | Meaning                   |
| -------------- | ------------------------- |
| REQUIRED       | Обязательное поле         |
| INVALID_FORMAT | Неверный формат           |
| NOT_ALLOWED    | Значение не разрешено     |
| EXCEEDS_MAX    | Превышен лимит            |
| OUT_OF_RANGE   | Вне допустимого диапазона |

Frontend Guidelines

400 → ошибка формы / структуры

422 → подсветка бизнес-ошибок

500 → общий экран ошибки

traceId использовать при обращении в поддержку



# Предрасчет

Набор полей для предрасчета определяется калькулятором и настройками валидатора предрасчета.

Минимальный обязательный набор полей договора   
1. Код продукта
2. Даты - дата начала-окончания, либо период ожидания-страхования, либо ничего, если это всегда фиксы
3. Пакет - по умолчанию 0, если не передали
4. Страховая сумма - может отсутствовать если это фикс 

```
{
  "productCode": "string",

  "startDate": "string",
  "endDate": "string",
  "issueDate": "string",
  "waitingPeriod": "P16D",
  "policyTerm": "P100D",

  "packageCode": "BASIC",

  "insuredObjects": [
    {
      "ioType": "Person",
      "sumInsured": "10000", 
    },
  ]
```
## Процесс
### draft_id
1. Если draftId не пустой, то сгенерить UUID.
### insCompanyCode
пока не валидируется
### product
Получить текущую версию продукта и pt_products. 
в productVersionNo записать номер сервии
### Проверка
Выполнить настроенную для версии продукта проверку предрасчета.
Если есть ошибки, вернуть их. 
Прервать процесс.
### policyNumber
= ''
### previousPolicyNumber
не валидируется
### status
status = NEW
### даты
Для предрасчета даты не обязательны, если они нужны, должна быть настроена проверка предрасчета.
Если можно расчитать даты и периоды, то расчитать, Если нет то остаются пустыми.
"startDate", "endDate", "issueDate", "waitingPeriod", "policyTerm"
createDate = текущая дата и время создания расчета в БД.
### placeOfIssue
не валидируется
### premium
premium = 0
```
  "draftId": "string",
  "insCompanyCode": string,
  "policyNumber": "string",
  "previousPolicyNumber": "string",
  "product": "string",
  "productVersNo": "string", 
  "status": "NEW",
  "startDate": "string",
  "endDate": "string",
  "issueDate": "string",
  "waitingPeriod": "P16D",
  "policyTerm": "P100D",
  "createDate": "string",
  "premium": 1000,
  "placeOfIssue": "string",
```
запустить засчет, сохранить расчет в БД.
